---
title: C# List<T>
date: 2023-04-27 17:13:22
tags: CSharp
categories: Languages
math: false
---

# List\<T\>

泛型 List 在底层实现中是由数组来承载数据，所以又被称为“动态数组”。数组的大小即为容量(capacity)，可以手动或自动地调整。当容量不足时，会自动创建一个更长的数组，将原数组中的内容复制到新的数组中实现扩容。默认新创建的不含任何元素的 List 容量为 0，加入一个元素后容量为 4，容量不足时进行二倍扩容，即容量变为 8、16、32... 当 List 调用 Clear 方法后，其数组长度即容量不变。泛型 List 与非泛型的 ArrayList 相对应，后者元素都为 object 类型。

<style>
.center 
{
  width: auto;
  display: table;
  margin-left: auto;
  margin-right: auto;
}
</style>

<div class="center">

| Operation | Category    | Member                               |
| :-------- | :---------- | :----------------------------------- |
| Create    | Constructor | List\<T\>()                          |
| Create    | Constructor | List\<T\>(IEnumerable\<T\>)          |
| Create    | Constructor | List\<T\>(Int32)                     |
| Create    |             | Add(T)                               |
| Create    |             | AddRange(IEnumerable\<T\>)           |
| Create    |             | Insert(Int32, T)                     |
| Create    |             | InsertRange(Int32, IEnumerable\<T\>) |
| Delete    |             | Clear()                              |
| Delete    |             | RemoveAt(Int32)                      |
| Delete    |             | RemoveRange(Int32, Int32)            |
| Delete    |             | Remove(T)                            |
| Delete    |             | RemoveAll(Predicate\<T\>)            |

</div>

* Add(T) 和 AddRange(IEnumerable\<T\>) 是在 List 最后添加新的元素。
* Insert(Int32, T) 和 InsertRange(Int32, IEnumerable\<T\>) 是在指定位置插入元素，会造成被插入位置和其后的所有元素整体后移。
* RemoveAt(Int32) 和 RemoveRange(Int32, Int32) 是移除指定位置的元素，会造成被移除元素位置之后的所有元素整体前移。
* Remove(T) 是移除特定对象的第一个匹配项，RemoveAll(Predicate\<T\>) 的参数是一个参数为 T，返回值为 bool 类型的委托，它会移除与指定的委托所定义的条件相匹配的所有元素。也会造成被移除元素位置之后的所有元素整体前移。

<div class="center">

| Operation | Category | Member                 |
| :-------- | :------- | :--------------------- |
| Read      |          | Count                  |
| Read      |          | Capacity               |
| Read      |          | Item\[Int32\]          |
| Read      |          | GetRange(Int32, Int32) |
| Read      | Iterate  | GetEnumerator()        |
| Read      | Iterate  | ForEach(Action\<T\>)   |

</div>

* Count 是当前列表中元素的个数，Capacity 是列表容量，即数组的长度，Item\[Int32\] 是索引器，即可以使用数组下标访问元素。
* GetRange(Int32, Int32) 参数分别为起始索引和终止索引，将当前列表中从起始索引到终止索引的元素加入子列表，将子列表返回。
* GetEnumerator() 拿到迭代器，调用返回值为 bool 类型的 MoveNext() 方法进行遍历，迭代器的指针默认指向的并不是 list 中的第一个元素，而是第一个元素之前的位置，遍历所有元素后会指向 list 之外的位置，所以遍历 list 最好使用 foreach。
* 使用 foreach 进行遍历时不可以添加或者删除元素，可以修改元素内部的属性值，但是不可以修改元素本身的值。
  
```csharp
List<int> intList = new List<int>{100, 200, 300, 400};
var e = intList.GetEnumerator();
Console.WriteLine(e.Current);
while(e.MoveNext()){
    Console.WriteLine(e.Current);
}
Console.WriteLine(e.Current);
//输出：
//0
//100
//200
//300
//400
//0

foreach(var val in intList){
    Console.WriteLine(val);
    //val++; //invalid 不可以修改元素本身的值
}
//输出：
//100
//200
//300
//400

class Book{
    public int Id{get;set;}
    public string Name{get;set;}
    public float Price{get;set;}
}
List<Book> bookList = new List<Book>();
for(int i = 0; i < 10; i++;)
    bookList.Add(new Book{Id = i, Name = $"Book-{i}", Price = 10 * i});

foreach(var val in bookList){
    val.Price++; //valid 可以修改元素的属性的值 
}
```

<div class="center">

| Operation | Category | Member                       |
| :-------- | :------- | :--------------------------- |
| Read      |          | Contains(T)                  |
| Read      |          | Exists(Predicate\<T\>)       |
| Read      |          | TrueForAll(Predicate\<T\>)   |
| Read      |          | IndexOf(T)                   |
| Read      |          | IndexOf(T, Int32)            |
| Read      |          | IndexOf(T, Int32, Int32)     |
| Read      |          | LastIndexOf(T)               |
| Read      |          | LastIndexOf(T, Int32)        |
| Read      |          | LastIndexOf(T, Int32, Int32) |

</div>

* Contains(T) 确定某元素是否在列表中，返回一个 bool 值，如果 T 是引用类型，那么参数可以为 null。这里判断相等的方法是调用底层的 Equals 方法，我们可以重写 Equals 方法来改变判断的结果，从而影响 Contains 方法返回的值。
* Exists(Predicate\<T\>) 确定是否含有与指定的委托所定义的条件相匹配的元素。
* TrueForAll(Predicate\<T\>) 确定所有元素是否与指定的委托所定义的条件相匹配。
* IndexOf(T) 返回整个列表中第一个与参数相等的元素的从零开始的索引。IndexOf(T, Int32) 返回整个列表中第一个与参数相等的元素的从指定位置开始的索引。IndexOf(T, Int32, Int32) 第二个参数为搜索开始的索引，第三个参数为要搜索的索引数。IndexOf(Item, 2, 3) 即表示从索引为 2 的位置开始向后搜索三个元素，并返回第一个与 Item 相等的元素的索引值。如果没有相等的元素，那么返回 -1，这里判断相等的方法也是调用底层的 Equals 方法。
* LastIndexOf 方法和 IndexOf 方法相似，区别是 IndexOf 方法是从前往后找，LastIndexOf 方法是从后往前找。


<div class="center">

| Operation | Category  | Member                                        |
| :-------- | :-------- | :-------------------------------------------- |
| Read      |           | Find(Predicate\<T\>)                          |
| Read      |           | FindLast(Predicate\<T\>)                      |
| Read      |           | FindAll(Predicate\<T\>)                       |
| Read      |           | FindIndex(Int32, Int32, Predicate\<T\>)       |
| Read      |           | FindIndex(Int32, Predicate\<T\>)              |
| Read      |           | FindIndex(Predicate\<T\>)                     |
| Read      |           | FindLastIndex(Int32, Int32, Predicate\<T\>)   |
| Read      |           | FindLastIndex(Int32, Predicate\<T\>)          |
| Read      |           | FindLastIndex(Predicate\<T\>)                 |
| Read      | Algorithm | BinarySearch(Int32, Int32, T, IComparer\<T\>) |
| Read      | Algorithm | BinarySearch(T)                               |
| Read      | Algorithm | BinarySearch(T, IComparer\<T\>)               |

</div>

* Find(Predicate\<T\>) 方法返回与指定的委托所定义的条件相匹配的元素，如果没有相匹配的元素，则返回类型 T 的默认值。
* FindLast(Predicate\<T\>) 与 Find(Predicate\<T\>) 相似，区别是 Find 方法是从前往后找，FindLast 方法是从后往前找。
* FindAll(Predicate\<T\>) 方法返回一个新的列表，其元素为与指定的委托所定义的条件相匹配的所有元素。
* FindIndex(Int32, Int32, Predicate\<T\>) 搜索与指定委托所定义的条件相匹配的一个元素，并返回列表中从指定的索引开始、包含指定元素个数的元素范围内第一个匹配项的从零开始的索引。
* FindIndex(Int32, Predicate\<T\>) 搜索与指定委托所定义的条件相匹配的元素，并返回列表中从指定索引到最后一个元素的元素范围内第一个匹配项的从零开始的索引。
* FindIndex(Predicate\<T\>) 搜索与指定委托所定义的条件相匹配的元素，并返回整个列表中第一个匹配元素的从零开始的索引。
* FindLastIndex 方法与 FindIndex 方法相似，区别是 FindIndex 方法是从前往后找，FindLastIndex 方法是从后往前找。
* BinarySearch 二分查找，调用者需为有序列表，即二分查找前要么使用 Sort() 方法排序，这时排序的对象需要实现 IComparable 泛型接口，即实现 CompareTo 方法；要么使用带 IComparer\<T\> 参数的重载，使用此比较器进行排序。而二分查找时需调用底层的 Equals 方法来判定是否相等。BinarySearch 方法返回的是已经排好序的列表中相等元素的索引值。