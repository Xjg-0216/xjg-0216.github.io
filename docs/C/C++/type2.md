

# 类型修饰符及类型转换


>类型修饰符用于改变变量或函数的基本类型的含义以及四种命名的类型转换操作符


#### 1. `const` (常量)

`const` 是最常用的修饰符之一，它表示一个对象是常量，其值在初始化后不能被修改。

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

#### 2. `volatile` (易变的)

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

#### 3. `mutable` (可变的)

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

#### 4. `static` (静态的)

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
    

---

### 类型转换

C++ 提供了四种命名的类型转换操作符

---

#### 1. `static_cast`

这是最常见的转换，用于在相关的类型之间进行转换，例如数字类型之间、指针类型之间（向上或向下转型）。它在编译时进行类型检查，但不提供运行时检查。

- **应用**:
    
    - 基本数据类型转换。
    - 类层次结构中的向上转型（子类指针转为父类指针，总是安全的）。
    - 类层次结构中的向下转型（父类指针转为子类指针，不安全，需要程序员保证转换的正确性）。
    
    ```cpp
    double d = 3.14;
    int i = static_cast<int>(d); // double -> int
    
    class Base {};
    class Derived : public Base {};
    
    Derived d_obj;
    Base* b_ptr = &d_obj;
    
    // 向上转型 (Upcasting) - 隐式转换通常也可以，但显式更清晰
    Base* b_ptr2 = static_cast<Base*>(&d_obj);
    
    // 向下转型 (Downcasting) - 不安全！
    // 只有当 b_ptr 确定指向一个 Derived 对象时才安全
    Derived* d_ptr = static_cast<Derived*>(b_ptr);
    ```
    

---

#### 2. `dynamic_cast`

专门用于处理多态（即含有虚函数的类）的类层次结构中的向下转型。它在运行时进行类型检查，如果转换不安全（即父类指针并非指向所期望的子类对象），它会返回 `nullptr`（对于指针）或抛出 `std::bad_cast` 异常（对于引用）。

- **应用**: 安全地将父类指针/引用转换为子类指针/引用。
    
    ```cpp
    class Base { public: virtual ~Base() {} }; // 必须有虚函数
    class Derived : public Base {};
    class Another : public Base {};
    
    Base* b_ptr = new Derived();
    Base* b_ptr2 = new Another();
    
    // 转换成功，因为 b_ptr 确实指向 Derived 对象
    Derived* d_ptr = dynamic_cast<Derived*>(b_ptr);
    if (d_ptr) {
        std::cout << "Cast to Derived successful." << std::endl;
    }
    
    // 转换失败，返回 nullptr，因为 b_ptr2 指向 Another 对象
    Derived* d_ptr2 = dynamic_cast<Derived*>(b_ptr2);
    if (!d_ptr2) {
        std::cout << "Cast to Derived failed." << std::endl;
    }
    
    delete b_ptr;
    delete b_ptr2;
    ```
    

---

#### 3. `const_cast`

用于添加或移除变量的 `const` 或 `volatile` 属性。这是唯一能做到这一点的 C++ 转换。滥用它可能会导致未定义行为。

- **应用**: 当你需要调用一个没有 `const` 重载的函数，但你确定该函数不会修改对象时。
    
    ```cpp
    void print(char* str) {
        std::cout << str << std::endl;
    }
    
    int main() {
        const char* my_str = "Hello, World!";
        // print(my_str); // 编译错误！不能将 const char* 传给 char*
        print(const_cast<char*>(my_str)); // 正确，但要保证 print 函数不会修改 str
    }
    ```
    
    **警告**: 如果你用 `const_cast` 去掉了 `const` 属性，然后试图修改一个本身被定义为 `const` 的对象，结果是未定义的。
    

---

#### 4. `reinterpret_cast`

这是最危险的转换，它允许任意指针类型之间的转换，甚至可以将指针转换为整数，反之亦然。它基本上只是重新解释内存中的位模式，不进行任何类型检查。

- **应用**: 底层代码、与硬件交互、或者一些非常规的转换。应极力避免使用。
    
    ```cpp
    int i = 0x12345678;
    int* p_i = &i;
    
    // 将 int 指针转换为 char 指针
    char* p_c = reinterpret_cast<char*>(p_i);
    
    // 在小端机器上，p_c[0] 的值会是 0x78
    std::cout << std::hex << static_cast<int>(p_c[0]) << std::endl;
    
    // 将地址转换为整数
    uintptr_t addr = reinterpret_cast<uintptr_t>(p_i);
    std::cout << "Address: " << addr << std::endl;
    ```
    