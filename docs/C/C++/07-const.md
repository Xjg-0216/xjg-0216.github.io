# 七 const

`const` 是 C++ 中非常重要的关键字，用来表示“常量”或“不允许修改”。它可以用于变量、指针、函数参数、函数返回值、成员函数等，帮助提升**程序的安全性、可读性**，并支持**编译期错误检查**。

---

## 1.const 的基本作用

```cpp
const int a = 10;
a = 20;  // ❌ 错误，a 是常量不能修改
```

---

## 2. 用法分类汇总

| 用法       | 示例                     | 含义                    |
| -------- | ---------------------- | --------------------- |
| 常量变量     | `const int a = 5;`     | 不能修改的变量               |
| 常量指针     | `int* const p = &a;`   | 指针指向不能变               |
| 指向常量的指针  | `const int* p = &a;`   | 指针指向的值不能改            |
| 常量引用     | `void f(const int& x)` | 函数内部不能修改 x            |
| 常量成员函数   | `void show() const;`   | 保证成员函数不修改对象           |
| const类对象 | `const MyClass obj;`   | 对象不可被修改，只能调用 const 函数 |

---

## 3.指针中的 const

### 3.1 `const int* p`

> 指向 **常量** 的指针：不能通过 `*p` 修改值，但 `p` 本身可以指向别处。

```cpp
int a = 10;
int b = 20;
const int* p = &a;

*p = 5;     // ❌ 错
p = &b;     // ✅ 可修改指向
```

---

### 3.2 `int* const p`

> **常量指针**：不能修改指向，但可以修改所指内容。

```cpp
int a = 10;
int* const p = &a;

*p = 20;    // ✅
p = &b;     // ❌ 错，不能改指向
```

---

### 3.3 `const int* const p`

> 指向常量的常量指针：什么都不能改。

```cpp
const int* const p = &a;

*p = 100;   // ❌
p = &b;     // ❌
```

---

## 4. 函数参数中的 const

### 4.1 值传递 + const（没啥用）

```cpp
void f(const int x) {
    // x是拷贝来的，原本就不能修改原变量
}
```

### 4.2 引用传递 + const（很常用）

```cpp
void printName(const std::string& name) {
    std::cout << name << std::endl;
}
```

* 既避免拷贝，又保证函数内部不改内容
* 高效安全，是推荐方式

---

## 5. 成员函数中的 const

用于**类的成员函数**后面，表示该函数不修改对象的成员变量（除了 `mutable` 修饰的除外）：

```cpp
class Student {
    std::string name;
public:
    std::string getName() const {
        return name;  // ✅ 只能访问 const 成员
    }

    void setName(std::string newName) {
        name = newName;
    }
};
```

> 只能用 const 对象调用 const 成员函数。

```cpp
const Student s;
s.getName();  // ✅
s.setName("Tom"); // ❌ 报错
```

---

## 6. 常量对象 / 常量成员

```cpp
const Student s;  // 整个对象是只读的，常对象只能调用常变量
```

---

## 7. 总结

| 写法                      | 含义           |
| ----------------------- | ------------ |
| `const int x`           | 变量不能改        |
| `int* const p`          | 指针不能改        |
| `const int* p`          | 指针指向的值不能改    |
| `const int* const p`    | 都不能改         |
| `void func(const T& t)` | 不修改传进来的参数    |
| `void method() const`   | 不修改当前对象的任何成员 |

