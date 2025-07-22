# 四 内存分配与释放


## 1. 内存的基本分区

C++ 程序运行过程中，内存通常被划分为几个区域：

| 区域       | 描述                                                    |
| -------- | ----------------------------------------------------- |
| 栈（Stack） | 函数调用时的局部变量、参数等，**由编译器自动分配与释放**                        |
| 堆（Heap）  | 程序员手动申请与释放的动态内存，**用 `new/delete` 或 `malloc/free` 管理** |
| 全局区/静态区  | 全局变量、静态变量，程序运行整个生命周期都存在                               |
| 常量区      | 字符串常量、const 常量                                        |
| 代码区      | 存放程序的指令代码                                             |

---

## 2. 栈内存（自动分配）

```cpp
void func() {
    int a = 10;  // a 是局部变量，分配在栈上
}  // 退出函数后，a 的内存自动释放
```

**特点：**

* 分配、释放由编译器管理。
* 执行效率高。
* 空间有限，不适合分配大对象或长生命周期对象。

---

## 3. 堆内存（动态分配）

堆内存由程序员通过关键字手动管理，可以跨函数、动态申请或释放。

---

### 3.1. 使用 `new` / `delete`

```cpp
int* p = new int(10);  // 分配一个 int，并赋值为 10
cout << *p << endl;    // 输出 10
delete p;              // 手动释放内存
```

#### 3.2 分配数组

```cpp
int* arr = new int[5];  // 分配含 5 个 int 的数组
delete[] arr;           // 用 delete[] 释放数组内存
```

---

### 2. 使用 `malloc` / `free`

```cpp
#include <cstdlib>

int* p = (int*)malloc(sizeof(int));  // 分配一个 int 大小的内存
*p = 20;
free(p);  // 释放内存
```

> ❗ **注意：`malloc` 不会调用构造函数，`free` 也不会调用析构函数，不推荐在 C++ 中用于对象分配。**

对类对象的影响
```cpp
class Person {
public:
    Person() { std::cout << "构造\n"; }
    ~Person() { std::cout << "析构\n"; }
};

// new/delete：构造 + 析构
Person* p1 = new Person();  // 输出：构造
delete p1;                  // 输出：析构

// malloc/free：没有构造、没有析构
Person* p2 = (Person*)malloc(sizeof(Person));  // 没有输出
free(p2);                                      // 没有输出
```

---

## 4. 堆内存管理注意事项

| 问题类型                  | 描述                                   |
| --------------------- | ------------------------------------ |
| 内存泄漏（Memory Leak）     | 忘记 `delete` 或 `free` 导致内存无法回收        |
| 野指针（Dangling Pointer） | `delete` 后没有将指针设为 `nullptr`，再使用它就会出错 |
| 重复释放                  | 对同一个地址 `delete` 多次会出错（未设为 `nullptr`） |

### 4.1 示例

```cpp
int* p = new int(5);
delete p;
p = nullptr;  // 👍 安全做法

delete p;     // ✅ 安全（delete nullptr 是无害的）
```

---

## 5. 对象的动态分配

```cpp
class Person {
public:
    Person() { cout << "构造" << endl; }
    ~Person() { cout << "析构" << endl; }
};

int main() {
    Person* p = new Person();  // 构造函数被调用
    delete p;                  // 析构函数被调用
}
```

如果是数组对象：

```cpp
Person* arr = new Person[3];  // 构造函数被调用 3 次
delete[] arr;                 // 析构函数被调用 3 次
```

---

## 6. 智能指针

手动管理内存容易出错，C++11 起推荐使用智能指针，自动管理资源：

```cpp
#include <memory>

std::unique_ptr<int> p1(new int(10));      // 独占式
std::shared_ptr<int> p2 = std::make_shared<int>(20);  // 共享式
```

不用手动 `delete`，离开作用域后自动释放，**更安全更现代**



智能指针是 C++11 引入的 **模板类**，用来 **自动管理堆内存对象的生命周期**，防止常见的内存泄漏和野指针问题。

> 智能指针会在对象不再需要时自动释放资源，相当于“带有自动释放功能的普通指针”。


### 6.1 三种主要智能指针类型

| 智能指针类型            | 简介               | 所需头文件      |
| ----------------- | ---------------- | ---------- |
| `std::unique_ptr` | 独占式拥有，不能共享       | `<memory>` |
| `std::shared_ptr` | 共享式拥有，引用计数管理     | `<memory>` |
| `std::weak_ptr`   | 弱引用，不拥有资源，防止循环引用 | `<memory>` |


`std::unique_ptr`（独占式）

只能有一个 `unique_ptr` 拥有某个对象的所有权，不能复制，但可以转移


```cpp
#include <iostream>
#include <memory>
using namespace std;

class Test {
public:
    Test() { cout << "构造" << endl; }
    ~Test() { cout << "析构" << endl; }
};

int main() {
    unique_ptr<Test> p1(new Test());  // 创建对象
    // unique_ptr<Test> p2 = p1;      // ❌ 错误：不能复制

    unique_ptr<Test> p2 = std::move(p1); // ✅ 所有权转移
    // p1 此时为空
}
```

---

`std::shared_ptr`（共享式）

多个 `shared_ptr` 可以共享同一个对象，**内部使用引用计数来管理内存**


```cpp
#include <iostream>
#include <memory>
using namespace std;

class Test {
public:
    Test() { cout << "构造" << endl; }
    ~Test() { cout << "析构" << endl; }
};

int main() {
    shared_ptr<Test> p1 = make_shared<Test>();
    shared_ptr<Test> p2 = p1; // 引用计数+1
    cout << "引用计数: " << p1.use_count() << endl; // 输出 2
} // 两个指针出作用域后，自动析构
```

---

`std::weak_ptr`（弱引用）

**用于配合 `shared_ptr`，防止循环引用**，它不会增加引用计数。

循环引用问题：

```cpp
struct B; // 前向声明

struct A {
    shared_ptr<B> bptr;
};

struct B {
    shared_ptr<A> aptr; // ❌ 与 A 相互持有 -> 内存泄漏
};
```

解决方案：使用 `weak_ptr`

```cpp
struct A;
struct B;

struct A {
    shared_ptr<B> bptr;
};

struct B {
    weak_ptr<A> aptr;  // ✅ 弱引用，防止循环引用
};
```

---

### 6.2 原理简介（以 `shared_ptr` 为例）

* 每个 `shared_ptr` 指向的对象都有一个 **控制块（control block）**。
* 控制块中保存：

  * 引用计数（shared）
  * 弱引用计数（weak）
  * 指针本体

当 `shared_ptr` 最后一个引用销毁时，析构被调用；但只有当所有 `weak_ptr` 也销毁后，内存才会被释放。

---

### 6.3 使用注意事项

| 项                                          | 建议/说明         |
| ------------------------------------------ | ------------- |
| 不要混用 `shared_ptr` 和原始指针                    | 防止重复释放        |
| 避免 `shared_ptr` 循环引用                       | 使用 `weak_ptr` |
| 不要 `delete` 智能指针指向的对象                      | 会出错，智能指针自动释放  |
| 使用 `make_shared` 比 `shared_ptr(new T)` 更高效 | 减少内存碎片和分配次数   |

---



### 裸指针方式（容易忘记释放）：

```cpp
Person* p = new Person();
// ... 使用 p
delete p; // ⚠️ 如果忘了就内存泄漏
```

### 智能指针方式：

```cpp
shared_ptr<Person> p = make_shared<Person>();
// 自动析构释放，无需手动 delete
```

---

### 6.4总结

| 类型           | 所有权 | 可复制 | 引用计数 | 自动释放                  | 应用场景     |
| ------------ | --- | --- | ---- | --------------------- | -------- |
| `unique_ptr` | 独占  | ❌   | 否    | ✅                     | 独立对象     |
| `shared_ptr` | 共享  | ✅   | 是    | ✅                     | 多人共享的资源  |
| `weak_ptr`   | 弱引用 | ✅   | 不增计数 | ❌（配合 `shared_ptr` 使用） | 解决循环引用问题 |

