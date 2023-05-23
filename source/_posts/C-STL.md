---
title: C++STL
date: 2023-05-23 17:03:11
tags: C++
---

# C++11 语言特性
## 1.1 nullptr和std::nullptr_t
C++11允许你使用nullptr取代0或者NULL，用来表示一个pointer（指针）指向no value。例如：
```C++
void f(int);
void f(void*);
f(0);           // call f(int)
f(NULL);        // call f(int) if NULL is 0 ambigous otherwise
f(nullptr);     // call f(void*)
```
nullptr会被自动转换成各种pointer类型，但不会被转换成任何整数类型

## 1.2 以auto完成类型自动推导
### auto类型推导
```C++
auto x = 5;                 // 正确，x是int类型
auto pi = new auto(1);      // 正确，p是int*
const auto* v = &x, u = 6;  // 正确，v是const int*类型，u是const int
static auto y = 0.0;        // 正确，y是double类型
auto int r;                 // 错误，auto不在表示存储类型的指示符
auto s;                     // 错误，auto无法推导出s的类型（必须马上初始化）
```
auto并不能代表一个实际的类型声明（上面s编译错误），只是一个声明类型的“占位符”。使用auto声明的变量必须马上初始化，让编译器推断出它的类型，并且在编译时将auto占位符替换成真正的类型。

### auto推导规则
```C++
int x = 0;
auto *a = &x;           // a -> int*，a被推导为int*
auto b = &x;            // b -> int，b被推导为int，忽略了引用
auto &c = x;            // c -> int&，c被推导为int&
auto d = c;             // d -> int，d被推导为int，忽略了引用
const auto e = x;       // e -> const int
auto f = e;             // f -> int
const auto& g = x;      // g -> const int&
auto& h = g;            // h -> const int&
```
在不声明为引用或者指针时，auto会忽略等号右边的引用类型和const限定
在声明为引用或者指针时，auto会保留等号右边的引用和const属性

### auto的限制
1. auto的使用必须马上初始化，否则无法推导出类型
2. auto在一行定义多个变量时，各个变量不能产生二义性，否则编译失败
3. auto不能用作函数的参数
4. 在类中auto不能用作非静态成员变量
5. auto不能定义数组，可以定义指针
6. auto无法推断出模板参数

## 1.3 decltype用于推导表达式类型
decltype用于推导表达式类型，这里只用于编译器分析表达式的类型，表达式实际不会进行运算
```C++
int func() { return 0; }
decltype(func()) i;     // i为int类型
int x = 0;
decltype(x) y;          // y为int类型
decltype(x + y) z;      // z为int类型
```
注意：decltype不会像auto一样忽略引用和const属性，decltype会保留表达式引用和cv属性

### decltype推导规则
1. exp是表达式，decltype(exp)和exp类型相同
2. exp是函数调用，decltype(exp)和函数返回值类型相同
3. 其他情况，若exp是左值，decltype(exp)是exp的左值引用

### auto和decltype配合使用
```C++
template<typename T, typename U>
return_value add(T t, U u) {
    // t和u类型不确定，无法推导出return_value类型
    return t + u
}
```
改进为
```C++
template<typename T, typename U>
auto add(T t, U u) -> decltype(t + u) {
    return t + u
}
```

## 1.4 一致性初始化与初值列
### 一致性初始化
面对任何初始化动作，可以用大括号进行初始化
```C++
int values[] {1, 2 ,3};
std::vector<int> v {2, 3, 5, 7, 11, 13, 17};
std::vector<std::string> cities {"beijing", "location", "cario"};
```

### 初值列
即使某个local变量属于某种基础类型，也会被初始化为0或者nullptr（如果是个指针）
```C++
int i;      // i has undefined value
int j{};    // j is initalized by 0
int *p;     // p has undefined value
int *q{};   // j is initalized by nullptr
```
注意：窄化--精度降低或造成数值变动，对大括号而言是不成立的
```C++
int x1(5.3);                            // ok, x1 -> 5
int x2 = 5.3;                           // ok, x2 -> 5
int x3{5.0};                            // error
int x4 = {5.0};                         // error
char c1{7};                             // ok
char c2{99999};                         // error
std::vector<int> {1, 2, 3, 4, 5};       // ok
std::vector<int> {1, 2, 3, 4, 5};       // error
```

### std::initalizer_list<>
`initalizer_list<>`用来支持（a list of value）进行初始化 \

实例1：
```C++
void print(std::initalizer_list<int> vals) {
    for (auto p = vals.begin(); p != vals.end(); ++p) {
        std::cout << *p << ' ';
    }
}
print({12, 3, 4, 5, 6, 7, 8});
```

实例2：
```C++
class P {
    public:
    P(int, int);
    P(std::initalizer_list<int>);
};
P p{77, 5};         // Calls P::P(int, int)
P q{77, 5};         // Calls P::P(initalizer_list)
P r{77, 5, 42};     // Calls P::P(initalizer_list)
P s = {77, 5, 42};  // Calls P::P(initalizer_list)
```

实例3：\
explicit构造函数如果接受的是一个初值列，会失去初值列带有0个，1个，多个初值的能力
```C++
class P {
    public:
    P(int, int);
    explicit P(std::initalizer_list<int>);
};
P p{77, 5};         // ok
P q{77, 5};         // ok
P r{77, 5, 42};     // ok
P s = {77, 5, 42};  // error
P s = {77, 5}       // ok
```

## 1.5 Range-Based for循环（foreach循环）
语法特性：
```C++
for (decl : coll) { // decl是给定值coll集合中每一个元素的声明
    statement
}

for (int i : {2, 3, 5, 7, 9, 13, 17, 19}) {
    std::cout << i << std::endl;
}

std::vector<double> vec;
for (auto& elem : vec) {
    elem *= 3;
}

// 打印某集合内所有元素的泛型函数实例
template<typename T>
void printElements(const T& coll) {
    for (const auto& elem : coll) {
        std::cout << elem << std::endl;
    }
}
```

可以针对初值列使用range-based for循环，因为class template std::initalizer_list<>提供了成员函数begin()和end()
```C++
int array[] = {1, 2, 3, 4, 5};
long sum = 0;
for (int x : array) {
    sum += x;
}
for (auto elem : {sum, sum * 2, sum * 4} {
    std::cout << elem << std::endl;
})
```


## 1.6 move语义和左值引用
###为何需要移动语义
假设有如下代码：
```C++
vector<string> vstr;
// build up a vector of 20,000 strings, each of 1000 characters
...
vector<string> vstr_copy1(vstr); // make vstr_copy1 a copy of vstr
```
vector和string类都使用了动态内存分配，因此它们必须定义某种new版本的复制构造函数，为初始化对象vstr_copy1，复制构造函数vector<string>将使用new给20000个string对象分配内存，而每个string对象又将调用string的复制构造函数，该构造函数使用new为1000个字符分配内存。接下来全部20000*1000个字符都将从vstr控制的内存中复制到vstr_copy1控制的内存中

假设以下面方式使用它：
```C++
vector<string> vstrl
// build up a vector of 20,000 strings, each of 1000 characters
vector<string> vstr_copy1(vstr);
vector<string> vstr_copy2(allcaps(vstr));
```
从表面上看，两个复制一致，它们都使用了一个现有的对象初始化一个vector<string>对象，如果深入探索这些代码，将发现allcaps()创建了对象temp，该对象管理着20000*1000个字符，vector和string的复制构造函数创建这20000*1000个字符的副本，然后程序删除allcaps()返回临时对象。（这里做了很多无用功）

### 使用移动语义
编译器对数据的所有权直接转让给vstr_copy2，不进行新的复制副本，在删除副本；而是将字符留在原来的地方，并将vstr_copy2与之相关联

### 如何实现移动语义
使用右值引用，让编译器知道什么时候需要复制，什么时候不需要

方法：使用移动构造函数，它使用右值引用作为参数，该引用关联到右值实参 \
注意1：复制构造函数可执行深复制，而移动构造函数只调整记录 \
注意2：在将所有权转移给新对象的过程中，移动构造函数可能修改其实参，这意味着右值引用参数不应该是const
```C++
#include <iostream>
using namespace std;

class Useless {
    private:
    int n;
    char *pc;
    static int ct;
    void ShowObject() const;
    
    public:
    Useless();
    explicit Useless(int k);
    Useless(int k, char ch);
    Useless(const Useless &f);
    Useless(Useless &&f);
    ~Useless();
    Useless operator+(const Useless &f) const;
    void ShowData() const;
}

int Useless::ct = 0;

Useless::Useless {
    ++ct;
    n = 0;
    pc = nullptr;
    ShowObject();
}

Useless::Useless(int k) : n(k) {
    ++ct;
    pc = new char[n];
    ShowObject();
}

Useless::Useless(const Useless& f) : n(f.n) {
    ++ct;
    pc = new char[n];
    for (int i = 0; i < n; i++) {
        pc[i] = ch;
    }
    ShowObject();
}

Useless::Useless(Useless&& f) : n(f.n) {
    ++ct;
    pc = f.pc;
    f.pc = nullptr;
    for (int i = 0; i < n; i++) {
        pc[i] =  ch;
    }
    ShowObject();
}

Useless::~Useless() {
    cout << "destructor called; objects left: " << --ct << endl;
    cout << "deleted object:\n";
    ShowObject();
    delete []pc;
}

Useless Useless::operator+(const Useless& f) const {
    cout << "Entering operator+()\n";
    Useless temp = Useless(n + f.n);
    for (int i = 0; i < n; i++) {
        pc[i] =  ch;
    }
    for (int i = 0; i < temp; i++) {
        temp.pc[i] = f.pc[i - n];
    }
    cout << "temp object:\n";
    cout << "Leaving operator+()\n";
    return temp;
}
```
两种构造函数比较

复制构造函数：
```C++
Useless::Useless(const Useless& f) : n(f.n) {
    ++ct;
    pc = new char[n];
    for (int i = 0; i < n; i++) {
        pc[i] = ch;
    }
}
```
移动构造函数：
```C++
Useless::Useless(const Useless&& f) : n(f.n) {
    ++ct;
    pc = f.pc;
    f.pc = nullptr;
    f.n = 0;
}
```

它让pc指向现有的数据，以获取这些数据的所有权，此时pc和f.pc指向相同的数据，调用析构函数时将带来麻烦，因为程序不能对同一个地址调用delete两次，为避免这个问题该构造函数将原来的指针设置为空指针（对空指针执行delete[]没有问题）

由于修改了f对象，这要求不能在参数声明中使用const