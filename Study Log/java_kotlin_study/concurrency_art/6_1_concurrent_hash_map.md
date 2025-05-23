---
title: 6.1 ConcurrentHashMap的原理与使用
chapter: "6"
order: "1"
---

## 6.1 ConcurrentHashMap的原理与使用

> [!attention]
> 本节使用[jdk1.7](https://github.com/openjdk/jdk/tree/jdk7-b147)版本。后续要更新接下来的jdk对于ConcurrentHashMap的升级。
> 
> - [ ] #TODO tasktodo1718346387211 升级ConcurrentHashMap。 ➕ 2024-06-14 🔼 

### 6.1.1 为什么用ConcurrentHashMap

#### 6.1.1.1 HashMap在多线程的缺陷

在并发编程中使用 HashMap 可能导致程序死循环。而使用线程安全的 HashTable 效率又非常低下，基于以上两个原因，便有了 ConcurrentHashMap 的登场机会。

在多线程环境下，使用 HashMap 进行 put 操作会引起死循环，导致 CPU 利用率接近100%，所以在并发情况下不能使用 HashMap。例如，执行以下代码会引起死循环：

```java
public static void main(String[] args) throws InterruptedException {
	final HashMap<String, String> map = new HashMap<>(2);
	Thread t = new Thread(new Runnable() {
		@Override
		public void run() {
			for (int i = 0; i < 10000; i++) {
				new Thread(new Runnable() {
					@Override
					public void run() {
						map.put(UUID.randomUUID().toString(), "");
					}
				}, "ftf" + i).start();
			}
		}
	}, "ftf");
	t.start();
	t.join();
}
```

> [!attention]
> 以上代码只有在jdk1.7以前才会出问题：[[Study Log/java_kotlin_study/java_kotlin_study_diary/hash_map#JDK 1.7 中的 HashMap|hash_map]]。这里我特地自己搞了一下。确实使用java8以及以后的版本，就不会有死循环的问题了。
> 
> 这里给出编译和执行的过程：
> 
> ```shell
> # 编译
> /usr/lib/jvm/java-7-j9/bin/javac UnsafehashMap.java
> # 执行
> /usr/lib/jvm/java-7-j9/bin/java UnsafehashMap
> ```
> 
> 注意包名不能写，不然搞的很麻烦。所以这个类就不参与主工程了。

#### 6.1.1.2 HashTable的低效

另外还有HashTable。它可以处理多线程访问的情况，但是效率太低了。我们来看看HashTable的注释是怎么说的：

> Unlike the new collection implementations, `Hashtable` is synchronized.  If a thread-safe implementation is not needed, it is recommended to use `HashMap` in place of `Hashtable`.  If a thread-safe highly-concurrent implementation is desired, then it is recommended to use `ConcurrentHashMap` in place of `Hashtable`.

这段注释强调了2点：

- 如果你根本用不到并发场景，那用HashMap，别用HashTable；
- 如果你需要**高**并发场景，那用ConcurrentHashMap，也别用HashTable。

主要的原因注释里也说了，因为它是synchronized。HashTable的get和set方法都是用synchronized修饰的，这意味着两个线程无论是读还是写，都会竞争同一把锁。甚至两个线程不能同时读，这就有点不太行了。

另外，据我观察，jdk1.7的HashTable和jdk17的HashTable的代码几乎就没啥区别，所以我们也能猜测出来官方本身就已经属于是半放弃这个类了。

#### 6.1.1.3 ConcurrentHashMap的优势

CHM高效就在它的数据不是一把锁干死的，是分段的。CHM里面的数据被分成若干段，每一段用一个锁给锁起来。这样多个线程大概率会访问到不同的段，也就能很大程度上提高并发效率。

### 6.1.2 ConcurrentHashMap的结构和初始化

简单的结构如下：

![[Study Log/java_kotlin_study/concurrency_art/resources/Drawing 2024-06-14 13.58.48.excalidraw.svg]]

主要的数据都存在segments里。里面的每个元素是一个Segment，也就是我们提到过的一段。而这一段我们可以看成一个小小的HashMap（实际上是HashTable），因为这一段里面依然有一个完整的链表数组。数组的每一个entry是一个HashEntry。

CHM构造的时候，无论你怎么调用，最后都会走到统一的构造方法。如果你有一些参数没传，那么就会用默认的。下面是这些参数的说明：

- `initialCapacity`：初始容量，**CHM中所有元素的数量**。这个参数用来创建segments数组中的第一个Segment，也就是和`segment[0]`内部的链表数组的大小有关。用它除以segment的数量（来自下面的`concurrencyLevel`），就能得到每个segment的大小；
- `loadFactor`：这个东西HashMap和HashTable都有，是用来控制扩容的。参考[[Study Log/java_kotlin_study/java_kotlin_study_diary/hash_map|hash_map]]，如果数组大到一定程度，hash碰撞的概率就会增加。所以需要进行扩容，才能进一步减少hash碰撞的概率；
- `concurrencyLevel`：这个东西主要是为了应付并发。它有多大主要看会修改CHM的线程有多少个，也就是并发量。并发量越高，那么这个level也就越大。和`initialCapacity`是对应的，**表示的是segment的数量**。因为segment的数量就代表了锁的数量。锁越多，则能应付的并发量就越大。

下面我们来介绍CHM初始化的时候会初始化的其他东西。

#### 6.1.2.1 segments数组

显然，这里面的segments就是最重要的数据+锁，所以它也是初始化的核心。所以我们来看看它是怎么构造出来的。主要就和刚刚的`concurrencyLevel`参数有关，因为之所以segments是个数组，就是为了多线程访问不同的Segment。而并发量越大，那么segments肯定就要越长，才能容纳这么多线程去访问。

下面是经过精简的，CHM的构造方法的最深层版本：

```java
public ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel) {
	... ...
	if (concurrencyLevel > MAX_SEGMENTS)
		concurrencyLevel = MAX_SEGMENTS;
	// Find power-of-two sizes best matching arguments
	int sshift = 0;
	int ssize = 1;
	while (ssize < concurrencyLevel) {
		++sshift;
		ssize <<= 1;
	}
	this.segmentShift = 32 - sshift;
	this.segmentMask = ssize - 1;
	... ...
	Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
	... ...
	this.segments = ss;
}
```

可以看到，最终的segments的大小是ssize，而这个变量的计算就是依赖于concurrencyLevel。但是我们需要注意的是，ssize并不是每次+1的，而是`<<= 1`。所以，实际上相当于`ssize *= 2`。

我们假设concurrencyLevel是15。那么while循环会走4次，退出循环后ssize为16，正好是**大于等于concurrencyLevel的2的整数次方**。这也就是注释中说的`power-of-two sizes best matching arguments`。如果有15个线程需要访问这个HashMap，那么segments的长度就应该是16。这样既能有足够大的并发量，同时由于正好是2的整数次方，所以也能满足按位与的hash散列算法来定位对应的Segment。

#### 6.1.2.2 每个segment

在书中介绍创建segment的时候，直接是把segments数组中的每个元素都初始化了，就像下面这样：

```java
for (int i = 0; i < this.segments.length; ++i) {
	this.segments[i] = new Segment<K, V>(cap, loadFactor);
}
```

但是[我看的版本](https://github.com/openjdk/jdk/tree/jdk7-b147)里面并没有这段代码，在CHM初始化的时候，仅仅是把`segments[0]`给创建了。也就是说，一开始segments里面只有一个链表数组。而其它的元素是在`ensureSegment`方法中构造出来的。也就是随用随构造。下面我们来分别介绍这两个位置。

首先是`segments[0]`构造的位置，和segments数组是同时构造的：

```java
public ConcurrentHashMap(int initialCapacity, float loadFactor, int concurrencyLevel) {
	... ...
	if (initialCapacity > MAXIMUM_CAPACITY)
		initialCapacity = MAXIMUM_CAPACITY;
	int c = initialCapacity / ssize;
	if (c * ssize < initialCapacity)
		++c;
	int cap = MIN_SEGMENT_TABLE_CAPACITY;
	while (cap < c)
		cap <<= 1;
	// create segments and segments[0]
	Segment<K,V> s0 =
		new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
						 (HashEntry<K,V>[])new HashEntry[cap]);
	Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
	UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
	this.segments = ss;
}
```

刚才我们说过，ssize是segments数组的长度。而initialCapacity就是CHM中所有entry的个数。比如我们一开始想要存100个元素，同时还是有差不多15个线程要访问，那么我们按照上面的代码算一下：

- 根据刚才分析，**ssize应该是16**；
- $100 \div 16 = 6 \cdots 4$，所以c就是6；
- 然后$6 \times 16 \lt 100$，所以会把c再+1变成7（其实这两步就相当于$\lceil \dfrac{initialCapacity}{ssize} \rceil = \lceil \dfrac{100}{16} \rceil = 7$）；
- 让cap的初值是table最小的容量，也就是说链表数组最小的容量，这个值是2（和书上不一样，书上说的是1）；
- 和之前ssize的计算一样，也是取大于等于它的2的整数次方。因此c如果是7的话，**cap就应该是$2^3 = 8$**；
- 最后构造`segments[0]`的时候，还需要进一步限制，这个和HashMap一样，就是用`cap * loadFactor`计算出`threshold`。

- [ ] #TODO tasktodo1739804317735 这里的`threshold`是怎么计算出来的？有什么用？ ➕ 2025-02-17 ⏫ 🆔 pwyv57 

> [!question]- 凭什么链表数组的大小计算要用$\lceil \dfrac{initialCapacity}{ssize} \rceil$？
> 首先我问个问题：对于HashMap，HashTable，当然也包括ConcurrentHashMap。它们的链表数组里面，链表是越长越好还是越短越好？答案是显而易见的：**肯定是越短越好**。因为链表越短，我们访问数据就越快。尤其是CHM这种高性能的组件，更加需要让链表变得短。那么问题来了：链表短，那总得有代价。如果是HashMap的话，链表想要缩短，那就是增加数组的长度，并且用一些hash散列的算法来规避这个问题。比如两个元素的hash值是一样的，本来因为碰撞要放到同一个链表里，但是有了hash散列之后就可以放到相邻或者其他的不同的链表中，这样就拆开了。
> 
> 回到CHM的构造这里，我们传入了initialCapacity，也就是**初始元素的个数**。那么既然我传了，我肯定就是想告诉你：这个CHM里一开始我就至少打算放100个元素。那么你既然想把他放进去，就要有这么多地方才行。~~为了少创建segment，CHM的策略是把初始元素都放到`segments[0]`中。所以一开始它才只创建了`segments[0]`。那么如果想要把100个元素都~~ 我们这么想：如果这100个元素**每一个都能放到一个空的链表里**，那不就是最快的？但是这样链表就太多了，足足一百个。因此要少一点。但是少一点，就会让每个链表里的元素数量变多。那么我们需要选一个最平衡的值。怎么平衡？就是`concurrencyLevel`参数。我们有15个线程要访问，那么差不多就需要15把锁，自然就是要15个segment。此时也自然能算出来每个segment的大小，就是$\dfrac{100}{15} \approx 7$。但是，为了保证这两个值都是2的整数次方，所以调整成了16和8。
> 
> - [ ] #TODO tasktodo1739804936538 这里取整到2的整数次方的目的是什么？ ➕ 2025-02-17 ⏫ 🆔 8p36iz 
> 
> 换句话说，我们当然可以让每个Segment中table的size是1，2，5，8或者任何随机的值。但是如果你选大了，那就浪费空间，如果选小了，链表就会变长，效率就会下降。所以这个公式就是为了**选一个最合适的size，考虑到当前并发量的情况下**。


至于ensureSegment方法中创建其它segment的逻辑，没什么好说的。就是不存在的话，就用`segments[0]`的参数创建一个新的然后放到数组里就行了。这里把代码贴一下：

```java
/**
 * Returns the segment for the given index, creating it and
 * recording in segment table (via CAS) if not already present.
 *
 * @param k the index
 * @return the segment
 */
@SuppressWarnings("unchecked")
private Segment<K,V> ensureSegment(int k) {
	final Segment<K,V>[] ss = this.segments;
	long u = (k << SSHIFT) + SBASE; // raw offset
	Segment<K,V> seg;
	if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
		Segment<K,V> proto = ss[0]; // use segment 0 as prototype
		int cap = proto.table.length;
		float lf = proto.loadFactor;
		int threshold = (int)(cap * lf);
		HashEntry<K,V>[] tab = (HashEntry<K,V>[])new HashEntry[cap];
		if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
			== null) { // recheck
			Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
			while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
				   == null) {
				if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
					break;
			}
		}
	}
	return seg;
}
```

下面来个图的总结：

![[Study Log/java_kotlin_study/concurrency_art/resources/Drawing 2024-06-15 18.47.42.excalidraw.svg]]

### 6.1.3 定位Segment

用hash散列算法给元素进行再次分散。这么做的主要目的就是为了让元素尽可能分散在不同的segment中。如果数据都集中在同一个segment中，存取速度会非常慢，而且本身的分段锁也失去了意义。

hash散列算法本身是一种映射的算法，比如md5就是其中一种。最主要的意义是，生成的hash code是**固定长度**的。所以用位操作去操作它的话会非常方便。

CHM使用的hash散列算法是`single-word Wang/Jenkins hash`的变种，代码如下：

```java
/**
 * Applies a supplemental hash function to a given hashCode, which
 * defends against poor quality hash functions.  This is critical
 * because ConcurrentHashMap uses power-of-two length hash tables,
 * that otherwise encounter collisions for hashCodes that do not
 * differ in lower or upper bits.
 */
private static int hash(int h) {
	// Spread bits to regularize both segment and index locations,
	// using variant of single-word Wang/Jenkins hash.
	h += (h <<  15) ^ 0xffffcd7d;
	h ^= (h >>> 10);
	h += (h <<   3);
	h ^= (h >>>  6);
	h += (h <<   2) + (h << 14);
	return h ^ (h >>> 16);
}
```

这东西一般是这么用的：

```java
int h = hash(key.hashCode())
```

就是已经有了hashcode，然后用这个算法进行再散列。因为hashcode本身就非常容易冲突，所以我们用这个算法能够让原本冲突的code再次分散。

### 6.1.4 ConcurrentHashMap的操作

#### 6.1.4.1 get

CHM的get操作非常高效，因为它不用加锁。代码如下：

```java
public V get(Object key) {
	Segment<K,V> s; // manually integrate access methods to reduce overhead
	HashEntry<K,V>[] tab;
	int h = hash(key.hashCode());
	long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
	if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
		(tab = s.table) != null) {
		for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
				 (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
			 e != null; e = e.next) {
			K k;
			if ((k = e.key) == key || (e.hash == h && key.equals(k)))
				return e.value;
		}
	}
	return null;
}
```

用散列算法定位到对应的segment，然后再用散列算法定位segment里面table里面的链表。最后才能在链表里找到对应的entry。这里涉及了两个散列算法，定位segment的就是u的计算；定位HashEntry的就是for循环里面的`((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE`。

另外，这里没加锁，但是使用的是volatile读。~~也是为了和put中的volatile写打配合来保证可见性。~~

> [!question] segments还有table已经是volatile的了，那为什么还要用getObjectVolatile？
> 看这里：[java - concurrentHashMap has a volatile table , why need unsafe.getObjectVolatile() when get() - Stack Overflow](https://stackoverflow.com/questions/59908363/concurrenthashmap-has-a-volatile-table-why-need-unsafe-getobjectvolatile-whe)按他的回答，主要有两点：
> 
> 1. 我们访问segment中的链表的时候，通过的是`s.table`。它是volatile的。但是，我们将它赋值给了`tab`变量。而tab这个方法内部的引用不是volatile的，所以访问的时候要用Unsafe；
> 2. `s.table`是volatile的，但是只能保证`table`这个变量本身的读写是volatile的。而对于CHM来说，get和set读写的不是`s.table`这个变量本身，而是它指向的内存（注意table是一个数组）。实际上java就不支持volatile数组。所以只能用Unsafe提供的方法。
> 
> - [x] #TODO tasktodo1718639680872 进一步补充问题：segments还有table已经是volatile的了，那为什么还要用getObjectVolatile？ 🆔 50m036 🔼 ➕ 2024-06-17 ✅ 2025-02-17

#### 6.1.4.2 put

put操作的步骤：

1. 定位Segment；
2. 判断Segment是否需要扩容；
3. 扩容后将元素插入到Segment中。

可以看到，和HashMap的策略是相反的。拿出HashMap的扩容逻辑做对比：

![[Study Log/java_kotlin_study/java_kotlin_study_diary/resources/Pasted image 20240613133227.png]]

HashMap是插入元素之后才进行扩容的。因此如果扩容之后没有元素再插入，这次扩容就白做了。而CHM是看空间不够先扩容，然后才插入元素。

还有一点就是上锁。CHM的锁是什么？提到过，其实就是Segment本身。我们可以看到Segment本身是继承自ReentrantLock的。

当定位到Segment之后，在里面操作table，扩容，插入的这些逻辑就需要上锁了。总体的put逻辑如下：

![[Study Log/java_kotlin_study/concurrency_art/resources/Pasted image 20240618144711.png]]

#### 6.1.4.3 size

我们自己想一下，CHM的总体大小应该怎么算：其实就是把每个Segment中所有的HashEntry的个数都加起来。所以应该是一个循环，遍历所有的Segment，累加它们HashEntry的个数就可以了。

然而，这是在多线程的场景下，所以，当你加到第3个segment的时候，可能第二个segment就被人给改了。因此我们不一定能获得对的值。

那我们这个时候可能会想到：算size之前，把所有的segment都锁上，然后我再算，这样不就可以了？是可以，但是太慢了。因为你算个size，其它所有线程都不能访问任何Segment，多少有点不讲理。

那怎么办？CHM的做法是：**赌**。我就赌我在算size的时候，没有其它线程修改。只要我赌对了，我就赚了。下面代码就是不加锁，直接算：

```java
public int size() {
	final Segment<K,V>[] segments = this.segments;
	int size;
	boolean overflow; // true if size overflows 32 bits
	size = 0;
	overflow = false;
	for (int j = 0; j < segments.length; ++j) {
		Segment<K,V> seg = segmentAt(segments, j);
		if (seg != null) {
			int c = seg.count;
			if (c < 0 || (size += c) < 0)
				overflow = true;
		}
	}
	return overflow ? Integer.MAX_VALUE : size;
}
```

这代码还是很好看懂的。就一个遍历和累加，再算一下是否超过int的限制。

显然，这段代码肯定是错的，因为我们是多线程。但是又不是完全错的，因为我只要赌对了，这段代码的执行结果就是正确的。那么，**我必须要有一个策略，能够判断我赌没赌对**。

这个策略就是[[Study Log/java_kotlin_study/java_kotlin_study_diary/2024-05-05-java-kotlin-study#Fail-fast And Fail-safe Iterators|modCount]]。它是每一个Segment的成员，标识着这个Segment被操作的次数。像put，remove，clear，replace等操作都会修改它，表明这个Segment被一个线程给修改过了。

什么叫赌失败了？就是我遍历所有的Segment两次，算出这两次的总modCount。如果两次的结果是一样的，那就表明至少我第二次遍历和第一次遍历之间没有其他线程干扰。也就是说，第二次遍历的结果是正确的。因此可以直接返回第二次遍历的结果。

而如果我没赌对，那就不行了。这个时候我有两种选择：再赌一次，或者老老实实加锁。对于CHM来说，这个是通过重试的次数决定的。当重试了两次（总共进行三次遍历）之后还是不行的话，就会老老实实加锁访问了。这里重试两次是通过常量`RETRIES_BEFORE_LOCK`控制。完整代码如下：

```java
public int size() {
	// Try a few times to get accurate count. On failure due to
	// continuous async changes in table, resort to locking.
	final Segment<K,V>[] segments = this.segments;
	int size;
	boolean overflow; // true if size overflows 32 bits
	long sum;         // sum of modCounts
	long last = 0L;   // previous sum
	int retries = -1; // first iteration isn't retry
	try {
		for (;;) {
			if (retries++ == RETRIES_BEFORE_LOCK) {
				for (int j = 0; j < segments.length; ++j)
					ensureSegment(j).lock(); // force creation
			}
			sum = 0L;
			size = 0;
			overflow = false;
			for (int j = 0; j < segments.length; ++j) {
				Segment<K,V> seg = segmentAt(segments, j);
				if (seg != null) {
					sum += seg.modCount;
					int c = seg.count;
					if (c < 0 || (size += c) < 0)
						overflow = true;
				}
			}
			if (sum == last)
				break;
			last = sum;
		}
	} finally {
		if (retries > RETRIES_BEFORE_LOCK) {
			for (int j = 0; j < segments.length; ++j)
				segmentAt(segments, j).unlock();
		}
	}
	return overflow ? Integer.MAX_VALUE : size;
}
```

### 6.1.5 新的ConcurrentHashMap

- [Java Concurrent HashMap Improvements over the years | by Vikas Taank | Medium](https://medium.com/@vikas.taank_40391/java-concurrent-hashmap-improvements-over-the-years-8d8b7be6ce37)
- [java - 深入解析 ConcurrentHashMap 实现内幕，吊打面试官，没问题 - 个人文章 - SegmentFault 思否](https://segmentfault.com/a/1190000021237438#item-2)

- [ ] #TODO tasktodo1719047249079 java8 ConcurrentHashMap ➕ 2024-06-22 ⏫ 🆔 51w1tb





---

```dataviewjs
const pages = dv.pages('"Study Log/java_kotlin_study/concurrency_art"')
let nextChapterHead = undefined
let res = undefined
const current = dv.current()
for (let page of pages) {
	if (page.chapter_root == true && page.order == Number(current.chapter) + 1) {
		console.log("found next head: " + page.name)
		nextChapterHead = page
		continue
	}
	if (page.chapter == undefined || page.chapter != current.chapter) {
		console.log("not current chapter: " + page.file.name)
		continue
	}
	if (page.order == Number(current.order) + 1) {
		res = page
	}
}
console.log("res: " + res)
console.log("next: " + nextChapterHead)
if (res == undefined) {
	res = nextChapterHead
}
let text = ""
if (res != undefined) {
	const path = res.file.path
	const title = res.title
	const decoLink = "[[" + path + "|" + title + "]]"
	text = "Next Article: " + decoLink
} else {
	text = "旅途的终点！"
}
dv.el("p", text, { attr: { align: "right" } })
```