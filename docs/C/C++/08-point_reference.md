# 八 指针与引用

>在 C++ 中，\*\*指针（Pointer）**和**引用（Reference）\*\*是两个用于间接访问变量的机制，它们有一些相似之处，但也有本质区别

---

## 1. 联系

- **都可以间接访问变量**：你可以通过指针或引用来访问和修改原始变量的值。
- **都支持传递变量的地址**：常用于函数参数传递，避免复制，支持修改原始变量。

---

## 2. 区别

| 特性              | 指针 (Pointer)      | 引用 (Reference) |
| --------------- | ----------------- | -------------- |
| 定义时是否必须初始化      | 否                 | 是（必须绑定一个变量）    |
| 是否可以为 `nullptr` | 可以                | 不可以            |
| 是否可以重新绑定        | 可以指向别的变量          | 不可以更改引用绑定的变量   |
| 语法是否需要解引用操作     | 需要用 `*` 解引用       | 不需要，像普通变量一样使用  |
| 支持运算            | 指针可以做算术运算         | 引用不支持算术运算      |
| 主要用途            | 动态内存管理、数组遍历、函数指针等 | 参数传递、别名、简化语法   |

---

## 3. 示例对比

### 3.1 指针示例：

```cpp
#include <iostream>
using namespace std;

int main() {
    int a = 10;
    int* p = &a;  // 定义指针并指向 a

    cout << "a = " << a << endl;
    cout << "*p = " << *p << endl;

    *p = 20;  // 修改指针指向的变量值
    cout << "a after modification = " << a << endl;

    return 0;
}
```

🟢 输出：

```
a = 10
*p = 10
a after modification = 20
```

---

### 3.2 引用示例：

```cpp
#include <iostream>
using namespace std;

int main() {
    int a = 10;
    int& ref = a;  // 定义引用，必须立即初始化

    cout << "a = " << a << endl;
    cout << "ref = " << ref << endl;

    ref = 30;  // 修改引用所绑定的变量值
    cout << "a after modification = " << a << endl;

    return 0;
}
```

🟢 输出：

```
a = 10
ref = 10
a after modification = 30
```

---

### 3.3 函数传参中的区别

```cpp
void modifyByPointer(int* p) {
    *p = 100;
}

void modifyByReference(int& r) {
    r = 200;
}

int main() {
    int x = 10;
    modifyByPointer(&x);   // 传地址
    cout << "x after pointer: " << x << endl;

    modifyByReference(x);  // 传引用
    cout << "x after reference: " << x << endl;

    return 0;
}
```

🟢 输出：

```
x after pointer: 100
x after reference: 200
```

---

## 4 总结

| 使用场景                  | 推荐方式          |
| --------------------- | ------------- |
| 修改函数外部变量值             | 引用（更安全，语法更简洁） |
| 动态内存分配、链表等需要可空或重新绑定指向 | 指针            |
| 提高可读性和函数接口安全性         | 引用            |

---
