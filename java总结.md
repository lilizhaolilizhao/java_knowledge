### 1 String 和 StringBuffer、StringBuilder 的区别
可变性

String 类中使用字符数组保存字符串，private final char value[]，所以 string 对象是不可变的。StringBuilder 与 StringBuffer 都继承自AbstractStringBuilder 类，在 AbstractStringBuilder 中也是使用字符 数组保存字符串，char[] value，这两种对象都是可变的。

线程安全性

String 中的对象是不可变的，也就可以理解为常量，线程安全。 AbstractStringBuilder 是 StringBuilder 与 StringBuffer 的公共父类，定义了一 些字符串的基本操作，如 expandCapacity、append、insert、indexOf 等公共 方法。StringBuffer 对方法加了同步锁或者对调用的方法加了同步锁，所以是线 程安全的。StringBuilder 并没有对方法进行加同步锁，所以是非线程安全的。 

性能

每次对 String 类型进行改变的时候，都会生成一个新的 String 对象，然后将 指针指向新的 String 对象。StringBuffer 每次都会对StringBuffer 对象本身进行操作，而不是生成新的对象并改变对象引用。相同情况下使用StirngBuilder 相比使用StringBuffer 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

### 2 hashCode 和 equals 方法的关系
equals 相等，hashcode 必相等;hashcode 相等，equals 可能不相等。

### 3 抽象类和接口的区别
语法层次

抽象类和接口分别给出了不同的语法定义。

设计层次

抽象层次不同，抽象类是对类抽象，而接口是对行为的抽象。抽象类是对整个 类整体进行抽象，包括属性、行为，但是接口却是对类局部(行为)进行抽 象。抽象类是自底向上抽象而来的，接口是自顶向下设计出来的。

跨域不同 

抽象类所体现的是一种继承关系，要想使得继承关系合理，父类和派生类之间 必须存在"is-a" 关系，即父类和派生类在概念本质上应该是相同的。对于接口则不然，并不要 求接口的实现者和接口定义在概念本质上是一致的，仅仅是实现了接口定义的契约而已，"like-a"的关系。

### 4 为什么要使用以及泛型擦除
Java 编译器生成的字节码是不包涵泛型信息的，泛型类型信息将在编译处理是 被擦除，这个过程即类型擦除。泛型擦除可以简单的理解为将泛型 java 代码转 换为普通 java 代码，只不过编译器更直接点，将泛型 java 代码直接转换成普 通 java 字节码。

### 5 HashMap 实现原理
1. HashMap 概述:
HashMap 是基于哈希表的 Map 接口的非同步实现。此实现提供所有可选的映射操作，
并允许使用 null 值和 null 键。此类不保证映射的顺序，特别是它不保证该顺序恒久不变。2. HashMap 的数据结构:

在 java 编程语言中，最基本的结构就是两种，一个是数组，另外一个是模拟指针(引
用)，所有的数据结构都可以用这两个基本结构来构造的，HashMap 也不例外。 HashMap 实际上是一个“链表散列”的数据结构，即数组和链表的结合体。
当我们往 HashMap 中 put 元素的时候，先根据 key 的 hashCode 重新计算 hash 值，根据 hash 值得到这个元素在数组中的位置(即下标)，如 果数组该位置上已经存放有其他元素了，那么在这个位置上的元素将以链表的形式存放， 新加入的放在链头，最先加入的放在链尾。如果数组该位置上没有元素，就直接将该元素 放到此数组中的该位置上。

hash(int h)方法根据 key 的 hashCode 重新计算一次散列。此算法加入了高位计算，防 止低位不变，高位变化时，造成的 hash 冲突。

		1. static int hash(int h) {
		2.     h ^= (h >>> 20) ^ (h >>> 12);
		3.     return h ^ (h >>> 7) ^ (h >>> 4);
		4. }

把 hash 值对数组长度取模运 算，这样一来，元素的分布相对来说是比较均匀的。但是，“模”运算的消耗还是比较大 的，在 HashMap 中是这样做的:调用 indexFor(int h, int length) 方法来计算该对象应该保 存在 table 数组的哪个索引处。indexFor(int h, int length) 方法的代码如下:

		1. static int indexFor(int h, int length) { 
		2. return h & (length-1);
		3. }

而当数组长度为 16 时，即为 2 的 n 次方时，2n-1 得到的二进制数的每个 位上的值都为 1，这使得在低位上&时，得到的和原 hash 的低位相同，加之 hash(int h)方 法对 key 的 hashCode 的进一步优化，加入了高位计算，就使得只有相同的 hash 值的两 个值才会被放到数组中的同一个位置上形成链表。

### 6 HashMap 的 resize(rehash):
当 HashMap 中的元素越来越多的时候，hash 冲突的几率也就越来越高，因为数组的长
度是固定的。所以为了提高查询的效率，就要对 HashMap 的数组进行扩容，数组扩容这 个操作也会出现在 ArrayList 中，这是一个常用的操作，而在 HashMap 数组扩容之后，最 消耗性能的点就出现了:原数组中的数据必须重新计算其在新数组中的位置，并放进去， 这就是 resize。

那么 HashMap 什么时候进行扩容呢?当 HashMap 中的元素个数超过数组大小 *loadFactor 时，就会进行数组扩容，loadFactor 的默认值为 0.75，这是一个折中的取值。 也就是说，默认情况下，数组大小为 16，那么当 HashMap 中元素个数超过 16*0.75=12 的时候，就把数组的大小扩展为 2*16=32，即扩大一倍，然后重新计算每个元素在数组中 的位置，而这是一个非常消耗性能的操作，所以如果我们已经预知 HashMap 中元素的个 数，那么预设元素的个数能够有效的提高 HashMap 的性能。

### 7 HashMap 的性能参数:
HashMap 包含如下几个构造器:

HashMap():构建一个初始容量为 16，负载因子为 0.75 的 HashMap。 HashMap(int initialCapacity):构建一个初始容量为 initialCapacity，负载因子为 0.75 的 HashMap。

HashMap(int initialCapacity, float loadFactor):以指定初始容量、指定的负载因子创建一个 HashMap。

HashMap 的基础构造器 HashMap(int initialCapacity, float loadFactor)带有两个参数，它们是初始容量 initialCapacity 和加载因子 loadFactor。

initialCapacity:HashMap 的最大容量，即为底层数组的长度。

loadFactor:负载因子 loadFactor 定义为:散列表的实际元素数目(n)/ 散列表的容量 (m)。

负载因子衡量的是一个散列表的空间的使用程度，负载因子越大表示散列表的装填程度 越高，反之愈小。对于使用链表法的散列表来说，查找一个元素的平均时间是 O(1+a)，因 此如果负载因子越大，对空间的利用更充分，然而后果是查找效率的降低;如果负载因子 太小，那么散列表的数据将过于稀疏，对空间造成严重浪费。

HashMap 的实现中，通过 threshold 字段来判断 HashMap 的最大容量:
		
		1. threshold = (int)(capacity * loadFactor);
	
结合负载因子的定义公式可知，threshold 就是在此 loadFactor 和 capacity 对应下允许 的最大元素数目，超过这个数目就重新 resize，以降低实际的负载因子。默认的的负载因 子 0.75 是对空间和时间效率的一个平衡选择。当容量超出此最大容量时， resize 后的 HashMap 容量是容量的两倍:

	1. if (size++ >= threshold)
	2. resize(2 * table.length);

### Fail-Fast 机制:
我们知道 java.util.HashMap 不是线程安全的，因此如果在使用迭代器的过程中有其他线
程修改了 map，那么将抛出 ConcurrentModificationException，这就是所谓 fail-fast 策 略。

这一策略在源码中的实现是通过 modCount 域，modCount 顾名思义就是修改次数，对 HashMap 内容的修改都将增加这个值，那么在迭代器初始化过程中会将这个值赋给迭代器 的 expectedModCount。

	1. HashIterator() {
	2. 		expectedModCount = modCount;
	3. 		if (size > 0) { // advance to first entry
	4. 			Entry[] t = table;
	5. 		while (index < t.length && (next = t[index++]) == null)
	6. 			;
	7. 		}
	8. }

在迭代过程中，判断 modCount 跟 expectedModCount 是否相等，如果不相等就表示已 经有其他线程修改了 Map:
注意到 modCount 声明为 volatile，保证线程之间修改的可见性。

	1. final Entry<K,V> nextEntry() {
	2. 		if (modCount != expectedModCount)
	3. 			throw new ConcurrentModificationException();

### 8 当两个对象的 hashcode 相同会发生什么?如果两个键的 hashcode 相同，你如何获取值对象?如 果 HashMap 的大小超过了负载因子(load factor)定义的容量，怎么办?你了解重新调整 HashMap 大小存在 什么问题吗?
因为 hashcode 相同，所以它们的 bucket 位置相同，‘碰撞’会发生。因为 HashMap 使用 LinkedList 存储对象，这个 Entry(包含有键值对的 Map.Entry 对象)会存储在 LinkedList 中。

找到 bucket 位置之后，会调用 keys.equals()方法去找到 LinkedList 中正确的节点，最终找到要找的值对象

当一个 map 填满了 75%的 bucket 时候，和其它集合类(如 ArrayList 等)一 样，将会创建原来 HashMap 大小的两倍的 bucket 数组，来重新调整 map 的大小，并
将原来的对象放入新的 bucket 数组中。这个过程叫作 rehashing，因为它调用 hash 方 法找到新的 bucket 位置。

当重新调整 HashMap 大小的时候，确实存在条件竞争，因为如果两个线程都发现 HashMap 需要重新调整大小了，它们会同时试着调整大小。在调整大小的过程中，存储 在 LinkedList 中的元素的次序会反过来，因为移动到新的 bucket 位置的时候， HashMap 并不会将元素放在 LinkedList 的尾部，而是放在头部，这是为了避免尾部遍历 (tail traversing)。如果条件竞争发生了，那么就死循环了。

### 9 HashMap多线程并发问题分析?
* hashing 的概念
* HashMap 中解决碰撞的方法
* equals()和 hashCode()的应用，以及它们在 HashMap 中的重要性
* 不可变对象的好处
* HashMap 多线程的条件竞争
* 重新调整 HashMap 的大小

### 10 HashMap会扩容，会缩容吗？
超过 容量 * 加载因子  会自动扩容，变成2倍

### 11 hash计算所在的位置
 
当 length 总是 2 的 n 次方时，h& (length-1)运算等价于对 length 取模，也就是 h%length，但是&比%具有更高的效率。

### 12 HashMap 的 resize(rehash)

当 HashMap 中的元素越来越多的时候，hash 冲突的几率也就越来越高，因为数组的长
度是固定的。所以为了提高查询的效率，就要对 HashMap 的数组进行扩容，数组扩容这 个操作也会出现在 ArrayList 中，这是一个常用的操作，而在 HashMap 数组扩容之后，最 消耗性能的点就出现了:原数组中的数据必须重新计算其在新数组中的位置，并放进去， 这就是 resize。
那么 HashMap 什么时候进行扩容呢?当 HashMap 中的元素个数超过数组大小 *loadFactor 时，就会进行数组扩容，loadFactor 的默认值为 0.75，这是一个折中的取值。 也就是说，默认情况下，数组大小为 16，那么当 HashMap 中元素个数超过 16*0.75=12 的时候，就把数组的大小扩展为 2*16=32，即扩大一倍，然后重新计算每个元素在数组中 的位置，而这是一个非常消耗性能的操作，所以如果我们已经预知 HashMap 中元素的个 数，那么预设元素的个数能够有效的提高 HashMap 的性能。
loadFactor:负载因子 loadFactor 定义为:散列表的实际元素数目(n)/ 散列表的容量 (m)。
负载因子衡量的是一个散列表的空间的使用程度，负载因子越大表示散列表的装填程度 越高，反之愈小。

### 13 Fail-Fast 机制
我们知道 java.util.HashMap 不是线程安全的，因此如果在使用迭代器的过程中有其他线
程修改了 map，那么将抛出 ConcurrentModificationException，这就是所谓 fail-fast 策 略。
这一策略在源码中的实现是通过 modCount 域，modCount 顾名思义就是修改次数，对 HashMap 内容的修改都将增加这个值，那么在迭代器初始化过程中会将这个值赋给迭代器 的 expectedModCount。
Java 代码
 	HashIterator() {
		expectedModCount = modCount;
		if (size > 0) { // advance to first entry
			Entry[] t = table;
		 	while (index < t.length && (next = t[index++]) == null)
		 	;
		}
	}
	
 modCount 声明为 volatile，保证线程之间修改的可见性。
 		
		final Entry<K,V> nextEntry() {
			if (modCount != expectedModCount)
				throw new ConcurrentModificationException();
				
### 14 HashMap初始容量
HashMap(int initialCapacity):构建一个初始容量为 initialCapacity，负载因子为 0.75 的 HashMap 这句话表述也有问题，初始容量并非 initialCapacity，看源码:
 	// Find a power of 2 >= initialCapacity
	int capacity = 1;
	while (capacity < initialCapacity)
    	capacity <<= 1;也是就是初始容量是大于 initialCapacity 的最小的 2 的 n次幂。如initialCapacity=6，那么初始容量为 8,初始容量永远是 2 的n次方。


### 15 HashMap的相同的HashKey放入链头还是链尾？
### 16 堆排序？？？
### 17 HashMap 的工作原理

HashMap 基于 hashing 原理，我们通过 put()和 get()方法储存和获取对象。当我 们将键值对传递给 put()方法时，它调用键对象的 hashCode()方法来计算 hashcode， 让后找到 bucket 位置来储存值对象。当获取对象时，通过键对象的 equals()方法找到正 确的键值对，然后返回值对象。HashMap 使用 LinkedList 来解决碰撞问题，当发生碰撞 了，对象将会储存在 LinkedList 的下一个节点中。 HashMap 在每个 LinkedList 节点中储存键值对对象。
当两个不同的键对象的 hashcode 相同时会发生什么? 它们会储存在同一个 bucket
位置的 LinkedList 中。键对象的 equals()方法用来找到键值对。


### 18 HashTable 实现原理（Hashtable介绍）

和 HashMap 一样，Hashtable 也是一个散列表，它存储的内容是键值对 (key-value)映射。Hashtable 的函数都是同步的，这意味着它是线程安全的。它的 key、value 都不可以为 null。

默认加载因子是 0.75, 这是在时间和空间成本上寻求一种折衷。加载 因子过高虽然减少了空间开销，但同时也增加了查找某个条目的时间。

### 19 HashMap和HashTable的异同

* HashMap 和 Hashtable 实现原理基本一样，都是通过哈希表实现。而且两者处理 冲突的方式也一样，都是通过链表法
* Hashtable 的默认容量为 11，默认负载因子为 0.75.(HashMap 默认容量为 16，默认负载因子也是 0.75)
* Hashtable 的容量可以为任意整数，最小值为 1，而 HashMap 的容量始终为 2 的 n 次方。
* 为避免扩容带来的性能问题，建议指定合理容量。
* HashMap 和 Hashtable 存储的是键值对对象，而不是单独的键或值。
* HashMap 计算索引的方式是 h&(length-1),而 Hashtable 用的是模运算，效率上是 低于 HashMap 的。
* 特别需要注意的是这个方法包括下面要讲的若干方法都加了 synchronized 关键字，也 就意味着这个 Hashtable 是个线程安全的类，这也是它和 HashMap 最大的不同点.
* Hashtable 每次扩容，容量都为原来的 2 倍加 1，而 HashMap 为原来的 2 倍。

### 20 HashTable和HashMap区别

**继承的父类不同**

		Hashtable继承自Dictionary类，而HashMap继承自AbstractMap类。但二者都实现了Map接口。
**线程安全性不同**
		
		关于hashmap的一段描述如下：此实现不是同步的。如果多个线程同时访问一个哈希映射，而其中至少一个线程从结构上修改了该映射，则它必须保持外部同步。
		Hashtable 中的方法是Synchronize的，而HashMap中的方法在缺省情况下是非Synchronize的。在多线程并发的环境下，可以直接使用Hashtable，不需要自己为它的方法实现同步，但使用HashMap时就必须要自己增加同步处理。（结构上的修改是指添加或删除一个或多个映射关系的任何操作；仅改变与实例已经包含的键关联的值不是结构上的修改。）这一般通过对自然封装该映射的对象进行同步操作来完成。如果不存在这样的对象，则应该使用 Collections.synchronizedMap 方法来“包装”该映射。最好在创建时完成这一操作，以防止对映射进行意外的非同步访问，      
		Map m = Collections.synchronizedMap(new HashMap(...));

**是否提供contains方法**

		HashMap把Hashtable的contains方法去掉了，改成containsValue和containsKey，因为contains方法容易让人引起误解。
		Hashtable则保留了contains，containsValue和containsKey三个方法，其中contains和containsValue功能相同。
		
**key和value是否允许null值**
	
		Hashtable中，key和value都不允许出现null值。但是如果在Hashtable中有类似put(null,null)的操作，编译同样可以通过，因为key和value都是Object类型，但运行时会抛出NullPointerException异常，这是JDK的规范规定的。
		HashMap中，null可以作为键，这样的键只有一个；可以有一个或多个键所对应的值为null。当get()方法返回null值时，可能是 HashMap中没有该键，也可能使该键所对应的值为null。因此，在HashMap中不能由get()方法来判断HashMap中是否存在某个键， 而应该用containsKey()方法来判断。

**两个遍历方式的内部实现上不同**

		Hashtable、HashMap都使用了 Iterator。而由于历史原因，Hashtable还使用了Enumeration的方式 。

**hash值不同**
		
		哈希值的使用不同，HashTable直接使用对象的hashCode。而HashMap重新计算hash值。

      hashCode是jdk根据对象的地址或者字符串或者数字算出来的int类型的数值。
      Hashtable计算hash值，直接用key的hashCode()，而HashMap重新计算了key的hash值，Hashtable在求hash值对应的位置索引时，用取模运算，而HashMap在求位置索引时，则用与运算，且这里一般先用hash&0x7FFFFFFF后，再对length取模，&0x7FFFFFFF的目的是为了将负的hash值转化为正值，因为hash值有可能为负数，而&0x7FFFFFFF后，只有符号外改变，而后面的位都不变。
      
**内部实现使用的数组初始化和扩容方式不同**

		HashTable在不指定容量的情况下的默认容量为11，而HashMap为16，Hashtable不要求底层数组的容量一定要为2的整数次幂，而HashMap则要求一定为2的整数次幂。
      Hashtable扩容时，将容量变为原来的2倍加1，而HashMap扩容时，将容量变为原来的2倍。
      Hashtable和HashMap它们两个内部实现方式的数组的初始大小和扩容的方式。HashTable中hash数组默认大小是11，增加的方式是 old*2+1。


### 21 ArrayList、Vector、HashMap、HashTable、HashSet的默认初始容量、加载因子、扩容增量
**List 元素**是有序的、可重复

**ArrayList、Vector**默认初始容量为10

**Vector：**
		
		线程安全，但速度慢
		底层数据结构是数组结构
		加载因子为1：即当 元素个数 超过 容量长度 时，进行扩容
		扩容增量：原容量的 1倍
			如 Vector的容量为10，一次扩容后是容量为20

**ArrayList：**
			
		线程不安全，查询速度快
		底层数据结构是数组结构
		扩容增量：原容量的 1.5倍
			如 ArrayList的容量为10，一次扩容后是容量为15
　　　　　　
**Set(集)** 元素无序的、不可重复。

**HashSet：**
		
		线程不安全，存取速度快
		底层实现是一个HashMap（保存数据），实现Set接口
		默认初始容量为16（为何是16，见下方对HashMap的描述）
		加载因子为0.75：即当 元素个数 超过 容量长度的0.75倍 时，进行扩容
		扩容增量：原容量的 1 倍
			如 HashSet的容量为16，一次扩容后是容量为32

**HashMap：**
		
		默认初始容量为16,长度始终保持2的n次方
		为何是16：16是2^4，可以提高查询效率，另外，32=16<<1       -->至于详细的原因可另行分析，或分析源代码）
		加载因子为0.75：即当 元素个数 超过 容量长度的0.75倍 时，进行扩容
		扩容增量：原容量的 1 倍
			如 HashMap的容量为16，一次扩容后是容量为32

**HashTable：**
		
		默认初始容量为11
		线程安全，但是速度慢，不允许key/value为null
		加载因子为0.75：即当 元素个数 超过 容量长度的0.75倍 时，进行扩容
		扩容增量：2*原数组长度+1
			如 HashTable的容量为11，一次扩容后是容量为23

### 22 HashTable总结
* Hashtable 是个线程安全的类(HashMap 线程安全);
* Hasbtable 并不允许值和键为空(null)，若为空，会抛空指针(HashMap 可以); 
* Hashtable 不允许键重复，若键重复，则新插入的值会覆盖旧值(同 HashMap);* Hashtable 同样是通过链表法解决冲突;
* Hashtable 根据 hashcode 计算索引时将 hashcode 值先与上 0x7FFFFFFF,这是为了 保证 hash 值始终为正数;
* Hashtable 的容量为任意正数(最小为 1)，而 HashMap 的容量始终为 2 的 n 次 方。Hashtable 默认容量为 11，HashMap 默认容量为 16;
* Hashtable 每次扩容，新容量为旧容量的 2 倍加 1，而 HashMap 为旧容量的 2 倍;
* Hashtable 和 HashMap 默认负载因子都为 0.75;

### 23 HashMap 和 HashTable 区别
* HashTable 的方法前面都有 synchronized 来同步，是线程安全的; HashMap 未经同步，是非线程安全的。
* HashTable 不允许 null 值(key 和 value 都不可以) ;HashMap 允许 null 值(key 和 value 都可以)。
* HashTable 有一个 contains(Object value)功能和 containsValue(Object
value)功能一样。
* HashTable 使用 Enumeration 进行遍历;HashMap 使用 Iterator 进行遍 历。
* HashTable 中 hash 数组默认大小是 11，增加的方式是 old*2+1; HashMap 中 hash 数组的默认大小是 16，而且一定是 2 的指数。 6).哈希值的使用不同，HashTable 直接使用对象的 hashCode; HashMap 重新计算 hash 值，而且用与代替求模。

### 24 ArrayList 和 vector 区别
ArrayList 和 Vector 都实现了 List 接口，都是通过数组实现的。
Vector 是线程安全的，而 ArrayList 是非线程安全的。
List 第一次创建的时候，会有一个初始大小，随着不断向 List 中增加元素，当 List 认为容量不够的时候就会进行扩容。Vector 缺省情况下自动增长原来一倍 的数组长度，ArrayList 增长原来的 50%。

### 25 ArrayList 和 LinkedList 区别及使用场景
**区别**

ArrayList 底层是用数组实现的，可以认为 ArrayList 是一个可改变大小的数 组。随着越来越多的元素被添加到 ArrayList 中，其规模是动态增加的。 LinkedList 底层是通过双向链表实现的， LinkedList 和 ArrayList 相比，增删的 速度较快。但是查询和修改值的速度较慢。同时，LinkedList 还实现了 Queue 接口，所以他还提供了 offer(),
peek(), poll()等方法。

**使用场景**

LinkedList 更适合从中间插入或者删除(链表的特性)。 

ArrayList 更适合检索和在末尾插入或删除(数组的特性)。

### 26 Collection 和 Collections 的区别
java.util.Collection 是一个集合接口。它提供了对集合对象进行基本操作的通用 接口方法。Collection 接口在 Java 类库中有很多具体的实现。Collection 接口 的意义是为各种具体的集合提供了最大化的统一操作方式。 java.util.Collections 是一个包装类。它包含有各种有关集合操作的静态多态方 法。此类不能实例化，就像一个工具类，服务于 Java 的 Collection 框架。

### 27 Concurrenthashmap

通过分析 Hashtable 就知道，synchronized 是针对整张 Hash 表的，即每 次锁住整张表让线程独占，ConcurrentHashMap 允许多个修改操作并发进 行，其关键在于使用了锁分离技术。它使用了多个锁来控制对 hash 表的不同 部分进行的修改。ConcurrentHashMap 内部使用段(Segment)来表示这些不同 的部分，每个段其实就是一个小的 hash table，它们有自己的锁。只要多个修改操作发生在不同的段上，它们就可以并发进行。

有些方法需要跨段，比如 size()和 containsValue()，它们可能需要锁定整个表 而而不仅仅是某个段，这需要按顺序锁定所有段，操作完毕后，又按顺序释放 所有段的锁。

### 28 java对象的浅克隆和深克隆
在Object基类中，有一个方法叫clone，产生一个前期对象的克隆，克隆对象是原对象的拷贝，由于引用类型的存在，有深克隆和浅克隆之分，若克隆对象中存在引用类型的属性，深克隆会将此属性完全拷贝一份，而浅克隆仅仅是拷贝一份此属性的引用。首先看一下容易犯的几个小问题

**浅克隆** 就是引用类型的属性无法完全复制，类User中包含成绩属性Mark，Mark是由Chinese和math等等组成的，浅克隆失败的例子

**深克隆**

* **方式一：clone函数的嵌套调用** 

既然引用类型无法被完全克隆，那将引用类型也实现Cloneable接口重写clone方法，在User类中的clone方法调用属性的克隆方法，也就是方法的嵌套调用

* **方式二：序列化**

上一种方法已经足够满足我们的需要，但是如果类之间的关系很多，或者是有的属性是数组呢，数组可无法实现Cloneable接口（我们可以在clone方法中手动复制数组），但是每次都得手写clone方法，很麻烦，而序列化方式只需要给每个类都实现一个Serializable接口，也是标记接口，最后同序列化和反序列化操作达到克隆的目的（包括数组的复制）。

	import java.io.*;
	import java.util.Arrays;

	public class User implements Serializable{
    private String name;
    private int age;
    private int[] arr;

    public User(String name, int age, int[] arr) {
        this.name = name;
        this.age = age;
        this.arr = arr;
    }
    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", arr=" + Arrays.toString(arr) +
                '}';
    }
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        int[] arr = {1,2,3,4,5,6};
        User user = new User("user",22,arr);

        ByteArrayOutputStream bo = new ByteArrayOutputStream();
        ObjectOutputStream oo = new ObjectOutputStream(bo);
        oo.writeObject(user);//序列化
        ByteArrayInputStream bi = new ByteArrayInputStream(bo.toByteArray());
        ObjectInputStream oi = new ObjectInputStream(bi);
        User userClone = (User) oi.readObject();//反序列化

        System.out.println("原user："+user);
        System.out.println("克隆的user："+userClone);
        user.arr[1] = 9;
        System.out.println("修改后的原user："+user);
        System.out.println("修改后的克隆user："+userClone);
    }
	}

### 29 Error、Exception 区别

Error 类和 Exception 类的父类都是 throwable 类，他们的区别是:
Error 类一般是指与虚拟机相关的问题，如系统崩溃，虚拟机错误，内存空间不 足，方法调用栈溢等。对于这类错误的导致的应用程序中断，仅靠程序本身无 法恢复和和预防，遇到这样的错误，建议让程序终止。
Exception 类表示程序可以处理的异常，可以捕获且可能恢复。遇到这类异 常，应该尽可能处理异常，使程序恢复运行，而不应该随意终止异常。

### 30 Java 中如何实现代理机制

JDK 动态代理:代理类和目标类实现了共同的接口，用到 InvocationHandler 接口。

CGLIB 动态代理:代理类是目标类的子类，用到 MethodInterceptor 接口。

### 31 多线程的实现方式

继承 Thread 类、实现 Runnable 接口、使用 ExecutorService、Callable、Future 实现有返回结果的多线程。

### 32 如何停止一个线程

* 使用退出标志，使线程正常退出，也就是当 run 方法完成后线程终止。
* 使用 stop 方法强行终止，但是不推荐这个方法，因为 stop 和 suspend 及 resume 一样都是过期作废的方法。
* 使用 interrupt 方法中断线程。

### 33 synchronized 如何使用

synchronized 是 Java 中的关键字，是一种同步锁。它修饰的对象有以下几 种:

* 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括 号{}括起来的代码，作用的对象是调用这个代码块的对象;* 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法， 作用的对象是调用这个方法的对象;
* 修饰一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个 类的所有对象;
* 修饰一个类，其作用的范围是 synchronized 后面括号括起来的部分，作用 主的对象是这个类的所有对象。

### 34 synchronized 和 Lock 的区别

**主要相同点:** Lock 能完成 synchronized 所实现的所有功能 

**主要不同点:** Lock 有比 synchronized 更精确的线程语义和更好的性能。Lock 的锁定是通过代码实现的，而 synchronized 是在 JVM 层面上实现的， synchronized 会自动释放锁，而 Lock 一定要求程序员手工释放，并且必须在 finally 从句中释放。Lock 还有更强大的功能，例如，它的 tryLock 方法可以非 阻塞方式去拿锁。Lock 锁的范围有局限性，块范围，而 synchronized 可以锁 住块、对象、类。

### 35 多线程如何进行信息交互

void notify() 唤醒在此对象监视器上等待的单个线程。
void notifyAll() 唤醒在此对象监视器上等待的所有线程。

void wait() 导致当前的线程等待，直到其他线程调用此对象的 notify()方法或 notifyAll()方法。

void wait(long timeout) 导致当前的线程等待，直到其他线程调用此对象的 notify()方法或 notifyAll()方法，或者超过指定的时间量。

void wait(long timeout, int nanos) 导致当前的线程等待，直到其他线程调用此 对象的 notify()方法或 notifyAll()方法，或者其他某个线程中断当前线程，或者 已超过某个实际时间量。

### 36 sleep 和 wait 的区别

sleep()方法是 Thread 类中方法，而 wait()方法是 Object 类中的方法。 sleep()方法导致了程序暂停执行指定的时间，让出 cpu 该其他线程，但是他的 监控状态依然保持者，当指定的时间到了又会自动恢复运行状态，在调用 sleep()方法的过程中，线程不会释放对象锁。而当调用 wait()方法的时候，线 程会放弃对象锁，进入等待此对象的等待锁定池，只有针对此对象调用 notify()方法后本线程才进入对象锁定池准备。

### 37 多线程与死锁

死锁是指两个或两个以上的进程在执行过程中，因争夺资源而造成的一种互相 等待的现象，若无外力作用，它们都将无法推进下去。
产生死锁的原因:

* 因为系统资源不足。
* 进程运行推进的顺序不合适。 
* 资源分配不当。

### 38 如何才能产生死锁

产生死锁的四个必要条件: 

* 互斥条件:所谓互斥就是进程在某一时间内独占资源。 
* 请求与保持条件:一个进程因请求资源而阻塞时，对已获得的资源保持不放。
* 不剥夺条件:进程已获得资源，在末使用完之前，不能强行剥夺。 
* 循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。

### 39 死锁的预防

* 打破互斥条件
* 打破不可抢占条件。
* 打破占有且申请条件。
* 打破循环等待条件，实行资源有序分配策略。

### 40 Java 线程池技术及原理

线程可以复用，就是执行完一个任务，并不被销毁，而是可以继续执行其他的任务
workQueue: 一个阻塞队列，用来存储等待执行的任务，这个参数的选择也很重要，会对 线程池的运行过程产生重大影响，一般来说，这里的阻塞队列有以下几种选择:

* ArrayBlockingQueue;
* LinkedBlockingQueue;
* SynchronousQueue;
ArrayBlockingQueue 和 PriorityBlockingQueue 使用较少，一般使用 LinkedBlockingQueue 和 Synchronous。线程池的排队策略与BlockingQueue 有关。

**拒绝策略**

* ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出 RejectedExecutionException 异常。 
* ThreadPoolExecutor.DiscardPolicy:也是丢弃任务，但是不抛出异常。
* hreadPoolExecutor.DiscardOldestPolicy:丢弃队列最前面的任务，然后重新尝试执行 任务(重复此过程)
* ThreadPoolExecutor.CallerRunsPolicy:由调用线程处理该任务

### 41 任务提交给线程池之后的处理策略

* 如果当前线程池中的线程数目小于 corePoolSize，则每来一个任务，就会创建一个线程去 执行这个任务;
* 如果当前线程池中的线程数目>=corePoolSize，则每来一个任务，会尝试将其添加到任 务缓存队列当中，若添加成功，则该任务会等待空闲线程将其取出去执行;若添加失败 (一般来说是任务缓存队列已满)，则会尝试创建新的线程去执行这个任务;
* 如果当前线程池中的线程数目达到 maximumPoolSize，则会采取任务拒绝策略进行处 理;
* 如果线程池中的线程数量大于 corePoolSize 时，如果某线程空闲时间超过 keepAliveTime，线程将被终止，直至线程池中的线程数目不大于 corePoolSize;如果允许为核心池中的线程设置存活时间，那么核心池中的线程空闲时间超过 keepAliveTime， 线程也会被终止。

### 42 线程池中的线程初始化

默认情况下，创建线程池之后，线程池中是没有线程的，需要提交任务之后才会创建线 程。
在实际中如果需要线程池创建之后立即创建线程，可以通过以下两个方法办到:

* prestartCoreThread():初始化一个核心线程;
* prestartAllCoreThreads():初始化所有核心线程

### 43 任务缓存队列及排队策略
workQueue 的类型为 BlockingQueue<Runnable>，通常可以取下面三种类型:
 
* ArrayBlockingQueue:基于数组的先进先出队列，此队列创建时必须指定大小;
* LinkedBlockingQueue:基于链表的先进先出队列，如果创建时没有指定此队列大 小，则默认为 Integer.MAX_VALUE;
* synchronousQueue:这个队列比较特殊，它不会保存提交的任务，而是将直接新建 一个线程来执行新来的任务。

### 44 线程池容量的动态调整

ThreadPoolExecutor 提供了动态调整线程池容量大小的方法: setCorePoolSize() 和 setMaximumPoolSize()

### 45 静态方法来创建线程池

* Executors.newCachedThreadPool(); 大小为 Integer.MAX_VALUE   //创建一个缓冲池，缓冲池容大小为 Integer.MAX_VALUE
* Executors.newSingleThreadExecutor();   //创建容量为 1 的缓冲池
* Executors.newFixedThreadPool(int);   //创建固定容量大小的缓冲池

### 46 java 并发包 concurrent 及常用的类
### 46.1 并发容器 ###
**阻塞队列**

* BlockingQueue.class，阻塞队列接口
* BlockingDeque.class，双端阻塞队列接口
* ArrayBlockingQueue.class，阻塞队列，数组实现
* LinkedBlockingDeque.class，阻塞双端队列，链表实现
* LinkedBlockingQueue.class，阻塞队列，链表实现
* DelayQueue.class，阻塞队列，并且元素是Delay的子类，保证元素在达到一定时
间后才可以取得到* PriorityBlockingQueue.class，优先级阻塞队列
* SynchronousQueue.class，同步队列，但是队列长度为 0，生产者放入队列的操作
会被阻塞，直到消费者过来取，所以这个队列根本不需要空间存放元素;有点像一个独木桥，一次只能一人通过，还不能在桥上停留

**非阻塞队列**

* ConcurrentLinkedDeque.class，非阻塞双端队列，链表实现 
* ConcurrentLinkedQueue.class，非阻塞队列，链表实现

**转移队列**

* TransferQueue.class，转移队列接口，生产者要等消费者消费的队列，生产者尝试
把元素直接转移给消费者
* LinkedTransferQueue.class，转移队列的链表实现，它比 SynchronousQueue 更快

**其它容器**

* ConcurrentMap.class，并发Map的接口，定义了putIfAbsent(k,v)、remove(k,v)、
replace(k,oldV,newV)、replace(k,v)这四个并发场景下特定的方法
* ConcurrentHashMap.class，并发HashMap
* ConcurrentNavigableMap.class，NavigableMap 的实现类，返回最接近的一个元素
* ConcurrentSkipListMap.class，它也是 NavigableMap 的实现类(要求元素之间可以
比较)，同时它比 ConcurrentHashMap 更加 scalable——ConcurrentHashMap 并不保证它的操作时间，并且你可以自己来调整它的 load factor;但是 ConcurrentSkipListMap 可以保证 O(log n)的性能，同时不能自己来调整它的并发 参数，只有你确实需要快速的遍历操作，并且可以承受额外的插入开销的时候，才 去使用它
* ConcurrentSkipListSet.class，和上面类似，只不过map变成了set
* CopyOnWriteArrayList.class，copy-on-write模式的arraylist，每当需要插入元
素，不在原 list 上操作，而是会新建立一个 list，适合读远远大于写并且写时间并
苛刻的场景
* CopyOnWriteArraySet.class，和上面类似，list变成set而已

### 46.2 同步容器 ###

* CountDownLatch.class，一个线程调用 await 方法以后，会阻塞地等待计数器被调 用 countDown 直到变成 0，功能上和下面的 CyclicBarrier 有点像
* CyclicBarrier.class，也是计数等待，只不过它是利用await方法本身来实现计数器 “+1”的操作，一旦计数器上显示的数字达到 Barrier 可以打破的界限，就会抛出 BrokenBarrierException，线程就可以继续往下执行
* Semaphore.class，功能上很简单，acquire()和release()两个方法，一个尝试获取许 可，一个释放许可，Semaphore 构造方法提供了传入一个表示该信号量所具备的许 可数量。
* Exchanger.class，这个类的实例就像是两列飞驰的火车(线程)之间开了一个神奇 的小窗口，通过小窗口(exchange 方法)可以让两列火车安全地交换数据。
* Phaser.class，功能上和第1、2个差不多，但是可以重用，且更加灵活，稍微有点 复杂(CountDownLatch 是不断-1，CyclicBarrier 是不断+1，而 Phaser 定义了两个 概念，phase 和 party)


### 46.3 原子对象 ###

* AtomicBoolean.class
* AtomicInteger.class
* AtomicIntegerArray.class
* AtomicIntegerFieldUpdater.class
* AtomicLong.class
* AtomicLongArray.class
* AtomicLongFieldUpdater.class
* AtomicMarkableReference.class，它是用来高效表述Object-boolean这样的对象标志位数据结构的，一个对象引用+一个 bit 标志位
* AtomicReference.class
* AtomicReferenceArray.class
* AtomicReferenceFieldUpdater.class
* AtomicStampedReference.class，它和前面的AtomicMarkableReference类似，但是它是用来高效表述 Object-int 这样的“对象+版本号”数据结构，特别用于解决 ABA 问题

### 46.4 锁 ###

* AbstractOwnableSynchronizer.class，这三个 AbstractXXXSynchronizer 都是为了创 建锁和相关的同步器而提供的基础，锁，还有前面提到的同步设备都借用了它们的 实现逻辑
* AbstractQueuedLongSynchronizer.class，AbstractOwnableSynchronizer 的子类，所 有的同步状态都是用 long 变量来维护的，而不是 int，在需要 64 位的属性来表示 状态的时候会很有用
* AbstractQueuedSynchronizer.class，为实现依赖于先进先出队列的阻塞锁和相关同 步器(信号量、事件等等)提供的一个框架，它依靠 int 值来表示状态
* Lock.class，Lock比synchronized关键字更灵活，而且在吞吐量大的时候效率更 高，根据 JSR-133 的定义，它 happens-before 的语义和 synchronized 关键字效果是一模一样的，它唯一的缺点似乎是缺乏了从 lock 到 finally 块中 unlock 这样容易 遗漏的固定使用搭配的约束，除了 lock 和 unlock 方法以外，还有这样两个值得注 意的方法:* ReadWriteLock.class，读写锁，读写分开，读锁是共享锁，写锁是独占锁;对于读 -写都要保证严格的实时性和同步性的情况，并且读频率远远大过写，使用读写锁 会比普通互斥锁有更好的性能。
* ReentrantLock.class，可重入锁(lock行为可以嵌套，但是需要和unlock行为一一 对应)。
* Condition.class，使用锁的 newCondition 方法可以返回一个该锁的 Condition 对
象，如果说锁对象是取代和增强了 synchronized 关键字的功能的话，那么 Condition 则是对象 wait/notify/notifyAll 方法的替代。在下面这个例子中，lock 生 成了两个 condition，一个表示不满，一个表示不空

### 46.5 Fork-join 框架 ###

### 46.6 执行器和线程池 ###

### 47 Lock 和 synchronized 有以下几点不同 ###

* Lock 是一个接口，而 synchronized 是 Java 中的关键字，synchronized 是内 置的语言实现;
* synchronized 在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象 发生;而 Lock 在发生异常时，如果没有主动通过 unLock()去释放锁，则很可能造成死锁 现象，因此使用 Lock 时需要在 finally 块中释放锁;* Lock 可以让等待锁的线程响应中断，而 synchronized 却不行，使用 synchronized 时，等待的线程会一直等待下去，不能够响应中断;
* 通过 Lock 可以知道有没有成功获取锁，而 synchronized 却无法办到。
* Lock 可以提高多个线程进行读操作的效率。
* 在性能上来说，如果竞争资源不激烈，两者的性能是差不多的，而当竞争资源非常激 烈时(即有大量线程同时竞争)，此时 Lock 的性能要远远优于 synchronized

### 48 锁相关概念 ###

**可重入锁**

如果锁具备可重入性，则称作为可重入锁。像 synchronized 和 ReentrantLock 都 是可重入锁，可重入性在我看来实际上表明了锁的分配机制:基于线程的分配，而不是基 于方法调用的分配。

举个简单的例子，当一个线程执行到某个 synchronized 方法时，比 如说 method1，而在 method1 中会调用另外一个 synchronized 方法 method2，此时 线程不必重新去申请锁，而是可以直接执行方法 method2。

synchronized 和 Lock 都具备可重入性

**可中断锁**

可中断锁，就是可以相应中断的锁。

在 Java 中，synchronized 就不是可中断锁，而 Lock 是可中断锁。

**公平锁**

公平锁即尽量以请求锁的顺序来获取锁。比如同是有多个线程在等待一个锁，当这个 锁被释放时，等待时间最久的线程(最先请求的线程)会获得该所，这种就是公平锁。

非公平锁即无法保证锁的获取是按照请求锁的顺序进行的。这样就可能导致某个或者一些线程永远获取不到锁。

在Java中,synchronized 就是非公平锁，它无法保证等待的线程获取锁的顺序。而对于 ReentrantLock 和 ReentrantReadWriteLock，它默认情况下是非公平锁， 但是可以设置为公平锁.

**读写锁**

读写锁将对一个资源(比如文件)的访问分成了 2 个锁，一个读锁和一个写锁。正因为有了读写锁，才使得多个线程之间的读操作不会发生冲突。

ReadWriteLock 就是读写锁，它是一个接口，ReentrantReadWriteLock 实现了这 个接口。

可以通过 readLock()获取读锁，通过 writeLock()获取写锁。

### 49 线程安全的集合 ###

### 49.1 ConcurrentHashMap ###

ConcurrentHashMap 是线程安全的 HashMap 的实现。

### 49.2 CopyOnWriteArrayList ###

CopyOnWriteArrayList 是一个线程安全、并且在读操作时无锁的 ArrayList。
CopyOnWriteArrayList 基于 ReentrantLock 保证了增加元素和删除元素动作的 互斥。在读操作上没有任何锁，这样就保证了读的性能，带来的副作用是有时候可能会读取到 脏数据。

### 49.3 CopyOnWriteArraySet ###

CopyOnWriteArraySet 是基于 CopyOnWriteArrayList 的，可以知道 set 是不容许重复数据的,因此 add 操作和 CopyOnWriteArrayList 有所区别，他是调用 CopyOnWriteArrayList 的 addIfAbsent 方法。

### 49.4 ArrayBlockingQueue ###

ArrayBlockingQueue 是一个基于数组、先进先出、线程安全的集合类，其特点是实现指定时间的阻塞读写，并且容量是可以限制的。

### 50 volatile 关键字 ###

volatile 变量可以被看作是一种“程度较轻的 synchronized”;与synchronized 块相比，volatile 变量所需的编码较少，并且运行时开销也较 少，但是它所能实现的功能也仅是 synchronized 的一部分。

锁提供了两种主要特性 :互斥(mutual exclusion)和可见性(visibility)。互斥即一次只允许一个线程持有某个特定的锁，因此可使用该特性实现对共享数据的协调访问协议，这样，一次就只有 一个线程能够使用该共享数据。可见性必须确保释放锁之前对共享数据做出的 更改对于随后获得该锁的另一个线程是可见的，如果没有同步机制提供的这种 可见性保证，线程看到的共享变量可能是修改前的值或不一致的值，这将引发 许多严重问题。

Volatile 变量具有 synchronized 的可见性特性，但是不具备原 子特性。这就是说线程能够自动发现 volatile 变量的最新值。

要使 volatile 变量提供理想的线程安全，必须同时满足下面两个条件:对变量的写操作不依赖于当前值;该变量没有包含在具有其他变量的不变式中。

**只能保证可见性，不能保证原子性**

每一个线程运行时都有一个线程栈，线程栈保存了线程运行时候变量值信息。 当线程访问某一个对象时候值的时候，首先通过对象的引用找到对应在堆内存 的变量的值，然后把堆内存变量的具体值 load 到线程本地内存中，建立一个变 量副本，之后线程就不再和对象在堆内存变量值有任何关系，而是直接修改副 本变量的值，在修改完之后的某一个时刻(线程退出之前)，自动把线程变量 副本的值回写到对象在堆中变量。这样在堆中的对象的值就产生变化了。

### 51 Java 中的 NIO，BIO，AIO 分别是什么 ###

**BIO**

同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求 时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成 不必要的线程开销，当然可以通过线程池机制改善。BIO 方式适用于连接数目 比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用 中，JDK1.4 以前的唯一选择，但程序直观简单易理解。

**NIO**

同步非阻塞，服务器实现模式为一个请求一个线程，即客户端发送的连接 请求都会注册到多路复用器上，多路复用器轮询到连接有 I/O 请求时才启动一 个线程进行处理。NIO 方式适用于连接数目多且连接比较短(轻操作)的架 构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4 开始支持。

**AIO**

异步非阻塞，服务器实现模式为一个有效请求一个线程，客户端的 I/O 请 求都是由 OS 先完成了再通知服务器应用去启动线程进行处理.AIO 方式使用于 连接数目多且连接比较长(重操作)的架构，比如相册服务器，充分调用 OS 参与并发操作，编程比较复杂，JDK7 开始支持。

### 52 IO 和 NIO 区别 ###

* IO 是面向流的，NIO 是面向缓冲区的。
* IO 的各种流是阻塞的，NIO 是非阻塞模式。
* Java NIO 的选择器允许一个单独的线程来监视多个输入通道，你可以注册 多个通道使用一个选择器，然后使用一个单独的线程来“选择”通道:这些通道 里已经有可以处理的输入，或者选择已准备写入的通道。这种选择机制，使得 一个单独的线程很容易来管理多个通道。

### 53 序列化与反序列化 ###

把对象转换为字节序列的过程称为对象的序列化。 

把字节序列恢复为对象的过程称为对象的反序列化。

对象的序列化主要有两种用途: 

* 把对象的字节序列永久地保存到硬盘上，通常存放在一个文件中; 
* 在网络上传送对象的字节序列。 

当两个进程在进行远程通信时，彼此可以发送各种类型的数据。无论是何种类 型的数据，都会以二进制序列的形式在网络上传送。发送方需要把这个 Java 对象转换为字节序列，才能在网络上传送;接收方则需要把字节序列再恢复为 Java 对象。

### 54 常见的序列化协议有哪些 ###

Protobuf, Thrift, Hessian, Kryo

### 55 内存溢出和内存泄漏的区别 ###

内存溢出和内存泄漏 用什么工具诊断？jmap和jstat如何使用？

内存溢出是指程序在申请内存时，没有足够的内存空间供其使用，出现 out of memory。

内存泄漏是指分配出去的内存不再使用，但是无法回收。

### 56 Java虚拟机栈 ###

在 VM Spec 中对这个区域规定了 2 中异常状况:如果线程请求的栈深度大于虚拟机所 允许的深度，将抛出 StackOverflowError 异常;如果 VM 栈可以动态扩展(VM Spec 中允许固定长度的 VM 栈)，当扩展时无法申请到足够内存则抛出 OutOfMemoryError 异常。

### 57 方法区(Method Area) ###

“方法区”，叫永久代(Permanent Generation)，叫做 Non-Heap(非堆)。


### 58 kafka数据重复消费和数据丢失如何处理？ ###

### 59 Java 堆 ###

Java 堆内还有更细致的划分:新生代、老年代，再细致一点的:eden、from survivor、 to survivor，甚至更细粒度的本地线程分配缓冲(TLAB)

### 60 JVM常用调优参数 ###

* -Xmx：最大JVM可用内存， 例：-Xmx4g
* -Xms：最小JVM可用内存， 例：Xms4g
* -Xmn：新生代内存大小，例：-Xmn2560m
* -XX:PermSize：永久代内存大小，该值太大会导致fullGC时间过长，太小将增加fullGC频率，例：-XX:PermSize=128m
* -Xss：线程栈大小，太大将导致JVM可建的线程数量减少，栈容 量只由-Xss 参数设定。例：-Xss256k
* -XX:+DisableExplicitGC：禁止手动fullGC，如果配置，则System.gc()将无效，比如在为DirectByteBuffer分配空间过程中发现直接内存不足时会显式调用System.gc()
* -XX:+UseConcMarkSweepGC：一般PermGen是不会被GC，如果希望PermGen永久代也能被GC，则需要配置该参数
* -XX:+CMSParallelRemarkEnabled：GC进行时标记可回收对象时可以并行remark-XX:+UseCMSCompactAtFullCollection 表示在fullGC之后进行压缩，CMS默认不压缩空间
* -XX:LargePageSizeInBytes：为java堆内存设置内存页大小，例：-XX:LargePageSizeInBytes=128m
* -XX:+UseFastAccessorMethods：对原始类型进行快速优化
* -XX:+UseCMSInitiatingOccupancyOnly：关闭预期开始的晋升率的统计
* -XX:CMSInitiatingOccupancyFraction：使用cms作为垃圾回收，并设置GC百分比，例：-XX:CMSInitiatingOccupancyFraction=70（使用70％后开始CMS收集）
* -XX:+PrintGCDetails：打印GC的详细信息
* -XX:+PrintGCDateStamps：打印GC的时间戳
* -Xloggc：指定GC文件路径

### 61 出现 OOM 如何解决 ###

* 可通过命令定期抓取 heap dump 或者启动参数 OOM 时自动抓取 heap dump 文件。
* 通过对比多个 heap dump，以及 heap dump 的内容，分析代码找出内存占用最多的地方。
* 分析占用的内存对象，是否是因为错误导致的内存未及时释放，或者数据 过多导致的内存溢出。

### 62 用什么工具可以查出内存泄漏 ###

**Memory**

Analyzer-是一款开源的 JAVA 内存分析软件，查找内存泄漏，能容易找到大 块内存并验证谁在一直占用它，它是基于 Eclipse
RCP(Rich Client Platform)，可以下载 RCP 的独立版本或者 Eclipse 的插件。 

**JProfiler**

**JRockit**

**YourKit**

### 63 HotSpot 虚拟机使用了两种技术来加快内存分配 ###

在 Eden 区，HotSpot 虚拟机使用了两种技术来加快内存分配。分别是 bump-the- pointer 和 TLAB(Thread- Local Allocation Buffers)，这两种技术的做法分别是:由于 Eden 区是连续的，因此 bump-the-pointer 技术的核心就是跟踪最后创建的一个对象，在 对 象创建时，只需要检查最后一个对象后面是否有足够的内存即可，从而大大加快内存分 配速度;而对于 TLAB 技术是对于多线程而言的，将 Eden 区分为若干 段，每个线程使用 独立的一段，避免相互影响。TLAB 结合 bump-the-pointer 技术，将保证每个线程都使用 Eden 区的一段，并快速的分配内 存。

### 64 年老代 ###

对象如果在年轻代存活了足够长的时间而没有被清理掉 (即在几次 Young GC 后存活了下来)，则会被复制到年老代，年老代的空间一般比年轻 代大，能存放更多的对象，在年老代上发生的 GC 次数也比年轻代少。当年老代内存不足 时， 将执行 Major GC，也叫 Full GC。

如果对象比较大(比如长字符串或大数组)，Young 空间不足，则大对象会直接分配 到老年代上(大对象可能触发提前 GC，应少用，更应避免使用短命的大对象)

可能存在年老代对象引用新生代对象的情况，如果需要执行 Young GC，则可能需要 查询整个老年代以确定是否可以清理回收，这显然是低效的。解决的方法是，年老代中维 护一个 512 byte 的块——”card table“，所有老年代对象引用新生代对象的记录都记录在这 里。Young GC 时，只要查这里即可，不用再去查全部老年代，因此性能大大提高。	

### 65 Java GC 机制-新生代 ###

所以，Eden 区与 Survivor 的比例较大，HotSpot 默认是 8:1，即分别占新生代的 80%，10%，10%。如果 一次回收中，Survivor+Eden 中存活下来的内存超过了 10%，则需要将一部分对象分配到 老年代。用-XX:SurvivorRatio 参数来配置 Eden 区域 Survivor 区的容量比值，默认是 8， 代表 Eden:Survivor1:Survivor2=8:1:1.

### 66 Java GC 机制-老年代 ###

老年代用的算法是标记-整理算法，即:标记 出仍然存活的对象(存在引用的)，将所有存活的对象向一端移动，以保证内存的连续。

在发生 Minor GC 时，虚拟机会检查每次晋升进入老年代的大小是否大于老年代的剩 余空间大小，如果大于，则直接触发一次 Full GC

### 67 Java GC 机制-方法区(永久代) ###

永久代的回收有两种:常量池中的常量，无用的类信息，常量的回收很简单，没有引 用了就可以被回收。对于无用的类进行回收，必须保证 3 点:

* 类的所有实例都已经被回收
* 加载类的ClassLoader已经被回收
* 类对象的Class对象没有被引用(即没有通过反射引用该类的地方)

### 68 Java GC 机制-CMS ###

CMS(Concurrent Mark Sweep)收集器:老年代收集器，致力于获取最短回收停顿 时间，使用标记清除算法，多线程，优点是并发收集(用户线程可以和 GC 线程同时 工作)，停顿小。

CMS 收集的方法是:先 3 次标记，再 1 次清除，3 次标记中前两次是初始标记和重新标记 (此时仍然需要停止(stop the world))， 初始标记(Initial Remark)是标记 GC Roots 能关联到的对象(即有引用的对象)，停顿时间很短;并发标记(Concurrent remark)是 执行 GC Roots 查找引用的过程，不需要用户线程停顿;重新标记(Remark)是在初始标 记和并发标记期间，有标记变动的那部分仍需要标记，所以加上这一部分 标记的过程，停 顿时间比并发标记小得多，但比初始标记稍长。在完成标记之后，就开始并发清除，不需 要用户线程停顿。

### 69 并发(Concurrent)和并行(Parallel)的区别 ###

并发是指用户线程与 GC 线程同时执行(不一定是并行，可能交替，但总体上是在同 时执行的)，不需要停顿用户线程(其实在 CMS 中用户线程还是需要停顿的，只是非常 短，GC 线程在另一个 CPU 上执行);

并行收集是指多个 GC 线程并行工作，但此时用户线程是暂停的; 所以，Serial 和 Parallel 收集器都是并行的，而 CMS 收集器是并发的.

### 70 JVM 三种预定义类型类加载器 ###

Java 缺 省开始使用如下三种类型类装入器:

**启动(Bootstrap)类加载器:**

引导类装入器是用本地代码实现的类装入器，它负责 将 <Java_Runtime_Home>/lib 下面的核心类库或-Xbootclasspath 选项指定的 jar 包加载 到内存中。由于引导类加载器涉及到虚拟机本地实现细节，开发者无法直接获取到启动类 加载器的引用，所以不允许直接通过引用进行操作。

**扩展(Extension)类加载器:**

扩展类加载器是由 Sun 的 ExtClassLoader 
(sun.misc.Launcher$ExtClassLoader)实现的。它负责将<
Java_Runtime_Home >/lib/ext 或者由系统变量-Djava.ext.dir 指定位置中的类库加载到内 存中。开发者可以直接使用标准扩展类加载器。

**系统(System)类加载器:**

系统类加载器是由 Sun 的 AppClassLoader (sun.misc.Launcher$AppClassLoader)实现的。它负责将系统类路径 java -classpath 或 -Djava.class.path 变量所指的目录下的类库加载到内存中。开发者可以直接使用系统类加 载器。


### 71 类加载双亲委派机制介绍和分析 ###

就是某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父类加载 器，依次递归，如果父类加载器可以完成类加载任务，就成功返回;只有父类加载器无法 完成此加载任务时，才自己去加载。

	1. public class LoaderTest {	2.	3. 	public static void main(String[] args) {	4. 		try {	5. 			System.out.println(ClassLoader.getSystemClassLoader());	6.
				System.out.println(ClassLoader.getSystemClassLoader().getParent(	));	7. 			System.out.println(ClassLoader.getSystemClassLoader().getParent( ).getParent());	8. 		} catch (Exception e) {	9.			 e.printStackTrace();	10. 	}	11. 	}	12. }

	代码输出如下: 	1. sun.misc.Launcher$AppClassLoader@6d06d69c
	2. sun.misc.Launcher$ExtClassLoader@70dea4e	3. null

### 72 java 程序动态扩展方式 ###

* 调用 java.lang.Class.forName(...)加载类
 
		public static Class<?> forName(String name, boolean initialize, ClassLoader loader) throws ClassNotFoundException
		
这里的 initialize 参数是很重要的。它表示在加载同时是否完成初始化的工作(说明:单参数版本的 forName 方法默认是完成初始化的)。有些场景下需要将 initialize 设置为 true来强制加载同时完成初始化。例如典型的就是利用 DriverManager 进行 JDBC 驱动程序类注册的问题。因为每一个 JDBC 驱动程序类的静态初始化方法都用 DriverManager 注册驱动程序，这样才能被应用程序使用。这就要求驱动程序类必须被初始化，而不单单被加载。Class.forName 的一个很常见的用法就是在加载数据库驱动的时候。如Class.forName("org.apache.derby.jdbc.EmbeddedDriver").newInstance() 用 来 加 载Apache Derby 数据库的驱动。

* 用户自定义类加载器

1. 首先检查请求的类型是否已经被这个类装载器装载到命名空间中了，如果已经装 载，直接返回;否则转入步骤 2;
2. 委派类加载请求给父类加载器(更准确的说应该是双亲类加载器，真实虚拟机中 各种类加载器最终会呈现树状结构)，如果父类加载器能够完成，则返回父类加载器加载 的 Class 实例;否则转入步骤 3;3. 调用本类加载器的 findClass(...)方法，试图获取对应的字节码，如果获取的 到，则调用 defineClass(...)导入类型到方法区;如果获取不到对应的字节码或者其他原 因失败，返回异常给 loadClass(...)， loadClass(...)转而抛异常，终止加载过程(注 意:这里的异常种类不止一种)。

### 73 类加载器常见问题分析 ###

### 73.1 由不同的类加载器加载的指定类还是相同的类型吗？###

在 Java 中，一个类用其完全匹配类名(fully qualified class name)作为标识，这里指的 完全匹配类名包括包名和类名。但在 JVM 中一个类用其全名和一个加载类 ClassLoader 的实例作为唯一标识，不同类加载器加载的类将被置于不同的命名空间。

### 73.2 在代码中直接调用Class.forName(Stringname)方法，到底会触发那个类加载器进行类加载行为? ###

Class.forName(String name)默认会使用调用类的类加载器来进行类加载。

### 73.3 在编写自定义类加载器时，如果没有设定父加载器，那么父加载器是谁? ###

在不指定父类加载器的情况下，默认采用系统类加载器

即使用户自定义类加载器不指定父类加载器，那么，同样可以加载如下三个地方的类:1.  <Java_Runtime_Home>/lib 下的类;2. < Java_Runtime_Home >/lib/ext 下或者由系统变量 java.ext.dir 指定位置中的类;3. 当前工程类路径下或者由系统变量 java.class.path 指定位置中的类。

### 73.4 在编写自定义类加载器时，如果将父类加载器强制设置 为 null，那么会有什么影响?如果自定义的类加载器不能加 载指定类，就肯定会加载失败吗? ###

JVM 规范中规定如果用户自定义的类加载器将父类加载器强制设置为 null，那么会自 动将启动类加载器设置为当前用户自定义类加载器的父类加载器

同时，我们可以得出如下结论:
即使用户自定义类加载器不指定父类加载器，那么，同样可以加载到 <Java_Runtime_Home>/lib 下的类，但此时就不能够加载 <Java_Runtime_Home>/lib/ext 目录下的类了。

### 73.5 编写自定义类加载器时，一般有哪些注意点? ###

* 一般尽量不要覆写已有的 loadClass(...)方法中的委派逻辑
* 正确设置父类加载器
* 保证 findClass(String name)方法的逻辑正确性

### 73.6 如何在运行时判断系统类加载器能加载哪些路径下的类 ###

* 可以直接调用 ClassLoader.getSystemClassLoader()或者其他方式获取到系统类 加载器(系统类加载器和扩展类加载器本身都派生自 URLClassLoader)，调用 URLClassLoader 中的 getURLs()方法可以获取到。
* 可以直接通过获取系统属性 java.class.path 来查看当前类路径上的条目信息 : System.getProperty("java.class.path")。

### 74 开发自己的类加载器 ###

在 Java 虚拟机判断两 个类是否相同的时候，使用的是类的定义加载器。也就是说，哪个类加载器启动类的加载 过程并不重要，重要的是最终定义这个类的加载器。两种类加载器的关联之处在于:一个 类的定义加载器是它引用的其它类的初始加载器。

对于一个类加载器实例来说，相同全名的类只加载一次，即 loadClass 方法 不会被重复调用。

### 75 xml 解析方式 ###

**区别:**

* DOM4J 性能最好，连 Sun 的 JAXM 也在用 DOM4J。目前许多开源项目中 大量采用 DOM4J，例如大名鼎鼎的 hibernate 也用 DOM4J 来读取 XML 配置 文件。如果不考虑可移植性，那就采用 DOM4J.
* JDOM 和 DOM 在性能测试时表现不佳，在测试 10M 文档时内存溢出。在小文档情况下还值得考虑使用 DOM 和 JDOM。虽然 JDOM 的开发者已经说明他们期望在正式发行版前专注性能问题，但是从性能 观点来看，它确实没有值得推荐之处。另外，DOM 仍是一个非常好的选择。 DOM 实现广泛应用于多种编程语言。它还是许多其它与 XML 相关的标准的基 础，因为它正式获得 W3C
推荐(与基于非标准的 Java 模型相对)，所以在某些类型的项目中可能也需要它(如在 JavaScript 中使用 DOM)。
* SAX 表现较好，这要依赖于它特定的解析方式-事件驱动。一个 SAX 检测 即将到来的 XML 流，但并没有载入到内存(当然当 XML 流被读入时，会有部分 文档暂时隐藏在内存中)。

### 76 Statement 和 PreparedStatement 之间的区别 ###

* PreparedStatement 是预编译的,对于批量处理可以大大提高效率. 也叫 JDBC 存储过程
* Statement 对象。在对数据库只执行一次性存取的时侯，用
Statement 对象进行处理。PreparedStatement
对象的开销比 Statement 大，对于一次性操作并不会带来额外的好处。 
* statement 每次执行 sql 语句，相关数据库都要执行 sql 语句的编译， preparedstatement 是预编译得,preparedstatement 支持批处理

### 77 servlet 生命周期及各个方法 ###

* Servlet 生命周期:Servlet 加载--->实例化--->服务--->销毁

### 78 Tomcat 缓存策略 ###

* 对于服务器端经常不变化文件，设置客户端缓存时间，在客户端资源缓存时间到期之前， 就不会去访问服务器获取该资源 -------- 比 tomcat 内置缓存策略更优手段
* 减少服务器请求次数，提升性能
设置静态资源缓存时间，需要设置Expires过期时间 ，在客户端资源没有过期之前，不 会产生对该资源的请求的
* 设置 Expires 通常使用 response.setDateHeader 进行设置 设置毫秒值

### 79 MVC 概念 ###

* Model :模型层(用于数据库打交道)
* View :视图层(用于展示内容给用户看)
* Controller :控制层(控制业务逻辑)

### 80 缓存机制对比 ###
### 80.1 Hibernate 缓存 ###

Hibernate 一级缓存是 Session 缓存，利用好一级缓存就需要对 Session 的生命周期进行 管理好。建议在一个 Action 操作中使用一个 Session。一级缓存需要对 Session 进行严格 管理。

Hibernate 二级缓存是 SessionFactory 级的缓存。 SessionFactory 的缓存分为内置缓存 和外置缓存。内置缓存中存放的是 SessionFactory 对象的一些集合属性包含的数据(映射元素据及预定 SQL 语句等),对于应用程序来说,它是只读的。外置缓存中存放的是数据库数 据的副本,其作用和一级缓存类似.二级缓存除了以内存作为存储介质外,还可以选用硬盘等 外部存储设备。二级缓存称为进程级缓存或 SessionFactory 级缓存，它可以被所有 session 共享，它的生命周期伴随着 SessionFactory 的生命周期存在和消亡。

### 81 Spring 如何实现 AOP 和 IOC ###

* IoC(Inversion of Control)是指容器控制程序对象之间的关系，而不是传统实现中，由程序代码直接操控。 控制权由应用代码中转到了外部容器，控制权的转移是所谓反转。对于 Spring 而言，就是由 Spring 来控制对象的生 命周期和对象之间的关系;IoC 还有另外一个名字——“依赖注入(Dependency Injection)”。从名字上理解，所谓 依赖注入，即组件之间的依赖关系由容器在运行期决定，即由容器动态地将某种依赖关系注入到组件之中。
* 在 Spring 的工作方式中，所有的类都会在 spring 容器中登记，告诉 spring 这是个什么东西，你需要什么东 西，然后 spring 会在系统运行到适当的时候，把你要的东西主动给你，同时也把你交给其他需要你的东西。所有的类 的创建、销毁都由 spring 来控制，也就是说控制对象生存周期的不再是引用它的对象，而是 spring。对于某个具体 的对象而言，以前是它控制其他对象，现在是所有对象都被 spring 控制，所以这叫控制反转。
* 在系统运行中，动态的向某个对象提供它所需要的其他对象。
* 依赖注入的思想是通过反射机制实现的，在实例化一个类时，它通过反射调用类中 set 方法将事先保存在 HashMap 中的类属性注入到类中。
* 在传统的对象创建方式中，通常由调用者来创建被调用者的实例，而在 Spring 中创建被调用者的工作由 Spring 来完成，然后注入调用者，即所谓的依赖注入 or 控制反转。
* IoC 的优点:降低了组件之间的耦合，降低了业务对象之间替换的复杂性，使之能够灵活的管理对象。

### 82 AOP ###

Spring 实现 AOP:JDK 动态代理和 CGLIB 代理

JDK 动态代理:其代理对象必须是某个接口的实现，它是通过在运行期间创建一个接口的实现类来完成对目标对象的代理;其核心的两个类是 InvocationHandler 和 Proxy。

CGLIB 代理:实现原理类似于 JDK 动态代理，只是它在运行期间生成的代理对象是针对目标类扩展的子类。 CGLIB 是高效的代码生成包，底层是依靠 ASM(开源的 java 字节码编辑类库)操作字节码实现的，性能比 JDK 强; 需要引入包 asm.jar 和 cglib.jar。

### 83 spring 有以下的优点 ###

* 降低了组件之间的耦合性 ，实现了软件各层之间的解耦 
* 可以使用容易提供的众多服务，如事务管理，消息服务等 
* 容器提供单例模式支持
* 容器提供了 AOP 技术，利用它很容易实现如权限拦截，运行期监控等功能 
* 容器提供了众多的辅助类，能加快应用的开发
* spring 对于主流的应用框架提供了集成支持，如 hibernate，JPA，Struts 等 * spring 属于低侵入式设计，代码的污染极低
* 独立于各种应用服务器
* spring 的 DI 机制降低了业务对象替换的复杂性
* Spring 的高度开放性，并不强制应用完全依赖于 Spring，开发者可以自由选择 spring 的部分或全部

### 84 Springbean 注入方式 ###

**Set注入**

**构造器注入**

**静态工厂注入**

**实例工厂注入**

### 85 事务有四个特性 ACID ###

**原子性(Atomicity)**

事务是一个原子操作，由一系列动作组成。事务的原子性确 保动作要么全部完成，要么完全不起作用。

**一致性(Consistency)**

一旦事务完成(不管成功还是失败)，系统必须确保它所建 模的业务处于一致的状态，而不会是部分完成部分失败。在现实中的数据不应该被 破坏。

**隔离性(Isolation)**

可能有许多事务会同时处理相同的数据，因此每个事务都应 该与其他事务隔离开来，防止数据损坏。

**持久性(Durability)**

一旦事务完成，无论发生什么系统错误，它的结果都不应该 受到影响，这样就能从任何系统崩溃中恢复过来。通常情况下，事务的结果被写到 持久化存储器中。

### 86 隔离级别 ###

* 脏读(Dirty reads)——脏读发生在一个事务读取了另一个事务改写但尚未提交的 数据时。如果改写在稍后被回滚了，那么第一个事务获取的数据就是无效的。
* 不可重复读(Nonrepeatable read)——不可重复读发生在一个事务执行相同的查
询两次或两次以上，但是每次都得到不同的数据时。这通常是因为另一个并发事务在两次查询期间进行了更新。
* 幻读(Phantom read)——幻读与不可重复读类似。它发生在一个事务(T1)读
取了几行数据，接着另一个并发事务(T2)插入了一些数据时。在随后的查询 中，第一个事务(T1)就会发现多了一些原本不存在的记录。

隔离级别  | 含义
------------- | -------------
ISOLATION_DEFAULT|使用后端数据库默认的隔离级别
ISOLATION_READ_UNCOMMITTED|最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读
ISOLATION_READ_COMMITTED|允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重
ISOLATION_REPEATABLE_READ|对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所仍有可能发生
ISOLATION_SERIALIZABLE|最高的隔离级别，完全服从 ACID 的隔离级别，确保阻止脏读、不可重复 为它通常是通过完全锁定事务相关的数据库表来实现的

### 87 TCP的三次握手与四次挥手理解 ###

![](https://img-blog.csdn.net/20180717201939345?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4OTUwMzE2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

    序列号seq：占4个字节，用来标记数据段的顺序，TCP把连接中发送的所有数据字节都编上一个序号，第一个字节的编号由本地随机产生；给字节编上序号后，就给每一个报文段指派一个序号；序列号seq就是这个报文段中的第一个字节的数据编号。

    确认号ack：占4个字节，期待收到对方下一个报文段的第一个数据字节的序号；序列号表示报文段携带数据的第一个字节的编号；而确认号指的是期望接收到下一个字节的编号；因此当前报文段最后一个字节的编号+1即为确认号。

    确认ACK：占1位，仅当ACK=1时，确认号字段才有效。ACK=0时，确认号无效

    同步SYN：连接建立时用于同步序号。当SYN=1，ACK=0时表示：这是一个连接请求报文段。若同意连接，则在响应报文段中使得SYN=1，ACK=1。因此，SYN=1表示这是一个连接请求，或连接接受报文。SYN这个标志位只有在TCP建产连接时才会被置1，握手完成后SYN标志位被置0。

    终止FIN：用来释放一个连接。FIN=1表示：此报文段的发送方的数据已经发送完毕，并要求释放运输连接

    PS：ACK、SYN和FIN这些大写的单词表示标志位，其值要么是1，要么是0；ack、seq小写的单词表示序号。

| 字段 | 含义 |
| --- | --- |
| URG | 紧急指针是否有效。为1，表示某一位需要被优先处理 |
| ACK | 确认号是否有效，一般置为1。 |
| PSH | 提示接收端应用程序立即从TCP缓冲区把数据读走。 |
| RST | 对方要求重新建立连接，复位。 |
| SYN | 请求建立连接，并在其序列号的字段进行序列号的初始值设定。建立连接，设置为1 |
| FIN     | 希望断开连接。 |

### 三次握手过程理解

![](https://img-blog.csdn.net/20180717202520531?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4OTUwMzE2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

第一次握手：建立连接时，客户端发送syn包（syn=j）到服务器，并进入**SYN\_SENT**状态，等待服务器确认；SYN：同步序列编号（Synchronize Sequence Numbers）。

第二次握手：服务器收到syn包，必须确认客户的SYN（ack=j+1），同时自己也发送一个SYN包（syn=k），即SYN+ACK包，此时服务器进入**SYN\_RECV**状态；

第三次握手：客户端收到服务器的SYN+ACK包，向服务器发送确认包ACK(ack=k+1），此包发送完毕，客户端和服务器进入**ESTABLISHED**（TCP连接成功）状态，完成三次握手。

### 四次挥手过程理解

![](https://img-blog.csdn.net/20180717204202563?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4OTUwMzE2/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

1）客户端进程发出连接释放报文，并且停止发送数据。释放数据报文首部，FIN=1，其序列号为seq=u（等于前面已经传送过来的数据的最后一个字节的序号加1），此时，客户端进入FIN\-WAIT\-1（终止等待1）状态。 TCP规定，FIN报文段即使不携带数据，也要消耗一个序号。
2）服务器收到连接释放报文，发出确认报文，ACK=1，ack=u+1，并且带上自己的序列号seq=v，此时，服务端就进入了CLOSE\-WAIT（关闭等待）状态。TCP服务器通知高层的应用进程，客户端向服务器的方向就释放了，这时候处于半关闭状态，即客户端已经没有数据要发送了，但是服务器若发送数据，客户端依然要接受。这个状态还要持续一段时间，也就是整个CLOSE\-WAIT状态持续的时间。
3）客户端收到服务器的确认请求后，此时，客户端就进入FIN\-WAIT\-2（终止等待2）状态，等待服务器发送连接释放报文（在这之前还需要接受服务器发送的最后的数据）。
4）服务器将最后的数据发送完毕后，就向客户端发送连接释放报文，FIN=1，ack=u+1，由于在半关闭状态，服务器很可能又发送了一些数据，假定此时的序列号为seq=w，此时，服务器就进入了LAST\-ACK（最后确认）状态，等待客户端的确认。
5）客户端收到服务器的连接释放报文后，必须发出确认，ACK=1，ack=w+1，而自己的序列号是seq=u+1，此时，客户端就进入了TIME\-WAIT（时间等待）状态。注意此时TCP连接还没有释放，必须经过2∗∗MSL（最长报文段寿命）的时间后，当客户端撤销相应的TCB后，才进入CLOSED状态。
6）服务器只要收到了客户端发出的确认，立即进入CLOSED状态。同样，撤销TCB后，就结束了这次的TCP连接。可以看到，服务器结束TCP连接的时间要比客户端早一些。

###  常见面试题

【问题1】为什么连接的时候是三次握手，关闭的时候却是四次握手？

答：因为当Server端收到Client端的SYN连接请求报文后，可以直接发送SYN+ACK报文。其中ACK报文是用来应答的，SYN报文是用来同步的。但是关闭连接时，当Server端收到FIN报文时，很可能并不会立即关闭SOCKET，所以只能先回复一个ACK报文，告诉Client端，"你发的FIN报文我收到了"。只有等到我Server端所有的报文都发送完了，我才能发送FIN报文，因此不能一起发送。故需要四步握手。

【问题2】为什么TIME\_WAIT状态需要经过2MSL(最大报文段生存时间)才能返回到CLOSE状态？

答：虽然按道理，四个报文都发送完毕，我们可以直接进入CLOSE状态了，但是我们必须假象网络是不可靠的，有可以最后一个ACK丢失。所以TIME\_WAIT状态就是用来重发可能丢失的ACK报文。在Client发送出最后的ACK回复，但该ACK可能丢失。Server如果没有收到ACK，将不断重复发送FIN片段。所以Client不能立即关闭，它必须确认Server接收到了该ACK。Client会在发送出ACK之后进入到TIME\_WAIT状态。Client会设置一个计时器，等待2MSL的时间。如果在该时间内再次收到FIN，那么Client会重发ACK并再次等待2MSL。所谓的2MSL是两倍的MSL(Maximum Segment Lifetime)。MSL指一个片段在网络中最大的存活时间，2MSL就是一个发送和一个回复所需的最大时间。如果直到2MSL，Client都没有再次收到FIN，那么Client推断ACK已经被成功接收，则结束TCP连接。

【问题3】为什么不能用两次握手进行连接？

答：3次握手完成两个重要的功能，既要双方做好发送数据的准备工作(双方都知道彼此已准备好)，也要允许双方就初始序列号进行协商，这个序列号在握手过程中被发送和确认。

       现在把三次握手改成仅需要两次握手，死锁是可能发生的。作为例子，考虑计算机S和C之间的通信，假定C给S发送一个连接请求分组，S收到了这个分组，并发 送了确认应答分组。按照两次握手的协定，S认为连接已经成功地建立了，可以开始发送数据分组。可是，C在S的应答分组在传输中被丢失的情况下，将不知道S 是否已准备好，不知道S建立什么样的序列号，C甚至怀疑S是否收到自己的连接请求分组。在这种情况下，C认为连接还未建立成功，将忽略S发来的任何数据分 组，只等待连接确认应答分组。而S在发出的分组超时后，重复发送同样的分组。这样就形成了死锁。

【问题4】如果已经建立了连接，但是客户端突然出现故障了怎么办？

TCP还设有一个保活计时器，显然，客户端如果出现故障，服务器不能一直等下去，白白浪费资源。服务器每收到一次客户端的请求后都会重新复位这个计时器，时间通常是设置为2小时，若两小时还没有收到客户端的任何数据，服务器就会发送一个探测报文段，以后每隔75秒钟发送一次。若一连发送10个探测报文仍然没反应，服务器就认为客户端出了故障，接着就关闭连接。

### 88 redis 清理策略 ###

当Redis所用内存打到maxmemory上限时会触发响应的溢出控制策略，具体策略受maxmemory\-policy参数控制，Redis支持6中策略，如下所示：

1.  noeviction：默认策略，不会删除任何数据，拒绝所有写入操作并返回客户端错误信息，此时Redis只响应读操作。
2.  volatitle\-rlu：根据LRU算法删除设置了超时属性的键，知道腾出足够空间为止。如果没有可删除的键对象，回退到noeviction策略。
3.  allkeys\-lru：根据LRU算法删除键，不管数据有没有设置超时属性，直到腾出足够空间为止。
4.  allkeys\-random：随机删除所有键，知道腾出足够空间为止。
5.  volatitle\-random：随机删除过期键，知道腾出足够空间为止。
6.  volatitle\-ttl：根据键值对象的ttl属性，删除最近将要过期数据。如果没有，回退到noeviction策略

### 89 kafka消息如何不重复消费和消息丢失 ###

### 90 springmvc 用过哪些注解 ###

**@Controller**

Controller 控制器是通过服务接口定义的提供访问应用程序的一种 行为，它解释用户的输入，将其转换成一个模型然后将试图呈献给用户。

**@RequestMapping**

 @RequestMapping 注解将类似 “/favsoft”这样的 URL 映 射到整个类或特定的处理方法上。
 
**@PathVariable**

使用 @PathVariable 注解方法参数并将其绑 定到 URI 模板变量的值上。

**@RequestParam**

@RequestParam 将请求的参数绑定到方法中的参数上

**@RequestBody**

@RequestBody 是指方法参数应该被绑定到 HTTP 请求 Body 上

**@ResponseBody**

@ResponseBody 与@RequestBody 类似，它的作用是将返回类型直接输 入到 HTTP response body 中。

### 91 Restful 好处 ###

* 客户-服务器:客户-服务器约束背后的原则是分离关注点。通过分离用户接 口和数据存储这两个关注点，改善了用户接口跨多个平台的可移植性;同时通 过简化服务器组件，改善了系统的可伸缩性。 
* 无状态:通信在本质上是无状态的，改善了可见性、可靠性、可伸缩性.
* 缓存:改善了网络效率减少一系列交互的平均延迟时间，来提高效率、可伸 缩性和用户可觉察的性能。* 统一接口:REST 架构风格区别于其他基于网络的架构风格的核心特征是， 它强调组件之间要有一个统一的接口。

### 92 Tomcat，Apache，JBoss 的区别 ###

Apache:HTTP 服务器(WEB 服务器)，类似 IIS，可以用于建立虚拟站点，编译 处理静态页面，可以支持 SSL 技术，支持多个虚拟主机等功能。 

Tomcat:Servlet 容器，用于解析 jsp，Servlet 的 Servlet 容器，是高效，轻量级 的容器。缺点是不支持 EJB，只能用于 java 应用。

Jboss:应用服务器，运行 EJB 的 J2EE 应用服务器，遵循 J2EE 规范，能够提 供更多平台的支持和更多集成功能，如数据库连接，JCA 等，其对 Servlet 的 支持是通过集成其他 Servlet 容器来实现的，如 tomcat 和 jetty。

### 93 redis的分布式锁原理和实现 ###

### 94 json 和 xml 区别 ###

**XML**

* 应用广泛，可扩展性强，被广泛应用各种场合; 
* 读取、解析没有 JSON 快; 
* 可读性强，可描述复杂结构。

**JSON**

* 结构简单，都是键值对;
* 读取、解析速度快，很多语言支持; 
* 传输数据量小，传输速率大大提高; 
* 描述复杂结构能力较弱。

### 95 设计模式 ###
23 种设计模式

**创建型**

* Factory Method(工厂方法)
* Abstract Factory(抽象工厂) 
* Builder(建造者)
* Prototype(原型)
* Singleton(单例)

**结构型**

* Adapter Class/Object(适配器) 
* Bridge(桥接)
* Composite(组合)
* Decorator(装饰)      * Facade(外观)
* Flyweight(享元)
* Proxy(代理)

**行为型**

* Interpreter(解释器)
* Template Method(模板方法)
* Chain of Responsibility(责任链) 16. Command(命令)
* Iterator(迭代器)
* Mediator(中介者)
* Memento(备忘录)
* Observer(观察者) 
* State(状态)
* Strategy(策略) 
* Visitor(访问者)

### 96 设计模式的六大原则 ###

* 单一职责原则 
* 里氏替换原则 
* 依赖倒置原则 
* 接口隔离原则 
* 迪米特法则 
* 开闭原则

**单一职责原则**

一个类只负责一项职责。

**里氏替换原则**

子类可以扩展父类的功能，但不能改变父类原有的功能。

* 子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法。 
* 子类中可以增加自己特有的方法。
* 当子类的方法重载父类的方法时，方法的前置条件(即方法的形参)要比父类方法的输入参数 更宽松。
* 当子类的方法实现父类的抽象方法时，方法的后置条件(即方法的返回值)要比父类更严格。

**依赖倒置原则**

高层模块不应该依赖低层模块，二者都应该依赖其抽象;抽象不应该依赖细节; 细节应该依赖抽象。

**接口隔离原则**

客户端不应该依赖它不需要的接口;一个类对另一个类的依赖应该建立在最小的接口上。

**迪米特法则**

一个对象应该对其他对象保持最少的了解。尽量降低类与类之间的耦合。

**开闭原则**

一个软件实体如类、模块和函数应该对扩展开放，对修改关闭。

### 97 高内聚，低耦合方面的理解 ###

* 耦合、内聚的评估标准是强度，耦合越弱越好，内聚越强越好;
* 所谓过度指的是由于错误理解导致的效果相反的设计;
* 内聚指的是模块内部的功能，最强的内聚就是功能单一到不能拆分，也就是原子化，
* 所以强内聚和弱耦合是相辅相成的，一个良好的设计是由若干个强内聚模块以弱耦合的方式组装起来的

### 98 深度优先和广度优先算法 ###

图的存储结构有两种：

* 一种是基于二维数组的邻接矩阵表示法。
* 另一种是基于链表的的邻接表。

**深度优先遍历算法**

**广度优先搜索算法**

所谓广度，就是一层一层的，向下遍历，层层堵截

### 99 两种算法分析 ###

两种算法的复杂度分析
**深度优先**
数组表示:查找所有顶点的所有邻接点所需时间为 O(n2)，n 为顶点数，算法时间复杂度为 O(n2)

**广度优先**

数组表示:查找每个顶点的邻接点所需时间为 O(n2)，n 为顶点数，算法的时间复杂度为 O(n2)

### 100 常用排序算法的时间复杂度和空间复杂度 ###
![这里写图片描述](https://img-blog.csdn.net/20160527111747945)

总结：

* 当排序记录个数n较大，关键码分布较随机，且对稳定性不作要求时，采用快速排序为宜。
* 当待排序记录个数n较大，内存空间允许，且要求稳定排序时，采用归并排序。
* 当待排序记录个数n较大，关键码分布可能出现正序或逆序的情况，且对稳定性不作要求时，采用堆排序或归并排序。
* 当待排序记录个数n较大，而只要找出最小的前几个记录，采用堆排序或简单选择排序。
* 当待排序记录个数n较小（如小于100）时，记录已基本有序，且要求稳定时，采用直接插入排序。
* 当待排序记录个数n较小，记录所含数据项较多，所占存储空间较大时，采用简单选择排序。

补充：
时间复杂度和空间复杂度

时间复杂度，就是说算法在运行过程中所耗用的时间，而空间复杂度，则是算法运行过程中所占用的空间（内存、硬盘等等）。

### 101 B树、B-树、B+树、B*树都是什么 ###

**B树**

 即二叉搜索树：

1. 所有非叶子结点至多拥有两个儿子（Left和Right ）；

2. 所有结点存储一个关键字；

3. 非叶子结点的左指针指向小于其关键字的子树，右指针指向大于其关键字的子树；

 如：

       [![image](https://images.cnblogs.com/cnblogs_com/syxchina/201110/201110012318571704.png "image")](http://images.cnblogs.com/cnblogs_com/syxchina/201110/201110012318579229.png)‍

       B树的搜索，从根结点开始，如果查询的关键字与结点的关键字相等，那么就命中；否则，如果查询关键字比结点关键字小，就进入左儿子；如果比结点关键字大，就进入右儿子；如果左儿子或右儿子的指针为空，则报告找不到相应的关键字；

 如果B树的所有非叶子结点的左右子树的结点数目均保持差不多（平衡），那么B 树的搜索性能逼近二分查找；但它比连续内存空间的二分查找的优点是，改变B树结构（插入与删除结点）不需要移动大段的内存数据，甚至通常是常数开销；

 如：

      [![image](https://images.cnblogs.com/cnblogs_com/syxchina/201110/201110012318575575.png "image")](http://images.cnblogs.com/cnblogs_com/syxchina/201110/201110012318575052.png)‍

   但B树在经过多次插入与删除后，有可能导致不同的结构：

    [![image](https://images.cnblogs.com/cnblogs_com/syxchina/201110/201110012318587178.png "image")](http://images.cnblogs.com/cnblogs_com/syxchina/201110/20111001231857559.png)‍

   右边也是一个B树，但它的搜索性能已经是线性的了；同样的关键字集合有可能导致不同的树结构索引；所以，使用 B树还要考虑尽可能让B树保持左图的结构，和避免右图的结构，也就是所谓的“平衡”问题；

 实际使用的B树都是在原B树的基础上加上平衡算法，即“平衡二叉树”；如何保持 B树结点分布均匀的平衡算法是平衡二叉树的关键；平衡算法是一种在B树中插入和删除结点的策略；

**B\-树**

 是一种多路搜索树（并不是二叉的）：

1. 定义任意非叶子结点最多只有M个儿子；且M>2 ；

2. 根结点的儿子数为\[2, M\]；

3. 除根结点以外的非叶子结点的儿子数为\[M/2, M\]；

4. 每个结点存放至少M/2\-1（取上整）和至多M\-1个关键字；（至少 2个关键字）

5. 非叶子结点的关键字个数\=指向儿子的指针个数\-1；

6. 非叶子结点的关键字：K\[1\], K\[2\], …, K\[M\-1\]；且K\[i\] < K\[i+1\] ；

7. 非叶子结点的指针：P\[1\], P\[2\], …, P\[M\]；其中P\[1\] 指向关键字小于K\[1\]的子树，P\[M\]指向关键字大于 K\[M\-1\]的子树，其它P\[i\]指向关键字属于(K\[i\-1\], K\[i\]) 的子树；

8. 所有叶子结点位于同一层；

 如：（M=3）

     [![image](https://images.cnblogs.com/cnblogs_com/syxchina/201110/201110012318587145.png "image")](http://images.cnblogs.com/cnblogs_com/syxchina/201110/201110012318582162.png)‍

       B\-树的搜索，从根结点开始，对结点内的关键字（有序）序列进行二分查找，如果命中则结束，否则进入查询关键字所属范围的儿子结点；重复，直到所对应的儿子指针为空，或已经是叶子结点；

B\-树的特性：

1. 关键字集合分布在整颗树中；

2. 任何一个关键字出现且只出现在一个结点中；

3. 搜索有可能在非叶子结点结束；

4. 其搜索性能等价于在关键字全集内做一次二分查找；

5. 自动层次控制；

 由于限制了除根结点以外的非叶子结点，至少含有M/2个儿子，确保了结点的至少利用率，其最底搜索性能为：

             [![image](https://images.cnblogs.com/cnblogs_com/syxchina/201110/201110012318589936.png "image")](http://images.cnblogs.com/cnblogs_com/syxchina/201110/201110012318584953.png)‍

 其中，M为设定的非叶子结点最多子树个数，N为关键字总数；

 所以B\-树的性能总是等价于二分查找（与M值无关），也就没有 B树平衡的问题；

 由于M/2的限制，在插入结点时，如果结点已满，需要将结点分裂为两个各占M/2 的结点；删除结点时，需将两个不足M/2的兄弟结点合并；

**B+树**

B+ 树是B\-树的变体，也是一种多路搜索树：

1. 其定义基本与B\-树同，除了：

2. 非叶子结点的子树指针与关键字个数相同；

3. 非叶子结点的子树指针P\[i\]，指向关键字值属于\[K\[i\], K\[i+1\]) 的子树（B\-树是开区间）；

5. 为所有叶子结点增加一个链指针；

6. 所有关键字都在叶子结点出现；

 如：（M=3）

     [![image](https://images.cnblogs.com/cnblogs_com/syxchina/201110/20111001231859983.png "image")](http://images.cnblogs.com/cnblogs_com/syxchina/201110/201110012318582096.png)‍

 B+的搜索与 B\-树也基本相同，区别是B+树只有达到叶子结点才命中（B\- 树可以在非叶子结点命中），其性能也等价于在关键字全集做一次二分查找；

B+ 的特性：

1. 所有关键字都出现在叶子结点的链表中（稠密索引），且链表中的关键字恰好是有序的；

2. 不可能在非叶子结点命中；

3. 非叶子结点相当于是叶子结点的索引（稀疏索引），叶子结点相当于是存储（关键字）数据的数据层；

4. 更适合文件索引系统；

**B\*树**

 是B+树的变体，在B+树的非根和非叶子结点再增加指向兄弟的指针；

     [![image](https://images.cnblogs.com/cnblogs_com/syxchina/201110/201110012319014688.png "image")](http://images.cnblogs.com/cnblogs_com/syxchina/201110/201110012318597013.png)‍

 B\*树定义了非叶子结点关键字个数至少为 (2/3)\*M，即块的最低使用率为2/3（代替B+ 树的1/2）；

B+ 树的分裂：当一个结点满时，分配一个新的结点，并将原结点中1/2的数据复制到新结点，最后在父结点中增加新结点的指针；B+ 树的分裂只影响原结点和父结点，而不会影响兄弟结点，所以它不需要指向兄弟的指针；

B\* 树的分裂：当一个结点满时，如果它的下一个兄弟结点未满，那么将一部分数据移到兄弟结点中，再在原结点插入关键字，最后修改父结点中兄弟结点的关键字（因为兄弟结点的关键字范围改变了）；如果兄弟也满了，则在原结点与兄弟结点之间增加新结点，并各复制1/3的数据到新结点，最后在父结点增加新结点的指针；

 所以，B\*树分配新结点的概率比B+树要低，空间使用率更高；

**小结**

B 树：二叉树，每个结点只存储一个关键字，等于则命中，小于走左结点，大于走右结点；

B\- 树：多路搜索树，每个结点存储M/2到M个关键字，非叶子结点存储指向关键字范围的子结点；

 所有关键字在整颗树中出现，且只出现一次，非叶子结点可以命中；

B+ 树：在B\-树基础上，为叶子结点增加链表指针，所有关键字都在叶子结点中出现，非叶子结点作为叶子结点的索引；B+ 树总是到叶子结点才命中；

B\* 树：在B+树基础上，为非叶子结点也增加链表指针，将结点的最低利用率从1/2 提高到2/3；

### 102 hash 算法及常用的 hash 算法 ###

散列表,它是基于快速存取的角度设计的，也是一种典型的“空间换时间”的做法。顾名思义，该数据结构可以理解为一个线性表，但是其中的元素不是紧密排列的，而是可能存在空隙。

散列表(Hash table，也叫哈希表)，是根据关键码值(Key value)而直接进行访问的数据结构。也就是 说，它通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做散列函 数，存放记录的数组叫做散列表。

解决冲突是一个复杂问题。冲突主要取决于: 

(1)散列函数，一个好的散列函数的值应尽可能平均分布。

(2)处理冲突方法。 

(3)负载因子的大小。太大不一定就好，而且浪费空间严重，负载因子和散列函数是联动的。

解决冲突的办法: 

(1)线性探查法:冲突后，线性向前试探，找到最近的一个空位置。缺点是会出现堆积现象。存取时，可能不是同义词的词也位于探查序列，影响效率。

(2)双散列函数法:在位置 d 冲突后，再次使用另一个散列函数产生一个与散列表桶容量 m 互质的数 c，依次试探(d+n*c)%m，使探查序列跳跃式分布。

### 103 常用的构造散列函数的方法 ###

散列函数能使对一个数据序列的访问过程更加迅速有效，通过散列函数，数据元素将被更快地定位: 

* **直接寻址法**:取关键字或关键字的某个线性函数值为散列地址。即 H(key)=key 或 H(key) =a?key + b，其中 a 和 b 为常数(这种散列函数叫做自身函数)
* **数字分析法**:分析一组数据，比如一组员工的出生年月日，这时我们发现出生年月日的前几位数 字大体相 同，这样的话，出现冲突的几率就会很大，但是我们发现年月日的后几位表示月份和具体日期 的数字差别很大，如果用后面的数字来构成散列地址，则冲突的几率会 明显降低。因此数字分析法就是 找出数字的规律，尽可能利用这些数据来构造冲突几率较低的散列地址。
* **平方取中法**:取关键字平方后的中间几位作为散列地址。
* **折叠法**:将关键字分割成位数相同的几部分，最后一部分位数可以不同，然后取这几部分的叠加 和(去除进位)作为散列地址。
* **随机数法**:选择一随机函数，取关键字的随机值作为散列地址，通常用于关键字长度不同的场 合。
* **除留余数法**:取关键字被某个不大于散列表表长 m 的数 p 除后所得的余数为散列地址。即 H(key) = key MOD p, p<=m。不仅可以对关键字直接取模，也可在折叠、平方取中等运算之后取模。 对 p 的选择很重要，一般取素数或 m，若 p 选的不好，容易产生同义词。

### 104 著名的 hash 算法 ###

**MD4**

**MD5**

**SHA-1 及其他**

### 105 Hash 算法在信息安全方面的应用主要体现在以下的 3 个方面 ###

**文件校验**

**数字签名**

**鉴权协议**

### 105 Hash 函数可以简单的划分为如下几类 ###

1. 加法 Hash;
2. 位运算 Hash;
3. 乘法 Hash;
4. 除法 Hash; 
5. 查表 Hash;
6. 混合 Hash;

### 106 如何判断一个单链表是否有环 ###

**三类情况**

![](https://upload-images.jianshu.io/upload_images/1013002-5b6e77d38107adab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/460)

（1）

![](https://upload-images.jianshu.io/upload_images/1013002-525fc0c649adc276.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/166)

（2）

![](https://upload-images.jianshu.io/upload_images/1013002-bdc5fb37cce6c648.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/398)

（3）

1、遇到这个问题，首先想到的是遍历链表，寻找是否有相同地址，借此判断链表中是否有环。

```
listnode_ptr current =head->next;
while(current)
{
  if(current==head)
  {
    printf("有环！\n");
    return 0;
  }
  else
  {
    current=current->next;
  }
}
printf("无环！\n");
return 0;

```

这段代码满足了（1）（链表无环）、（2）（链表头尾相连)两类情况，却没有将（3）情况考虑在内，如果出现（3）类情况，程序会进入死循环。

2、将（3）考虑在内，首先想到我们可能需要一块空间来存储指针，遍历新指针时将其和储存的旧指针比对，若有相同指针，则该链表有环，否则将这个新指针存下来后继续往下读取，直到遇见NULL，这说明这个链表无环。

#### 上述方法虽然可行，可是否还有更简便的算法？

3、假设有两个学生A和B在跑道上跑步，两人从相同起点出发，假设A的速度为2m/s，B的速度为1m/s,结果会发生什么？
答案很简单，A绕了跑道一圈之后会追上B！
将这个问题延伸到链表中，跑道就是链表，我们可以设置两个指针，a跑的快，b跑的慢，如果链表有环，那么当程序执行到某一状态时，a==b。如果链表没有环，程序会执行到a==NULL，结束。

```
listnode_ptr fast=head->next;
listnode_ptr slow=head;
while(fast)
{
    if(fast==slow)
    {
        printf("环！\n");
        return 0;
    }
    else
    {
        fast=fast->next;
        if(!fast)
        {
            printf("无环！\n");
            return 0;
        }
        else
        {
            fast=fast->next;
            slow=slow->next;
        }
    }
}
printf("无环！\n");
return 0;

```

4、关于算法复杂度：

![](https://upload-images.jianshu.io/upload_images/1013002-432d55b55ec8e41f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/556)

如图，链表长度为n，环节点个数为m，则循环 t=n\-m 次时，slow进入环中，此时，我们假设fast与slow相距x个节点，那么，经过t'次循环，二者相遇时，有：
2t'=t'+（m\-x） \-> t'=m\-x \-> t'<=m.
因此，总共循环了T=t+t' <= n. 算法复杂度为O(n).

**判断是否有环**

package demo6;

import java.util.HashMap;

import demo6.LinkReverse2.Node;


/**
 * 判断链表是否有环的方法
 */
public class LinkLoop {
	
	public static boolean hasLoop(Node n){
		//定义两个指针tmp1,tmp2
		Node tmp1 = n;
		Node tmp2 = n.next;
		
		while(tmp2!=null){
			tmp1 = tmp1.next;  //每次迭代时，指针1走一步，指针2走两步
			tmp2 = tmp2.next.next;
			if(tmp2 == null)return false;//不存在环时，退出
			int d1 = tmp1.val;
			int d2 = tmp2.val;
			if(d1 == d2)return true;//当两个指针重逢时，说明存在环，否则不存在。
			
		}
		return true; //如果tmp2为null，说明元素只有一个，也可以说明是存在环
	}
	
	//方法2：将每次走过的节点保存到hash表中，如果节点在hash表中，则表示存在环
	public static boolean hasLoop2(Node n){
		Node temp1 = n;
		HashMap<Node,Node> ns = new HashMap<Node,Node>();
		while(n!=null){
			if(ns.get(temp1)!=null)return true;
			else ns.put(temp1, temp1);
			temp1 = temp1.next;
			if(temp1 == null)return false;
		}
		return true;
	}
	
	public static void main(String[] args) {
		Node n1 = new Node(1);
		Node n2 = new Node(2);
		Node n3 = new Node(3);
		Node n4 = new Node(4);
		Node n5 = new Node(5);
		
		n1.next = n2;
		n2.next = n3;
		n3.next = n4;
		n4.next = n5;
		n5.next = n1;  //构造一个带环的链表,去除此句表示不带环
		
		System.out.println(hasLoop(n1));
		System.out.println(hasLoop2(n1));
	}
}
执行结果：
true
true

### 107 栈、队列、链表、树、堆 ###

## 栈##

栈是一种动态集合，它是一种LIFO（last in first out后进先出）结构
**栈的实现**：
（1）数组
（2）链表
**栈要记录的数据**：
（1）栈顶位置top
注意这个top有两种理解方式，一种是表示栈的最后一个数据的位置，另一种是表示栈的最后一个数据的下一个位置，这两种理解对栈的操作代码有一定的影响
（2）栈最大大小size
**栈的操作**：
（1）STACK\_EMPTY（）：判断栈是否为空
（2）PUSH（X）：向栈中添加一个值，注意栈是否为满的
（3）POP（）：从栈中弹出一个值，注意栈是否为空

![](https://upload-images.jianshu.io/upload_images/2702997-8837f5d2ba8b3a57.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)

栈

**栈的简要实现**：[github栈](https://link.jianshu.com?t=https://github.com/jackiechen0708/Algorithm/blob/master/datastructure/stack.h)
**栈的应用**：
（1）括号匹配问题

## 队列##

与栈不同，它是一种FIFO（first in first out先进先出）结构
**队列的实现**：
（1）数组
（2）链表
**队列要记录的数据**：
（1）队首位置head：第一个元素位置
（2）队尾位置tail：下一个元素要插入的位置（最后一个元素的下一个位置）
（3）队列最大大小size
**队列的操作**：
（1）ENQUEUE（x）：入队
（2）DEQUEUE（）：出队
（3）EMPTY（）：队列为空，head=tail
（4）FULL（）：队列为满，head=（tail+1）%size

![](https://upload-images.jianshu.io/upload_images/2702997-5c6f36f10b149ae3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/755)

队列

**队列的简要实现**：[github队列](https://link.jianshu.com?t=https://github.com/jackiechen0708/Algorithm/blob/master/datastructure/Queue.h)
**队列的应用**：
（1）

## 链表##

与数组中元素地址连续不同，链表中两个元素地址不一定连续，而是由专门的一个指针指明该元素的后一个（前一个）元素的地址。
**链表种类**：
（1）单向链表：只有指向后一个元素的指针
（2）双向链表：有指向后一个和前一个元素的指针
（3）循环链表：链表内存在一个环
**链表节点（Node）记录的数据**：
（1）要存储的数据data
（2）下一个节点地址Node\* next
（3）若是双向链表还要存储前一个节点地址Node *prev
**链表（LinkedList）记录的数据**：
（1）链表的头指针Node* head
（2）可能还记录链表的尾指针 Node\* tail
**链表的操作**：
（1）SEARCH(x)：链表的搜索
（2）INSERT(i,x)：链表的插入，在第i个位置插入x
（3）DELETE（x）:链表的删除
**哨兵（sentinel）**：
为了减少边界条件的判断（是否为空链表等等），引入哨兵，使得链表永远不为“空”。
**指针和对象的实现**：
有些语言比如Java没有指针（C中就存在指针），这时我们需要考虑指针的替代实现方式。
（1）用二维数组表示指针
我们可以设置一个n\*3的数组记录n个节点，那个3就表示存储的数据、前一个元素的坐标（index）和后一个元素的坐标。
（2）用一维数组表示指针

![](https://upload-images.jianshu.io/upload_images/2702997-0e42e4decc040760.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/610)

一维数组代替指针

## 树##

**树的种类**：
（1）二叉树
二叉树要存储4个数据，分别是节点携带的信息和其父节点、左右子节点的指针。
（2）分支无限制的有根树：
上面二叉树每个节点最多有两个子节点，而分支无限制的有根数则没有这个限制，可能有3个、5个甚至更多子节点。所以存储这种数据结构的问题在于我们事先并不知道应该设置多少个child指针，若设置的少了不能满足要求，设置的过多又会浪费空间。所以这里提出一种新的描述这种数据结构的方法—— **左孩子右兄弟表示法**，这种方法每个节点设置3个指针：父指针、从左数第一个孩子的指针、其右侧相邻的兄弟指针，如下图所示

![](https://upload-images.jianshu.io/upload_images/2702997-685f1b945ace6244.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/757)

左孩子右兄弟表示法

## 堆##

堆实际上是以数组形式存储的二叉树
**堆需要存储的数据**:
（1）数组的大小max\-size
（2）堆元素个数size，这里size要小于max\-size
**堆中元素通过坐标来确定父节点、左右子节点，具体来说**：
一个节点i的父节点：\[i/2\]
一个节点i的左子节点：\[i *2\]
一个节点i的右子节点：\[i*2+1\]
**堆的分类**：
（1）最大堆
满足所有节点都比其父节点值小（小于等于）的堆
A\[i/2\]>=A\[i\]
（2）最小堆
满足所有节点都比其父节点值大（大于等于）的堆
A\[i/2\]<=A\[i\]
**堆的操作**：
（1）维护堆的性质（HEAPIFY）
这里指维护最大堆或最小堆的性质。假设一个数组中下标为i的节点的子节点满足最大（小）堆性质，但自身不一定满足这个性质，这时就需要HEAPIFY，具体来说是要比较这个节点和其两个子节点的大小，将其中的大（小）的和该节点位置交换，这样这个节点及其两个子节点就满足最大（小）堆的性质了，但是可能交换后子节点不满足堆的性质，所以这里要递归调用HEAPIFY，直到达到最下层节点，这样就维护了堆的性质。HEAPIFY耗时O（lgn）
（2）建堆（BUILD\-HEAPIFY）
从中间那个元素开始到第一个元素，逐一调用HEAPIFY函数，即可完成建堆。
逐一从中间那个元素开始递减而不是从第一个元素递增，这时为了保证每次调用HEAPIFY都能保证该节点的子节点都满足最大（小）堆的性质，否则无法调用HEAPIFY。中间那个元素是第一个可能不满足最大（小）堆性质的节点，所以从这里开始维护（HEAPIFY）。一个建堆的例子如下所示：\[5,3,17,10,84,19,6,22,9\]

![](https://upload-images.jianshu.io/upload_images/2702997-12961ae700b3cb69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/382)

建堆

建堆的期望时间为O（n）

**堆的应用**:
（1）堆排序（详见 [排序算法](https://www.jianshu.com/p/cd1cd85f4b96)）
（2）优先队列

### 108 产生死锁的必要条件 ###

所谓死锁<DeadLock>: 是指两个或两个以上的进程在执行过程中,因争夺资源而造成的一种 互相等待的现象,若无外力作用,它们都将无法推进下去.此时称系统处于死锁状态或系统产生了死锁,这些永远在互相等待的进程称为死锁进程.由于资源占用是互斥的，当某个进程提出申请资源后，使得有关进程在无外力协助下，永远分配不到必需的资源而无法继续运行，这就产生了一种特殊现象死锁。

### 109 产生死锁的原因 ###

(1) 因为系统资源不足。(2) 进程运行推进的顺序不合适。(3) 资源分配不当等。如果系统资源充足，进程的资源请求都能够得到满足，死锁出现的可能性就很低，否则就会因争夺有限的资源而陷入死锁。其次，进程运行推进顺序与速度不同，也可能产生死锁。

### 110 数据库事务隔离级别 ###

数据库事务的隔离级别有 4 个，由低到高依次为 Read uncommitted、Read committed、Repeatable read、Serializable，这四个级别可以逐个解决脏读、 不可重复读、幻读这几类问题。


| 事务隔离级别 | 脏读 | 不可重复读 | 幻读 |
| --- | --- | ---  | --- |
| Read uncommitted | √| √ |√ |
| Read committed | ×| √ |√ |
| Repeatable read | × | × |√ |
| Serializable | × | × |× |

当隔离级别设置为 Read uncommitted 时，就可能出现脏读，如何避免脏读， 请看下一个隔离级别。

当隔离级别设置为 Read committed 时，避免了脏读，但是可能会造成不可重 复读。   Sql Server , Oracle

虽然 Repeatable read 避免了不可重复读，但还有可能出现幻读。  mysql

Serializable 是最高的事务隔离级别，同时代价也花费最高，性能很低，一般很 少使用，在该级别下，事务顺序执行，不仅可以避免脏读、不可重复读，还避 免了幻像读。

### 111 乐观锁和悲观锁 ###

在关系数据库管理系统里，悲观并发控制(又名“悲观锁”，Pessimistic Concurrency Control，缩写“PCC”)是一种并发控制的方法。它可以阻止 一个事务以影响其他用户的方式来修改数据。

**优点与不足**

悲观并发控制实际上是“先取锁再访问”的保守策略，为数据处理的安全提供
了保证。但是在效率方面，处理加锁的机制会让数据库产生额外的开销，还有
增加产生死锁的机会;另外，在只读型事务处理中由于不会产生冲突，也没必要使用锁，这样做只能增加系统负载;还有会降低了并行性，一个事务如果锁
定了某行数据，其他事务就必须等待该事务处理完才可以处理那行数据。

乐观并发控制(又名“乐观锁”，Optimistic Concurrency Control，缩写“OCC”)是一种并发控制的方法。它假设多用 户并发的事务在处理时不会彼此互相影响，各事务能够在不产生锁的情况下处理各自影响的那部分数据。在提交数据更新之前，每个事务会先检查在该事务 读取数据后，有没有其他事务又修改了该数据。如果其他事务有更新的话，正 在提交的事务会进行回滚。

**优点与不足**

乐观并发控制相信事务之间的数据竞争(data race)的概率是比较小的，因此尽 可能直接做下去，直到提交的时候才去锁定，所以不会产生任何锁和死锁。但 如果直接简单这么做，还是有可能会遇到不可预期的结果，例如两个事务都读 取了数据库的某一行，经过修改以后写回数据库，这时就遇到了问题。


### 112 MySQL InnoDB 中使用悲观锁 ###

使用 select...for update 会把数据给锁住，不过我们需要注意 一些锁的级别，MySQL InnoDB 默认行级锁。行级锁都是基于索引的，如果一 条 SQL 语句用不到索引是不会使用行级锁的，会使用表级锁把整张表锁住，这 点需要注意。

### 113 SQL 注入的原理，如何预防 ###

SQL 注入是目前比较常见的针对数据库的一种攻击方式。在这种攻击方式中，攻击者 会将一些恶意代码插入到字符串中。

SQL 注入式攻击的主要形式有两种。

* 一是直接将代码插入到与 SQL 命令串联在一起并使得其以执行的用户输入变量。上面笔者举的例子就是采用了这种方法。由于其直接与 SQL 语句捆绑，故也被称为直接注入式攻击法。
* 二是一种间接的攻击方法，它将恶意代码 注入要在表中存储或者作为原书据存储的字符串。在存储的字符串中会连接到一个动态的 SQL 命令中，以执行一些恶意的 SQL 代码。

### 114  SQL 注入式攻击的防治 ###

* 普通用户与系统管理员用户的权限要有严格的区分
* 强迫使用参数化语句
* 加强对用户输入的验证
* 多层环境如何防治 SQL 注入式攻击
* 必要的情况下使用专业的漏洞扫描工具来寻找可能被攻击的点

### 115 数据库索引的实现(B+树介绍、和 B 树、R 树区别) ###

数据库索引，是数据库管理系统中一个排序的数据结构，以协助快速查询、更新数据库表中数据。索引的实现通常使用 B 树及其变种 B+树。

在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方 式引用(指向)数据，这样就可以在这些数据结构上实现高级查找算法。这种数据结构，就是索引。 为表设置索引要付出代价的:一是增加了数据库的存储空间，二是在插入和修改数据时要 花费较多的时间(因为索引也要随之变动)。

为表设置索引要付出代价的:一是增加了数据库的存储空间，二是在插入和修改数据时要 花费较多的时间(因为索引也要随之变动)。

### 116 数据库三种索引 ###

根据数据库的功能，可以在数据库设计器中创建三种索引:唯一索引、主键索引和聚集索 引。

**唯一索引**
唯一索引是不允许其中任何两行具有相同索引值的索引。

**主键索引**

数据库表经常有一列或列组合，其值唯一标识表中的每一行。该列称为表的主键。

**聚集索引**

在聚集索引中，表中行的物理顺序与键值的逻辑(索引)顺序相同。一个表只能包含一个聚集索引。

### 117 局部性原理与磁盘预读 ###

局部性原理:当一个数据被用到时，其附近的数据也通常会马上被使用。程 序运行期间所需要的数据通常比较集中。

### 118 B-/+Tree 索引的性能分析 ###

每次新建节点时，直接申请一个页的空间，这样就保证一个节点物理上也存储在一个页 里，加之计算机存储分配都是按页对齐的，就实现了一个 node 只需一次 I/O。
B-Tree 中一次检索最多需要 h-1 次 I/O(根节点常驻内存)，渐进复杂度为 O(h)=O(logdN)。一般实际应用中，出度 d 是非常大的数字，通常超过 100，因此 h 非常 小(通常不超过 3)。

### 119  B-树和 B+树数据结构 ###

**B 树**

B 树中每个节点包含了键值和键值对于的数据对象存放地址指针，所以成功搜索一个对象可以不用到达树的叶节点。
成功搜索包括节点内搜索和沿某一路径的搜索，成功搜索时间取决于关键码所在的层次以 及节点内关键码的数量。
在 B 树中查找给定关键字的方法是:首先把根结点取来，在根结点所包含的关键字 K1,...,kj 查 找给定的关键字(可用顺序查找或二分查找法)，若找到等于给定值的关键字，则查找成功; 否则，一定可以确定要查的关键字在某个 Ki 或 Ki+1 之间，于是取 Pi 所指的下一层索引节点块 继续查找，直到找到，或指针 Pi 为空时查找失败。

**B+树**

B+树非叶节点中存放的关键码并不指示数据对象的地址指针，非也节点只是索引部分。所 有的叶节点在同一层上，包含了全部关键码和相应数据对象的存放地址指针，且叶节点按 关键码从小到大顺序链接。如果实际数据对象按加入的顺序存储而不是按关键码次数存储 的话，叶节点的索引必须是稠密索引，若实际数据存储按关键码次序存放的话，叶节点索 引时稀疏索引。

B+树有 2 个头指针，一个是树的根节点，一个是最小关键码的叶节点。 所以 B+树有两种搜索方法:一种是按叶节点自己拉起的链表顺序搜索。
一种是从根节点开始搜索，和 B 树类似，不过如果非叶节点的关键码等于给定值，搜索并 不停止，而是继续沿右指针，一直查到叶节点上的关键码。所以无论搜索是否成功，都将 走完树的所有层。
B+ 树中，数据对象的插入和删除仅在叶节点上进行。

这两种处理索引的数据结构的不同之处:
a，B 树中同一键值不会出现多次，并且它有可能出现在叶结点，也有可能出现在非叶结点中。 而 B+树的键一定会出现在叶结点中，并且有可能在非叶结点中也有可能重复出现，以维持 B+ 树的平衡。
b，因为 B 树键位置不定，且在整个树结构中只出现一次，虽然可以节省存储空间，但使得在 插入、删除操作复杂度明显增加。B+树相比来说是一种较好的折中。
c，B 树的查询效率与键在树中的位置有关，最大时间复杂度与 B+树相同(在叶结点的时候)， 最小时间复杂度为 1(在根结点的时候)。而 B+树的时候复杂度对某建成的树是固定的。

### 120 Redis 的数据类型 ###

Redis 的键值可以使用物种数据类型:字符串，散列表，列表，集合，有序集合。

一个字符串允 许存储的最大容量为 512MB

### 121 OSI 七层模型以及 TCP/IP 四层模型 ###

TCP/IP五层协议和OSI的七层协议对应关系如下。

![](https://images2015.cnblogs.com/blog/705728/201604/705728-20160424234825491-384470376.png)

* 物理层:包含了多种与物理介质相关的协议，这些物理介质用以支撑 TCP/IP 通信。
* 数据链路层:包含了控制物理层的协议，是基于数据链路上的流控和差错控制机制。例如: 如何访问和共享介质、怎样标识介质上的设备、数据在介质上发生之前如何完成数据帧等;
* 网络层:主要负责定义数据包的格式和地址形式，为经过逻辑网络路径的数据进行路由选 择;
* 传输层:包含了控制网络层的协议，是基于逻辑链路上的流控和差错控制;

**数据封装过程**

TCP 头:TCP 数据报，包含源端和目的端的端口号，用于寻找发端和收端的应用进程;

IP 头:用于寻找网络中目的主机在逻辑网络中的位置;

LLC 头:负责识别网络层协议，然后对它们进行封装。LLC 报头告诉数据链路层一旦帧被接收 到时，应当对数据包做何处理。它的工作原理是这样的:主机接收到帧并查看其 LLC 报头，以 找到数据包的目的地，比如说，在网络层的 IP 协议。

MAC 头:用于寻找主机在网络设备中的位置;

### 122 TCP/IP 四层模型 ###

**应用层**:应用程序间沟通的层，如简单电子邮件传输(SMTP)、文件传输协议 (FTP)、网络远程访问协议(Telnet)等。
**传输层**:在此层中，它提供了节点间的数据传送服务，如传输控制协议(TCP)、 用户数据报协议(UDP)等，TCP 和 UDP 给数据包加入传输数据并把它传输到下一 层中，这一层负责传送数据，并且确定数据已被送达并接收。

**互连网络层**:负责提供基本的数据封包传送功能，让每一块数据包都能够到达目的 主机(但不检查是否被正确接收)，如网际协议(IP)。

**网络接口层**:对实际的网络媒体的管理，定义如何使用实际网络(如 Ethernet、 Serial Line 等)来传送数据。

### 123 HTTP 和 HTTPS ###

HTTP 是一个属于应用层的面向对象的协议，由于其简捷、快速的方式，适用于分布式超 媒体信息系统。

**HTTP**

*HTTP 协议的主要特点可概括如下:*

* 支持客户/服务器模式。 
* 简单快速:客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有 GET、HEAD、POST。
* 灵活:HTTP 允许传输任意类型的数据对象。正在传输的类型由 Content-Type 加以标 记。 
* 无连接:无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并 收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。
* 无状态:HTTP 协议是无状态协议。

**HTTPS**

HTTPS(Hypertext Transfer Protocol over Secure Socket Layer，基于 SSL 的 HTTP 协议)使用了 HTTP 协议，但 HTTPS 使用不同于 HTTP 协议的默认端口及一个 加密、身份验证层(HTTP 与 TCP 之间)

### 124 HTTP 与 HTTPS 有什么区别 ###

* https 协议需要到 ca 申请证书，一般免费证书较少，因而需要一定费用。 
* http 是超文本传输协议，信息是明文传输，https 则是具有安全性的 ssl 加密传输
协议。* http 和 https 使用的是完全不同的连接方式，用的端口也不一样，前者是 80，后 者是 443。
* http 的连接很简单，是无状态的;HTTPS 协议是由 SSL+HTTP 协议构建的可进行 加密传输、身份认证的网络协议，比 http 协议安全。
* HTTP 使用 TCP 三次握手建立连接，客户端和服务器需要交换 3 个包;HTTPS 除了 TCP 的三 个包，还要加上 ssl 握手需要的 9 个包，所以一共是 12 个包。

### 125 HTTP 请求报文解剖和响应报文解剖 ###

HTTP 请求报文由 3 部分组成(请求行+请求头+请求体)
HTTP 的响应报文也由三部分组成(响应行+响应头+响应体)

### 126 get 提交和 post 提交的区别 ###

GET 用于信息获取，而且应该是安全的和幂等的。GET 方式提交的数据最多只能是 1024 字节(有字节限制)
幂等的意味着对同一 URL 的多个请求应该返回同样的结果。

POST 表示可能修改变服务器上的资源的请求，POST 是没有大小限制的。

### 127 session 和 cookie 的区别 ###

* cookie 数据存放在客户的浏览器上，session 数据放在服务器上。
* cookie 不是很安全，别人可以分析存放在本地的 COOKIE 并进行 COOKIE 欺骗 考虑到安全应当使用 session。
* session 会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能 考虑到减轻服务器性能方面，应当使用 COOKIE。
* 单个 cookie 保存的数据不能超过 4K，很多浏览器都限制一个站点最多保存 20 个cookie。
* 所以个人建议: 将登陆信息等重要信息存放为 SESSION;其他信息如果需要保留，可以放在 COOKIE 中

### 128 redirect 与 forward 区别 ###

**从地址栏显示来说** forward 是服务器请求资源,服务器直接访问目标地址的 URL,把那个 URL 的响应内容读取过来,然后把这些内容再发给浏览器.浏览器根本不知道服务器发 送的内容从哪里来的,所以它的地址栏还是原来的地址.
redirect 是服务端根据逻辑,发送一个状态码,告诉浏览器重新去请求那个 地址.所以地址栏显示的是新的 URL.所以 redirect 等于客户端向服务器端发出两次 request，同时也接受两次 response。

**从数据共享来说** forward:转发页面和转发到的页面可以共享 request 里面的数据. redirect:不能共享数据.

**从运用地方来说** forward:一般用于用户登陆的时候,根据角色转发到相应的模块.redirect:一般用于用户注销登陆时返回主页面和跳转到其它的网站等.

**从效率来说** forward:高.redirect:低.

### 129 TCP 和 UDP 区别 ###

* 基于连接与无连接;* 对系统资源的要求(TCP 较多，UDP 少);* UDP 程序结构较简单;
* 流模式与数据报模式 ;* TCP 保证数据正确性，UDP 可能丢包，TCP 保证数据顺序，UDP 不保证

### 130 DDos 攻击 ###

DoS 是 Denial of Service 的简写就是拒绝服务,而 DDoS 就是 Distributed Denial of Service的简写就是分布式拒绝服务,而 DRDoS 就是 Distributed Reflection Denial of Service 的简写,这是分布反射式拒绝服务的意思。

目前最流行也是最好用的攻击方法就是使用 SYN-Flood 进行攻击,SYN-Flood 也就是 SYN 洪水攻击。SYN-Flood 不会完成 TCP 三次握手的第三步,也就是不发送确认连接的信息给服务器。

### 131 hascode 为什么用31做权重 ###

**原因一：更少的乘积结果冲突**

　　31是质子数中一个“不大不小”的存在，如果你使用的是一个如2的较小质数，那么得出的乘积会在一个很小的范围，很容易造成哈希值的冲突。而如果选择一个100以上的质数，得出的哈希值会超出int的最大范围，这两种都不合适。而如果对超过 50,000 个英文单词（由两个不同版本的 Unix 字典合并而成）进行 hash code 运算，并使用常数 31, 33, 37, 39 和 41 作为乘子，每个常数算出的哈希值冲突数都小于7个（国外大神做的测试），那么这几个数就被作为生成hashCode值得备选乘数了。

**原因二：31可以被JVM优化**

　　JVM里最有效的计算方式就是进行位运算了：

* 左移 << : 左边的最高位丢弃，右边补全0（把 << 左边的数据*2的移动次幂）。
* 右移 >> : 把>>左边的数据/2的移动次幂。
* 无符号右移 >>> : 无论最高位是0还是1，左边补齐0。 　　

  所以 ： 31 * i = (i << 5) - i（左边  31\*2=62,右边2*2^5-2=62） - 两边相等，JVM就可以高效的进行计算啦。。。

　ps:以前家里人老说学号数理化，走遍全天下。貌似很有道理^_^
















      


  




 

 

 

 

 



  
