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



### const/static对比

**static**

不考虑类的情况：

- 隐藏：所有不加static的全局变量和函数具有全局可见性，可以在其他文件中使用，加了之后只能在该文件所在的编译模块中使用；
- 默认初始化为0， 包括未初始化的全局静态变量与局部静态变量，都存在全局未初始化区；
- 静态变量在函数内定义，始终存在，且只进行一次初始化，具有记忆性，其作用范围与局部变量相同，函数退出后仍然存在，但不使用；

考虑类的情况：

- static成员变量：只与类关联，不与类的对象关联。定义时要分配空间，不能在类声明中初始化，必须在类定义体外部初始化，初始化时不需要标示为static， 可以被非static成员函数任意访问；
- static成员函数：不具有this指针，无法访问类对象和非static成员变量和非static成员函数，不能被声明为const, 虚函数和volatile；可以被非static成员函数任意访问；

**const**

不考虑类的情况：

- const常量在定义时必须初始化，之后无法更改；
- const形参可以接收const和非const类型的实参
  
考虑类的情况：
- const成员变量：不能在类定义外部初始化，只能通过构造函数初始化列表进行初始化，并且必须有构造函数；不同类对其const数据成员的值可以不同，所有不能在类中声明时初始化；
- const成员函数：const对象不可以调用非const成员函数；非const对象都可以调用；不可以改变非mutable数据的值

