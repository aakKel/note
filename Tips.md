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


## 2.lazy evaluation

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

## 3.静态成员函数
静态成员函数没有 this 指针，不知道指向哪个对象，无法访问对象的成员变量，也就是说静态成员函数不能访问普通成员变量，只能访问静态成员变量。
类外使用要用访问符


## 4.const 成员函数


const 对象又可以调用非const 成员函数
如果 const 对象要删除 ：
```c++
void destory() const {
	delete this;
}
```


const  ->  小权限 不能调用 非const -> 大权限 的资源

## 5.智能指针shared_ptr
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


## 6.在unordered_map<> 中使用pair<>作为key
```cpp
struct pair_hash  
{  
    template<class T1, class T2>  
    std::size_t operator() (const std::pair<T1, T2>& p) const  
    {  
        auto h1 = std::hash<T1>{}(p.first);  
        auto h2 = std::hash<T2>{}(p.second);  
        return h1 ^ h2;  
    }  
};  
  
int main() {  
    unordered_map<pair<int, int>, int, pair_hash> table;  
    table.emplace(make_pair(1, 1), 1);  
    cout << table[make_pair(1, 1)];  
    return 0;  
}
```
https://zh.cppreference.com/w/cpp/utility/hash
![[Pasted image 20221113151904.png]]


## 7.deque
deque的排序是将空间复制到vector 然后调用STL sort 最后复制回去

空间不连续

有一个map类型的中控器，其类型是一个二级指针，二级指针所指向的一级指针，其指向的是一段连续的内存空间（缓冲区：存放元素）。若存储map的空间满，则找一块更大的空间复制过去
map默认大小为512bytes

push_back / push_front :
如果缓冲区空闲位置个数大于=2个，则直接插入，若只剩一个空间。缓冲区需要重新配置，也导致了迭代器的重新配置，重新配置->开辟一个新的缓冲区，cur 指向新缓冲区最前端

pop同理，若缓冲区只剩下一个元素，则释放缓冲区

erase 清除完元素点/区间后会移动元素，将剩余元素移动至缓冲区中间

insert同理


## 8.stack queue priority_queue都无迭代器
底层都可用vector作为容器  只作为一个配接器存在
## 9.make_heap

```cpp
vector<int> v = {1,34,2,3,4,42,49};  
std::make_heap(v.begin(), v.end(),greater<>());  
for (int i : v) {  
    cout<<i<<" ";  
}
```
```
1 3 2 34 4 42 49
```

push_heap / pop_heap同样三个参数，第三个参数应该与make_heap 一致
push_heap 前，应该在原始容器里插入数据后再调用。
而且 pop_heap 不会把原始容器里的元素删除，只是移到最后一个位置
原始容器不应该使用array  不可增长

## 10.set

为什么不可以通过迭代器修改set的值？
a：
set的元素值就是key值，且set\<T>::iterator 是RB-TREE 的 const_iterator 
![[Pasted image 20221115141946.png]]
Cannot assign to return value because function 'operator*' returns a const value
![[Pasted image 20221114185527.png]]

底层由RB-TREE实现，接口也是调用RB-TREE
插入调用的是insert_unique()
![[Pasted image 20221114192113.png]]
find -> 应该调用自带的成员函数而不是STL提供的find

## 11.map
模版：
![[Pasted image 20221115141749.png]]


底层红黑树，也有使用hash table的map -> hash_map

value_type 是一个pair\<const key,value>
![[Pasted image 20221115141707.png]]

与set 不同，迭代器不是使用const_iterator,所以可以更改value，key不可更改，声明为const
![[Pasted image 20221115141906.png]]



接口/成员函数：
可以看到：begin,end 等 都是调用RB-TREE接口
![[Pasted image 20221115142128.png]]
插入同样使用的insert_unique();
![[Pasted image 20221115142419.png]]

insert_or_assign()
三个参数，迭代器，key ，value
不存在该key 插入，存在 替换value
![[Pasted image 20221115142648.png]]
