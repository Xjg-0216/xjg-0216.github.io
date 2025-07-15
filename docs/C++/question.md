# c++常见问题

## 1. 段错误(Segmentation Fault)常见原因

段错误(Segmentation Fault, 简称Segfault)是程序试图访问未被允许访问的内存区域时发生的错误。以下是可能导致段错误的常见原因：

**空指针解引用**
```c
int *p = NULL;
*p = 42;  // 解引用空指针
```

**访问已释放的内存**
```c
int *p = malloc(sizeof(int));
free(p);
*p = 10;  // 访问已释放的内存
```

**数组越界访问**
```c
int arr[10];
arr[10] = 5;  // 越界访问，合法索引是0-9
```

**栈溢出**
```c
void recursive_func() {
    recursive_func();  // 无限递归导致栈溢出
}
```

**修改字符串常量**
```c
char *str = "constant string";
str[0] = 'C';  // 尝试修改只读内存区域
```

**错误的指针运算**
```c
int arr[10];
int *p = arr - 1;  // 非法的指针运算
*p = 0;
```

**错误的类型转换**
```c
int num = 10;
char *p = (char *)num;  // 错误的类型转换
*p = 'a';
```

**多线程竞争条件**
多个线程同时访问同一内存区域，可能导致不可预测的行为。

**调试段错误的工具和方法**

1. **gdb**：GNU调试器，可以定位段错误发生的位置
   ```
   gcc -g program.c -o program
   gdb ./program
   ```

2. **valgrind**：内存调试工具
   ```
   valgrind ./program
   ```

3. **核心转储文件**：分析程序崩溃时的内存状态
   ```
   ulimit -c unlimited
   ./program  # 崩溃后生成core文件
   gdb program core
   ```

4. **打印调试**：在关键位置添加打印语句

预防段错误的最佳实践包括：**始终初始化指针**、**检查内存分配是否成功**、**避免越界访问**、**谨慎使用指针运算**等。


## 2. 智能指针如何克服裸指针的问题

智能指针是C++中管理动态内存的RAII(资源获取即初始化)对象，它们克服了裸指针的许多常见问题：

### 2.1 内存泄漏问题
**裸指针问题**：需要手动`delete`，容易忘记释放内存
```cpp
void func() {
    int* p = new int(10);
    // 如果这里抛出异常或忘记delete，就会内存泄漏
    // delete p;
}
```

**智能指针解决**：自动释放内存
```cpp
void func() {
    std::unique_ptr<int> p(new int(10));
    // 离开作用域时自动释放
}
```

### 2.2 悬垂指针问题
**裸指针问题**：访问已释放的内存
```cpp
int* p = new int(10);
delete p;
*p = 20; // 未定义行为
```

**智能指针解决**：
- `unique_ptr`：独占所有权，不可能出现悬垂指针
- `shared_ptr`：引用计数，最后一个引用消失时才释放
- `weak_ptr`：不增加引用计数，可检测对象是否已被释放

### 2.3 所有权不明确问题
**裸指针问题**：无法明确谁负责释放内存
```cpp
int* create() { return new int(10); }
void consume(int* p) { delete p; }

int* p = create();
// 谁负责delete？不清楚
```

**智能指针解决**：明确所有权语义
```cpp
std::unique_ptr<int> create() { return std::make_unique<int>(10); }
// 所有权转移明确
void consume(std::unique_ptr<int> p) {}

auto p = create();
consume(std::move(p)); // 明确的所有权转移
```

### 2.4 异常安全问题
**裸指针问题**：异常可能导致内存泄漏
```cpp
void func() {
    int* p = new int(10);
    some_function_that_may_throw(); // 如果抛出异常
    delete p; // 这行不会执行
}
```

**智能指针解决**：异常安全
```cpp
void func() {
    auto p = std::make_unique<int>(10);
    some_function_that_may_throw(); // 即使抛出异常
    // p也会被正确释放
}
```

### 2.5 多线程安全问题
**裸指针问题**：多线程环境下难以安全共享数据
```cpp
int* shared_data = new int(0);

// 线程1
*shared_data = 42;

// 线程2
int val = *shared_data; // 数据竞争
```

**智能指针解决**：
```cpp
auto shared_data = std::make_shared<int>(0);

// 线程安全取决于如何使用，但shared_ptr本身保证引用计数原子操作
// 结合mutex可实现线程安全访问
```

### 主要智能指针类型及其用途

1. **`unique_ptr`**：
   - 独占所有权
   - 轻量级，零开销
   - 不可复制，只能移动

2. **`shared_ptr`**：
   - 共享所有权
   - 引用计数
   - 可复制，较重的开销

3. **`weak_ptr`**：
   - 不增加引用计数
   - 解决shared_ptr循环引用问题
   - 必须转换为shared_ptr才能访问对象

智能指针通过自动化内存管理和明确所有权语义，大大减少了C++中与裸指针相关的常见错误，是现代C++内存管理的首选方式。

## 3. NULL和nullptr


| 比较项      | `nullptr`        | `NULL`                    |
| -------- | ---------------- | ------------------------- |
| 类型       | `std::nullptr_t` | 宏（通常是 `0` 或 `((void*)0)`） |
| 是否类型安全   | ✅ 类型安全           | ❌ 可能与整型混淆                 |
| C++11 引入 | ✅                | ❌（C/C++都有）                |
| 推荐程度     | ✅ 推荐（现代 C++）     | ❌ 不推荐（已过时）                |


```cpp
void func(int);
void func(void*);

func(NULL);     // 调用 func(int)?  func(void*)? 编译器可能歧义
func(nullptr);  // 明确调用 func(void*)
```

⚠️ **NULL 的问题**

在 C++ 中 `NULL` 通常是 `#define NULL 0`，即它是 `int` 类型，因此会出现函数重载歧义。

**为什么 `nullptr` 更好**

1. `nullptr` 是一个真正的“空指针”类型

```cpp
std::nullptr_t np = nullptr;  // OK
int i = nullptr;              // ❌ 错误：不能将 nullptr 赋值给 int
```

2. `nullptr` 不会和整型冲突

```cpp
void foo(int x);
void foo(char* p);

foo(0);        // 调用 foo(int)
foo(NULL);     // 调用 foo(int)，容易误判
foo(nullptr);  // 调用 foo(char*)，安全
```

| 场景        | 推荐做法                                         |
| --------- | -------------------------------------------- |
| C++11 及以后 | 使用 `nullptr` 替代 `NULL`                       |
| 与 C 库接口   | 可以继续使用 `NULL`（兼容）                            |
| 判断指针是否为空  | `if (ptr == nullptr)`，不要写 `if (ptr == NULL)` |

如你用的是 C++11 及以上版本（建议默认使用），**请避免使用 `NULL`，统一用 `nullptr`**。这样可以避免二义性和类型错误

## 4. 指针对比

这三个看起来很相似的 C/C++ 声明：

```cpp
int *p[10];
int (*p)[10];
int *p(int);
```

语法结构不一样，**含义完全不同**。下面我逐个为你清晰解释。


### 4.1 `int *p[10];`


> `p` 是一个**数组**，这个数组有 10 个元素，**每个元素是 `int*` 类型**（即指向 int 的指针）。


```text
p: ┌────────────┬────────────┬─── ... ──┬────────────┐
   │ int*       │ int*       │          │ int*       │
   └────────────┴────────────┴──────────┴────────────┘
     p[0]        p[1]                     p[9]
```


```cpp
int a = 1, b = 2;
int *p[10];     // 数组，每个元素都是 int*
p[0] = &a;
p[1] = &b;
```


### 4.2 `int (*p)[10];`


> `p` 是一个**指针**，这个指针指向一个**包含 10 个 `int` 元素的数组**。


```text
p ──► [ int, int, int, ..., int ]  // 长度为10的数组
```

### 举例使用：

```cpp
int arr[10];
int (*p)[10];   // p 是一个指向 int[10] 的指针
p = &arr;       // 注意：取地址时要取整个数组的地址
```

> 这种写法通常用于二维数组、传递数组引用等场景。


### 4.3 `int *p(int);`


> `p` 是一个**函数**，它接受一个 `int` 类型的参数，返回一个 `int*` 类型的值。


```cpp
int* p(int x) {
    static int val = x;
    return &val;
}
```

也可以是函数声明：

```cpp
int *p(int);  // 声明一个函数，返回 int* 类型
```

### 4.4 `int (*p)(int)`


> `p` 是一个**函数指针**，它指向一个函数，这个函数**接收一个 `int` 参数，返回一个 `int` 类型**的值。


1. 先看 `p`：它被 `(*p)` 包裹 → `p` 是**指针**
2. 然后看外面的 `(int)` → 这个指针**指向一个以 `int` 为参数的函数**
3. 最左边 `int` 是这个函数的返回类型

所以整体意思：

```text
p 是一个函数指针：指向一个 (int) → int 的函数
```


```cpp
int add_one(int x) {
    return x + 1;
}
```
```cpp
int (*p)(int);  // 声明函数指针
p = add_one;    // 指向函数
```

```cpp
int result = p(10);      // 和调用函数一样
// 或者 (*p)(10);         // 也是合法写法
```

**函数指针在实际用途**

* 回调函数（如 sort 比较器）
* 接口函数映射
* 动态选择算法（策略模式）
* 事件处理机制


| 声明               | 含义                                            |
| ---------------- | --------------------------------------------- |
| `int *p[10];`    | **数组**：包含 10 个指向 `int` 的指针                    |
| `int (*p)[10];`  | **指针**：指向一个 `int[10]` 的数组                     |
| `int *p(int);`   | **函数**：接受一个 `int` 参数，返回一个 `int*`              |
| `int (*p)(int);` | **函数指针**：`p` 是指针，指向一个接受 `int` 参数、返回 `int` 的函数 |



### 🔧记忆小技巧

| 规则说明                                       | 示例                          |
| ------------------------------------------ | --------------------------- |
| `[]` 优先级比 `*` 高 → `int *p[10]` 是数组，不是指针    | ✅                           |
| `()` 包住 `*` 可以改变结合顺序 → `(*p)[10]` 是指针，指向数组 | ✅                           |
| `p(int)` 说明是个函数                            | `int *p(int)` 是函数，返回 `int*` |
| `(*p)(int)` 表示函数指针                         | `p` 是指向函数的指针，返回 `int`       |



## 5. c++/C的区别

* c++中new/delete是对内存分配的运算符， 取代了C中的malloc/free；
* 标准C++中的字符串取代了C函数库头文件中字符数组处理函数，C中没有字符串类型；
* C++中用来做控制输入输出的iostream类库取代了标准C中的stdio函数库；
* c++中的try/catch/throw异常处理机制取代了C中的setjmp()和ongjmp()函数；
* 在c++中，允许有相同的函数名， 不过它们的参数类型不能完全相同，这样这些函数可以区别开来，而这些C语言中事不允许的， 也就是C++可以重载， C语言不允许；
* c++语言中，允许变量定义语句在程序中的任何地方，只要是在使用它之前就可以；c语言中，必须要在函数开头部分，而且C++不允许重复定义变量， C语言做不到这一点；
* 在c++中，除了值和指针之外，新增了引用。引用型变量是其他变量的一个别名，我们可以认为他们只是名字不相同，其他都是相同的；
* c++相对于c增加了一些关键字， 如：bool, using, dynamic_cast, namespace等；


## 6. `extern"C"`用法


`extern "C"` 是 C++ 中的一个**语言连接规范声明**，主要用于解决 **C++ 与 C 语言混编时的链接兼容性问题**。
它告诉编译器：**按照 C 语言的方式来处理被包裹的函数或变量名，不进行 C++ 名字修饰（name mangling）**。



> **防止 C++ 编译器对函数名进行重整（name mangling），使得 C++ 能调用 C 编译的函数，或让 C 能调用 C++ 的函数。**


例如下面这个 C++ 函数：

```cpp
int add(int a, int b);
```

在 C++ 编译后，函数名可能变成这样：

```cpp
_add_int_int__FiT1  // 举例，因参数类型不同名字会变化
```

而 C 语言中不做这种处理，函数名就简单是 `add`。


### 6.1 声明一个 C 风格的外部函数

```cpp
extern "C" int add(int a, int b);
```

告诉 C++ 编译器：`add` 是一个 C 风格的函数，不要做 name mangling。


### 6.2 在头文件中包裹 C 接口（推荐写法）

```cpp
// mylib.h
#ifdef __cplusplus
extern "C" {
#endif

void c_function();  // 声明的函数将使用 C 链接规则

#ifdef __cplusplus
}
#endif
```

> ✅ 这样可以让 `.h` 头文件同时被 C 和 C++ 包含，安全通用。

假设有一个 C 文件：

```c
// foo.c
#include <stdio.h>

void hello() {
    printf("Hello from C\n");
}
```

对应的头文件：

```c
// foo.h
#ifndef FOO_H
#define FOO_H

void hello();

#endif
```

然后在 C++ 中调用：

```cpp
// main.cpp
extern "C" {
#include "foo.h"
}

int main() {
    hello();
    return 0;
}
```


```cpp
extern "C" class MyClass {};  // ❌ 错误，类不能 extern "C"
```

## 7. 野指针和悬空指针

野指针：没有被初始化过的指针

```cpp
int main(){
   int *p;
   std::cout << *p << std::endl; // 未初始化过就被使用
   return 0;
}
```
因此，为了防止出错，对于指针初始化时都是赋值为nullptr，这样在使用时编译器就不会直接报错，产生非法内存访问；

悬空指针： 指针最初指向的内存已经被释放了的一种指针

```cpp
int main()
{
   int *p = nullptr;
   int *p2 = new int(0);
   p = p2;
   delete p2;
   return 0;
}
```
此时 p 和p2就是悬空指针，指向的内存已经被释放，继续使用这俩指针，行为不可预料。需要设置`p=p2=nullptr`，此时再使用，编译器会直接报错。避免野指针比较简单，但悬空指针比较麻烦。c++引入了智能指针，本质就是避免悬空指针的产生


## 8. 浅拷贝和深拷贝

**浅拷贝**

浅拷贝只是拷贝的一个指针，并没有新开辟一个地址，拷贝的指针和原来的指针指向的同一块地址，如果原来的指针所指向的资源释放了，那么再释放浅拷贝的指针资源就会出现错误。

**深拷贝**

深拷贝不仅拷贝值，还开辟出一块新的空间用来存放新的值，即使原来的对象被析构，释放的内存也不会影响到深拷贝的值。在自己实现拷贝赋值时，如果有指针变量需要自己实现深拷贝

```cpp
#include<iostream>
#include<string>
using namespace std;

class Student
{
private:
      int num;
      char* name;
public:
      Student()
      {
         name = new char(20);
         cout << "Student" << endl;
      };
      ~Student()
      {
         cout << "~Student"<<endl;
         delete name;
         name = NULL;
      };
      Student(const Student &s){//拷贝构造函数
         // 浅拷贝，当对象的name和传入对象的name 指向相同的地址
         name = s.name;
         // 深拷贝
         //name = new char(20);
         //memcpy(name, s.name, strlen(s.name));
         cout << "copy Student" << endl;
      };

};

int main(){

   {
      Student s1;
      Student s2(s1); //复制对象

   }
   return 0;
}

```

浅拷贝在对象的拷贝创建时存在风险，即被拷贝的对象析构释放资源后，拷贝对象析构时再次释放一个已经释放的资源，深拷贝的结果是两个对象之间没有任何关系，各自成员地址不同。

## 9.c++的异常处理方法

常有的异常有：
- 数组下标越界
- 除法计算时除数为0
- 动态分配空间不足

### 9.1 try/throw/catch关键字

| 关键字     | 作用                               |
| ------- | -------------------------------- |
| `try`   | 定义一个可能抛出异常的代码块。必须和 `catch` 搭配使用。 |
| `throw` | 抛出一个异常，可以是内建类型或自定义类型。            |
| `catch` | 捕获异常，并定义如何处理该异常。                 |


cpp中的异常处理机制主要使用try, throw和catch三个关键字，其在程序中的用法如下：
```cpp
#include<iostream>
uusing namespace std;

int main()
{
   double m = 1, n = 0;
   try{
      cout <<"before dividing" << endl;
      if (n == 0)
      {
         throw -1; // 抛出int型异常
      }
      else if (m == 0){
         throw -1.0; //抛出double型异常
      }
      else{
         cout << m / n << endl;
      }
      cout << "after dividing" << endl;
   }
   catch (double d){
      cout << "catch (double)" << d << endl;
   }
   catch (...){
      cout << "catch (...)" << endl;
   }
   cout << "finish" << endl;
   return 0;
}

```
## 10 静态变量什么时候初始化

1. 初始化只有一次，但是可以多次赋值，在主程序之前，编译器已经为其分配好了内存
2. 静态局部变量和全局变量一样，数据都存放在全局区域，所以在主程序之前，编译器已经为其分配好了内存，但是C和C++中静态局部变量的初始化节点又有点不一样。在C中，初始化发生在代码执行之前，编译阶段分配好内存之后，就会进行初始化。所以我们看到在C语言中无法使用变量对静态局部变量进行初始化，在程序运行结束，变量所处的全局内存会被全局回收；
3. 而在C++中，在执行相关代码时才会初始化，主要是由于C++引入对象后，要进行初始化执行相应构造函数和析构函数，在构造函数或析构函数中经常会需要进行某些程序中需要进行的特定操作，并非简单的分配内存，所以C++标准定为全局或静态对象是有首次用到时才会进行构造，所以在C++中是可以使用变量对静态局部变量进行初始化；



## 11 malloc/realloc/calloc的区别

malloc

```cpp
void* malloc(unsigned int num_size);
int *p = malloc(20 * sizeof(int)); // 申请20个int类型的空间
```

calloc

```cpp
void* calloc(size_t n, size_t size);
int *p = calloc(20, sizeof(int)); 
```

省去了人为空间计算，malloc申请的空间的值是随机的， calloc申请的空间的值是初始化为0的；

realloc

```cpp
void realloc(void *p,  size_t new_size);
```
给动态分配的空间分配额外的空间，用于扩充容量



