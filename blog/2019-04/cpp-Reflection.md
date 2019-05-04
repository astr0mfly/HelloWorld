---
date: "2019-04-25T03:10:41-07:00"
draft: false
title: "C++如何做到反射"
tags: [cpp]
#series: []
categories: ["编码之道"]
toc: true
---

## 什么是反射
反射机制在java中是一个非常重要的特性，比如在spring框架中，就使用了xml + 反射来完成类的动态扩展。
换句话说，反射就是根据一个字符串查找到一个类，查找到一个函数，并能生成这个类，或者函数的实例；就是在运行期通过字符串到内存单元的反向映射。

## 怎么做到反射
在动态语言中，反射是非常容易实现的，在java中，由于有字节码作为中间层的存在，所以实现也并不复杂。但是对于c++来说，静态编译的程序在运行期能打交道的，只是一些内存地址，没有任何字符串相关的东西，也就是说，你写程序时的那些变量名，真正编译结束后，只是一些内存地址。当然了，万事没有绝对。 要想在c++中实现类似java的反射功能，也是可以做到的，比如——增加中间层。
unix设计艺术里曾说过：所有的问题都可以通过增加中间层的方式来解决。

### 编译期增加中间层
程序在编译期，是有一个全局的符号对照表的，这个里面保存了字符串（变量或者类名）到内存地址的映射，但是一旦编译完成，在链接时，这些“多余”的信息都将不存在。我们只要把静态链接改为动态链接，就可以将需要的符号信息保存下来。

比如：
1. 将所有需要动态创建的类放置于一个或多个独立的文件中；
2. 将这些文件编译为一个.so文件；
3. 在主程序中通过dlopen打开so文件；
4. 调用dlsym(*pvSo, "create")获得需要的函数；

**劣势**
对于c++来说，函数的命名方式和c完全不同，所以需要通过各种extern来修饰函数。而且，由于使用了动态编译，在so中就不能再使用一些模板的特性了。

### 运行期增加中间层
选择静态编译。也就是说在运行期增加一个字符串到函数的中间层。我们需要准备一个全局的map，以字符串为key，函数指针作为value。同时提供一个注册机制，把每个需要的类注册到这个全局map中。

谁来完成这个注册的过程？如果是调用者来注册，那么首先，我们需要在调用的文件中include用到的所有类的头文件，之后，我们需要一个一个把类注册进来，如果需要注册的类有上千个，这个过程将会非常的繁杂。如果是类的作者自己完成注册，就涉及到另一个问题，如何注册？因为注册的这个过程，本身需要在类外面，而且还必须是一段可执行代码。这样就又绕回了上面那步，谁来调用注册代码？


### 类的静态对象
我们知道，类的静态对象是全局的，而所有的全局变量，编译器保证了一定会在main执行前被初始化，换句话说，我们只要把注册代码放置于类的静态变量的初始化过程中，就可以了。

于是我们可以知道
~~~c++
#include <iostream>
#include <string>
#include <map>

using namespace std;

#define BASE_CLASS Test

#define GLOBAL_FUN_MAP FunMap<BASE_CLASS>::get_fun_map()

#define DEFINE_CLASS(class_name, fun_name) \
    class_name(std::string) \
    {\
        GLOBAL_FUN_MAP.regist(#fun_name, class_name::fun_name);\
    }\
    class_name(){}\
    static class_name class_name##_;\
    static BASE_CLASS* fun_name()\
    {\
        return new class_name;\
    }

#define REGIST_CLASS(class_name) \
    class_name class_name::class_name##_(#class_name);

 
template <class T>
class FunMap{
    typedef T* (*FUN)(void);
    map<std::string, FUN> fun_map_;
public:
    void regist(string fun_name, FUN fun){
        fun_map_[fun_name] = fun;  
    }

    T* get(const string fun_name){
        if (fun_map_.end() != fun_map_.find(fun_name)){
            return fun_map_[fun_name](); 
        } else {
            return NULL;
        }
    }

    static FunMap<T>& get_fun_map(){
        static FunMap<T> fun_map;
        return fun_map;
    }
};

 
class Test{

};

 
class Test1 : public Test
{
public:
    DEFINE_CLASS(Test1, test1) //第一个参数是类名，第二个参数是字符串名字。
};

 
class Test2 : public Test{
public:
    DEFINE_CLASS(Test2, test2)
};

REGIST_CLASS(Test1)//在类的实现添加，参数是类名
REGIST_CLASS(Test2)


int main(){

    GLOBAL_FUN_MAP.get("test1");   
    GLOBAL_FUN_MAP.get("test2");   
    GLOBAL_FUN_MAP.get("11111111");
}

~~~

**宏定义知识补充**

>`\`就是当前宏的定义除了本行下面还有，右边不能添加空格

>#define OUT(name) printf(#name);
>以这个为例其中的#name之前的#就表示把name格式化为字符串

>#define T(name) Class## name
>这里的## name 中的##就是连接符的意思,Class和可变字符name连接成一个字符串

## 版本控制

| Version | Action                   | Time       |
| ------- | ------------------------ | ---------- |
| 1.0     | Init                     | 2019-04-25T03:10:41-07:00|
