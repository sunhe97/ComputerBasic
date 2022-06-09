

version:1.0 writed by He Sun

[TOC]



# 编译过程

## 预编译：处理#开头的预处理指令，包括#define的文本替换工作，#include拷贝包含的文件的代码，处理条件编译指令(#ifdef,#ifndef,#endif)->.i

### *直接ctrl c ctrl v替换*

## 编译优化：确定所有指令是否符合规则，之后翻译成汇编代码->.s

+ 词法分析

+ 语法分析

+ 语义分析

+ 优化后生成相应的汇编代码

## 汇编:将汇编代码转换成目标机器指令，即与原程序等同的目标机器语言代码->.obj .o。由段组成，至少两个段，代码段：主要程序指令，数据段：全局变量或静态数据

## 连接过程：静态链接执行之前链接（拷贝出来链接到一起一个文件，更新困难需要重新编译程序，但效率高），动态链接可执行程序运行时链接（只需要重新编译动态库，程序不用编译，但效率低）

### 连接，前向声明，让链接器去其他.obj去寻找相关定义，如果不是static函数，不调用也会检查内部LNK问题，STATIC 限制函数不让link，不调用即使内部有问题也不报错

### #ifndef问题可以让函数变为static 解决，让每个#include “.h”文件有唯一个函数版本，不与文件外的公用，不会link，inline也可以，直接将代码复制过去



# 内存管理

## C++内存分区(5个)：栈、堆、静态/全局变量区、常量储存区、代码区

## 栈与堆的区别：

+ 栈是系统分配，堆是程序员主动申请，自己释放
+ 存放的内容：栈中存放的是局部变量，函数的参数；堆中存放的内容由程序员控制
+ 堆内存很大，栈内存较小
+ 堆内存地址从小到大，栈内存地址从大到小
+ 栈空间剩余空间大于申请空间则分配成功；堆空间类似于链表，记录空闲节点的地址，找一块大于申请空间的将节点pop，大多数系统中该块空间的首地址存放的是本次分配空间的大小，造成内存碎片

## 内存对齐：x86 CPU读取按4字节读取，不对齐会产生读取int两次以及拼接操作，效率低![image-20220304224639526](Picture\内存对齐.png)

<font color="red">实际测试下面是对的</font>

+ 结构体的大小等于其最大成员的整数倍；
+ 结构体成员的首地址相对于结构体首地址的偏移量是其类型大小的整数倍。比如double型成员的首地址相对于结构体首地址的偏移量应该是8的倍数；

## 数据类型

+ 全局变量：具有全局作用域。包含该文件即可用，不包含该文件用extern，如果在头文件中定义全局变量，当该头文件被多个文件 `include` 时，该头文件中的全局变量就会被定义多次，导致重复定义，因此不能再头文件中定义全局变量

+ 全局静态变量：具有文件作用域，出了该文件看不见

+ 局部变量：{}为作用域

+ 局部静态变量：{}作用域，它只被初始化一次，自从第一次被初始化直到程序运行结束都一直存在，可以被修改，每次调用函数修改，都是在上一次结果上进行修改，出了作用域可以有同名局部变量，也是不同的变量

![image-20220304224912308](Picture\stl容器.png)

# 对象创建限制在栈或堆中

## 限制栈中：类中private重载new delete

## 限制堆中：析构函数变为private，但是继承实现多态的话，派生类无法访问；delete无法从外部调用析构函数。使用static A* （）单例，构造函数、析构变为protected，destory（）

# 内存泄漏：但指针交换时失去对内存空间的掌控

## 封装到类中，在构造的时候申请内存，析构的时候释放内存。类中使用int指针进行引用计数**指针保证浅复制的对象不会多次释放同一内存空间**

```c++
#include <iostream>
 #include <cstring>

 using namespace std;
 class A
 {
 private:
     char *p;
     unsigned int p_size;
     int *p_count; // 计数变量
 public:
     A(unsigned int n = 1) // 在构造函数中申请内存
     {
         p = new char[n];
         p_size = n;
         p_count = new int;
         *p_count = 1;
         cout << "count is : " << *p_count << endl;
     };
     A(const A &temp)
     {
         p = temp.p;
         p_size = temp.p_size;
         p_count = temp.p_count;
         (*p_count)++; // 复制时，计数变量 +1
         cout << "count is : " << *p_count << endl;
     }
     ~A()
     {
         (*p_count)--; // 析构时，计数变量 -1
         cout << "count is : " << *p_count << endl; 

         if (*p_count == 0) // 只有当计数变量为 0 的时候才会释放该块内存空间
         {
             cout << "buf is deleted" << endl;
             if (p != NULL) 
             {
                 delete[] p; // 删除字符数组
                 p = NULL;   // 防止出现野指针
                 if (p_count != NULL)
                 {
                     delete p_count;
                     p_count = NULL;
                 }
             }
         }
     };
     char *GetPointer()
     {
         return p;
     };
 };
 void fun()
 {
     A ex(100);
     char *p = ex.GetPointer();
     strcpy(p, "Test");
     cout << p << endl;

     A ex1 = ex; // 此时计数变量会 +1
     cout << "ex1.p = " << ex1.GetPointer() << endl;
 }
 int main()
 {
     fun();
     return 0;
 }
```

# 智能指针

## unique_ptr:通过move转移占有权

```c++
    unique_ptr<Test> ptest(new Test("123"));
    unique_ptr<Test> ptest2(new Test("456"));
    ptest->print();
    ptest2 = std::move(ptest);//不能直接ptest2 = ptest//转移后ptest == nullptr
    if(ptest == NULL)cout<<"ptest = NULL\n";
    Test* p = ptest2.release();//将ptest2 = nullptr,将独占权给到p
    p->print();
    ptest.reset(p);//将独占权给到ptest，如果ptest不为nullptr，会将它原先指向的内存释放掉，再等于nullptr，如果reset中无参数，即将ptest释放内存 = nullptr
    ptest->print();
    ptest2 = fun(); //这里可以用=，因为重载了赋值函数
    ptest2->print();
    return 0;
```

## shared_ptr : 引用计数，通过指针进行计数,引用计数使用atomic进行，引用计数线程安全，shared_ptr浅拷贝，多个线程可以同时读取一个对象，多个线程进行写一个对象，需要加锁。多个线程进行赋值，初始化会发生悬空指针

![image-20220405012537267](Picture\image-20220405012537267.png)

## weak_ptr:不拥有所有权，用于解决shared_ptr循环引用无法释放内存的问题，要使用lock去变为shared_ptr,在作用域结束后变为weak

# C++11新特性****

+ ## 智能指针

    ```c++
    class Type {
    public:
    	int a = 0;
    };
    
    class Share_Count {
    private:
    	int a;
    public:
    	Share_Count() :a(1) {}
    	~Share_Count() {}
    	void plusone() { ++a; }
    	int minorone() { return --a; }
    	int getCount() { return a; }
    };
    
    template<typename T>
    class my_shared_ptr {
    public:
    	my_shared_ptr(T* _m_ptr = nulllptr) :m_ptr(_m_ptr) {
    		if (_m_ptr) {
    			m_count = new Share_Count();
    		}
    	}
    	~my_shared_ptr() {
    		if (m_ptr && !m_count->minorone()) {
    			delete m_ptr;
    			delete m_count;
    			m_ptr = nullptr;
    			m_count = nullptr;
    
    		}
    	}
    	T* operator->() const{ return m_ptr; }// (p.operator->())->m
    	T& operator*() const { return *m_ptr; }
    	//当用搬移构造而没有提供拷贝构造，那拷贝构造就自动被禁用,unique仅用赋值构造、赋值重载、只能移动构造
    	//my_shared_ptr(T&& _m_ref) noexcept{
    	//	m_ptr = _m_ref.m_ptr;
    	//	_m_ref.m_ptr = nullptr;
    	//}
    
    	//my_shared_ptr& operator=(T&& _m_ref) {
    	//	m_ptr = _m_ref.m_ptr;
    	//	_m_ref.m_ptr = nulllptr;
    	//	return *this;
    	//}
    	my_shared_ptr(const my_shared_ptr& rhs) {
    		m_ptr = rhs.m_ptr;
    		m_count = rhs.m_count;
    		m_count->plusone();
    
    
    	}
    	my_shared_ptr& operator=(const my_shared_ptr& rhs) {
    		m_ptr = rhs.m_ptr;
    		m_count = rhs.m_count;
    		m_count->plusone();
    		return *this;
    	}
    private:
    	T* m_ptr;
    	Share_Count* m_count;
    };
    
    int main() {
    	return 0;
    }
    ```

    

+ ## auto与decltype：都在编译器

    + auto 根据`=`右边的初始值 value 推导出变量的类型，必须初始化

    + decltype 根据 exp 表达式推导出变量的类型，跟`=`右边的 value 没有关系。

    + 如果表达式的类型不是[指针](http://c.biancheng.net/c/80/)或者引用，auto 会把 cv 限定符直接抛弃，推导成 non-const 或者 non-volatile 类型。

    + 如果表达式的类型是指针或者引用，auto 将保留 cv 限定符；decltype一直保留

    + auto 不保留引用；decltype保留引用

        ```c++
            int n = 10;
            int &r1 = n;
            //auto推导
            auto r2 = r1;
            r2 = 20;
            cout << n << ", " << r1 << ", " << r2 << endl;// 10 10 20
            //decltype推导
            decltype(r1) r3 = n;
            r3 = 99;
            cout << n << ", " << r1 << ", " << r3 << endl;//99 99 99
            return 0;
        ```

        

+ ![image-20220310012416440](Picture\image-20220310012416440.png)

    ![image-20220310012249719](Picture\image-20220310012249719.png)

+ ## lambda:匿名类（重载了（）运算符且为const（如果不使mutable））

+ ## 范围for循环：用于序列，这些序列要通过begin，end返回迭代器

+ ## 右值引用

+ ## move：先remove——reference再返回T

+ ## delete 函数和 default 函数。 delete 函数：= delete 表示该函数不能被调用 。default 函数：= default 表示编译器生成默认的函数，对特殊函数（构造、析构、拷贝构造），例如：生成默认的构造函数

+ ## consetexpr

+ nullptr 当NULL（= 0），调用同名函数会引起歧义

# C与C++区别***

+ C面向过程，C++面向对象
+ C主要用于嵌入式与硬件打交道的场景，C++主要用于应用层，与操作系统交互
+ C++ 对 C 的“增强”，表现在以下几个方面：类型检查更为严格。增加了面向对象的机制、泛型编程的机制（Template）、异常处理、运算符重载、标准模板库（STL）、命名空间（避免全局命名冲突）

# 面向对象：对象是指具体的某一个事物，这些事物的抽象就是类，类中包含数据（成员变量）和动作（成员方法）

## 三大特性*********：

+ 封装：将具体的实现过程和数据封装成一个函数，只提供接口进行访问

+ 继承：子类继承父类非private的成员变量与方法，子类可以对父类的方法进行重写，但当父类/父类函数被final修饰，父类不可以被继承/父类方法不可以被重写

+ 多态：不同继承类对象对同一消息具有不同的相应，基类指针指向派生类的对象，基类指针呈现不同的表现方式

## 重载、重写、隐藏的区别

### 重载：同一个类中，函数名相同，参数个数、类型、顺序不同为重载，返回值不同其他相同不是重载（报错）

### 重写：virtual实现，不同类，子类对父类的同名方法重新实现，参数、返回值不能变

### 隐藏：子类隐藏父类方法（与父类同名方法）

### 重载与重写区别：

+ 范围区别：同一个类，不同类
+ 参数区别：不同，相同
+ 重写必须加virtual，重载可加可不加

### 隐藏和重写，重载的区别：

+ 范围区别：隐藏与重载范围不同，隐藏发生在不同类中。
+ 参数区别：隐藏函数和被隐藏函数参数列表可以相同，也可以不同，但函数名一定相同；当参数不同时，无论基类中的函数是否被 virtual 修饰，基类函数都是被隐藏，而不是重写（无法重写）



# 类（重点讲多态、也就是虚函数）

## 构造函数

### 默认构造函数：未提供任何实参，来控制默认初始化过程的构造函数（建立对象时不需要提供实参的构造函数）

## 多态：多态<font color="red">就是不同继承类的对象，对同一消息做出不同的响应，基类的指针指向或绑定到派生类的对象，使得基类指针呈现不同的表现方式。</font>在基类的函数前加上 virtual 关键字，在派生类中重写该函数，*运行时将会根据对象的实际类型来调用相应的函数。*如果对象类型是派生类，就调用派生类的函数；如果对象类型是基类，就调用基类的函数

## 虚函数：成员函数上加virtual的函数

### 纯虚函数在类的声明上加=0

### 含有纯虚函数的类叫抽象类，类只有接口，没有具体的实现方式

### 子类继承纯虚函数没有实现其方法，依然为抽象类，不能实例化

## <font color="red">不需要成为虚函数的成员函数</font>

+ 构造函数。**原因**：虚函数的调用需要虚表指针，虚函数表在编译期间就会构建，但是虚表指针是在对象生成时才会构建，而创建对象需要构造函数，二者冲突。
+ 静态成员函数：没有this指针无法访问虚表指针，且所有对象都是一个函数不存在多态
+ 内联函数：需要在编译时确定函数体，但虚函数运行时才有函数体

## 虚指针：存在虚函数的类都有一个一维的虚函数表叫虚表，虚表由<font color="red">数组构成</font>，每一个类对象具有一个虚表指针，所以虚表与类对应，虚表指针与对象对应

## 虚函数实现原理\*

### 编译器在编译阶段会为每一个包含虚函数的类建立一个虚表，该表是一个一维数组，数组中存放每一个虚函数的地址，如果重写了基类的函数，子类的虚函数地址覆盖父类的地址，不然就是父类地址。如果是虚继承<font color="red">先单独生成B1，B2以及覆盖B，再将B合并成一个，D来覆盖</font>，子类的成员函数被放到<font color="red">第一个父类的表</font>中。（第一个父类是按照声明顺序来判断的）,base2* t = new derive（） ，调用第二个继承的类虚函数指针

https://blog.csdn.net/qq_36359022/article/details/81870219

![image-20220227193656854](Picture\image-20220227193656854.png)

### 定位虚表：每个类都只有一个虚表， 编译器为每一个带有虚函数的类的对象自动创建一个虚表指针，在程序运行时（构造函数中进行初始化），构造函数里有为这个对象的vptr赋值的语句

## 基类析构函数需要定义为virtual的原因：基类指针可以指向派生类的对象（多态性），如果删除该指针（delete p），就会调用该指针指向的派生类析构函数，而派生类的析构函数又自动调用基类的析构函数。如果不定义virtual，只会调用基类的析构函数

## 父类为virtual，子类不加virtual，该方法也是virtual，子类如果不重写父类虚函数，会有一个与父类虚函数表<font color="red">内容相同</font>的虚函数表，但虚函数表指针内的地址<font color="red">不同</font>

## 如何减少构造函数开销:使用初试化列表，会少一次默认构造函数，（中间有一次临时构造的变量）一次赋值函数

```c++
#include <iostream>
using namespace std;
class A
{
private:
    int val;
public:
    A()
    {
        cout << "A()" << endl;
    }
    A(int tmp)
    {
        val = tmp;
        cout << "A(int " << val << ")" << endl;
    }
    A& operator= (const A& a)
    {
    	if(this==&a)
    	{
    		return *this;
		}
		this->val=a.val;
		cout<<"A& operator= (const& A) "<<endl;
		return *this;
	}
};

class Test1
{
private:
    A ex;

public:
    Test1() : ex(1) // 成员列表初始化方式
    {
    }
};

class Test2
{
private:
    A ex;

public:
    Test2() // 函数体中赋值的方式
    {
        ex = A(2);
    }
};
int main()
{
    Test1 ex1;
    cout << endl;
    Test2 ex2;
    return 0;
}

//运行结果 

/*
A(int 1)

A()
A(int 2)//生成临时变量
A& operator= (const& A)//将上面的临时变量通过赋值构造函数赋给A（）构建的变量
*/

```



## 对象初始化的顺序：父类构造函数–>成员类对象构造函数–>自身构造函数，其中成员变量的初始化与声明顺序有关，构造函数的调用顺序是类派生列表中的顺序.

+ 先按基类继承顺序调用父类构造函数构建
+ 然后是父类中的成员
+ 本身的成员的构造函数
+ 最后是本身

```c++
#include <iostream>
using namespace std;

class A
{
      public:
            A(){ cout<<“A”<<endl;}
            virtual ~A(){ cout<<“~A”<<endl; }
};

class B: public A
{
      public:
             B(){ cout<<“B”<<endl;}
             ~B() {cout<<“~B”<<endl; }
      private:
              A a;
};
AAABAABAC
class C: public A, public B     //类在派生表中的继承列表
{
      public:
              C() {cout<<“C”<<endl;}
              ~C() {cout<<“~C”<<endl; }
      private:
              B b;
      public:
             A a;
};

int main()
{
    C * p = new C;
    delete p;
   
    system(“PAUSE”);
    return 0;
}

结果和分析：

A          //1 父类A的构造函数
A         //2 父类B中A的构造函数
A         //3 父类B中成员变量b初始化 (调用父类A的构造函数）
B         //4  父类B中成员变量b初始化 (调用父类B的构造函数）
A        //5   C中成员变量b 的构造
A
B
A         //6 C中成员变量a的构造
C         //7 C的构造函数最后调用 （finally ，-__-|||)
~C
~A
~B
~A
~A
~B
~A
~A
~A
```

## 多重继承会出现什么情况：会出现命名冲突、数据冗余。1.使用虚继承的目的：保证存在命名冲突的成员变量/**成员函数**在派生类中只保留一份（使用及继承时前面加virtual）。2.使用类名：：变量名说明该变量来自哪个类

### 虚继承使派生类除了继承基类成员作为自己的成员之外，内部还会有一份内存来保存哪些是基类的成员。当Derive继承Base2和Base3之后，编译器根据虚继承多出来的内存，查到Base2和Base3拥有共同的基类的成员，就不会从Base2和Base3中继承这些，而是直接从共同的基类中继承成员，也就是说，Derive直接继承base的成员，然后再继承Base2和Base3各自新增的成员。这样，Derive就不会继承两份内存

```c++
#include <iostream>
using namespace std;

// 间接基类，即虚基类
class Base1
{
public:
    int var1;
};

// 直接基类 
class Base2 : virtual public Base1 // 虚继承
{
public:
    int var2;
};

// 直接基类 
class Base3 : virtual public Base1 // 虚继承
{
public:
    int var3;
};

// 派生类
class Derive : public Base2, public Base3
{
public:
    void set_var1(int tmp) { var1 = tmp; } //不使用虚继承，Base2::var1 = temp
    void set_var2(int tmp) { var2 = tmp; }
    void set_var3(int tmp) { var3 = tmp; }
    void set_var4(int tmp) { var4 = tmp; }

private:
    int var4;
};

int main()
{
    Derive d;
    return 0;
}

```

## 空类的对象占一个字节，为了防止不同对象占用相同的内存地址，空类对象有六个成员函数：默认构造函数、拷贝构造函数、赋值构造函数、取址运算符、const版本取址运算符、析构函数

## 类的成员函数后面加 const，表明这个函数不会对这个类对象的数据成员（准确地说是非静态数据成员）作任何改变。在设计类的时候，一个原则就是对于不改变数据成员的成员函数都要在后面加 const，而对于改变数据成员的成员函数不能加 const。所以 const 关键字对成员函数的行为作了更加明确的限定：

+ 有 const 修饰的成员函数（指 const 放在函数参数表的后面，而不是在函数前面或者参数表内），只能读取数据成员，不能改变数据成员；没有 const 修饰的成员函数，对数据成员则是可读可写的。
+ 除此之外，在类的成员函数后面加 const 还有什么好处呢？那就是常量（即 const）对象可以调用 const 成员函数，而不能调用非const修饰的函数。

## 不重要：只有当一个类没有定义任何自己的版本的拷贝控制成员，且它的所有数据成员都能移动构造或移动赋值时，编译器才会为它合成移动构造函数或移动赋值运算符

## 阻止类实例化，构造函数priavte，变为抽象类，构造函数=delete

## 为什么用成员初始化列表会快一些：初试化列表在调用构造函数前执行（虚指针也在这时赋值），但是构造函数先要调用默认构造函数，构造有参构造函数，再使用赋值构造函数（这种初试化方式，如果没有默认构造函数，就会报错）

## 深拷贝和浅拷贝的区别

+ 深拷贝：该对象和原对象占用不同的内存空间，既拷贝存储在栈空间中的内容，又拷贝存储在堆空间中的内容。

+ 浅拷贝：该对象和原对象占用同一块内存空间，仅拷贝类中位于栈空间中的内容

    ```c++
    #include <iostream>
    
    using namespace std;
    
    class Test
    {
    private:
    	int *p;
    
    public:
    	Test(int tmp)
    	{
    		p = new int(tmp);
    		cout << "Test(int tmp)" << endl;
    	}
    	~Test()
    	{
    		if (p != NULL)
    		{
    			delete p;
    		}
    		cout << "~Test()" << endl;
    	}
    	Test(const Test &tmp) // 定义拷贝构造函数
    	{
    		p = new int(*tmp.p);
    		cout << "Test(const Test &tmp)" << endl;
    	}
    
    };
    
    int main()
    {
    	Test ex1(10);	
    	Test ex2 = ex1; 
    	return 0;
    }
    /*
    Test(int tmp)
    Test(const Test &tmp)
    ~Test()
    ~Test()
    */
    
    ```

    

## 成员函数中的const{}，不允许修改成员变量的值，使用int mutable var1， var2在const中都可修改

## 让类不能被继承：使用final关键字；使用友员，私有构造析构函数以及虚继承（会直接从最上层的父类开始构造）

```c++
#include <iostream>
using namespace std;

template <typename T>
class Base{
    friend T;
private:
    Base(){
        cout << "base" << endl;
    }
    ~Base(){}
};

class B:virtual public Base<B>{   //一定注意 必须是虚继承,B为base的友元可以访问私有成员
public:
    B(){
        cout << "B" << endl;
    }
};

class C:public B{//B不可以被继承，C不是Base的友元，因为虚继承会直接从Base开始构造
public:
    C(){}     // error: 'Base<T>::Base() [with T = B]' is private within this context
};


int main(){
    B b;  
    return 0;
}

```

## 友元：通过友元，一个不同函数或另一个类中的成员函数可以访问类中的私有成员和保护成员

# 关键字

## extern告诉程序在本文件外的其他文件去找一个全局变量（即该变量的定义处），extern int a（不能直接接“=”）；a = 1

## static_cast相较于c的强制转换（）在编译期间进行类型检查,dynamic_cast<>()，由指向子类的基类指针转换为子类指针是安全的，由指向基类对象的基类指针转换为子类会进行安全检测，转换失败返回0

## sizeof与strlen区别

+ sizeof测量字符数组分配的大小；strlen计算字符个数以\0为终点
+ sizeof在测量形参的字符数组名时返回char*的大小（指针大小）；strlen计算字符个数以\0为终点
+ sizeof在编译的时候获得大小；strlen在运行时获得大小
+ sizeof可以是类型或变量；strlen只能是char*输入
+ sizeof是运算符；strlen是库函数`<cstring>`

## explicit保证类构造函数是显示调用的，不能隐式转换

```c++
#include <iostream>
#include <cstring>
using namespace std;

class A
{
public:
    int var;
    A(int tmp)
    {
        var = tmp;
    }
};
int main()
{
    A ex = 10; // 发生了隐式转换//直接调用构造函数A ex = A(10)
    return 0;
}

```

```c++
#include <iostream>
#include <cstring>
using namespace std;

class A
{
public:
    int var;
    explicit A(int tmp)
    {
        var = tmp;
        cout << var << endl;
    }
};
int main()
{
    A ex(100);
    A ex1 = 10; // error: conversion from 'int' to non-scalar type 'A' requested//显式调用
    return 0;
}

```

## static\*

+ ## 作用于局部变量：保持变量内容持久：`static` 作用于局部变量，改变了局部变量的生存周期，使得该变量存在于定义后直到程序运行结束的这段时间

+ ## 作用于全局变量/全局函数：改变了全局变量/全局函数的作用域，使其由全文件可见变为只能在定义它的文件可用

+ ## 作用于类的成员函数与成员变量：使类的成员函数与成员变量与类有关，不用建立对象，可以直接使用类名进行访问。静态成员函数不能定义为虚函数。static的成员函数无this指针所以只能访问静态成员变量（在类外进行访问）以及静态成员函数

## static在类中的注意事项\*

+ ### 静态成员变量要在类外初始化

    ```c++
    class A
    {
    private:
        int var;
        static int s_var; // 静态成员变量
    public:
        void show()
        {
            cout << s_var++ << endl;
        }
        static void s_show()
        {
            cout << s_var << endl;
    		// cout << var << endl; // error: invalid use of member 'A::a' in static member function. 静态成员函数不能调用非静态成员变量。无法使用 this.var
            // show();  // error: cannot call member function 'void A::show()' without object. 静态成员函数不能调用非静态成员函数。无法使用 this.show()
        }
    };
    int A::s_var = 1;
    
    ```

+ ### 静态成员变量被所有对象共享，包括派生类的对象

+ ### 静态成员变量可以作为成员函数的参数、静态成员变量可以申请为类对象但普通成员变量只能申请为该类的指针或引用

+ ### 静态成员函数无this指针不能访问成员函数或成员变量，不能申请为virtual、const、volatile

## static局部静态变量与全局变量区别

+ ### static在{}为界可以访问，在不同的函数中同名的变量也是各干各的

+ ### static只在编译时初始化一次，后面的初始化语句都没用（相当于不执行）

## const

+ ### 对于成员变量：在构造函数初始化后在整个类中就不能更改，但对于不同对象是不同的，不能在声明时就初始化，不然都相同了

+ ### 对于成员函数：不允许对成员变量做任何更改（volatile除外）,不能调用非const成员函数（因为可能会修改成员变量）

+ ### 对于形参：在函数体中不允许更改

## const（不可修改，不在常量区（“adv”，1），视情况在静态\全局变量区）与define，readonly

+ 编译阶段：`define` 是在编译预处理阶段进行替换，`const` 是在编译阶段确定其值
+ 安全性：define 定义的宏常量没有数据类型，只是进行简单的替换，不会进行类型安全的检查；const 定义的常量是有类型的，是要进行判断的，可以避免一些低级的错误

## #define 与 typedef区别

+ #define只是简单替换不做安全检查；typedef做安全检查
+ #define对数据类型、常量、变量、函数等都可替换；typedef只对数据类型继续替换
+ #define在预编译；typedef在编译期
+ #define没有作用域；typedef有作用域
+ #define处理指针const INTP = const int * ；typedef定于const INTP = int* const，多个是define只有第一个是指针，int *a，b（int变量），但是typedef就全是指针

## #define比较大小

```c++
#include <iostream>
#define MAX(X, Y) ((X)>(Y)?(X):(Y))//括号必须有
#define MIN(X, Y) ((X)<(Y)?(X):(Y))
using namespace std;

int main ()
{
    int var1 = 10, var2 = 100;
    cout << MAX(var1, var2) << endl;
    cout << MIN(var1, var2) << endl;
    return 0;
}
/*
程序运行结果：
100
10
*/

```

## inline：在编译阶段将每个调用内联函数的地方加入函数体

+ 在函数点调用处展开，效率更好
+ 取出只能定义一次的限制
+ 在运行时内存中有定义因为可能不能变为内联函数

## new / malloc 与delete / free区别

+ new调用构造函数、析构函数；malloc不会
+ new使用自由储存区（是一种抽象概念，使用new就是在自由储存区，可以是堆，也可以不是）；malloc是堆（有操作系统管理的特殊内存）
+ 安全：new严格返回对象类型的的指针；malloc返回void* 需要强转
+ 分配失败：new会bad_alloc异常不返回null；malloc返回null
+ new不需要指定内存大小；malloc需要
+ 销毁时new用delete或delete[]；malloc只给内存自己指定
+ new可以调用malloc
+ new可重载
+ malloc可使用realloc扩充
+ 内存不够：客户能够指定处理函数或重新制定分配器

## C中struct与C++区别

+ c中是自定义数据类型；c++是一个抽象数据类型，可以有成员函数
+ C中无访问权限；C++可以有访问权限
+ 定义时用struct；C++不用

## union和struct区别

+ union按其中最大的基本类型（不是数组）的倍数去容量，这个容量要大于最大的变量的大小；struct遵循内存对齐
+ 同一时间union只有一个数据类型有效；struct所有都有效
+ 对联合体的不同成员赋值，将会对覆盖其他成员的值；结构体不会

## struct与class区别

+ struct默认为public；class默认private
+ struct继承默认public继承；class是private继承。取决于派生类
+ 都自定义数据类型、都可继承

## public、protected 或 private继承，不影响子类访问父类权限，都能访问public与protected成员，无法访问private成员

+ 公有继承public与protected成员是派生类的公有成员，基类的保护成员也是派生类的保护成员
+ 当一个类派生自保护基类时，基类的公有和保护成员将成为派生类的保护成员
+ 当一个类派生自私有基类时，基类的公有和保护成员将成为派生类的私有成员

## volatile 的作用，可以与<font color="red">const使用</font>：

+ 当对象的值可能在程序的控制或检测之外被改变时，应该将该对象声明为 violatile，告知编译器不应对这样的对象进行优化。

+ volatile不具有原子性。

+ volatile 对编译器的影响：使用该关键字后，编译器不会对相应的对象进行优化，即不会将变量从内存缓存到寄存器中，防止多个线程有可能使用内存中的变量，有可能使用寄存器中的变量，从而导致程序错误

# 左值引用与移动语义

## 临时值为纯右值（prvalue），不能取地址，在表达式结束后销毁；表达式后还存在的为左值，可以取地址，所有的具名变量或对象都是左值，而匿名变量则是右值

## C++所有值为左值、将亡值、纯右值三者之一

+ 非引用返回的临时变量、运算表达式产生的临时变量、原始字面量和lambda表达式等都是纯右值
+ 将亡值是C++11新增的、与右值引用相关的表达式，比如，将要被移动的对象、T&&函数返回值、std::move返回值和转换为T&&的类型的转换函数的返回值等
+ 函数以值return会发生一次拷贝构造，=会调用拷贝构造

## 只能通过引用的方式（相当于延长生命周期）获取右值，相当于生命周期与右值引用变量一样长

## 右值引用

### const A& 即可以左值引用、右值引用、左值常量引用、右值常量引用

### 右值引用独立于左值和右值。意思是右值引用类型的变量可能是左值也可能是右值

### T&& t在发生自动类型推断的时候，它是未定的引用类型（universal references），如果被一个左值初始化，它就是一个左值；如果它被一个右值初始化，它就是一个右值，它是左值还是右值取决于它的初始化。注意：<font color="red">根据universal references的特点，t被一个右值初始化，那么t就是右值；当参数为左值x时，t被一个左值引用初始化，那么t就是一个左值。需要注意的是，仅仅是当发生自动类型推导（如函数模板的类型自动推导，或auto关键字）的时候，T&&才是universal references</font>

```c++
template<typename T>
void f(T&& param); //自动推断，可左可右

template<typename T>
class Test {
    Test(Test&& rhs); //右值引用,确定类型
};
//模版函数f发生了类型推断，而Test&&并没有发生类型推导
//返回值是一个临时变量
```

## 移动构造:move: static_cast<T&&>(lvalue) 会先remove && &，再转

+ ### 标准库使用比如vector::push_back 等这类函数时,会对参数的对象进行复制,连数据也会复制.这就会造成对象内存的额外创建, 本来原意是想把参数push_back进去就行了,通过std::move，可以避免不必要的拷贝操作

+ ### std::move是将对象的状态或者所有权从一个对象转移到另一个对象，只是转移，没有内存的搬迁或者内存拷贝所以可以提高利用效率,改善性能

## this 指针是 指针常量 

## 悬空指针：若指针指向一块内存空间，当这块内存空间被释放后，该指针依然指向这块内存空间，此时，称该指针为“悬空指针”。

## 野指针：“野指针”是指不确定其指向的指针，未初始化的指针为“野指针”。

## 指针与引用区别*

+ 指针可以不初始化；引用必须初始化
+ 指针可以更改；引用一旦绑定不可以修改
+ 指针可以有多级；引用不可以

## 值传递、指针传递、引用传递

+ 值传递：相当于值进行复制
+ 指针传递：指针地址进行复制，指向实参的指针
+ 引用传递：形参相当于是实参的“别名”，对形参的操作其实就是对实参的操作，在引用传递过程中，被调函数的形式参数虽然也作为局部变量在栈中开辟了内存空间，但是这时存放的是由主调函数放进来的实参变量的地址。被调函数对形参的任何操作都被处理成间接寻址，即通过栈中存放的地址访问主调函数中的实参变量。
+ 指针传递的形参可以改变指向，可以指向nullptr；引用传递不可以转变，不能指向nullptr

## 模板：类模板以及函数模板  template <typename T, typename U, ...>。定义：模板就是实现代码重用机制的一种工具，它可以实现类型参数化，即把类型定义为参数， 从而实现了真正的代码可重用性

### 函数模板：通过定义一个函数模板，可以避免为每一种类型定义一个新函数，返回值、形参类型以及以及在函数体内用于变量声明或类型转换。实例化自动判断参数类型

```c++
#include<iostream>

using namespace std;

template <typename T>
T add_fun(const T & tmp1, const T & tmp2){
    return tmp1 + tmp2;
}

int main(){
    int var1, var2;
    cin >> var1 >> var2;
    cout << add_fun(var1, var2);

    double var3, var4;
    cin >> var3 >> var4;
    cout << add_fun(var3, var4);
    return 0;
}

```

### 类模板：类似函数模板，类模板以关键字 template 开始，后跟模板参数列表。但是，编译器不能为类模板推断模板参数类型，需要在使用该类模板时，在模板名后面的尖括号中指明类型。(c++17后可以自动推导)

```c++

#include <iostream>

using namespace std;

template <typename T>
class Complex
{
public:
    //构造函数
    Complex(T a, T b)
    {
        this->a = a;
        this->b = b;
    }

    //运算符重载
    Complex<T> operator+(Complex &c)
    {
        Complex<T> tmp(this->a + c.a, this->b + c.b);
        cout << tmp.a << " " << tmp.b << endl;
        return tmp;
    }

private:
    T a;
    T b;
};

int main()
{
    Complex<int> a(10, 20);
    Complex<int> b(20, 30);
    Complex<int> c = a + b;

    return 0;
}

```

## 全特化（所有类型指明）以及偏特化（类模板不可重载，函数模板可以重载）。为原函数模板定义了一个特殊实例，而不是函数重载，函数模板特化并不影响函数匹配

+ 类模板和函数模板都可以被全特化；

+ 类模板能偏特化，不能被重载；

+ 函数模板全特化，不能被偏特化。

+ 模板类调用优先级

    对主版本模板类、全特化类、偏特化类的调用优先级从高到低进行排序是：
    **全特化类>偏特化类>主版本模板类*

    

## 类型萃取，使用模板特化来进行，对不同类型数据进行不同处理（深浅拷贝、std：：move通过remove_reference提取T）*(用了偏特化（模板指针）与全特化)

```c++
template<typename T>
void Copy(T *dest, const T *src, size_t sz)
{
    if (_TypeTraits<T>::_IsPODType().Get())
    {
        cout << "内置类型用memcpy函数进行拷贝：" << endl;
        memcpy(dest, src, sz * sizeof(T));
    }
    else
    {
        cout << "自定义类型用for循环进行拷贝：" << endl;
        for (size_t i = 0; i < sz; i++)
        {
            *(dest + i) = *(src + i);
        }
    }
}

struct _TrueType
{
    bool Get()
    {
        return true;
    }
};
struct _FalseType
{
    bool Get()
    {
        return false;
    }
};
template<typename T>
struct _TypeTraits
{
    typedef _FalseType _IsPODType;
};
template<>
struct _TypeTraits<int>
{
    typedef _TrueType _IsPODType;
};

struct _TypeTraits<short>
{
    typedef _TrueType _IsPODType;
};
struct _TypeTraits<long>
{
    typedef _TrueType _IsPODType;
};
struct _TypeTraits<long long>
{
    typedef _TrueType _IsPODType;
};
struct _TypeTraits<unsigned>
{
    typedef _TrueType _IsPODType;
};
template<>
struct _TypeTraits<char>
{
    typedef _TrueType _IsPODType;
};
struct _TypeTraits<unsigned char>
{
    typedef _TrueType _IsPODType;
};
template<>
struct _TypeTraits<double>
{
    typedef _TrueType IsPODType;
};
struct _TypeTraits<float>
{
    typedef _TrueType _IsPODType;
};
struct _TypeTraits<bool>
{
    typedef _TrueType _IsPODType;
};
```



# 数据结构

## 二叉查找树：查找一个数的时间可以控制在缺点在于插入可能导致二叉搜索树的不平衡，查找效率接近线性查找

### 二叉搜索数的插入：

## 红黑树：为了解决二叉查找树不平衡的问题，保证从根节点到叶子节点的最长路径不会超过最短路径的两倍

### 红黑树的特点

+ 节点都是黑色或红色
+ 每个叶子节点都是黑色的空节点（NIL节点）
+ 根节点是黑色
+ 每个红色节点的两个节点都是黑色
+ 从某一节点到任意叶子节点的路径所遇到的黑色节点数目相同

### 保证红黑树插入和删除依然是红黑树的方法

+ #### 变色![image-20220301234029522](Picture\红黑树变色.png)

+ #### 旋转

    + 左旋转![image-20220301234138519](Picture\红黑树左旋转.png)
    + 右旋转![image-20220301234206939](Picture\红黑树右旋转.png)

+ 插入情况的四种局面的调整方式
    + 新加入的红色节点的父节点为黑色，不需要任何调整![image-20220302000220152](Picture\红黑树局面1.png)
    + 新加入的红色节点的父节点和叔叔节点都是红色：![image-20220302000658884](Picture\红黑树局面2.png)
    + 新结点（D）的父结点是红色，叔叔结点是黑色或者没有叔叔，且新结点是父结点的右孩子，父结点（B）是祖父结点的左孩子![image-20220302000845909](Picture\红黑树局面3.png)
    + 新结点（D）的父结点是红色，叔叔结点是黑色或者没有叔叔，且新结点是父结点的左孩子，父结点（B）是祖父结点的左孩子![image-20220302001113665](Picture\红黑树局面4.png)
    + https://blog.csdn.net/bjweimengshu/article/details/106345677?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522164614788416780269873846%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=164614788416780269873846&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-106345677.pc_search_insert_ulrmf&utm_term=%E7%BA%A2%E9%BB%91%E6%A0%91&spm=1018.2226.3001.4187

## 平衡⼆叉树（AVL树）：

+ 红黑树是在AVL基础上提出
+ 平衡二叉树是一种特殊的二叉排序树
+ 左右子树都是平衡二叉树，且左右子树高度差不会超过1

## set与multiset

+ key与value一样
+ set的迭代器不允许写入操作（const_iterator），写入会影响红黑树的排列
+ set的key必须唯一使用insert_unique进行插入，multiset的key可以重复使用insert_equal插入

## map与multimap

+ map与multimap中的iterator无法改变key，key被初始化为const，可以通过迭代器改变value
+ map的key必须唯一使用insert_unique进行插入，multimap的key可以重复使用insert_equal插入
+ map与multimap排序依据key

## 哈希表/散列表，主要面向查找的储存结构，不适合范围查找，单个关键字不能对应多个记录，使用键值对储存，在内存和查找顺序进行平衡（要想O(1),key值很大需要内存浪费）

### 通过哈希函数进行查找的，key为关键字，f为哈希函数，f(key)是存储位置，哈希函数要计算简单、使储存地址分布均匀，保证储存空间的有效利用

### 哈希函数选择

+ 直接寻址法：适用于地址大小 = 关键字集合的情况，事先知道关键字分布，表较小的情况，ax+b
+ 数字分析法：处理关键字位数较长的情况 
+ 平方取中法：不知道关键字分布、位数不是很大。关键字平方扩大差异，而后取中间数作为地址
+ 折叠法：不需要知道关键字分布、蛇和关键字位数较多的情况。从左到右分割成位数相等的几部分，几部分叠加求和，按照散列表表厂，取后几位作为散列地址。
+ 除留余数法（常用）：H(key) = key % p  p < m(表长)，p最好是接近m的最小质数，或是不包含小于20质因子的合数
+ 随机数法：H（key）= Random（key），关键字的长度不等，采用该方法比较合适

### 冲突：两个不同的key，计算出的储存地址一致，解决冲突：

+ 开方定制法：fi( key) = ( f ( key ) + di ) MOD m
    + di = di++
    + fi ( key ) = ( f ( key ) + di ) MOD m  di = 1^2, -1^2, 2^2, -2^2 ···
    + fi( key ) = ( f ( key ) + di) MOD  di 是一个随机数列

+ 再散列函数法：换不同的哈希函数
+ 链地址法：将相同的哈希结果的元素放在同一个位置，形成单链表![image-20220302213515350](Picture\链地址法.png)
+ 公共溢出区法 ：建立一个特殊存储空间，专门存放冲突的数据，适用于数据和冲突较小的
    + 先于基本表的相应位置进行对比
        - 相等，查找成功
        - 不相等,到溢出表里进行顺序查找![image-20220302213747980](Picture\公共溢出区间.png)

## unordered_map 、 unordered_set 、unordered_multimap、unordered_multiset

### 基于哈希表，insert_unique insert_equal

## B与B+树：相比于哈希表，支持范围扫描.[B树B+树面试_牛客博客 (nowcoder.net)](https://blog.nowcoder.net/n/f9018d0180244e1caa5e8b2d085fc983?from=nowcoder_improve#基本知识)

[『数据结构与算法』B树图文详解（含完整代码） - 简书 (jianshu.com)](https://www.jianshu.com/p/bb3d95c5086b)

[B+树看这一篇就够了（B+树查找、插入、删除全上） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/149287061)

### 定义：B树（B-tree）是一种树状数据结构，能够<font color="red">用来存储排序后的数据</font>。这种数据结构能够让<font color="whitblue">查找数据、循序存取、插入数据及删除</font>的动作，都在<font color="red">对数时间内完成</font>。B树，概括来说是一个一般化的二叉查找树，可以拥有多于2个子节点。与自平衡二叉查找树不同，B-树为<font color="red">系统最优化大块数据的读和写操作</font>。B-tree算法减少定位记录时所经历的中间过程，从而加快存取速度。这种数据结构常被应用在数据库和文件系统的实作上。

### B树增添过程

+ 如果数量没有超过界限，直接插入
+ 如果数量超过界限，将中间数增添到父节点，将小于中位数放到左边，大于右边
+ 如果第二步父节点也超过，继续分离![img](Picture\B Tree.png)

## B树删除过程

+ 如果删除后个数符合要求，直接删除
+ 对于叶子节点：如果删除后元素个数太小，将父节点下移一个，兄弟节点（删除一个元素也符合数量要求）上移
+ 对于内部节点：删除节点，需要从子树边缘值抽取一个，若左右节点元素数达到下限，进行合并（与兄弟、父、自己）![img](Picture\B树删除.png)

### 对于m阶子树，B树根节点关键字数量`1<=k<=m-1`，非根节点关键字数量`m/2<= k <=m - 1`

### B+与B树之间的不同

+ B+树n个关键字有n个子树；B树n个关键字有n + 1个子树
+ B+树关键字个数m/2（向上取整）<= n <=m(根：1<= n <=m);B树根节点关键字数量`1<=k<=m-1`，非根节点关键字数量`m/2(向上取整) - 1<= k <=m - 1`
+ B+树中有一个指针指向关键字最小的叶子节点，所有叶子节点链接成一个单链表
+ B+树叶子节点包含了所有关键字，叶子节点指向记录；B树中，叶子节点的关键字与其他的关键字不同

### B+树插入相较于B树，中位数在去到父节点后，还会保存在底部叶子节点中

### B+树的分裂以及合并需要更改上面的节点(不是最大最小只用更替父节点，否则从根节点到叶子节点都更替)，与兄弟节点借或者和兄弟节点合并

### B+树优点：

+ B+树的磁盘读写代价更低
+ B+树查询效率更加稳定
+ B+树更有益于数据库扫描

## vector为什么成倍增长

+ vector在push_back以成倍增长可以在均摊后达到O(1)的时间复杂度，相对于增长指定大小的O(n)时间复杂度更好。
+ 为了防止申请内存的浪费，现在使用较多的有2倍与1.5倍的增长方式，而1.5倍的增长方式可以更好的实现对内存的重复利用，因为更好

## 回调函数：声明函数指针 = 函数名 返回值  （*funp）（int， string） = func；

![image-20220331184618633](Picture\image-20220331184618633.png)

## 锁

`std::mutex mut std::unique_lock<std::mutex> lk(mut)`std::unique_lock相对std::lock_guard更灵活的地方在于在等待中的线程如果在等待期间需要解锁mutex，并在之后重新将其锁定

`cond_.wait(lk, [this]{return !Empety();}); `。方括号内表示捕获变量。当lambda表达式返回true时（即queue不为空），wait函数会锁定mutex。当lambda表达式返回false时，wait函数会解锁mutex同时会将当前线程置于阻塞或等待状态

## 假设线程1需要线程2的数据，那么组合使用方式如下：

+ 线程1初始化一个promise对象和一个future对象，promise传递给线程2，相当于线程2对线程1的一个承诺；future相当于一个接受一个承诺，用来获取未来线程2传递的值
+ 线程2获取到promise后，需要对这个promise传递有关的数据，之后线程1的future就可以获取数据了。
    如果线程1想要获取数据，而线程2未给出数据，则**线程1阻塞**，直到线程2的数据到达.

```c++
#include <semaphore.h>

class Foo {
private:
    sem_t sem_1, sem_2;

public:
    Foo() {
        sem_init(&sem_1, 0, 0), sem_init(&sem_2, 0, 0);
    }

    void first(function<void()> printFirst) {
        printFirst();
        sem_post(&sem_1);
    }

    void second(function<void()> printSecond) {
        sem_wait(&sem_1);
        printSecond();
        sem_post(&sem_2);
    }

    void third(function<void()> printThird) {
        sem_wait(&sem_2);
        printThird();
    }
};


class Foo {
    condition_variable cv;
    mutex mtx;
    int k = 0;
public:
    void first(function<void()> printFirst) {
        printFirst();
        k = 1;
        cv.notify_all();    // 通知其他所有在等待唤醒队列中的线程
    }

    void second(function<void()> printSecond) {
        unique_lock<mutex> lock(mtx);   // lock mtx
        cv.wait(lock, [this](){ return k == 1; });  // unlock mtx，并阻塞等待唤醒通知，需要满足 k == 1 才能继续运行
        printSecond();
        k = 2;
        cv.notify_one();    // 随机通知一个（unspecified）在等待唤醒队列中的线程
    }

    void third(function<void()> printThird) {
        unique_lock<mutex> lock(mtx);   // lock mtx
        cv.wait(lock, [this](){ return k == 2; });  // unlock mtx，并阻塞等待唤醒通知，需要满足 k == 2 才能继续运行
        printThird();
    }
};

class Foo {
    promise<void> pro1, pro2;

public:
    void first(function<void()> printFirst) {
        printFirst();
        pro1.set_value();
    }

    void second(function<void()> printSecond) {
        pro1.get_future().wait();
        printSecond();
        pro2.set_value();
    }

    void third(function<void()> printThird) {
        pro2.get_future().wait();
        printThird();
    }
};




void thread_set_promise(std::promise<int>& promiseObj) {
    std::cout << "In a thread, making data...\n";
    std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    promiseObj.set_value(35);
    std::cout << "Finished\n";
}

int main() {
    std::promise<int> promiseObj;
    std::future<int> futureObj = promiseObj.get_future();
    std::thread t(&thread_set_promise, std::ref(promiseObj));
    std::cout << futureObj.get() << std::endl;
    t.join();

    system("pause");
    return 0;
}
```

![image-20220309123720987](Picture\image-20220309123720987.png)

## final可以作用于类不可以继承，作用于虚函数不允许重写

# 个人理解

## 类的构造函数具有全局可见性（类似全局变量）

## 普通函数是将程序执行转移到被调用函数所存放的内存地址，当函数执行完后，返回到执行此函数前的地方。转移操作需要保护现场，被调函数执行完后，再恢复现场，该过程需要较大的资源开销

## 在c++中struct和class的唯一区别是成员的默认访问权限。struct默认是public，而class默认是private

## C没有布尔类型

## delete和delete【】，在无析构函数的类型一致，但是有析构，内存删除是一致的都全删除，但是delete对于多个对象指针只调用一次析构函数，delete调用size次数

## switch case在同一作用域，在上面case的变量，当符合下面case的条件，还会对其进行操作

## <>在系统类库目录中进行查找，找不到报错；“”先当前目录（。vcxproj目录）然后include文件最后类库文件

## 迭代器

+ 迭代器：一种抽象的设计概念，在设计模式中有迭代器模式，即提供一种方法，使之能够依序寻访某个容器所含的各个元素，而无需暴露该容器的内部表述方式。

+ 作用：在无需知道容器底层原理的情况下，遍历容器中的元素。

