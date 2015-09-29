title: Object equals hashcode方法解析
date: 2015-09-29 16:47:29
categories: Java
---

## equals
equal是指示其他某个对象是否与此对象“相等”，默认比较的是引用地址的比较或者是内存地址。以下是Object源码中的实现代码：

		public boolean equals(Object obj) {
        return (this == obj);
    	}

要比较对象对内容的话需要重写equals方法，重写了equals时最好也和hashCode方法一起重写，这样可以保证查找的性能。


**摘自JDK API 稳定描述**

equals 方法在非空对象引用上实现相等关系： 

- 自反性：对于任何非空引用值 x，x.equals(x) 都应返回 true。 
- 对称性：对于任何非空引用值 x 和 y，当且仅当 y.equals(x) 返回 true 时，x.equals(y) 才应返回 true。 
- 传递性：对于任何非空引用值 x、y 和 z，如果 x.equals(y) 返回 true，并且 y.equals(z) 返回 true，那么 x.equals(z) 应返回 true。 
- 一致性：对于任何非空引用值 x 和 y，多次调用 x.equals(y) 始终返回 true 或始终返回 false，前提是对象上 equals 比较中所用的信息没有被修改。 
- 对于任何非空引用值 x，x.equals(null) 都应返回 false。

 
Object 类的 equals 方法实现对象上差别可能性最大的相等关系；即，对于任何非空引用值 x 和 y，当且仅当 x 和 y 引用同一个对象时，此方法才返回 true（x == y 具有值 true）。 

**注意：当此方法被重写时，通常有必要重写 hashCode 方法，以维护 hashCode 方法的常规协定，该协定声明相等对象必须具有相等的哈希码。** 

在Java 提供的类库中，很多都已经实现了equals和hashCode方法，比如 AbstractList,AbstractMap,AbstractSet,BigDecimal,String, Math,Double等封装类，具体的实现方式可以参看这些类的实现方式。

**在不允许重复对象的集合使用中，要重写equals和hashCode方法**


## hashCode
hashCode返回该对象的哈希码值。支持此方法是为了提高哈希表（例如 java.util.Hashtable 提供的哈希表）的性能。从下面的代码看出它是一个本地方法。

	public native int hashCode();

如果想看具体的实现可以查看jdk源码synchronizer.cpp文件，我们可以在自己写的类中重写hashCode。想要弄明白hashCode的作用，必须要先知道Java中的集合。

Java中的集合主要有List、Set、Map,List存有允许重复但有序的对象，Set存储无序不允许重复的对象，Map将键映射到值的对象，一个映射不能包含重复的键，每个键最多只能映射到一个值。 

这里引出一个问题，如何保证集合中的对象不重复。这就是Object.equals方法了。但是，如果每增加一个元素就检查一次，那么当元素很多时，后添加到集合中的元素比较的次数就非常多了。也就是说，如果集合中现在已经有1000个元素，那么第1001个元素加入集合时，它就要调用1000次equals方法，这显然会大大降低效率。

于是，Java采用了哈希表的原理。哈希算法也称为散列算法，是将数据依特定算法直接指定到一个地址上，可简单理解为hashCode方法实际上返回的就是对象存储的物理地址（实际可能并不是）。

 这样一来，当集合要添加新的元素时，先调用这个元素的hashCode方法，就一下子能定位到它应该放置的物理位置上。如果这个位置上没有元素，它就可以直接存储在这个位置上，不用再进行任何比较了。如果这个位置上已经有元素了，就调用它的equals方法与新元素进行比较，相同的话就不存了，不相同就散列其它的地址。所以这里存在一个冲突解决的问题。这样一来实际调用equals方法的次数就大大降低了，几乎只需要一两次。

所以，Java对于eqauls方法和hashCode方法是这样规定的：

      1.如果两个对象相同，那么它们的hashCode值一定要相同；
      2.如果两个对象的hashCode相同，它们并不一定相同（这里说的对象相同指的是用eqauls方法比较）。  
        如不按要求去做了，会发现相同的对象可以出现在Set集合中，同时，增加新元素的效率会大大下降。
      3.equals()相等的两个对象，hashcode()一定相等；equals()不相等的两个对象，却并不能证明他们的hashcode()不相等。


换句话说，equals()方法不相等的两个对象，hashcode()有可能相等（我的理解是由于哈希码在生成的时候产生冲突造成的）。反过来，hashcode()不等，一定能推出equals()也不等；hashcode()相等，equals()可能相等，也可能不等。

 在object类中，hashcode()方法是本地方法，返回的是对象的地址值，而object类中的equals()方法比较的也是两个对象的地址值，如果equals()相等，说明两个对象地址值也相等，当然hashcode()也就相等了；在String类中，equals()返回的是两个对象内容的比较，当两个对象内容相等时，Hashcode()方法根据String类的重写代码的分析，也可知道hashcode()返回结果也会相等。以此类推，可以知道Integer、Double等封装类中经过重写的equals()和hashcode()方法也同样适合于这个原则。当然没有经过重写的类，在继承了object类的equals()和hashcode()方法后，也会遵守这个原则。


## 集合中的equals和hashCode
Hashset是继承Set接口，Set接口又实现Collection接口，这是层次关系。那么Hashset、Hashmap、Hashtable中的存储操作是根据什么原理来存取对象的呢？

下面以HashSet为例进行分析，我们都知道：在hashset中不允许出现重复对象，元素的位置也是不确定的。在hashset中又是怎样判定元素是否重复的呢？在java的集合中，判断两个对象是否相等的规则是：

 1.判断两个对象的hashCode是否相等

    如果不相等，认为两个对象也不相等，完毕
    如果相等，转入2
   （这一点只是为了提高存储效率而要求的，其实理论上没有也可以，但如果没有，实际使用时效率会大大降低，所以我们这里将其做为必需的。）

 2.判断两个对象用equals运算是否相等
            
	如果不相等，认为两个对象也不相等
	如果相等，认为两个对象相等（equals()是判断两个对象是否相等的关键）
	为什么是两条准则，难道用第一条不行吗？不行，因为前面已经说了，hashcode()相等时，equals()方法也可能不等，
	所以必须用第2条准则进行限制，才能保证加入的为非重复元素。



## 重写equals()和hashcode()小结

1. 重点是equals，重写hashCode只是技术要求（为了提高效率）

2. 为什么要重写equals呢？因为在java的集合框架中，是通过equals来判断两个对象是否相等的

3. 在hibernate中，经常使用set集合来保存相关对象，而set集合是不允许重复的。在向HashSet集合中添加元素时，其实只要重写equals()这一条也可以。但当hashset中元素比较多时，或者是重写的equals()方法比较复杂时，我们只用equals()方法进行比较判断，效率也会非常低，所以引入了hashCode()这个方法，只是为了提高效率，且这是非常有必要的。比如可以这样写：

		public int hashCode(){
		   return 1; //等价于hashcode无效
		}

这样做的效果就是在比较哈希码的时候不能进行判断，因为每个对象返回的哈希码都是1，每次都必须要经过比较equals()方法后才能进行判断是否重复，这当然会引起效率的大大降低。