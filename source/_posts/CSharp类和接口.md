---
title: C# 类和接口
date: 2023-04-24 15:30:32
tags: CSharp
categories: Languages
math: false
---

# 类(Class)


## 什么是类


* 一种数据结构

* 一种数据类型

* 代表现实世界中的种类


## 构造函数和析构函数

### 构造函数

public ClassName(){}
在被new实例化时自动调用此构造器

private ClassName(){}
用于防止类被new ClassName()实例化

static ClassName(){}
只用于构造静态成员，不能用于构造实例成员

###  析构函数

~ClassName(){}
在被GC回收托管资源时调用此析构器

## 抽象类

* 不能被实例化，但是有构造函数

* 抽象成员必须包含在抽象类中

* 抽象类除了抽象成员外，还可以包含别的成员(不用关键字 abstract)

* 子类继承抽象父类后，必须把父类中所有抽象成员都重写(非抽象成员不必重写)。除非子类也是个抽象类

* 抽象成员的访问修饰符不能是private
  
* abstract关键字不能用于字段成员，但是可以用于属性
  
* abstract关键字用于方法时，方法不能定义主体
  
* 抽象类也可实现接口，但要将接口的成员用abstract修饰


## 修饰符

### 访问限制

* private：访问级别为类的成员，不能直接修饰类，仅当该类是其他类的成员的时候可以修饰。

* public：访问没有限制，所有的本程序集以及其他的程序集都能够访问。

* internal：本程序集内的成员可以访问，是默认的访问级别。

### 继承相关

* sealed：封闭类，修饰类时表示该类不能够被继承。

* protected：类的访问级别被限制在类成员之间，例如当父类成员被protect修饰，子类成员可以访问，其他类不能访问。可以跨程序集。

* abstract：修饰类的时候表示该类为抽象类，不能创建该类的实例。修饰方法的时候表示该方法需要由子类来实现，如果子类没有实现该方法那么子类同样是抽象类；且含有抽象方法的类一定是抽象类。

* new：只能用于嵌套的类，表示对继承父类同名类型的隐藏。

### 其他

* static：修饰类时表示该类为静态类，不能实例化该类的对象，这个类也不能含有对象成员，即该类的所有成员为静态。

* partial：部分类，可以将一个类分成几部分写在不同的文件中，最终编译时将合并成一个文件，且各个部分不能分散在不同程序集中。


## 其他补充


* 父类构造器不能直接被子类全盘继承，调用时先调用父类的构造器再调用子类的构造器

```csharp
class Vehicle {
    public Vehicle(string owner){
        this.Owner = owner;
    }
    public string Owner {get; set;}
}

class Car : Vehicle {
    public Car(string owner) : base(owner){}
    public void ShowOwner(){
        Console.WriteLine(Owner);
    }
}
\\new 一个新 Car 对象时先执行 Vehicle 的构造函数，再执行 Car 的构造函数
```

* 子类的访问级别不能超越父类的访问级别

* 父类中的虚函数(virtual)可以有方法体，被子类继承后可以重写(override)。而抽象(abstract)方法无函数体，含有抽象方法的类一定是抽象类，继承抽象类的子类必须把抽象父类中的所有抽象方法都重写，除非子类也是个抽象类。

* 不通过构造器创建实例的方法：
```csharp
Type t = typeof(ClassName);
object o = Activator.CreateInstance(t);
ClassName className = o as ClassName;
```

# 接口(Interface)

* 接口中只能包含方法(方法、属性、索引器、事件)，且接口中的方法不能有任何实现。

* 接口中的成员不能有任何访问修饰符，默认为public的
  
* 接口不能被实例化
  
* 实现接口的类，必须实现接口的所有成员，而且没有 override 关键字。
  
* 一个接口可以继承多个接口
  
* 一个类可以实现多个接口，但是只能继承一个类
  
* 如果一个类同时继承了父类并实现接口的时候，要把父类写在最前面
  
* 当父类实现了接口，子类继承父类后，则可以使用接口引用子类

* 接口方法的显示实现：如果多个接口里有相同的方法名，那么第一个方法名默认的引用的是排在最前面的接口，如果要使用后面接口的方法，就要显示实现：接口名.方法名。访问修饰符是private

# 其他

* 结构体可以实现接口，但是不能继承其他类或者其他结构体；结构体不能有显式的无参构造器，但是可以有显式的有参构造器。

* 枚举成员默认类型是 int 类型，通过继承可以声明枚举成员为其他类型，枚举类型一定是继承自 byte、sbyte、short、ushort、int、uint、long 和 ulong 中的一种，不能是其它类型。比如：

```csharp
public enum Days : byte
{
    Monday = 1,
    Tuesday = 2,
    Wednesday = 3,
    Thursday = 4,
    Friday = 5,
    Saturday = 6,
    Sunday = 7
}
```

* 枚举成员如果没有手动指定成员的值的话，就是一个从 0 开始，步长为 1 的等差数列，即 1，2，3，4，5...

* 枚举成员如果被手动赋值的话，那么被赋值的成员的值即为所指定的值，但是其递增步长不会变，总是为 1：
  
```csharp
public enum DriveType : sbyte 
{     
    CDRom,                                  //值为0
    Fixed = -2,                             //值为-2
    Network,                                //值为-1
    NoRootDirectory = -1,                   //值为-1
    Ram,                                    //值为0
    Removable = Network * NoRootDirectory,  //值为1
    Unknown                                 //值为2
}  
```

* Flags关键字允许我们在使用枚举变量时,可以使用多个组合值。

```csharp
[Flags]
public enum Options : byte
{
    None    = 0,
    One     = 1 << 0,   // 1 0b1
    Two     = 1 << 1,   // 2 0b10
    Three   = 1 << 2,   // 4 0b100
    Four    = 1 << 3,   // 8 0b1000
}
static void Main(){
    Options option = Options.None | Options.One | Options.Two | Options.Three | Options.Four;
    Console.WriteLine(option.ToString());
    //输出：One, Two, Three, Four  如果不加 Flags 的特性，输出：15
    Console.WriteLine((int)option);
    //输出：15
    Console.WriteLine(option.HasFlag(Options.None));
    //输出：True
    Console.WriteLine(option.HasFlag(Options.One));
    //输出：True

    option = Options.One | Options.Two;
    Console.WriteLine(option.ToString());
    //输出：One, Two  如果不加 Flags 的特性，输出：3
    Console.WriteLine((int)option)
    //输出：3
    Console.WriteLine(option.HasFlag(Options.None));
    //输出：True
    Console.WriteLine(option.HasFlag(Options.One));
    //输出：True
     Console.WriteLine(option.HasFlag(Options.Three));
    //输出：False
}
```
所以有 [Flags] 特性时尽量不要将成员的值设为 0。