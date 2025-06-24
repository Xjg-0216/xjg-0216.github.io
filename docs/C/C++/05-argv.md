# 五 命令行参数
在 C 和 C++ 中，`int main(int argc, char **argv)` 是程序主函数的一种标准签名，用于接收命令行参数。。

---

### 1. `main` 函数的参数

#### 1.1 `main` 函数的基本形式
- C 和 C++ 标准允许 `main` 函数有几种签名，其中最常用的是：
  ```c
  int main(int argc, char **argv)
  ```
- **`argc`**：表示命令行参数的数量（argument count），包括程序名本身。
- **`argv`**：表示命令行参数的内容（argument vector），是一个字符串数组。

#### 1.2 命令行参数示例
假设你在终端运行程序：
```bash
./myprogram hello world 123
```
- `argc` = 4（4 个参数：`myprogram`, `hello`, `world`, `123`）。
- `argv` 是一个数组，包含指向这些字符串的指针：
  - `argv[0]` = `"myprogram"`（程序名）。
  - `argv[1]` = `"hello"`。
  - `argv[2]` = `"world"`。
  - `argv[3]` = `"123"`。
  - `argv[4]` = `NULL`（标记数组结束）。

---

### 2. 为什么用 `char**`？

- **`char**` 是指向指针的指针**：
  - `char*`：指向一个字符数组（字符串）。
  - `char**`：指向一个指针数组，而这些指针又指向字符串。
- **内存视图**：
  - `argv` 是一个指针，指向一个连续的 `char*` 数组。
  - 每个 `char*` 指向内存中某个字符串的起始地址。


### 3. 简单示例
```c
#include <stdio.h>
int main(int argc, char **argv) {
    printf("Argument count: %d\n", argc);
    for (int i = 0; i < argc; i++) {
        printf("argv[%d]: %s\n", i, argv[i]);
    }
    return 0;
}
```
- 运行：`./program hello world`
- 输出：
  ```
  Argument count: 3
  argv[0]: ./program
  argv[1]: hello
  argv[2]: world
  ```

**检查 `argv` 的结束**
- `argv[argc]` 总是 `NULL`，可以用作循环终止条件：
```c
#include <stdio.h>
int main(int argc, char **argv) {
    int i = 0;
    while (argv[i] != NULL) {
        printf("argv[%d]: %s\n", i, argv[i]);
        i++;
    }
    return 0;
}
```

**等价写法：`char *argv[]`**
- `char **argv` 和 `char *argv[]` 是等价的：
  - `char *argv[]` 是数组形式，更直观地表示“指针数组”。
  - 在函数参数中，数组会退化为指针，因此 `char *argv[]` 等同于 `char **argv`。
```c
int main(int argc, char *argv[]) {
    printf("argv[0]: %s\n", argv[0]);
    return 0;
}
```


### 4. argv中字符串转换为其他类型
```cpp
#include <iostream>
#include <cstdlib>  // for atoi, atof

int main(int argc, char **argv)
{
  if (argc < 3)
  {
    std::cout << "Please provide at least two arguments" >> std::endl;
  }

  // 获取第一个参数并转换为整数
  int int_value = std::atoi(argv[1]);
  // 获取第二个参数并转换为浮点数
  float float_value = std::atof(argv[2]);

  std::cout << "Integer value: " << int_value << std::endl;
  std::cout << "Float value: " << float_value << std::endl;
}

```




