C++ 中的结构体（`struct`）是用于将**不同类型的变量组合成一个整体**的数据结构，适合描述一类“实体”的多个属性。它既继承了 C 的用法，又被扩展得和 `class` 类似，拥有构造函数、成员函数、访问控制等特性。

---

## ✅ 一、基本结构体定义和使用

```cpp
struct Student {
    std::string name;
    int age;
    float score;
};

int main() {
    Student s1;
    s1.name = "Alice";
    s1.age = 20;
    s1.score = 89.5;

    std::cout << s1.name << ", " << s1.age << ", " << s1.score << std::endl;
}
```

> 结构体对象通过`.`访问其成员。

---

## ✅ 二、带构造函数的结构体（C++ 扩展）

```cpp
struct Student {
    std::string name;
    int age;
    float score;

    // 构造函数
    Student(std::string n, int a, float s)
        : name(n), age(a), score(s) {}
};
```

```cpp
Student s("Bob", 21, 91.0);
```

---

## ✅ 三、结构体数组

```cpp
Student class1[3] = {
    {"Tom", 20, 88},
    {"Jerry", 19, 92},
    {"Lucy", 21, 85}
};
```

---

## ✅ 四、结构体指针

```cpp
Student s = {"Mike", 22, 90};
Student* p = &s;

std::cout << p->name << std::endl;  // 使用 -> 访问
```

---

## ✅ 五、结构体作为函数参数

### 1. 按值传递（会拷贝一份）

```cpp
void printStudent(Student s) {
    std::cout << s.name << std::endl;
}
```

### 2. 按引用传递（推荐）

```cpp
void printStudent(const Student& s) {
    std::cout << s.name << std::endl;
}
```

---

## ✅ 六、结构体嵌套

```cpp
struct Date {
    int year, month, day;
};

struct Student {
    std::string name;
    Date birthday;
};

Student s = {"Anna", {2003, 5, 18}};
std::cout << s.birthday.year << std::endl;
```

---

## ✅ 七、结构体默认成员初始化（C++11 起）

```cpp
struct Student {
    std::string name = "Unknown";
    int age = 0;
    float score = 0.0;
};
```

---

## ✅ 八、结构体与 class 区别（重点）

| 特性      | `struct` | `class`   |
| ------- | -------- | --------- |
| 默认访问权限  | `public` | `private` |
| 支持成员函数  | ✅        | ✅         |
| 支持继承/多态 | ✅（C++）   | ✅         |
| 用途倾向    | 数据结构     | 面向对象编程    |

### 示例对比

```cpp
struct A {
    int x;  // 默认 public
};

class B {
    int x;  // 默认 private
};
```

---

## ✅ 九、结构体与 typedef / using 结合（类型别名）

```cpp
struct Student {
    std::string name;
    int age;
};
typedef struct Student Stu;

using Stu2 = Student;
```

---

## ✅ 十、现代结构体（C++11\~20 新特性）

* 列表初始化：`Student s{"Tom", 18, 95.0};`
* 支持移动构造、复制构造、=default、=delete
* 支持 `constexpr` 成员函数（C++14 起）

---

## 📌 总结要点

* `struct` 适合做纯数据结构
* 支持构造函数、成员函数、运算符重载（C++）
* 建议用引用传递提高效率
* 区别在默认访问权限（`struct` 是 public）

---

如果你想了解更深层内容，例如：

* 如何给结构体重载运算符（如 `<`、`==`）
* 如何定义结构体比较规则以用于 `std::map` 或 `std::set`
* `packed` 结构体对齐和内存布局问题

