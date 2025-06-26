

# 九 类型修饰符


>类型修饰符用于改变变量或函数的基本类型的含义以及四种命名的类型转换操作符


## 1. `const`

`const` 是最常用的修饰符之一，它表示一个对象是常量，其值在初始化后不能被修改。具体可参考[const](docs/C/C++/07-const.md)

- **修饰变量**: 变量的值不可更改。
    
    ```cpp
    const int MAX_SIZE = 100;
    // MAX_SIZE = 200; // 编译错误！
    ```
    
- **修饰指针**:
    
    - **指向常量的指针 (Pointer to const)**: 指针指向的内容不可通过该指针修改，但指针本身可以指向其他地址
        
        ```cpp
        const int a = 10;
        const int* ptr = &a;
        // *ptr = 20; // 编译错误！
        int b = 20;
        ptr = &b; // 正确，指针可以指向别处
        ```
        
    - **常量指针 (Const pointer)**: 指针本身是常量，必须在声明时初始化，之后不能再指向其他地址，但它指向的内容可以通过指针修改（前提是内容本身不是常量）。
        
        ```cpp
        int a = 10;
        int* const ptr = &a;
        *ptr = 20; // 正确，可以修改指向的内容
        // int b = 30;
        // ptr = &b; // 编译错误！指针不能指向别处
        ```
        
    - **指向常量的常量指针 (Const pointer to const)**: 指针本身和它指向的内容都不能修改。
        
        ```cpp
        const int a = 10;
        const int* const ptr = &a;
        // *ptr = 20;    // 编译错误！
        // int b = 30;
        // ptr = &b;    // 编译错误！
        ```
        
- **修饰成员函数**: `const` 成员函数承诺不会修改类的任何成员变量（除非该成员被 `mutable` 修饰）。
    ```cpp
    class MyClass {
    private:
        int value;
    public:
        int getValue() const {
            // value = 10; // 编译错误！不能在 const 成员函数中修改成员变量
            return value;
        }
    };
    ```
    

---

## 2. `volatile` (易变的)

`volatile` 告诉编译器，变量的值可能会在程序无法检测到的情况下被改变（例如，由操作系统、硬件或其他线程）。因此，编译器每次访问该变量时都必须从内存中重新读取，而不是使用缓存的寄存器中的值。

- **常见应用**:
    
    - 多线程环境中共享的变量。
    - 与硬件交互的内存映射 I/O。
    - 中断服务程序中修改的全局变量。
    
    ```cpp
    volatile int sharedValue = 0;
    
    // 线程 A
    void writer() {
        sharedValue = 123;
    }
    
    // 线程 B
    void reader() {
        while (sharedValue == 0) {
            // 等待...
            // volatile 确保每次循环都会从内存重新读取 sharedValue
        }
    }
    ```
    

---

## 3. `mutable` (可变的)

`mutable` 允许一个 `const` 成员函数修改类的某个成员变量。这在需要保持对象的逻辑常量性，但又需要修改某些内部状态时非常有用。

- **示例**: 缓存计算结果。
    
    ```cpp
    class MyData {
    private:
        mutable int cachedValue; // 即使在 const 函数中也能修改
        bool cacheValid;
        int data;
    
    public:
        MyData(int d) : data(d), cacheValid(false), cachedValue(0) {}
    
        int getExpensiveComputation() const {
            if (cacheValid) {
                return cachedValue; // 直接返回缓存
            }
            // ... 进行一些复杂的计算 ...
            cachedValue = data * 10; // 正确！可以修改 mutable 成员
            // cacheValid = true;  // 编译错误！cacheValid 不是 mutable
            return cachedValue;
        }
    };
    ```
    
    **注意**: 在上面的例子中，`cacheValid` 也应该是 `mutable` 的，否则也无法在 `const` 函数中修改。

---

## 4. `static` (静态的)

`static` 修饰符的含义取决于它的使用上下文。

- **在函数内部**: 变量只会被初始化一次，并在程序的整个生命周期内存在。它保留了上次函数调用结束时的值。
    
    ```cpp
    void counter() {
        static int count = 0; // 只在第一次调用时初始化
        std::cout << "Called " << ++count << " times." << std::endl;
    }
    ```
    
- **在类中**:
    
    - **静态成员变量**: 所有类的实例共享同一个静态成员变量。它必须在**类外进行初始化**。
    - **静态成员函数**: 可以在没有创建类实例的情况下调用。它只能访问静态成员变量和静态成员函数。
    
    ```cpp
    class Car {
    public:
        static int totalCars; // 声明静态成员变量
        Car() { totalCars++; }
        static int getTotalCars() { // 静态成员函数
            return totalCars;
        }
    };
    
    int Car::totalCars = 0; // 在类外初始化静态成员变量
    
    int main() {
        Car c1, c2;
        std::cout << Car::getTotalCars() << std::endl; // 输出 2
    }
    ```
    