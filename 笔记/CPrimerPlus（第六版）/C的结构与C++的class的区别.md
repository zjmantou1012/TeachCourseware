C语言和C++语言之间有很多相似之处，但C++引入了一些面向对象的概念，其中最显著的是类（class）。下面是C语言中的结构（struct）与C++中的类（class）之间的一些主要区别：

1. **数据封装：**
    - **C语言：** 结构体（struct）主要用于组织和存储数据，但没有明确的封装特性。结构体中的成员默认是公有的。
    - **C++：** 类（class）引入了私有（private）、保护（protected）和公有（public）等访问控制关键字，提供了更好的数据封装。类的成员默认是私有的，需要通过访问控制关键字显式声明成员的可见性。

```cpp
// C++中的类示例
class MyClass {
private:
    int privateMember;

public:
    int publicMember;
};

```

2. **成员函数：**
    - **C语言：** 结构体中通常只包含数据成员，没有直接支持成员函数。
    - **C++：** 类中可以定义成员函数，这些函数可以访问类的私有成员。成员函数通常用于在类内部定义操作类成员的行为。

```cpp
// C++中的类示例，包含成员函数
class MyClass {
private:
    int privateMember;

public:
    int publicMember;

    void setPrivateMember(int value) {
        privateMember = value;
    }

    int getPrivateMember() {
        return privateMember;
    }
};

```

3. **构造函数和析构函数：**
    - **C语言：** 结构体没有构造函数和析构函数的概念。
    - **C++：** 类可以包含构造函数（用于初始化对象）和析构函数（用于清理对象资源）。构造函数和析构函数的命名与类名相同，前面有一个波浪号（~）表示析构函数。

```cpp
// C++中的类示例，包含构造函数和析构函数
class MyClass {
private:
    int privateMember;

public:
    int publicMember;

    // 构造函数
    MyClass() {
        privateMember = 0;
        publicMember = 0;
    }

    // 析构函数
    ~MyClass() {
        // 清理资源的代码
    }
};

```

总体而言，C++的类提供了更多的面向对象特性，如数据封装、成员函数、构造函数和析构函数等，使得代码更加模块化和可维护。 C++中的类可以看作是对C语言结构体的扩展，引入了更多的面向对象编程的概念。