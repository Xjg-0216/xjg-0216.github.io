
1. `std::string`（C++风格字符串）
2. `char[]`（C 风格字符串：字符数组）
3. `char*`（C 风格字符串：字符指针）

它们在实际开发中用途不同，各有优劣。下面我会分别讲解它们的**定义、用法、内存管理、注意事项，并附带示例**。

---

## ✅ 一、`std::string`（C++ 风格字符串）

### 🔸 特点

* C++ 标准库中的类（`std::basic_string<char>`）
* 自动内存管理
* 功能丰富：拼接、查找、切片等
* 安全方便，推荐使用

### 🔹 示例

```cpp
#include <iostream>
#include <string>

int main() {
    std::string name = "Alice";

    name += " Smith";  // 拼接
    std::cout << name << std::endl;

    std::cout << name.length() << std::endl;     // 长度
    std::cout << name.substr(0, 5) << std::endl; // 截取
}
```

---

## ✅ 二、`char[]`（字符数组）

### 🔸 特点

* C 风格字符串，用字符数组 + 结尾的 `'\0'` 表示字符串
* 内存大小需要提前分配
* 适合嵌入式、旧接口、C 库函数

### 🔹 示例

```cpp
#include <iostream>
#include <cstring>

int main() {
    char name[20] = "Alice";
    strcat(name, " Smith");  // 字符串拼接
    std::cout << name << std::endl;

    std::cout << strlen(name) << std::endl;  // 字符串长度
}
```

### ⚠️ 注意

* `strcat`, `strcpy`, `strlen` 是 C 函数（在 `<cstring>` 里）
* **数组长度必须保证足够**，否则容易越界导致崩溃

---

## ✅ 三、`char*`（字符指针）

### 🔸 特点

* 指向字符或字符串的首地址
* 通常用于动态分配（如 `new`/`malloc`）或字符串常量
* 常和底层系统接口、C 库接口打交道

### 🔹 示例

```cpp
#include <iostream>

int main() {
    const char* name = "Alice";  // 指向字符串常量，不能修改
    std::cout << name << std::endl;

    // char* name2 = "Alice";   // ❌ C++ 中不推荐，字符串常量不可修改
}
```

### 🔹 动态分配

```cpp
#include <iostream>
#include <cstring>

int main() {
    char* name = new char[20];
    strcpy(name, "Alice");
    strcat(name, " Smith");
    std::cout << name << std::endl;

    delete[] name;  // 释放内存
}
```

---

## 🔄 四、三者互转

### 1. `std::string` ↔ `char*`

```cpp
std::string str = "Hello";
const char* cstr = str.c_str();  // string → char*

char buffer[] = "World";
std::string str2(buffer);        // char[] → string
```

### 2. `char[]` ↔ `char*`

```cpp
char arr[] = "Test";
char* p = arr;  // 指针指向数组首地址
```

---

## ✅ 总结对比表

| 类型            | 本质     | 是否自动管理内存 | 是否可变     | 用途          |
| ------------- | ------ | -------- | -------- | ----------- |
| `std::string` | 类（对象）  | ✅ 是      | ✅ 是      | 推荐，用于一般开发   |
| `char[]`      | 静态字符数组 | ❌ 否      | ✅ 是      | 适用于嵌入式/C 接口 |
| `char*`       | 指针     | ❌ 否      | ❌/✅（看具体） | 系统底层/C 接口   |

---

## 🚀 实战建议

| 需求场景                             | 推荐用法               |
| -------------------------------- | ------------------ |
| 一般字符串处理                          | `std::string` ✅✅✅  |
| 接口要求 C 风格字符串（如 `printf`, socket） | `char*` / `char[]` |
| 固定大小、嵌入式、堆栈开销小                   | `char[]`           |
| 动态拼接字符串、性能稳定、安全性高                | `std::string`      |

---
