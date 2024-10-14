### 构造、析构、拷贝语意学

- 不要在纯虚基类里声明成员变量！

  > 这是《effective c++》里的一条忠告，现在我们回顾一下。

  下面是个漏洞百出的例子，并且围绕漏洞会有所讨论：

```cpp
class Abstract_base
{
public:
	virtual ~Abstract_base() = 0;
	virtual void interface() const = 0;
	virtual const char*
		mumble() const {return _mumble;}
protected:
	char* _mumble; // damn bug...
};  //这个类需要一个显式的构造函数初始化mumble，否则后续使用很容易出问题
```

可能你想的是后续子类能完成初始化工作。这样想的话，纯虚基类还是要提供一个接口：带有参数的构造函数（最好还是 protected 权限）。

```cpp
Abstract_base::
    Abstract_base(char* mumble_value = 0) //like this
	:_mumble(mumble_value){}
```

真是有够麻烦的......不要再提后面的使用者或修改者忘记赋值之类的问题了。。

最实用的观点还是：**不要在纯虚基类里声明成员变量。**

> 但在某些情况下，把子类共享的数据放到基类里也算是一种自然的想法和设计。

---

1.  **纯虚函数的存在**

虽然和设计初衷相违背，但纯虚函数确实可以在虚基类里定义甚至调用。

```cpp
inline void
Abstract_base:: interface() const
{
	function
	//...
}	 
 
inline void
Concrete_derived::interface() const
{
	//静态调用
	//调用一个pure virtual function
	Abstract_base::interface(); //调用的方式是静态调用而非虚拟机制特有的执行期动态调用。
} 
```

- Pure Virtual destruction

例外是纯虚析构。你必须定义它。

> 因为析构“树”的存在，子类对象的析构连带着其父类的所有析构，到纯虚基类这里不能缺席啊。
>
> 准确来说：静态子类析构会被编译器扩张，静态调用其基类（包括虚基类）的析构函数。析构函数的缺失会导致链接失败。

纯虚函数定义的可能性，使编译器不会在面对纯虚析构函数时停止执行。

另外，编译器不能像默认构造函数那样自动合成纯虚析构函数。

> 因为编译器对可执行文件采取“分离编译模型”。编译器是看不到那些必要的信息的。

这里的结论秉承上文：**不要把虚析构声明成纯虚函数。**

---

2. **虚拟准则（virtual specification）**

所谓“虚构准则”：不要把所有成员函数都一刀切的声明为虚函数，并妄想编译器的优化操作能去除非必要的``virtual-invocation``。

像上例中只是返回成员变量的 ``Abstract_base::mumble``：

- 因为函数定义内容和继承类型无关，完全没必要 virtual 。

- 它的non-virtual函数实例是个inline函数，常常调用会拉低效率。

  > 理论上，编译器如果发现整个继承体系中只有这一个virtual函数，是否能将其调用操作转换为静态调用，并允许其调用操作的 inline expansion 呢？
  >
  > 这么做以后，新的 class 加入又包含了这个单一virtual函数的虚构函数，就会破坏这个优化：函数会被重新编译，产生第二个 virtual 函数实例来配合多态。
  >
  > (实例能够以二进制形式存放在 library 中)

啊，别把这些麻烦事推给编译器了，记住这句话：

**不要为了图省事把所有函数声明为 virtual 。**

---

3. **虚拟准则中 对 const 的态度**

> 注意：语境为虚拟继承体系中

**态度**：==别用const==

决定虚函数是否用const

- 使用const：预期子类对象中的 subclass object 会被使用无数次。
- 不适用const：该函数将不能获得一个 const 引用和指针。

真正的难题：声明为const，但发现子类对象又要修改相关成员变量。

**别用const。**

---

4. **重新考虑 class 的声明**

上例的最终版本如下：

```cpp
class Abstract_base
{
public:
	virtual ~Abstract_base(); //不再是pure
	virtual void interface() const = 0; //不再是const
	 const char*
		mumble() const {return _mumble;} //不再是virtual
protected:
	Abstract_base(char* pc = 0);  //新增一个带唯一参数的constructor
	char* _mumble;
};
```



## 5.1 “无继承”情况下的对象构造

下面展示产生不同对象的方式

```cpp
Point global; //global 内存配置
Point foobar()
{
	Point local;  //local内存配置
	Point *heap = new point; //heap内存配置
	*heap = local; //把一个类对象指定给另一个（拷贝赋值）
    //...stuff...  
	delete heap;  //显式 delete 删除 heap object
	return local;
}
```

**注意:这里出现的 Point 数据类型尚未定义，因为接下来会根据不同情况进行分析。**

# 情况1：质朴的C struct

```cpp
typedef struct
{
	float x, y, z;
}Point;
```

> 普及一个概念：POD —— Plain OI' Data ，可以理解为与C兼容的 c++ 数据类型。
>
> 这里的 Point 便是 POD 。

用C++编译器编译这个POD时：

观念上Point的trival constructor和destructor都会被产生出来并被调用，constructor在程序起始处被调用而destructor在程序的exit()处被调用（exit()是由系统产生的，放在main()结束之前）。**然而，事实上那些trival members要不是没被定义，就是没被调用。**表现和C编译器没什么区别。

1. local变量

   作为POD没有被构造也没有必要析构，但这里没有初始化

2. heap object

   ```cpp
   Point *heap = new Point;
   //会被转换为
   Point *head = __new(sizeof(Point));   //空间而已
   ```

   > 没有默认构造函数调用在 new 出来的 Point 对象上。

3. 拷贝赋值

   local 如果被初始化了就当然没问题，但没初始化的话问题也不大：local 对象是个POD，所以赋值操作只是简单的C风格的二进制码位搬移。

4.  delete 操作

   ```cpp
   delete heap;
   //转换为
   __delete(heap);
   ```

   > 这个操作理应触发编译器产生的 ``trival destructor``，但析构函数要么没被产生要么没被调用。
   >
   > 函数最后通过传值方式把local传回，这也理应触发 ``trival constructor``，但这里 return 仅仅是个位拷贝操作，因为对象是个POD。

注意例外：global 变量 ——

- 在C中被视为临时定义：因为它没有显式初始化操作，所以可在程序中定义多次。

  > 定义多次的实例会被链接器折叠起来，只留下单独一个实例，存储在程序 data segment 中一个空间中。
  >
  > 这个空间“特别保留给未初始化的全局对象使用”，称作BBS（Block Started by Symbol）。

- 在C++中被视为完全定义（它会阻止第二个或更多个定义）。C++根本不支持临时定义，因为class构造行为的应用。

> C和C++的一个差异就在于：C++的所有全局对象都被以“初始化过的数据”来对待。 BBS对c++来说没那么重要。
>
> 即使C++有能力判断这个类是 class object 还是 POD。



# 情况2：抽象数据类型

```cpp
class Point {
pubblic:
	Point(float x = 0.0, float y = 0.0, float z = 0.0)
		: _x(x), _y(y), _z(z) {}
private:
	float _x, _y, _z;
};
```



# 情况3：为继承做准备

```cpp
class Point {
public:
	Point(float x = 0.0, float y = 0.0)
		: _x(x), _y(y){}
	virtual float z();
protected:
	float _x, _y;
};
```



## 5.2 继承体系下的对象构造

当我们定义了一个``object``，如：

```cpp
T object
```

除了会调用其构造函数外，还可能伴随大量的隐藏码：这些隐藏代码由编译器扩充。扩充程度要看 ``class T``的继承体系。

扩充操作如下：

1. ==初始化列表==：在成员初始化列表中的成员变量初始化操作会被放进构造函数本体，并以成员的声明顺序为顺序。

   > 我个人对“以成员的声明顺序为顺序“ 存疑

2. ==成员构造函数==：成员如果没有被初始化，而它本身有个默认构造函数，这个默认构造函数会被强制调用。

3. ==虚表指针（vptr)==:如果类对象里有虚表指针，编译器会为其设定初值来指向适当的虚表。

4. ==基类构造函数==：父辈的基类构造函数会按声明顺序(和成员初始化列表无关)被调用。

   > - 如果基类在成员初始化列表中，就应把需要显式指定的参数都传递过去
   > - 基类没有在成员初始化列表中，但它有默认构造函数(默认 memberwise 拷贝构造也行)，就调用。
   > - 如果基类是多重继承下，第二或后继的基类，this 指针就要有所调整。

5. 调用所有虚基类构造函数（从左到右，从深到浅）：

   > - 如果类在成员初始化列表中，就把需要显式指定的参数都传递过去，没有就调用默认构造函数。
   > - 类中的虚基类子对象的偏移量，要能在执行期存取。
   > - 如果类处于继承体系中最底层，其构造函数会可能被调用，所以其调用机制也要由编译器放进来。

下面结合实际例子，来看看这些扩充机制的必要性。



以Point为例（增加了拷贝函数和虚析构）:

```cpp
class Point {
	Point(float x = 0.0, float y = 0.0);
	Point(const Point&);
	Point& operator=(const Point&);
	virtual ~Point();
	virtual float z() { return 0.0; }
protected:
	float _x, _y;
};
class Line {
	Point _begin, _end; //它由两个点构成~
public:
	Line(float = 0.0, float = 0.0, float = 0.0, float = 0.0);
	Line(const Point&, const Point&);
	void draw();
}；
```

先看看 line class 的扩充结果：

> 每一个显式构造函数都会扩充和调用其两个成员类对象(Point)的构造函数。

```cpp
Line::Line(const Point &begin, const Point &end)
	:_end(end), _begin(begin)
{}
```

会被编译器扩充为

```cpp
Line* Line::Line(Line *this, const Point &bebgin, const Point &end)
{
	this->_begin.Point::Point(begin);
	this->_end.Point::Point(end);
	return this;
}
```

> 于Point声明了一个copy constructor、一个copy operator，以及一个destructor，所以Line class的implicit copy constructor、copy operator和destructor都将是有具体效用的，即non-trivial。 (trivial是没意义的杂项)

---

再看一个例子：

```cpp
Line a;
```

implicit Line copy destructor会被合成出来，同时会调用其成员类对象的析构函数(以构造的相反顺序)。

```cpp
// C++伪代码:合成出来的Line destructor
inline void Line::~Line(Line *this) {
	this->_end.Point::~Point(); //虽然Point::~Point()是virtual
	this->_begin.Point::~Point();//其调用操作仍然会静态决议，在containing class destructor中
}
//如果Point的析构是inline函数，则会在调用地点扩展。 
```

同理

```cpp
line b = a;
//mplicit line copy constructor会被合成出来，成为一个inline public member。
a = b;
//mplicit copy assignment operator会被合成出来，成为一个inline public member。
```



---

最后，多数编译器会缺少对自我指派情况的处理。

```cpp
if(this == &rhs) return *this; //like this
```

然而很多时候都要考虑这种情况，像拷贝操作时忘记：

```cpp
// 使用者供应的copy assignment operator
// 忘记提供一个自我拷贝时的筛选
String &String::operator=(const String &rhs) {
	// 这里需要筛选(在释放资源之前) 
    //if(this == &rhs) return *this;
	delete []str;
	str = new char[strlen(rhs.str) + 1];
}
```



# 虚拟继承

虚拟继承，啊，还是继承我们的``Point class ``吧。

```cpp
class Point3d : public virtual Point {
public:
	Point3d(float x = 0.0, float y = 0.0, float z = 0.0) : Point(x, y), _z(z)
	{}
	Point3d(const Point3d &rhs) : Point(rhs), _z(rhs._z)
	{}
	~Point3d();
	Point3d &operator=(const Point3d &);
	virtual float z() { return _z; }
protected:
	float _z;
};
```

 传统的"constructor扩充现象"并没有用,这是因为 virtual base class 的"共享性"的缘故: 

```cpp
// C++伪代码:不合法的constructor扩充内容
Point3d *Point3d::Point3d(Point3d *this, float x, float y, float z) {
	this->Point::Point(x, y);
	this->__vptr_Point3d = __vtbl_Point3d;
	this->__vptr_Point3d__Point = __vtbl_Point3d_Point;
	this->_z = rhs._z;
	return this;
}
```

> Point3d constructor 扩充内容有错误，这里卖个关子。

- 现在对不同继承层次对象的初始化策略

![1665828502658](C:\Users\fancyzzz\AppData\Roaming\Typora\typora-user-images\1665828502658.png)

```cpp
//看看这三种派生情况

class Vertex : virtual public Point { ... };
//Vertex的constructor必须也调用Point的constructor。
class Vertex3d : public Point3d, public Vertex { ... }; //holly shit double inherit
//当Point3d和Vertex同为Vertex3d的subobjects时,它们对Point constructor的调用操作一定不可以发生,取而代之的是,作为一个最底层的class,Vertex3d有责任将Point初始化。
class PVertex : public Vertex3d { ... };
//更往下的继承,则由PVertex(不再是Vertex3d)来负责完成"被共享的Point subobject"的构造。
```

- 传统的初始化策略

>  传统的初始化策略如果要支持初始化虚基类,会导致constructor中有更多的扩充内容,用以指示 virtual base class constructors应不应该被调用。
>
> constructor的函数本身因而必须尝试测试传进来的参数,然后决定调用或不调用相关的 virtual base class constructors。

下面是Point3d的扩充内容（伪码）

```cpp
// C++伪代码:在virtual base class情况下的constructor扩充内容
Point3d *Point3d::Point3d(Point3d *this, bool __most_derived, float x, float y, float z) {
	if (__most_derived != false) 
		this->Point::Point(x, y);
	this->__vptr_Point3d = __vtbl_Point3d;
	this->__vptr_Point3d_Point = __vtbl_Point3d__Point;
	this->_z = rhs._z;
	return this;
}
```

>  在更深层的继承情况下,例如Vertex3d, 当调用Point3d和Vertex的constructor时,总是会把__most_derived参数设为 false,于是就压制了两个constructors中对Point constructor的调用操作。
>
> ```cpp
> // C++伪代码:在virtual base class情况下的constructor扩充内容
> Vertex3d *Vertex3d::Vertex3d(Vertex3d *this, bool __most_derived, float x, float y, float z) {
> 	if (__most_derived != false)
> 		this->Point::Point(x, y);
> 	// 调用上一层base classes
> 	// 设定__most_derived为false
> 	this->Point3d:::Point3d(false, x, y, z);
> 	this->Vertex::Vertex(false, x, y);
> 	// 设定vptrs
> 	// 插入user mode
> 	return this;
> }
> ```
>
>  这样的策略得以保持语意的正确无误.例如,
>
> - 当定义:  ``Point3d origin `` 时， Point3d constructor可以正确地调用其Point virtual base class subobject 
> - 当定义：``Vertex3d cv``时， Vertex3d constructor正确地调用Point constructor.Point3d和Vertex的constructor会做每一件该做的事情——对Point的调用操作除外。

==结论==：**只有当一个完整的类对象被定义出来（origin)，虚基类构造函数才会被调用。**

如果object 只是个子对象，就不会调用。

> 某些新的编译器，为了产生更有效率的构造函数，将每个构造函数一分为二：
>
> - **一个针对完整的object**：“完整object”版无条件地调用virtual base constructor，设定所有的vptrs等。
>
> - **一个针对 subobject **：“subobject”版则不调用virtual base constructors，也可能不设定vptrs等。 



# vptr初始化语意学

- 继承体系下，构造函数的调用顺序

  以 PVertex 对象为例，它的构造函数调用顺序：

  ```cpp
  Point();
  Point3d();
  Vertex();
  Vertex3d();
  PVertex();
  ```

    假设每个class都定义了一个`virtual function size();`返回该class的大小。 

   我们来看看定义的PVertex constructor： 

  ```cpp
  PVertex::Pvertex(float x, float y, float z)
      : _next(0), Vertex3d(x, y, z), Point(x, y) {
      if(spyOn)//每个构造函数内含一个调用操作
          cerr << "Within Pvertex::PVertex()"
               << "size: " << size() << endl;
  }
  ```
  在一个类的构造函数或析构函数中，通过构造对象来调用一个虚函数，其函数实例应该是在此类中真正有作用的那个。（本例中“该类”为Point3d）
  > 在Point3d 构造函数中调用的 size() 函数，必须被决议为 Point3d::size() 而非PVertex::size()。
  >
  > 
  >
  > **基类构造函数执行时，子类还没有构造起来：**
  >
  > Pvertex构造函数没完成前，Pvertex还不是完整对象；
  >
  > Point3d构造函数执行完毕后，紧紧意味着Point3d的子对象构造完毕了

  构造函数顺序：由父到子，由内而外。

- 如何保证适当的重名函数被调用

  上面的实现很妥帖，因为每个Pvertex 基类构造函数被调用时，编译器保证了适当的size函数被调用。

  如何实现呢？

  1. 静态决议每个调用操作

     既然静态决议了，就不要用虚拟机制。

     在Point3d 的构造函数里，就调用Point3d 的size()。

     > 如果size()里又调用虚函数，这个调用必须决议为Point3d的函数实例。

     > 其他情况下，这个调用会视作virtual，要通过正常虚拟机制决定执行。也就是说虚拟机制本身要知道这个调用源来不来自一个构造函数中。

  2. 在构造函数/析构函数中设立一个标志

     标志的作用是判断是否要以静态方式决议。

     但更好的设计是执行构造函数后，让可能要调用的虚函数数量少些。

     > 决定虚函数数量的关键是虚表。决定虚表如何处理的关键是vptr。

- vptr ：决定虚函数调用的关键

  > vptr的决定效果来自初始化和设定操作。这些操作是编译器的责任，程序员不用瞎操心。

  但还是要看看编译器怎么做到的:

  vptr 什么时候初始化？无非三个情况：1.在其他任何操作前 2.在基类构造函数调用后，但还没进行成员初始化。3.所有操作后

  > 情况2 更好。

   令每一个base class constructor设定其对象的vptr，使它指向相关的virtual table之后，构造中的对象就可以严格而正确地变成“构造过程所幻化出来的每一个class”的对象。

  >  一个PVertex对象会先形成一个Point对象、一个Point3d对象、一个Vertex对象、一个Vertex3d对象，然后才成为一个PVeretex对象。 

   在每一个base class constructors中，对象可以与constructors’s class 的完整对象作比较。对于对象而言，“个体发生学”概况了“系统发生学”。 

  > 我的理解是“一个接一个”进化到了“整体”。
  >
  >  constructor的执行步骤：
  >
  > 1.  在derived class constructor中，“所有virtual base classes”及“上一层base class”的constructors会被调用
  > 2. 上述完成之后，对象的vptrs被初始化，指向相关的virtual tables
  > 3. 如果有member initialization list的话，将在constructor体内扩展开来。这必须在vptr被设定之后才做，以免有一个virtual member function被调用。
  > 4. 最后，执行程序员所提供的代码 
  >
  > ```cpp
  > PVertex::PVertex(float x, float y, float z)
  > _next(0), Vertex3d(x, y, z), Point(x, y)
  > {
  > if(spyOn){
  > cerr << “Within PVertex::PVertex()”
  > << "size: " << size() << endl;
  > }
  > }
  > //它可能被扩展为：
  > 
  > PVertex* PVertex::PVertex(PVertex *this, bool _most_derived,
  > float x, float y, float z){
  > //条件式调用virtual base constructor
  > if(_most_derived != false)
  > this->Point::Point(x, y);
  > //无条件地调用上一层base
  > this->Vertex3d::Vertex3d(x, y, z);
  > 
  > //将相关的vptr初始化
  > this->_vptr_PVertex = _vtbl_PVertex;
  > this->_vptr_Point_PVertex = _vtbl_Point_PVertex;
  > 
  > //程序员缩写代码
  > if(spyOn){
  >     cerr << "Within PVertex::PVertex()"
  >             Point3d::Point3d(),
  >          << "size: " 
  >          << (*this->_vptr_PVertex[3].faddr)(this) 
  >          << endl;
  > }
  > 
  > //传回被构造的对象
  > return this;
  > 
  > ```

- vptr 的初始化

下面是vptr必须被设定的两种情况：

1. 当一个完整的对象被构造起来时，如果我们声明一个Point对象，Point constructor必须设定其vptr。
2. 当一个subobject constructor调用了一个virtual function(不管是直接调用还是间接调用时)。

> 如果我们声明一个PVertex对象，然后由于我们对其base class constructors的最新定义，其vptr将不再需要在每一个base class constructors中被设定。

解决之道是把constructor分裂为一个完整的object实体和一个subobject实体。在subobject实体中，vptr的设定可以省略(如果可以的话)。 

> 这样能回答两个问题：
>
> 1.类的构造函数的成员初始化列表调用类的虚拟函数，安全吗？
>
> - 编译器使 vptr 能保证在成员列表初始化前设定好，挺安全；
> - 函数本身可能会依赖没有设定初值的成员，语意上不太安全。
>
> 2.什么时候给基类构造函数一个参数？这种情况，问题1情况还安全吗？
>
> 不安全。vptr 还没设定好或者指向错误的类。该函数存取的任何类成员数据一定还未初始化。

## 5.3对象复制

> 对象复制，即研究 copy assignment operator 的语意，看看它们怎么被塑造出来。

- bitwise copy 

  > 所谓bitwise copy 和 memberwise copy 即深拷贝和浅拷贝。
  >
  >  深拷贝（memberwise copy）和浅拷贝（bitwise copy）的区别在于：
  >
  > - 深拷贝(对象拷贝)是指源对象与拷贝对象互相独立，其中任何一个对象的改动都不会对另外一个对象造成影响。
  > - 浅拷贝（按位拷贝）在拷贝指针、引用时，按位拷贝会导致拷贝的指针和原指针指向了同一地址。

利用Point class 来讨论。

```cpp
class Point {
public:
    Point(float x= 0.0, float y = 0.0);
    // ... 没有virtual function
protected:
    float _x, _y;
};
```

关于拷贝赋值操作，先看看默认生成的拷贝行为是否够用。

> 如果够用，那么默认拷贝操作将更有效率，不需要再画蛇添足，重写为新的拷贝操作

默认行为不够用，甚至可能导致一些不安全、不正确的操作，需要自己设计一个 copy assginment operator 。拿上面的``Point class``来说，默认的 ``memberwise copy``，编译器不会产生示例（类似拷贝构造的情况），因为**该类已经有了 bitwise copy 语意(这个 class 人畜无害，没有指针也没有多态)，所以隐式拷贝赋值操作没什么意义。**

> 一个 class 对于默认的copy assignment operator,在下面情况不会表现出bitwise copy语意:
>
>1. 当 class 内带一个**member object,而其 class 有一个copy assignment operator时.**
>
>2. 当一个 class 的**base class 有一个copy assignment operator时**.
>
>3. 当一个 class **声明了任何 virtual functions**
>
>   (**一定不可以拷贝右端 class object的vptr地址,由于它可能是一个derived class object**).
>
>4. 当 class **继承自一个 virtual base class**(不论此base class 有没有copy operator)时。

C++ Standard上说copy assignment operators并不表示 bitwise copy semantics 是 nontrivial 。

实际上,只有存在nontrivial instances时才会被合成出来。

```cpp
Point a, b;
a = b;
//其间并没有copy assignment operator被调用.
```

进行按位拷贝,把Point b拷贝给Point a。

> 注意:
>
> 1. 我们还是可能提供一个copy constructor,来配合 name return value (NRV) 的优化。
>
> 2. copy constructor的出现不应该暗示出也一定要提供一个copy assignment operator 。



- 继承下的拷贝构造行为

现在导入一个拷贝赋值操作，来说明该操作在继承下的行为

```cpp
inline Point &Point::operator=(const Point &p) {
    _x = p._x;
    _y = p._y;
    return *this
}
//虚继承
class Point3d : virtual public Point {
public:
    Point3d(float x = 0.0, float y = 0.0, float z = 0.0);
protected:
    float _z;
};
```

如果没有声明拷贝赋值函数，编译器就会合成类似下面的代码：

```cpp
// C++伪代码:被合成的copy assignment operator

inline Point3d &Point3d::operator=(Point3d *const this, const Point3d &p) {
    //调用base class的函数实体
    this->Point::operator=(p);
    // memberwise copy the derived class members
    _z = p._z;
    return *this;
}
```

这时，拷贝赋值操作有一个不太理想的情况：**缺乏成员初始化列表**。

这导致下面的情况将不存在：

```cpp
// C++伪代码,下面性质并不支持
inline Point3d &Point3d::operator=(const Point3d &p3d) : Point(p3d), z(p3d._z)
{}
//取而代之的是
Point::operator=(p3d);
//或
(*(Point *)this) = p3d;
```

- 缺乏初始化列表，在继承体系中该如何阻止基类的拷贝操作

  为什么要阻止呢？

  > a 

  看下面的例子：

  ```cpp
  // class Vertex : virtual public Point
  inline Vertex &Vertex::operator=(const Vertex &v) {
      this->Point::operator(v);
      _next = v._next;
      return *this;
  }
  inline Vertex3d &Vertex3d::operator=(const Vertex3d &v) {
      this->Point::operator=(v);
      this->Point3d::operator(v);
      this->Vertex::operator=(v);
  }
  ```

  1. 传统的 constructor 解决方案：附加额外参数

     附加额外参数没用，因为：取拷贝赋值函数地址是合法的，下面的使用将推翻拷贝赋值函数的设计。

     ```cpp
     typedef Point3d &(Point3d::*pmfPoint3d) (const Point3d &);
     pmfPoint3d pmf = &Point3d::operator=;
     (x.*pmf)(x);
     ```

     > 仍然需要根据其独特的继承体系,插入任何可能数目的参数给copy assignment operator

  2. 为copy assignment operator 产生分化函数（split  function)

     > 产生后，希望函数能支持这个类成为中间基类或最底层子类。

     最好让编译器借助分化函数产生拷贝赋值操作。class-defined user 亲自操刀，可能面临某些函数很难分化的困境：

     ```cpp
     inline Vertex3d &Vertex3d::operator=(const Vertex3d &v) {
         init_bases(v);//甚至让它成为虚函数
     }
     ```

     > copy assignment operator在虚拟继承情况下很复杂,需要特别小心地设计和说明.

     如果使用一个以语言为基础的解决方法,那么应该为copy assignment operator提供一个附加的**"member copy list"**。

     > 简单地说,任何解决方案如果是以程序操作为基础,将导致较高的复杂度和较大的错误倾向. 一般公认,这是语言的一个弱点,也是应该小心检验程序代码的地方(当使用 virtual base classes时).

  3. 语言为基础的方法：在子类拷贝函数示例最后调用那个 operator

     > **在derived class 的copy assignment operator函数实体的最后,明确地调用那个operator** 

     ```cpp
     inline Vertex3d &Vertex3d::operator=(const Vertex3d &v) {
         this->Point3d::operator=(v);
         this->Vertex:;operator=(v);
         // must place this last if your compiler dose not suppress intermediate class invocations
         this->Point::operator=(v);
     }
     ```

     > 这并不能省略subobjects的多重拷贝,但却可以保证语意正确.另一个解决方案要求把 virtual subobjects拷贝到一个分离的函数中,并根据call path条件化调用它。

     最好的办法是 尽可能不要允许一个 virtual base class 的拷贝操作。

     甚至有一个奇怪的方法是: 不要在任何 virtual base class 中声明数据

## 5.4对象的效能（略）

略

## 5.5析构语意学

# 析构函数不是所有情况都是必要的

 如果class没有定义destructor，那么只有在class内含的member object(或者是class自己的base class)拥有destructor的情况下，编译器才会合成出一个来。

否则，destructor被视为不需要，也就不需要被合成（当然更不需要被调用）。

下面举出一个没有合成析构函数的 class （它甚至还有个虚函数）

 ```cpp
 class Point {
public:
	Point(float x = 0.0, float y = 0.0);
	Point(const Point&);
	virtual float z();
private:
	float _x, _y;
};
 ```

 类似的道理，如果把两个Point对象组合成一个Line class: 

```cpp
class Line {
public:Line(const Point&, const Point&);
	   virtual draw();
protected:
	Point _begin, _end;
};
```

Line也不会拥有一个被合成出来的destructor，因为Point并没有destructor。

> 当我们从Point派生出Point3d(即使是一种虚拟派生关系)时，如果我们没有声明一个destructor，编译器也就没必要合成一个destructor。

 你应该拒绝某种强迫症：你已经定义了一个constructor，所以你觉得提供一个destructor是天经地义的事情。事实上，程序员应该根据需要而非感觉来选择是否提供destructor。

# 怎么判断 class 是否需要一个程序层面的析构函数/构造函数

**考虑标准：**

1. 保证对象完整性
2. 类对象生命周期的起点和终点

**析构函数顺序：**

一个程序员定义的析构函数的扩展方式与构造函数方式相同，但顺序相反:

>1. destructor的函数本体现在被执行，也就是说vptr会在程序员的代码执行前被重设（reset）。
>
>2. 如果class拥有member class objects，而后者拥有destructors，那么它们会以声明顺序的相反顺序被调用。
>
>3. 如果object内含一个vptr，那么首先重设（reset）vptr来指向相关的virtual table。
>
>4. 如果有任何直接的（上一层）nonvirtual base classes拥有destructor，它们会以其声明顺序的相反顺序被调用。
>5. 如果有任何virtual base classes拥有destructor，到目前讨论的这个class的最尾端（most-derived）的class，那么它们会以其原来的构造顺序的相反顺序被调用。

析构函数的最佳实现策略和5.2章节最后，构造函数采取的“一分为二”法一样：

>就像constructor一样，目前对于destructor的一种最佳实现策略就是维护两份destructor实例：
>1、一个complete object实例，总是设定好vptr(s)，并调用virtual base class destructors。
>2、一个base class subobject实例；除非在destructor函数中调用一个virtual function，否则它绝不会调用virtual base class destructors并设定vptr。

- 一个object的生命结束于其destructor开始执行之时。由于每一个base class destructor都轮番被调用，所以derived object实际上变成了一个完整的object。

  > 例如一个PVertex对象归还其内存空间之前，会依次变成一个Vertex3d对象、一个Vertex对象，一个Point3d对象，最后成为一个Point对象。

- 当我们在destructor中调用member functions时，对象的蜕变会因为vptr的重新设定（在每一个destructor中，在程序员所供应的代码执行之前）而受到影响。
  

**举个例子**：

```cpp
{
	Point pt;
	Point *p = new Point3d;
	foo(&pt, p);
    ...
	delete p;
}
```

> pt 、 p 作为参数之前，要初始化为坐标值，通过构造函数或显式提供坐标值。
>
> 类使用者没法检验local或heap变量是否需要初始化。
>
> 所以构造函数很有必要。

   那么显式delete掉 p 是否需要提前处理呢？

    ```cpp
  p->x(0);p->y(0); //like this
    ```

> 没必要。没有任何理由说在delete一个对象之前先得将其内容清除干净。

如果确保在结束pt和p的生命之前，没有任何和该 class 有关的程序操作是必要的，往往不一定会需要一个destructor。 

---

**例外情况** ：delete 存在“结束前和该 class 有关的程序操作”



考虑Vertex class，它维护了一个由紧邻的顶点所形成的链表，并且当一个顶点的生命结束时，在链表上来回移动以完成删除操作。如果这正是程序员所需要的，那么这就是Vertex destructor的工作。 

>这个delete 存在“结束前和该 class 有关的程序操作”

当我们从Point3d和Vertex派生出Vertex3d时，如果我们不供应一个explicit Vertex3d destructor，那么我们还是希望Vertex destructor被调用，以结束一个Vertex3d object。

因此编译器必须合成一个Vertex3d destructor，其唯一任务就是调用Vertex destructor。

> 如果我们提供一个Vertex3d destructor，编译器会扩展它，使它调用Vertex destructor(在我们所供应的程序代码之后)。
> 