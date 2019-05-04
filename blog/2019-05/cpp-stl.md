---
date: "2019-05-01T08:22:27-07:00"
draft: true
title: "Cpp-Stl"
#tags: []
#series: []
#categories: []
toc: true
---

# STL结构
20世界90年代早期，惠普实验室的Alex Stepanov 和Meng Lio 使用包含了一些类和函数模板的库扩展了C++，这个库称为标准模板库STL，是标准C++库之一。 
标准模板库STL关注的重点是“泛型数据结构”和“算法”，关键的组成部分是“容器（Container）”、“算法（Algorithms）”、“迭代器(Iterators)”。   

容器是某种类型对象的集合。可以将容器理解为对常用“抽象数据类型”的分类。
在STL中，将容器分为两大类型：序列式容器（Sequence Containers）和关联式容器（Associative）。

- 常见比较典型的序列式容器有（1）向量vectgor（2）双端队列deque(3)表list；
- 常见比较典型的关联式容器有（1）集合set（2）多重集合multiset（3）映射map（4）多重映射multimap。

序列式容器和关联式容器的主要区别是数据在容器中的位置关系：
- 序列式容器中数据的位置关系与数据插入容器的时间和顺序以及插入位置有关而与数据的值没有关系；
- 关联式容器中的数据位置与数据的值有关，且存储在关联式容器中的数据通常来自有序集，这些数据通常可以根据某种排序准则来比较大小，数据在关联式容器中的位置关系也是按照数据的这种排序准则来确定的，与数据插入关联容器的时间和位置没有关系。关联式容器中的这种特点，在关联式容器中搜索一个元素的效率较高，通常需要时间复杂度为O(lgN)。
- STL还有一类容器称为“适配器容器”，如栈stack、队列queue、和优先队列priority queue。这一类“适配器容器”实在原有容器的基础上适当裁剪得到的适合“特殊用途”的容器。
-     
迭代器iterator，一种容器元素访问、找出后继、找出前驱的通用方法，即按照一定的顺序访问容器中的元素。使用迭代器iterator的最大的便利之处是可以为任意类型的容器提供一个很小的公共接口，每一个STL容器都提供了一组自己的迭代类型并且（至少）两种返回迭代器的方法

（1）begin：返回一个定位在容器第一个元素的迭代器
（2）end：返回一个定位在容器最后一个元素后面且紧挨着该元素的迭代器。对STL中的所有容器，迭代器接口与C++中的指针几乎一样，可以将迭代器看做指向容器中元素的指针，但是迭代器可以访问更复杂的元素，将迭代器称为“广义指针smart pointer”

```cpp
//迭代器标示容器中的一个位置，声明：vector<int> :: iterator itor;
*itor;    //返回迭代器所指位置处的元素
itor++/itor--;    //返回当前迭代器所指位置的下一个位置/返回当前迭代器所致位置的上一个位置
itor1 == itor2;
itor1 != itor2;    //判断两个迭代器是否相等，即是否所指的是同一位置
itor1 = tior2;    //将一个迭代器的值赋给另一个迭代器
```
STL中的算法，指对容器中的元素进行常用的数据处理，STL中的算法是一种泛型算法，算法中使用迭代器来访问容器中的元素，并且对所有容器迭代器的接口完全相同同意容器。


# 规范
1. "插入操作"
　　新插入的数据位于‘哨兵迭代器“所指的节点的前方，并返回指向新插入位置的‘迭代器（指针）“。这是STL对于插入操作的标准规范。
2.链表操作
　　对于一个链式表，如果添加一个新的节点，双向链表修改的原则：“修新，改旧”，即先修改新创建到节点，使其“前向指针”接管position（哨兵）的前驱，后向指针指向哨兵本身；然后在修改原先链表结构，使哨兵的前驱的后继指针指向新的节点，哨兵的前驱指向新的节点。这样可以防止误操作先修改原来的表结构导致链表的逻辑关系丢失。
　　这样处理，是遵循STL插入操作的规范。
　　对于链表，新插入节点虽然破幻容器的长度，但是原来的迭代器仍然有效；二对于数组or vector，则不然，如果新插入节点导致了容器的长度变化，实质容器有重新申请空间以及数据迁移，所以对于数组类的容器，原来的迭代器将失效。固在遍历容器时，列表类是可以修改链表的结构的。

3迭代器的指向
　　begin（），指向容器的首个元素位置；
　　end（）指向尾部元素位置的后一个元素（空）位置，即指向"超尾"位置；
　　front（），首个元素；
　　back（），末尾元素；

4.对于STL的区间表示，永远是前闭后开的形式，即 [ a, b )
　　表示，包含a元素但不包含b元素，即[a, b-1] 。所以对于区间操作的源代码中，首先一个就是对区间是否有效进行判定，即 if (first != last)

5. 一般STL中的非直接对外公开的函数，一般以 “_ _” 大头；而对外公开的函数，则直接以函数原型开头，如 splice（）
   
6. List不能使用STL的sort()算法，必须使用自己的sort（）成员函数，因为STL的sort（）算法只接受RandomAcctessIterator类型的迭代器。
   
7.deque是一段一段的定量连续空间构成，deque的最大任务是，辨识在这些分段定量连续空间上，维护器整体连续的假象。从而避开vector的’从新配置，复制，释放‘的轮回。所以，除非有必要，我们应尽可能选择vector，而非的确，对的确进行排序操作，为了最高的效率，常常先将deque复制到一个vector上，在vector上排序后，利用STLsort算法，然后在复制回deque。

# 内存与效率

1. 使用reserve()函数提前设定容量大小，避免多次容量扩充操作导致效率低下。
   vector容器 支持随机访问，因此为了提高效率，它内部使用动态数组的方式实现的。

　　(1) size()告诉你容器中有多少元素。它没有告诉你容器为它容纳的元素分配了多少内存。 
　　(2) capacity()告诉你容器在它已经分配的内存中可以容纳多少元素。那是容器在那块内存中总共可以容纳多少元素，而不是还可以容纳多少元素。如果你想 知道一个vector或string中有多少没有被占用的内存，你必须从capacity()中减去size()。如果size和capacity返回同 样的值，容器中就没有剩余空间了，而下一次插入（通过insert或push_back等）会引发上面的重新分配步骤。 
　　(3) resize(Container::size_type n)强制把容器改为容纳n个元素。调用resize之后，size将会返回n。如果n小于当前大小，容器尾部的元素会被销毁。如果n大于当前大小，新默认 构造的元素会添加到容器尾部。如果n大于当前容量，在元素加入之前会发生重新分配。 
　　(4) reserve(Container::size_type n)强制容器把它的容量改为至少n，提供的n不小于当前大小。这一般强迫进行一次重新分配，因为容量需要增加。（如果n小于当前容量，vector忽略 它，这个调用什么都不做，string可能把它的容量减少为size()和n中大的数，但string的大小没有改变。

回到本条款的主旨，通常有两情况使用reserve来避免不必要的重新分配。
　　（1）第一个可用的情况是当你确切或者大约知道有多少元素将最后出现在容器中。那样的话，就像上面的vector代码，你只是提前reserve适当数量的空间。
　　（2）第二种情况是保留你可能需要的最大的空间，然后，一旦你添加完全部 数据，修整掉任何多余的容量。

2. 使用“交换技巧”来修整vector过剩空间/内存
　　有一种方法来把它从曾经最大的容量减少到它现在需要的容量。
　　这样减少容量的方法常常被称为“收缩到合适（shrink to fit）”。该方法只需一条语句：vector<int>(ivec).swap(ivec)。
　　表达式vector<int>(ivec)建立一个临时vector，它是ivec的一份拷贝：vector的拷贝构造函数做了这个工作。但 是，vector的拷贝构造函数只分配拷贝的元素需要的内存，所以这个临时vector没有多余的容量。然后我们让临时vector和ivec交换数据， 这时我们完成了，ivec只有临时变量的修整过的容量，而这个临时变量则持有了曾经在ivec中的没用到的过剩容量。在这里（这个语句结尾），临时 vector被销毁，因此释放了以前ivec使用的内存，收缩到合适。

3. 用swap方法强行释放STL Vector所占内存
```cpp
template < class T> void ClearVector( vector<T>& v )
{ 
    vector<T> vtTemp;
    vtTemp.swap( v );
} 
如 
    vector<int> v ;
    nums.push_back(1);
    nums.push_back(3);
    nums.push_back(2);
    nums.push_back(4);
    vector<int>().swap(v);

/* 或者v.swap(vector<int>()); */

/*或者{ std::vector<int> tmp = v;   v.swap(tmp);   }; 
//加大括号{ }是让tmp退出{ }时自动析构*/,因为退出了临时变量的作用域

```
4. Vector 内存管理成员函数的行为测试
   
5. vector的其他成员函数
```cpp
c.assign(beg,end)
//将[beg; end)区间中的数据赋值给c。
c.assign(n,elem)
//将n个elem的拷贝赋值给c。 
c.at(idx)
//传回索引idx所指的数据，如果idx越界，抛出out_of_range。 
c.back()
//传回最后一个数据，不检查这个数据是否存在。
c.front()
//传回地一个数据。 
get_allocator
//使用构造函数返回一个拷贝。 
c.rbegin()
//传回一个逆向队列的第一个数据。 
c.rend()
//传回一个逆向队列的最后一个数据的下一个位置。 
c.~ vector <Elem>()
//销毁所有数据，释放内存。
```

# algorithm

    搜索算法：find() 、search() 、count() 、find_if() 、search_if() 、count_if() 
    分类排序：sort() 、merge() 
    删除算法：unique() 、remove() 
    生成和变异：generate() 、fill() 、transformation() 、copy() 
    关系算法：equal() 、min() 、max() 

# 容器

# vector
vector 是一个类模板，不是一种数据类型，vector<int>是一种数据类型。
Vector的存储空间是连续的(存储效率高),list不是连续存储的。

## init

　　（1）如果没有指定元素初始化式，标准库自行提供一个初始化值进行值初始化。
　　（2）如果保存的式含有构造函数的类类型的元素，标准库使用该类型的构造函数初始化。
　　（3）如果保存的式没有构造函数的类类型的元素，标准库产生一个带初始值的对象，使用这个对象进行值初始化。

```cpp
vector< typeName > v1;       //默认v1为空，故下面的赋值是错误的v1[0]=5;
//v2是v1的一个副本，若v1.size（）>v2.size()则赋值后v2.size()被扩充为 v1.size()。
vector< typeName >v2(v1); 或v2=v1;
vector< typeName > v2(v1.begin(), v1.end());
vector< typeName > v3(n,i);//v3包含n个值为i的typeName类型元素
vector< typeName > v4(n); //v4含有n个值为0的元素
int a[4]={0,1,2,3,3}; vector<int> v5(a,a+5);//v5的size为5，v5被初始化为a的5个值。后一个指针要指向将被拷贝的末元素的下一位置。
vector<int> v6(v5);//v6是v5的拷贝
vector< 类型 > 标识符（最大容量，初始所有值）；

vector < double > RowVector(COLUMNS, 0.0);
vector < vector < double > > Table(ROWS, RowVector);


reserve： 只改变capacity，不改变size；
resize ： 改变seize，也影响capacity； 
swap：    当与初始对象交换是，快速将一个vector容量变为0，且size变为0
clear：   清空容器元素，size=0，但是capacity不变。
resize：  后语新增加的元素，初始化值，对于原有元素，值不影响。

```

由于vector的内存占用空间只增不减，比如你首先分配了10,000个字节，然后 erase掉后面9,999个，留下一个有效元素，但是内存占用仍为10,000个。所有内存空间是在vector析构时候才能被系统回收。 empty()用来检测容器是否为空的，clear()可以清空所有元素。但是即使clear()，vector所占用的内存空间依然如故，无法保证内存 的回收。
（1）如果需要空间动态缩小，可以考虑使用deque。如果非vector不可，可以用swap()来帮助你释放内存。
（2）但是如果nums是一个类的成员，不能把 vector<int>.swap(nums)写进类的析构函数中，否则会导致double free or corruption (fasttop)的错误，原因可能是重复释放内存。
```cpp
template < class T >
void ClearVector( vector< T >& vt ) 
{
    vector< T > vtTemp; 
    veTemp.swap( vt );
}
```
（3） 如果Vector中存放的是指针
　　如果vector中存放的是指针，那么当vector销毁时，这些指针指向的对象不会被销毁，那么内存就不会被释放。　每次new之后调用v.push_back()该指针，在程序退出或者根据需要，用以下代码进行内存的释放：
```cpp
for (vector<void *>::iterator it = v.begin(); it != v.end(); it ++) 
    if (NULL != *it) 
    {
        delete *it; 
        *it = NULL;
    }
v.clear();

```

# Map
　　STL 中的map 与 hash_map的理解
　　1、STL的map底层是用红黑树存储的，查找时间复杂度是log(n)级别；
　　2、STL的hash_map底层是用hash表存储的，查询时间复杂度是常数级别；
　　3、什么时候用map，什么时候用hash_map?

　　这个要看具体的应用，不一定常数级别的hash_map一定比log(n)级别的map要好，hash_map的hash函数以及解决地址冲突等都要耗时，而且众所周知hash表是以空间效率来换时间效率的，因而hash_map的内存消耗肯定要大。一般情况下，如果记录数非常大时，考虑hash_map，查找效率会高很多，如果要考虑内存消耗，则要谨慎使用hash_map。

 **hash_map和map的区别在哪里？**
构造函数。hash_map需要hash函数，等于函数；map只需要比较函数(小于函数).
存储结构。hash_map采用hash表存储，map一般采用红黑树(RB Tree)实现。因此其memory数据结构是不一样的。

 什么时候需要用hash_map，什么时候需要用map?
　　总 体来说，hash_map 查找速度会比map快，而且查找速度基本和数据数据量大小，属于常数级别;而map的查找速度是log(n)级别。并不一定常数就比log(n)小， hash还有hash函数的耗时，明白了吧，如果你考虑效率，特别是在元素达到一定数量级时，考虑考虑hash_map。但若你对内存使用特别严格，希望 程序尽可能少消耗内存，那么一定要小心，hash_map可能会让你陷入尴尬，特别是当你的hash_map对象特别多时，你就更无法控制了，而且 hash_map的构造速度较慢。 
现在知道如何选择了吗？权衡三个因素: 查找速度, 数据量, 内存使用。

如何用hash_map替换程序中已有的map容器？
　　这个很容易，但需要你有良好的编程风格。建议你尽量使用typedef来定义你的类型： 
typedef map<Key, Value> KeyMap;
　　当你希望使用hash_map来替换的时候，只需要修改: 
　　typedef hash_map<Key, Value> KeyMap;
其他的基本不变。当然，你需要注意是否有Key类型的hash函数和比较函数。 





## 版本控制

| Version | Action                   | Time       |
| ------- | ------------------------ | ---------- |
| 1.0     | Init                     | 2019-05-01T08:22:27-07:00|
