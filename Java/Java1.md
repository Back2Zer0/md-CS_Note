

# Java 的运行过程

![Snipaste_2023-08-01_17-08-23](C:\Users\fancyzzz\Desktop\Snipaste_2023-08-01_17-08-23.png)



# Main函数：程序的入口

 ![img](https://img-blog.csdnimg.cn/20210308154044110.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80OTc3MDQ0Mw==,size_16,color_FFFFFF,t_70) 



- public：表示的这个程序的访问权限，表示的是任何的场合可以被引用，这样 Java 虚拟机就可以找到 main() 方法,从而来运行 javac 程序。
- static： 表明方法是静态的，不依赖类的对象的，是属于类的，在类加载的时候 main() 方法也随着加载到内存中去。
- void: 方法是不需要返回值的。
- main：主程序，约定俗成，规定的。
- String[ ] args：从控制台接收参数。

> 参数String[] args是一个字符串数组，接收来自程序执行时传进来的参数。如果是在控制台，可以通过编译执行将参数传进来。

**总结：**

1. main方法必须声明为public、static、void，否则JVM没法运行程序

2. 如果JVM找不到main方法就抛出NoSuchMethodError:main异常，例如：如果你运行命令：`java HelloWrold`，JVM就会在HelloWorld.class文件中搜索public static void main (String[] args) 放法

3. main方式是程序的入口，程序执行的开始处。

4. main方法被一个特定的线程”main”运行，程序会一直运行直到main线程结束或者non-daemon线程终止。

5. 当你看到“Exception in Thread main”如：**Excpetion in Thread main:Java.lang.NullPointedException** ,意味着异常来自于main线程

6. 你可以声明main方法使用java1.5的可变参数的方式如：

   

7. 除了static、void、和public，你可以使用final，synchronized、和strictfp修饰符在main方法的签名中，如：

   

8. main方法在Java可以像其他方法一样被重载，但是JVM只会调用上面这种签名规范的main方法。

9. 你可以使用throws子句在方法签名中，可以抛出任何checked和unchecked异常

10. 静态初始化块在JVM调用main方法前被执行，它们在类被JVM加载到内存的时候就被执行了。

> **为什么main方法是静态的（static）**
>
> 1. 正因为main方法是静态的，JVM调用这个方法就**不需要创建任何包含这个main方法的实例**。
> 2. 因为C和C++同样有类似的main方法作为程序执行的入口。
> 3. **如果main方法不声明为静态的，JVM就必须创建main类的实例，因为构造器可以被重载，JVM就没法确定调用哪个main方法。**
> 4. **静态方法和静态数据加载到内存就可以直接调用**而不需要像实例方法一样创建实例后才能调用，如果main方法是静态的，那么它就会被加载到JVM上下文中成为可执行的方法。
>
> **为什么main方法是公有的（public）**
>
> Java指定了一些可访问的修饰符如：private、protected、public，任何方法或变量都可以声明为public，Java可以从该类之外的地方访问。因为main方法是公共的，JVM就可以轻松的访问执行它。
>
> **为什么main方法没有返回值（Void）**
>
> 因为main返回任何值对程序都没任何意义，所以设计成void，意味着main不会有任何值返回



# 基本数据类型

**整数类型：**

byte short int long ，分别对应 1 2 4 8 字节。

**浮点类型：**

| 类型   | 储存   | 范围                       |
| :----- | :----- | :------------------------- |
| float  | 4 字节 | ± 3.40282347E+38F          |
| double | 8 字节 | ± 1.79769313486231570E+308 |

浮点数字面量默认是`double`类型，使用`float`类型需要加`f`/`F`后缀。（`double`类型也有后缀`d`/`D`） 

```java
float f = 1.0F
double d = 1.0
```

 Java 也允许使用科学计数法，例如`103E5`表示`10300000`。还有十六进制的科学计数法`0x103p5`。 



在 Java 中，浮点计算溢出（1d/0）或出错（0d/0）时的三个特殊浮点值：

`Double.POSITIVE_INFINITY`（正无穷，定义为 1d/0）

`Double.NEGATIVE_INFINITY`（负无穷，定义为 -1d/0）

`Double.NaN`（not a number，定义为 0d/0）

若想要验证某个值是否为 infinite 的，或是否不是一个数字，应使用 Double 包装类中的方法：

```java
boolean isNaN(double v);
boolean isFinite(double d);
boolean isInfinite(double v);
```

关于`isNaN`这个方法：

```java
public static boolean isNaN(double v) {
    return (v != v);
    // Java 认为任意两个 not a number 都是不相等的
}
```



**字符类型：**

| 类型 | 储存   |
| :--- | :----- |
| char | 2 字节 |

 Java 中的 char 类型通过 UTF-16 的约定来描述一个代码单元，即 Java 的 char 使用 UTF-16 的编码方式编码 Unicode 字符。 

**布尔类型**：

| 类型    | 储存   | 范围          |
| :------ | :----- | :------------ |
| boolean | 1 字节 | true or false |

 与 C/C++ 不同，在 Java 中，不存在任何从非布尔类型转换为布尔类型的隐式转换。 

# 引用数据类型

数组

**声明一个数组**：`int[] arr;`（也可以使用这种风格`int arr[];`）。

```java
int[] arr;
int arr[];
```

**初始化数组：**

布尔数组将会初始化为`false`，数值数组初始化为`0`，对象数组则为`null`。

```java
new int[10];
new int[]{1, 2, 3, 4};
```

申请一段内存空间，将`arr`指向该内存：

```java
int[] arr = new int[10];
```

声明数组及初始化的简写形式：

```java
int[] arr = {1, 2, 3, 4};
```

**数组拷贝**

Arrays 中的静态方法：

```java
static int[] copyOf(int[] original, int newLength);
```

例如：

```java
int[] arr2 = Arrays.copyOf(arr1, arr1.length);
```

# 变量、常量、声明

**变量**

变量名

Java 允许使用很多 Unicode 字符作为变量名，若想要知道哪些字符可用作命名，可以通过 `Character` 类的 `isJavaldentifierStart`和 `isJavaldentifierPart`方法来检测。

！注意，`$`作为变量名合法，但不要这样做，因为 Java 生成内部类字节码时会将`$`作为文件名的一部分，可能会导致一些很麻烦的问题。

**声明**

在 C/C++ 中，`extern int a;`被称为**声明**，`int a = 10;`被称为**定义**。

而在 Java 中，不区别声明和定义，**未初始化的变量不允许使用**，是编译期错误。

**常量**

通过`final`指示常量，例如`final int a = 10;`。常量只能被赋值一次，此后便无法更改。

除此之外，`final`还可以修饰方法和类，表示"最终的"，不可重写或继承。

```java
final class className {
    // ...
    final public void method() {
        // ...
    }
}
```

!!!注意，`final`与 C++ 中的`const`不同，它关注的是变量直接保存的数据。对于引用类型来说（他就是一个指针），`final`禁止地址的变化，但不禁止更改该地址处的数据。

---



 **1.修饰类的区别**

Java中的final可以用来修饰类，代表该类不能被继承，其内部成员函数也就不能被重构；但C++中的const不能够用来修饰类。

 **2.修饰函数的区别**

final修饰函数，代表该函数不能够被重构；const在函数中的运用，主要还是用来修饰变量，比如返回值、参数。

 **3.修饰变量的区别**

final修饰变量是不可改变的，但它的值可以在运行时刻初始化，也可以在编译时刻初始化，甚至可以放在构造函数中初始化，而不必在声明的时候初始化；而const修饰的全局变量或局部变量，因为是静态，所以无法使用构造方法初始化，必须在声明的时候初始化。



# 隐式转换

二元操作

进行运算时会将两个操作数转换为同一类型，遵循向精度更高的方向转化。

例如：

```
int + double → double + double
int + long → long + long
```

```java
if 存在 double
    则另一个转换为 double
else if 存在 float
    则另一个转换为 float
else if 存在 long
    则另一个转换为 long
else 
    两操作数一起转换为 int
```

在其他情况中，隐式转换发生的情况很少，Java 比 C/C++ 严格的多，例如 long 不会隐式转换为 int。

**显式转换**

与 C/C++ 风格的类型转换完全相同。



#Java 的块作用域

与 C++ 不同， Java 不允许在块内重定义块外的变量。

下面的 Java 代码是无法通过编译的：

```java
//...

int var;
{
    int var;
    //...
}

//...
```





# 条件、循环

与 C/C++ 完全相同。

# for each 循环

对于数组或某个实现了`Iterable`接口的集合，我们可以使用 for each 循环来遍历该集合。

```java
for (Type e : collention){
    //...
}
```

# switch 语句

与 C/C++ 的 switch 完全相同。

不过，Java 提供了带 lambda 表达式的 switch 语句，例如：

```java
switch(/* expression */) {
    case /* constant */ -> {
        //...
    }
    case /* constant */-> {
        //...
    }
    default -> {
        //...
    }    
}
```

# 带标签的 break

goto 语句虽然多遭诟病，但其在跳出深层循环时却意外地简洁。

Java 当然不会引入 goto 语句，但它提供了带标签的 break —— 作为 goto 跳出深层循环的替代方案。

用法大概是这样的：

```java
loop1:
while(/* condition */) {
    loop2:
    for(/* expression */) {
        loop3:
        for(/* expression */) {
            //...
            if(/* condition */) {
                break loop2;
                // 将会跳出第二层循环
            }
            //...
        }
    }
}
```

另外，带标签的 break 可以实现 goto 的部分功能。

例如下面的代码是合法的：

```java
//...

lable:
{
    //...
    if(/* condition */) {
        break lable;
    }
}

//...
```

可以跳出到外层块，但不允许跳入内层块。



# intelliJ IDEA

 **常用快捷键**

| 键                     | 说明                 |
| :--------------------- | :------------------- |
| `Ctrl + Alt + Space`   | 唤出代码补全         |
| `Ctrl + Shift + 方向`  | 移动整行             |
| `Ctrl + D`             | 复制整行             |
| `Ctrl + Y`             | 删除行               |
| `Ctrl + W`             | 扩展选取             |
| `Ctrl + Shift + W`     | 取消扩展选取         |
| `Ctrl + Alt + L`       | 格式化代码           |
| `Shift + Alt`          | 多选                 |
| `鼠标中键`             | 列选择               |
| `Shift + Alt + Insert` | 切换行选择和列选择   |
| `Shift + F6`           | 重构                 |
| `Alt + Enter`          | 导入包，自动修正代码 |
| `Ctrl + Alt + M`       | 抽取方法             |