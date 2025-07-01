

# 十 类型转换

C++ 提供了四种命名的类型转换操作符


## 1. `static_cast<T>(value)`

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
    


## 2. `dynamic_cast<T>(value)`

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

## 3. `const_cast<T>(value)`

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
    


## 4. `reinterpret_cast<T>(value)`

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

## 5. 总结

|操作符|用途|安全性|推荐用途|
|---|---|---|---|
|`static_cast`|常规转换|✅ 安全（编译期检查）|基本类型、类层次结构转换|
|`dynamic_cast`|RTTI 转换（多态）|✅ 运行时检查|父类指针 → 子类指针|
|`const_cast`|添加/移除 const/volatile|⚠️ 稍不当会出错|去掉 const 修饰|
|`reinterpret_cast`|重新解释内存|❌ 危险|指针、底层位级转换等|
