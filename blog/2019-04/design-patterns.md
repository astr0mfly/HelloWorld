---
date: "2019-04-24T08:14:44-07:00"
draft: false
title: "设计模式"
tags: [“cpp”]
categories: [“编码人生”]
toc: true
---

## [简介](https://www.bilibili.com/video/av24176315/?spm_id_from=333.788.videocard.7)
推荐书籍《设计模式--可复用面向对象软件的基础》
底层思维：向下，把握机器底层->语言构造、编译转换、内存模型、运行机制
抽象思维：面向对象、组件封装、设计模式、架构模式

深入理解三大面向对象机制：
 - 封装，隐藏内部实现
 - 继承，复用现有代码
 - 多态，改写对象行为

## 设计模式详解

### 策略模式

#### 起因
软件构造的过程中，算法和对象容易耦合，那么能不能让算法与对象本身解耦呢？

#### 要点
 1. Strategy及其字类为组建提供了一系列可重用的算法，使得类型在运行时方便根据需要在算法之间进行切换
 2. 消除条件判断语句就是在解耦合
 3. 如果Strategy对象没有实例变量，那么各个上下文可以共享同一个Strategy对象，从而节省对象开销

#### 示例
~~~C++
//BEFORE:
enum TaxBase {
    CN_TAX,
    US_TAX,
    UA_TAX
}
class SalesOrder{
    TaxBase tax;

public:
    double CalcTax(){
        if (tax == CN_Tax){
            //do something
        } else if （tax == US_Tax）{
            //do something
        } else if (tax == UA_Tax) {
            //do something
        } else {
            //exception
        }
    }
};
//AFTER:
class TaxStrategy{
public:
    virtual double Calc(const Context& context)
    virtual ~TaxStrategy(){}
};

class CN_Tax : public TaxStrategy {
public:
    virtual double Calc(const Context& context){
        //do something
    }
}

class US_Tax : public TaxStrategy {
public:
    virtual double Calc(const Context& context){
        //do something
    }
}

class UA_Tax : public TaxStrategy {
public:
    virtual double Calc(const Context& context){
        //do something
    }
}

class SalesOrder{
private:
    TaxStrategy* strategy;

public:
    SalesOrder(TaxStrategy* strategy){
        this->strategy = strategyFactory->NewStrategy();
    }

    ~SalesOrder(){
        delete this->strategy;
    }

    public double CalcTax(){

        double val = strategy->Calc(context);
    }

};
~~~

### 迭代器模式
#### 起因
但对于集合对象，我们希望不暴露其内部结构的同时，可以让外部客户代码透明地访问其中包含的元素； 同时这种”透明遍历“也为”同一种算法在多种集合对象上进行操作“提供了可能?

#### 要点
 1. 迭代抽象：访问一个聚合对象的内容而无需暴露它的内部表示。
 2. 迭代多态：为遍历不同的集合对象提供一个统一的接口，从而支持同样的算法在不同的集合结构上进行操作。
 3. 对C++来说是过时的，现在迭代器用模板，面向对象的方式性能低

#### 示例
~~~
template<typename T>
class Iterator{
public:
    virtual void first() = 0;
    virtual void next() = 0;
    virtual bool isDone() const = 0;
    virtual T& current() = 0;
};

template<typename T>
class MyCollection{
public:
    Iterator<T> GetIterator(){
        //...
    }
};

template<typename T>
class CollectionIterator : public Iterator<T>{
    MyCollection<T> mc;
public:
    CollectionIterator(const MyCollection<T> & c): mc(c){ }
    void first() override {

    }
    void next() override {

    }
    bool isDone() const override{

    }
    T& current() override{

    }
};


void MyAlgorithm(){
    MyCollection<int> mc;
    Iterator<int> iter= mc.GetIterator();

    for (iter.first(); !iter.isDone(); iter.next()){
        cout << iter.current() << endl;
    }
}

~~~

### 单例模式
#### 起因
在软件系统中，经常有这样一些特殊的类，必须保证它们在系统中只存在一个实例，才能确保它们的逻辑正确性、以及良好的效率.

#### 要点
 1. Singleton模式中的实例构造器可以设置为protected以允许子类派生。
 2. Singleton模式一般不要支持拷贝构造函数和Clone接口，因为这有可能导致多个对象实例，与Singleton模式的初衷违背
 3. 如何实现多线程环境下安全的Singleton？注意对双检查锁的正确实现。

2. Singleton模式优点：
（1）Singletion模式的构造函数声明为protected，则不允许通过new进行实例化；可以设置为private，更为严格。
（2）Singletion 模式与 factory method 模式配合使用，一般情况下，工厂方法模式在系统中一般只需要一个工厂对象即可，随意可以
工厂对象实现为Singletion模式，则比较简单便捷使用。或者使用静态工厂方法，也比较简单；
（3）系统中某些对象创建、销毁过程开销很大，使用单例模式优势明显。可以在系统启动的时候直接产生单例模式创建。
然后永久驻留在内存的方式来解决。
（4）单例模式能够处理一下全局访问点控制情况；

3. Singleton模式使用场景：
（1）对系统进行全局的控制环境的情形；
（2）创建一个对象消耗资源很多，如处理IO、数据库等资源，可以“有上限多例模式”非常有优势。
（3）需要定义大量静态常量、静态方法的工具类（环境），采用单利，变为创建非静态的成员使用，也可以就默认使用static情况。

4. Singleton 讨论注意问题：
（1）Singleton 存在并发隐患，在单例函数处，需要添加并发控制；
    因为一些对象的构造可能开销很大，所有存在一个时间区间在高并发量的情况下，出现程序异常。
推荐采用饿汉模式创建单例：
static CXXSingletion *pSingletion = new CXXSingletion();
(2) 虽然类的构造函数声明为Protect或者private情况，则在不允许在类定义之外创建对象，但是在Java中可以通过类的特殊复制方法创建。

5. Singletion模式的扩展
（1）有上限的多例模式
    可以设计在系统启动时候，创建某些开销很大对象的多例模式，方便修正单例模式在性能响应的不足。

#### 示例
~~~C++
//pattern.h
#pragma once
#ifndef _CXXSIINGLETION_H_
#define _CXXSINGLETION_H_

class Singleton{
private:                 //或者 protected:
    Singleton();
    Singleton(const Singleton& other);
public:
    ~Singleton();
    static Singleton* getInstance();
private:
    static Singleton* m_instance;
};
#endif

//pattern.cpp
#include "XXSingletion.h"
#include <iostream>
Singleton* Singleton::m_instance=nullptr;

//线程非安全版本
Singleton* Singleton::getInstance() {
    if (m_instance == nullptr) {
        m_instance = new Singleton();
    }
    return m_instance;
}

//线程安全版本，但锁的代价过高
Singleton* Singleton::getInstance() {
    Lock lock;
    if (m_instance == nullptr) {
        m_instance = new Singleton();
    }
    return m_instance;
}

//双检查锁，但由于内存读写reorder不安全
Singleton* Singleton::getInstance() {
    if(m_instance==nullptr){
        Lock lock;
        if (m_instance == nullptr) {
            m_instance = new Singleton();
            std::cout << "create.." << std::endl;
        }
    }
    return m_instance;
}

Singleton::~Singletion()
{
    if (m_instance)
        delete m_instance;
    m_instance = nullptr;
}

//C++ 11版本之后的跨平台实现 (volatile)
std::atomic<Singleton*> Singleton::m_instance;
std::mutex Singleton::m_mutex;
Singleton* Singleton::getInstance() {
    Singleton* tmp = m_instance.load(std::memory_order_relaxed);
    std::atomic_thread_fence(std::memory_order_acquire);//获取内存fence
    if (tmp == nullptr) {
        std::lock_guard<std::mutex> lock(m_mutex);
        tmp = m_instance.load(std::memory_order_relaxed);
        if (tmp == nullptr) {
            tmp = new Singleton;
            std::atomic_thread_fence(std::memory_order_release);//释放内存fence
            m_instance.store(tmp, std::memory_order_relaxed);
        }
    }
    return tmp;
}

//main.cpp

// Singleton.cpp : 定义控制台应用程序的入口点。
//
#include "stdafx.h"
#include " pattern.h"
#include <iostream>
using namespace std;

int _tmain(int argc, _TCHAR* argv[])
{
    Singletion* p = Singletion::getInstance();
    p = Singletion::getInstance();
    p = Singletion::getInstance();
    p = Singletion::getInstance();

    system("pause");
    return 0;
}

//----------OUT
create..

~~~


### 工厂模式
#### 起因
一个创建对象的接口，让子类决定实例化哪一个类，工厂方法使一个类实例化延迟到子类
如何应对这种变化？如何绕过常规的对象创建方法(new)，提供一种“封装机制”来避免客户程序和这种“多系列具体对象创建工作”的紧耦合。

#### 要点
 1. 如果没有应对”多系列对象创建“的需求变化，则没有必要使用Abstract Factory模式，这时候使用简单的工厂即可。
 2. ”系列对象“指的是在某一个特定系列的对象之间有相互依赖、或作用的关系。不同系列的对象之间不能相互依赖。
 3. Abstract Factory模式主要在于应用”新系列“的需求变动。其缺点在与难以应对”新对象“的需求变动。

（1）工厂方法是典型的解构框架，高层模块只需要知道产品的“抽象类”，其他的实现类都不需要关心（高层与底层特例解耦）；
高层只依赖产品的抽象类。
（2）屏蔽产品特例类。产品的实例化是由工厂负责的，如使用使用JDBC连接数据库，数据库从MySQL切换到oracle，
只需要改动的地方是棋换一下“驱动名称”，这是factory模式灵活性的直接案例。
（3）扩展性非常优秀。对于后续变化的产品，只需要添加产品类，修改一下工厂方法即可。
（4）良好的封装。上层的调用只需要特定的（约束性）的特征，即可封装同类产品的创建过程，降低模块间的耦合

3.factory method 模式 适用场景：
（1）factory方法 是 一类对象new 的良好替代品。需要考虑是否增加一个工厂类管理的，增加代码的复杂度。
（2）需要灵活地额可扩展的方法，可以考虑factory方法。 

4. factory method 模式的扩展
（1）缩小为“简单工厂方法”模式，或“静态工厂方法”模式
    如果此模块只需要一个工厂方法，则没有必要把他实例处理new 出来，只需要使用类的静态成员方法即可
Factory::getProduct(),这样简化了工厂类的创建。
    调用者也变得简单了，该模式是factory method 方法的弱化， 成为Simple Factory Method，
（2）升级为“多工厂类“
    如果初始化一个类会相当复杂，且使代码结构不清晰，所有的类创建都放在一个factory method 方法中，方法会变得巨大
    可以为每一个类的创建都定义一个factory，这样结构清晰简单。或者再抽象出工厂基类，作为协调类，封装子类工厂。
但是因为每一个类对应一个工厂类，维护性变得差，增加了扩展的难度。
（3）延迟初始化，
    以factory method 模式为基础，通过增加map类型成员，容纳不同类型的初始化对象。通过map缓冲，对初始化对象缓冲保留，下次直接取出。
    还可以通过map控制相应类的初始化数量。

#### 示例
简单工厂：
~~~cpp
//Product.h
#pragma once
#ifndef _PRODUCT_H_
#define _PRODUCT_H_

//抽象产品
class CAbsProductA
{
protected://保护构造接口,且抽象类不能实例化
    CAbsProductA();
public:
    ~CAbsProductA();
public:
    //虚函数
    virtual void CommonMethod1() = 0;
};
enum ProductAEnum {eProductA1, eProductA2 };

//具体派生类A
class CProductA1 :public CAbsProductA
{
public:
    CProductA1();
public:
    ~CProductA1();
public:
    void CommonMethod1();

};
class CProductA2 :public CAbsProductA
{
public:
    CProductA2();
public:
    ~CProductA2();
public:
    void CommonMethod1();

};
//具体派生类B...
#endif

//Factory.h
#pragma once
#ifndef _FACTORY_H_
#define _FACTORY_H_
#include "Product.h"

//工厂模式
class CFactory
{
public:
    //CFactory();
    //~CFactory();
    //可以维护一类产品的创建
    static CAbsProductA* getProduct(ProductAEnum productAType);
};

#endif

//factory.cpp
#include "Factory.h"

CAbsProductA* CFactory::getProduct(ProductAEnum type)
{
    CAbsProductA *pProdcutA = nullptr;
    switch (type)
    {
    case eProductA1:
        pProdcutA = new CProductA1();
        break;
    case eProductA2:
        pProdcutA = new CProductA2();
        break;
    }
    if (!pProdcutA)
        throw "CProdutA constructed fail!!!";
    return pProdcutA;
}

//Product.cpp

#include "Product.h"
#include <iostream>
using namespace std;

CAbsProductA::CAbsProductA()
{

}

CAbsProductA::~CAbsProductA()
{
}

CProductA1::CProductA1()
{
    cout << "CProductA1 构建" << endl;
}

CProductA1::~CProductA1()
{
    cout << "CProductA1 析构" << endl;
}

void CProductA1::CommonMethod1()
{
    cout << "CProductA1 业务逻辑" << endl;
}


CProductA2::CProductA2()
{
    cout << "CProductA2 构建" << endl;
}

CProductA2::~CProductA2()
{
    cout << "CProductA2 析构" << endl;
}

void CProductA2::CommonMethod1()
{
    cout << "CProductA2 业务逻辑" << endl;
}

//mian.cpp
#include "stdafx.h"
#include "Factory.h"
using namespace std;
#include <iostream>
int _tmain(int argc, _TCHAR* argv[])
{
    //instream_iteratork
    CFactory* pFactory = new CFactory();
    CAbsProductA* pProductA = pFactory->getProduct(eProductA1);
    pProductA->CommonMethod1();
    pProductA = pFactory->getProduct(eProductA2);
    pProductA->CommonMethod1();

    //简单工厂方法模式
    pProductA = CFactory::getProduct(eProductA1);
    pProductA->CommonMethod1();
    pProductA = CFactory::getProduct(eProductA2);
    pProductA->CommonMethod1();

    system("pause");
    return 0;
}

//--------OUT
A1构建
B1业务
A2构建
B2业务
A1构建
B1业务
A2构建
B2业务
~~~

抽象工厂：
```cpp
//factory.h
#pragma once
#ifndef _FACTORY_H_
#define _FACTORY_H_
#include "Product.h"

//抽象工厂模式
class CAbsFactory
{
public:
    virtual CAbsProductA* getProductA() = 0;
    virtual CAbsProductB* getProductB() = 0;
};

//1类型的不同产品
class CFactory1 : public CAbsFactory
{
public:
    //可以维护一组或相互依赖的类
    CAbsProductA* getProductA();
    CAbsProductB* getProductB();
};

//2类型的不同产品
class CFactory2 : public CAbsFactory
{
public:
    //可以维护一组或相互依赖的类
    CAbsProductA* getProductA();
    CAbsProductB* getProductB();
};

#endif

//factory.cpp
#include "Factory.h"

CAbsProductA* CFactory1::getProductA()
{
    return new CProductA1();
}
CAbsProductB* CFactory1::getProductB()
{
    return new CProductB1();
}

//相互依赖的类组合在一个类中
CAbsProductA* CFactory2::getProductA()
{
    return new CProductA2();
}
CAbsProductB* CFactory2::getProductB()
{
    return new CProductB2();
}

//product.h
#pragma once
#ifndef _PRODUCT_H_
#define _PRODUCT_H_

//抽象产品
class CAbsProductA
{
//protected://保护构造接口,且抽象类不能实例化
//    CAbsProductA();
//public:
//    ~CAbsProductA();
public:
    //虚函数
    virtual void CommonMethodA() = 0;
};
enum ProductAEnum {eProductA1, eProductA2 };

//具体派生类A
class CProductA1 :public CAbsProductA
{
public:
    CProductA1();
public:
    ~CProductA1();
public:
    void CommonMethodA() override;

};
class CProductA2 :public CAbsProductA
{
public:
    CProductA2();
public:
    ~CProductA2();
public:
    void CommonMethodA() override;


};
//具体派生类B...

//抽象产品
class CAbsProductB
{
//protected://保护构造接口,且抽象类不能实例化
//    CAbsProductB();
//public:
//    ~CAbsProductB();
public:
    //虚函数
    virtual void CommonMethodB() = 0;
};


//具体派生类A
class CProductB1 :public CAbsProductB
{
public:
    CProductB1();
public:
    ~CProductB1();
public:
    void CommonMethodB() override;

};
class CProductB2 :public CAbsProductB
{
public:
    CProductB2();
public:
    ~CProductB2();
public:
    void CommonMethodB() override;


};

#endif

//product.cpp
#include "Product.h"
#include <iostream>
using namespace std;

CProductA1::CProductA1()
{
    cout << "CProductA1 构建" << endl;
}

CProductA1::~CProductA1()
{
    cout << "CProductA1 析构" << endl;
}

void CProductA1::CommonMethodA()
{
    cout << "CProductA1 业务逻辑" << endl;
}


CProductA2::CProductA2()
{
    cout << "CProductA2 构建" << endl;
}

CProductA2::~CProductA2()
{
    cout << "CProductA2 析构" << endl;
}

void CProductA2::CommonMethodA()
{
    cout << "CProductA2 业务逻辑" << endl;
}

//
CProductB1::CProductB1()
{
    cout << "CProductB1 构建" << endl;
}

CProductB1::~CProductB1()
{
    cout << "CProductB1 析构" << endl;
}

void CProductB1::CommonMethodB()
{
    cout << "CProductB1 业务逻辑" << endl;
}


CProductB2::CProductB2()
{
    cout << "CProductB2 构建" << endl;
}

CProductB2::~CProductB2()
{
    cout << "CProductB2 析构" << endl;
}

void CProductB2::CommonMethodB()
{
    cout << "CProductB2 业务逻辑" << endl;
}


//dafd

main.cpp
// AbstractFactory.cpp : 定义控制台应用程序的入口点。
//

#include "stdafx.h"
#include <iostream>
#include "Factory.h"

int _tmain(int argc, _TCHAR* argv[])
{
    //抽象工厂，关键通过一个相互关联的类组合在一起，完成不同工厂类的管理
    CAbsFactory *pFactroy = nullptr;
    CAbsProductA *pProductA = nullptr;
    CAbsProductB *pProductB = nullptr;

    pFactroy = new CFactory2();
    pProductA = pFactroy->getProductA();
    pProductB = pFactroy->getProductB();
    pProductA->CommonMethodA();
    pProductB->CommonMethodB();
    delete pFactroy;
    //在创建1类的产品族时，只创建了一个对象,便于维护
    //曾经产品族等级扩展容易，增加新产品困难。
    pFactroy = new CFactory1();
    pProductA = pFactroy->getProductA();
    pProductB = pFactroy->getProductB();
    pProductA->CommonMethodA();
    pProductB->CommonMethodB();
    delete pFactroy;

    system("pause");
    return 0;
}

//--------OUT
A2构建
B2构建
A2业务
B2业务
A1构建
B1构建
A1业务
B1业务

```

### 建造者模式
1. Builder 模式定义：
    将一个复杂的对象的构建与他的表示分离，不同的构建过程可以创建不同的表示。
    要包含四个部分
    （1）抽象构建角色 ： 构建过程抽象方法
    （2）具体构建角色 ： 构建一种类型的对象
    （3）零部件角色 ： 一般采用模板方法来确定所有零部件，
    （4）监管角色Director： 怎么来构建，监管来确定，采用组合的设计原则来封装不同的构建者
    关键是Director 角色完成不同零件的构建程序的协调，并直接向上层返回已经特定构造类型
2. Builder 模式优点
（1）封装性，上层模块不必知道内部构建过程细节。
（2）构建角色相互独立，并且容易扩展，这对程序非常有利。因为抽象构建角色已经定义好接口，很好扩展。
（3）因为构建角色是独立的，所以构建细节进行修改，对其他模块没有影响。

3.Builder 模式应用场景
（1）产品类非常复杂，并且产品类中的构建书序不同产生的对象不同。可以使用。

4.Builder 模式讨论
（1）Builder 模式与Abstract Factory模式相同点，都是创建大量不同类型的对象，
    他们的区别是Builder模式是按照不同类的对象的不同零件的构建顺序创建对象，并通过相同的创建过程来获得不同的对象，他们的不是直接返回的，而是由Director
角色维护创建；Abstract Factory模式对象是直接放回的，即根据创建复杂的抽象接口创建。
    其次，虽然二者都是创建类的，但是他们的注重点不同，Builder 模式注重零件的类型和构建过程顺序，而Abstract Factory注重产生对象。
```cpp
//Builder.h
#pragma once
#ifndef _CBUILDER_H_
#define _CBUILDER_H_
#include "CarModel.h"
//抽象构建者角色
class CBuilder
{
public:
    CBuilder();
    ~CBuilder();
public:
    virtual void setBuilderOrders(const vector<CarBuilderOrder>& orders) = 0;
    virtual CCarModel* getCarModel() = 0;
    //(1)CCArModel 不能是纯虚函数,因为需要只用对象
    //(2)因为父类对象不能直接转化为子类对象，但是可以使用指针向下转型，传递，所以返回父类的指针。
    //即子类指针 可以直接复制给父类指针； 父类指针可以接受子类指针，多态行为。
};

//具体构建者角色
class CBenzeCarBuilder : public CBuilder
{
public:
    CBenzeCarBuilder()
    {
        m_pBenzeCar = new CBenzeCar();
    }
    ~CBenzeCarBuilder()
    {
        delete m_pBenzeCar;
    }
public:
    void setBuilderOrders(const vector<CarBuilderOrder>& orders) override
    {
        m_pBenzeCar->setSequence(orders);
    }
    //返回的是对象的副本 注意！！！
    CCarModel* getCarModel() override
    {
        return m_pBenzeCar;
    }
private:
    CBenzeCar* m_pBenzeCar;
};

//具体构建者角色
class CBMWCarBuilder : public CBuilder
{
public:
    CBMWCarBuilder()
    {
        m_pBMWCar = new CBMWCar();
    }
    ~CBMWCarBuilder()
    {
        delete m_pBMWCar;
    }
public:
    void setBuilderOrders(const vector<CarBuilderOrder>& orders) override
    {
        m_pBMWCar->setSequence(orders);
    }
    //返回的是对象的副本 注意！！！
    CCarModel* getCarModel() override
    {
        return m_pBMWCar;
    }
private:
    CBMWCar* m_pBMWCar;
};


#endif

//Builder.cpp
#include "Builder.h"


CBuilder::CBuilder()
{
}


CBuilder::~CBuilder()
{
}

//CarModel.h
#pragma once
#ifndef _CCARMODEL_H_
#define _CCARMODEL_H

#include <vector>
using namespace std;
//模板方法模式 + 构建模式
//带Hook Method 方法的模板方法模式
enum CarBuilderOrder{eStart, eAlearm, eStop};
#define OrderSize 3

class CCarModel
{
public:
    CCarModel();
    ~CCarModel();
//不对外公开
protected:
    //不能用纯虚函数，因为需要根据父类复制对象
    virtual void start() = 0;
    virtual void alearm() = 0;
    virtual void stop() = 0;
    //Hook Method, 由子类决定模板的顺序情况
    virtual    bool isAlearm()
    {
        return true;
    }
public :
    //对外公开的模板方法
    void setSequence(const vector<CarBuilderOrder>& sequence)
    {
        this->m_eOrders = sequence;
    }
    void run()
    {
        for (auto temp : m_eOrders)
        {
            switch (temp)
            {
            case eStart:
                start();
                break;
            case eAlearm:
                alearm();
                break;
            case eStop:
                stop();
                break;
            default:
                throw "the constructor orders error !!!";
            }
        }
 
    }

private:
    vector<CarBuilderOrder> m_eOrders;
};

//奔驰车在运行不需要警报声...
class CBenzeCar : public CCarModel
{
public:

protected:
    void start() override;
    void alearm() override;
    void stop() override;
    bool isAlearm() override
    {
        return false;
    }

};

class CBMWCar : public CCarModel
{
public:
    CBMWCar()
    {
        //默认是打开
        m_bAlearm = true;
    }
    void setAlearm(bool flag)
    {
        m_bAlearm = flag;
    }
    //不对外公开的方法
protected:
    void start() override;
    void alearm() override;
    void stop() override;
    bool isAlearm()
    {
        return m_bAlearm;
    }
    //提供友好的接口形式
private:
    bool m_bAlearm;
};

#endif

//CarMode.cpp
#include "CarModel.h"
#include <iostream>
using namespace std;

CCarModel::CCarModel()
{
    m_eOrders.reserve(OrderSize);
}


CCarModel::~CCarModel()
{
}

//CBenzeCar
void CBenzeCar::alearm()
{
    cout << "CBenzeCar alearm ... " << endl;
}
void CBenzeCar::start()
{
    cout << "CBenzeCar start ... " << endl;
}
void CBenzeCar::stop()
{
    cout << "CBenzeCar stop ... " << endl;
}


//CBenzeCar
void CBMWCar::alearm()
{
    cout << "CBMWCar alearm ... " << endl;
}
void CBMWCar::start()
{
    cout << "CBMWCar start ... " << endl;
}
void CBMWCar::stop()
{
    cout << "CBMWCar stop ... " << endl;
}

//Decorator.h
#pragma once
#ifndef _CDIRECTER_H_
#define _CDIRECTER_H_
#include "Builder.h"

//监管者角色，采用组合的方式
class CDirecter
{
public:
    CDirecter();
    ~CDirecter();
private:
    CBMWCarBuilder* m_pBMWBuilder;
    CBenzeCarBuilder* m_pBenzeBuilder;
    vector<CarBuilderOrder> m_eOrders;
public:
    //监管者角色分别构建不同类型的对象，与模板方法相结合
    CBenzeCar getBenzeCarA()
    {
        //正常汽车
        m_eOrders.clear();
        m_eOrders.push_back(eStart);
        m_eOrders.push_back(eAlearm);
        m_eOrders.push_back(eStop);

        //设置构建顺序
        m_pBenzeBuilder->setBuilderOrders(m_eOrders);
        //父类转化为子类
        //采用传递对象副本的方式，能与创建新的对象。
        return *(static_cast<CBenzeCar*>(m_pBenzeBuilder->getCarModel()));
    }
    CBenzeCar getBenzeCarB()
    {
        //反常汽车
        m_eOrders.clear();
        m_eOrders.push_back(eStop);
        m_eOrders.push_back(eAlearm);
        m_eOrders.push_back(eStart);
        //设置构建顺序
        m_pBenzeBuilder->setBuilderOrders(m_eOrders);
        //父类转化为子类
        //采用传递对象副本的方式，能与创建新的对象。
        return *(static_cast<CBenzeCar*>(m_pBenzeBuilder->getCarModel()));

    }
    CBMWCar getBMWCarA()
    {
        //正常汽车
        m_eOrders.clear();
        m_eOrders.push_back(eStart);
        m_eOrders.push_back(eAlearm);
        m_eOrders.push_back(eStop);

        //设置构建顺序
        m_pBMWBuilder->setBuilderOrders(m_eOrders);
        //父类转化为子类
        //采用传递对象副本的方式，能与创建新的对象。
        return *(static_cast<CBMWCar*>(m_pBMWBuilder->getCarModel()));
    }
    CBMWCar getBMWCarB()
    {
        //反常汽车
        m_eOrders.clear();
        m_eOrders.push_back(eStop);
        m_eOrders.push_back(eAlearm);
        m_eOrders.push_back(eStart);
        //设置构建顺序
        m_pBMWBuilder->setBuilderOrders(m_eOrders);
        //父类转化为子类
        //采用传递对象副本的方式，能与创建新的对象。
        return *(static_cast<CBMWCar*>(m_pBMWBuilder->getCarModel()));
    }
};

#endif

//Decorator.cpp
#include "Directer.h"


CDirecter::CDirecter()
{
    m_pBenzeBuilder = new CBenzeCarBuilder();
    m_pBMWBuilder = new CBMWCarBuilder();
    m_eOrders.reserve(OrderSize);
}


CDirecter::~CDirecter()
{
}

//main.cpp
// 04Builder.cpp : 定义控制台应用程序的入口点。
//

#include "stdafx.h"
#include "Directer.h"
#include <iostream>
using namespace std;

int _tmain(int argc, _TCHAR* argv[])
{
    CDirecter* director = new CDirecter();
    cout << "BenzeCarA-------------" << endl;
    for (int i = 0; i < 5; i++)
    {
        director->getBenzeCarA().run();
    }
    
    cout << "BenzeCarB-------------" << endl;
    for (int i = 0; i < 5; i++)
    {
        director->getBenzeCarB().run();
    }

    cout << "BMWCarA-------------" << endl;
    for (int i = 0; i < 5; i++)
    {
        director->getBMWCarA().run();
    }

    cout << "BMWCarB-------------" << endl;
    for (int i = 0; i < 5; i++)
    {
        director->getBMWCarB().run();
    }

    system("pause");
    return 0;
}

//--------OUT
```
### 原型模式
1. Prototype  原型模式定义
    提供了类的自我复制的能力，即通过已有对象进行新对象的创建。因为原型模式实在内存中进行二进制流的拷贝，
所以比直接通过new 一个对象性能好。不同的实现方式与具体的语言相关。

2. 原型模式的优点
（1）性能优良，实在内存中基于二进制的拷贝
（2）在进行原型模式的时候，并不通过构造函数。

3. 原型模式的适用场景
（1）需要根据已有对象创建大量的对象；
（2）某个对象的创建需要有大量的前期准备，需要等待其他模块准备完毕；
（3）把相同的对象付给其他对象处理。

4. 原型模式的讨论与注意问题
（1）原型模式注意浅层拷贝与深层拷贝
    语言基础类型浅层拷贝，指针、引用浅层拷贝只传递地址内容需要注意
    C++：需要依赖类的复制构造函数实现
（2）这就是需要提到对象的复制问题

* 类成员是new 多态创建的类成员，要进行深层拷贝，同时要在~XX析构函数中进行析构处理。
* 当记性赋值操作是即=（），右边是一个临时由于运算产生的临时对象，则可以执行高效的赋值操作，即将成员指正直接指向“临时对象”内的成员地址位置，
然后，将“临时对象”的成员指针设置为nullptr，即可。《移动赋值构造》
 原型：
 CXXObject(const CXXObject && LinShi)
  {//移动构造函数
    m_pName = Linshi.m_pName;
    LinShi.m_pName = nullptr;

  }
 （3）实际上，Prototype， Builder, Abstract Factory都是通过一个类（对象） 来创建其他类
但是，他们之间有侧重之分。
builder 侧重对象的构建过程，并不直接返回类对象，由监管者Director协调具体的Builder类分别根据不同的构建过程创建对象；
Abstract Factory 侧重于具体的工厂创建本类型的不同对象，并且是直接返回对象；
Prototye 侧重根据一个对象返回这个对象的副本，必须注意深度拷贝。 

```cpp
// Prototype.h
#pragma once

#include <string>
#include <iostream>
#include <vector>
class CAbsPrototype
{
public:
    CAbsPrototype();
    ~CAbsPrototype();
public:
    virtual CAbsPrototype* clone() const;
};

class CConcentPrototype : public CAbsPrototype
{
private:
    //语言基础类型浅层拷贝，指针、引用浅层拷贝只传递地址内容需要注意
    int m_iAge;
    char* m_pName;//深层拷贝，构造冲初始化，系统中保持协调系统
    std::vector<double> m_dScores;

public:
    // 注意，没有默认值得形参需要排在有默认值的前面，必然的原因
    CConcentPrototype(const std::vector<double>& socresV, const char* aName = "Default Name", int age = 18) :m_iAge(age)
    {
        m_pName = nullptr;

        int l = strlen(aName) + 1;
        m_pName = new char[l];
        strcpy_s(m_pName, l, aName);

        //m_dScores.reserve(socresV.size());
        m_dScores = socresV;
        std::cout << "一般构造函数执行。" << std::endl;
    }
    ~CConcentPrototype()
    {
        delete[] m_pName;
        std::cout << "delete"  << std::endl;
    }
    CConcentPrototype(const CConcentPrototype& obj)//赋值构造函数
    {
        m_iAge = obj.m_iAge;

        int l = strlen(obj.m_pName) + 1;
        m_pName = new char[l];
        strcpy_s(m_pName, l, obj.m_pName);

        m_dScores.clear();
        m_dScores = obj.m_dScores;
        std::cout << "赋值构造函数执行。" << std::endl;
    }
    //要考虑深层拷贝与浅层拷贝
    CAbsPrototype* clone() const override;
    void outPut()
    {
        std::cout << m_pName << m_iAge << "  score:";

        for (auto t : m_dScores)
            std::cout << t << "   ";
            std::cout << std::endl;
    }

};

//Prototype.cpp
#include "AbsPrototype.h"

CAbsPrototype::CAbsPrototype()
{
}
CAbsPrototype::~CAbsPrototype()
{
}

CAbsPrototype* CAbsPrototype::clone() const
{
    return nullptr;
}

//
CAbsPrototype* CConcentPrototype::clone() const
{
    //利用赋值构造函数实现 Prototype 模式
    CConcentPrototype* p = new CConcentPrototype(*this);
    return p;
    //子类指针，赋值给父类指针，在调用处转化为子类指针
}

//main.cpp
// Prototype.cpp : 定义控制台应用程序的入口点。
//
#include "stdafx.h"
#include "AbsPrototype.h"
#include <iostream>


int _tmain(int argc, _TCHAR* argv[])
{
    std::vector<double> scores{11.2, 33.4, 45.6, 99.9};
    CConcentPrototype* pObj = new CConcentPrototype(scores, "zhongguo", 26);
    pObj->outPut();

    std::cout << "-----------------" << std::endl;
    for (int i = 0; i < 10; i++)
    {
        CConcentPrototype* pTem = (CConcentPrototype*)pObj->clone();
        pTem->outPut();
    }

    system("pause");
    return 0;
}

//------------OUT
一般构造函数执行
zhongguo 26 score:11.2 33.4 45.6 99.9
---------------
赋值构造函数执行
zhongguo 26 score:11.2 33.4 45.6 99.9
赋值构造函数执行
zhongguo 26 score:11.2 33.4 45.6 99.9
赋值构造函数执行
zhongguo 26 score:11.2 33.4 45.6 99.9


```
### 桥接模式
1. Bridge模式的定义
    将抽象与实现解耦，使两个可以独立的发生变化。桥接模式的重点在“解耦”。
    一般桥接模式有以下四个角色
    （1）“抽象角色的抽象”：通过组合保留“实现橘色”的引用。
    （2）具体的“抽象角色”：通过组合引用的实现类，对实现类抽象进行操作处理。
    （3）“实现角色的抽象”：抽象类，定义实现角色必须的动作和行为。
    （4）具体的实现角色：完成实现类角色抽象的具体的实现。
    主要是通过抽象角色引用实现角色（组合的方式），或者是抽象角色的部分实现是实现角色提供的，
即实现角色通过实现角色的抽象的引用为抽象角色提供部分的实现支持。
即（4）通过（3）的必须的动作和行为接口为（1）的（2）完成部分或者全部的实现支持。

2. Bridge 模式的优点
(1)抽象与实现相分离
    解决的继承的缺点，因为继承严格限制了类之间的强关联关系，而Bridge模式则描述了一种类之间这种弱的关联关系，实现可以不受抽象的约束
，不需要被严格限制在抽象层面的。
（2）优秀的扩展能力
    抽象与实现都发生变化，从而实现“解耦”，扩充能力强
（3）抽象层通过聚合的方式，实现了对实现的封装。

3. Bridge 模式适用场景
（1）接口或者抽象类不明确的情况
（2）用继承存在问题，继承层次过度，无法进行更细致化的操作，需要使用桥接模式
（3）重用性要求较高，采用继承则严格收到父类的限制，不可以设计太细的颗粒度。
（4）在系统进行设计的时候，如果类的继承有N层，则可以考虑桥接模式。

4. Bridge 模式的讨论与注意
（1）继承的特征：可以把公共的方法或属性抽取，父类封装共性，子类实现特性，这是继承的基本运用。
    缺点：父类具有很强的入侵性，子类必须无条件接受父类的一切，并且父类的某个接口不能轻易改动，因为可能影响到底层的子类的执行。
    桥接模式Bridge：描述了类之间的弱的关联关系，可以吧可能会发生变化的方法放出去，对于关联关系很弱的，则考虑用桥接模式。
```cpp
// AbsCorp.h
#pragma once
/*
    通过桥接模式，完成公司与产品的分离
    因为在公司是一个整体，产品是不断变化的，并且公司也是不断兼并重构的，二者都是发展变化的，不能同限制的继承的思想完成二者的处理。
    公司的通用特征是可以抽象的，产品的通用行为可以抽象，公司的子类可以对产品行为进行部分修改处理。具体的产品不同的属性方法可以特殊定义。
*/
#include "AbsProduct.h"

//
class CAbsCorp
{
public:
    CAbsCorp()
    {
        m_pProduct = nullptr;
    }
    ~CAbsCorp()
    {
        delete m_pProduct;
    }
    CAbsCorp(CAbsProduct* pProduct) : m_pProduct(pProduct){};

public:
    virtual void setProduct(CAbsProduct*  pProduct)
    {
        _ASSERT(pProduct);
        if (m_pProduct)
            delete m_pProduct;
        m_pProduct = pProduct;
    }
    virtual void makeMoney()
    {
        m_pProduct->toMake();
        m_pProduct->toSell();
    }
    virtual void expendCrop() = 0;

protected:
    CAbsProduct* m_pProduct;
};

class CCarCop : public CAbsCorp
{
public:

    CCarCop(CAbsProduct* pProduct) : CAbsCorp(pProduct){};
    void expendCrop() override
    {
        cout << "本CAr公司要做些特殊的事情" << endl;
        m_pProduct->doSomethingSpecial();
        cout << "本CAr公司要扩张..." << endl;
    }
};

class CShanZhaiCop : public CAbsCorp
{
public:
    CShanZhaiCop(CAbsProduct* pProduct) : CAbsCorp(pProduct){};
    void expendCrop() override
    {
        cout << "本CShanZhaiCop公司要做些特殊的事情" << endl;
        m_pProduct->doSomethingSpecial();
        cout << "本CShanZhaiCop公司要扩张..." << endl;
    }
};

//AbsProduct.h
#pragma once
#include <iostream>
using namespace std;

class CAbsProduct
{
public:
    CAbsProduct();
    ~CAbsProduct();
public:
    virtual void toMake() = 0;
    virtual void toSell() = 0;
    virtual void doSomethingSpecial() = 0;
};


class CCarProduct : public CAbsProduct
{
public:
    void toMake() override
    {
        cout << "汽车产品进行生产..." << endl;
    }
    void toSell() override
    {
        cout << "汽车产品进行销售..." << endl;
    }
    void doSomethingSpecial() override
    {
        cout << "汽车的市场调研分析..." << endl;
    }
};

class CComputer : public CAbsProduct
{
public:
    void toMake() override
    {
        cout << "Computer进行生产..." << endl;
    }
    void toSell() override
    {
        cout << "Computer进行销售..." << endl;
    }

    void doSomethingSpecial() override
    {
        cout << "Computer市场调研分析..." << endl;
    }
};

// main.cpp
// Bridge.cpp : 定义控制台应用程序的入口点。
//

#include "stdafx.h"
#include "AbsCorp.h"

#include <iostream>
using namespace std;

int _tmain(int argc, _TCHAR* argv[])
{
    //一般公司的操作
    cout << "\n\n一般公司操作...." << endl;
    CAbsCorp* pCorp = new CCarCop(new CCarProduct());
    pCorp->makeMoney();
    pCorp->expendCrop();
    delete pCorp;

    //山寨公司的操作
    cout << "\n\n山寨公司的操作山寨汽车...." << endl;
    pCorp = new CShanZhaiCop(new CCarProduct());
    pCorp->makeMoney();
    pCorp->expendCrop();
    cout << endl;
    cout << "山寨公司要山寨所有的产品了...." << endl;
    pCorp->setProduct(new CComputer());
    pCorp->expendCrop();
    pCorp->makeMoney();
    delete pCorp;

    system("pause");
    return 0;
}

//----------OUT
一般操作。。。。。

山寨公司的操作山寨汽车
XXX
要做特殊的事
XXX


山寨公司要山寨所有产品了
XXX公司有特殊操作
XXX


```
### 适配器模式
1. Adapter 定义
    将一个类的接口变换为客户端所期待的另一种接口形式，使被适配者Adaptee能用用在目标target的环境中
    一般可以分为以下角色：
    （1）Target角色：
        定义把其他类或者对象转化为的目标接口。通常为一个抽象类（接口）。
    （2）Adaptee角色：
        定义被转化的类，它是已经存在的类或者对象
    （3）Adapte角色：
        通过继承Adaptee或者关联组合Adaptee的方式，适配Target目标
    通常适配器实在系统维护后期的系统扩展应用的。

2. Adapter 优点
（1）强两个没有关系的类适配在一起使用使用。提高复用度，维护系统容易，灵活性好。
（2）增加透明度，即把taget的接口实现，委托给了具体的adaptee完成。

3. Adapter 适应条件
（1）系统扩展后期，使用一个现有类或者原来的类，适配新接口或者原来的接口的时候。

4. Adapter 讨论与注意问题
（1）适配器模式的两种实现方式区别：类继承适配器方式与对象适配器方式
    这是面向对象领域的重要概念。
    类继承适配器是通过继承Adaptee方式（private）获取Adaptee的接口以及实现，
即如果以private继承则只获取了实现，如果以public继承则同时也获取了接口，父类接口可以在子类中提供服务。
则可以通过纯虚类继承实现接口继承。
    对象适配器方式，是通过Adapter组合的方式通过adaptee成员获取相应的接口内容，然后适配目标对象的接口。
（2）类继承适配器是类之间的继承关系；而对象适配器是类之间通过组合形成的关联关系。

```cpp
// Adapte.h
#pragma once
//适配器定义
/*
    两种适配器定义
    （1）类适配器方式：adapter私有继承adaptee(或抽象类)，并实现target的接口（继承抽象类）
    （2）对象适配器方式：datapter组合adaptee对象，并实现target的接口（继承抽象类）
*/
#include "TargetAdptee.h"

class CClassAdepter :public CAdbTarger, private CAdeptee
{
public:
    CClassAdepter(int age) :CAdeptee(age){};
public:
    void request() override
    {
        //调用类适配器的被适配者
        doMyWay();
    }

};


class CObjectAdatper : public CAdbTarger
{
public:
    CObjectAdatper()
    {
        m_pAdaptee = nullptr;
    };

    CObjectAdatper(CAdeptee* pAdaptee)
    {
        m_pAdaptee = pAdaptee;
    }
    ~CObjectAdatper()
    {
        delete m_pAdaptee;
    }
public:
    void request() override
    {
        _ASSERT(m_pAdaptee);
        //调用类适配器的被适配者
        m_pAdaptee->doMyWay();
    }


private:
    CAdeptee* m_pAdaptee;

};

//TargetAdptee
#pragma once
#include <iostream>
using namespace std;
//目标类，被适配类
class CAdbTarger
{
public:
    virtual void request() = 0;
};

class CAdeptee
{
public:
    CAdeptee(int age) :m_iAge(age){};
public:
    //被适配的方法
    void doMyWay()
    {
        cout << "我的接口实现，以adeptee方式实现...." << m_iAge << endl;
    }
private:
    int m_iAge;
};

// main.cpp
// Adapter.cpp : 定义控制台应用程序的入口点。
//
#include "stdafx.h"
#include "AdepterDef.h"
#include <iostream>
using namespace std;

int _tmain(int argc, _TCHAR* argv[])
{
    cout << "类适配器方式" << endl;
    CClassAdepter* pAdapter = new CClassAdepter(26);
    pAdapter->request();
    delete pAdapter;
    cout << "对象适配器方式" << endl;
    CObjectAdatper *pObj = new CObjectAdatper(new CAdeptee(28));
    pObj->request();
    delete pObj;

    system("pause");
    return 0;
}

//------OUT
类适配器方式
接口实现。。。
对象适配器方式
接口实现。。。

```
### 装饰器模式
1. Decorator 模式定义
    一般可以分为4个角色
    （1）Component抽象类：定义被装饰者的抽象行为或者特征。
    （2）具体的Component类：具体的待被装饰的对象。
    （3）抽象的Decorator类：
    抽象装饰者需要继承抽象Component类，以拥有Component相同的行为和特征，便于具体的Decorator继承，在具体的Decorator的继承方法中，进行行为装饰。
    抽象Decorator中的成员需要定义为Protected，这样在子类中才能够访问。
    （4）具体的Decorator类：不同的场景使用不同的具体的装饰者类。
    特征：AbsDecorator继承AbsComponent,并且具体的Cdecorator继承自ABSDecorator，所以这三个角色拥有共同的操作方法，但是在
    具体Decorator中的component方法，可以被具体的Decorator装饰修改。

2. Decorator 优点
（1）扩展相当容易，不但AbsComponent可以派生出多种的“被装饰者”，而且AbsDecorator也可以派生出具体的Decorator，
总的来说：不同的具体Decorator可以任意的方法装饰“具体的component”的规范操作方法。

3. Decorator 适用场景

4. Decorator 讨论 
（1）Decorator模式同样在某些情况下，比使用继承多态效果好，因为Decorator更灵活，且继承容易产生问题。
比如继承多态使用时，引起的问题是:通过父类的指针多态的调用子类的个性化接口，所以这样就必须在父类中添加不同子类中的不同的个性化接口
这样，在父类中就众多派生子类中就会出现与自己无关的接口，并且这种情况是无法避免。所以Decorator就非常灵活地应对“被装饰者Component”的个性化问题。
只需要在具体的Decorator中添加相应的个性化接口，然后完成相应“被装饰者”的常规接口的多态调用，用AbsComponent的指针。

```cpp
//component.h
#pragma once
#include <iostream>
using namespace std;

class CAbsComponent
{
public:
    virtual void componentOperation() = 0;
};

//具体的被装饰者的类对象
class CCompontentA : public CAbsComponent
{
public:
    void componentOperation() override
    {
        cout << "*******被装饰者的具体的的操作*********" << endl;
    }
};

//还可以扩展其他component类...

//decorator.h
#pragma once
#include <iostream>
#include "Compontent.h"
class CAbsDecorator : public CAbsComponent
{
public:
    CAbsDecorator()
    {
        m_pComponent = nullptr;
    }
    CAbsDecorator(CAbsComponent* pComponent)
    {
        m_pComponent = pComponent;
    }
    ~CAbsDecorator()
    {
        delete m_pComponent;
    }
public:
    //
    virtual void componentOperation() override
    {
        _ASSERT(m_pComponent);
        m_pComponent->componentOperation();
    }
protected:
    //这样在抽象Decorator的子类中才能够进行引用，并对具体的继承的方法进行装饰
    CAbsComponent* m_pComponent;
};

//具体的装饰者，可以通过不同的行为装饰，被装饰者的行为
class CDecoratorADef : public CAbsDecorator
{
public:
    CDecoratorADef(CAbsComponent* pComponent) :CAbsDecorator(pComponent){};
public:
    //特殊的装饰行为，可以任意修改
    void doSpecialPre() 
    {
        cout << "AAAAA特殊的特殊行为，可以任意修改,。" << endl;
    }
    //
    void componentOperation() override
    {
        cout << "可以通过继承的方式，操作组合对象的同种属性（特殊操作在之前）" << endl;
        doSpecialPre();
        m_pComponent->componentOperation();
    }
};


//具体的装饰者，可以通过不同的行为装饰，被装饰者的行为
class CDecoratorBDef : public CAbsDecorator
{
public:
    CDecoratorBDef(CAbsComponent* pComponent) :CAbsDecorator(pComponent){};
public:
    void doSpecialEnd()
    {
        cout << "BBBBB特殊的特殊行为，可以任意修改,。" << endl;
    }
    //特殊的装饰行为，可以任意修改
    void componentOperation()
    {
        cout << "可以通过继承的方式，操作组合对象的同种属性(特殊操作在之后)" << endl;
        m_pComponent->componentOperation();//组合的被装饰者的行为，
        doSpecialEnd();
    }
};

//main.cpp
// Decorator.cpp : 定义控制台应用程序的入口点。
//

#include "stdafx.h"
#include "DecoratorDef.h"

#include <iostream>
using namespace std;


int _tmain(int argc, _TCHAR* argv[])
{
    cout << "\n用A装饰器装饰ComponentA..." << endl;
    CAbsDecorator* pDecorator = new CDecoratorADef(new CCompontentA());
    pDecorator->componentOperation();
    delete pDecorator;

    
    cout << "\n用B装饰器装饰ComponentA..." << endl;
    pDecorator = new CDecoratorBDef(new CCompontentA());
    pDecorator->componentOperation();
    delete pDecorator;

    system("pause");
    return 0;
}

//----------OUT
A装饰ComponentA..
继承的方式，操作组合对象的同中书向
AAA的特殊行为可以任意修改。


```

### 组合模式
1. Composite组合模式
    又称为部分整体模式，主要用来描述部分与整体的关系。
    将对象组合成树状结构以表示“部分-整体”的层次结构，是用户对“单个对象”和“组合对象”的使用具有一致性。
    一般该模式具有三个角色：
    （1）AbsComponent角色：一般为抽象父类，抽象“单个对象”“组合对象”公共的行为和方法，可以提供一些默认的方法或行为实现。
    （2）Leaf角色：描述的是“单个对象”，具有不能再细分的对象，成为树叶。
    （3）Composite：组合对象，可以成为“树枝”角色，同时完成对lefa，和composite角色的管理组织，一般用Vector类型实现，
    从而形成一个完整的Tree结构，完成由上到下的遍历。

    Leaf和Composite都要继承抽象AbsComponent类，
    如果在Abscomponent类中添加一个指向自己类型的成员，则可以实现又下到上的遍历。

2. Composite优点
(1)高层调用很简单
    因为一棵树的所有结构都是抽象的CAbsComposite的派生类，所以局部枝叶或者树枝没有区别，高层对枝叶或者树枝的处理都一样，简化了高层的代码
具体的处理交给底层细节。
（2）节点扩展自由
    使用组合模式，对枝叶或者树枝的修改很容易。

3. Composite适用场景
（1）维护和展示具有整体部分的场景
（2）只要是树形结构，就一定考虑使用组合模式

4. Composite讨论
（1） 组合模式的实现方式有两种：安全模式，透明模式
安全模式：组合对象角色单独完成对Leaf 树叶 和Composite 树枝的管理，在高层的遍历访问时，不会出现通过Leaf的指针操作Composite的行为方法异常
透明模式：对结构的管理放在抽象AbsComponent角色中，则高层遍历的时候，因为Leaf树叶以及Composite树枝行为都一样，随意对高层来说是透明的，但是
          会出现误调用的异常。
在C++中可以通过dynamic_cast<>来在运行时完成对实际指针所质量的派生类的转换，可以在此之前使用typeid(p) == typeid(ClassA) 来判定是否是需要的处理类型
（2）
    1）vector中保留指针，删除的时候不delete管理的内容；
    2）保留管理对象的副本，
    从效率上说，采取第一种

```cpp
// Composite.
#pragma once
#include <iostream>
#include <string>
#include <vector>
//Composite模式
/*
    组合模式用于管理一个树状结构的模型，故采用此模式管理学校的常用树状的解构层次，并且采用安全实现方式。
    （1）校长
    （2）老师
    （3）学生
    具有反向遍历功能，
    */
using namespace std;
class CAbsPersonComponent
{
public:
    CAbsPersonComponent(const string& name, int age, const string& address) :m_strName(name), m_iAge(age), m_strAddress(address)
    {
        m_pParent = nullptr;
    };
    ~CAbsPersonComponent()
    {
        //指向父类的指针，析构的时候不用收回空间
        //m_pParent = nullptr;
    };
public:
    bool operator== (const CAbsPersonComponent& person)
    {
        return (m_strName == person.m_strName) && (m_iAge == person.m_iAge) && (m_strAddress.compare(person.m_strAddress) == 0);
        //if (m_strName != person.m_strName)
        //    return m_strName.compare(person.m_strName);
        //else if (m_iAge != person.m_iAge)
        //    return m_iAge - person.m_iAge;
        //else
        //    return m_strAddress.compare(person.m_strAddress);
    }
protected:
    //
    virtual string getMyShenFen() = 0;
protected://需要派生类操作所以最好是protected
    string m_strName;
    int m_iAge;
    string m_strAddress;

    CAbsPersonComponent* m_pParent;//保留指向父类的指针，实现由下向上遍历

public:
    //共有的行为属性
    virtual void getInfo()
    {
        cout << "姓名：" << m_strName << "年龄：" << m_iAge << "地址：" << m_strAddress << "身份： " << getMyShenFen() << endl;
        
    };
    virtual void setParent(CAbsPersonComponent* person)
    {
        this->m_pParent = person;
    }
    virtual CAbsPersonComponent* getParent()
    {
        return m_pParent;
    }
};

/*
    Leaf结构定义，即学生身份的定义
    */
#define KEMU 3//定义三门成绩
class CStudentLeaf : public CAbsPersonComponent
{
public:
    CStudentLeaf(const string& name, int age, const string& address, const double scores[]) :
        CAbsPersonComponent(name, age, address)
    {
        for (int i = 0; i < KEMU; i++)
        {
            m_dScores[i] = scores[i];
        }
    }
protected:
    double m_dScores[KEMU];
public:
    //叶子结构特有的行为
    void setScores(const double scores[])
    {
        for (int i = 0; i < KEMU; i++)
        {
            m_dScores[i] = scores[i];
        }
    }
    void getScores()
    {
        cout << "学生分数： " << endl;
        for (auto t : m_dScores)
        {
            cout << t << " ";
        }
        cout << endl;
    }
    string getMyShenFen() override
    {
        return "学生";
    }
};
/*
    树枝组合结构composite, 校长，老师都是管理者
    因为管理者与被管理者只是在逻辑上存在，并不会应为一个管理者的删除，从而所有的Leaf对象都会删除。
    所以（1）vector中保留指针，删除的时候不delte管理的内容；（2）保留管理对象的副本，
    从何效率，采取第一种
*/

class CManagerComposite : public CAbsPersonComponent
{
public:
    CManagerComposite(const string& name, int age, const string& address) :
        CAbsPersonComponent(name, age, address)
    {
    }

public:
    //树枝（树根）结构特有的行为
    void addPersonComponent(CAbsPersonComponent* pPerson)
    {
        pPerson->setParent(this);
        m_vecFellowers.push_back(pPerson);
    }
    /*
        删除只删除管理的指针，不实际删除对象
    */
    void removePersonComponent(CAbsPersonComponent* pPerson)
    {
        //m_vecFellowers.erase(&pPerson);

        auto temp = m_vecFellowers.begin();
        for (; temp != m_vecFellowers.end(); temp++)
        {
            //比较对象比较，或者利用对象地址比较
            if (*(*temp) == *pPerson)
                break;
        }
        if (temp != m_vecFellowers.end())
            m_vecFellowers.erase(temp);
    }
    //返回vector对象的const 引用（提高效率）
    const vector<CAbsPersonComponent*>& getFellows()
    {
        return m_vecFellowers;
    }
    string getMyShenFen() override
    {
        if (m_pParent)
            return "老师";
        else
            return "校长";
    }
private:
    vector<CAbsPersonComponent*> m_vecFellowers;//管理者管理的人员
};

//main.cpp
// Composite.cpp : 定义控制台应用程序的入口点。
//

#include "stdafx.h"
#include "CompositeDef.h"

static void getTreeMap(CAbsPersonComponent* p)
{
    //cout << "当前类的名字： " << typeid(*p).name();
    p->getInfo();
    if (typeid(*p) == typeid(CManagerComposite))
    {
        CManagerComposite* pTem = dynamic_cast<CManagerComposite*>(p);
        vector<CAbsPersonComponent*> vector = pTem->getFellows();//引用
        for (auto temp : vector)
        {
            getTreeMap(temp);
        }
    }
    //其他情况s说明是叶子结构，不在处理
    //else if (typeid(*p) == typeid(CStudentLeaf))
    //{

    //}

}

int _tmain(int argc, _TCHAR* argv[])
{
    //定义校长
    CManagerComposite root("张百顺", 46, "A省B中全区");

    CManagerComposite a("博预期", 35, "A省B中全区");
    CManagerComposite b("张百顺", 46, "A省B中全区");
    CManagerComposite c("卢梦雅", 33, "A省B中全区");
    double scores1[] { 3.4, 33.3, 75.5 };
    double scores2[] { 4.4, 33.3, 75.5 };
    double scores3[] { 5.4, 37.3, 55.5 };
    double scores4[] { 6.4, 33.3, 75.5 };
    double scores5[] { 7.4, 33.3, 65.5 };
    double scores6[] { 9.4, 33.3, 55.5 };

    CStudentLeaf a1("a1", 16, "A省B中全区", scores1);
    CStudentLeaf a2("a2", 16, "A省B中全区", scores1);
    CStudentLeaf b1("b1", 16, "A省B中全区", scores1);
    CStudentLeaf b2("b2", 16, "A省B中全区", scores1);
    CStudentLeaf c1("c1", 16, "A省B中全区", scores1);
    CStudentLeaf c2("c2", 16, "A省B中全区", scores1);

    if (root == b)
        cout << "root 与 b 相等" << endl;

    root.addPersonComponent(&a);
    root.addPersonComponent(&b);
    root.addPersonComponent(&c);

    a.addPersonComponent(&a1);
    a.addPersonComponent(&a2);
    b.addPersonComponent(&b1);
    b.addPersonComponent(&b2);
    c.addPersonComponent(&c1);
    c.addPersonComponent(&c2);


    getTreeMap(&root);
    root.removePersonComponent(&a);
    c.removePersonComponent(&c1);

    cout << endl;
    getTreeMap(&root);

    cout << endl;
    b1.getParent()->getInfo();

    system("pause");
    return 0;
}



```
### 享元（共享单元）模式
1. Flyweight 模式，即享元（共享单元）模式
    Flyweeight是拳击比赛中的特有名词，称为“特轻量级别”，则在设计模式中指的的是类要轻量、类的粒度要细，可以实现细粒度类的复用，但没有
缺乏共享的机制，即多线程下类不可复用。
    享元模式，是“池技术”的重要实现方式，但二者并不等价。使用享元模式可以支持大量的细粒度的对象的共享。
    因为创建太多的对象到程序中有损程序的性能，可以采用享元模式的共享技术，将对象视为为“细粒度对象”然后实现“共享对象”。
细粒度对象是指对象的数量多且性质相近，可以将对象的信息分为“内部状态intrinsic”“外部状态extriinsic”.
内部状态Intrinsic：是对象可共享的信息，存储在具体享元对象的内部，
外部状态Extrinsic：则是被作为一个标记，因为外部状态是根据外部条件制定的标记分类，所以外部状态会随环境改变而改变，所以这部分信息
    一般来说Flyweight可以分为如下4个角色：
（1）AbsFlyweight，抽象享元角色（共享单元）
    就是一个共享单元的抽象行为属性，一般为抽象类，在项目中可能是一个具体类，一般可以把外部状态和内部状态先定义（实现）出来，避免了
派生类中的随意扩展。
（2）ConcreteflyWeight, 具体的享元角色（内部状态初始化）
    具体的一个产品类，实现AbsFlyweight的抽象方法或业务逻辑。完成内部属性的各种个性化操作。
（3）UnshareFlyweight, 不可共享的享元角色（在设计模式中没有体现）
    不能够使用共享技术的对象，该对象一般不出现在享元工厂中。
（4）FlyweightFactory， 享元工厂
    构造一个池容器，同时根据享元中的外部属性最为Key值，搜索池容器中缓冲的向原对象。


2. Flyweight 优点


3. Flyweight 适用场景
（1）使一些细粒度的对象可以共享，采用享元模式。

```cpp

```

### 外观模式
1. 门面模式 facade ，又称为外观模式
    要求所有外部与一个子系统的所有通信必须通过一个“统一的对象”进行。这个对象就是子系统的“门面”，即门面提供一个统一的调用接口，使得
    子系统能够简单使用。
    门面模式注重“统一的对象”，除了这个统一的对象外，不允许以其他方式调用子系统的行为发生。子系统可以是一个类对象或者一组对象的集合，更直观的
    讲不管子系统内是多么杂乱无章，只要“统一对象”是简约整洁即可，即“金玉其外败絮其中”的效果。
    一般两个角色：
    （1）Facade门面对象
        该角色没有实际的业务逻辑，只是一个委托类。一定注意，门面方法不参与业务逻辑，否则就产生一个倒依赖的问题，子系统必须依赖门面才能被访问。
        这违反了接口的单一原则。在系统中，门面模式应该是稳定的，不应该经常发生变化，应该发所有可能发生变化的操作封装在子类中完成，无论怎样，
        从外部来看就是一个单纯的门面。
    （2）Subsystem 子系统对象的集合
2. Facade 模式优点
    

3. Facade 实用场景
    一个系统比较复杂时，可以通过门面模式实现一个很好的封装。如果算法或者业务比较复杂时，可以封装一个多个门面出来，结构简单并且扩展性好。

```cpp
//Fecade.h
#pragma once
#include <iostream>
using namespace std;

class CSubSysA
{
public:
    void doMethodA()
    {
        cout << "A系统的调用" << endl;
    }
};

class CSubSysB
{
public:
    void doMethodB()
    {
        cout << "B系统的调用" << endl;
    }
};

//为了不门面对象不参与具体的业务逻辑，将子系统进行一层的封装，提供统一的接口
class CSubSysC
{
public:
    CSubSysC(CSubSysA* pA, CSubSysB* pB) :m_pSubA(pA), m_pSubB(pB){};
    //不管理成员指针指向对象
    ~CSubSysC()
    {

    }

public:
    void doMethodAB()
    {
        m_pSubA->doMethodA();
        m_pSubB->doMethodB();
    }
private:
    CSubSysA* m_pSubA;
    CSubSysB* m_pSubB;
};
class CFacadeDef
{
public:
    CFacadeDef();
    ~CFacadeDef();
};

class CFacade
{
public:
    CFacade()
    {
        m_pSubA = new CSubSysA();
        m_pSubB = new CSubSysB();
        m_pSubC = new CSubSysC(m_pSubA, m_pSubB);
    }
    ~CFacade()
    {
        delete m_pSubA;
        delete m_pSubB;
        delete m_pSubC;
    }
public:
    //统一模板类接口对象
    void doMethodA()
    {
        m_pSubA->doMethodA();
    }
    void doMethodB()
    {
        m_pSubB->doMethodB();
    }
    //这样处理是为了不使模板类参与具体的业务对象
    void doMethodAC()
    {
        m_pSubC->doMethodAB();
    }

private:
    CSubSysA* m_pSubA;
    CSubSysB* m_pSubB;
    CSubSysC* m_pSubC;
};

//main.cpp
// Facade.cpp : 定义控制台应用程序的入口点。
//

#include "stdafx.h"
#include "FacadeDef.h"

int _tmain(int argc, _TCHAR* argv[])
{
    CFacade* pFacade = new CFacade();
    pFacade->doMethodA();
    pFacade->doMethodB();
    pFacade->doMethodAC();
    delete pFacade;
    system("pause");
    return 0;
}

```
### 代理模式
1. Proxy 代理模式
    为其他对象提供一种代理可以间接控制这个对象的访问。
    又称为“委托模式”，其他设计模式也是在代理模式的基础上扩展的，如“策略模式”“状态模式”“访问者模式”，代理模式的特殊应用。
在Spring的AOP技术，以及Struts的From元素映射就是代理行为。
    一般代理具有多种版本（扩展）如：普通代理模式，强制代理模式，动态代理模式
    一般代理的角色：
    （1）CAbsSubject： 抽象对象，被代理对象的一类的行为，抽象类或者接口。
    （2）Proxy： 代理类，通过组合的方式（has a）拥有一个CabsSubject的指针或者引用，通过将抽象类的接口调用委托被真实对象实现，
    从而可以通过代理控制真实对对象接口的“预处理”“善后处理”工作。代理类必须要是想CAbsSubjecrt接口，这样才能和被代理者保持相似。
    （3）RealSubject： 具体的被代理的对象，具体的业务执行者。

2. 代理的优点
（1）职责清晰
    被代理的对象，真实的subject就是实际的业务逻辑，只完成本职责的事物。
（2）高扩展性
    具体的Subject随时都可能发生变化，只要接口不变，代理类能不变，及时Subject发生变换。
（3）智能化
    通过“动态代理”，完成智能完成多点接入操作。

3. 代理的适用场景
    在实际的编程中，代理的应用场景非常多，SpringAOP（面向切面编程）就是一个智能动态代理应用。
（1）当常见一个大的对象的时候，如创建一个大的图片，可以使用代理来完成创建的过程
（2）为网络上的一个对象创建一个局部的本地代理，我们将操作网络上对象的过程通过操作本地代理来完成。
（3）对对象进行不同的权限控制访问的时候，不同权限的用户提供不同的操作权限，把这一控制过程交给代理对象来完成。
（4）类似于智能指针的操作方法，也使用到了代理思想。
4. 代理的讨论与扩展

（1）代理的扩展
 普通代理模式：
    环境只能访问代Proxy角色，而不能访问真实的Subject。即环境不能new 具体的Subject对象，只能通过代理类操作相应的Subject。
    即代理角色管理真实角色，通过代理角色直接操作真实对象。
    （在实际项目中，一般是通过禁止new真实Subect来实现。）
 强制代理模式：
    强制环境必须通过真实角色来查找代理角色，然后才能访问真实角色的属性方法。即不支持直接new CrealSubject，以及不允许new CProxy
    都不能访问。强制使用CRealSubject指定的CProx类才能访问。即真实角色管理代理类。在CabsSubject中添加getProxy（）方法，返回指定的代理对象。
    并在真实对象实现的方法中进行检测代理对象是否有效来判定限制。
有个性的代理类：
    代理类Proxy不仅可以实现CAbsSubject接口，而且也可以实现其他不同的接口，通过对CRealSubject的借口进行“过滤”拦截，从而增强CRealSubject。
    也可以组合其他真实的角色，这种情况下，代理类的职责并不一定单一。增强CrealSubject的预处理、过滤过滤消息、消息转发、事后处理消息。
 动态代理模式：
    定义，实现阶段并不关心代理的对象是谁，而是在运行阶段才指定哪一个对象。前面自己写代理类的属于“静态代理类”。
    AOP（Aspect Oriented Programming）核心思想就是采用了动态代理机制。在Java语言环境下可以提供动态代理接口InvocationHandler。
    AOP编程，对于我们的日志、事物、权限等都可以在系统设计阶段不用考虑，而在设计之后通过AOP贬称该技术方法，简单实现负责的作用。
    动态代理是代理的职责，是一个通用类，不具有业务意义。具体的CRealsubject实现相关的逻辑功能，二者之间没有相互耦合的关系

```cpp
//Proxy.h
#pragma once
#include <iostream>

using namespace std;

/*
    1.0 最简化的proxy类型结构
    （1）CAbsSubject类抽象接口
    （2）CRealSubject ， CProxy都要实现CAbsSubject接口，这样二者维持相似
    （3）CProxy 组合拥有一个CAbsSubject的引用或者指针，通过此指针完成对真实实现的预处理以及售后处理。
*/
class CAbsSubject
{
public:
    virtual void methodInvoke() = 0;
};

class CRealSubject :public CAbsSubject
{
public:
    void methodInvoke() override
    {
        cout << "实际对象调用方法执行中..." << endl;
    }

};

class CProxy : public CAbsSubject
{
public:
    CProxy(CAbsSubject* p) :m_pSubject(p){};
    //不释放被代理的对象
    ~CProxy(){};
    void methodInvoke() override
    {
        cout << "对象预处理...." << endl;
        m_pSubject->methodInvoke();//真实调用的实现部分
        cout << "对象善后处理...." << endl;
    }

private:
    CAbsSubject* m_pSubject;
};

/*
    1.1A 代理扩展：普通代理模式框架A
    环境只能访问代Proxy角色，而不能访问真实的Subject。即环境不能new 具体的Subject对象，只能通过代理类操作相应的Subject。
    （在代理内部实现真实Subject的创建，然后在真实Subject的沟站函数传入表示病判断）
    //或者直接将CRealSubject类构造声明为Protect：即可，但是太绝对
*/

class CAbsSubjectA
{
public:
    virtual void methodInvoke() = 0;

};

class CRealSubjectA : public CAbsSubjectA
{
public:
    CRealSubjectA(CAbsSubjectA* p = nullptr)
    {
        if (!p)
            throw "不能直接创建具体Subject使用，请借助于派生类...";
        
    }
    void methodInvoke() override
    {
        cout << "实际对象调用方法执行中..." << endl;
    }

};

class CProxyA : public CAbsSubjectA
{
public:
    CProxyA()
    {
        m_pSubject = new CRealSubjectA(this);
    }
    ~CProxyA()
    {
        delete m_pSubject;
    }
    //如果不继承基累的virtual纯虚拟方法，则不能创建对象this；
    void methodInvoke() override
    {
        cout << "对象预处理...." << endl;
        m_pSubject->methodInvoke();//真实调用的实现部分
        cout << "对象善后处理...." << endl;
    }

private:
    CAbsSubjectA* m_pSubject;
};

/*
1.1B 代理扩展：强制代理模式框架A
强制环境必须通过真实角色来查找代理角色，然后才能访问真实角色的属性方法。即不支持直接new CrealSubject，以及不允许new CProxy
都不能访问。强制使用CRealSubject指定的CProx类才能访问。即真实角色管理代理类。在CabsSubject中添加getProxy（）方法，返回指定的代理对象。
并在真实对象实现的方法中进行检测代理对象是否有效来判定限制。
    只要CRealSubject的getprox()没有调用，则成员一致为空，则方法就不能调用，所以必须在CREalSubject中的方法中进行代理类判定。
*/

class CAbsSubjectB
{
public:
    virtual void methodInvoke() = 0;
    virtual CAbsSubjectB* getTheProxy() = 0;
};

class CProxyB : public CAbsSubjectB
{
public:
    //代理特定的对象
    CProxyB(CAbsSubjectB* p)
    {
        m_pSubject = p;
    }
    //只使用，不清楚管理的指针
    ~CProxyB()
    {
    }
    //如果不继承基累的virtual纯虚拟方法，则不能创建对象this；
    void methodInvoke() override
    {
        cout << "对象预处理...." << endl;
        m_pSubject->methodInvoke();//真实调用的实现部分
        cout << "对象善后处理...." << endl;
    }
    //必须实现--没有意义
    CAbsSubjectB* getTheProxy()
    {
        return this;
    }
private:
    CAbsSubjectB* m_pSubject;
};

//智能使用真实对象指定的带来执行
class CRealSubjectB : public CAbsSubjectB
{
public:
    CRealSubjectB()
    {
        m_pProxy = nullptr;
    }
    void methodInvoke() override
    {
        if (isValidateProx())
        {
            //管理指定的具体对象的代理
            cout << "实际对象调用方法执行中..." << endl;
        }
        else
        {
            throw "强制使用(1)CrelSubject指定的(2)Cproxy对象调用...两个条件限制";
        }
    }
    //此方法不调用，m_pProxy永远为nullptr，则其方法永远不能调用
    CAbsSubjectB* getTheProxy()
    {
        m_pProxy = new CProxyB(this);
        return m_pProxy;
    }

private:
    bool isValidateProx()
    {
        if (m_pProxy)
            return true;
        else
            return false;
    }
private:
    CAbsSubjectB* m_pProxy;
};

/*
1.1B 代理扩展：C++实现动态代理机制实现？？？
*/

//main.cpp
// Proxys.cpp : 定义控制台应用程序的入口点。
//

#include "stdafx.h"
#include <iostream>

#include "ProxyDef.h"
using namespace std;


int _tmain(int argc, _TCHAR* argv[])
{

    cout << "--------------------简单框架" << endl;
    CAbsSubject* p = new CRealSubject();
    CProxy* pProxy = new CProxy(p);
    pProxy->methodInvoke();


    cout << "--------------------普通代理类型框架：只允许使用代理类" << endl;
    try
    {
        CAbsSubjectA* pA = new CRealSubjectA();

    }
    catch (const char* error_c)
    {
        cout << "Error catched! ; " << error_c << endl;

    }

    CProxyA* pProxA = new CProxyA();
    pProxA->methodInvoke();


    cout << "--------------------强制代理框架：只允许使用CRealSubject" << endl;
    try
    {
        CAbsSubjectB* pB = new CRealSubjectB();
        CProxyB* pProxyB = new CProxyB(pB);
        //直接另外新建代理调用
        pProxyB->methodInvoke();

    }
    catch (const char* error_c)
    {
        cout << "Error catched! ; " << error_c << endl;

    }

    try
    {
        CAbsSubjectB* pB = new CRealSubjectB();
        //对象调用
        pB->methodInvoke();

    }
    catch (const char* error_c)
    {
        cout << "Error catched! ; " << error_c << endl;

    }
    //正常调用
    CRealSubjectB* pRealB = new CRealSubjectB();
    CAbsSubjectB* pB = pRealB->getTheProxy();
    pB->methodInvoke();

    cout << "--------------------动态代理框架" << endl;

    system("pause");
    return 0;
}


```

## 版本控制

| Version | Action                   | Time       |
| ------- | ------------------------ | ---------- |
| 1.0     | Init                     | 2019-04-24T08:14:44-07:00|
