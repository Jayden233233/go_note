# 1.什么是变量

内存中开辟了空间，这个空间可以存放值，同时空间也有一个地址范围，把这个物理上的空间和空间的范围定义为一个变量
即变量的实质可以理解为 （空间存放的值，空间的地址范围）这样的二元组

# 2.什么是变量赋值

`a=9 或 a = x`, 就是把对应的物理空间上存放特定的值

# 3.变量的类型怎么理解

从物理存储的角度来看，变量分为
值类型变量 go中的基本类型 struct。java中的基本类型变量（空间存放的数值，空间的地址）
指针类型变量 go中用&取了地址后指针变量， java中的引用类型。	 变量（空间存放的指向其他位置的地址数值，空间的地址）

例如java中 `A a = new A();`。 a的值就是对`new A();` 这个堆上空间取了地址&， 只是java中取地址的操作是隐式的。
例如go中，`a := A{}`  a就是值类型，只要有“赋值”或者“传递”操作就会开辟新的空间。 `a := &A{}` ，这时a就是指针变量，类似于java中的引用，a进行传递时就是pass by value of reference

# 4.java中的传递方式有：pass by value 和 pass by value of reference。

​	**java引用类型的变量就是指针变量，同理，怎么理解指针变量？把它当成java中引用类型 。**
​	传递时，就是开辟了新的空间，并把原有的指针变量的值复制了一份。pass by value of reference
​	

# 5.go语言中的传递方式有，pass by value 和 pass by value of reference，pass by pointer 

​    pass by pointer 强调在传递时，要有取地址的操作
​	因此，a := &struct A{},这是一个 pass by pointer。 ivokeFuc(a), 这是一个 pass by value of reference
​	即想要发生pass by value of reference，必然先发生了取地址pass by pointer 的操作，只是java中取指针和解指针的操作都是在底层完成，语言层面没有这样的语法。

指针变量传递到方法中后，产生的效果。

# 6.方法中如果修改指针的值，不会对原有变量有任何影响，相当于让指针指向了新的内存区域。

java 和go中都是同样的逻辑。

```
func modifySlicePtr(slicePtr *[]int) {
	slicePtr = &[]int{10, 20}//指针变量指向了新地址
}  
```



# 7.方法中如果修改了“解指针”的值，那么就是修改了方法外的内存空间。

java中没有解指针的操作，但go中存在。

```
func modifySlicePtr(slicePtr *[]int) {
    newSlice := []int{10, 20}
    *slicePtr = newSlice  // *slicePtr = newSlice直接修改了原指针指向的内存空间的值，=号是赋值，也是复制操作
}  
```



# 8.方法中对解指针赋值给了新的变量，那么肯定发生了开辟新空间的动作，对新的变量的操作，不影响原有内存空间。

```
func modifySlicePtr(slicePtr *[]int) {
	t ：= *slicePtr
    newSlice := []int{10, 20}
    t = newSlice 
}  
```



# 9关于传递方式

值传递（开辟新的变量空间）
	pass by value
	pass by pointer 
	pass by value of reference
	pass by value of struct
	
引用传递：存在与c和c++中

关于传递方式可以参考：https://github.com/Jayden233233/go_note/blob/master/%E5%80%BC%E4%BC%A0%E9%80%92%E4%B8%8E%E5%BC%95%E7%94%A8%E4%BC%A0%E9%80%92.md