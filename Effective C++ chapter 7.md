## 7 模版与泛型编程
### 7.1 typename
```c++
template<typename C>
void print(const C& container) {
    if (container.size() >= 2) {
    	//此处使用typename是为了指出C::const_iterator是一个类型（嵌套从属类型名），而不是变量
        typename C::const_iterator iter(container.begin());
        ++iter;
        int value = *iter;
        cout<<value;
    }
}
```

typename 不得在 base class lists  / member ini list 修饰
```c++
template<typename Company>  
class derived : public base</*typename*/Company> {  
public:  
    derived():base</*typename*/Company>(){}  
    using base<Company>::f;  
    void f_der() {  
        f();  
    }  
};
```
### 7.2 模版基类
```c++
template<typename Company>  
class base {  
public:  
    void f() {  
          
    }  
};  
template<typename Company>  
class derived : public base<Company> {  
    void f_der() {  
	    // 此处无法通过编译
        f();  
    }  
};

```
派生类 derived 继承于 base\<Company> ,但是 Company 是一个template,
不到运行期无法得知是什么, 就无法得知他是否有个函数为 f()

解决办法：
```c++
//三种方法 
//1 ：
void f_der() {  
        this -> f();  
    } 


//2 :
using base<Company>::f;
void f_der() {  
        f();  
    } 

//3 :
void f_der() {  
        base<Company>::f();  
    } 
```

## 7.3 避免代码膨胀

条款44

template 不应该与参数造成关系
```c++
templatr<typename T,size_t n>
	···

```
如果类型一致，但是参数n不一致，会生成多份相同的代码造成膨胀
应该抽离template
或者私有继承于公共父类

避免类型参数的代码膨胀，如int long 可能位数一致 
指针版本调用无类型指针 ：T* -> void* 由后者完成

## 7.4 成员函数模版接受兼容类型
条款45
```c++
template<typename T>  
class A{  
public:  
    explicit A(T* a):m(a){}  
    template<typename U>  
    A(const A<U>& a):m(a.get()) {}  
    [[nodiscard]] T* get() const { return m;}  
    void print(){cout<<*m;}  
private:  
    T* m;  
};
```
此处只有存在某个隐式转换可以将一个 U* 转化为T* 的指针才可以通过编译
<font color = 'red'> 注：此处泛化构造函数前不加explicit 是因为原始指针类型的转化是合理的</font>

## 7.5 
如：
```c++
template<typename T>
const A<T> operator * (const A<T> & l,const A<T> & r)
{···}

A<int> a(1,2);
//此处无法编译
A<int> res = a * 2; 
```
在类型推导时,operator 第一个参数为A<T> 类型，而对象a也是A<T> 类型，所以T推导为int ，
但是operator 第二个参数为A<T> 类型，2为int类型，
由于template 在实参推导时，不将隐式类型转换纳入考虑，不能通过类A中的非explicit构造函数进行转换，所以无法通过编译。
于是：可以将operator 声明为类A的友元函数，

```c++
friend 
const A operator*(const A& l,const A& r)
{
    return ···;
}
```

当对象a被声明后，friend 函数operator * 也被声明出来了，在调用的时候对于int 2 ，可通过隐式转换为A\<int>



或者使用help template

在operator 调用 help template



