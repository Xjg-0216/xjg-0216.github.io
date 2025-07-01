
# 二 类-继承


C++ 中的 **继承（Inheritance）** 是面向对象编程的三大特性之一（封装、继承、多态）。它允许通过已有的类派生出新类，从而实现代码复用和体系结构的扩展。



## 1. 父类与子类

```cpp
class Animal {   // 父类
public:
    void eat() {
        cout << "Animal is eating" << endl;
    }
};

class Dog : public Animal {  // 子类
public:
    void bark() {
        cout << "Dog is barking" << endl;
    }
};
```

`Dog` 继承了 `Animal`，可以直接使用 `Animal` 的 `eat()` 方法：

```cpp
Dog d;
d.eat();   // 继承来的方法
d.bark();  // 自己的功能
```

---

## 2. 继承的类型（访问控制）

继承的语法：`class 子类 : 继承方式  父类`

### 2.1 继承方式



**继承方式一共有三种：**

* 公共继承
* 保护继承
* 私有继承


```cpp
class A {
public:    int x;
protected: int y;
private:   int z;
};

class B1 : public A {  // 公有继承
    // x 是 public, y 是 protected, z 不可访问
};

class B2 : protected A {  // 保护继承
    // x 是 protected, y 是 protected, z 不可访问
};

class B3 : private A {  // 私有继承
    // x 是 private, y 是 private, z 不可访问
};
```

| 成员权限        | public继承    | protected继承 | private继承 |
| ----------- | ----------- | ----------- | --------- |
| public成员    | 变 public    | 变 protected | 变 private |
| protected成员 | 变 protected | 变 protected | 变 private |
| private成员   | 不可访问        | 不可访问        | 不可访问      |

**示例：**

```C++
class Base1
{
public: 
	int m_A;
protected:
	int m_B;
private:
	int m_C;
};

//公共继承
class Son1 :public Base1
{
public:
	void func()
	{
		m_A; //可访问 public权限
		m_B; //可访问 protected权限
		//m_C; //不可访问
	}
};

void myClass()
{
	Son1 s1;
	s1.m_A; //其他类只能访问到公共权限
}

//保护继承
class Base2
{
public:
	int m_A;
protected:
	int m_B;
private:
	int m_C;
};
class Son2:protected Base2
{
public:
	void func()
	{
		m_A; //可访问 protected权限
		m_B; //可访问 protected权限
		//m_C; //不可访问
	}
};
void myClass2()
{
	Son2 s;
	//s.m_A; //不可访问
}

//私有继承
class Base3
{
public:
	int m_A;
protected:
	int m_B;
private:
	int m_C;
};
class Son3:private Base3
{
public:
	void func()
	{
		m_A; //可访问 private权限
		m_B; //可访问 private权限
		//m_C; //不可访问
	}
};
class GrandSon3 :public Son3
{
public:
	void func()
	{
		//Son3是私有继承，所以继承Son3的属性在GrandSon3中都无法访问到
		//m_A;
		//m_B;
		//m_C;
	}
};
```

### 2.2 继承中的对象模型

**问题:** 从父类继承过来的成员，哪些属于子类对象中？

**示例：**

```C++
class Base
{
public:
	int m_A;
protected:
	int m_B;
private:
	int m_C; //私有成员只是被隐藏了，但是还是会继承下去
};

//公共继承
class Son :public Base
{
public:
	int m_D;
};

void test01()
{
	cout << "sizeof Son = " << sizeof(Son) << endl; // 16
}

int main() {

	test01();

	system("pause");

	return 0;
}
```

私有成员 m_C 虽然在派生类中被封装（不可直接访问），但依旧是父类子对象的一部分，所以它的大小也计入 Son 的总体大小。

---

## 3. 继承的构造和析构

子类继承父类后，当创建子类对象，也会调用父类的构造函数


子类对象创建时：

* 先调用父类构造函数
* 然后调用子类构造函数
* 析构顺序与构造相反

```cpp
class A {
public:
    A() { cout << "A构造\n"; }
    ~A() { cout << "A析构\n"; }
};

class B : public A {
public:
    B() { cout << "B构造\n"; }
    ~B() { cout << "B析构\n"; }
};

int main() {
    B b;
}
```

**输出：**

```
A构造
B构造
B析构
A析构
```

---

## 4. 继承与同名覆盖

### 4.1 继承同名成员处理方式


>问题：当子类与父类出现同名的成员，如何通过子类对象，访问到子类或父类中同名的数据呢？


* 访问子类同名成员   直接访问即可
* 访问父类同名成员   需要加作用域

**示例：**

```C++
class Base {
public:
	Base()
	{
		m_A = 100;
	}

	void func()
	{
		cout << "Base - func()调用" << endl;
	}

	void func(int a)
	{
		cout << "Base - func(int a)调用" << endl;
	}

public:
	int m_A;
};


class Son : public Base {
public:
	Son()
	{
		m_A = 200;
	}

	//当子类与父类拥有同名的成员函数，子类会隐藏父类中所有版本的同名成员函数
	//如果想访问父类中被隐藏的同名成员函数，需要加父类的作用域
	void func()
	{
		cout << "Son - func()调用" << endl;
	}
public:
	int m_A;
};

void test01()
{
	Son s;

	cout << "Son下的m_A = " << s.m_A << endl;
	cout << "Base下的m_A = " << s.Base::m_A << endl;

	s.func();
	s.Base::func();
	s.Base::func(10);

}
int main() {

	test01();

	system("pause");
	return EXIT_SUCCESS;
}
```

总结：

1. 子类对象可以直接访问到子类中同名成员
2. 子类对象加作用域可以访问到父类同名成员
3. 当子类与父类拥有同名的成员函数，子类会隐藏父类中同名成员函数，加作用域可以访问到父类中同名函数


### 4.2 继承同名静态成员处理方式

>问题：继承中同名的静态成员在子类对象上如何进行访问？



静态成员和非静态成员出现同名，处理方式一致



- 访问子类同名成员   直接访问即可
- 访问父类同名成员   需要加作用域

**示例：**

```C++
class Base {
public:
	static void func()
	{
		cout << "Base - static void func()" << endl;
	}
	static void func(int a)
	{
		cout << "Base - static void func(int a)" << endl;
	}

	static int m_A;
};

int Base::m_A = 100;

class Son : public Base {
public:
	static void func()
	{
		cout << "Son - static void func()" << endl;
	}
	static int m_A;
};

int Son::m_A = 200;

//同名成员属性
void test01()
{
	//通过对象访问
	cout << "通过对象访问： " << endl;
	Son s;
	cout << "Son  下 m_A = " << s.m_A << endl;
	cout << "Base 下 m_A = " << s.Base::m_A << endl;

	//通过类名访问
	cout << "通过类名访问： " << endl;
	cout << "Son  下 m_A = " << Son::m_A << endl;
	cout << "Base 下 m_A = " << Son::Base::m_A << endl;
}

//同名成员函数
void test02()
{
	//通过对象访问
	cout << "通过对象访问： " << endl;
	Son s;
	s.func();
	s.Base::func();

	cout << "通过类名访问： " << endl;
	Son::func();
	Son::Base::func();
	//出现同名，子类会隐藏掉父类中所有同名成员函数，需要加作作用域访问
	Son::Base::func(100);
}
int main() {

	//test01();
	test02();

	system("pause");

	return 0;
}
```

> 总结：同名静态成员处理方式和非静态处理方式一样，只不过有两种访问的方式（通过对象 和 通过类名）

---

## 5. 多继承（不推荐过度使用）


C++允许**一个类继承多个类**



语法：` class 子类 ：继承方式 父类1 ， 继承方式 父类2...`



多继承可能会引发父类中有同名成员出现，需要加作用域区分



**C++实际开发中不建议用多继承**





**示例：**

```C++
class Base1 {
public:
	Base1()
	{
		m_A = 100;
	}
public:
	int m_A;
};

class Base2 {
public:
	Base2()
	{
		m_A = 200;  //开始是m_B 不会出问题，但是改为mA就会出现不明确
	}
public:
	int m_A;
};

//语法：class 子类：继承方式 父类1 ，继承方式 父类2 
class Son : public Base2, public Base1 
{
public:
	Son()
	{
		m_C = 300;
		m_D = 400;
	}
public:
	int m_C;
	int m_D;
};


//多继承容易产生成员同名的情况
//通过使用类名作用域可以区分调用哪一个基类的成员
void test01()
{
	Son s;
	cout << "sizeof Son = " << sizeof(s) << endl;
	cout << s.Base1::m_A << endl;
	cout << s.Base2::m_A << endl;
}

int main() {

	test01();

	system("pause");

	return 0;
}
```



> 总结： 多继承中如果父类中出现了同名情况，子类使用时候要加作用域

---

## 6. 虚继承（解决菱形继承问题）


**菱形继承概念：**

- 两个子类继承同一个父类
- 又有某个类同时继承者两个子类
- 这种继承被称为菱形继承，或者钻石继承




**示例：**

```C++
class Animal
{
public:
	int m_Age;
};

//继承前加virtual关键字后，变为虚继承
//此时公共的父类Animal称为虚基类
class Sheep : virtual public Animal {};
class Tuo   : virtual public Animal {};
class SheepTuo : public Sheep, public Tuo {};

void test01()
{
	SheepTuo st;
	st.Sheep::m_Age = 100;
	st.Tuo::m_Age = 200;

	cout << "st.Sheep::m_Age = " << st.Sheep::m_Age << endl;
	cout << "st.Tuo::m_Age = " <<  st.Tuo::m_Age << endl;
	cout << "st.m_Age = " << st.m_Age << endl;
}


int main() {

	test01();

	system("pause");

	return 0;
}
```

总结：

* 菱形继承带来的主要问题是子类继承两份相同的数据，导致资源浪费以及毫无意义
* 利用虚继承可以解决菱形继承问题
