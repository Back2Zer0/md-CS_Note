
> 前言：拖了好久的巨坑。不是很深，打算大火猛烹，一口气翻完星空cpp（effective c++），为开学和另一本蓝书做准备。不过我买的那本星空模糊的不行，盗版的感觉。阅读体验还不如pdf+pad。
>
> 还有就是学习知识后及时复习。除了 温故知新 可为师也 外，还因为人是真的很健忘的 好吧。
>
> 另：由于目前在学英语的转型期，会有很多半英半汉的别扭语句。首先这不是为了~~装逼~~，其次如果阅读体验实在不佳可以换其他的blog，~~不要浪费时间~~。

@[TOC]

### 第一章：Accustoming Yourself to C++



## 1.视C++为一个语言联邦（federation of languages）

- **C++作为一个多重泛型编程语言，斟酌选择使用其某一部分是很有必要的。**

如上所言，今天的C++早已超过了C with class 的要求，它的延伸性非常高——因为它是一个“语言联邦”。联邦包含四部分：

1.  ==C语言== ：C的基础模块。blocks、statemnts、preprocessor、built-in data types、array、pointers等都是和C语言同气连枝。

2.  ==Object-Orientd C++== ：面向对象的编程。即C with class，功能包括继承、封装、多态
3.  ==Template C++== ：cpp的泛型编程(generic programming) 部分。它的背后有TMP(metaprogramming)模板元编程的思路，是很强大的编程范型。 
4.  ==STL==   ：containers,iterators,algorithms 和 function objects 由template程序库 —— STL 统筹配合，共同发挥作用。

在追求高效编程时，这四部分的规则往往有所分歧。这也是为什么要根据需求考虑这四部分，从而达到最佳效率。

> 例如：同样的按值传递(pass by value)和按引用传递(pass-by-reference), 前者对C语言和STL两部分更加使用，而后者在面向对象和泛型编程时则更加高效（尤其Template），这是由于user-defined 构造和析构函数 的存在。

##  2.尽量以const，enum，inline 替换 #define

- 单纯常量，用const或enums 来替代 #define
- 函数宏，改用内联函数(inline)替换 #define

<u>一言蔽之，这是用 **编译器**的处理 替代 **预处理器**的处理。</u>

**原因：**

1. 宏定义的常量由预处理处理，这意味着编译器看不到他。编译器看不到，意味着宏常量一旦出错，在追踪它时会给你造成不小麻烦。（看不到的原因：宏常量没有进入记号式调试器(symbolic debugger)和记号表(symbol table)）

2. 对于浮点常量，define 直接替换可能导致目标码(object code)出现多份浮点数，导致数据冗余。



常量定义式通常被放在头文件内。

**三类场景**

1. class专属常量

   确保这个常量只有一个实体，有必要加上static状态，即``class{static const type}``。

   > 如果需要取地址/编译器有定义式要求，可以在类外提供定义式：`` const typename classname: : 名称``; (这里可以赋值)

   > 如果编译器不允许初值设定,可以放弃static const 改用枚举类型作为补偿：``class{enum {num = 5}; int array_1[num];}``，使其成为记号名称

   Define由于作用域/封装性原因，有很大局限性。

2. enum hack

​      可以用enum实现常量定义，它相比cosnt也更像#define。       

- enum能帮助常量实现阻止其他指针和引用指向的约束。

- eunm不会导致非必要内存分配。

>  取const地址合法； 取enum不合法，取define常量同样不合法。

3. 宏函数

   用template inline 替代define

   ```c++
   #define MAX(a,b) f((a)>(b)?(a):(b))  //选择较大值作为f函数的参数
   ```

   替换为

   ```cpp
   template<typename T>
   inline void MAX(const T&a,const T&b){f(a>b?a:b) ; }
   ```



--------

## 3.尽可能使用const

- 在任何可行的场景声明const提醒编译器，也提醒你自己。

- 编译器强制实施 biwwise constness，编写程序时要利用conceptual constness 灵活实现目的。
- const 和 non-const 成员函数有同样实现时，把两份实现合并成一份，且只允许令non-const调用const。



1. 细节注意：

   * const的位置：在星号 '*' 左边是**指针常量**指向内容不可改变；在右边则是**常量指针**本身不可改变。

   - 迭代器的const用法是静态迭代器：const_iterator。

   - 多数情况，函数返回一个常量值能够防止客户操作出意外。

2. 对于 const 成员函数的操作有两种态度：

   * bitwise(physical） constness: const 成员函数不能修改对象的任何成员变量，一个bit都不行。

   - conceptual（logical） constness: const 成员函数可以通过==mutable==修改对象的**某些**bit。

3. 面对 non-cosnt 和const 两种类型都有需求，而在实现上完全重叠的情况时，合并。

   > 示例：
   >
   > ```cpp
   > Class TextBlock{
   >  public:
   >  const char& operator[](std::size_t position) const
   >  {
   >   #罗里吧嗦一大堆东西
   >   return text[position];
   >  }
   >  char& operator[](std::size_t position) const
   >  {
   >   #罗里吧嗦一大堆东西
   >   return text[position];
   >  }
   >  private: std::string text;
   > }
   > ```
   >
   > 更改为：
   >
   > ```cpp
   >  const char& operator[](std::size_t position) const
   >  {
   >   #罗里吧嗦一大堆东西
   >   return text[position];
   >  }
   >  //const类型的实现，将由non-const 借助调用
   >   char& operator[](std::size_t position) const
   >  {
   >   return const_cast<char&>( //2.调用完成后移除const
   >        static_cast<const TextBlock&>(*this) //1.为*this 添加const
   >        [position];                       //同时帮助operator[]调用const版本
   >   );
   >  }
   > ```
   >
   > 提醒大家：反之用 non-const 去实现的话，const调用是很危险[^2]的。
   >
   > [^2]: const里包含**解const**操作，即const_cast 会打破金身，带来风险。



----

##  4.使用对象前初始化它

- 对内置对象你要手动初始化，克服懒惰。

- 构造函数用成员初值列(member initialization list) 的方法初始化成员变量（初始化顺序对应其声明顺序），

  > 构造函数里用 "=" 赋值 仅仅是赋值操作(assignment) ，与初始化是两码事

- 用 local static 对象(函数内对象)替换 non-local static 来避免 Object_1 使用另一个“未经初始化”的Object_2 来初始化自己。

我们来着重解释下第三点。

> 存在周期和程序一样的对象不多。static、global、in-namespace 的对象都属于此。函数内静态对象称为local static（ local 是相对于函数而言），其余为non-local static。

C++对两个编译单元内的静态对象互相初始化的行为，并无明确定义。这就需要我们在编写程序时留份心。

示例如下：

```cpp
class Server {public:      size_t numDisks() const; }
extern Server tfs; //extern 意味着要到另一个编译单元。 这里tfs还没有初始化
```

这里我们设想Server作为服务端，对提供给客户的tfs对象，在自己的类构造函数里有一个极其优雅的初始化操作。可一旦用户**使用该对象先于它的初始化操作**，就会出现非常Crazy的后果。就像下面这样：

```cpp
class User
{public:  
    Init(params){ //初始化操作
        size_t disks = tfs.numDisks();//在此编译单元使用 tfs 。
    }
}
User temp(params);   //这个初始化调用，可能变成万恶之源。
```

改进：

```cpp
//Server端：
Server:: Server& tfs {static Server tfs; return tfs;}
//先搞个函数初始化好，成为local static对象↑

//User端：
User::Init(params){size_t disks = tfs().numDisks() ;} //这里调用tfs()函数，返回初始化的对象
User& temp(){static User td; return td;} //同Server 替换对象
```



----

### 第二章 : Constructors, Destructors, and Assignment Operators



##  5.了解C++偷偷编写调用的那些函数

- 编译器会为类创建默认构造函数、拷贝构造函数、析构函数 和 拷贝赋值函数。

1. 你自己声明相关函数，不管有参没参，编译器都不会插手这件事情。

2. ```cpp
   class nameobject{	
   public:
   	nameobject(string& name , const T&value):
   		name(this->name),value(this->value){};
   private:
   	string& name;
   	const T value;
   }; 
       string newdog("Persephone");
   	string olddog("Satch");
   	nameobject<int>p(newdog,2);
   	nameobject<int>s(olddog,36);
   	p=s; //What happened in P member？
   ```

   CPP编译器会拒绝编译13行的拷贝赋值。因为你得自己定义。

3. base class 声明拷贝赋值为 private 权限，编译器会拒绝为继承它的子类生成拷贝赋值。

   > 子类继承拷贝赋值操作是为了处理base class元素。这一元素却是私人权限，编译器无能为力。

4. 实际上，这些自动生成的函数只有在被调用且没有实现时，才会被编译器创建出来（给你擦屁股喽）。

----

##  6.明确拒绝不想使用，而编译器自动生成的函数

- 拒绝 Compiler 自动生成的函数：
  1. 你可以将对应成员函数声明为 private 且不予实现；
  2. 继承特定的base class 

copy构造函数和copy assignment（赋值）函数 很容易因为调用被编译器创建出来。但有些场景你会反感这种“帮倒忙”的行为。看看怎么明确拒绝吧。

1. 将copy 函数声明为private，并实现为空。

   > 调用的话会获得连接错误（linkage error）。但member函数和friend函数调用还是可以调用它。

   > 源码中的 ios_base , basic_ios , sentry 都有这种花招出现。

2. 继承特定的 base class 。

   > 这种方法能把连接错误转移到编译期。

   ```cpp
   class Uncopyable{
       protected:
          Uncopyable(){}
          ~Uncopyable(){}
       private:
          Uncopyable(const Uncopyable&);
          Uncopyable& operator=(const Uncopyable&);
   }
   ```

   以 private 类型继承它，不要犹豫。

   > 尝试拷贝它的子类时，编译期会尝试自动生成copy函数，但base class的出现使编译器转向调用 base class 的函数，然后被 private 权限拒绝。 



##  7.为多态基类声明virtual析构函数

- <u>polymorphic（多态）</u>==性质==的 base class 应该声明一个 virtual 析构函数。

  > 即：class 中出现了virtual 就要有virtual析构

- Class==只有==在作为 base class 的需求和具备多态性质时，==才==声明 virtual 析构函数。

  > 这两点是同一问题的两个角度

- 以上两种情况的 virtual 当然可以用 pure virtual 纯虚析构函数 实现

  > 纯虚析构会使该 base class 抽象化。这能督促其 derived class 主动实现完成虚析构。

这里需要一个概念：

> **factory function 工厂函数**：返回一个base class 指针,指向新生成的 derived class 对象。
>
> PS：为了避免泄露内存，返回指针指向的对象必须在heap上，由此使用后务必Delete掉。

**问题场景：**子类继承父类后，子类对象接收了一个父类的指针，并由父类指针删除。这会导致<u>局部销毁</u>而内存泄漏：继承的父类部分销毁而子类部分未被销毁（子类析构函数没有调用）。

**解决方法：**

1. 父类本身的析构函数设为virtual 或 pure virtual。
2. 子类要拒绝继承non-virtual析构函数的父类。

**一些原理：**

> 1. **pure virtual 纯虚析构的运作方式**：最深层派生（辈分最小的）derived class 的析构先被调用，然后逐个调用其 base class。编译器会为子类的析构函数中，创建一个对父类纯析构实现的调用（实现需要自己写）。
>
> 2. **virtual 函数的实现**，要求对象携带一些信息，从而在运行时能 准确调用我们需要的那个 virtual。这份信息由 vptr(virtual table pointer) **指针** 指出。这个小家伙指向一个由**函数指针构成的数组**，即 vtbl(virtual table) 。每个有 virtual 函数的 class 都有一个相应的vtbl。
>
> 所以当对象调用virtual函数， 实际函数取决于该对象的 vptr 所指向的 vtbl。通过 vtbl 编译器会找到合适的函数指针。
>
> 3. **使用 virtual 函数 会带来更大的内存占用。**vptr 作为指针，不可避免会占用内存。而且 vptr 占用的空间和其他语言也不同（C甚至没有 vptr ），所以移植性也不好。



反例：并非所有base class 都有多态性质。像 STL 和 string，它们没有设计用来“通过 base class 接口来处理 derived class 对象”，它们不需要 virtual 析构函数。相信我。

---

## 8.别让异常从析构函数这溜走

- 析构函数不能发生异常。一旦有可能发生异常的情况，析构函数也要能捕捉到，停止异常的传播或结束程序。
- class 应该提供一个普通函数（非析构），帮助使用者对某个操作函数运行期间的异常作出反应。

一个示例：DBConnection( 数据库连接类 )

```cpp
class DBConnection{
    public:
    ...
    static DBConnection create(); //省略不要参数，该函数返回DBConnection对象
    void close();         //关闭联机
}
```

很自然的做法：创建一个管理DBC资源的 class ，在该 class 的析构函数中 调用close。

类似这样：

```cpp
class DBManage{ //DBM 和 DBC 都在服务端。（虚拟场景）
public:
    ...
    ~DBManage()
    {
        db.close();
    }; 
private:
    DBConection db;
};
```

对于用户：

```cpp
//开辟一个block
{DBConn dbc(DBConnection::create()) };
```

当然，以上是建立在调用close()成功的基础上。

如果调用导致异常，析构仍然会传播异常，引出无法驾驭的麻烦。

两个思路解决：

1. close()抛出异常，结束程序 —— 通过调用abort()

   ```cpp
   DBConn::~DBConn()
   {
       try{ db.close();}
       catch(...){
          // 记录失败日志。
           std::abort();
       }
   }
   ```

   

2. 吞下这个异常 —— 记录信息，然后跳过它继续“正常运行”

   ```cpp
   DBConn::~DBConn()
   {
       try{ db.close();}
       catch(...){
           //记录失败日志。
       }
   }
   ```

不过这两种方法不能主动对异常情况做出反应。

这里解决方案是，重新设计DBM接口，使它增加一个主动反应的函数，这个函数的控制权给用户。

```cpp
//class DBManage::
public:
    void close(){db.close(); close = true;}
    ...
    ~DBManage()
    {
        if(!closed)
            try{
               db.close();
            }
            catch(...){
                //记录失败日志；
            }
    }; 
private:
    DBConection db;
    bool closed;
```

所以到最后，用户和“资源类” 成为了双保险，以尽最大努力解决析构函数抛出的异常。



----

## 9.绝不在构造和析构过程中调用virtual function

- Base class 不要在构造和析构过程中调用virtual function，因为这类调用不能与子类里面的实现完成呼应。

> 解决方法：你可以用 non-virtual 函数传参的方法，<u>代替父类函数根据不同子类信息实现类似功能</u>的需求

- Base class 调用的所有函数尽量服从统一约束。否则为了封装性你会牺牲安全性。（错误更加隐蔽）

**原因：**

1. 子类对象创建后，首先进行父类构造函数。在父类成分完成初始化之前，该子类的数据状态始终视为父类（即使是dynamic_cast 或其他 runtime type information 检测也是相同结果）。当然，这个状态下，子类成分完全没有初始化。**Derived class is not derived class until its construction finished.**

2. 根据构造顺序，即使调用 virtual function，编译器也不会选择调用子类函数。因为子类还没有完成初始化。
3. 纯虚函数的存在甚至不能让程序完成连接 —— 是的，它在父类中。



对于上面的“封装性”，也就是第二点，这里给出错误示范：

```cpp
class test
{
public:
    test(){ init(); }
    virtual void vir_func() const = 0;
private:
    void init()
    {
        ...
        vir_func();
    }
}
```

对于这种多态需求的解决方案的示例（构造函数传参）：

```cpp
class base{
    public: 
    explicit base(const string& information){non_vir_func( information )};  
    void non_vir_func(const string& information) const ;} //货真价实的 non-virtual
()
class deprived:public base{
    public:
    deprived(const& content):base( str(content) ){}; //初始化序列，传参给父类构造函数（有参）
    
    private:
    static string str(content);  //这里利用了string初始化,不让他指向为初始化的类内其他成员
}
```

----

## 10. 让 operator= 返回一个 引用(reference to) *this

- 赋值操作符返回一个 reference to *this。

```cpp
class Widget(){
    public:
    ...
        Widget& operator+=(const Widget& rhs)
        {
             ... 
             return *this;
        }
        Widget& operator=(int rhs)
        {
            ...
            return *this;
        }
}
```

这是为了让自定义类型能完成连续操作。

类似  ``x = y = z = 15 ;``

## 11. 在 operator= 中处理“自我赋值”

- 确保当对象自我赋值时 operator = 有正常行为。
- 确定任何函数操作多个对象，且多个对象为同一对象时的情况，行为仍然正确。

```cpp
//错误示范：
Widget& Widget：：operator=(const Widget& rhs)
{
    delete pb;
    pb = new Bitmap(*rhs.pb);
    return *this;
}
//相同对象会导致pb指向一个已经删除的对象。
```

解决思路：

1. identity test

   ```cpp
   Widget& Widget：：operator=(const Widget& rhs)
   {
       if(this == &rhs) return *this;
       delete pb;
       pb = new Bitmap(*rhs.pb);
       return *this;
   }
   ```

   这样做的坏处就是即使判断通过，pb在 new 操作时还会不可避免的遇到异常（内存不足或bitmap的copy构造）。然后pb指向原地址，即已经删除的那块内存。

2. exception safety 

   ```cpp
   Widget& Widget：：operator=(const Widget& rhs)
   {
       Bitmap* pOrig = pb;
       pb = new Bitmap(*rhs.pb);
       delete pOrig;
       return *this;
   }
   ```

   因为pOrig的副本存在，new操作即使失败，pb指向的原地址仍然存在，之后正常delete。

3. copy and swap 

   方法一：

   ```cpp
   class Widget {
   ...
   void swap(Widget& rhs);  //*this 和 参数 rhs 交换
   };
   Widget& Widget::operator=(const Widget& rhs)
   {
       Widget temp(rhs); //rhs数据的副本
       swap(temp);       //副本 temp 和 *this 交换
       return *this;
   }
   ```

   方法二：

   ```cpp
   Widget& Widget::operator=(Widget rhs) //这里是形参，pass by value
   {                                     //直接传过来副件
       swap(temp);                       //副本 temp 和 *this 交换
       return *this;
   }
   ```

   

   ----
1. ## 12.复制对象时勿忘其每一个成分

   - Copy 函数 必须保证复制 本 class 内所有成员变量 和 所有 base class 成分。

     > 1.复制所有local member varaible 2.调用所有 base class 的 适当 的 copy 函数

   - 不要尝试以copy函数实现另一个copy函数。用第三个函数来实现两者共同功能。

   

   1. Copy函数 必须保证复制 本 class 内所有成员变量 和 所有 base class 成分

      - 写好copy函数后，一旦class 内的成员变量有改动（增加），你的copy函数就要随之相应的更改。否则就会导致 copy 副本初始化工作不完整，产生不必要麻烦。
      - 当 derived class 继承父类后，copy 函数不能只考虑自己。因为随同拷贝的还有隐藏的 base class 成分。这一成分需要你同主动实现 derived class的 copy 函数一样，去主动实现 base class 的copy函数。

      > 这里给出一个示例：
      >
      > ```cpp
      > class derive::public base{       //继承父类
      >     public:
      >     derive(const derive& rhs);    
      >     derive& operator= (const derive& rhs);
      >     private:
      >     int value;
      > }
      > derive::derive(const derive& rhs):value(rhs.value)
      > { ...  }
      > 
      > derive&
      > derive::operator=(const derive& rhs)
      > {
      >  ...
      >  value = rhs.value;
      >  return *this;
      > }
      > ```
      >
      > 不够全面。应当如下：
      >
      > ```cpp
      > derive::derive(const derive& rhs):value(rhs.value),base(rhs) //base 成分
      > { ...  }
      > derive::operator=(const derive& rhs) 
      > {
      >  ...
      >  base::operator=(rhs);      //base 成分
      >  value = rhs.value;
      >  return *this;
      > }
      > ```

   2. 不要尝试以coping函数实现另一个coping函数
      - 从copy assignment 调用 copy constructor 不合理：你在构造一个已经存在并完成初始化工作的对象。同时这会影响对象的结构。
      - 从copy constructor 调用 copy assignment 不合理：对一个还没有完成构造的对象赋值，没有任何意义。
      - 毫无疑问，以上两个选择是为了节省代码。这一点可用第三方函数来实现，让两者共同调用该函数。
----
### 第三章：Resource Management

## 13.用对象管理资源

- 秉承“RAII”理念，利用对象初始化资源。

  > RALL：Resource Acquisition is Initialization   资源获取时，即是初始化时。
  >
  > 对象的构造函数能获得资源，析构函数能释放资源。

- RALL classes 有两个经典类：auto_ptr 和 tr1:: shared_ptr 。

  > 前者的复制动作会使被复制对象指向null，后者 copy 能同时指向一个对象

**场景**：你希望在某个函数作用域内获得资源（比如利用factory function)，并在资源使用完成后的某个位置delete掉它。一旦申请和删除资源的中间环节出现问题，删除操作会被完全无视，而造成内存泄漏。

解决：为确保申请的资源能够被及时释放，有两种方法可循。

1. 在对象的析构函数自动调用机制。

   > 这里可以参考下面的 Mutex 对象

2. 利用两个 pointer-like (类指针) 对象,你也可以称之为智能指针。

   > 以下，利用 investment 类来举个例子。        ``class Investment{...}``

   - auto_ptr  :

     ```cpp
     auto_ptr<Investment> pInv(createInv()); //createInv()即 Inv类的工厂函数，返回资源
     ```

     1. 多个 auto_ptr 指向同一对象会导致重复释放，从而产生意料之外的错误。

     2. auto_ptr 这家伙过于忠诚，一旦 auto_ptr 间发生 copy 操作， copy 类指针指向原内容，被 copy 的原指针被设为 NULL 。

        ```cpp
        auto_ptr<Investment> pInv2(pInv1); //p2 exist , p1 null
        pInv1  =  pInv2;    // p1 exist , p2 null
        ```

   - shared_ptr :

     TR1 中的一个类指针，属于RCSP型。

     > RCSP: reference-counting smart pointer
     >
     > RCSP 能提供 garbage collection （回收），
     >
     > 但因为太单纯，不能打破环状引用：两个没有初始化的RCSP彼此互相指向，他们都以为自己心有所属（处于被使用状态）。

3. 实际上没有专门针对 array 类型的 资源管理类指针。

   但你可以考虑Boost里的scoped_array , shared_array ；但我更推荐考虑 STL容器里的 vector。



----

## 14.在资源管理类中小心 coping 行为

- 对于上一条提到过的资源管理类（RAII class），copy 其对象时，一定要同时复制它所管理的资源。

  > 资源的 copy 行为，决定RAII对象的 copy 行为。

- 两个 RAII class copy 行为：1. 拒绝copy 2. reference counting

**场景**：

RAII对象：

```cpp
//这里有一个类型为 Mutex 的互斥器对象
void lock(Mutex* pm);   //锁定pm指向的互斥器
void unlock(Mutex* pm);  //解锁
class lock{ 
public:
    explicit lock(Mutex* pm):mutexPtr(pm)
    {lock(mutexPtr); //锁定，获得资源}
     ~lock(){ unlock (mutexPtr); } //析构自动释放 解锁
private:    
     Mutex *mutexPtr;
}
```

用户端：

```cpp
Mutex m;
// user block
{
    lock ml(&m); //正常锁定
    ...
}
lock ml2(ml); //这种copy要怎样处理？

```

**处理：**

1. 禁止copy ：继承``Uncopyable class``等方法。 [可参考条款6](## 明确拒绝不想使用，而编译器自动生成的函数) 

2. reference-count： 提高该资源的“被引用数”。

   将 Mutex* 改为 ``shared_ptr<Mutex> ``实现。

   > 这里注意：shared_ptr 在引用次数为0时，默认delete指向的目标。
   >
   > 而这里我们的释放操作是 unlock 而非 delete。
   >
   > 还好 shared_ptr 允许指定删除器(deleter),在引用次数为0时，调用这个家伙。
   >
   > deleter 是一个函数/函数对象，在该类指针里是第二参数（可有可无）

   ```cpp
   class lock{ 
   public:
       explicit lock(Mutex* pm):mutexPtr(pm,unlock) //注意第二参数
       {lock(mutexPtr.get());  }    //.get()返回智能指针内部的原始指针副件
       // shared_ptr 的自动析构特性帮我们省略了析构函数。所以让编译器生成吧。
   private:    
        Mutex std::tr1::shared_ptr<Mutex> mutexPtr// Not *mutexPtr;
   }
   ```

3. deep coping：深度拷贝。copy 时 ，面对要 copy 的指针时，生成一个指向副件的指针。

4. 转移底部资源拥有权：auto_ptr 那样，拥有权只存在于一个ptr上。

   

---

## 15.在RAII类中提供对原始资源的访问接口

- RAII应该提供一个取得它所管理的原始资源的方法，来满足API访问 raw resources 的要求。
- 对原始资源的访问方法有显式转换和隐式转换两种方法。显式转换更安全，隐式转换更方便。

> 这里应用上面的 Investment class 例子

对于``int daysHeld(const Investment* pi)``这样一个函数：它的参数是原始类型，可你的对应资源却被RAII类封装起来，如 shared_ptr ,它将无法作为参数传入函数。

这时我们需要一个方法去取得 RAII 类管理的原始资源。

方法：

1. 显式： shared_ptr 和 auto_ptr 都有一个 get() 成员函数，来执行显式转换。他们能返回指向的原始资源。

   > 实际上，智能指针都重载了指针取值操作符 -> 和 * ；
   >
   > 他们会隐式转换到底部原始指针，通过该指针及相应操作符去访问原始类中的成员变量/成员函数。

   ```cpp
   class Font{
   public:
       FontHandle get() const {return f;}
   }
   void changeFontsize(FontHandle f);
   Font f(getFont()); //factory func
   changeFontsize(f.get());  //good
   ```

   总是用显式转换来访问，不免麻烦，但安全明白。

2. 隐式：利用隐式类型转换运算符 ``class: operator type () {return type-static ;} ``

   ```cpp
   class Font{
   public:
       ...
       operator FontHandle() const
       { return f;}
   }
   changeFontSize(f,newFontSize);  //nice
   ```

   隐式转换也会发生在你察觉不到而不希望发生的地方。

   ```cpp
   FontHandle f2 = f1;   //本想 copy Font 对象的
   ```

   f1销毁后，f2会沦入 dangle 状态，处于未被初始化的情况。



> 实际上，RAII class 本身的目的不是为封装性服务，它是为了特殊行为：资源释放而服务的。
>
> tr1::shared_ptr 结合十分松散的底层资源封装，获得了真正的封装效果——你很容易搞到该 class 的原始指针，同时其他的 reference-count 元素则被封装的很严实。
>
> 这是很理想的 class 。

-----

## 16.成对使用new和delete时，要采取相同形式

- new表达式里使用[ ],相应的 delete 也要用[ ]; 

  同理，new表达式里没有[ ], 就不要给对应 delete 画蛇添足加[ ]。

> 这里不讨论new和delete的底层形式。

new的对象，是单一对象还是对象数组？这两个数据类型的内存布局是不一样的。delete和delete[ ]的方式也是不同的。



## 17.以独立语句将newed对象置入智能指针

- 以独立语句将newed对象置入智能指针

  > 不独立的话，当函数同时存在多个实参时，C++的实参执行顺序是随机的。一旦new出的对象和初始化智能指针这两个操作中间出现其他参数调用，并发生异常，就会导致 newed 对象自己留在原地不知所措，内存泄漏。 

```cpp
void processWidget(shared_ptr<Widget> pw, int priority);
processWidget(std::tr1::shared_ptr<Widget>(new Widget),priority());
```

上述函数调用中，分三个过程：

- 调用priority
- 执行 new Widget
- 调用shared_ptr 构造函数

一旦 调用priority 操作插在了new Widget 和 调用shared_ptr构造函数 中间并发生异常中断，就没有好果子吃了。

解决：分离语句，把智能指针和其他参数独立。

```cpp
shared_ptr<Widget> pw(new Widget);
processWidget(pw, int priority());
```



----

### 第四章：Designs and Declaration

## 18.让接口不易被误用，更容易被正确使用。
- 一个好的接口：使用时简单易用，难以误用。

  > 简单易用：接口的一致性、内置类型的行为兼容。
  >
  > 难以误用：建立新类型、限制类型操作、束缚对象值、不要让客户涉足资源管理

- tr1::shared_ptr 支持定制型删除器( custom deleter )。可防范 DDL 问题，同时也能自动解除互斥锁（[条款14](## 14.在资源管理类中小心 coping 行为）)。 

  > DDL问题：即 cross-DLL problem 。object 在一个DDL中 (Dynamic link library) 中被 new 创建，却在另一个 DLL 内被 delete 销毁。
  >
  > shared_ptr 能追踪记录应该调用的 DLL 的 delete。

- 除非有更好的实现，否则就要令你的 types 行为和内置 types 一致。

  >参考 STL 的设计（像统一的 size（））；


可以放心的是，用户总会有各种奇葩操作，你能做的就是巧妙的解决、避免预想中的问题，让接口达成以上八字效果。

> 如果你要求客户必须去做某件事，就是在亲手给自己在埋坑——客户很可能忘记这件事情。

**以下给出两个例子。**

1. 对于一个  Date class ，默认构造参数可能是 ``Date(int month , int day , int year);``

   但用户会把日期顺序搞错，会在填写数据时输入这个星球上根本不存在的日期。

   许多错误可以用一个 new class 解决。可以称之为一个 type system 。

   这里可以用 wrapper types 完成区分。

   ``class Day``  , ``class Month`` , ``class Year`` ,三个特定类型来配合 Date 的默认构造函数。

   在这个类里，可以定义不同函数来限定范围：

   ```cpp
   class Month
   {public:
      static Month Jan() { return Month(1); }
      ...
   }
   //这里用了一个技巧：以函数代替对象，来避免了 non-local static 对象初始化次序问题（详见条款4）。
   ```

2. class 设计者希望用户面对从 factory function 取得 Investment* 指针时，用特定的 "class-delete" (比如 getRidOfInvestment )去代替普通的 delete 。

   > 这时，用户 delete Investment 的隐患就产生了。

   针对这个隐患，你可以利用返回一个自带绑定删除器的 tr1::shared_ptr 来解决。

   把它加入到create Investment()里：

   ```cpp
   std::tr1::shared_ptr<Investment> createInvestment()
   {                                              //这个强转很有必要，让编译器通过
       std::tr1::shared_ptr<Investment> retVal(static_cast<Investment*>(0),
                                               getRidofInvestment);
       retVal = ... ;   //到这里再安排指针
       return retVal;
   }
   ```

---

## 19. 设计 class 犹如设计 type

- class 设计就是 type 设计。定义 new type 前，考虑以下问题。

维护你的 personal type , 意味着 Overloading (重载) function / instruction character (操作符) , 控制内存的分配和归还、定义对象的初始化/终结 等等，都需要在你设计语言类型时来考虑。

对于一个严谨的 class / type ,下面是你必须要考虑的问题：

- [ ] 新 type 的对象如何创建与销毁？

  >考虑各种构造函数和析构函数 、内存分配和释放函数（ ``operator new``  , `` operator new[ ] `` )的设计。

- [ ] 对象的初始化和对象的赋值 的差别?

  > 这决定了 type 的构造函数 constructor 和赋值操作符 assign 的不同行为。

- [ ] type 对象被 pass by value 的实现？

  > 考虑下copy constructor function 。

- [ ] type 的值域？

  > 值域意味着约束条件；约束条件意味着要在你的成员函数、赋值和构造函数中需要做相应的范围检查。
  >
  > 而且不要忘记相应的错误检查工作 —— exception specifications 。

- [ ] type 需要配合某个 inheritance graph 吗？

  > type 继承了 existed class，那这个 type 就势必会受到 base class 的约束（尤其是virtual / non-virtual）；如果你希望其他 class 来继承该 type ，那请你好好考虑该 type 的 inconstructor function  is / is not  VIRTUAL 。

- [ ] 你的新类型允许怎样的类型转换？

  > 如果希望 type 有隐式转换功能，就得写出专门负责转换的函数（不能是 type conversion operators / non-exlicit-one-argument 构造函数）。
  >
  > 如果不希望，将构造函数声明为explicit来避免隐式类型转换吧。

- [ ] 哪些操作符和函数能合理用于 type ？

  > 问题的答案决定了你将为你的 class 声明的函数，和函数是否为 member 型。

- [ ] 需不需要那些编译器生成的默认函数？

  > 需要的话 → 参考：Item 6：禁用那些不需要的默认方法

- [ ] 谁可以访问 type 的成员？

  > 说白了就是权限问题：1.继承关系   2.友元 friends function   3. 一些嵌套关系

- [ ] type 的 undeclared interface （潜在接口）有哪些？

  > 这会关系到 异常/安全性、效率、资源使用(如动态内存/多任务锁定)等。
  >
  > 这些潜在接口将会影响你的实现，有一些约束性的要求。

- [ ] 你的类型有多么通用和普适？

  > 如果这个 type 倾向于作为一大批 type 的地基，祖宗级地位，那就别定义成 class 了，定义成一个 class template 不香么。



---

## 20.多用 const& 替换 值传递 （pass by : reference-to-const replace value）

- 多数情况下用 const& 替换 pass by value

  > 提高效率，同时避免 slicing problem。

- 少数情况是内置类型、STL的迭代器、函数对象。

  > 对于这些数据类型，效率上反而是pass by value 更胜一筹。

**替换原因**：

1. 对一个体量巨大的 class 来说 ，pass by value 意味着生成一个副本，成本为该 class 的构造函数和析构函数，成员变量，继承的父类数据，甚至遇到嵌套的类（哪怕是 string ）还会成倍复制。这个开支远不及一个 const & 来的痛快。
2. pass by reference 能够避免 slicing problem。当一个 derived 类以值传递方式到达一个 base class 参数上，参数副本只会调用 base class :: constructor 。derived 类对象会被切割掉，无影无踪。

**少数情况**（ ？)：

> C++编译器的底层中，references 往往以指针实现出来。这意味着 pass by reference 真正传递的是指针。

1. **内置类型**：如果有个对象是内置类型，值传递的内存占用不一定比一个指针差（甚至效率可能更高）。

2. **STL**：<u>对象小不意味着 copy constructor 就很轻松。</u>多数 STL 容器的体量只比指针大一点儿，但复制 STL 这种东西，还要承担“复制那些指针指向的每一样东西”的责任，相对指针来说还是比较冗余的。

3. **对于用户自定义类型**：即使“内置类型”和“自定义类型”的 underlying representation (底层表述) 完全相同，某些编译器还是会对用户自定义类型特殊对待。

   > 比如：该编译器拒绝把一个只包含 double 数据的对象放入缓存器，却对光秃秃的double类型数据本身敞开大门。这种事情发生时，就有必要用 by reference 消除这种偏见喽。

   另外，user-defined class 的扩展性意味着它随时可能变成一个大家伙。这时还是一个指针，即 by reference 更让我们省心。

   

## 21. 如果一定要返回对象，别妄想返回它的 reference

- 一定要 return 一个 object ，大胆用 pass-by value 吧。

  > - 情况1：
  >
  >   return pointer / reference 指向一个 local static object 的同时，又需要多个这样的 object
  >
  > - 情况2：
  >
  >   指针/引用 指向一个 local stack 对象 或 return reference 指向一个 heap-allocated 
  >
  > 以上两种 Crazy 情况会在下文细说，但你的基本功足够的话，应该也能猜出个所以然。

为了表现两种情况的张力，这里需要一个 classical instance：

```cpp
class Rational{
public:
    Rational(int numerator = 0,
             int denominator = 1);
    ...
private:
    int n,d;   //n: numerator (分子)    d：denominator (分母)
   
    friend const Rational               //const：见 item 3
        operator*(const Rational& lhs
                  const Rational& rhs)
}
```



情况1：

```cpp
bool operator==(const Rational& lhs , const Rational& rhs);
Rational a,b,c,d;
...
if((a*b) == (c*d)){
    //True：乘积相等， Do something boring.
}else{
    //false:乘积不等， Do something boring more.
}
```

此情况能避免调用构造函数，但：

const& 指向一个 static 数据会导致上述 IF 语句==永远为真==。因为 a * b 与 c * d 在重载*的情况下指向了同一地址，也就指向了同一静态变量。这个静态变量先是 a * b ，然后被覆盖为 c * d 。

情况2：

- stack 中的对象在调用 * 乘法运算重载函数后，作为局部变量就已经消失了。const &指向了虚无。

```cpp
const Rational& operator*(const Rational& lhs , const Rational& rhs)
{
        Rational result(lhs.n * rhs.n , lhs.d * rhs.d);
        return result;
}
```

- heap：

```cpp
const Rational& operator*(const Rational& lhs , const Rational& rhs)
{  //这里的 new 要求分配内存之后，用构造函数来初始化。即使 reference 也没能避免
   Rational* result = new Rational(lhs.n * rhs.n , lhs.d * rhs.d);
   return result;
}
```

乍一看很完美。只是需要调用者如履薄冰一样，保证 delete 掉每个对象。

可看看这个应用：

```cpp
Rational w,x,y,z;    w = x * y * z;  //operator*(operator*(x,y),z);
```

这意味着嵌套进去的 operator* 函数返回的指针将被迅速使用，然后在急不可耐中消失，最后内存泄漏。

**所以综上所述：**stack 和 heap 都在 *operator 上摔了个狗啃泥。

---

pass by value 意味着返回值的构造成本和析构成本。但请相信 C++ ，它会允许编译器实现优化，在保证行为相同的情况下尽量改善产出码效率，这样来看，你的程序将在原油行为的基础上，执行的比预期要快一些。

## 22.将成员变量声明为 private

- protected 不能比 public 更有封装性。

- 成员变量声明为 private 。

  > 这意味着客户访问数据的一致性 、 可细微划分访问控制、约束条件的保证、充分的class作者实现弹性。

privated member variable:

1. ==语法一致性== ：成员变量都声明为 private 权限了，访问它只能靠 public 成员函数了。所以所有接口的使用都不用考虑加不加括号的问题——都是函数，加就是了。

2. ==强调权限==   ：public 权限的成员变量意味着每个人都可以对它指手画脚。只有通过函数访问 private member variable 时，才能实现“不准访问”、“只读访问”、“读写访问”等等。

3. ==封装性==：Public 意味着不封装，几乎等价于不可改变（除非以破坏用户码为代价）。

   即使使用 protected 权限，面对子类的狂轰滥炸，你仍然很难改变这个 protected member variable 的一切。所以在这个角度上，它的意义和 public 相差不大（两种权限 : private 和 其他）。

   > 通过函数访问这个成员变量，就好像搞一个黑盒——即使以某个计算替换掉这个成员变量，也没有关 系。将成员变量隐藏在成员函数背后，能为“所有可能的实现”提供弹性: 这可使得成员变量被读或被写时轻松通知其它对象、能够验证class的约束条件及函数的前提和事后状态、能够在多线程环境中运行同步控制等等。

​---
## 23.用 non-member 、 non-friend 替换 member 函数

- Replace **member** **function** by **non-member** 、**non-friend function**.

  > 可以增加封装性、弹性 ( packaging flexibility ) 、扩充性。

这个 item 下，**封装性**是核心。

> 封装性强意味着弹性大：越多的数据被封装，即可访问数据的代码越少，我们也就越能自由地改变对象数据。
>
> item 22 提到成员变量应是 private 。如果不这样做，就意味着无限量的函数可以访问它们，也就毫无封装性可言了。实际上，一个 member function 可以访问 private variable ，还可以任意取用 private function 、 enums 、typedefs 等等。而 non-member 则什么都干不了。这意味着两种函数功能相同时，后者封装性更大。（friend function 的权利和member function 大差不差，所以它的封装性也不大）。

基于对封装性重要性的认识，来看以下例子：

```cpp
clss WebBrowser{
    public:
        void clearCache();
        void clearHistory();
        void removeCookies();
    ...
};
```

对于这样一个类，同时执行以上三个函数的实现方法有两种：

1. member function:

   ```cpp
   void class WebBrowser::clearEverything(){//调用三个函数 ... }
   ```

2. non-member function:

   ```cpp
   void clearBrowser(WebBrowser& wb)
   {
       wb.clearCache();
       wb.clearHistory();
       wb.removeCookie();
   }
   ```

non-member 函数的**封装性**比 member 函数要高，并且有更大的封装弹性，进而存在更低的编译依赖度，增加WebBrowser 的可延伸性。

> 实际上，面向对象守则要求：数据以及操作数据的那些函数应该捆绑在一块。
>
> 这意味着认可 member function is better choice. 这是一个误解。
>
> 因为该守则的目的是要求数据尽可能被封装，与实际选择恰恰相反。

**注意**：强调封装性的 non-member function 同样可以是另一个 class 的 menber 。 上面的 clearBrowser 可以是某 utility class 的一个 static member function 。只要它不是 WebBrowser 的一部分或 friend func，就不会影响封装性。

---

**namespace**：

不过相比成为 static memer func ，更自然的选择是让 non-member function （clearBrowser） 和对应的 class （WebBrowser） 位于同一个 ==namespace== 中。

```cpp
namespace WebBrowserStuff{
    class WebBrowser { ... };
    void clearBrowser(WebBrowser& wb);
}
```

同时你也可以用 namespace 配合 #incluede 实现分离式编程，保证封装性的同时让客户按需所取。

```cpp
//webbrowser.h 头文件内
namespace WebBrowserStuff {
    class WebBrowser{ ... };
    ...  //WebBrowser 核心
}
//webbrowser_bookmarks.h 头文件内
namespace WebBrowserStuff {
    class WebBrowser{ ... };
    ...//书签相关便利函数
}
//webbrowser_cookies.h 头文件内
namespace WebBrowserStuff {
    class WebBrowser{ ... };
    ...//cookie相关便利函数
}
```

As shown above , 需要啥功能，#include 就是了。

**namespace 相比 class 的优势：**

> 1. 移植性：namespace 可跨越多个源码， class 不能。
> 2. 分离性：通常用户用到的功能只有一部分。配合头文件编写同一命名空间的函数，能达到解耦效果。
> 3. 扩展性：用户有其他需求，在命名空间内加个头文件就是了。新函数和旧有函数一样可用且作为一个整体。                    

这里多一句嘴：

> 这也是 C++ 标准程序库的组织方式：不是一个庞大的<C++StandardLibrary>头文件和包含着std namespace 的每一样东西，而是数十个头文件各自负责自己的部分（<vector>,<algorithm>等）。
>
> 这种分割方式是 class function 力所不及的，它必须整体定义。



----

## 24.若所有参数都要类型转换，那就用 non-member 函数

- 如果你需要为某个函数的所有参数进行类型转换，这个函数就必须是 non-member

  > 参数包括被 this pointer 指向的那个隐含参数。

class 支持隐式类型转换本身比较危险，但有些例外情况：像建立一个数值 class ，你往往需要 类似 int 转 double 那样的 隐式转换。

下面以有理数类 Rational class 为例：

```cpp
class Rational {
public:
    Rational (int numerator = 0,
               int denominator = 1);
    int numerator() const;
    int denominator() const;
private:
    ...
}
```

隐式转换的实现形式有 member function 、non-member function 或 non-member friend 函数。

面向对象的角度来看，我们应该把 operator* 扔到 class 内(尤其在你忘掉 item 23 的情况下)，像这样：

```cpp
class Rational {
public:
    ...
    const Rational operator* (const Rational& rhs) const;
}
```

它的实现有其合理性，但也会遇到一些麻烦：

```cpp
Rational ondEighth(1,8);
Rational oneHalf(1,2);
Rational result = oneHalf * oneEighth;
result = result * oneEighth;
//no question ↑
result = oneHalf * 2;  //okay  equal to  : oneHalf.operator*(2);
result = 2 * oneHalf;  //wrong equal to  : 2.operator*(oneHalf);
```

> 式子1都发生了隐式类型转换（没有涉及 explicit constructor function)，
>
> 式子2却没能顺利完成：
>
> 混合式算术中，整数2没有对应的 class ，更没有相应的 operator* member-function 。
>
> (这种情况下，compiler 还会尝试寻找替代的 non-member function，但本例同样没有)
>
> ---
>
> 式子1通过编译时，编译器对2的操作大概如下：
>
> ```cpp
> //编译器知道参数类型Rational虽然和int不符，但只要调用构造函数就有机会完成一切。
> const Rational temp(2);  
> result = oneHalf * temp;
> //如果 Rational constructor 为 explicit ，两式没有一个能通过编译。
> ```

**结论：**

1. 只有当参数位于 parameter list 时，才能正常进行隐式类型转换 —— 类似式子1；

2. 而情况类似 The object that called-member-function belong to 的参数 —— 类似式子2，

   即 this 指针指向的那个隐藏参数是不可以进行隐式转换的。

3. explicit 能保持相关函数的一致性，让式子1和式子2都编译失败。

**更好的解决方式：**non-member function ：允许编译器在每一个实参上执行 implicit type conversion。

师承上例：

```cpp
const Rational operator*(const Rational& lhs,
                         const Rational & rhs)
{
    return Rational(lhs.numerator() * rhs.numerator(),
                    lhs.denominator() * rhs.denominator());
}
```

这对上面的式子都适用。

最后的问题是 operator* 是否有必要成为 class 内的 friend function。

答案是没这个必要（至少在类似情况）：因为该重载函数凭借 class-public interface 就能完成任务，不需要冒着破坏封装性的风险。

> member function 的反面是 non-member function ，而非 friend func 。

## 25.试着写出一个不抛异常的 swap 函数
- 如果 std::swap 对于你的类型来说是低效的，请提供一个 swap成员函数。并确保你的 swap不会抛出异常。
- 如果你提供一个成员 swap，请同时提供一个调用成员 swap 的非成员 swap。对于类（非模板），还要特化 std::swap。
- 调用 swap 时，请为 std::swap 使用一个 using declaration，然后在调用 swap 时不使用任何 namespace 限定条件。
- 为用户定义类型完全地特化 std 模板没有什么问题，但是绝不要试图往 std 中加入任何全新的东西。

**前言**：

> 1. 特化是 **template c++** 的知识。
>
> 2. 模板的全特化和偏特化都是在<u>已经定义的模板基础</u>之上的，**不能单独存在**。
>
> 3. 特化根据模板参数是否全部特化而分作 ==**全特化**== 和==**偏特化**==。
>
> 4. 偏特化： **类模板支持偏特化，函数模板没有偏特化[^3]。**
>
> > (偏特化使用很灵活，可以是限定参数的数据类型，比如int；也可以是限定参数的数据传输方式，比如引用、指针；等等） 
>
> 5. 编译器调用顺序：全特化>偏特化>通用模板

[^3]: **函数有重载特性，因此在某些场景下，特化会产生一些预期之外的结果**。[--->详见](**"http://www.gotw.ca/publications/mill17.htm") 。

**swap本身**：

> swap 函数最初由 STL 引入，已经成为异常安全编程的关键函数， 同时也是解决自赋值问题的通用机制。  

标准程序库中的 swap 算法

```cpp
namespace std{
    template<typename T>
    void swap(T& a, T& b){
        T tmp(a);
        a = b;
        b = tmp;
    }
}
```

> 可以看出，只要 T 支持 coping 就能完成 swap。
>
> 但这种 coping 方式未免有些简单粗暴。

**==类的swap==**（**配合member swap**）

> 有时对自定义类型而言 std::swap 并不高效。 比如采用 pimpl idiom 设计的类中，只需要交换实现对象的指针即可：

```cpp
class WidgetImpl;
class Widget {           // pimpl idiom 的一个类
    WidgetImpl *pImpl;   // 指向Widget的实现（数据）        
public:
    Widget(const Widget& rhs);
}; 
 
namespace std {
    template<>                      // 模板参数为空，表明这是一个全特化
    void swap<Widget>(Widget& a, Widget& b){   
        swap(a.pImpl, b.pImpl);     // 只需交换它们实体类的指针 
    }
}
```

上述代码是不能编译的，因为 pImpl 是私有成员！所以，Widget 应当提供一个 swap 成员函数或友元函数。 惯例上会提供一个成员函数：

```cpp
class Widget {
public:       
  void swap(Widget& other){     //member func
    using std::swap;        
    swap(pImpl, other.pImpl);   
  }
};
//接着我们继续特化std::swap，在这个通用的 swap 中调用那个成员函数：
namespace std 
{
	template<>
	void swap<Widget>(Widget& a, Widget& b) {
		a.swap(a,b);              //调用成员函数
	}
}
```

>  上述实现与STL容器是一致的：**提供共有 swap 成员函数， 并特化std::swap 来调用那个成员函数**。 

**==类模板swap==（配合 non-member function）**

如果 class 涉及到 template ，情况会不大一样。按照上例照猫画虎的写一下：

```cpp
template<typename T>
class WidgetImpl { ... };
 
template<typename T>
class Widget { ... };  //之前的 class
 
namespace std {
    template<typename T>
    // swap后的尖括号表示这是一个特化，而非重载。
    // swap<>中的类型列表为template<>中的类型列表的一个特例。
    void swap<Widget<T> >(Widget<T>& a, Widget<T>& b){
        swap(a,b); 
    }
}
```

> 这里涉及到编译器特点：不允许偏特化 function template 。如果你非要偏特化，可以考虑用 overload 来实现同样功能。

```cpp
namespace std {
    template<typename T>
    // 注意swap后面没有尖括号，这是一个新的模板函数。
    // 由于当前命名空间已经有同名函数了，所以算函数重载。
    void swap(Widget<T>& a, Widget<T>& b){ //这时就需要配合 non-member function 了
        a.swap(a,b); 
    }
}
```

> 又遇到了问题： 这里重载了 std::swap，相当于在 std 命名空间添加了一个函数模板。而 C++ 标准中是不允许添加新的 templates （包括 funtion 或 class 等）。 
>
> C++ 标准中，客户只能特化 std 中的模板，但不允许在 std 命名空间中添加任何新的模板。 上述代码虽然在有些编译器中可以编译，但会引发未定义的行为。 

不再使用特化/重载版的 non-member-func ，将原本的 non-member func 转移到其他 namespace （不一定非得是 global 啊）。

```cpp
namespace WidgetStuff {
    template<typename T> 
    class Widget { ... };
 
    template<typename T> 
    void swap(Widget<T>& a, Widget<T>& b){
        a.swap(a,b);
    }
}
```

> 任何地方在两个 Widget 上调用 swap时，C++根据其 argument-dependent lookup 会找到 `WidgetStuff` 命名空间下的 swap。

那么似乎 class 的 swap 也只需要在同一命名空间下定义 swap 函数，而不必特化 std::swap。 但是！有人喜欢直接写std::swap(w1, w2)，特化 std::swap可以让你的类更加健壮,更加稳定。

> 所以定义 class 专属 swap 后，特化一下 std::swap 吧。

**==编译器调用swap==**

```cpp
template<typename T>
void doSomething(T& obj1, T& obj2){
  using std::swap;           // 使得 std::swap 在该作用域内可见
  swap(obj1, obj2);          // 现在，编译器会帮你选最好的Swap
}
```

从用户角度来看，好像 swap 函数调用以后会面临选择的困难。但C++ 的 ``name lookup rules`` (包括 ``argument-dependent lookup``)能确保找到 global 作用域或 T 所在命名空间的任何 T 专属的 swap。编译器会偏爱专属版 swap 的，std::swap 只是一个最后选项而已。

> 最好不要画蛇添足一个额外修饰符：``std::swap``，这会让编译器只认 std 内的 swap ，**这也是为什么你需要把专属类的 std::swap 进行特化的原因。**

**总结**：

> 当然了，如果默认 swap 实现不会对你的 class / template 有任何影响，就不用再费这番心思了。

1. 如果在效率上确实不足，那就考虑以下事情：

- 提供一个`public swap`成员函数，让它高效的置换你的类型的两个对象值，并且这个`swap`函数绝不能抛出异常。

- 在你的`class`或`template`所在的命名空间内提供一个`non-member swap`，并令它调用上述`swap`成员函数。

- 如果你在编写一个`class`而非`class template`，为你的`class`特化`std::swap`。并令它调用你的`` swap ``成员函数

2. 调用 swap 前，记得包含一个using 声明式，让 std::swap 在函数范围内可见。调用 swap 时不需要添加 namespace 修饰，编译器会帮你完成查找。

3. 成员版 swap 不要抛出异常。你需要帮助 class /class template 提供 exception safety 保障。

   > 非成员版通常不会有这个问题：如 swap 默认版本的 coping 有着不错的稳定性，基于对内置类型的操作也往往不会出现异常。






---

### 第五章：Implementation

## 26. 尽可能延后变量定义式的出现时间

- 尽可能延后变量定义式的出现时间，增加程序清晰度的同时改善效率。

  > 定义变量或类型，构造/析构函数会在 control flow 到达时产生成本，即使你未曾使用。

用例子来看看所谓“延后”的思路：

```cpp
//原版
encrypt(string password); //加密赋值函数
string encryptPassword(string& password)
{
    sring encrypted;
    //if(一些密码要求判断)
    return encrypted;
}
```

- 定义时机延后：需要时再去构造/析构

  ```cpp
  string encryptPassword(string& password)
  {
      //if(一些密码要求判断)
      string encrypted;//放到if后
      encrypted = password;
      encrypt(encrypted);
      return encrypted;
  }
  ```

  目前成本：encrypted 的 default constructor function  ，copy assignment function

- 赋值时机"提前"：尽量在定义时就对变量完成初始化，不要浪费成本

  ```cpp
  string encryptPassword(string& password)
  {
      //if(一些密码要求判断)
      string encrypted(password);//直接初始化
      encrypt(encrypted);
      return encrypted;
  }
  ```

总结：

1. 避免无意义的 default constructor；
2. 延后定义到使用变量（给它初值实参）的前一刻

还有**循环问题**：同一变量在循环使用变量的选择。

两种情形：

- 情形A：(变量类型)定义于循环外

  ```cpp
  Widget w;
  for(int i = 0;i < n; ++i){
      w = i...
  }
  ```

  成本：1 构造函数 + 1 析构函数 + n 复制操作

- 情形B：(变量类型)定义于循环内

  ```cpp
  for(int i = 0;i < n; ++i){
      Widget w(i...);
  }
  ```

  成本： 1 构造函数 + 1 析构函数 

结论：

选择A：

1. class 的赋值成本低于一组 构造+析构 func
2. n比较大
3. 正在处理代码中的 performance-sensitive code

否则选择B。

-----

## 27.少做转型操作

- 尽量避免转型，防止意外；避免 dynamic_casts ，防止拉低效率。

  > 如果实现需要转型，尽量寻找无需转型的替代品

- 转型如果必要，尽量把它隐藏在函数背后

  > 不要让 user 把转型放到自己的 code 里，call it 就是了。

- 尽量用 C++ 风格 转型，去替代旧式转型。

  > C++ style 的 cast 操作更有辨识度，且分工清楚，易于排错。

四种转型：

- const_cast ：cast away the constness, 移除对象常量性。
- dynamic_cast : safe downcasting ，安全向下转型，继承方面。
- reinterpret_cast : 低级转型，不可移植。结果取决于编译器。
- static_cast ：强迫隐式转换，只是不能把 const 转为 non-const。

> 任何一个类型转换都会令编译器编译出运行期间执行的代码。并且由于计算器体系结构中，数据结构的底层表述不同，int 转 double 的过程比我们想象的会复杂一点。

两个注意：

1. 转型操作的移植性很低

   ```cpp
   class Base {...}
   class Derived: public Base {...};
   Derived d;
   Base* pb = &d; //implicit conversion ： Derived* → Base*
   ```

   base 指针指向 derived 对象导致隐式转换，实现的方式是：在运行期施行一个偏移量（offset）到Derived指针上，来取得要求的 Base* 值。

   > 这意味着，一个对象可能有一个以上的地址：Derive* 指向它的地址和 Base* 指向他的地址就有所不同。一旦使用多重继承，这种情况还会频繁发生。
   >
   > 而对象的布局方式和地址计算方式由编译器决定，这就意味着，转型操作的底层实现会在不同平台有所不同，移植性堪忧。

2. 不熟练的转型操作容易想当然，导致程序似是而非

   这里用两个例子来看看 static_cast 和 dynamic_cast 的错误示例：

   - static_cast

   ```cpp
   //需求是：derived类要首先重写base.virtual，
   //并且重写的第一步：调用base.function() 父类某个函数
   class window{ virtual void onResize(){ ... } };
   class specialWindow:public Window {
   public:
       virtual void onResize(){   
        static_cast<window>(*this).onResize(); //this转为window再调用onRize()
       }
   }
   ```

   这个程序的思路很好，只是似是而非的地方是，强转后调用的window::onResize(),其对象是一个**window副本**，而不是真正的父类。如果这个函数内有一些变量操作，就相当于改动副本的数据，再眼睁睁看着它销毁，剩下未经变动的原 base class ，竹篮打水一场空。

   ```cpp
   //更自然的方法：
   virtual void specialwindow:: onResize(){
       window::onResize();
   }
   ```

   - dynamic_cast

     首先，使用这个转换类型，意味着要用一个 base pointer/reference 来执行 derived function 。

     **这种情况有两种思路：使用类型安全容器 或 endue base class with virtual function。**

     > 延续上面的例子，假设 specialwindow 具有独一无二的 blink 功能（以函数呈现）。

     1. 类型安全容器：在容器（as usual to shared_ptr）中存储指向 derived class 对象的指针，直接消除Using base interface to handle object 的需要。

        ```cpp
        //实际上这个方法看起来有点臭屁，因为它把条件直接改了。但它毕竟是强于要类型转换的原需要
        typedef vector<tr1::shared_prt<SpecialWindow> > VPSW;
        VPSW winPtrs;
        ...
        for( VPSW::iterator iter = winPtrs.begin();
              iter != winPtrs.end();
              ++iter)
            (*iter)->blink();
        ```

        这种方法使用类型转换就是下面的样子：

        ```cpp
        typedef vector<tr1::shared_ptr<window> > VPW;
        VPW winPtrs;
        for(VPW::iterator iter = winPtrs.begin();
            iter != winPtrs.end();++iter){
            if(SpecialWindow* psw = dynamic_cast<SpecialWindow*>(iter->get()))
                psw->blink();
        }
        ```

        Anyway，受限于容器类型要求，这两种方法都不能处理“指向所有window派生类”的指针，除非你甘愿用多种容器且都具备 type-safe。

     2. 令父类使用虚函数

        (在 base class 内提供 virtual 函数做各个derive class 做的事）

        ```cpp
        class Window{
        public:
            virtual void blink(){ }
        }
        class SpecialWindow:public Window{...}//虚函数重写不赘述了
        typedef vector<tr1::shared_ptr<window> >VPW;
        VPW winPtrs;
        for(.../*同上1*/  ){ (*iter)->blink();  }
        ```

        这种方法支持多种类型的派生类。

        > 注意必须避免 cascading dynamic_casts ，一连串的强转
        >
        > ```cpp
        > for(VPW::iterator iter = winPtrs.begin();
        >   iter != winPtrs.end();++iter){
        >  if(SpecialWindow1 * psw1 = 
        >     dynamic_cast<SpecialWindow1*>(iter->get())){...}
        >  else if(SpecialWindow2 * psw2 = 
        >     dynamic_cast<SpecialWindow2*>(iter->get())){...}
        >  ...
        > }
        > ```
        >
        > 这种功能用 virtual function 平替即可。

结论：少做转型，能替就替。除非你应对的是 int 转 double 这种无伤大雅、简洁明了的操作。

---

## 28.避免返回handles指向对象内部成分

- 避免返回 handles( references , 迭代器 , pointer) 指向对象内部。

  > 遵守它，提高封装性，让 const member function 没有忘记 const 的初衷。
  >
  > 同时把“空指针”、"空引用"的概率降到最低。

Talk is cheap ,show me the code:

```cpp
class Point{ //坐标点
public:
    Point(int x,int y);
    void setX(int val);
    void setY(int val);
}

class recData{
    point top_left;
    point low_right;
}

class recTangle{
    ...
    Point& upperLeft()const{ return pData->top_left;}
    Point& lowerRight()const{ return pData->low_right;}
private:
    tr1::shared_ptr<recData> pData;
}
```

以上例子非常自然，一个坐标点 - 一个包含两个坐标点的矩形类 - 一个包含矩形类指针和点位接口的类。

实际上这里出现了矛盾：

1. ==封装性==：你把返回点位的函数设为 const 保持封装性的同时，又把私密成员变量毫无保留的 return 了出去。``rectangle . lowerRight().setX(num)`` 类似这种实现会完全破坏封装性。

2. ==dangling handles==：handle本身所指向的对象不存在了，指向了虚空。

   > 这一矛盾在本例中没有出现。简单说，你让 static handle 指向一个局部对象，对象销毁后，handle的状态就是 dangling handles 了。

解决：

1. Add const to the function which return private member variable.

   > This can'd avoid "dangling handles" phenomenon.

2. Avoid using that function about returning varible  as far as possible.

反思：

1. 成员变量的封装性 小于等于 “ 返回其 reference 的函数“的访问级别。

   > 本例中的成员变量表面是 private，实际是public：由 public-member-function导致的。

2. const member function 传出一个引用，引用所指数据和对象自身有关系，而且被存在对象之外，调用者就可以有恃无恐的修改这个数据。

   > bitwise constness 的一个表现。

---

## 29.为“异常安全”努力是非常值得的
- 即使当异常被抛出时，异常安全的函数不会泄露资源，也不允许数据结构被恶化。这样的函数提供基本的，强烈的，或者不抛出保证。
- 强烈保证经常可以通过 copy-and-swap 被实现，但是强烈保证并非对所有函数都可用。
- 一个函数通常能提供的保证不会强于他所调用的函数中最弱的保证。



**异常安全**是指当异常发生时： 不会泄漏资源， 也不会使系统处于不一致的状态。 

通常有三个异常安全级别：基本保证、强烈保证、不抛异常保证：

- **基本保证**。抛出异常后，对象仍然处于合法（valid）的状态。但不确定处于哪个状态。

  > 所有对象内部前后一致。

- **强烈保证** : 成功便成功，失败则恢复到原状态。

  > 如果抛出了异常，程序的状态没有发生任何改变。就像没调用这个函数一样。

- **不抛异常(nothrow)保证**。这是最强的保证，函数总是能完成它所承诺的事情而绝不抛出异常。

  > 内置类型上的操作都提供了 nothrow 保证。

**一个抛出异常的场景**

现在实现一个菜单类，可以设置它的背景图片，提供切换背景计数，同时提供线程安全。

```cpp
class Menu{
    Mutex m;
    Image *bg;
    int changeCount;
public:
    void changeBg(istream& sr);
};
```

changeBg 用来改变背景图片，可能是这样实现的：

```cpp
void Menu::changeBg(istream& src){
    lock(&mutex);
    delete bg;
    ++ changeCount;
    bg = new Image(src);
    unlock(&mutex);
}
```

> 因为 C++ 继承自C，完全避免抛异常是不可能的。比如申请内存总是可能失败的，如果内存不够就会抛出 "bad alloc" 异常。加入 new Image(src) 抛出异常， 那么异常安全的两个条件都会破坏：
>
> - mutex 资源被泄露了。没有被 unlock。
> - Menu 数据一致性被破坏。首先 bg 变成了空，然后 changeCount 也错误地自增了。

通常来讲提供强烈保证是不困难的。首先我们把资源都放到智能指针里去，通常 shared_ptr 比 auto_ptr 更加符合直觉， 这样可以保证资源不被泄露；再调整 ++changeCount 的位置来保证异常发生后对象仍然一致。

一个好的状态变更策略是：**只有当某种事情（比如背景变更）已经发生了，才去改变某个状态来指示它发生了。**

```cpp
class Menu{
    shared_ptr<Image> bg;
    ...
};
void Menu::changeBg(istream& src){
    Lock m1(&m);
    bg.reset(new Image(src));
    ++changeCont;
}
```

智能指针的 reset 是用来重置其中的资源的，在其中调用了旧资源的 delete。

这时如果 new Image 发生了异常，便不会进入 reset 函数，因而 delete 也不会被调用。

> 事实上，上述代码并不能提供完美的强烈保证，比如Image构造函数中移动 了istream& src 的读指针然后再抛出异常，那么系统还是处于一个被改变的状态。 这是一种对整个系统的副作用，类似的副作用还包括数据库操作，因为没有通用的办法可以撤销数据库操作。 不过这一点可以忽略，我们暂且认为它提供了完美的强烈保证。

**copy & swap 范式**

一个叫做 "copy and swap" 的设计策略通常能够提供异常安全的强烈保证。当我们要改变一个对象时，先把它复制一份，然后去修改它的副本，改好了再与原对象交换。 为了更好地示例这个过程，我们将 Menu 的实现改变一下，采用 "pimpl idiom" 把它的实现放在 MenuImpl 中。

```cpp
class Menu{
    ...
private:
    Mutex m;
    std::shared_ptr<MenuImpl> pImpl;
};
Menu::changeBg(std::istream& src){
    using std::swap;           
    Lock m1(&mutex);
 
    std::shared_ptr<MenuImpl> copy(new MenuImpl(*pImpl));
    copy->bg.reset(new Image(src));
    ++copy->changeCount;
 
    swap(pImpl, copy);
}
```

这样我们的操作都是在 copy 上的，发生任何异常都不会影响到当前对象。只有改好了之后我们才 swap 它们。 swap 应当提供不抛异常的异常安全级别。

**连带影响**

```cpp
void Menu::changeBg(istream& src){
    ...
    f1();
    f2();
}
```

因为其它的函数调用例如 f1() 一旦不提供强烈的保证，那么整个函数不可能提供强烈的保证（因为 changeBg 无法修复 f1 造成的资源泄漏和不一致性）。

如果函数只操作局部性状态，就能提供更好的保证。如果函数对非局部性数据有连带影响，不可控因素就会成倍增加。

> 例如：如果f1() 修改了数据库，这个动作发生以后，便没有什么操作可以取消复原 —— 用户可能已经看到了新改动。

<u>这意味着盲目追求最佳安全保证不是一个灵活的方式。</u>

**效率问题**

copy&swap 的关键在于副本，而副本意味着耗用可能巨大的空间和时间，以及费劲心力带来的复杂度。所以退而求其次选择基本保证 并不丢人。

---

> 一个函数的异常安全级别不会高于它调用的所有函数中安全级别最低的那个。这也是为什么我们为什么要为自己的函数提供强烈的安全保证， 一旦某个函数不具备异常安全性，最终整个系统都是不安全的。“一颗老鼠屎，坏了一锅粥。”是也

当你写新的代码或修改原有代码时，考虑如何让它具备异常安全性：

- 一马当先的是“通过对象管理资源( item 13)”，来阻止内存泄漏。

- 之后坚持三个“异常安全保证”中的一种，来贯彻在自己的每一行代码。

- 最后将你的思路写成文档，为用户和维护人员着想。

  > 函数的 ``异常安全性保证`` 是可见接口的一部分，请慎重选择。

> 理论上，你最好选择需求场景下安全程度最高的代码。


---
## 30.了解 inlining 的里里外外

- 将大多数 inlining 限制在**小型、频繁调用**的函数身上。

  > 这能让日后的调试过程和 binary upgradability 更容易，最小化代码膨胀，提高程序速度。

- function template出现在头文件，不意味着就要用 inline 形式。

注意点：

1. inline 帮助编译器免除函数调用成本，对 inline function 的每一个调用以函数本体替换。牺牲的是增加的目标码(object code) 大小。

   > 机器内存有限，inline function 反而会因为太大的程序体积造成额外的 paging 和 降低的 instruction cache hit rate。（当然了，inline 函数本体小的话反而会比函数调用的 object code 更小，产生相反的良性效果）

2. inline 本身可说可不说。编译器可能偷偷声明 inline （比如 一部分class member function），你也可以按你的想法要求编译器这样做（virtual 即使要求也没有用，它需要等待到运行期）。       

   > 一个误区：inline function 在头文件内，template 也在头文件里。但function templates 不一定是 inline。

3. 函数指针的调用通常不会被 inline 允许。

   > 编译器 inline 的过程，首先要生成一个 outlined 函数本体，再通过指针指向那些函数。函数指针有时会扰乱编译器的这一操作。

4. 构造函数和析构函数的 inline 比表面看起来复杂的多。这哥俩不适合 inline 。

   > ```cpp
   > class Base{
   > public:
   >  ……
   > private:
   >  std::string bm1, bm2;
   > };
   > 
   > class Derived: public Base {
   > public:
   >  Derived(){}//Derived构造函数
   >  ……
   > private:
   >  std::string dm1, dm2, dm3;
   > };
   > 
   > ```
   >
   > 看起来是个空构造函数，人畜无害。实际上......
   >
   > 在编译器处理下可能是这个样子：
   >
   > ```cpp
   > Derived::Derived()//概念实现
   > {
   >  Base::Base();
   >  try{ dm1.std::string::string();}//构造dm1
   >  catch(……){
   >      Base::~Base();//销毁base class部分
   >      throw;//抛出异常
   >  }
   > 
   >  try{ dm2.std::string::string(); }//构造dm2
   >  catch(……){
   >      dm1.str::string::~string();//销毁dm1
   >      Base::~Base();
   >      throw;
   >  }
   > 
   >  try{ dm3.std::string::string();}
   >  catch(……){
   >      dm2.std::string::~string();
   >      dm1.std::string::~string();
   >      Base::~Base();
   >  }
   > }
   > //实际代码会更精致复杂
   > ```
   >
   > 由此可知，inline constructor/deconstructor 的不明智（编译器自己可能会直接拒绝 inline）。


5. 你得随时评估 inline 函数的冲击力。

   >inline function 不能随着程序库升级而升级。一旦 inline function 被改变，所有的客户端程序都要重新编译。而 non-inline function 只需要客户端重新连接。

综上结论：慎重 inline。正常流程是先不要 inline （除非契合度太高），把所有函数写完再根据二八法则观察是否有 inline 的需求。

> 80-20经验法则：程序往往将八成执行时间花在二成代码上。
   
## 31.将文件间的编译依存关系降至最低
- 程序库头文件应该以“完整且仅有声明式”的形式存在。

  > 该原则无论涉及 template 都适用。

- 实现依存性最小化的思想：相依于声明式，不要相依于定义式。

  > 基于思想的两个手段：Handle class 和 Interface class 。

**场景：**

```cpp
class Person{
public:
    Person(const string&name ,const Date& birthday,
           const Address& addr);
    string name() const;
    Date theBirthDate;
    Address theAddress;
}
```

显然这个 class 不能编译。你还得提供 composition 数据类型的定义(实现)：string，Date，Address这种。

而这种类型定义通常在头文件里：#include 。

这会导致一个问题：Person 定义文件和 Person 内含文件形成了一种**编译依存关系（compilation dependency）**。在高编译依存关系下，一旦内含在 class 中的类型被重新编译(包括类型定义文件中的头文件改动)，其他所有同一文件中包含的头文件都要进行编译，甚至波及那些使用了 Person class 的文件。这将导致非常夸张的**蝴蝶效应**。

---

降低编译依存关系的一个手段：将对象实现细目隐藏于一个指针背后

> 把 Person 分割为两个 class ，一个只提供接口，一个负责接口后的具体实现。

```cpp
#include<string> //string 不参与前置声明，准确来说它是个内置 typedef ，真正的声明比较复杂。
#include<memory> //不要误会，这里是为了共享指针才引入
//前置声明
class PersonImpl;
class Date;
class Address;

class Person{  //Person 分半:此Person class负责接口声明
public:
    Person(const string&name ,const Date& birthday,
           const Address& addr);
    string name() const;
    Date theBirthDate;
    Address theAddress;
private:      //Person 分半：负责实现的那 个Person class 取名为 PersonImpl
    tr1::shared_ptr<PersonImpl> pImpl; //主角登场了
}
```

> class 只内含一个指针成员，来指向实现类。
>
> 这种设计称为 pimpl idiom （pointer to implementation）

这样一来，Person 客户与类型具体实现分离了，class的实现修改也不需要客户端重新编译，并且保证了封装性。

这就是“接口与实现分离”的妙处。

**结论**：

- 能使用object reference / object pointer 可以完成任务，就不要用 objects 。

  > 可以只靠一个类型声明式就定义出指向这个类型的 reference / pointer 。

- 尽量以 class 声明式替换 class 定义式

  > 要用到某个 class 时，不一定非要这个 class 的定义。
  >
  > ```cpp
  > class Date;
  > Date today();
  > void clear(Date d);
  > ```
  >
  > 声明对应函数而无需定义 Date ，听起来很神奇。
  >
  > 后续在调用相应函数时，对应类型 Date 的定义就需要提前曝光了。
  >
  > 有可能一个函数库有数百个函数声明，用户弱水三千而独取一瓢。这时将 class 定义式从<u>函数声明所在头文件</u>转移到<u>函数调用的客户文件</u>，就可以将<u>“不必要的类型定义”</u>带来的**与客户端之间**的编译依存性去掉。

- 为声明式和定义式提供不同的头文件

  > 声明和定义分离的设计需要两个头文件。一个用于声明式，一个用于定义式。两个文件要保持一致性，一个随另一个改变而改变(此时是否会想起 .h 和 .cpp 两个文件类型)。
  >
  > 承接上述 Data 例子：
  >
  > ```cpp
  > #include "datefwd.h" //Data 在.h头文件里声明而未定义
  > Date today();
  > void clear(Date d);
  > ```
  >
  > > 这里可参考标准程序库 header file <iosfwd>。这个家伙内含各组件声明式。其对应定义分布在其他不同头文件内：<sstream>,<streambuf>,<fstream>,<iostream> 
  > >
  > > 另外，<iosfwd> 也说明了一个事实：这个准则既适用于 template ，也适用于 non-template。
  > >
  > > 可以将**只含声明式**的头文件提供给 template （有些 build environment 允许定义式在**头文件**内，也有一些定义式放在**非头文件**内）。
  >
  > 这里 C++有个关键字：export，允许将 template 声明式和定义式分割在不同文件内。不过支持它的编译器很少，这里不赘述。

---

**Handle classes 和 Interface classes**

Handle classes ：使用 pimpl idiom 的 class。

这种 class 有两种发挥作用的场景：

1. 将该 class 的所有函数转交给对应实现类，并让实现类完成真正工作。

   ```cpp
   #include "Person.h"       // 我们正在实现Person class，所以必须#include其class定义式
   #include "PersonImpl.h"   // 我们也必须#include PersonImpl的class定义式，否则无法调用                           // 其成员函数；
                  // 注意，PersonImpl有着和Person完全相同的成员函数，两者接口完全相同。
   Person::Person(const std::string& name, const Date& birthday,
                  const Address& addr)
   	:pImpl(new PersonImpl(name, birthday, addr))
   {}
   std::string Person::name() const
   {
   	return pImpl->name();
   }
   ```

   > 妙极！Person constructor 通过 new 调用 PersonImpl constructor ，以及 Person::name 函数内调用 PersonImpl::name ，这是很值得关注的地方。
   >
   > **让Person 成为 Handle class 不会改变它本该做的事，只会改变它做事的方式。**

2. 令 Person 成为一种特殊的 abstract base class ，即 ==Interface class==。

   > abstract class 意味着此君往往没有成员变量，也没有构造函数，只剩些 virtual 析构函数和一组 pure virtual 函数。

   一个针对 Person 的 Interaface class ：

   ```cpp
   class Person{
   public:
       virtual ~Person();
       virtual string name() const =0;
       virtual string birthDate() const =0;
       virtual string address() const =0;
   }
   ```

   > 该 class 不能具现出实体。用户通常会调用**类似** derived class 的构造函数来完成具现化。这个函数即 factory class 或 virtual constructor 。它们会返回指针，指向动态分配的对象（这个对象支持 interface class 的接口）。
   >
   > 函数实现范例：
   >
   > ```cpp
   > static:
   > class Person{
   > public:
   > ...  //同上
   > //factory func ↓   
   > static tr1::shared_ptr<Person>  //返回一个shared_ptr ，指向一个新person，
   >     create(const string& name,    //利用参数初始化
   >            const Date& birthday.
   >            const Address& addr);
   >     ...
   > };
   > ```
   >
   > 有了这个 class ，支持 Interface class 接口的具象类(concrete class) 必须被定义出来，而且真正的构造函数需要被调用。这一切都在 virtual 构造函数实现代码（当然包括create）的所在文件内发生。
   >
   > 像这样：
   >
   > ```cpp
   > class RealPerson : public Person  {
   > public:
   >     RealPerson(const std::string& name,const Date& birthday,
   >                const Address& addr) 
   >         : theName(name),theBirthDate(birthday),theAddress(addr)
   >         {}
   >     virtual ~RealPerson()  { }
   >     std::string name() const;
   >     std::string birthDate() const;
   >     std::string address() const;
   > private:
   >     std::string theName;
   >     Date theBirthDate;
   >     Address theAddress;
   > };
   > ```
   >
   > RealPerson 下的 ``Person::create``:
   >
   > ```cpp
   > std::tr1::shared_ptr<Person> Person::create( const std::string& name,const Date& birthday, const Address& addr)
   > {
   >     return std::tr1::shared_ptr<Person>(new RealPerson(name,birthday,addr));
   > }
   > ```
   >
   > RealPerson 演示了从interface class 继承接口，然后实现接口所覆盖的函数。
   >
   > > 当然，实际情况中的 Person::create 定义实现会创建不同类型的 derived class 对象，来完成额外参数值等任务。

**结论**：Handle classes 和 Interface classes 解除了接口和实现之间的耦合关系，从而降低了文件间的编译依存性（compilation dependencies）。当然它牺牲的是：使你在运行期降低速度，同时对每个对象付出更多内存。

- Handle class：
  1. **动态分配的额外开销**：成员函数必须通过 implementation pointer 取得对象数据，从而为访问增加一层间接性。但同时这也导致访问对象消耗的内存必须增加 i-pointer 的大小。
  2. **bad_alloc 异常**(内存不足)：implementation pointer 必须初始化（在 Handle class constructor 内)，来指向动态分配得到的 implementation object 。
- Interface class：
  1. virtual 成本：每个函数都是 virtual ，意味着每次 function-called 都要增加一个 indirect jump 成本。
  2. vptr 指针成本：Virtual 意味着 Interface class 的派生对象必须有一个 vptr，这个指针可能会增加存放对象的内存大小。

> 无论 Handle class 还是 Interface class，一旦脱离 inline 函数都很难有所作为。哥俩用来隐藏实现细节（像函数本体）的设计正好契合 inlined function body 应该被置于头文件的设计。

最后忠告：额外成本不是放弃这二位的理由。像 virtual function 一样，如果比起运行速度/文件大小，你更关注classes 之间的耦合性的话，就用这 Handle class 和 Interface class 代替 concrete class 吧。

###  第六章：Inheritance and Object-Oriented Design
## 32.public 继承意味着 is-a 关系

- public 继承意味着 is-a 关系。

  > 适用于base class 的每一件事也一定适用于 derived class 身上。
  >
  > 因为每一个 derived class object 也是一个 base class object。

public 继承的 derived class object 与其父类对象能发生隐式类型转换，即子类对象可以代替父类对象。但反之不行。两者是一般化和具象化的区别。

**误区**：理想塑模关系和逻辑上的关系可能不大一样：

```cpp
class Bird{
public:
    virtual void fly();
    ...
}
class Penguin:public Bird{
    ...
}
```

企鹅是鸟，鸟会飞，但企鹅并不会飞 —— <u>这是不严谨的表述在逻辑上的硬伤。</u>

**解决：**使用 virtual

1. 编译期解决:更完善的继承体系

   ```cpp
   //细化，使逻辑上更严谨
   class Bird{...}
   class FlyingBird:public Bird{ //利用FlyingBird
       public:
           virtual void fly();
   }
   class Penguin:public bird{
       ...    //企鹅压根就没有飞翔的函数
   }
   ```

   > 实际上，有时需求压根就不需要区分这个鸟会飞还是不会飞。（也许注意点在鸟嘴和鸟翅上）
   >
   > 这表明，没有完美的设计，只有完美的满足需求。根据场景，选择最佳设计。

2. 运行期解决：运行期出错

   ```cpp
   //表述没问题，但针对企鹅，视企鹅本身为一个例外
   void error(const string& msg);
   class Penguin:public Bird{
       virtual void fly(){ error(" Error Happened !");}
   }
   ```

   > 我们知道“好的接口应防止无效代码通过编译”，那就在编译期拒绝他吧。

实际上，”正方形是矩形“的例子也很有代表性。

正方形是矩形没有任何问题，应该用 public inherit 明确表示 is-a 关系。但正方形四条边相同，更新一条边时其余边同步更新的特性和矩形则大为不同。这种差异在 is-a 关系中是灾难性的。也是违反直觉的事情。

> is-a 外，还有 is implementation in terms of 和 has-a 关系。

---

## 33.避免掩盖继承而来的名称

- derived class 名称会覆盖 base class ；这和 public 继承理念格格不入。
- 可以通过 using 声明式或转交函数让被覆盖的名称重见天日。

内层作用域名称会遮掩外围作用域的名称。对于继承来说，则是 derived class 作用域嵌套在 base class 作用域内。

```cpp
class Base {
public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    void mf3();
    void mf3(double);
}
class Derived :public Base{
public:
    virtual void mf1();
    void mf3();
}
```

> 这个实现很离谱。你用 public 继承是 is-a 关系，而这些重载函数又明确划分了子类和父类的界限。
>
> 编译器明面上在阻止这件事（防止建立 new derived class 同时无意间继承远房的 base class）。
>
> 而你却在冒 compiler 之大不讳 (is-a)。

名称遮掩规则没有改变。子类的`mf1() `  , ``mf3()`` 都会彻底覆盖父类的同名函数。

```cpp
Derived d;
d.mf1();  //derived::mf1();
d.mf1(6); //error hiding   虽然参数类型不同，但还是被遮盖了
d.mf3();  //derived::mf2();
d.mf3(6); //error hiding
```

> 需求1：子类无参同名函数覆盖父类无参同名函数，但也要继承父类同名**带参**函数。

解决：``using`` 声明式

```cpp
class Derived:public Base{
public:
    using Base::mf1;//Base class 内名为mf1和mf3的所有东西
    using Base::mf3;//在 Derived scope 全部可见且public
    virtual void mf1(); //定义
    ...//mf3同理
}
d.mf1();   //derived::mf1();
d.mf1(6);  //base::mf1(6);
```

> 需求2：只想继承无参同名函数。

解决：forwarding function （转交函数)

```cpp
class Base { ... //同上}
class Derived:private Base{ //private 继承，这东西类似 has-a 而非 is-a
public:
    virtual void mf1()
    { Base::mf1();} //转交函数，暗自成为 inline
    ...
}
d.mf1(); //调用 derived::mf1，但实际上还是 base::mf1
d.mf1(6); //error
```

> inline 转交函数的另一个用途是为不支持 using 的老旧编译器提供一种方法，将继承名称汇入子类 scope 内。



---

## 34.区分接口继承和实现继承

- 接口继承和实现继承不同。

  > public 继承下，derived class 总是继承 base class 接口

- 区分不同接口，使之各司其职。

  > 1. pure virtual function只负责接口继承。
  > 2. impure virtual function 负责指定 接口继承 和 默认实现继承。
  > 3. non-virtual function 具体指定接口继承以及强制性实现继承(不可更改)。

public 继承 严格来说分为两部分：

1. function interface inheritance  (函数接口继承)
2. function implementation (函数实现)

> 设计一个 class ，往往你有三种可能的需求：
>
> 1. 子类只继承成员函数接口（函数声明）；
> 2. 子类同时继承函数的接口和实现，但希望能 override 这些继承的实现。
> 3. 子类同时继承函数的接口和实现；但不允许 覆写任何东西。

```cpp
class Shape{
public:
	virtual void draw() const = 0;
	virtual void error(const std::string& msg);
	int objectID() const;
};
 
class Rectangle: public Shape{...};
class Ellipse: public Shape{...};
```

1. pure virtual function ：让 derived class 只继承函数接口

   ````
   virtual void draw() const = 0;
   ````

   实际上，你可以为 pure virtual function 提供一份**定义**，作为 base class (Shape) 的实现代码。只需要在调用时**明确指出其 class 名**。这种实现能提供一种机制：<u>为 impure virtual 函数提供更安全的默认实现。</u>

2. impure virtual function ：让 derived class 同时继承接口和实现，并允许 override 。

   这个接口表示：每个 class 可以自由定义自己的错误处理函数；但如果你不想写任何东西，该函数将退回到 base class 来实现(默认版本)。

   > 但这会带来一个问题：
   >
   > 如果 derived class 应该重新定义对应 impure function ，而程序员又忘记了在 class 里写上定义，这就会导致函数在无意间正常运行了 base::function() 。这不是你想要的结果。
   >
   > 解决：切断 virtual func-interface 和 默认实现 的连接
   >
   > ```cpp
   > class Airplane{
   > public:
   > 	virtual void fly(const Airport& destination) = 0;  //采用纯虚函数只提供接口
   >     ...
   > protected:
   > 	void defaultFly(const Airport& destination)
   > 	{
   > 		...  //缺省行为。并且访问方式是protected。
   > 	}
   > };
   > ```
   >
   > 这样，impure 就变成了 pure 。只提供飞机飞行接口。
   >
   > 原来的默认实现以独立函数的姿态出现。
   >
   > ```cpp
   > //希望使用默认实现的 model A
   > class ModelA:public Airplane{
   > public:
   >     virtual void fly(const Airport& destination)
   >     { defaultFly(destination);}
   >     ...
   > }
   > //自己实现的 model B
   > class ModelB:public Airplane{
   > public:
   >     virtual void fly(const Airport& destination)
   >     { ... //自己实现吧   }
   >     ...
   > }
   > ```
   >
   > 当然，粗心的程序员直接剪切code segment 还是会引发问题。但安全性至少高了一些。
   >
   > 有些人反对用不同的函数提供接口和实现。如果实在感觉别扭，你可以利用上文提到过的，pure virtual 的默认实现。
   >
   > ```cpp
   > //希望使用同一函数默认实现的 model A
   > class ModelA:public Airplane{
   > public:
   >     virtual void fly(const Airport& destination)
   >     { Airplane::fly(destination);}
   >     ...
   > }
   > //B不变。
   > ```

3. non-virtual function ：令 derived class 继承函数接口和一份强制性实现

   ```cpp
   int objectID() const;
   ```

   强制性，也可以理解为“invariant greater than specialization”。它绝对不能被重新定义。

两个常见错误：

1. 将所有函数声明为 non-virtual

   > 首先 non-virtual 析构会带来问题。其次你没必要太担心 virtual function 的效率成本。任何 class 打算作为 base class，都得有好多 virtual 。不要忘记80-20法则。

2. 将所有成员函数声明为virtual

   > 这属于矫枉过正。对于某些函数，它就是不该被重新定义。要有坚定的立场。



---

## 35.考虑virtual函数以外的选择

- virtual 函数的替代方案有 NVI 方法和不同形式的 Strategy 设计模式。

  > NVI（non-virtual Interface） 方法属于特殊形式的 Template Method 设计模式

- 实现从 member function 更换为 class 外部函数，会导致无法访问 class 内部的 non-public 成员。

  > 除非你愿意降低封装性使用 friend

- tr1::function 对象的行为像一般函数指针，但它能接纳与 target signature 兼容的所有 callable entities 。

  > 兼容性强大：函数参数和函数返回值的隐式转换。

```cpp
class GameCharacter{
public:
    virtual int healthValue() const; //省略实现
    ...
}
```

计算游戏角色的血量。血量有不同计算方法，而同一方法又可能适用于不同人物。

virtual 当然是很好的选择。但下面会提供几种思路供参考，来**尽量**避开 virtual func 的使用。

1. **NVI 手法实现 Template Method 模式**

   > 该方法主张 virtual 函数应该几乎全是 private ，最好保留 Healthvalue 为 public 成员函数，但让它成为 non-virtual ，并调用一个 private virtual 函数。

   ```cpp
   class GameDCharacter{
   public:
       int healthValue() const // non-virtual : a wrapper
       {
           ... //事前工作
           int retVal = doHealthValue();  //核心工作 
           ...//事后工作
           return retVal;
       }
       ...
   private:
       virtual int doHealthValue() const //被转移到此
       {
           ...     //默认算法，计算健康指数
       }
   };
   ```

   客户能通过 public non-virtual 成员函数间接调用 private 函数，即NVI（non-virtual interface）手法，是Template Method 设计模式的表现形式之一。

   这个 non-virtual 可称之为 virtual 函数的 wrapper 。

   > 优点：
   >
   > 1. 事前工作和事后工作有很大的弹性：能够设置不同条件场景。
   >
   > 2. NVI 手法允许 derived class 重新定义 private virtual 函数（实现）。
   >
   >    > ”重新定义“ 意味着函数如何完成 ，由 derived class 决定。
   >    >
   >    > “调用 virtual ”意味着函数何时被完成，由 base class 决定。
   >
   > 注意：
   >
   > NVI手法下，没必要 virtual 函数必须是 private。有些场景下 derived class 在 virtual 函数实现内必须调用 base class 的对应同名兄弟。为了使之合理， virtual 就只能是 protected 了。
   >
   > 遇到多态 base class 的 析构函数，你甚至得是 public。

2. **基于 Function Pointers 的 Strategy 模式**

   NVI手法只是对 public virtual 函数的一个修饰。Strategy 设计将使 人物的healthvalue 与人物本身毫无关系，相互独立。

   > 可以猜出来：每个人物的构造函数接收一个函数指针，指向哪个健康计算函数，就是哪个喽。

   ```cpp
   class GameCharacter;// foward declaration
   //主角之一：健康计算的默认算法
   int default_Health_Calc(const GameCharacter& gc);
   class GameCharacter
   {public:
       typedef int (*Health_calc_func)(const GameCharacter&);
       explicit GameCharacter(Health_calc_func hcf = default_Health_Calc)
           :healthFunc(hcf){}           //默认参数↑
    
       int healthValue() const      //返回血量（返回前通过指针调用一次函数）
       {return healthFunc(*this);}
       ...
    private:
        Health_calc_func healthFunc; //主角之二：函数指针
   }
   
   ```

   它和普通 virtual function 相比，弹性很大：

   - 同一人物类型的不同对象，仍然可以有不同健康计算函数：

     ```cpp
     class GameCharacter { //同上 };
         
     int loseHealthQuickly(const GameCharacter&);
     int loseHealthSlowly(const GameCharacter&); //不同血量计算
         
     class EvilBadGuy :public GameCharacter { //构造函数同character：作为接口插入函数指针
         explicit EvilBadGuy(Health_cala_func hcf = default_Health_Calc)
               :GameCharacter(hcf) {}
     };
     
     void test()
     {
         EvilBadGuy ebg1(loseHealthQuickly);  //相同类型人物，不同血量计算
         EvilBadGuy ebg2(loseHealthSlowly);
     }
     ```

   - 已知人物的健康计算函数，可以在运行期变更。

     ```cpp
     GameCharacter::set_Health_Calc 
     //作为接口，随时替换定义。
     ```

   

   ---

   > 健康计算函数不再是 GameCharacter **继承体系内**的成员函数。
   >
   > 这些计算函数没有访问 class 的内部 non-public 成分。

   问题：如果你需要 class 内的 non-public 信息进行精确计算，就会产生问题。

   > 当你需要平替 class 内的成分为 class 外部的等价成分（如 non-member non-friend 函数 / 另一个 class 的non-friend 成员函数）时都会引发争议。

   解决：弱化class封装

   > 1. non-member → friend
   > 2. 为实现的某一部分提供 public 访问函数

   它的优点(灵活性)相对缺点（封装性潜在降低），需要看你的使用场景。

3. **基于tr1::function 的Strategy 模式**

   哎呀，函数指针看起来终归有些苛刻而死板。改用函数指针为 tr1::function 对象，能舍去很多约束。

   > 这东西兼容性很强。只要 signature 和需求端匹配，其他的都可以商量变通。

   ```cpp
   class GameCharacter;
   int defaultHealthCala(const GameCharacter& gc);
    
   class GameCharacter {
   public:
       ...       //其余部分同上
       //只是将函数指针改为了function模板，其接受一个const GameCharacter&参数，并返回int
       typedef tr1::function<int(const GameCharacter&)> HealthCalcFunc;
       //以 int 为例，返回兼容int的everything，接收兼容GC的everything （隐式转换）
       
       explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc)
       :healthFunc(hcf) {}
       
   private:
       HealthCalcFunc healthFunc;
   };
   ```

   仅仅是 private 的载体改变了：函数指针 →tr1::function 。

   下面看看它发挥的作用。

   ```cpp
   class GameCharacter { //同上};
   class EvilBadGuy :public GameCharacter {
       //载体改变，其他如假包换。
   };
   class EyeCandyCharacter :public GameCharacter {  
       //构造函数类似EvilBadGuy 
   };
    
   //下面是不同类型的健康计算函数，来突出 tr1::function 的兼容性
   //1.类型不同的普通函数 
   short calcHealth(const GameCharacter&);
   
   //2.函数对象，用来计算健康指数
   struct HealthCalculator {
   int operator()(const GameCharacter&)const {}
   };
   
   //3.一个类：提供一个成员函数，用以计算健康
   class GameLevel {
   public:
       float health(const GameCharacter&)const;
   };
   
   void test()
   {
       //人物1，其使用calcHealth()函数来计算健康指数
       EvilBadGuy ebg1(calcHealth);
    
       //人物2，其使用HealthCalculator()函数对象来计算健康指数
       EyeCandyCharacter ecc1(HealthCalculator());
    
       //人物3，其使用GameLevel类的health()成员函数来计算健康指数
       GameLevel currentLevel;
       EvilBadGuy ebg2(std::tr1::bind(&GameLevel::health, currentLevel, _1));
       
   }
   ```

   > 人物1,2的那种隐式转换你已经司空见惯。下面我们来看 ebg2 —— 人物3
   >
   > GameLevel::health 宣称自己接收一个参数。但它实际上有一个第二参数：this指针(GameLevel 类型)
   >
   > GameCharacter 构造函数真正只接受单一参数。
   >
   > 
   >
   > 如果你使用 GameLevel::health 作为ebg2 的健康计算函数，你得说服 GameCharacter 接受它：
   >
   > 让GameLevel::health 可以通过绑定，让它不接收两个参数↓
   >
   > 将 currentLevel 绑定为GL对象，让他在每次“GameLevel::health被调用来计算 ebg2健康”时使用。
   >
   > 这就是 tr1::bind 的工作了 —— 它表明：ebg2 的健康计算函数应该总是以 currentLevel 作为 GL对象。
   >
   > (OK,这里 cuttrentLevel 作为一个 GameLevel 载体，作用被具现化、框定了)
   >
   > _1: 当为ebg2 调用 GameLevel::health 时系以 cuttrentLevel 作为 GameLevel 对象。

   综上，这种兼容性还是很吸引人的。

4. **传统古典 Strategy 模式**

   将健康计算函数做成一个分离体系中的 virtual 成员函数。

   - UML图如下，意义如下：

     - GameCharacter是一个继承体系的根类，其派生类有EvilBadGuy、EyeCandyCharacter
     - HealthCalcFunc是一个继承体系的根类，其派生类有SlowHealthLoser、FastHealthLoser
     - 每一个GameCharacter对象都内含一个指针，指向于一个来自HealthCalcFunc继承体系中的对象

     ![img](https://i-blog.csdnimg.cn/blog_migrate/960ce5b961642cd78df40f0a5b90bd79.png)

   - Code：

     ```cpp
     class GameCharacter;
     class HealthCalcFunc { //计算健康指数的类 
     public:
         virtual int calc(const GameCharacter& gc)const {} 
     };
      
     HealthCalcFunc defaultHealthCalc;
     
     class GameCharacter {
     public:
          explicit GameCharacter(HealthCalcFunc* hcf = &defaultHealthCalc)
     :pHealthCalc(hcf) {}
      
     int healthValue() {
         return pHealthCalc->calc(*this); 
     }
     private:
         HealthCalcFunc* pHealthCalc;
     };
     ```

     > 这个设计提供：“将一个现成的健康算法纳入使用”的渠道 —— 为 HealthCalcFuc 继承体系添加一个 derived class 即可。

   **总结：替换 virtual 函数的四中方法**

   1. 使用non-virtual interface 方法，以 public non-virtual 成员函数 包裹 较低访问性(private / ptotected) 的 virtual 函数。

   2. 将 virtual 函数替换为“函数指针成员变量”

      > 这是 Strategy 设计模式一种表现形式。

   3. 将 virtual 函数替换为 tr1::function 成员变量。极强的兼容性。

      > 这也是 Strategy 设计模式一种表现形式。

   4. 将继承体系内的 virtual 函数替换为另一个继承体系内的 virtual 函数。

      > Strategy 设计模式的传统设计手法。

   

   ----

 ## 36.绝不重新定义继承的 non-virtual 函数

- 绝不重新定义继承的 non-virtual 函数

看看例子吧：

```cpp
// 正常情况
class B{
public:
    void mf();
    ...
};
class D:public B{...};

D x;
B *pB = &x;
pB -> mf();//经由该指针调用	B::mf
D *pD = &x;
pD -> mf();//经由该指针调用B::mf

//重新定义
class B{
public:
    void mf();
    ...
};
class D: public B{
public:
    void mf();     //遮掩了B:mf;
    ...
}
D x;
B *pB = &x;
pB -> mf();//调用B::mf
D *pD = &x;
pD -> mf();//调用D::mf
```

**原因：**non-virtual 都是静态绑定（statically bound），而virtual 函数是动态绑定（dynamically bound）

>静态：pb 作为 pointer to B ，调用的 non-virtual 函数永远是 B 所定义的版本。
>
>动态：如果 mf 是 virtual function ，不论是通过 pointer-to-B或 p-to-A ,都是调用 D::mf，因为两者指向的都是 D 对象。

这样一来，一个D对象既可能表现 D object 行为，也可能表现 B object 行为。决定因素不在对象自身，而在指向该对象的指针的类型。

**理念冲突：**public 继承意味着 is-a 关系；non-virtual 意味着 invariant greater than specialization 。

- 适用于 B 对象的每一件事，也适用于 D 对象，因为每个 D对象都是一个 B对象。

- B 的子类一定继承 mf 函数接口的实现，因为 mf 是一个 non-virtual

  > B 此时却要覆盖掉 mf

两者冲突。

----

## 37.绝不重新定义继承来的默认参数

- 绝不重新定义继承来的默认参数

  > 默认参数值都是静态绑定。 virtual 函数是动态绑定。

如 item 36所言，继承 non-virtual 而重新定义并不合理。所以只需讨论 virtual 函数问题。

下面解释下静态/动态绑定：

- [ ] statically bound:被声明时所采用的类型。

```cpp
//一个描述几何形状的class
class Shape{
    public:
    enum ShapeColor{Red,Green,Blue};
    //所有函数都必须提供一个函数用来绘制自己
    virtual void draw(ShapeColor color=Red)const=0; //red
    ...
};

class Rectangle:public Shape{
    public:
    //注意这里对缺省参数的改变，糟糕透了
    virtual void draw(ShapeColor color=Green)const; //green
    ...
};

class Circle:public Shape{
    public:
    virtual void draw(ShapeColor color)const; //不指定参数
}
```

>    - **用户以对象调用时必须指定参数** : 因为静态绑定下，这个函数并不从base继承缺省参数值
>    - **用户以引用调用时可以不指定参数** : 因为动态绑定下，这个函数会从base继承缺省参数值

- [ ] dynamically bound：

  ```cpp
  Shape* ps;				//静态类型为Shape*，没有动态类型
  Shape* pc=new Circle;	//静态类型为Shape*，动态类型为Circle*
  Shape* pr=new Rectangle;//静态类型为Shape*,动态类型为Rectangle*
  //注意上述三个指针，不论他们真正指向什么，他们的静态类型都不会变
  
  //动态类型则可以在程序执行过程中改变（通常是赋值引起）
  ps = pc;
  ps = pr;
  //动态类型改变
  ```

  virtual 函数则是动态绑定：调用一个 virtual 函数，调用哪一份 function code ，取决于 ``*this`` 的动态类型。

  **问题：**

  ```cpp
  // Shape* pr=new Rectangle;
  pr->draw();
  ```

  pr 的动态类型为 Rectangle* ，调用的应是 Rectangle 的 virtual function draw。但由于 pr 的静态类型是

  Shape* ，所以默认参数值来自 Shape class 而非 Rectangle class  ，参数为 Red 而非 Green。

  <u>这不是我们想要的结果。</u>

  > 这是 C++ 为了保证运行期效率的表现。默认参数是动态绑定的话，编译器必须有某种办法在运行期为 virtual 函数决定适当的参数默认值。这会比目前的“编译器决定”机制更慢更复杂。

  **解决：**

  单纯的让子类 virtual function 和 父类一致，会导致代码重复和相依性（with dependencies），牵一发而动全身。

  更好的解决方法可参考NVI手法：

  ```cpp
  class Shape {
  public:
  	enum ShapeColor { Red, Green, Blue };
  	void draw(ShapeColor color = Red) const      // 如今它是non-virtual
  	{
  		doDraw(color);                           // 调用一个virtual
  	}
  	...
  private:
  	virtual void doDraw(ShapeColor color) const = 0;  // 真正的工作在此处完成
  };
  class Rectangle: public Shape {
  public:
  	...
  private:
  	virtual void doDraw(ShapeColor color) const;     // 注意，不须指定缺省参数值
  	...
  };
  ```

  > 令 base class 内的 public non-virtual function 调用 private virtual 函数，后者可被 derived class 重新定义。
  >
  > non-virtual 绝对不能被子类覆写，所以该设计理论上让 draw function 默认参数始终为 Red。


---
## 38.Model "has-a" or “is-implemented-in-terms-of”through composition

- 复合(composition)的意义和 public 继承完全不同
- 在 application domain ，复合意味着 has-a 。在 implementation ，复合意味着 is-implemented-in-terms-of （根据某物实现）。

复合表示一种类型关系：某种类型的对象内包含另一种类型的对象。

```cpp
//composition (很多同义词： layering , containment , aggregation , embedding)
class Address {...};
class PhoneNumber {...};
class Person {
public:
    ...
private:
    string name;
    Address address;
    PhoneNumber voice Number;
}
```

如同 public 意味着 is-a 一样，复合意味着两种关系：has-a 和 is-implemented-in-terms-of 。这两种关系对应两种 domains 。

> application domain: 人，汽车，视频画面等。
>
> implementation domain: 缓冲区，互斥器，查找树等。可以理解为应用域的微观实现细节。

- has-a 和 is-a 的区分：

  Person class 有地址，电话号码。但显然不是 is-a 所指的人是个电话，人是个号码这样。即 has-a。

- is-implementation-in-terms-of 和 is-a：

  “根据某物实现出”：举一个例子，你把自行车改造成了巡洋舰(这里忽略改造细节)，自行车作为巡洋舰的主体，实现了巡洋舰的功能。这是 is-i-in-terms-of 的效果了。这时你不能说这艘巡洋舰是自行车，反过来更不行。

  > 这里强调一个 "reuse" 效果，表现就是 composition 复用。

  真正的例子：你要手搓一个 set template ，秉承就地取材（根据某物实现）的精神，你准备以 list 容器为主体。

  ```cpp
  template<class T>       //list 应用于 set
  class Set{
  public:
      bool member(cosnt T& item) const;
      void insert(const T& item);  
      void remove(const T& item);
      ...
  private:
      list<T> rep;    //主体，承载 set 类数据
  }
  //实现
  template<typename T>
  bool set<T>::member(const T& item)
  {
      return find(rep.begin(),rep.end(),item) != rep.end();
  }
  ...
  ```

  > 从这个例子中初见端倪，is-imp-in-terms-of 强调的是实现域，has-a 强调的是应用域。

## 39.明智审慎的使用 private 继承

- private 继承意味着 is-implemented-in-terms-of 。

  > 这通常比复合的级别低，更适用于子类需要访问 protected 父类成员，或者重新定义继承的 virtual 函数时。

- private inheritance 可以完成空白基类(empty base)优化。

  > 这对“对象尺寸最小化”的开发需求来说很重要。

1. **private inheritance 不意味着 is-a**：

   item 32 呈现 is-a关系时(public inherit)提到编译器在必要时允许 Student 类偷偷转换为 Person 。而 private 继承则不行。

2. **private inheritance 意味着所有继承成员是 private** 。

对于 private 继承的意义：

**private inherit 纯粹是一种实现技术**：作为实现细节，由子类继承来使用父类中的现成的某些特性。

> 这种现成特性的利用就是 is-implementation-in-terms-of 。
>
> 复合的意义也是这样，但尽量用复合，必要时才用 private inheritance：即当牵扯到 protected member 和virtual function 这些麻烦家伙时。还有一种空白基类的优化技术要利用 private 继承。

例子：Widgets class，一个窗口类

```cpp
//Widget 需要借助 Timer 记录成员函数调用次数
class Timer{
public:
    explicit Timer(int tickFrequency);
    virutal void onTick() const;
    ...
};
```

private inherit：

```cpp
class Widget:private Timer{
private:                      //public 破坏封装性，误导用户
    virtual void onTick() const;
    ...
}
```

composition：

```cpp
class Widget {
private:
    class WidgetTimer:public Timer{
        public:
             virtual void onTick() const;
    }
    WidgetTimer timer;
}
//这个设计涉及 public inherit ， composition，同时导入了新 class
```

两个注意点：

1. 封装性让我们希望 derived class 不能修改 base class 里的关键函数（onTick）

   > 但 derived 可以重新定义virtual func，private继承也不行。
   >
   > 而如果你用上面的 composition 来实现，这一要求将得到实现。
   >
   > (Allocate a new class (called "WidgetTimer" ) public inheriting Timer as private member in Widget .)
   >
   > (Then , derived class inheriting Widget can't modify that virtual function .)

2. 降低编译依存性

   >Widget 继承 Timer ，编译 Widget时 Timer 定义就必须可见，于是就不可避免的``#include<Timer.h>``。
   >
   >如果 WidgetTimer 定义放到 Widget 外，取而代之的是把指向定义的指针放在 Widget 里，这样你就只需要一个简单声明和一个指针就够了。
   >
   >这样的解耦性(decoupling)操作非常重要，尤其对于大型系统。

----



神秘主角：empty base 

> 既然是 empty base ，就不要有任何东西。

按理说它不该占用任何内存。而 C++ 有一个准则：<u>==独立==对象都必须有非零大小。</u>

like this：

```cpp
class empty {};
class Hold_An_Int{
private:
    int x;
    Empty e;
}
```

实际情况是，sizeof（Hold_An_Int）> sizeof（int）。

> 多数编译器中，sizeof(empty) = 1。
>
> 面对大小为0的独立对象 empty，c++ 会偷偷插入一个 char 到空对象里。
>
> 但这样一来，齐位需求（alignment）会要求编译器再加入一些 padding，扩大到一个 int 等。

不过既然要求是独立对象，那就不独立好了，就可以为0了。

```cpp
class Hold_An_Int:private Empty{
private:
    int x;
}
```

sizeof（Hold_An_Int）= sizeof（int），效果达到了。这就是所谓的**EBO（empty base optimization）**。

> EBO 一般只在单一继承下才行。
>
> 现实情况下，EBO的 empty class 并不是真的 empty。里面往往有一些 unary_function  ,   binary_function 。这些东西是用户自定义的函数对象，用来被继承（是否想起了 is-implementation-in-terms-of）。

说完EBO，请允许我在最后多一句嘴：composition 混合了 public 继承和复合的设计看起来很复杂，但可行性依然非常高。相对于这个思路，“明知而审慎”的使用 private inherit 是值得你考虑的事情。

---


## 40.明智审慎的使用多重继承(judiciously)

- 多重继承比单一继承复杂。

  > 这可能导致歧义性，以及对 virtual 继承的需求。

- virtual 继承会增加大小、速度、初始化复杂度等成本。

  > 如果 virtual base classes 不带任何数据

- 多重继承适用于特定的应用场景。

  > 例如" <u>public 继承某个 interface class</u> "  和  "<u>private 继承某个协助实现的 class</u>" 两项组合

下面来逐个分析：

1. 继承带来的歧义性

   - 同名函数

     有着相同函数``checkout()``不同实现的两个 class A和B，被一个 derived class 多重继承（同时继承两个 class ），该子类调用``checkout``时，编译器会考虑两个函数对调用的匹配程度（类似 resolving 重载函数调用那样），该例两个调用匹配程度相同。为此你必须指出调用哪个函数：``C.A::checkout()``。

   - 钻石继承 (菱形继承)

     ```cpp
     class File{...};
     class InputFile:public File{...};
     class OutFile:public File{...};
     class IOFile:public InputFile,
                   public OutputFile
                   {...};
     ```

     > 继承体系出现一个以上的相同路线（上例有两条），就要考虑：是否打算让 base class 的成员在每一条路径都被复制一遍。
     >
     > 如果回答为是，就意味着收束子类(``IOFile``)将有两份一模一样的代码段（成员变量，成员函数）。逻辑上来说，IOFile object 只需要一份就够了，不该重复。

     C++默许了两种方法：

     1. 重复几份：如上例一样。

     2. 只保留一份：virtual 继承：

        ```cpp
        class InputFile:virtual public File{...};
        class OutFile:virtual public File{...};
        ```

        

2. virtual 的成本

   > 场景需求当然是 virtual 继承多一点。但这不意味着你碰到多重继承便要 virtual 。

   - virtual 继承对象 往往有更大的体积，访问 virtual base class member-variable 时，速度也更慢。

   - virtual base class 初始化规则比起 non-virtual 更为复杂且不直观。

     > 1.derived class 若选择 virtual inherit base class 而需要初始化，必须清楚知晓 base class 的所有相关成分，不管继承有多远。
     >
     > 2.新的 derived class 加入继承体系中，也就意味着必须承担 base class 的初始化责任。

   成本这么大，怎样看待 virtual base class 呢？（v-inherit）

   > 1.非必要不用 virtual base class。能用 non-virtual 就用。
   >
   > 2.必须使用 virtual base class ，那就避免在其中放置数据。

   

3. 多重继承的适用范例

   一个 model 人的 C++ interface class。

   ```cpp
   class IPerson{ 
   public: 
       virtual ~IPerson(); 
       virtual std::string name() const = 0; 
       virtual std::string birthDate() const =0; 
   };//no doult ：这玩意儿需要实体化，也就是下面的 factory function
   
   //factory function,根据一个独一无二的数据库ID创建一个Person对象 
   std::tr1::shared_ptr<IPerson> makePerson(DatabaseID personIdentifier); 
   
   DatabaseID askUserForDatabaseID(); //从使用者手上取得数据库ID
   
   DatabaseID id(askUserForDatabaseID()); 
   std::tr1::shared_ptr<IPerson> pp(makePerson(id));//object
   ```

   Interface class IPerson 的具现化工作在 factory function 里，意味着 f-function 需要定义具现化 derived class 。我们用 CPerson 充当 derived class 。这时恰好有个 PersonInfo 有现成的元素使用）：

   ```cpp
   class PersonInfo{ 
   public: 
       explicit PersonInfo(DatabaseID pid); 
       virtual ~PersonInfo(); 
       virtual const char* theName()const;  //const char*有点老套，不过这不是重点
       virtual const char* theBirthDate() const; 
   
   private: 
       virtual const char* valueDelimOpen() const; //这个东西是标明字符串起始点和结束点的
       { return "[" ;}
       virtual const char* valueDelimClose() const;  //以特殊字符串为界（默认[]，可自定义）
       { return "]" ;}
   };
   const char* PersonInfo::theName() const 
   { 
       //保留缓冲区给返回值使用：static，自动初始化为“全0” 
       static char value[Max_Formatted_Field_Value_Length]; 
       //写入起始符号 
       std::strcpy(value, valueDelimOpen()); 
       //将value内的字符串附到这个对象的name成员变量中 
       //写入结尾符号 
       std::strcat(value, valueDelimClose()); 
       return value; 
   }
   ```

   theName 调用 valueDelimOpen 产生字符串起始符号 → 产生 name 值 → 调用 valueDelimClose。 

   > valueDelimOpen 和 valueDelimClose 都是 virtual funciton ，所以 theName 返回的结果不仅取决于 PersonInfo，也取决于从 PersonInfo 派生下去的 class 。

这里使用基于 private 继承的 is-implemented-in-terms-of 方法。

```cpp
class Cperson: public IPerson, private PersonInfo{  //info 是 in terms 功能要求
public:                                            //iperson 是 具现工作 要求
    explicit Cperson(DatabaseID pid): PersonInfo(pid){} 

    virtual std::string name() const 
    { 
        return PersonInfo::theName(); 
    } 

    virtual std::string birthDate() const 
    { 
        return PersonInfo::theBirthDate(); 
    } 

private: 
    const char* valueDelimOpen() const {return "";} 
    const char* valueDelimClose() const {return "";} 
};
```

> 以 CPerson 为主体，利用 Private PerosnInfo 实现出 IPerson 的接口完成具现化工作。

---
### 7.Templates and Generic Programming

## 41.了解隐式接口和编译期多态

- class 和 template 都支持接口和多态

  > - 对 class 而言 : 接口是显式的，以函数签名为中心；多态是通过 virtual function 发生在**运行期**。
  > - 对 template 而言：接口是隐式的，以有效表达式为中心；多态是通过 template 具现化/函数重载发生在**编译期**。

这一章开始进入模板和泛型编程。这需要我们以另一个思路来考虑。

首先来区分不同接口在函数多态上的不同：

显式接口：

```cpp
class Widget{ 
public: 
    Widget(); 
    virtual ~Widget(); 
    virtual std::size_t size() const; 
    virtual void normalize(); 
    virtual swap(Widget& other); 
}; 
void doProcessing(Widget& w) 
{ 
    if (w.size() > 10 && w != someNastyWidget){ 
        Widget temp(w); 
        temp.normalize(); 
        temp.swap(w); 
    } 
}
```

- 接口定义由函数的签名式构成（函数名称，参数类型，返回类型）。

  > 上例 public 接口中的 constructor ,size,swap...... ，当然包括各种类型、常量性。

- 接口定义在源码中清晰可见

- 基于 virtual 函数的运行期多态

  > 运行期根据 object 的动态类型决定调用哪一个函数。

隐式接口：

```cpp
template<typename T> 
void doProcessing(T& w) 
{ 
    if (w.size() > 10 && w != someNastyWidget){ 
        T temp(w); 
        temp.normalize(); 
        temp.swap(w); 
    } 
}
```

- 接口定义基于需求有些约束。

  > 看看条件语句，可以知道 t 必须提供 size member function，必须支持 operator ！=  等。
  >
  > 以上的“必须”可以通过 operator overloading 有所改观：
  >
  > 以 size 为例，T 类型对象可以单纯从 base class 继承，并且返回类型X不一定非要是 int —— 你只需要让X和10一起支持 operator> 就行（甚至X可以发生隐式转换达到这一目的）。

- 对象(本例为 w )支持的接口，由 template 执行在对象本身时的操作决定。

  > 需要用到什么，compiler 就在编译期具现化什么。

- 对象的函数调用发生在编译期。

  > 编译期会让这种多态以多个重载函数的形式实现。

---

## 42.了解 typename 的双重意义

- 作为 template 参数，class 和 typename 可互换

- typename 能够标识嵌套从属类型名称。

  > 不能再 base class list 或 member initialization list 内作为 base class 修饰符。

关于第二点：

```cpp
template<typename C>
void print2nd(const C& container)    // 打印容器内第二个元素
{                                    // 注意这不是有效C++代码
    if (container.size() >= 2) {
        C::const_iterator iter(container.begin());    // 取得第一元素的迭代器
        ++iter;                                       // 将iter移往第二元素
        int value = *iter;                            // 将该元素复制到某个int
        std::cout << value;                           // 打印那个int
    } 
}
```

> **从属名称**: template 内，包含模板参数的名称。
>
> **嵌套从属名称**：从属名称在 class 内呈嵌套状。(模板类中的模板成员)
>
> **嵌套从属类型名称**：具备以上属性的同时，指涉了某类型。
>
> 示例：C::const_iterator            /          int（非从属名称）

typename的另一重要用法：**证明一个名称是嵌套从属类型名称** 而不是变量 或者其他什么东西。

所以上面的代码需要更正：

```cpp
    if (container.size() >= 2) {
        typename C::const_iterator iter(container.begin());  //精髓
```

typename 例外情况：

```cpp
template<typename T>
class Derived: public Base<T>::Nested {    // 1.base class list中不允许“typename”
public:
    explicit Derived(int x)
    : Base<T>::Nested(x)                   // 2.mem.init.list中不允许“typename”
    {
        typename Base<T>::Nested temp;     // 3.嵌套从属类型名称满足：
        ...                                // 既不在base class list中也不在mem.init.list
    }                                      // 作为一个base class修饰符需加上typename
    ...
};
```

最后：typename 在不同编译器上有不同规则。它在移植性方面有一些无伤大雅的问题。

---

## 43.学习处理模板化基类的名称

- 编译器看不到模板父类的成员，有三种方法处理这种情况。

模板全特化 对 调用继承模板类函数 的冲击：

```cpp
template<typename T1>
class Class_A{
public:
	void send1(T1 var);
};

template <typename T1>
class Class_B:public Class_A< T1 >{
public:
	void send2(T1 var) { send1(var); } //编译错误，调用模板基类内函数失败
};
```

Class_B 继承 Class_A后，调用 A 的函数理所应当，但编译发生错误：编译器看不到模板父类的成员。

> 更准确的说，即使看到了也无济于事：模板中的全特化会让原模板类造成编译器的错觉，所以干脆假装看不见喽。
>
> （CPP compiler 秉持着 "早发现早治疗" 的语法检测原则 : 与其在 template 实参具现化时，不如在 parsing 子类模板定义式 时）

全特化示例：

```cpp
template<>
class Class_A<Type_1>{ //特化模板类，类型为Type_1时的模板类
public:
	void send3(Type_1 var); // 该特化类中压根不存在send1函数
};
```

> 继承在 template CPP 中不像 Object Oriented CPP 那样畅通无阻了。

**解决方法：**

1. base class 函数调用前加上 “this->”

   ```cpp
   void Class_B:: send2(T1 var) { this->send1(var); }
   ```

2. using 声明式

   ```cpp
   template <typename T1>
   class Class_B:public Class_A< T1 >{
   public:
   	using Class_A<T1>::send1;
       void send2(T1 var) { send1(var); }
   };
   ```

3. 指明调用函数的所在区域

   ```cpp
   void Class_B:: send2(T1 var) { Class_A<T1>::send1(var); }
   ```

   > 如果被调用的是 virtual 函数，上述的 explicit qualification 会关闭 “virtual 绑定行为”。



总结：

1. 三个方法本质相同：让编译器知道，base class template 的所有特化版本都能支持我们需要的普通接口。
2. 如果你对编译器保证，却不那么做，编译器也不会让你通过的。



---

## 44.将与参数无关的代码抽离 template

- Template生成多个classes与多个函数，所以任何template代码都不该与某个造成膨胀的template参数产生相依关系。

- 因非类型模板参数(non-type template parameters)而造成的代码膨胀，往往可以消除，做法是以函数参数或者class成员变量替换template参数。

- 因类型参数(type parameters)而造成的代码膨胀，也可以降低，做法是让带有完全相同二进制表述(binary representations)的具现类型(instantiation types)共享实现码。

  

template 使用不当可能会导致 code bloat ，让二进制码呈现一批雷同的代码，浪费空间。

> 这需要你洞悉函数的共同部分，抽离那些个性的部分。
>
> 不过面向对象的设计中，重复部分可以通过肉眼看出来； template 设计中则需要提前考虑可能的重复。

```cpp
template <class T, size_t n>
class SquareMatrix
{
public:
    void Invert();
};

int main()
{
    SquareMatrix<int, 10> a;
    SquareMatrix<int, 5> b;
}
```

这会具现化两份 invert 函数。两个函数除了常量5和10，其他部分完全相同。

为了减少代码冗余，你会想建立一个函数（类实现）。

```cpp
template<typename T>
class SquareMatrixBase
{
public:
    SquareMatrixBase(size_t n,T* pMem) : size(n),pDate(p){}
    void Invert(size_t n){}
    void setDataPtr(T* ptr){pData = ptr;}
private:
    T* pData;
    size_t size;
};
```

两种继承：栈和堆

- 栈：

  ```cpp
  template <class T, size_t n>
  class SquareMatrix: private SquareMatrixBase<T>
  {
  public:
      SquareMatrix() : SquareMatrixBase<T>(n,data){}
      void Invert() { this->Invert(n);  }
  private:
      T data[n * n]; //你也可以用 T*[pData]指向矩阵内容
  };
  ```

- 堆：

  ```cpp
  template <class T, size_t n>
  class SquareMatrix: private SquareMatrixBase<T>
  {
  public:                                                // //将基类的 数据指针 设为NULL
      SquareMatrix() : SquareMatrixBase<T>(n,0),data(new T[n*n] ){
          this->setDatePtr(data.get();) //将指向该内存的指针存起来，将它的副本交给基类
      }
      ...
      void Invert() { this->Invert(n);  }
  private:
      boost::scoped_array<T> data;
  };
  ```

  成员函数可以用 inline 方式调用 base class 版本（如 invert），并且不同对象有着不同类型：<double,5>和<double,10>共同调用 base<double> 成员函数的同时，前者部分不会跑到后者部分中去。

  > 一方面，尺寸专属版函数可能比这种共享版有更好的代码。
  >
  > （尺寸专属版中，尺寸本身是个编译器常量，于是常量的广传达到最优化，把它们折进被生成指令中成为直接操作数等）
  >
  > 另一方面：不同大小矩阵共享单一版本 invert ，可减少执行文件大小，降低程序的 working set 大小，强化指令高速缓存区内的 locality of reference 。
  >
  > 相反的两个作用，要靠亲自上机 test 来确定哪方面占主导地位。

至于对象大小，你可以把 上面最近的版本（与矩阵大小无关的函数版本）扔到 base class 里，这会增加每一个对象的大小 。例如每个 derived class 有一个指针成员指向 base class 内的数据。这会让每个 derived class 增加至少一个指针的大小，但能通过代码重复来获得更有条理的结构

> 当然你可以尝试令一个 base class 贮存一个 protected 指针指向矩阵数据（即使丧失封装性）。这样 delete 一个指针的判断又会让人伤脑筋。不过有一点毋庸置疑 : 越精密的做法意味着越复杂。


**最后**：这个 item 只讨论 non-type template parameters 的膨胀。 type parameters 也会带来膨胀，像一些平台对于同一模板的 int 和 long 不会合并 导致两个版本的成员函数产生。

多数平台上，所有指针类型有着相同的二进制表述，因此template 持有指针类型的名称，应对每一个成员使用同一份底层实现。

---


**最后**：多重继承确乎有其独特的应用场景，并且确实好用。但在大部分情况下，有单一继承可以实现的设计，为了让程序更好理解和简洁，还是使用单一继承更好。



---
## 45.运用成员函数模板接受所有兼容类型

- 使用成员函数模板可以生成接受所有兼容类型的函数。
- 如果你为泛化拷贝构造或泛化赋值声明了成员模板，你依然需要声明常规拷贝构造函数和拷贝赋值运算符。

智能指针是行为像指针而具备一些特性的类指针对象。这种对象在 STL的迭代器中有所应用。

但智能指针的类型转换需要手动设计，相比之下内置指针的好处就是支持各种隐式转换。

```cpp
class Top{};
class Middle: public Top{};
class Bottom: public Middle{};
```

内置指针：

```cpp
Top *p1 = new Bottom;
const Top *p2 = p1;
```

智能指针转换：

```cpp
template<typename T>
class SmartPtr{
    public:
    explicit SmartPtr(T* realPtr);
    ..
};
SmartPtr<Top>pt1 = 
    SmartPtr<Middle>(new Middle);
SmartPtr<Top>pt2 = 
    SmartPtr<Bottom>(new Bottom);
SmartPrt<const Top> pct2 = pct1;
```

> 这是由于，模板继承关系的两个类型，在具现化以后的对象不会有继承关系。 在编译器看来 `SmartPtr` 和 `SmartPtr` 是完全不同的两个类 。

用 Middle 初始化 Top 需要重载 SmartPtr 的构造函数。层级的增加伴随着重载构造函数的增加，理论上是没有穷尽的。这时 需要为 SmartPtr 写一个 template 构造模板。即所谓的 member function template

```cpp
template<typename T>
class SmartPtr{
public:
    template<typename U>
    SmartPtr(const SmartPtr<U>& other);  //为了生成 copy 构造函数
    
} 
```

> 对任何类型，根据 ``SmartPtr<U>`` 可以生成一个 ``SmartPtr<T> `` : 后者的构造函数可以接收前者参数。两者类型是同一个 template 的不同具现体。这可称作 generalized copy construactor function .

>为了支持隐式转换,构造函数未加入 explicit 。

不过这种转换有风险（比如子类转成父类），需要一些约束：

```cpp
template<typename T>
class SmartPtr{
public:
    template<typename U>
    SmartPtr(const SmartPtr<U>& other )  //other 的 heldPtr 初始化 this 的 heldPtr
        :heldPtr(other.get()){...}  //这里能够发生隐式类型转换
    T* get() const { return heldPtr;}
    ...
private:
    T* heldPtr;      //SmartPtr 的原始指针
}
```

> 借助 原始指针 的转型操作 来完成``SmartPtr``的隐式转换。

同时 member function template 的作用不止构造函数，还有对赋值操作的支持。

用 tr1::shared_ptr 作为范例：

```cpp
template<class T> 
class shared_ptr{
public:
    template<class Y>
        explicit shared_ptr(Y *p); //构造，来自兼容内置指针。
    template<class Y>                                  //泛化构造函数
        shared_ptr<shared_ptr<Y> const& r>;            //或shared_ptr
    template<class Y>
        explicit shared_ptr& operator=(shared_ptr<Y> const& r); //或weak_ptr
    template<class Y>
        explicit shared_ptr(auto_ptr<Y>& r);           //或auto_ptr
    template<class Y>
        shared_ptr& operator=(shared_ptr<Y> const& r); //赋值
    tempalte<class Y>                                  //兼容 shared_ptr
        shared_ptr& operator=(auto_ptr<Y>& r);         //和 auto_ptr
};
```

> 泛化构造函数支持隐式转换，其他构造函数则不然（加上explicit）。
>
> 另外，auto_ptr 的特性不支持 const 。

 事实上，成员函数模板不会改变C++的规则。C++规则讲：如果你没有声明拷贝构造函数，那么编译器应该生成一个。 所以`Y = T`时拷贝构造函数不会从成员函数模板实例化，而是会自己生成一个。 

这时你如果想控制 copy 构造函数的一切，那就同时声明 泛化``copy``构造函数和正常``copy``构造函数吧。（这同样适用于赋值)

```cpp
template<class T>
class shared_ptr{
public:
    shared_ptr(shared_ptr const& r);
    template<class Y>
        shared_ptr(shared_ptr<Y> const& r);
 
    shared_ptr& operator=(shared_ptr const& r);
    template<class Y>
        shared_ptr& operator=(shared_ptr<Y> const& r);
};
```

---

## 46.需要类型转换时，可为模板定义非成员函数。

- 阿萨德

**问题：**

item 24 描述过 non-member 函数在所有实参身上的隐式类型转换。在这里，我们将 ``Rational``类型和``operator*``模板化了：

```cpp
template<typename T>
class Rational {
public:
  Rational(const T& numerator = 0, const T& denominator = 1);
  const T numerator() const;           
  const T denominator() const;        
};
 
template<typename T>
const Rational<T> operator*(const Rational<T>& lhs, const Rational<T>& rhs){}
```

模板化以后，理所应当的认为它应该支持混合式算术运算：

```cpp
Rational<int> oneHalf(1, 2);            // OK
Rational<int> result = oneHalf * 2;     // Error!
```

实际上这并不能通过编译 : **template 在实参推导过程中不会进行隐式类型转换。**

> 编译器不能像 item 24 那样知道我们调用哪个函数。template 下它们在寻找具现化出来的在被名为 operator 的函数时，会因为 T 的偏差而受阻。
>
> 上例来说，编译器通过第一参数 oneHalf 发现模板参数T是 Rational 后，应该把 2 隐式转换为 Rational<int> 与 oneHalf 对应。但 template 场景这种情况不能出现。
>
> 模板推导和函数调用是两个过程： 隐式类型转换发生在函数调用时，而在函数调用之前编译器需要实例化一个函数。而在模板实例化的过程中，编译器无从推导T的类型。 

**解决**: 发挥 friend 的另一个功能：template class 内的 friend 声明式可以指涉某个特定函数。

> 类模板不再依赖实参推导，编译器可以在 class Rational<T>具现化时得知 T。

```cpp
template<typename T>
class Rational {
public:
    friend const Rational operator*(const Rational& lhs, const Rational& rhs);
};
template<typename T>
const Rational<T> operator*(const Rational<T>& lhs, const Rational<T>& rhs){}
```

 在 `Rational` 中声明的 friend 没有添加模板参数T，这是一个简便写法，它完全等价于：

```cpp
friend const Rational<T> operator*(const Rational<T>& lhs, const Rational<T>& rhs);
```

> 通过编译后，operator* 通过编译，class Rational<int>被具现化出来。friend 函数 operator* 被自动声明出来。于是进入函数调用过程，支持隐式类型转换。

通过编译后，链接还会出错。 虽然在类中声明了 `friend operator*`，编译器却不会实例化该声明对应的函数定义。 由于函数是我们自己声明的，编译器认为我们有义务自己去定义那个函数。 

```cpp
template<typename T>
class Rational {
public:
    friend const Rational operator*(const Rational& lhs, const Rational& rhs)
    {
        return Rational(lhs.numerator() * rhs.numerator(), lhs.denominator() * 									rhs.denominator());
    }
};
```

总结：

- 为了对所有参数支持隐式类型转换，`operator*` 需要声明为非成员函数；
- 为了让编译器推导出模板参数，`operator*` 需要在类中声明；
- 为了函数自动具现化，需要在类中声明非成员函数并令其为 friend；
- 声明的函数的同时我们有义务给出函数定义，所以在函数定义也应当放在 friend 声明中。

**特殊场景：**

如 item30 所说，定义在类定义中的函数是 inline 函数。 如果 operator* 函数体变得很大，那么 inline 函数就不再合适了，这时我们可以让 operator* 调用外部的一个辅助函数：

```cpp
template<typename T> class Rational;
template<typename T>
const Rational<T> doMultiply(const Rational<T>& lhs, const Rational<T>& rhs);
 
template<typename T>
class Rational{
public:
    friend Rational<T> operator*(const Rational<T>& lhs, const Rational<T>& rhs){
        return doMultiply(lhs, rhs);
    }
};
```

编译器可能有把 template 定义式放到头文件里的要求。你可以在头文件内定义helper template ,这里用 doMultiply 举例：

```cpp
template<typename T>
class Rational<T> doMultiply(const Rational<T>& lhs,
                         const Rational<T>& rhs){
        return Rational<T>(lhs.numerator() * rhs.numerator(),
                         lhs.denominator() * rhs.denominator()) ;
    }
```

> doMultiply 作为模板，像本文最开始的 operator* 一样不支持混合式乘法。但它只负责被 operator* 调用。而 operator* 负责隐式转换，确保两个对象能被相乘，传递给适当的 doMultiply template 具现体，完成乘法操作。

---

## 47.使用traits classes 表现类型信息

- traits classes 使关于类型的信息在编译期间可用。它们使用模板和模板特化实现。
- 结合重载，traits classes 使得执行编译期类型 if...else 检验成为可能。

**迭代器分类：**

- 最简单的迭代器是输入迭代器（input iterator）和输出迭代器（output iterator）， 它们只能向前移动，可以读取/写入它的当前位置，但只能读写一次。比如 ostream_iterator 就是一个输出迭代器。
- 比它们稍强的是前向迭代器（forward iterator），可以多次读写它的当前位置。 单向链表（slist，STL 并未提供）和 TR1 哈希容器的迭代器就属于前向迭代器。
- 双向迭代器（bidirectional iterator）支持前后移动，支持它的容器包括 set, multiset, map, multimap。
- 随机访问迭代器（random access iterator）是最强的一类迭代器，可以支持 +=, -= 等移动操作，支持它的容器包括 vector, deque,string 等。



对于上述五种迭代器，C++ 提供了五种 Tag 来标识迭代器的类型，它们之间是 ”is-a” 的关系：

```cpp
struct input_iterator_tag {};
struct output_iterator_tag {};
struct forward_iterator_tag: public input_iterator_tag {};
struct bidirectional_iterator_tag: public forward_iterator_tag {};
struct random_access_iterator_tag: public bidirectional_iterator_tag {};
```



STL有一工具性 template 名为 advance ，用来将指定迭代器移动给定距离。

```cpp
template<typename IterT, typename DistT>
void advance (IterT& iter, DistT d);
```

> 移动距离通常是不断执行 ++ -- 。只有随机访问迭代器能 += 。

```cpp
template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d){
  if (iter is a random access iterator) {
    iter += d;                                      // use iterator arithmetic
  }                                                  // for random access iters
  else {
    if (d >= 0) { while (d--) ++iter; }              // use iterative calls to
    else { while (d++) --iter; }                     // ++ or -- for other
  }                                                  // iterator categories
}
```

> 这里的关键是取得类型信息，让条件判断语句得以执行。

traits 的意义就是让你在编译期间取得类型信息。

> - Traits 不是关键字或者预定构件 ； 它是一种技术，需要在面对内置类型和用户自定义类型时的表现一样好。
>
> - 对内置类型的要求意味着 traits 不能借助类来实现。

下面通过针对迭代器的 traits 来探索这种技术（这种 template 在标准程序库中并不少见）。

``iterator_traits`` 将会标识 ``IterT`` 的迭代器类别：针对该类型，在 struct iterator_traits<IterT>内一定声明 typedef 作为 iterator_category 来确认分类。

`iterator_traits` 的实现包括两部分：

- 用户定义类型的迭代器

以 deque 寄存器为例：

```cpp
template < ... >                    // template params elided
class deque {
public:
  class iterator {
  public:
    typedef random_access_iterator_tag iterator_category;
  }:
};
```

然后在全局的 iterator_traits 模板中响应 typedef 那个用户类型中的 Tag，以提供全局和统一的类型识别。

```cpp
template<typename IterT>
struct iterator_traits {
  typedef typename IterT::iterator_category iterator_category;
};
```

- 基本数据类型指针

指针嵌套不了 typedef 。iterator_traits 可以针对指针类型提供一个偏特化版本(partial template specialization)。

```cpp
template<typename IterT>               // partial template specialization
struct iterator_traits<IterT*>{
    typedef random_access_iterator_tag iterator_category; //指针类似随机访问迭代器
    ...
};
```

总结一下 trait class ：

- 确认若干可能取得的类型相关信息。

  > 对迭代器而言是取得其 catagory

- 为该信息起一个名称

  > 本例中的 iterator_category

- 提供一个 template 和一组特化版本(iterator_traits) ，包含希望支持的类型相关信息。



我们已经用 iterator_traits 提供了迭代器的类型信息，是时候给出 advance 的实现了。

```cpp
template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d) {
  if (typeid(typename std::iterator_traits<IterT>::iterator_category) ==
    typeid(std::random_access_iterator_tag))   //这里仍然有缺陷
  ...
}
```

 上述实现其实并不完美，if 语句中的条件（IterT类型）在编译时就已经决定，它的判断却推迟到了运行时（显然是低效的）。 

编译器条件判断语句的实现，需要为不同的 iterator 提供不同的方法，然后在 advance 里调用它们。 所谓“提供不同方法”即重载。

> 对于传过来的实参，哪一个重载件最匹配就调用哪一个。这正是一个针对类型的编译器条件语句。

```cpp
template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d) {
  doAdvance(iter, d,typename std::iterator_traits<IterT>::iterator_category());                                             
}                                                       
// 随机访问迭代器
template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d, std::random_access_iterator_tag) {
  iter += d;
}
 
// 双向迭代器
template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d, std::bidirectional_iterator_tag) {
  if (d >= 0) { while (d--) ++iter; }
  else { while (d++) --iter; }
}
 
// 输入迭代器
template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d, std::input_iterator_tag) {
  if (d < 0 ) {
     throw std::out_of_range("Negative distance");    // see below
  }
  while (d--) ++iter;
}
```

>这里利用 doAdvance 的重载版本，让 Advance 调用它们并额外传递一个对象来区分迭代器。

不同迭代间存在 “is-a” 继承关系，能够让适用于父类的迭代器同时使用子类的迭代器。

最后总结如何使用 traits class：

- 建立一组重载函数（身份像劳工）或函数模板（例如doAdvance），彼此间的差异只在于各自的traits参数，令每个函数实现码与其接受之traits信息相应和。
- 建立一个控制函数（身份像工头）或函数模板（例如Advance），它调用上述那些劳工函数并传递 traits class 所提供的信息。

> 啊，除了针对迭代器的 Traits ，标准程序库里还有 char_traits 用来保存字符类型的相关信息，以及numeric_limits 用来保存数值类型相关信息(比如数值类型的最大值和最小值)。
>
> tr1 也导入了很多新的 traits classes 来提供类型信息。

---

## 48.认识元编程

- Template metaprogramming（TMP，模板元编程）可将工作由运行期移往编译期。

  > 这能帮助实现早期错误侦测和更高的执行效率。

- TMP 可被用来生成“based on combinations of policy choices”的客户定制代码，也可用来避免生成对特殊类型并不适合的代码

> 模板元编程（Template Metaprogramming，TMP）就是利用模板来编写template-based C++程序并使其运行于编译期的过程。 模板元程序（Template Metaprogram）是由C++写成的，运行在编译器中的程序。当程序运行结束后，它的输出仍然会正常地编译。

C++并不是为模板元编程设计的，但自90年代以来，模板元编程的用处逐渐地被世人所发现（不是发明）。

**好处：**

- 模板编程提供的很多便利在面向对象编程中很难实现；
- 程序的工作时间从运行期转移到编译期，可以更早发现错误，运行时更加高效。
- 在设计模式上，可以基于不同的策略，自动组合而生成具体的设计模式实现。

将<u>运行期程序转移到编译期</u>的功能非常优秀。以 item 47 中 advance( )为例：

1. 运行期转移到编译期能降低可执行文件大小；

2. 能够避免不必要的静态类型检查错误

> 在 typeid-based 条件判断语句的 advance 函数内，会出现不能编译的情况：设想以下`advance::iterator, int>`中的这条语句：
>
> ```cpp
> iter += d;
> ```
>
> 即使  `list::iterator` 是双向迭代器，不支持`+=`运算，也就是不会运行该死的``+=``算术运算，编译期仍然会提醒类型错误，"iterator 不支持 += 运算符" 。

**元编程的威力**：

TMP 是图灵完全（turing-comlete）的，可以利用它 声明变量、执行循环、编写和调用函数等等。 但它的使用风格和普通 C++ 完全不同。 TMP 主要是个函数式语言。用正常 c++ 的阶乘循环函数举例：

> TMP 没有真正的循环构件，它是靠递归完成类似功能。

```cpp
template<unsigned n>
struct Factorial{
    enum{ value = n * Factorial<n-1>::value };
};
template<>
struct Factorial<0>{
    enum{ value = 1 };
};
 
void test(){
    cout<<Factorial<5>::value;   //5!  =  120
}
```

> TMP以 recursive tepmlate instantiation 取代循环，每个具现体都有自己的一份 value ，每个 value 有自己“  循环“的适当值。

**最后：**

总结一下 TMP 的实际应用场景：

- 确保量纲正确。在科学计算中，量纲的结合要始终保持正确。比如一定要单位为 ”m” 的变量和单位为 ”s” 的变量相除才能得到一个速度变量（其单位为”m/s”）。 使用 TMP 时，编译器可以保证这一点。因为不同的量纲在 TMP 中会被映射为不同的类型。
- 优化矩阵运算。比如矩阵连乘问题，TMP 中有一项表达式模板的技术，可以在编译期去除临时变量和合并循环。 可以做到更好的运行时效率。
- 自定义设计模式的实现。设计模式往往有多种实现方式，而一项叫基于策略设计的 TMP 技术可以帮你创建独立的设计策略，而这些设计策略可以以任意方式组合。生成无数的设计模式实现方式。

往小场景里说：

- 运行期转移到编译器的效率提升
- 完成“不能在运行期实现的功能”

 都令人印象深刻。

---
### 第八章：Customizing new and delete

## 49.了解 new-handler 的行为

- set_new_handler 允许客户指定一个当内存分配请求不能被满足时可以被调用的函数。
- nothrow new 作用有限，因为它仅适用于内存分配，随后的 constructor 调用可能依然会抛出 exceptions。

==new-handler 登场==：

new 申请内存失败时会抛出 `"bad alloc"` 异常，此前会调用一个由 std::set_new_handler() 指定的错误处理函数``new-handler``。 

“new-handler” 函数通过 std::set_new_handler() 来设置，std::set_new_handler() 定义在``中： 

```cpp
namespace std{
    typedef void (*new_handler)();  //定义出一个指针指向函数（new 失败时所调用）
    new_handler set_new_handler(new_handler p) throw();//尾端的 throw 是一份异常明细
}                                                      //表示该函数不抛出任何异常
```

使用示例：

```cpp
void outOfMem(){
    std::cout<<"Unable to alloc memory";
    std::abort();
}
int main(){
    std::set_new_handler(outOfMem);
    int *p = new int[100000000L];
}
```

当 new 申请内存失败时，它会不断调用 ``new-handler`` 函数，直到找到足够内存。

**总结一下 new-handler 的作用：**

- 使更多内存可用；

  > 这是为了让下一次内存分配可能成功。(可能程序开始就分配了大块内存供此使用)

- 安装一个新的 ”new-handler”；

  > 当前new-handler失效后可以替换为另一个更有效的。（可以修改自己的成员来增强下次调用）

- 卸载当前 ”new-handler”；

  > 将 null 传给`` set_new_handler ``。（没有安装 new-handler 会导致内存分配不成功时抛出异常）

- 抛出 bad_alloc（或它的子类）异常；

  > 这个异常不会被 new 捕捉，而是传输到申请内存的地方。

- 不返回，可以 abort 或者 exit 。



==重载 operator new==：

了解完 new-handler ，来考虑下面对不同类型时的处理情况。

 std::set_new_handler 设置的是全局的 bad_alloc 的错误处理函数，C++并未提供类型相关的 bad_alloc 异常处理机制。  但你可以在 class 内重载 new 和 new-handler ，用完在改回去就是了。

以 ``Widget`` 类为例：

```cpp
class Widget{
public:
    static std::new_handler set_new_handler(std::new_handler p) throw();
    static void * operator new(std::size_t size) throw(std::bad_alloc);
private:
    static std::new_handler current;
};
 
// 静态成员需要定义在类的外面，具体看 item 2
std::new_handler Widget::current = 0; //初始化
std::new_handler Widget::set_new_handler(std::new_handler p) throw(){
    std::new_handler old = current;
    current = p;
    return old;//替换使用完毕后 归还原指针
}
```

> 关于 abort, exit, terminate 的区别：abort 会设置程序非正常退出，exit 会设置程序正常退出，当存在未处理异常时C++会调用 terminate， 它会回调由 std::set_terminate 设置的处理函数，默认会调用 abort。 

operator new 的工作分为三个步骤：

- 调用 std::set_new_handler，把 Widget::current 设置为全局的错误处理函数；

- 调用全局的 operator new 来分配真正的内存；

  > 如果分配内存失败，Widget::current 将会抛出异常；

- 不管成功与否，都卸载 Widget::current，并安装调用 Widget::operator new 之前的全局错误处理函数。

  > 成功申请内存后由 widget 析构函数恢复，失败后由 new 函数本身处理。

==RAII类==

```cpp
class NewHandlerHolder{ //资源处理类，帮助其他类实现 new-handler
public:
    explicit NewHandlerHolder(new_handler nh): handler(nh){}  //取得目前的new-hander
    ~NewHandlerHolder(){ set_new_handler(handler); } //释放
private:
    new_handler handler;
    NewHandlerHolder(const HandlerHolder&);     // 禁用拷贝构造函数 见 item 14
    const NewHandlerHolder& operator=(const NewHandlerHolder&); // 禁用赋值运算符
};
//于是 Widget::operator new 的实现其实非常简单：

void * Widget::operator new(size_t size) throw(bad_alloc){
    NewHandlerHolder h(set_new_handler(currentHandler));//返回原new-h来记录，使用现在的nh
    return ::operator new(size);    // 调用全局的new，抛出异常或者成功
}   // 函数调用结束，恢复 global new-handler
```

在客户端使用情况是这样的：

```cpp
    void outofMem();
    Widget::set_new_handler(outOfMem); // 设定outOfMem为Widget的new-handing
    Widget* pw1 = new Widget;          // 如果内存分配失败调用outOfmem
    std::string* ps = new std::string; // 如果内存分配失败调用global new-hanlding函数
    Widget::set_new_handler(0);        // 设定Widget的new-handing为null
    Widget* pw2 = new Widget;          // 如果内存分配失败，立刻抛出异常
```

仔细观察上面的代码，很容易发现自定义 ”new-handler” 的逻辑其实和 class 本身是无关的。我们可以把这些逻辑抽取出来作为一个 template base class (功能类）： 

> 你只需要让template部分确保每一个 class 获得一个不同的 currentHandler 成员变量

```cpp
template<typename T>
class NewHandlerSupport{
public:
    static std::new_handler set_new_handler(std::new_handler p) throw();
    static void * operator new(std::size_t size) throw(std::bad_alloc);
private:
    static std::new_handler current;
};
 
template<typename T>
std::new_handler NewHandlerSupport<T>::set_new_handler(std::new_handler p) throw(){
    std::new_handler old = current;
    current = p;
    return old;
}
 
template<typename T>
void * NewHandlerSupport<T>::operator new(std::size_t size) throw(std::bad_alloc){
    NewHandlerHolder h(std::set_new_handler(current));
    return ::operator new(size);
}
//将每一个 currentHandler 初始化为null
template<typename T>
std::new_handler NewHandlerSupport<T>::current = 0;
 
```

> 实际上该 base class 从未使用参数T。那没什么影响。只是由于成员变量 current 是 static，需要我们为不同 class 依次定制 currentHandler 副本。类型T只是用来区分不同 derived class ，

有了这个模板基类后，给 Widget 添加 ”new-handler” 支持只需要 public 继承即可：

```cpp
class Widget: public NewHandlerSupport<Widget>{ ... };
```

> 子类继承一个模板化的 base class ,同时后者以前者为类型参数。
>
> 这个有点绕弯弯的技术称作：curiously recurring template pattern CRTP 。

==关于nothrow==：

1993 年之前 C++ 的 operator new 在失败时会返回 null 而不是抛出异常。如今的 C++ 仍然支持这种 nothrow 的operator new：



```
Widget *p1 = new Widget;    // 失败时抛出 bad_alloc 异常
assert(p1 != 0);            // 这总是成立的
if(p1 == 0) ...             //这个测试一定失败
Widget *p2 = new (std::nothrow) Widget; 如果分配 widget 失败，返回0
if(p2 == 0) ...             // 可能成功，失败时 p2 == 0
```

“nothrow new” 只适用于内存分配错误。而构造函数也可以抛出的异常，这时它也不能保证是 new 语句是 ”nothrow” 的。



## 50.了解 new 和 delete 的合理替换时机

- 有很多正当的编写 new 和 delete 的自定义版本的理由，包括改进性能，调试 heap（堆）用法错误，以及收集 heap（堆）用法信息。

**列举一下替换默认 new 和 delete 的场景**：

- 用来检测运用上的错误。

  >自行定义的 new 能够超额分配内存，用额外空间放置特定的 byte pattern（即 signature）。delete 能够得以检查上述签名是否原封不动。不动说明分配区某个时间发生了 overrun 或 underrun，delete 此时可以记录这个指针。

- 提高效率

  > 编译器自带的 new 和 delete 要处理长时间执行程序和各种需求，大小块内存和各种分配形态。这意味着 自带 new 和 delete 会有着强大的兼容性，同时不会对某个需求有太多的倾向。所以定制版 new 和 delete 会比自带的更有效率。

- 收集动态分配内存的使用信息。

  > 在继续自定义 new 之前，你可能需要先自定义一个 new 来收集地址分配信息，比如动态内存块大小是怎样分布的？分配和回收是先进先出 FIFO 还是后进先出 LIFO？

- 为了增加分配和归还速度

  > 定制分配器针对特定类型和固定尺寸能够有更好的效果。像Boost::Pool

- 为了降低默认内存管理器的额外开销

- 弥补默认分配器的非最佳齐位(suboptimal alignment)

  > 将不保证对齐的new替换为对齐的版本，可能导致程序效率大幅提升。 

- 将相关对象集中

  > 如果指定某个数据结构往往一起使用，而你有希望处理这些数据时，将“内存页错误”（page fault）的频率降至最低，那么为此数据结构创建另一个heap就有意义，这样它们就可以被成簇集中在尽可能少的内存页（page）上。见条款52。 

- 完成非传统行为

  > 有时希望operator new和delete做编译器提供的缺省版本没做的事情，如将C API封装成C++ API，将归还内存覆盖为0 

**重写示例**：

自定义一个 operator new 很容易的，比如实现一个支持越界检查的 new：

```cpp
static const int signature = 0xDEADBEEF;    // 边界符
typedef unsigned char Byte; 
 
void* operator new(std::size_t size) throw(std::bad_alloc) {
    // 多申请一些内存来存放占位符 
    size_t realSize = size + 2 * sizeof(int); 
 
    // 申请内存
    void *pMem = malloc(realSize);
    if (!pMem) throw bad_alloc(); 
 
    // 写入边界符
    *(reinterpret_cast<int*>(static_cast<Byte*>(pMem)+realSize-sizeof(int))) 
        = *(static_cast<int*>(pMem)) = signature;
 
    // 返回真正的内存区域
    return static_cast<Byte*>(pMem) + sizeof(int);
}
```

>其实上述代码是有一些瑕疵的：
>
>- operator new 应当不断地调用new handler，上述代码中没有遵循这个惯例；
>- 齐位要求（alignment）：
>
>  1. 许多 computer architectures 体系结构下，不同的类型被要求放在对应的内存位置。比如 double 的起始地址应当是 8 的整数倍，int的起始地址应当是 4 的整数倍。上述代码可能会引起运行时硬件错误。
>  2. 起始地址对齐。C++要求动态内存申请的起始地址对所有类型来说都是字节对齐的，new 和 malloc 都遵循这一点，然而我们返回的地址偏移了一个int。这将造成程序崩溃或执行速度变慢。
>     到此为止你已经看到了，实现一个 operator new 很容易，但实现一个好的 operator new 却很难。

齐位和内存管理器有关。你可以浏览一下不同内存管理器来重新连接，到开放源码中去找（如 boost 程序库的 Pool）。

## 51.编写 new 和 delete 时固守常规

- operator new 应该包含一个设法分配内存的无限循环，如果它不能满足一个内存请求，应该调用 new-handler，还应该处理零字节请求。class-specific（类专用）版本应该处理对比预期更大的区块的请求。
- operator delete 如果收到一个空指针应该什么都不做。class-specific（类专用）版本应该处理比预期更大的区块。

如题，下面说明所谓 new 和 delete 的常规标准：

- 返回值必须正确：申请成功返回内存地址，申请失败就要调用 new-handling 函数。

- 具备重复申请内存的能力：在每次失败后调用 new-handling 函数。

  > 只有当指向 new-handling 函数的指针是 null，operator new 才会抛出异常

- 申请大小为零时也应返回合法的指针。 

**示例：一个 non-member operator new 。**

```cpp
void * operator new(std::size_t size) throw(std::bad_alloc){
    if(size == 0) size = 1;
    while(true){
        // 尝试申请
        void *p = malloc(size);
 
        // 申请成功,直接返回
        if(p) return p;
 
        // 申请失败，获得new handler
        new_handler h = set_new_handler(0);
        set_new_handler(h);
 
        if(h) (*h)();
        else throw bad_alloc();
    }
}
```

> 1.申请大小为0时，示例给出的解决方法是把大小改为1。虽然笨但是有效。
>
> 2.重复申请内存的能力即死循环 while(true) 。
>
> 3.调用 new-handling 函数的方法略微有些笨拙：设为null，利用返回值返回旧函数指针，再返回原样。
>
> (因为没有办法直接取得 new-handling 指针，只能通过 set_new_handler 找出它)

**member operator new function 在继承时的问题：**

自定义 new 函数会被 derived class 继承。但这个函数的需求往往只针对特定 class ，而非它的所有 derived class 。为防止不必要的麻烦，可以根据大小不同来判断：

```cpp
class Base {
public:
    static void* operator new(std::size_t size) throw(std::bad_alloc);
    ...
};
class Derived: public Base  // 假设Derived未声明operator new
{ ... };
Derived* p = new Derived;   // 这里调用的是Base::operator new
//为防止不必要的麻烦，可以根据大小不同来判断：
void* Base::operator new(std::size_t size) throw(std::bad_alloc)
{
    if (size != sizeof(Base))             // 如果大小错误，
        return ::operator new(size);      // 令标准的operator new起而处理。
    ...                                   // 否则在这里处理。
}
```

> 不用担心 ``size=0`` 的情况，编译器会依据 C++ “独立对象必须有非零大小”。(插入一个 char 等)

另外对于 array 内存分配的 class new，需要实现 ``operator new[ ]`` 。写它的时候只需要注意一点：分配一块 未加工内存 (raw memory) 。

> 你不能知道 array 里的元素对象有多大，也不能计算出个数。并且动态分配的 arrays 可能需要额外空间来存放元素个数。

**至于 operator delete** 

> 相比于new，实现delete的规则要简单很多。唯一需要注意的是 C++ 保证了delete 一个 NULL总是安全的，你尊重该惯例即可。 

non-member   版本:

```cpp
void operator delete(void *rawMem) throw(){
    if(rawMem == 0) return; //面对空指针，do nothing
    # 不是空指针 ， 归还 rawmemory 所指内存
}
```

member delete 版本：

> 多加一个检查删除数量的动作：这是处理定制 class new 将大小有误的分配行为转交给``::operator new``执行。一旦出现这种情况转而调用对应的 ::operator delete 执行。

```cpp
class Base{
public:
    static void* operator new(std::size_t size) throw(std::bad_alloc);
    static void operator delete(void* rawMemory,size_t size) throw();
    ...
};
void Base::operator delete(void* rawMemory,size_t size) throw()
{
    if(rawMemory == 0) return ;
    if(size != sizeof(Base)) {
        ::operator delete(rawMemory);
        return;
    }
    #到这里，正常归还rawMemory所指内存
    return;
}
```

>1.注意上面的检查的是 rawMem 为空，size 是不会为空的。
>
>2.其实 size 实参的值是通过调用者的类型来推导的（如果没有虚析构函数的话）：
>
>```
>Base *p = new Derived;  // 假设Base::~Base不是虚函数
>delete p;  // 传入`delete(void *rawMem, std::size_t size)`的`size == sizeof(Base)`。
>```
>
>3.如果 Base::~Base() 声明为 virtual，则上述 size 就是正确的 sizeof(Derived)。 这也是为什么Item 7 指出析构函数一定要声明 virtual。



----

## 52.写了 placement new 就要写 placement delete

- 在编写一个 operator new 的 placement 版本时，确保同时编写 operator delete 的相应的 placement 版本。否则，你的程序可能会发生微妙的，断续的 memory leaks（内存泄漏）。
- 当你声明 new 和 delete 的 placement 版本时，确保不会无意中覆盖这些函数的常规版本。

**阐述定义：**

==placement new==： 

- 广义上的 ”placement new” 指的是拥有额外参数的 operator new。 

- 狭义上通常是专指指定了位置的 new(std::size_t size, void *pmemory) throw()，用于 vector 申请 capacity 剩余的可用内存。 

**问题场景：**

```cpp
Widget* pw = new Widget;
```

> 这个语句会调用两个函数： Widget 的 operator new ，Widget 的 默认构造函数

> 当内存申请成功，而接收内存的默认构造函数抛出异常时，需要及时取消分配恢复原样，否则就会内存泄漏。

分配的责任不在用户身上，而在 C++ 运行期系统上：对于一个正常的 operator new，系统会调用默认的 delete function。对于 user-defined operator new , 系统如果找不到对应 delete 就会跳过删除操作从而产生问题。

**示例：**

```cpp
class Widget{
public:
    ...
    static void* operator new(size_t size,ostream& logStream)
        throw(bad_alloc);
    /*static void* operator delete(void* pMemory size_t size)
        throw();*/          //默认delete，反面教材
    void* operator delete(size_t size,ostream& logStream)
        throw();      //placement new
    ...
}
```

> 但客户还可能直接调用 ``delete p``，这时 C++ 运行时不会把一个普通指针解释为 ”placement delete”。 所以在 Widget 中不仅要声明 ”placement delete”，还要声明一个正常的 delete。 

placement delete 只有在“伴随 placement new 调用而触发构造函数”出现异常时才会被寻找调用。

```cpp
Widget{
public:
    static void* operator new(std::size_t size, std::ostream& log) throw(std::bad_alloc);
    static void operator delete(void *mem, std::ostream& log);
    static void operator delete(void *mem) throw();
    Widget(){ throw 1; }
};
```

 这样，无论是构造函数抛出异常，还是用户直接调用 delete p，内存都能正确地回收了。 

**覆盖问题：**

在Item 33中提到，类中的名称会隐藏外部的名称，子类的名称会隐藏父类的名称。 所以当你声明一个 ”placement new  ” 时：

```cpp
class Base{
public:
    static void* operator new(std::size_t size, std::ostream& log) throw(std::bad_alloc);
};
Base *p = new Base;     // Error!
Base *p = new (std::cerr) Base;     // OK
```

> 普通的 new 将会抛出异常，因为 ”placement new” 隐藏了外部的 ”normal new”。 

同样道理，derived class 中的 operator new 会掩盖 global 版本和继承而得的 operator new 版本。

```cpp
class Derived: public Base{
public:
    static void* operator new(std::size_t size) throw(std::bad_alloc);
};
Derived *p = new (std::clog) Derived;       // Error!
Derived *p = new Derived;       // OK
```

这是因为子类中的 ”normal new” 隐藏了父类中的 ”placement new”，虽然它们的**函数签名不同**。 但 Item 33 中提到，按照 C++ 的名称隐藏规则会**隐藏所有同名**（name）的东西，**和签名无关**。 

**覆盖解决：**

解决前观察 global 作用域的 operator new：

```cpp
void* operator new(std::size_t) throw(std::bad_alloc);      // normal new
void* operator new(std::size_t, void*) throw();             // placement new
void* operator new(std::size_t, const std::nothrow_t&) throw();     // 见 Item 49
```

非必要情况，请确保这些函数在你的 user-defined operator new 之外还可以正常使用。如果类内有一般性需求， 在创建自定义的 ”new” 时，也要声明这些签名的 ”new” 并调用全局的版本。

> 为了方便，我们可以为这些全局版本的调用声明一个父类 StandardNewDeleteForms： 
>
> ```cpp
> class StandardNewDeleteForms {
> public:
>   // normal new/delete
>   static void* operator new(std::size_t size) throw(std::bad_alloc) { return ::operator new(size); }
>   static void operator delete(void *pMemory) throw() { ::operator delete(pMemory); }
>  
>   // placement new/delete
>   static void* operator new(std::size_t size, void *ptr) throw() { return ::operator new(size, ptr); }
>   static void operator delete(void *pMemory, void *ptr) throw() { return ::operator delete(pMemory, ptr); }
>  
>   // nothrow new/delete
>   static void* operator new(std::size_t size, const std::nothrow_t& nt) throw() { return ::operator new(size, nt); }
>   static void operator delete(void *pMemory, const std::nothrow_t&) throw() { ::operator delete(pMemory); }
> };
> ```
>
> 然后在用户类型 Widget 中 using StandardNewDeleteForms::new/delete 即可使得这些函数都可见：
>
> ```
> class Widget: public StandardNewDeleteForms {           // inherit std forms
> public:
>    using StandardNewDeleteForms::operator new;         
>    using StandardNewDeleteForms::operator delete;     
>  
>    static void* operator new(std::size_t size, std::ostream& log) throw(std::bad_alloc);   // 自定义 placement new
>    static void operator delete(void *pMemory, std::ostream& logStream) throw();            // 对应的 placement delete
> };
> ```



----

### 第九章：Miscellany
## 53.不要轻易忽视编译器的警告
- **严肃对待编译器发出的警告信息。努力在你的编译器的最高（最严苛）警告级别下争取无任何警告的荣誉。**

- **不要过度倚赖编译器的报警能力，因为不同的编译器对待事情的态度并不相同。一旦移植到另一个编译器上，你原来倚赖的警告信息有可能消失。**

1. 编译器的警告可能给出很重要的信息,例如:

```cpp
class B{
public:
    virtual void f() const;
    ...
};
class D:public B{
public:
    virtual void f();
    ...
};
```



  由于 B 中的 ``f()`` 是 const 成员函数,而D中的``f()`` 是非const,因此``D::f()``是对``B::f()`` 的 override 而非重新声明,编译器通常会给出"``warning: D:f() hides virtual B::f()``"的警告,这个警告其实包含两层意思:


>1. D并没有重新声明virtual void f() const,因而它继承了B的virtual void f() const实现.
>2. 由于D声明了virtual void f(),它是对virtual void f() const的重写,由于名称遮掩,将不能通过D类型对象来调用virtual void f() const.

  因此,会出现以下情况(见注释):

```cpp
B b;
D d;
B* pb=&b;
pb->f();    //调用的是B::f()
pb=&d;
pb->f();   //调用的仍然是B::f() !
```

> 不管怎么说，面对警告信息时，你一定要清楚的了解它的真实含义，然后才可以选择性的处理或者忽略。

2. 不同编译器有不同警告标准,因而不能依赖编译器来指出错误.

   > 警告信息天生和编译器相关，`不同的编译器有不同的警告标准`。所以，草率依赖编译器为你指出错误，并不可取。一些老旧的编译器面对同样的问题可能半句抱怨都没有。 

## 54.让自己熟悉包括TR1在内的标准程序库
详情看书
## 55.让自己熟悉boost
详情看书
