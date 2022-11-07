## 1.explicit
C++中的explicit关键字只能用于修饰只有 <font color = 'yellow'>一个参数</font> 的类构造函数, 它的作用是表明该构造函数是<font color = 'yellow'>显示</font>的, 而非隐式的, 跟它相对应的另一个关键字是implicit, 意思是隐藏的,类构造函数默认情况下即声明为implicit(隐式).

```c++
template<typename T>  
class B{  
public:  
    B(int a):m(a){}  
    void print() {cout<<m ;}  
private:  
    int m;  
};
```
此处构造函数只有一个参数 a  但是未声明为explicit 当执行
```c++
B<int> b = 10;
```
时， 会默认转换为
```c++
B<int> b(10);
```
防止类构造函数的隐式自动转换
当有多参数的时候，explicit就失效了, 例外, 就是当除了第一个参数以外的其他参数都有默认值的时候, explicit关键字依然有效


## lazy evaluation

```c++
struct LazyEvaluator{  
    LazyEvaluator() = delete;  
    LazyEvaluator(const int& a, const int& b) : a_(a), b_(b) {}  
  
    operator int() const {  
        if (res)  
            return *res;  
        cout << endl << "evaluation here" << endl;  
        res = make_unique<int>(a_ + b_);  
        return *res;  
    }  
  
    const int& a_;  
    const int& b_;  
  
    mutable unique_ptr<int> res;  
};  
  
void calc(LazyEvaluator& eval, int i) {  
    if (random() % 2 == 0) {  
        cout << "skip " << i << endl;  
    } else {  
        cout << "calculate " << i << " result: "<< int(eval) << endl;  
    }  
}  
  
int main()  
{  
    int x = 2;  
    int y = 3;  
    srand (time(nullptr));  
    LazyEvaluator eval (x, y);  
    for (int i = 0; i < 5; ++i) {  
        calc(eval, i);  
    }  
    return 0;
```

```result
calculate 0 result: 
evaluation here
5
skip 1
calculate 2 result: 5
calculate 3 result: 5
calculate 4 result: 5
```

## 静态成员函数
静态成员函数没有 this 指针，不知道指向哪个对象，无法访问对象的成员变量，也就是说静态成员函数不能访问普通成员变量，只能访问静态成员变量。
类外使用要用访问符


## const 成员函数


const 对象又可以调用非const 成员函数
如果 const 对象要删除 ：
```c++
void destory() const {
	delete this;
}
```


const  ->  小权限 不能调用 非const -> 大权限 的资源

## 智能指针shared_ptr
```c++
int main() {  
    int *t = new int (1);  
    shared_ptr<int> q(t);  
    {  
        shared_ptr<int> p;  
        p.reset(q.get());  
        cout << *p<<" "<<p.get()<<" "<<p.use_count()<<endl;  
        cout << *q<<" "<<q.get()<<" "<<q.use_count()<<endl;  
    }  
    cout << *q<<" "<<q.get()<<" "<<q.use_count()<<endl;  
}
```
```out
1 0x800048440 1
1 0x800048440 1
-2144192248 0x800048440 1
```

指针p reset成指针q所指的对象，虽然指向的对象一致，但是use_count() 都为1，两个指针是完全不一样的对象。
当作用域结束的时候，指针p析构，则指针 q所指向的内存是未定义的，如果指针q再一次析构，则相应的内存会再次被delete 一次