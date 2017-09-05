---
title: C++ STL入门
date: 2017-09-04 13:35:00
categories:
- cpp
tags:
- cpp
keywords: C++ STL入门
---

> 
C++ STL入门，`标准模板库（Standard Template Library）`，STL可分为`容器(containers)`、`迭代器(iterators)`、`算法(algorithms)`、`适配器(adapters)`、`空间配置器(allocator)`、`仿函数(functors)`六个部分

<!-- more -->

## 前言
STL 提供了一些常用的数据结构和算法的模板，1998 年加入 C++ 标准；STL 中有三个重要的概念：
- `容器`：容纳各种数据类型的数据结构，是一系列的类模板；
- `迭代器`：迭代器用来迭代地访问容器中的元素；
- `算法`：用来操作容器中的元素，是一系列的函数模板；


> 
`容器`："数据结构"的泛指、`迭代器`："指针"的泛指、`算法`："算法"的泛指；


**STL 容器**
STL 中的容器定义在`std`命名空间下，需要引入头文件`<vector>`、`<list>`、`<deque>`、`<stack>`、`<queue>`、`<set>`、`<map>`等；容器可以分为三大类：
- **顺序容器**
 - `vector`："动态数组"，随机存取性能高，尾端插入删除方便，内部插入删除效率低；
 - `list`："双向链表"，插入删除方便，不支持随机存取，只能从头开始遍历；
 - `deque`："双端队列"，首尾插入删除都有较高性能，同时支持随机存取，占用内存较多；
- **关联容器**
 - `set`："红黑树"，无重复元素的集合；
 - `multiset`："红黑树"，元素的集合，元素可重复；
 - `map`："红黑树"，键值对的集合，键不可重复；
 - `multimap`："红黑树"，键值对的集合，键可重复；
- **容器适配器**
 - `stack`：栈，后进先出（LIFO）的数据结构；
 - `queue`：队列，先进先出（FIFO）的数据结构；
 - `priority_queue`；优先队列，队列的一种，可以自定义数据的优先级，进行动态排序；


这些容器有一些通用的方法：`empty`、`size`、`swap`、`max_size`；
顺序容器和关联容器支持迭代器，称为`第一类容器`；
顺序容器还有以下通用方法：`front`、`back`、`pop_back`、`push_back`；

存储键值对关联容器`map`和`multimap`的迭代器是一个`pair<T1, T2>`的指针；
插入时，可以使用`[]`运算符，也可以使用`insert`方法，它接受一个`pair<T1, T2>`对象；

容器适配器是逻辑数据结构，需要用一种顺序容器来实现；例如，stack 默认使用 deque 来实现，我们也可以指定它的实现方式；

> 
容器之间的比较取决于第一个不等的元素；如果长度相同且所有元素相等，两个容器相等；如果一个是另一个的子序列，则较短的容器小于较长的容器；


**STL 迭代器**
头文件：`<iterator>`
> 
只有第一类容器支持迭代器（容器适配器不支持迭代器）；

迭代器按功能的强弱分为 5 类：
- `Input Iterator`：提供只读访问
- `Output Iterator`：提供只写访问
- `Forward Iterator`：支持逐个向后迭代访问
- `Bidirectional Iterator`：能够双向地逐个迭代访问
- `Random Access Iterator`：可随机访问每个元素


`vector`和`deque`支持`Random Access Iterator`，`list`、`set/multiset`、`map/multimap`支持`Bidirectional Iterator`；


**STL 算法**
STL 通过函数模板提供了很多作用于容器的通用算法，例如`查找`、`插入`、`删除`、`排序`等，需要引入头文件`<algorithm>`；

变化序列的：`copy`、`remove`、`reverse`、`fill`、`replace`、`swap`等；
不变化序列的：`find`、`count`、`for_each`、`equal`等；

> 
这些算法的实现较为通用，也可以作用于 C 语言的数组；


## 顺序容器
### vector
**vector 和 built-in 数组类似，是一个在堆上建立的一维数组，它拥有一段连续的内存空间，并且起始地址不变**
- vector 有非常好的随机存取性能，支持`[]`操作符；
- vector 不同于静态数组，系统会自动按照一定的比例（一般为1.5 ~ 2倍）进行扩充；
- 在 vector 末尾插入或删除的效率高，在中间插入或删除的效率低；
主要是要进行元素的移动和内存的拷贝，原因就在于当内存不够用的时候要执行重新分配内存，拷贝对象到新存储区，销毁 old 对象，释放内存等操作；
如果对象很多的话，这种操作代价是相当高的；为了减少这种代价，使用 vector 最理想的情况就是事先知道所要装入的对象数目，用成员函数 reserve 预定下来；
- vector 最大的优点莫过于是检索（operator[]）速度在这三个容器中是最快的；


定义于头文件：`<vector>`
原型：`template <typename T, typename Allocator = std::allocator<T>> class vector;`
- `T`：元素的类型；
- `Allocator`：用于获取/释放内存及构造/析构内存中元素的分配器；


vector 上的常见操作的`时间复杂度`如下：
- 随机访问：常数`O(1)`
- 在末尾插入或移除元素：常数`O(1)`
- 插入或移除元素：与到 vector 结尾的距离成线性`O(n)`


`vector<Elem> c`：产生一个空的 vector，其中没有任何元素；
`vector<Elem> c1(c2)`：拷贝构造，将 c2 的所有元素拷贝至 c1；
`vector<Elem> c(n)`：利用元素的默认构造函数生成一个大小为 n 的 vector；
`vector<Elem> c(n, elem)`：生成一个大小为 n 的 vector，并且每个元素的值都为 elem；
`vector<Elem> c(beg, end)`：生成一个 vector，以左闭右开区间`[beg, end)`为元素初值；

`c1 = c2`：支持Copy、Move语义，赋值操作；
`c.assign(n, elem)`：复制 n 个 elem，赋值给 c；
`c.assign(beg, end)`：将区间`[beg, end)`赋值给 c；
`c1.swap(c2)`：交换 c1 和 c2；
`swap(c1, c2)`：同上，全局函数；

`c.at(i)`：返回索引 i 所在的元素，若越界，抛出 out_of_range；
`c[i]`：返回索引 i 所在的元素，不检查索引范围的有效性；
`c.front()`：返回第一个元素；
`c.back()`：返回最后一个元素；

`c.insert(pos, elem)`：在 pos 位置插入一个 elem 副本；
`c.insert(pos, n, elem)`：在 pos 位置插入 n 个 elem 副本；
`c.insert(pos, beg, end)`：在 pos 位置插入区间`[beg, end)`；
`c.push_back(elem)`：在尾部添加一个 elem 副本；
`c.pop_back()`：移除最后一个元素；
`c.erase(pos)`：移除 pos 位置上的元素；
`c.erase(beg, end)`：移除区间`[beg, end)`中的元素；
`c.clear()`：移除所有元素，将容器清空，但是 capacity() 并不改变；

`operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=`：比较操作；

`c.size()`：返回当前元素的数量；
`c.capacity()`：返回当前 vector 的容量；
`c.empty()`：判断当前 vector 是否为空；
`c.max_size()`：返回 vector 可容纳的元素最大数量；

`c.reserve(n)`：调整预留的容量为 n 个元素长度，如果 n 大于当前 capacity()，那么将重新分配一块大小为 n 个元素长度的内存，拷贝 old 中的元素，并释放 old 容器；如果 n 小于等于当前 capacity()，则不做任何操作；
`c.resize(n)`：调整元素数量，如果 n 小于当前 size()，那么 n 后面的元素被丢弃；如果 n 大于当前 size()，新增的元素由元素的默认构造函数完成；
`c.resize(n, elem)`：同上，多出的元素都是 elem 的副本；
`c.shrink_to_fit()`：回收未使用的容量，非强制性请求（C++11 起）；

`c.begin()`：返回一个迭代器，指向第一个元素；
`c.end()`：返回一个迭代器，指向最后一个元素的下一位置；
`c.rbegin()`：返回一个逆向迭代器，指向第一个元素；
`c.rend()`：返回一个逆向迭代器，指向最后一个元素的下一位置；
`c.cbegin()`：const 修饰的 begin；
`c.cend()`：const 修饰的 end；
`c.crbegin()`：const 修饰的 rbegin；
`c.crend()`：const 修饰的 rend；

简单例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>
using namespace std;

typedef vector<int> vec;

int main() {
    vec v{5, 2, 1, 3, 9, 4, 8, 7, 6, 10};

    // 访问元素，方式1
    for (vec::iterator it = v.begin(); it != v.end(); it++) {
        cout << *it << ", ";
    }
    cout << "\b\b " << endl;

    // 访问元素，方式2，C++11
    for (auto it = v.begin(); it != v.end(); it++) {
        cout << *it << ", ";
    }
    cout << "\b\b " << endl;

    // 访问元素，方式3，for_each，头文件<algorithm>，lambda表达式，C++11
    for_each(v.begin(), v.end(), [](int &i){cout << i << ", ";});
    cout << "\b\b " << endl;

    // 访问元素，方式4，for_each，头文件<algorithm>，泛型lambda，C++14
    for_each(v.begin(), v.end(), [](auto &i){cout << i << ", ";});
    cout << "\b\b " << endl;

    // 访问元素，方式5，区间for，C++11
    for (auto &i : v) {
        cout << i << ", ";
    }
    cout << "\b\b " << endl;

    cout << "-----------------------------" << endl;

    // 通用print函数
    auto print = [] (auto &v) {
        if (v.empty()) {
            cout << "container is empty" << endl;
        } else {
            for (auto &i : v) {
                cout << i << ", ";
            }
            cout << "\b\b " << endl;
        }
    };

    // 升序(默认)
    sort(v.begin(), v.end());
    print(v);

    // 降序(通用函数对象，greater，位于头文件<functional>)
    sort(v.begin(), v.end(), greater<int>());
    print(v);

    // 升序(通用函数对象，less，位于头文件<functional>)
    sort(v.begin(), v.end(), less<int>());
    print(v);

    // 降序(lambda表达式，C++11)
    sort(v.begin(), v.end(), [](int &lhs, int &rhs){return lhs > rhs;});
    print(v);

    // 升序(lambda表达式，C++11)
    sort(v.begin(), v.end(), [](int &lhs, int &rhs){return lhs < rhs;});
    print(v);

    // 降序(泛型lambda，C++14)
    sort(v.begin(), v.end(), [](auto &lhs, auto &rhs){return lhs > rhs;});
    print(v);

    // 升序(泛型lambda，C++14)
    sort(v.begin(), v.end(), [](auto &lhs, auto &rhs){return lhs < rhs;});
    print(v);

    cout << "-----------------------------" << endl;

    // 通过C数组进行初始化、赋值
    int arr[5] = {1, 2, 3, 4, 5};
    vec v1(arr, arr+sizeof(arr)/sizeof(int)), v2;
    print(v1);

    v2.assign(arr, arr+sizeof(arr)/sizeof(int));
    print(v2);

    cout << "-----------------------------" << endl;

    vec v3;
    cout << "size: " << v3.size() << ", capacity: " << v3.capacity() << endl;
    v3.reserve(50); // 扩容至 50 个长度
    cout << "size: " << v3.size() << ", capacity: " << v3.capacity() << endl;
    v3.resize(10);  // 填充 10 个元素，size = 10
    cout << "size: " << v3.size() << ", capacity: " << v3.capacity() << endl;

    // 回收多余的容量，swap方法，C++11之前
    vec(v3).swap(v3);   // vec(v3)拷贝构造函数，创建一个匿名对象，然后调用vector的swap方法，交换vector
    cout << "size: " << v3.size() << ", capacity: " << v3.capacity() << endl;

    // 回收多余的容量，shrink_to_fit方法，C++11
    v3.reserve(50);
    cout << "size: " << v3.size() << ", capacity: " << v3.capacity() << endl;
    v3.shrink_to_fit();
    cout << "size: " << v3.size() << ", capacity: " << v3.capacity() << endl;

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [16:58:39]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [16:59:00]
$ ./a.out
5, 2, 1, 3, 9, 4, 8, 7, 6, 10
5, 2, 1, 3, 9, 4, 8, 7, 6, 10
5, 2, 1, 3, 9, 4, 8, 7, 6, 10
5, 2, 1, 3, 9, 4, 8, 7, 6, 10
5, 2, 1, 3, 9, 4, 8, 7, 6, 10
-----------------------------
1, 2, 3, 4, 5, 6, 7, 8, 9, 10
10, 9, 8, 7, 6, 5, 4, 3, 2, 1
1, 2, 3, 4, 5, 6, 7, 8, 9, 10
10, 9, 8, 7, 6, 5, 4, 3, 2, 1
1, 2, 3, 4, 5, 6, 7, 8, 9, 10
10, 9, 8, 7, 6, 5, 4, 3, 2, 1
1, 2, 3, 4, 5, 6, 7, 8, 9, 10
-----------------------------
1, 2, 3, 4, 5
1, 2, 3, 4, 5
-----------------------------
size: 0, capacity: 0
size: 0, capacity: 50
size: 10, capacity: 50
size: 10, capacity: 10
size: 10, capacity: 50
size: 10, capacity: 10
</script></code></pre>


### list
**list 的本质是一个双向链表，内存空间不连续，通过指针进行操作**
- list 的高效率首先表现是插入，删除元素，进行排序等等需要移动大量元素的操作；
- list 没有检索操作 operator[]，也就是说不能对链表进行随机访问，而只能从头至尾地遍历，这是它的一个缺陷；
- list 有不同于前两者的某些成员方法，如合并 list 的方法 splice，排序 sort，交换 list 的方法 swap 等等；


定义于头文件：`<list>`
原型：`template <typename T, typename Allocator = std::allocator<T>> class list;`
- `T`：元素的类型；
- `Allocator`：用于获取/释放内存及构造/析构内存中元素的分配器；


`list<Elem> c`：产生一个空的 list，其中没有任何元素；
`list<Elem> c1(c2)`：拷贝构造，将 c2 的所有元素拷贝至 c1；
`list<Elem> c(n)`：利用元素的默认构造函数生成一个大小为 n 的 list；
`list<Elem> c(n, elem)`：生成一个大小为 n 的 list，并且每个元素的值都为 elem；
`list<Elem> c(beg, end)`：生成一个 list，以左闭右开区间`[beg, end)`为元素初值；

`c1 = c2`：支持Copy、Move语义，赋值操作；
`c.assign(n, elem)`：复制 n 个 elem，赋值给 c；
`c.assign(beg, end)`：将区间`[beg, end)`赋值给 c；
`c1.swap(c2)`：交换 c1 和 c2；
`swap(c1, c2)`：同上，全局函数；

`c.insert(pos, elem)`：在 pos 位置插入一个 elem 副本；
`c.insert(pos, n, elem)`：在 pos 位置插入 n 个 elem 副本；
`c.insert(pos, beg, end)`：在 pos 位置插入区间`[beg, end)`；

`c.erase(pos)`：移除 pos 位置上的元素；
`c.erase(beg, end)`：移除区间`[beg, end)`中的元素，（使用`advance(c.begin(), 5)`移动迭代器）；

`c.remove(elem)`：删除链表中所有等于 elem 的元素；
`c.remove_if(comp)`：删除满足条件 comp（即返回 true）的元素；

`c.push_front(elem)`：在首部添加一个 elem 副本；
`c.pop_front()`：移除首部一个元素；
`c.push_back(elem)`：在尾部添加一个 elem 副本；
`c.pop_back()`：移除尾部一个元素；

`c.front()`：返回第一个元素；
`c.back()`：返回最后一个元素；

`c1.merge(c2)`：合并 2 个有序的链表并使之有序，释放 c2；
`c1.merge(c2, comp)`：合并 2 个有序的链表并使之按照自定义规则排序，释放 c2；

`c1.splice(c1.beg, c2)`：将 c2 连接在 c1 的 beg 位置，释放 c2；
`c1.splice(c1.beg, c2, c2.beg)`：将 c2 的 beg 位置的元素连接到 c1 的 beg 位置，并且在 c2 中释放掉 beg 位置的元素；
`c1.splice(c1.beg, c2, c2.beg, c2.end)`：将 c2 的`[beg, end)`位置的元素连接到 c1 的 beg 位置并且释放 c2 的`[beg, end)`位置的元素；

`c.unique()`：删除重复的元素；
`c.reverse()`：反转链表；
`c.sort()`：默认升序排序；
`c.sort(comp)`：自定义排序；

`c.clear()`：移除所有元素，将容器清空；

`operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=`：比较操作；

`c.size()`：返回当前元素的数量；
`c.empty()`：判断当前 list 是否为空；
`c.max_size()`：返回 list 可容纳的元素最大数量；

`c.resize(n)`：调整元素数量，如果 n 小于当前 size()，那么 n 后面的元素被丢弃；如果 n 大于当前 size()，新增的元素由元素的默认构造函数完成；
`c.resize(n, elem)`：同上，多出的元素都是 elem 的副本；

`c.begin()`：返回一个迭代器，指向第一个元素；
`c.end()`：返回一个迭代器，指向最后一个元素的下一位置；
`c.rbegin()`：返回一个逆向迭代器，指向第一个元素；
`c.rend()`：返回一个逆向迭代器，指向最后一个元素的下一位置；
`c.cbegin()`：const 修饰的 begin；
`c.cend()`：const 修饰的 end；
`c.crbegin()`：const 修饰的 rbegin；
`c.crend()`：const 修饰的 rend；

简单例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <list>
#include <algorithm>
#include <functional>
using namespace std;

typedef list<int> li;

int main() {
    auto print = [] (auto &c) {
        if (c.empty()) {
            cout << "container is empty" << endl;
        } else {
            for (auto &i : c) {
                cout << i << ", ";
            }
            cout << "\b\b " << endl;
        }
    };

    li c1{5, 3, 8, 9, 1, 2, 4, 6, 10, 7}, c2{15, 13, 18, 19, 11, 12, 14, 16, 20, 17};
    print(c1); print(c2);

    // 升序排列
    c1.sort(); print(c1);
    c2.sort(); print(c2);

    // merge c2 -> c1，升序
    c1.merge(c2);
    print(c1); print(c2);

    c2.splice(c2.begin(), c1, [&c1]{auto beg = c1.begin(); advance(beg, 10); return beg;}(), c1.end());
    print(c1); print(c2);

    // 降序排列
    c1.sort([](auto &lhs, auto &rhs){return lhs > rhs;}); print(c1);
    c2.sort([](auto &lhs, auto &rhs){return lhs > rhs;}); print(c2);

    // merge c2 -> c1，降序
    c1.merge(c2, [](auto &lhs, auto &rhs){return lhs > rhs;});
    print(c1); print(c2);

    // remove_if
    c1.sort();
    print(c1);
    c1.remove_if([](auto &i){return i > 10;});
    print(c1);

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [18:53:11]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [18:53:27]
$ ./a.out
5, 3, 8, 9, 1, 2, 4, 6, 10, 7
15, 13, 18, 19, 11, 12, 14, 16, 20, 17
1, 2, 3, 4, 5, 6, 7, 8, 9, 10
11, 12, 13, 14, 15, 16, 17, 18, 19, 20
1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20
container is empty
1, 2, 3, 4, 5, 6, 7, 8, 9, 10
11, 12, 13, 14, 15, 16, 17, 18, 19, 20
10, 9, 8, 7, 6, 5, 4, 3, 2, 1
20, 19, 18, 17, 16, 15, 14, 13, 12, 11
20, 19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1
container is empty
1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20
1, 2, 3, 4, 5, 6, 7, 8, 9, 10
</script></code></pre>


### deque
**deque（double-ended queue），即双端队列，是由多个连续内存块构成**
- deque 是 list 和 vector 的兼容，分为多个块，每一个块大小是 512 字节，块通过 map 块管理，map 块里保存每个块得首地址；
- deque 也有索引操作 operator[]，效率没 vector 高；
- deque 比 vector 多了 push_front、pop_front 操作；在两端进行此操作时与 list 的效率差不多；


定义于头文件：`<deque>`
原型：`template <typename T, typename Allocator = std::allocator<T>> class deque;`
- `T`：元素的类型；
- `Allocator`：用于获取/释放内存及构造/析构内存中元素的分配器；


deque 上常见操作的`时间复杂度`如下：
- 随机访问：常数`O(1)`
- 在结尾或开头插入或移除元素：常数`O(1)`
- 插入或移除元素：线性`O(n)`


`deque<Elem> c`：产生一个空的 deque，其中没有任何元素；
`deque<Elem> c1(c2)`：拷贝构造，将 c2 的所有元素拷贝至 c1；
`deque<Elem> c(n)`：利用元素的默认构造函数生成一个大小为 n 的 deque；
`deque<Elem> c(n, elem)`：生成一个大小为 n 的 deque，并且每个元素的值都为 elem；
`deque<Elem> c(beg, end)`：生成一个 deque，以左闭右开区间`[beg, end)`为元素初值；

`c1 = c2`：支持Copy、Move语义，赋值操作；
`c.assign(n, elem)`：复制 n 个 elem，赋值给 c；
`c.assign(beg, end)`：将区间`[beg, end)`赋值给 c；
`c1.swap(c2)`：交换 c1 和 c2；
`swap(c1, c2)`：同上，全局函数；

`c.insert(pos, elem)`：在 pos 位置插入一个 elem 副本；
`c.insert(pos, n, elem)`：在 pos 位置插入 n 个 elem 副本；
`c.insert(pos, beg, end)`：在 pos 位置插入区间`[beg, end)`；

`c.erase(pos)`：移除 pos 位置上的元素；
`c.erase(beg, end)`：移除区间`[beg, end)`中的元素；

`c.push_front(elem)`：在首部添加一个 elem 副本；
`c.pop_front()`：移除首部一个元素；
`c.push_back(elem)`：在尾部添加一个 elem 副本；
`c.pop_back()`：移除尾部一个元素；

`c.at(i)`：返回索引 i 所在的元素，若越界，抛出 out_of_range；
`c[i]`：返回索引 i 所在的元素，不检查索引范围的有效性；
`c.front()`：返回第一个元素；
`c.back()`：返回最后一个元素；

`c.clear()`：移除所有元素，将容器清空；

`operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=`：比较操作；

`c.size()`：返回当前元素的数量；
`c.empty()`：判断当前 deque 是否为空；
`c.max_size()`：返回 deque 可容纳的元素最大数量；
`c.shrink_to_fit()`：通过释放未使用的内存减少内存的使用（C++11 起）；

`c.resize(n)`：调整元素数量，如果 n 小于当前 size()，那么 n 后面的元素被丢弃；如果 n 大于当前 size()，新增的元素由元素的默认构造函数完成；
`c.resize(n, elem)`：同上，多出的元素都是 elem 的副本；

`c.begin()`：返回一个迭代器，指向第一个元素；
`c.end()`：返回一个迭代器，指向最后一个元素的下一位置；
`c.rbegin()`：返回一个逆向迭代器，指向第一个元素；
`c.rend()`：返回一个逆向迭代器，指向最后一个元素的下一位置；
`c.cbegin()`：const 修饰的 begin；
`c.cend()`：const 修饰的 end；
`c.crbegin()`：const 修饰的 rbegin；
`c.crend()`：const 修饰的 rend；

简单例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <deque>
using namespace std;

typedef deque<int> deq;

int main() {
    deq d{1, 2, 3};

    for (auto &i : d) {
        cout << i << ", ";
    }
    cout << "\b\b " << endl;

    d.push_front(0);
    d.push_back(4);
    d.push_back(5);

    for (auto &i : d) {
        cout << i << ", ";
    }
    cout << "\b\b " << endl;

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [19:46:57]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [19:47:08]
$ ./a.out
1, 2, 3
0, 1, 2, 3, 4, 5
</script></code></pre>


### 如何选择
选择顺序容器类型的一些准则：
- 如果需要`随机访问`一个容器，vector 比 list 好
- 如果`经常`需要`插入`或`删除`容器元素，list 比 vector 好
- 如果既要`随机存取`，又要关心`两端`数据的`插入`与`删除`，则选择 deque


## 容器适配器
### stack
定义于头文件：`<stack>`
原型：`template <typename T, typename Container = std::deque<T>> class stack;`
- `T`：元素的类型；
- `Container`：底层容器的类型，默认为`std::deque`；


`stack<Elem> c`：构造一个空的 stack；
`stack<Elem> c(C)`：通过底层容器 C 进行构造；
`stack<Elem> c1(c2)`：通过另一个 stack 进行构造；

`c1 = c2`：支持Copy、Move语义，赋值操作；
`c1.swap(c2)`：交换两个容器适配器；
`swap(c1, c2)`：同上，全局函数；

`c.size()`：返回 stack 的元素数量；
`c.empty()`：判空操作；

`c.push(elem)`：压栈；
`c.pop()`：出栈，不返回弹出元素的值；
`c.top()`：读取栈顶元素的值；

`operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=`：比较操作；

简单例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <stack>
using namespace std;

typedef deque<int> de;
typedef stack<int> st;

int main() {
    de d{1, 2, 3, 4, 5};
    st s(d);

    while (!s.empty()) {
        cout << s.top() << ", ";
        s.pop();
    }
    cout << "\b\b " << endl;

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [20:22:23]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [20:22:25]
$ ./a.out
5, 4, 3, 2, 1
</script></code></pre>


### queue
定义于头文件：`<queue>`
原型：`template <typename T, typename Container = std::deque<T>> class queue;`
- `T`：元素的类型；
- `Container`：底层容器的类型，默认为`std::deque`；


`queue<Elem> c`：构造一个空的 queue；
`queue<Elem> c(C)`：通过底层容器 C 进行构造；
`queue<Elem> c1(c2)`：通过另一个 queue 进行构造；

`c1 = c2`：支持Copy、Move语义，赋值操作；
`c1.swap(c2)`：交换两个容器适配器；
`swap(c1, c2)`：同上，全局函数；

`c.size()`：返回 queue 的元素数量；
`c.empty()`：判空操作；

`c.push(elem)`：入队；
`c.pop()`：出队，不返回元素的值；
`c.front()`：读取队头元素的值；
`c.back()`：读取队尾元素的值；

`operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=`：比较操作；

简单例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <queue>
using namespace std;

typedef deque<int> DQ;
typedef queue<int> Q;

int main() {
    DQ dq{1, 2, 3, 4, 5};
    Q q(dq);

    while (!q.empty()) {
        cout << q.front() << ", ";
        q.pop();
    }
    cout << "\b\b " << endl;

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [20:47:59]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [20:48:23]
$ ./a.out
1, 2, 3, 4, 5
</script></code></pre>


### priority_queue
定义于头文件：`<queue>`
原型：`template <typename T, typename Container = std::vector<T>, typename Compare = std::less<typename Container::value_type>> class priority_queue;`
- `T`：元素的类型；
- `Container`：底层容器的类型，默认为`std::vector`；
- `Compare`：优先级定义，默认为`std::less`即值大的先出队；


`priority_queue<T> c`：默认构造；
`priority_queue<T> c1(c2)`：拷贝/移动构造；
`priority_queue<T, vector<T>, Cmp> c(cmp)`：排序方式 cmp；
`priority_queue<T, vector<T>, Cmp> c(cmp, cont)`：排序方式 cmp，使用 cont 初始化；
`priority_queue<T> c(beg, end)`：区间`[beg, end)`；
`priority_queue<T, vector<T>, Cmp> c(beg, end, cmp)`：区间`[beg, end)`，排序方式 cmp；

`c1 = c2`：支持Copy、Move语义，赋值操作；
`c1.swap(c2)`：交换两个容器适配器；
`swap(c1, c2)`：同上，全局函数；

`c.size()`：返回元素的数量；
`c.empty()`：判空操作；

`c.top()`：访问队头元素；
`c.push(elem)`：入队；
`c.pop()`：出队，不返回元素的值；

简单例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <queue>
using namespace std;

int main() {
    auto cmp = [] (auto &lhs, auto &rhs) { return lhs > rhs; };
    priority_queue<int, vector<int>, decltype(cmp)> q(cmp);

    for (auto &i : {1, 2, 3, 4, 5}) {
        q.push(i);
    }

    while (!q.empty()) {
        cout << q.top() << ", ";
        q.pop();
    }
    cout << "\b\b " << endl;

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [21:25:09]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [21:25:33]
$ ./a.out
1, 2, 3, 4, 5
</script></code></pre>


## 关联容器
### set
定义于头文件：`<set>`
原型：`template <typename Key, typename Compare = std::less<Key>, typename Allocator = std::allocator<Key>> class set;`
- `Key`：元素的类型；
- `Compare`：优先级定义，默认为`std::less`；
- `Allocator`：用于获取/释放内存及构造/析构内存中元素的分配器；


set 是一个关联容器，是一个`有序的集合`，集合中包含`不可重复的、类型为Key的元素`；排序通过使用类型为 Compare 的比较函数比较来实现；搜索，删除和插入操作具有对数时间复杂度；set 通常实现为`红黑树`；

构造函数
`set(const Compare &comp = Compare(), const Allocator &alloc = Allocator());`
`set(InputIt first, InputIt last, const Compare &comp = Compare(), const Allocator &alloc = Allocator());`
`set(const set &other);`
`set(const set &other, const Allocator &alloc);`
`set(set &&other);`
`set(set &&other, const Allocator &alloc);`
`set(std::initializer_list<value_type> init, const Compare &comp = Compare(), const Allocator &alloc = Allocator());`

赋值运算符
`set & operator=(const set &other);`
`set & operator=(set &&other);`

迭代器
`c.begin()`：返回一个迭代器，指向第一个元素；
`c.end()`：返回一个迭代器，指向最后一个元素的下一位置；
`c.rbegin()`：返回一个逆向迭代器，指向第一个元素；
`c.rend()`：返回一个逆向迭代器，指向最后一个元素的下一位置；
`c.cbegin()`：const 修饰的 begin；
`c.cend()`：const 修饰的 end；
`c.crbegin()`：const 修饰的 rbegin；
`c.crend()`：const 修饰的 rend；

容量
`c.empty()`：判断当前 set 是否为空；
`c.size()`：返回当前元素的数量；
`c.max_size()`：返回 set 可容纳的元素最大数量；

修改
`c.insert(elem)`：插入一个 elem 副本；
`c.insert(n, elem)`：插入 n 个 elem 副本；
`c.insert(beg, end)`：插入区间`[beg, end)`；
`c.erase(pos)`：移除 pos 位置上的元素；
`c.erase(beg, end)`：移除区间`[beg, end)`中的元素；
`c.erase(key)`：移除键 key；
`c.clear()`：移除所有元素，将容器清空；
`c1.swap(c2)`：交换 c1 和 c2；
`swap(c1, c2)`：同上，全局函数；

查找
`c.count(key)`：返回 key 的数目；
`c.find(key)`：返回 key 的迭代器；
`c.equal_range(key)`：返回匹配特定键的元素范围；
`c.lower_bound()`：返回一个迭代器，指向第一个“大于等于”给定值的元素；
`c.upper_bound()`：返回一个迭代器，指向第一个“大于”给定值的元素；

查询
`c.key_comp()`：返回用于比较键的函数；
`c.value_comp()`：返回用于在 value_type 类型的对象中比较键的函数；

比较
`operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=`

简单例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <algorithm>
#include <functional>
#include <set>
using namespace std;

int main() {
    set<int> s1{1, 1, 2, 2, 3, 3, 3, 4, 5};
    for (auto &i : s1) {
        cout << i << ", ";
    }
    cout << "\b\b " << endl;

    set<int, greater<int>> s2{1, 1, 2, 2, 3, 3, 3, 4, 5};
    for (auto &i : s2) {
        cout << i << ", ";
    }
    cout << "\b\b " << endl;

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:01:54]
$ g++ a.cpp

# root @ arch in ~/work on git:master x [9:02:03]
$ ./a.out
1, 2, 3, 4, 5
5, 4, 3, 2, 1
</script></code></pre>


### map
定义于头文件：`<map>`
原型：`template <typename Key, typename T, typename Compare = std::less<Key>, typename Allocator = std::allocator<std::pair<const Key, T>>> class map;`
- `Key`：键的类型；
- `T`：值的类型；
- `Compare`：比较器，默认`std::less`；
- `Allocator`：空间分配器，默认`std::allocator`；


map 是一个有序关联容器，包含具有`唯一键`的`键值对`；键使用比较函数 Compare 比较来进行排序；搜索，删除和插入操作具有对数复杂性；map 通常实现为`红黑树`；

构造函数
`map(const Compare &comp = Compare(), const Allocator &alloc = Allocator());`
`map(InputIt first, InputIt last, const Compare &comp = Compare(), const Allocator &alloc = Allocator());`
`map(const map &other);`
`map(const map &other, const Allocator &alloc);`
`map(map &&other);`
`map(map &&other, const Allocator &alloc);`
`map(std::initializer_list<value_type> init, const Compare &comp = Compare(), const Allocator &alloc = Allocator());`

赋值运算符
`map & operator=(const map &other);`
`map & operator=(map &&other);`

访问元素
`T & at(const Key &key);`
`T & operator[](const Key &key);`
`T & operator[](Key &&key);`

迭代器
`c.begin()`：返回一个迭代器，指向第一个元素；
`c.end()`：返回一个迭代器，指向最后一个元素的下一位置；
`c.rbegin()`：返回一个逆向迭代器，指向第一个元素；
`c.rend()`：返回一个逆向迭代器，指向最后一个元素的下一位置；
`c.cbegin()`：const 修饰的 begin；
`c.cend()`：const 修饰的 end；
`c.crbegin()`：const 修饰的 rbegin；
`c.crend()`：const 修饰的 rend；

容量
`c.empty()`：判断当前 map 是否为空；
`c.size()`：返回当前元素的数量；
`c.max_size()`：返回 map 可容纳的元素最大数量；

修改
`c.insert(elem)`：插入一个 elem 副本；
`c.insert(n, elem)`：插入 n 个 elem 副本；
`c.insert(beg, end)`：插入区间`[beg, end)`；
`c.erase(pos)`：移除 pos 位置上的元素；
`c.erase(beg, end)`：移除区间`[beg, end)`中的元素；
`c.erase(key)`：移除键 key；
`c.clear()`：移除所有元素，将容器清空；
`c1.swap(c2)`：交换 c1 和 c2；
`swap(c1, c2)`：同上，全局函数；

查找
`c.count(key)`：返回 key 的数目；
`c.find(key)`：返回 key 的迭代器；
`c.equal_range(key)`：返回匹配特定键的元素范围；
`c.lower_bound()`：返回一个迭代器，指向第一个“大于等于”给定值的元素；
`c.upper_bound()`：返回一个迭代器，指向第一个“大于”给定值的元素；

查询
`c.key_comp()`：返回用于比较键的函数；
`c.value_comp()`：返回用于在 value_type 类型的对象中比较键的函数；

比较
`operator==`、`operator!=`、`operator<`、`operator<=`、`operator>`、`operator>=`

简单例子：
<pre><code class="language-cpp line-numbers"><script type="text/plain">#include <iostream>
#include <algorithm>
#include <functional>
#include <map>
#include <string>
using namespace std;

int main() {
    map<char, string> ipaddr{
        {'A', "0.0.0.0/8 - 127.255.255.255/8"},
        {'B', "128.0.0.0/16 - 191.255.255.255/16"},
        {'C', "192.0.0.0/24 - 223.255.255.255/24"},
        {'D', "224.0.0.0/4 - 239.255.255.255/4"},
        {'E', "240.0.0.0/4 - 255.255.255.255/4"},
    };

    for (auto i=ipaddr.begin(); i!=ipaddr.end(); i++) {
        cout << "IP地址类别: " << i -> first << "\tIP地址范围: " << i -> second << endl;
    }

    cout << "-----------------------------------------------------------" << endl;

    for (auto &i : ipaddr) {
        cout << "IP地址类别: " << i.first << "\tIP地址范围: " << i.second << endl;
    }

    cout << "-----------------------------------------------------------" << endl;

    for_each(ipaddr.begin(), ipaddr.end(), [](auto &i){cout << "IP地址类别: " << i.first << "\tIP地址范围: " << i.second << endl;});

    return 0;
}
</script></code></pre>

<pre><code class="language-cpp line-numbers"><script type="text/plain"># root @ arch in ~/work on git:master x [9:33:41] C:130
$ g++ a.cpp

# root @ arch in ~/work on git:master x [9:33:43]
$ ./a.out
IP地址类别: A	IP地址范围: 0.0.0.0/8 - 127.255.255.255/8
IP地址类别: B	IP地址范围: 128.0.0.0/16 - 191.255.255.255/16
IP地址类别: C	IP地址范围: 192.0.0.0/24 - 223.255.255.255/24
IP地址类别: D	IP地址范围: 224.0.0.0/4 - 239.255.255.255/4
IP地址类别: E	IP地址范围: 240.0.0.0/4 - 255.255.255.255/4
-----------------------------------------------------------
IP地址类别: A	IP地址范围: 0.0.0.0/8 - 127.255.255.255/8
IP地址类别: B	IP地址范围: 128.0.0.0/16 - 191.255.255.255/16
IP地址类别: C	IP地址范围: 192.0.0.0/24 - 223.255.255.255/24
IP地址类别: D	IP地址范围: 224.0.0.0/4 - 239.255.255.255/4
IP地址类别: E	IP地址范围: 240.0.0.0/4 - 255.255.255.255/4
-----------------------------------------------------------
IP地址类别: A	IP地址范围: 0.0.0.0/8 - 127.255.255.255/8
IP地址类别: B	IP地址范围: 128.0.0.0/16 - 191.255.255.255/16
IP地址类别: C	IP地址范围: 192.0.0.0/24 - 223.255.255.255/24
IP地址类别: D	IP地址范围: 224.0.0.0/4 - 239.255.255.255/4
IP地址类别: E	IP地址范围: 240.0.0.0/4 - 255.255.255.255/4
</script></code></pre>
