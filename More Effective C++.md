## 1 . 限制类对象个数

```c++
template<typename T>  
class Count {  
public:  
    Count() {  
        init();  
    }  
    Count(const Count & rhs) {  
       init();  
    }  
    static int object_count() { return cur_num;}  
    virtual ~Count(){  
        --cur_num;  
    }  
    void print() {  
        cout<<"cur_num"<<" "<<cur_num<<" ";  
        cout<<"max_num"<<" "<<max_num<<endl;  
    }  
private:  
    static const int max_num;  
    static int cur_num;  
    void init() {  
        if (cur_num > max_num) {  
            cout << "out_of_range" << endl;  
        }  
        ++cur_num;  
    }  
};  
  //私有继承，使用using 使用成员函数
class Pub : private Count<Pub>{  
public:  
  
    using Count<Pub>::print;  
    using Count<Pub>::object_count;  
    static Pub *makePub() {  
        return new Pub;  
    }  
    static Pub *makePub(const Pub &rhs) {  
        return new Pub(rhs);  
    }  
    ~Pub() override = default;  
private:  
    Pub();  
    Pub(const Pub &rhs);  
};  
//偏特化
template<> const int Count<Pub>::max_num = 2;  
template<> int Count<Pub>::cur_num = 0;  
int main() {  
    for (int i = 0; i < 4; ++i) {  
        Pub *p = Pub::makePub();  
        p -> print();  
        Pub *q = Pub::makePub(*p);  
        q -> print();  
        cout<<endl;  
    }  
    return 0;  
}
```
```out
cur_num 1 max_num 2
cur_num 2 max_num 2

cur_num 3 max_num 2
out_of_range
cur_num 4 max_num 2

out_of_range
cur_num 5 max_num 2
out_of_range
cur_num 6 max_num 2

out_of_range
cur_num 7 max_num 2
out_of_range
cur_num 8 max_num 2
```
