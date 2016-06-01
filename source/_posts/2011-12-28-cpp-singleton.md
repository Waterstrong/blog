---
title: "C++单例模式实现"
date: 2011-12-28 21:23:50
category: Design Patterns
tags: [设计模式, C++, Singleton]
---

之前遇到关于C++实现单例模式的问题，并非那么简单，主要有部分问题要解决，现在和大家分享一下。我们都知道在Java/C#中实现起来相当容易，但C++确实是有点绕，不过这正是其魅力所在，现在直接上代码，有注释。
<!-- more -->

```c_cpp
#include <iostream>
using namespace std;
class Singleton
{
private:
    static Singleton* sin;//如果定义为static Singleton sin;在C++里这句话相当于有对象产生，还调用了构造函数，而此时此刻的Singleton 还没有创造出来，所以调用里就会有无法解析的外部符号的编译错误
    Singleton() //阻止创建实例
    {
        cout<<"con"<<endl;
    }
public:
    static Singleton* GetInstance()//返回实例，要么返回指针，要么返回引用
    {
        cout<<"ins"<<endl;
        return sin;
    }
    static void Destroy()//应该显式释放sin，如果不显式释放，或者直接在析构函数里delete，当然在外部显式访问析构函数访问没问题，但是在delete sin时会再次调用析构函数，这样将造成不断地循环调用析构函数，这是非常可怕的。
    {
        cout<<"destroy"<<endl;
        if(sin)//显式释放
        {
            delete sin;
            sin = NULL;
        }
    }
    ~Singleton()
    {
        cout<<"descon"<<endl;
    }
};
Singleton* Singleton::sin = new Singleton();//全局初始化, 必须new出来，在编译时就初始化了，如果出现派生的情况也只会有一个惟一的实例，而且构造函数声明为private后，派生类也没法写出构造函数，要注意的一点就是如果只有一个文件就可以直接写在后面，如果有.h和.cpp两个文件，那初始化应该写在.cpp文件中，否则可能会重定义错误。如果初始化为null,下次用时再判断并new的话必须要加双重锁定，消耗比较大，所以就采用这种方式了，不过这种消耗只有第一次才会出现，以后都不会new，但每次都要判断是否为null,也是很不爽的事。

int _tmain(int argc, _TCHAR* argv[])
{
    Singleton* sin1 = Singleton::GetInstance();
    Singleton* sin2 = Singleton::GetInstance();
    if (sin1 == sin2)
    {
        cout<<"the same"<<endl;
    }
    else
    {
        cout<<"not the same"<<endl;
    }
    Singleton::Destroy();//一定要显式调用释放sin，可能还有其方法，但暂时还没有想到，最好能隐式释放，毕竟让客户端负责释放是不太明智的。
    system("pause");
    return 0;
}
```
----

**补充：**

```c_cpp
#include <process.h>
#include <Windows.h>
HANDLE mutex = CreateMutex(NULL,FALSE,NULL); //创建互斥量
WaitForSingleObject(mutex,INFINITE);//等待互斥量
```
//......互斥代码

```c_cpp
ReleaseMutex(mutex);//释放权限，让下一个线程可以进入
CloseHandle(mutex);//关闭创建的mutex句柄
mutex = NULL;
```
双重锁定，当然要先创建互斥量并初始化，最后要清除。

```c_cpp
static Singleton* GetInstance()
{
    if(!sin)
    {
        WaitForSingleObject(mutex,INFINITE);
        if (!sin)
        {
            sin = new Singleton();
        }
        ReleaseMutex(mutex);
        sin = new Singleton();
    }
    return sin;
}
```

