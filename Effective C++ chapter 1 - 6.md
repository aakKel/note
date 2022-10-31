## 1.尽量不使用#define

\#define 不注重作用域，在class中常量应该 static const T


enum hack:
属于枚举类型的数值，like define
取一个enum地址是非法的->无法用指针指向

## 2.尽量使用const
成员函数的常量性不同，可以被重载
const 成员函数可以想要修改类内变量时，可以把变量声明为mutable
<font color = "red">如何让重载函数中的非const 调用const 版本</font>

```c++
class Text
{
public:
    Text(const std::string s):text(s){}
    const char & operator[](size_t position) const
    {
        return text[position];
    }
    char &operator[](size_t position)
    {
        return text[position];
    }

private:
    string text;
};
```

首先，应当将 *\*this* 从原始的Text & 转为 const Text&

第二次是从const operator[] 的返回值移除const

```c++
char &operator[](size_t position)
{
    return 
        const_cast<char&>(
            static_cast<const Text&>(*this)
            [position]
        );
}
```

 不要在const版本的函数进行类型转换调用非const版本的函数

## 3.不要在构造和析构函数中调用虚函数

派生类对象在构造的时候，在顺序是基->派生, 当派生类的构造函数开始执行前，对象是一个基类对象。

## 4.operator 返回一个 reference *this

## 5.在operator = 中处理自我赋值
```c++
if (this == &rhs) return *this;
```
