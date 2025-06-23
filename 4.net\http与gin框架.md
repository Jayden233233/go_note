### 零、设计Web框架的核心特性  
1. **核心特性**  
   - **高性能**：路由解析速度快，内存占用少。  
   - **中间件支持**：支持全局、路由组和单个路由的中间件。  
   - **参数绑定**：自动解析JSON/XML/表单参数到结构体。  
   - **错误处理**：统一的错误处理机制。  
   - **内置渲染**：支持JSON、XML、HTML等多种响应格式。  
   - **丰富的HTTP方法**：支持GET、POST、PUT、DELETE等常用请求方法。  


### 一、net/http包的核心数据结构和功能  
#### 1. 服务端的职责  
**研究对象**：服务端数据结构、路由handler注册、服务端启动  
**数据结构**：  
```go
type Server struct {
    Addr string         // server地址
    Handler Handler     // 路由处理器，若为nil则使用http.DefaultServeMux
    // ...
}

type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}

type ServeMux struct { // 路由处理器实现，通过map维护路径到handler的映射
    mu sync.RWMutex
    m map[string]muxEntry
    es []muxEntry // 按路径长度排序的条目切片
    hosts bool    // 是否包含主机名模式
}

type muxEntry struct {
    h Handler
    pattern string 
}
```  
**注册流程**：将注册入口的函数转为Handler接口对象，注册到路由处理器的map（精确匹配）和list（前缀匹配）。  
**启动流程**：开启socket监听（可探寻epoll细节），通过ServeMux路由处理器，以map或前缀匹配方式找到处理方法。  

#### 2. 客户端的职责  
- **职责**：组装请求体，管理cookie、连接池及单个连接的消息处理。  
- **消息处理模型**：
  ```  

  主go程  <--channel--> 接收消息的go程 
          <--channel--> 发送消息go程
  后两者理解为守护go程
  
  ```  
 
- **设计目的**：  
  - 解耦：主go程与异步go程独立操作，无同步等待。  
  - 适配context模式：异步go程可设置结束时间。  
  - 连接绑定：守护的go程是和连接池中的一个连接绑定的，即主go和异步go的生命周期也是独立的


### 二、gin与net/http的关系  
#### 1. Gin对net/http的增强  
| 功能         | net/http                | Gin                          |  
|--------------|-------------------------|------------------------------|  
| 路由系统     | 简单路径匹配，不支持参数 | 基于httprouter，支持`:param`和`*wildcard` |  
| 中间件       | 需手动实现链模式        | 内置中间件机制，支持全局/分组/单个路由 |  
| 参数解析     | 手动解析表单、JSON等    | 自动绑定结构体，支持校验       |  
| 错误处理     | 需手动处理并返回响应    | 统一错误处理和恢复中间件       |  
| 响应格式     | 原生支持text/plain等    | 内置JSON/XML/HTML等多种渲染方式 |  
| 性能         | 基础实现，性能一般      | 高性能路由匹配（前缀树结构）   |  

#### 2. Gin与net/http的结合方式  
- **a. 组件替换**：Gin实现`gin.Engine`作为外层路由处理器，替换net/http的`ServeMux`，启动流程依赖net/http但修改部分组件实现。  
- **b. 数据结构与流程**：  
  - `gin.Engine`引用context、路由组和压缩前缀树（路由更灵活）。  
  - **注册流程**：路由组对象调用注册中间的方法，会把中间件挂载到handlerChain中。路由组对象调用注册handler方法，最终路径要拼接路由组中的路径，最终handler也要拼接路由组中的handlerchain，然后把最终路径和最终handlerchain加入到压缩前缀树中
  - **启动流程**：开启socket监听，找到gin.engin这个最外层的路由处理器，【重置gin.engin中的gin.Context】并从压缩前缀树中找到 handlerchain。调用handlerchain的方式是，把handlerchain加入到gin.Context
- **c. Gin.Context**  
  - **数据结构**：包含Request/Writer、handlers链、处理进度索引、Engine指针、互斥锁及共享数据map。  
  - **特点**：可复用，从缓存池获取context，处理后回收。  
  - **使用**：`Context.Next()`依次调用handlerChain，每个handler中都持有上下文对象；支持手动打断流程，模拟切面效果。  ；通过锁和map共享数据；


### 三、其他思考  
- **channel与socket**：channel是阻塞队列，socket连接（如Netty的channel）也可理解为阻塞队列，本质是“用通信共享内存，而非用内存通信”。  
- **闭包与函数转换**：`type ABC func()`是闭包载体而非闭包本身，闭包是运行时概念，关注外部变量引入；net/http将func转换为Handler接口对象时涉及此类转换。  


### 四、设计微服务框架的核心特性对比  
| 特性       | Gin                     | go-zero                  |  
|------------|-------------------------|--------------------------|  
| 服务发现   | 需手动集成（如etcd/clientv3） | 内置etcd支持             |  
| 负载均衡   | 需第三方库              | 内置加权轮询、一致性哈希 |  
| 熔断/限流  | 需第三方库（如hystrix-go） | 内置令牌桶、断路器       |  
| RPC框架    | 需额外集成（如gRPC）     | 内置go-zero/zrpc         |


参考：
https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484040&idx=1&sn=b710f4429188ea5f49f6a9155381b67f
https://mp.weixin.qq.com/s?__biz=MzkxMjQzMjA0OQ==&mid=2247484076&idx=1&sn=9492d326c820625700345a881b58a849
