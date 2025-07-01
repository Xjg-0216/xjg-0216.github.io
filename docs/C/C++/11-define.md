
# 预处理器与宏定义

## 1. 预处理器

>预处理器是 在编译前 处理源代码的程序，C/C++ 源代码在被编译之前，会先由预处理器进行处理

## 1.1 常见预处理指令

| 指令                                   | 作用         |
| ------------------------------------ | ---------- |
| `#include`                           | 引入头文件      |
| `#define`                            | 定义宏        |
| `#undef`                             | 取消宏定义      |
| `#ifdef`                             | 如果定义了宏就编译  |
| `#ifndef`                            | 如果没有定义宏就编译 |
| `#if` / `#elif` / `#else` / `#endif` | 条件编译       |
| `#pragma`                            | 编译器相关的控制指令 |

## 1.2 条件编译

条件编译（conditional compilation）在 C++ 中是非常实用的一种编译时选择机制，通常用于：

- 跨平台编译
- 调试/发布模式切换
- 启用或禁用特性
- 不同硬件或架构的处理

### 1.2.1 常见语法

```cpp
#ifdef 宏名
    // 如果定义了宏名，编译这里的代码
#endif

#ifndef 宏名
    // 如果没有定义宏名，编译这里的代码
#endif

#if 条件
    // 条件满足才编译
#elif 条件
    // 否则如果...
#else
    // 都不满足时
#endif
```




## 2. 宏定义

### 2.1 基本形式

```cpp
#define PI 3.14159
#define MAX(a, b) ((a) > (b) ? (a) : (b))
``` 

- 是 文本替换（不是变量，不占内存）

- 宏函数（带参数的宏）在替换时会展开成代码

- 没有类型检查

### 2.2 宏和const的区别

| 特性     | `#define` 宏  | `const` 常量  |
| ------ | ------------ | ----------- |
| 替换方式   | 编译前的文本替换     | 编译器管理，类型安全  |
| 类型检查   | ❌ 无          | ✅ 有         |
| 调试器可见性 | ❌ 不可见        | ✅ 可见        |
| 作用域控制  | ❌ 全局         | ✅ 局部/全局可选   |
| 支持复杂类型 | ❌ 不支持        | ✅ 支持（如数组、类） |
| 推荐用法   | ❌ 不推荐（仅简单常量） | ✅ 推荐        |


## 3. 头文件保护

如果一个头文件被多次包含，会出现 重复定义 错误。例如：
```cpp
// file1.cpp 和 file2.cpp 都包含了 myheader.h
// main.cpp 又同时包含了 file1.cpp 和 file2.cpp
```
编译器就会两次看到同一个定义

✅ **解决办法**：使用宏做条件编译

```cpp
// myheader.h

#ifndef MYHEADER_H
#define MYHEADER_H

// 声明和定义
int add(int a, int b);

#endif // MYHEADER_H
```

这叫做 Include Guard，防止多次包含头文件。

✅ 等价方式（现代编译器支持）：
```cpp
#pragma once
```
这行放在头文件最上方，效果与 `#ifndef/#define/#endif` 等价，更简洁，但不是所有编译器都支持（现代的如 GCC/Clang/MSVC 都支持）

✅ 示例：完整的头文件保护 + 宏定义
```cpp
// math_utils.h
#ifndef MATH_UTILS_H
#define MATH_UTILS_H

#define MAX(a, b) ((a) > (b) ? (a) : (b))

int add(int a, int b);

#endif  // MATH_UTILS_H
``` 

```cpp
// main.cpp
#include <iostream>
#include "math_utils.h"

int add(int a, int b) {
    return a + b;
}

int main() {
    std::cout << MAX(3, 5) << std::endl;
    std::cout << add(3, 5) << std::endl;
}
```