# lambda函数

Lambda 表达式是 C++11 引入的一种 轻量级的匿名函数。把它当成 “写在函数里的函数”，常用于 临时函数逻辑、回调函数、STL算法等。


当然可以，下面是 **C++ Lambda 表达式的基本使用**，适合初学者理解：



## 1. 基本语法

```cpp
[capture](parameter_list) -> return_type {
    // function body
}
```

其中：

| 部分                 | 说明                       |
| ------------------ | ------------------------ |
| `[capture]`        | 捕获外部变量方式                 |
| `(parameter_list)` | 参数列表（如 `(int a, int b)`） |
| `-> return_type`   | 可选，指定返回类型                |
| `{}`               | 函数体                      |


## 2. 基本用法示例

### 2.1 最简单的 Lambda

```cpp
auto say_hello = []() {
    std::cout << "Hello, Lambda!" << std::endl;
};

say_hello();  // 调用
```

### 2.2 带参数和返回值

```cpp
auto add = [](int a, int b) -> int {
    return a + b;
};

std::cout << add(3, 4);  // 输出 7
```

💡 返回值类型可以省略（让编译器推导）：

```cpp
auto add = [](int a, int b) {
    return a + b;
};
```


## 3. 捕获外部变量（`[capture]`）

### 3.1 按值捕获 `[=]`

```cpp
int x = 5;
auto f = [=]() {
    std::cout << x << std::endl;  // 输出 5
};
f();
```

### 3.2 按引用捕获 `[&]`

```cpp
int x = 5;
auto f = [&]() {
    x = 10;
};
f();
std::cout << x << std::endl;  // 输出 10
```

### 3.3 混合捕获

```cpp
int a = 1, b = 2;
auto f = [a, &b]() {
    std::cout << a << ", " << b << std::endl;
};
```


## 4. 常见应用场景

### 4.1 与 STL 算法配合使用

```cpp
std::vector<int> v = {1, 2, 3, 4};

std::for_each(v.begin(), v.end(), [](int x) {
    std::cout << x << " ";
});
```

### 4.2 自定义排序规则

```cpp
std::sort(v.begin(), v.end(), [](int a, int b) {
    return a > b;  // 降序
});
```

### 4.3 多线程函数体

```cpp
#include <thread>

std::thread t([] {
    std::cout << "Hello from thread" << std::endl;
});
t.join();
```

---

## 5. `mutable` 关键字

默认情况下，按值捕获的变量是 **只读** 的。如果你想修改它的副本，需要加上 `mutable`：

```cpp
int x = 10;
auto f = [x]() mutable {
    x += 5;  // 允许修改副本
    std::cout << x << std::endl;
};
f();  // 输出 15
std::cout << x << std::endl;  // 还是 10，原始变量没变
```

---

## 6. 小结

| 特性        | 说明         |
| --------- | ---------- |
| 支持匿名函数    | ✅          |
| 可以捕获外部变量  | ✅（按值/引用）   |
| 可以作为回调使用  | ✅（如线程、排序等） |
| 支持推导返回类型  | ✅          |
| 类似函数对象/闭包 | ✅          |

