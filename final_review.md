# 期末复习

2023~2024面对对象程序设计（C++）

## 基础

#### new和delete的使用

- 数组对应new ElementType**[]** 和 delete**[]**，记得`[]`
- `new double[100]{}`或`new double[100]()`会初始化成$0$
- `delete nullptr`是**安全**的
- `delete this`是允许的，但之后**不允许**再调用`this`
- `delete 指针`，**只**析构指针**指向的类**（**基类**）
- 不应该多次`delete`同一个指针

#### 引用

- `T& refname = name;`

- 定义时必须**初始化**

- 引用的目标必须是**左值**（有地址）
  - 例如`i * 3`就不行

- 引用关系在运行过程中**不会改变**，对引用赋值只会改变原变量的值

- **不允许**引用的引用
  
  <img src="D:\Leaning\OOP\image-20240101155432948.png" alt="image-20240101155432948" style="zoom:80%;" />
  
  这是一个很$confusing$的例子，但事实上b和c都是a的引用。引用的引用应该写成`int &(&a) = b`
  
- **不允许**指针指向一个引用
  
  - `int&* p;`是**非法**的
  
- **允许**引用指针
  - `int*& p;`是**允许**的

- **没有**数组的引用

#### const

- 必须被**初始化**，并且值必须是**常数**（除非是`extern const T`）

- **const指针**

  - `int const* a` 和 `const int* a` 指针指向的值不可变
  - `int* const a`指针不可变
  - 除了`const_cast`

- **const函数**

  - `const T& f()` 函数返回值不可修改
  - `T f() const` 修饰成员函数，函数体中不能修改**成员变量**（本质上是`const this`）

- **const对象**

  - **const对象**只能调用**const成员函数**

  - \* 编译阶段常量

     <img src="D:\Leaning\OOP\image-20240101164401249.png" alt="image-20240101164401249" style="zoom:50%;" />



## 类与对象（封装）

#### 构造函数 `C'tor`

- 与类名相同，没有返回值

- 默认构造函数，自定义之后会被覆盖

- 构造时如果**没有参数**，就**不应该**加`()`

- **初始化列表**

  - 初始化顺序是变量的**声明顺序**，析构时顺序相反

  - 可以初始化的变量包括：**非静态**数据成员或**基类**（利用基类构造函数）

  - **常量**，**引用**，**没有默认构造函数**的类**必须**在初始化列表里初始化

  - `explicit` **禁止**参数隐式转换

    ```cpp
    class A{
    	explicit A(int i){
            // do something
        }
    }
    int main(){
    	A a1(0);	// OK
    	A a2 = 0;	// ERROR
    }
    ```

- 构造顺序：（事实上最先是静态成员）先构造**初始化列表**，然后构造**成员变量**（调用成员变量的**构造函数**），最后**执行**构造函数**函数体**


#### 拷贝构造函数

- 函数名同构造函数，参数是`const T&`
- **没有返回值**
- 可以访问类的**私有**变量
- 内存分配相关——深拷贝（分配新地址）
- 拷贝构造函数也会**覆盖**默认构造函数
- 理论上，通过**值**传递的函数参数以及返回值都会调用拷贝函数，但这**可能**会被编译器优化
- 如果你不允许拷贝构造函数，将拷贝函数设置为私有，或声明为`T(const T& rhs) = delete;`**（better）**

#### 析构函数 `D'tor`

- 类名前加`~`，**没有返回值**
- 析构顺序：**完全**与**构造顺序相反**

#### 默认参数

- 默认参数是在**编译**时解析的，所以不能动态绑定

- **同一个默认参数只能写一次**，最好写在声明里（如果声明与定义分开的话）

- 默认参数必须写在所有必要参数的**后面**

- 对于非模板函数，如果已声明的函数在同一作用域重新声明，则可以**添加**默认参数（**已有**的**不能**重写）

  ```cpp
  void f(int, int);
  void f(int, int = 7);
  ```

- 默认参数**不允许**使用局部变量，除了`sizeof n`等“not evaluated”的值
- 除了`()`以外的**运算符重载**不能有默认参数

#### 内联函数

- inline声明应写在头文件里，**只有**函数体写在**类定义**中的成员函数才可能会被内联
- 曾经的含义是允许编译器内联，现在的意义是**防止多重定义**
- 简短的函数应该内联，递归调用的函数不应该内联

#### 静态

- 静态普通变量 / 静态普通函数
  - 编译时确定，**仅初始化一次**
  - 仅能在本文件使用，不能在其他文件声明为`extern`（外部变量）
- 静态对象
  - 构造函数最多call一次，**全局静态变量在`main`之前创建**，**局部静态对象不使用不创建**
  - 析构函数在程序退出时调用
- 静态成员变量
  - **所有对象共享**
  - 静态成员变量的初始化只能在**类外**，需要“**声明**”，**不初始化无法调用！**（因为在类里只是留了个位置）
    - e.g. `int Box::objectCount = 0;`
- 静态成员函数
  - 即使在类**对象不存在**的情况下也能被调用，静态函数只要使用**类名**加范围解析运算符 **::** 就可以访问。
    - e.g. `Box::print(x);`
  
  - 只能访问**静态成员数据**、其他**静态成员函数**和类**外部**的其他函数以及**函数参数**
  - **不能**访问`this`指针，因为不和对象绑定
  - **不能**动态重写，**没有**多态
  - 当然，它也受到`private`**访问权限的限制**

#### Tips

 <img src="D:\Leaning\OOP\image-20240101164954894.png" alt="image-20240101164954894" style="zoom:40%;" />



## 嵌套与继承

#### 嵌套

- 内层嵌套的类**必须**被初始化，如果没有提供参数，会调用**默认**构造函数（如果有），否则无法构造

#### 继承

- 理解：基类概念更**广**，派生类是**特化**
- 派生类构造函数**初始化列表**要构造基类，基类总是**最先**构造

- **允许继承**的部分：成员变量、成员函数、接口
- **不能**被继承的部分
  - 构造函数
  - 析构函数
  - 重载**`=`**运算符
- **Name Hiding**
  - 如果在派生类中重定义成员函数，那么基类中**所有的重载**会被**覆盖**【请使用虚函数】
- **访问权限**
- `Public`：都可以访问
  - `protected`：只有**自己**和**派生类**（和**友元类**）可以访问
  
- `private`：只有**自己**和**友元类**可以访问
  
-   <img src="D:\Leaning\OOP\image-20240101171752945.png" alt="image-20240101171752945" style="zoom: 50%;" />
  
- 继承关系
  
  -   <img src="D:\Leaning\OOP\image-20240101171456150.png" alt="image-20240101171456150" style="zoom:50%;" />
    - `Public`：权限不变【通常使用这种】
    - `protected`：`public`变为`protected`，其余不变
  
  - `private`：所有变量的权限都变为`private`

#### 友元

- 友元是**单向**的

- 友元**不能**被继承

- 分为友元类，全局友元函数，类的友元成员函数

- 友元类（函数）可以访问该类的所有成员变量以及成员函数

  - 对于**友元成员函数**，情况有些**复杂**

    ```cpp
    class A;    //当用到友元成员函数时，需注意友元声明与友元定义之间的互相依赖。这是类A的声明
    class B{
    public:
        void set_show(A &a);             //该函数是类A的友元函数
    };
    
    class A{
    public:
        friend void B::set_show(A &a);   //该函数是友元成员函数的声明
    private:
        int data;
    };
    
    void B::set_show(A &a)       //只有在定义类A后才能定义该函数，毕竟，它被设为友元是为了访问类A的成员
    {
        cout << a.data << endl;
    }
    
    ```


#### 向上造型 `Upcast`

- 所有需要**基类**的地方（包括基类的**指针**或**引用**）都可以使用**派生类**，这是**安全**的
- `downcast`是不安全的

#### 多继承

- 多用于**抽象类**或**接口类**
- 建议是不用（樂

```cpp
class B1 { int m_i; }
class D1 : public B1 {}
class D2 : public B1 {}
class M : public D1, public D2 {};

int main() {
M m ; // OK
    B1* p = &m; // ERROR: 你要走哪条路捏
    B1* p1 = static_cast<D1*>(&m); // OK
    B1* p2 = static_cast<D2*>(&m); // OK
}
```

- 虚基类

  虚基类的**构造函数**仅由**继承层次最深**的类调用，只会产生一个对象

```cpp
class Base {}
class A: virtual public Base {}
class B: virtual public Base {}
class D: public A, public B {}
```



## 多态

​	只有**虚函数**可以发生**多态**和**动态绑定**（运行时确定）

​	函数默认参数是**静态解析**的（**基类**）

​	成员变量是**自己**的：如果发生**多态**，就是**派生类**；否则是**基类**

#### 虚函数

- 关键字 `virtual`，**动态绑定**
- **<u>静态成员函数</u>** 和 **<u>构造函数</u> 不可以**设置为虚函数，**<u>析构函数</u> 应该**设置为虚函数

- 如果把派生类的**对象**赋值给基类，仅基类的部分会保存，同时会使用基类的函数；

​	如果把基类的**指针**指向派生类，就会调用派生类的虚函数；

​	**引用**与指针类似；

​	总的来说，只有**指针**和**引用**涉及虚函数：实际是什么就调用什么；**对象赋值不是**。

​	注意派生类指针**不能**指向基类对象

- 如果通过传递**对象本身**而不是**指针**或**引用**的方法传参，那么会进行**类型转换**，也就没有多态了。

- 函数、虚表都是**类公有**的，虚表<u>指针</u>、成员变量是**对象私有**的

  所以非虚的部分不能动态绑定，基类的指针也不能访问派生类独有的非虚成员

- 虚函数会**自动继承**，派生类无需声明。（除非用了`final`）

- 重写`override`

  - `void func() override;`

  - 表示基类的虚函数需要**重写**

  - 基类被重写的**虚函数**仍然可以**重用**：

     <img src="D:\Leaning\OOP\image-20240101185314137.png" alt="image-20240101185314137" style="zoom:50%;" />

  - **返回类型放松**

    派生类重写基类函数时，允许把**函数定义**中：基类**指针或引用**的**返回值**改成**派生类**的指针或引用

  - Name Hiding

     如果重写了其中一个基类函数，所有的（同名）重载函数都会被覆盖

- 纯虚函数

  - `virtual int func() = 0;`
  - 不能定义虚函数的实现
  - 在重写之前无法调用该函数

- 抽象类

  - 至少包含一个纯虚函数
  - 用于定义接口

- 协议类（接口）

  - 所有**非静态**的成员函数都是**纯虚函数**，**除了析构函数**
  - 析构函数**函数体为空**
  - **没有非静态**的成员变量，或继承变量
  - 可能有**静态成员**

  - e.g.

     <img src="D:\Leaning\OOP\image-20240101192027949.png" alt="image-20240101192027949" style="zoom:50%;" />

#### 重载

- 两个**只有**返回值类型不同的函数不是合法重载，因为无法完成重载解析。换言之，**参数表必须不同**
- 参数列表只有**是否引用**或是否**const**不同，也无法重载
- **同时存在**const函数和非const**重载**函数的前提下，若**非const对象**想调用**const成员函数**，则需要**显式的转化**；反之同理
- **析构函数**不能被重载

##### 运算符重载

- 关键字：`operator`

- **参数**：`const T&`（理应如此）

- **不能**被重载的运算符：

  `.`	`.*`	`::`	`?:`

  `sizeof`	`typeid`

  四类强制转换—— `static_cast`	`dynamic_cast`	`const_cast`	`reinterpret_cast`

- 成员指针`::*`，`.*`（类似于起别名，作用于整个类）

  - 成员变量指针

    ```cpp
    int MyClass::*ptrToData = &MyClass::data;
    cout << obj.*ptrToData << endl;
    ```

  - 成员函数指针

    ```cpp
    void (MyClass::*ptrToFunc)() = &MyClass::display;
    (obj.*ptrToFunc)();
    ```

  - 定义时绑定在**类**：`::*`

    调用时绑定于**对象**：`.*`

- 限制

  - 只有**存在**的运算符可以被重载
  - 保留操作数**个数**
  - 保留**优先级**

- 分类

  - 成员函数

    - 写法

      ```cpp
      String String::operator+(const String& that);
      ```

    - **隐含**第一个参数

    - 运算符的接收者（即运算符**左侧**的对象）不会进行类型转换，即冲突时会转换**右侧**对象

  - 自由函数（全局函数）

    - 写法

      ```cpp
      String operator+(const String& l, const String& r);
      ```

    - **两侧**对象都可能进行类型转换（转换成接受赋值对象的类型）

    - 可能需要声明成**友元**函数

  - 规则

    - **=  ()   []   ->**必须是**成员**函数
    - 流运算符必须是**自由函数**
    - **单目**运算符应该声明为**成员**函数（声明为成员函数时**不能**带形参）
    - 其它**双目**运算符最好声明为**自由**函数

- 返回值

  - 算数运算应该返回一个**新对象（const）**
  - 逻辑运算应该返回**bool**
  - 下标运算符应该返回**const&**

- **自增 / 自减**运算符

  - **++i**

    ```cpp
    const Integer& operator++();
    ```

    **引用表示直接自增**

  - **i++**

    ```cpp
    const Integer operator++(int);
    ```

    **括号内的`int`表示后缀，有temp**

- **`=`** **赋值**运算符

  - 有默认形式（**自动**创建）

  - 有指针时需要**释放**分配原来的内存，注意检查**自我赋值**

  - 注意与**拷贝构造函数**的区别

    初始化变量调用**拷贝构造函数**，给**已存在**的变量赋值调用**`=`**

- `()` **函数调用**运算符

  让**对象**表现得像**函数**一样

  ```cpp
  struct F {
      void operator()(int x) const {
      	std::cout << x << "\n";
      }
  }; // F is a functor
  F f;
  f(2); // calls f.operator()
  ```

#### 类型转换

- 只能被重载成**成员函数**

- 会被**自动**调用

- 调用方法

  ```cpp
  PathName xyz(abc); // 显式
  xyz = abc; // 隐式
  ```

- 禁止隐式转换：**`explicit`**

  ```cpp
  explicit PathName(const string& s);
  ```

- 声明方式

  **声明**中**没有返回值**，实际return的类型与**函数名**相同

  ```cpp
  T::operator double() {
  	return numerator_/(double)denominator_;
  }
  ```

- 编译器总是优先调用**最符合的**

- **类型转换运算符**

  - **static_cast** \<type> (expression)

    只能做**常规**类型的转换，不能移除`const`属性


  ```cpp
  char a = 'a';
  int b = static_cast<int>(a);	// OK
  int* c = static_cast<int*>(&a);	// Error
  ```

  - **dynamic_cast** \<type> (expression)

    type**只能**是**指针**或**引用**！

    **基类**指针**无法**转换成**派生类指针**，否则结果会变成**`nullptr`！**（downcast不安全）

  - **const_cast** \<type> (expression)

    针对关于`const`常量的**指针**、**引用**赋值，可以移除`const`属性

  - **reinterperet_cast** \<type> (expression)

    **不能**reinterpret_cast成**原来的类型**


​		<!--编译器可能会做一些奇怪的检查或者优化【雾】-->

- 补充
  - 为了类型安全，C++**不允许**`void*`隐式转换到其他指针类型


#### 模板

- 关键字：**`template`**

- 模板值可以是**`class/typename`**，**`const`**值

- 可以有缺省值（**默认参数**）

  ```cpp
  template <typename T, int i = 100>
  void swap(T& x, T& y){
      T temp = x;
      x = y;
      y = temp;
      cout << i << endl;
  }
  ```

- 模板函数**不会**使用隐式类型转换

- 与普通函数**共存**（重载）

  - 首先检查参数**精确符合**的函数
  - 然后检查能够符合的**模板**函数
  - 最后检查能够**隐式转换**的**普通**函数

- 实例化

  - 编译器会通过传参**自动推断**类型

  - 也可以**显式指定**

    ```cpp
    foo<float>();
    ```

  - 实例化的时机在**运行时**

- 继承

  - 模板类和非模板类**都可以**互相继承

  - 继承自模板类时需要指定**模板参数**

    ```cpp
    class SupervisorGroup : public List<Employee*>{
        ...
    }
    ```

- 注意

  - **嵌套**时的空格

    ```cpp
    Vector<Vector<double*> > 	// Note space > >
    ```

  - 模板类/模板函数的**声明**和**定义都应该**放在**头文件**(`.hpp`)

#### 命名空间

- 用在**头文件**里

- 声明

  ```cpp
  using namespace MyLib;	// 引入命名空间
  using MyLib::foo;	// 仅引入命名空间的一个函数
  ```

- 在调用的时候可能出现语义模糊的问题

- 可以通过**多文件**添加

- 定义后可以**多次**添加新函数或类



## 标准库

#### STL（Standard Template Library，标准模板库）

##### containers

- pair
- vector
- string
- deque 双端队列
- list 双向链表
- set 自动排序去重
- map
- ...

##### basic algorithms

- sort
- search
- ...

##### iterators

- 提供了一种在不知道细节的情况下，**顺序遍历容器**的方法，是一种一般化的**指针**

- 实例化：在**模板容器类**的命名空间下

  -  `list<int>::iterator li`

     ```cpp
     vector<double>::iterator fbegin, fend;
     fbegin = L.begin();
     fend = L.end();
     fbegin = find(fbegin, fend, 3.0);
     ```

- 要求

  - 可以**自增**（需要重载 `++`）

    ```cpp
    T& operator*() { return ptr val(); }
    ```

  - 可以**解引用**（需要重载 `*`）

    ```cpp
    T* operator->() { return &(**this);}
    ```

- 类型定义：**`typedef`**，**`typename`**

  ```cpp
  template <class T>
  struct myIter{
  	typedef T value_type;
  };
  template <class I>
  typename I::value_type func(I iter)
  {
  	return *iter;
  }
  ```

- 类型萃取：**`iterator_traits`**

  <img src="D:\Leaning\OOP\image-20240102233047783.png" alt="image-20240102233047783" style="zoom:50%;" />

  <img src="D:\Leaning\OOP\image-20240102233242352.png" alt="image-20240102233242352" style="zoom:50%;" />

- 模板特化

  - 全特化

    ```cpp
    class A <int, double, 5> { 
    	...
    }
    ```

  - 偏特化

    ```cpp
    template<typename T>
    class A <int, T, 5> {
    	...
    }
    ```

*** 基于范围的for循环（c++11）**

​	最好是用**引用**

```cpp
for (auto &x: array) {
    cout << x << endl;  
}
```

##### typedef

- 用于简化代码

- C++11: **auto**, **using**

##### 使用自定义模板类

- 可能需要
  - 重载运算符 `=`，`<`
  - 默认构造函数
  - 排序函数（对于有序容器）

### 流

* 不同流声明所属的头文件

​	 <img src="D:\Leaning\OOP\image-20240102202602370.png" alt="image-20240102202602370" style="zoom:50%;" />

- **Extractors**

  - `cin`

  - 函数方法

    - `int get()`

      如果没有剩余字符，则返回`EOF`

    - `getline(char buf , int limit, char delim)`

      **消耗**定界符

    - `int gcount()`

      返回刚读入的字符数

    - `char peek()`

      返回下一个字符，但不消耗它

- **Inserters**

  - `cout`

  - `cerr`

    强制输出刷新

  - `clog`

  - 函数方法

    - `put(char)`

    - `flush()`

      强制流输出

- **Manipulators**

  - Manipulators are objects to be inserted or extracted into/from streams.

  - 需求库 **\<iomanip>**

  - 举栗

    - `setprecision(int)`
    - `setw(int)`

  - 自定义

    ```cpp
    ostream& tab(ostream& out) {
    	return out << 't';
    }
    ```

  - 设置**flags**

    ```cpp
    cout << setiosflags(ios::left | ios showpos);
    cout.setf(ios::hex, ios::showbase);
    ```

- 重载流运算符

  必须声明成**友元**函数，类对象要从**外部**传入，流参数应该为**引用&**

  ```cpp
  // friend
  ostream& operator<<(ostream &o, const Student& s)
  {
      o << s.m_id << "," << s.m_age << "," << s.m_name;
      return o;
  }
  ```

注：

1. `<<`可以输出任意值，包括指针值。
2. `<<`是输出，`>>`是输入，**不是**等价的，`cout`，`cin`的位置也不能变

## 异常处理

- 调用方式

  由底至顶，依层上传，直到有能够handle的层为止，否则**程序终止**；

  发生错误之后的代码段**不会被执行**

  栈中的变量会被正确析构，`throw`的东西在`catch`之后析构，但堆上的不会自动析构

  catch需要有一个错误参数，并且应该是**引用！**

  ```cpp
  void func(){
  	...
  	if(...){
  		throw(someerr);
  	}
  }
  
  try{
  	func();
  }catch(err& A){
      // 注意err可以是任何类型
  	...
  }catch(err& B){
      // 可以raise其他错误
  	...
  }catch(...){
  	...
  }
  ```

- 检测顺序

  按照`handler`句柄出现的顺序，所以**范围越广**的`handle`应该写在**越后面**，`...`**必须**写在最后面

- 标准异常类

  <img src="D:\Leaning\OOP\image-20240103001459608.png" alt="image-20240103001459608" style="zoom:50%;" />

- 动态异常规范

  函数可以声明**可能抛出的异常类型**，或者**一定不会抛出异常**

  ```cpp
  // 可能抛出int类型异常
  void b(int b) throw(int){
  	...
  }
  
  // 一定不会抛出异常
  void a(int a) noexcept{
  	...
  }
  void a(int a) throw(){
  	...
  }
  ```

  **【Warning】**动态异常规范在C++11规范中已经被**弃用**

- 异常用于构造函数失败

  在类构造函数中，如果抛出异常，那么**析构函数不会被调用**，需要**手动释放**

  如果处理异常的析构函数再次抛出异常，那么**程序终止**（`terminate`）

  一种解决方案：*<u>智能指针</u>*

- **善用异常，但不要滥用异常**



## 智能指针

##### 标准库智能指针

- 需要包含**\<memory>**

- **std::auto_ptr\<T>**（在C++11标准被**弃用**）

- **std::unique_ptr\<T>**

  - 独占资源所有权的指针

  - 声明方式

    ```cpp
    auto uptr(std::make_unique<int>());
    std::unique_ptr<int> uptr1(new int);
    std::unique_ptr<int> uptr2 = std::make_unique<int>(20);
    ```

    注：`std::make_unique`在C++14标准中**引入**；

    ​	`std::make_unique<T[]>`在C++20标准中**引入**

  - 只能通过`std::move()`进行移动

  - 可以指向一个数组

    ```cpp
    std::unique_ptr<int[]> uptr = std::make_unique<int[]>(10);
    ```

- **std::shared_ptr\<T>**

  - **共享资源**所有权的指针，涉及资源的计数

  - 声明方式

    与`unique_ptr`类似，使用`std::make_shared`

  - 资源计数

    ```cpp
    // 返回该shared_ptr指向同一个资源的引用次数
    sptr.use_count()
    ```

  - 如果两个`shared_ptr`互相引用，就会**死锁**

- **std::weak_ptr\<T>**

  - 用于解决死锁问题

  - 把`std::shared_ptr`赋值给`std::weak_ptr`，可以用来监测共享资源的使用状况

    ```cpp
    auto sptr = std::make_shared<int>(111);
    std::weak_ptr<int> wptr = sptr;
    ```

  - **不会改变**引用计数，也**不会改变**资源的生命周期

  - **不能**用`*`和`->`直接访问对象，如果对象**没有释放**，可以利用`lock()`作为`shared_ptr`使用

    ```cpp
    *wptr.lock() = 222;
    ```

##### * 自定义智能指针

- 利用`UCPointer`维持引用计数
- 利用`UCObject`进行封装



## 考前提醒

1. 记得delete**<u>[]</u>**
2. 函数定义记得写**返回值**
3. **变量名**，**大小写**看清楚，最好复制

4. 成员函数在外面记得加**参数模板**
5. 成员函数在类声明里**不加**`A::`
