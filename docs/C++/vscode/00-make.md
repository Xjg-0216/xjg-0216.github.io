# 零 CMake&make&Makefile


## 🔧 1. make

* `make` 是一个 **自动化构建工具**。
* 它读取名为 **Makefile** 的文件，根据里面的规则来：

  * 编译源代码 `.cpp`
  * 链接成可执行文件 `.out`

### ✅ 举例：

```bash
make
```

会自动查找当前目录下的 `Makefile` 文件，并执行里面的规则。

---

## 📄 2. Makefile

* `Makefile` 是一个文本文件，里面写着 “怎么编译、链接程序的步骤”。

### ✅ 一个最简单的 Makefile：

```make
main: main.o math.o
	g++ -o main main.o math.o

main.o: main.cpp
	g++ -c main.cpp

math.o: math.cpp
	g++ -c math.cpp

clean:
	rm -f *.o main
```

你可以在终端运行：

```bash
make        # 默认构建 main
make clean  # 清理临时文件
```

---

## 🏗️ 3. CMake

* `CMake` 是一个 **跨平台的自动化构建系统生成器**。
* 它会读取一个叫 `CMakeLists.txt` 的配置文件，**自动生成 Makefile 或其他构建系统的工程文件（如 Visual Studio 工程）**。
* 你只需要维护一套配置（`CMakeLists.txt`），CMake 会帮你适配不同系统、不同编译器。

### ✅ 示例 `CMakeLists.txt`：

```cmake
cmake_minimum_required(VERSION 3.10)
project(MyApp)

set(CMAKE_CXX_STANDARD 17)

add_executable(main main.cpp math.cpp)
```

### ✅ 使用流程：

```bash
mkdir build
cd build
cmake ..       # 生成 Makefile
make           # 编译
```

---

## ✅ 4. 总结：三者区别与联系

| 名称         | 类型 | 作用                       |
| ---------- | -- | ------------------------ |
| `make`     | 工具 | 读取 Makefile，执行编译         |
| `Makefile` | 文件 | 写明编译规则                   |
| `CMake`    | 工具 | 自动生成 Makefile（或其他构建系统工程） |

---

## 🚀 图示理解：

```
你写的 CMakeLists.txt
         ↓
     ┌────────┐
     │ CMake  │ ← 负责读取配置
     └────────┘
         ↓ 生成
      Makefile（或 Ninja 等）
         ↓
     ┌────────┐
     │ make   │ ← 负责执行编译动作
     └────────┘
         ↓
     编译后的程序
```

