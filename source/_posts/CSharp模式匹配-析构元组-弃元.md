---
title: C# 模式匹配、析构元组和弃元
date: 2023-04-26 10:24:57
tags: CSharp
categories: Languages
math: false
---

# 模式匹配

“模式匹配”是一种测试表达式是否具有特定特征的方法。 C# 模式匹配提供更简洁的语法，用于测试表达式并在表达式匹配时采取措施。 “is 表达式”目前支持通过模式匹配测试表达式并有条件地声明该表达式结果。 “switch 表达式”允许你根据表达式的首次匹配模式执行操作。 这两个表达式支持丰富的模式词汇。

## Null 检查

模式匹配最常见的方案之一是确保值不是 null。 使用以下示例进行 null 测试时，可以测试可为 null 的值类型并将其转换为其基础类型：

```csharp
int? maybe = 12;

if (maybe is int number)
{
    Console.WriteLine($"The nullable int 'maybe' has the value {number}");
}
else
{
    Console.WriteLine("The nullable int 'maybe' doesn't hold a value");
}
```

只判断是否为 null 值可以使用 not 模式

```csharp
string? message = "This is not the null string";

if (message is not null)
{
    Console.WriteLine(message);
}
```

## 类型测试

以下代码测试实现 IEnumerable 泛型接口的参数是否为 null 值，如果不是 null 值那么测试它是否也实现了 IList 泛型接口，并根据是否实现 IList 接口来查找中间索引。

```csharp
public static T MidPoint<T>(IEnumerable<T> sequence)
{
    if (sequence is null)
    {
        throw new ArgumentNullException(nameof(sequence), "Sequence can't be null.");
    }
    else if (sequence is IList<T> list)
    {
        return list[list.Count / 2];
    }
    else
    {
        int halfLength = sequence.Count() / 2 - 1;
        if (halfLength < 0) halfLength = 0;
        return sequence.Skip(halfLength).First();
    }
}
```

## 比较离散值

可以代替 switch 来对枚举中声明的所有可能值进行数值测试：

```csharp
public enum State
{
    Run,
    Idle,
    Jump
}

public string PerformState(State state)
{
    return state switch
    {
        State.Idle => "Idle",
        State.Run => "Run",
        State.Jump => "Jump",
        _ => "Invalid enum value for State"
    };
}
```
_ 案例为与所有数值匹配的弃元模式。 它处理值与定义的 enum 值之一不匹配的任何错误条件。

{% note warning %}
switch表达式是输出语句！
{% endnote %}


## 关系模式

你可以使用关系模式测试如何将数值与常量进行比较。 例如，以下代码基于华氏温度返回水源状态：

```csharp
string WaterState(int tempInFahrenheit) =>
    tempInFahrenheit switch
    {
        (> 32) and (< 212) => "liquid",
        < 32 => "solid",
        > 212 => "gas",
        32 => "solid/liquid transition",
        212 => "liquid / gas transition",
    };
```

## 多个输入

可以写入检查一个对象的多个属性的模式：

```csharp
class Order
{
    public int Items;
    public decimal Cost;
}

decimal CalculateDiscount(Order order)
{
    return order switch
    {
        { Items: > 10, Cost: > 1000.00m } => 0.10m,
        { Items: > 5, Cost: > 500.00m } => 0.05m,
        { Cost: > 250.00m } => 0.02m,
        null => throw new ArgumentNullException(nameof(order), "Can't calculate discount on null order"),
        var someObject => 0m,
    };
}
```

## 列表模式

可以使用列表模式检查列表或数组中的元素。 列表模式提供了一种方法，将模式应用于序列的任何元素。 此外，还可以应用弃元模式 (_) 来匹配任何元素，或者应用切片模式来匹配零个或多个元素。

当数据不遵循常规结构时，列表模式是一个有价值的工具。 可以使用模式匹配来测试数据的形状和值，而不是将其转换为一组对象。

看看下面的内容，它摘录自一个包含银行交易信息的文本文件：

```
04-01-2020, DEPOSIT,    Initial deposit,            2250.00
04-15-2020, DEPOSIT,    Refund,                      125.65
04-18-2020, DEPOSIT,    Paycheck,                    825.65
04-22-2020, WITHDRAWAL, Debit,           Groceries,  255.73
05-01-2020, WITHDRAWAL, #1102,           Rent, apt, 2100.00
05-02-2020, INTEREST,                                  0.65
05-07-2020, WITHDRAWAL, Debit,           Movies,      12.57
04-15-2020, FEE,                                       5.55
```

它是 CSV 格式，但某些行的列数比其他行要多。 对处理来说更糟糕的是，WITHDRAWAL 类型中的一列具有用户生成的文本，并且可以在文本中包含逗号。 一个包含弃元模式、常量模式和 var 模式的列表模式用于捕获这种格式的值处理数据：

```csharp
decimal balance = 0m;
foreach (string[] transaction in ReadRecords())
{
    balance += transaction switch
    {
        [_, "DEPOSIT", _, var amount]     => decimal.Parse(amount),
        [_, "WITHDRAWAL", .., var amount] => -decimal.Parse(amount),
        [_, "INTEREST", var amount]       => decimal.Parse(amount),
        [_, "FEE", var fee]               => -decimal.Parse(fee),
        _                                 => throw new InvalidOperationException($"Record {string.Join(", ", transaction)} is not in the expected format!"),
    };
    Console.WriteLine($"Record: {string.Join(", ", transaction)}, New balance: {balance:C}");
}
```
前面的示例采用了字符串数组，其中每个元素都是行中的一个字段。 第二个字段的 switch 表达式键，用于确定交易的类型和剩余列数。 每一行都确保数据的格式正确。 弃元模式 (_) 跳过第一个字段，以及交易的日期。 第二个字段与交易的类型匹配。 其余元素匹配跳过包含金额的字段。 最终匹配使用 var 模式来捕获金额的字符串表示形式。 表达式计算要从余额中加上或减去的金额。

# 析构元组

## 元组

元组提供一种从方法调用中检索多个值的轻量级方法。 

```csharp
public class Example
{

    private static (string, int, double) QueryCityData(string name)
    {
        if (name == "New York City")
            return (name, 8175133, 468.48);

        return ("", 0, 0);
    }

    public static void Main()
    {
        var result = QueryCityData("New York City");

        //一旦检索到元组，就必须处理它的各个元素。 按元素逐个操作比较麻烦
        var city1 = result.Item1;
        var pop1 = result.Item2;
        var size1 = result.Item3;

        //可以在括号内显式声明每个字段的类型。
        (string city2, int population2, double area2) = QueryCityData("New York City");
         
        //可使用 var 关键字，以便 C# 推断每个变量的类型。 将 var 关键字放在括号外。
        var (city3, population3, area3) = QueryCityData("New York City");

        //还可在括号内将 var 关键字单独与任一或全部变量声明结合使用。
        (string city4, var population4, var area4) = QueryCityData("New York City");

        //可将元组析构到已声明的变量中。
        string city5 = "Raleigh";
        int population5 = 458880;
        double area5 = 144.8;

        (city5, population5, area5) = QueryCityData("New York City");

        //从 C# 10 开始，可在析构中混合使用变量声明和赋值。
        string city = "Raleigh";
        int population = 458880;

        (city, population, double area) = QueryCityData("New York City");
    }
}
```

## 使用弃元的元组元素

析构元组时，通常只需要关注某些元素的值。 可以利用 C# 对弃元的支持，弃元是一种仅能写入的变量，且其值将被忽略。 在赋值中，通过下划线字符 (_) 指定弃元。 可弃元任意数量的值，且均由单个弃元 _ 表示。

```csharp
var (_, _, area3) = QueryCityData("New York City");
```

## 用户定义类型

用户作为类、结构或接口的创建者，可通过实现一个或多个 Deconstruct 方法来析构该类型的实例。 该方法返回 void，且要析构的每个值由方法签名中的 out 参数指示。

```csharp
using System;

public class Person
{
    public string FirstName { get; set; }
    public string MiddleName { get; set; }
    public string LastName { get; set; }
    public string City { get; set; }
    public string State { get; set; }

    public Person(string fname, string mname, string lname,
                  string cityName, string stateName)
    {
        FirstName = fname;
        MiddleName = mname;
        LastName = lname;
        City = cityName;
        State = stateName;
    }

    // Return the first and last name.
    public void Deconstruct(out string fname, out string lname)
    {
        fname = FirstName;
        lname = LastName;
    }

    public void Deconstruct(out string fname, out string mname, out string lname)
    {
        fname = FirstName;
        mname = MiddleName;
        lname = LastName;
    }

    public void Deconstruct(out string fname, out string lname,
                            out string city, out string state)
    {
        fname = FirstName;
        lname = LastName;
        city = City;
        state = State;
    }
}

public class ExampleClassDeconstruction
{
    public static void Main()
    {
        var p = new Person("John", "Quincy", "Adams", "Boston", "MA");

        // Deconstruct the person object.
        var (fName, lName, city, state) = p;
        Console.WriteLine($"Hello {fName} {lName} of {city}, {state}!");


        var (fName2, _, city2, _) = p;
        Console.WriteLine($"Hello {fName2} of {city2}!");
    }
}
```

# 弃元

可使用独立弃元来指示要忽略的任何变量。 一种典型的用法是使用赋值来确保一个参数不为 null。 下面的代码使用弃元来强制赋值。 赋值的右侧使用 Null 合并操作符，用于在参数为 null 时引发 System.ArgumentNullException。 此代码不需要赋值结果，因此将对其使用弃元。 该表达式强制执行 null 检查。 弃元说明你的意图：不需要或不使用赋值结果。

```csharp
public static void Method(string arg)
{
    _ = arg ?? throw new ArgumentNullException(paramName: nameof(arg), message: "arg can't be null");

    // Do work with arg.
}
```


{% note info %}
`?? 操作符和 ??= 操作符`

如果左操作数的值不为 null，则 null 合并运算符 ?? 返回该值；否则，它会计算右操作数并返回其结果。 如果左操作数的计算结果为非 null，则 ?? 运算符不会计算其右操作数。 仅当左操作数的计算结果为 null 时，Null 合并赋值运算符 ??= 才会将其右操作数的值赋值给其左操作数。 如果左操作数的计算结果为非 null，则 ??= 运算符不会计算其右操作数。
{% endnote %}