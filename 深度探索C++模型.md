## 1.默认构造函数

以下四种情况，编译器会为没有显式定义构造函数的类生成默认构造函数

- 成员变量中含有其他类对象成员
- 基类有构造函数
// 若以上有定义构造函数，但是未对基类/其他类对象初始化，编译器会“扩张构造函数，调用他们的构造函数。
- 类内有声明/继承到虚函数
- 虚继承

默认构造函数并不会初始化所有成员

在默认构造函数中，按照默认构造函数的行为还可划分为
平凡的（trivial）：没有任何行为（不进行任何动作）
和
非平凡的（nontrivial）：调用其基类的默认构造函数等。

当一个隐式默认构造函数满足下列所有条件，则它是平凡的，反之则为非平凡的。

（1）没有虚成员函数

（2）没有虚基类

（3）没有拥有默认初始化器的非静态数据成员（C++11）

（4）类的所有直接基类都拥有平凡默认构造函数

（5）每个类类型的非静态数据成员都拥有平凡默认构造函数

总的来说，默认构造函数会在需要的时候被编译器生成，但这个需要是指编译器需要而不是用户需要。

```cpp
class Test {  
public:  
    int m;  
    int *p;  
}; 

int main()  
{  
    Test t;  
    cout<<t.m;  
    return 0;  
  
}
```


## 2.拷贝构造函数

`Bitwise Copy Semantics`（位逐次拷贝）
BCS
- 类内含有其他类对象，且该类对象有copy constructor
- 继承自base class ，且···
- 类内有虚函数
- 虚继承

当出现以上4种情况时，不能使用BCS
NRV优化：（Named Return Value）

```c++
X bar() {
	X xx;
	···
	return xx;
}
```
会优化成
```c++
void
bar (X& _res) {
	_res.X::X();
	···
	return ;
}
```
不经过拷贝构造函数获得


如：
这种无虚函数，继承的类，其中拷贝构造函数使用`memcpy`是允许的，
```c++
class Test{  
public:  
    Test(int _m,int  _n) : m(_m),n(_n) { }  
    Test(const Test& rhs) {  
        memcpy(this,&rhs,sizeof(Test) );  
        cout<<"aa";  
    }  
    int m;  
    int n;  
  
};
```
但是若有虚函数等。会有vptr  若将构造函数定义为：
```c++
class Test {
	Test() {memset(this,0,sizeof(Test));}
};
```
则会将vptr设置为0，引发错误

## 3.member initialization list

list 中的项目顺序是按声明顺序决定的。
若：
```c++
class X {
	int i;
	int j;
public:
	X(int val) : j (val),i(j) { }
}
```
会发生错误，因为i先声明，所以先初始化，但是以j初始化，j却是未定义的

在初始化列表中调用成员函数初始化是可以的，但是不能用此方法对于基类进行初始化。

## 4.空class
占用空间为1byte，被编译器安插进去的char

```c++
class X{}; // 1
class Y: public virtual X{}; //1 + vptr(4) + ···
class Z: public virtual X{};
class A: public Y,public Z {}; // 1 + 4 + 4 + ···
```
alignment (边界调整)
一般大小调整至4的倍数  8 ·· 12

单继承关系中，无虚函数的派生类应注意边界调整，并不是会填充基类空虚的内存，而是在原本边界调整后的边界再次加上派生类的内存空间


## 5.类绑定问题
```c++
typedef int length;  
class Test{  
public:  
    Test(int _m,int  _n) : m(_m),n(_n) { }  
    void f() {  
        length k = 1;  // float
    }  
    int m;  
    int n;  
    typedef float length;  
    length val{};  // float
};
```

## 6.static member
- 每一个static data menmber 只有一个实例，存放在data segment中，每一次的取值，都转化为对该extern实例的直接操作

- 存取static member 并不通过需要class object

- 若取一个static data menmber的地址，会得到一个指向其数据类型的指针例如
`&Test::sm` -> `const T*`

- 存放在data segment中,若多个class 声明了相同名称的static member -> 冲突
 -> name-mangling
![[Pasted image 20221119143334.png]]

## 7.内存空间
- 多重继承
对于一个多重派生对象，将其地址指定给第一个base class的指针，情况和单一继承一致，因为二者都有相同的内存起始地址，第二个且后面的base class 需要在原地址上加上sizeof()
偏移

- 虚表指针占用
- ![[Pasted image 20221122205919.png]]
64位8个字节


## 8.虚表空间配置
在  __单一继承__ 情况下
- 子类会继承父类所有的虚函数实例，这些虚函数实例的地址会被拷贝到子类虚表相对应的位置

- __对应的位置__：简单来说，父类虚函数f()放在第二个slot中，子类重写或不重写，子类虚表中f()也是在第二个slot

- 在子类新增一个父类没有的虚函数时，虚表尺寸大一个slot，新的函数实例地址会放入该slot中

- 这样做的好处是：
	每个虚函数所在的位置是固定的，若执行 `ptr->f()`，虽然不知道该执行哪个对象的f(),但是函数地址在编译期的时候是知道的


在  __多重继承__ 情况下


```cpp
class Base1{  
public:  
    explicit  Base1(float _d) :data_base1(_d){};  
    virtual ~Base1() { cout<<"~BASE1"<<endl; }  
    virtual void sp(){data_base1 ++;};  
    [[nodiscard]] virtual Base1* clone() const {return new Base1(data_base1);}  
    virtual void print() const{cout<<"BASE1"<<endl;}  
protected:  
    float data_base1{};  
};  
class Base2{  
public:  
    explicit  Base2(float _d) :data_base2(_d){};  
    virtual ~Base2() {cout<<"~BASE2"<<endl;}  
    virtual void mut(){data_base2 --;};  
    [[nodiscard]] virtual Base2* clone() const {return new Base2(data_base2);}  
    virtual void print() const{cout<<"BASE2"<<endl;}  
protected:  
    float data_base2{};  
};  
class Dri:public Base1,public Base2 {  
public:  
    explicit Dri(float  _m):Base1(_m), Base2(_m),data_dri(_m){};  
    ~Dri() override {cout<<"~Dri"<<endl;}  
    [[nodiscard]] Dri* clone() const override{return new Dri(data_dri);}  
    void print() const override{cout<<"Dri"<<endl;}  
protected:  
    float data_dri;  
};  
int main() {  
    Base2 *p = new Dri(3);  
    p->print();  
	Base2 *q = p->clone();
    return 0;  
}
```

Dri 多重继承public Base1,public Base2
含有2个虚表。
Base2 \*q = p->clone();

p指针首先应该指向Dri的地址，执行Dri的clone()函数，然后传回一个指针，指向新的Dri对象，该对象赋值给q函数的之前，应该调整指向Base2 地址

## 9.虚继承的构造函数方式

A -> BASE

B,C -> virtual public A

D -> public B,C

E -> public D

- 当B,C为D的subobject 时，他们对A的构造操作不能实现，取而代之，由D进行实现初始化


构造函数的  底层编译器的扩容操作中：在构造函数中有多一个  参数 __most_derived

若不为false  则调用基类的构造函数


在  ··当B,C为D的subobject 时，··  D调用 B,C 的构造函数时，将该参数设置为false，从而限制对基类A的构造调用。

- `this->B::B(false,···);`
- `this->C::C(false,···);`

以上，可总结为：若当一个完整的class 对象被定义出来后，他才会被调用，如果对象只是部分，就不会被调用。


- 自上而下的构造函数调用的时候，若在每一次构造函数体内都调用一个子类基类都有的同名函数，则调用的函数是来自当前正在构造的类。若该函数又调用虚函数，一致，不会发生多态，全部都是调用静态函数。
- 原因是 在构造完成前，vptr未被初始化，无法发生多态

- 在base class 构造操作结束后，vptr 保证能在成员初始化列被扩展之前被初始化。
		-> 在成员初始化列表中调用虚函数是可发生多态的

## 4种类型转换

const_cast
```c++
const int a = 10;  
int const *p = &a;  
int *b = const_cast<int*>(p);  
*b = 2;  
cout<<*b<<" "<<*p<<" "<<a;

//2 2 10
```

只能修改指针或者引用的const属性，变量是无法修改的

static_cast
基本等价于隐式转换的一种[类型转换](https://so.csdn.net/so/search?q=%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2&spm=1001.2101.3001.7020)运算符，以前是编译器自动隐式转换，static_cast可使用于需要明确隐式转换的地方。c++中用static_cast用来表示明确的转换。


**dynamic_cast**

dynamic_cast主要用于类层次间的上行转换和下行转换，还可以用于类之间的交叉转换。
在类层次间进行上行转换时，dynamic_cast和static_cast的效果是一样的；在进行下行转换时，dynamic_cast具有类型检查的功能，比static_cast更安全。

C++中层次类型转换中两种：上行转换和下行转换。

对于上行转换，static_cast和dynamic_cast效果一样，都安全；
对于下行转换：你必须确定要转换的数据确实是目标类型的数据，即需要注意要转换的父类类型指针是否真的指向子类对象，如果是，static_cast和dynamic_cast都能成功；如果不是static_cast能返回，但是不安全，可能会出现访问越界错误，而dynamic_cast在运行时类型检查过程中，判定该过程不能转换，返回NULL。














