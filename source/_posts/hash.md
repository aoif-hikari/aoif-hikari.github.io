---
tags: 数据结构
mathjax: true 
title: 数据结构/Hash
---

> Hash，一般译做散列或音译为哈希，是把任意长度的输入通过散列算法变换成固定长度的输出，该输出就是散列值，是一种压缩映射。

<!--more-->

## 哈希函数

哈希的过程中需要使用哈希函数进行计算。
哈希函数是一种**映射关系**，根据数据的关键词$ key $，通过一定的函数关系，计算出该元素存储位置的函数。
表示为：$address = H [key]$
哈希函数属性：

1. 哈希函数接收任意长度的输入。哈希函数拥有较为庞大的输入值域，接受长度非常长的输入值。实际实现中会指明一个具体可接收的阈值，例如SHA-2最高接受$\frac{2^{64}-1}{8}$长度的字节字符串。
2. 产生固定长度的输出值。
3. 不可逆性，已知哈希函数fn与x的哈希值无法反向求出x。这里的不可逆是指计算上行不通，正着算很好算，反着算在当前的计算机计算能力条件下做不到。
4. 对于特定的哈希函数，只要输入值不变，输出的值也是唯一不变的。
5. 哈希函数的计算时间不应过长，即通过输入值求出输出值的时间不宜过长。
6. 无冲突性，即对于输入值x与哈希函数$f_n$，无法求出一个值$y$，使得$x$与$y$的哈希值相等。但是由于哈希函数实际上代表着一种映射（对应关系），将一个大区间上的数值映射到一个小区间上，它一定是有冲突的，这里的无冲突同样是指“求冲突在计算上行不通”，正向地计算很容易，但是反向的计算在当前的计算机能力条件下做不到。
7. 即使修改了输入值的一个比特位，也会使得输出值发生巨大的变化。
8. 哈希函数产生的映射应当保持均匀，即不要使得映射结果堆积在小区间的某一块区域。

常见的哈希函数构造方法：

### 直接定址法 

- 取关键字或关键字的某个线性函数值为散列地址。
- 即 $H(key) = key$ 或$ H(key) = a*key + b$，其中a和b为常数。

### 除留余数法 

- 取关键字被某个不大于散列表长度$ m$ 的数$ p$ 求余，得到的作为散列地址。
- 即 $H(key) = key\ \%\ p$，$ p \leq m$或$H(key) = key\ mod\ p$，$ p \leq m$

### 数字分析法 

- 当关键字的位数大于地址的位数，对关键字的各位分布进行分析，选出分布均匀的任意几位作为散列地址。
- 仅适用于所有关键字都已知的情况下，根据实际应用确定要选取的部分，尽量避免发生冲突。

### 平方取中法 

- 先计算出关键字值的平方，然后取平方值中间几位作为散列地址。
- 随机分布的关键字，得到的散列地址也是随机分布的。

### 折叠法（叠加法） 

- 将关键字分为位数相同的几部分，然后取这几部分的叠加和（舍去进位）作为散列地址。
- 用于关键字位数较多，并且关键字中每一位上数字分布大致均匀。 

### 随机数法 

- 选择一个随机函数，把关键字的随机函数值作为它的哈希值。
- 通常当关键字的长度不等时用这种方法。 

构造哈希函数的方法很多，实际工作中要根据不同的情况选择合适的方法，总的原则是尽可能少的产生冲突。

通常考虑的因素有**关键字的长度**和**分布情况**、**哈希值的范围**等。

如：当关键字是整数类型时就可以用除留余数法；如果关键字是小数类型，选择随机数法会比较好。

## 哈希冲突的解决

选用哈希函数计算哈希值时，可能不同的$ key $会得到相同的结果，一个地址如何存放多个数据？这就是冲突。

常用的主要有两种方法解决冲突：

### 链接法（拉链法）

拉链法解决冲突的做法是： 将所有关键字为同一哈希值的结点链接在同一个单链表中。

若选定的散列表长度为 $m$，则可将散列表定义为一个由 $m$ 个头指针组成的指针数组 $T[0..m-1] $。

凡是散列地址为 $i$ 的结点，均插入到以$ T[i] $为头指针的单链表中。 
$T $中各分量的初值均应为空指针。

在拉链法中，装填因子$ α $可以大于 1，但一般均取 $α ≤ 1$。

### 开放定址法

用开放定址法解决冲突的做法是：

> 用开放定址法解决冲突的做法是：当冲突发生时，使用某种探测技术在散列表中形成一个探测序列。沿此序列逐个单元地查找，直到找到给定的关键字，或者碰到一个开放的地址（即该地址单元为空）为止（若要插入，在探查到开放的地址，则可将待插入的新结点存人该地址单元）。查找时探测到开放的地址则表明表中无待查的关键字，即查找失败。

简单的说：当冲突发生时，使用某种探查(亦称探测)技术在散列表中寻找下一个空的散列地址，只要散列表足够大，空的散列地址总能找到。

按照形成探查序列的方法不同，可将开放定址法区分为线性探查法、二次探查法、双重散列法等。

#### 线性探查法

$h_i=(h(key)+i) ％ m $，$0 ≤ i ≤ m-1 $

基本思想是： 
探查时从地址 $d$ 开始，首先探查 $T[d]$，然后依次探查$ T[d+1]，…，$直到$ T[m-1]$，此后又循环到 $T[0]，T[1]，…，$直到探查到 **有空余地址** 或者到 $T[d-1]$为止。

#### 二次探查法

$h_i=(h(key)+i*i) ％ m$，$0 ≤ i ≤ m-1 $

基本思想是： 
探查时从地址$ d$ 开始，首先探查 $T[d]$，然后依次探查$ T[d+1^2]，T[d+2^2]，T[d+3^2],…，$等，直到探查到 **有空余地址** 或者到 $T[d-1]$为止。

缺点是无法探查到整个散列空间。

#### 双重散列法

$h_i=(h(key)+i*h_1(key)) ％ m$，$0 ≤ i ≤ m-1 $

基本思想是： 
探查时从地址$ d $开始，首先探查$ T[d]$，然后依次探查$ T[d+h1(d)], T[d + 2*h1(d)]，…，$等。
该方法使用了两个散列函数$ h(key)$ 和 $h_1(key)$，故也称为双散列函数探查法。

定义$ h_1(key)$ 的方法较多，但无论采用什么方法定义，都必须使$ h_1(key) $的值和$ m $互素，才能使发生冲突的同义词地址均匀地分布在整个表中，否则可能造成同义词地址的循环计算。
该方法是开放定址法中最好的方法之一。

## 应用

### 哈希表

哈希表（hash table）是哈希函数最主要的应用。哈希表是实现关联数组（associative array）的一种数据结构，广泛应用于实现数据的快速查找。

用哈希函数计算关键字的哈希值（hash value）,通过哈希值这个索引就可以找到关键字的存储位置，即桶（bucket）。哈希表不同于二叉树、栈、序列的数据结构一般情况下，在哈希表上的插入、查找、删除等操作的时间复杂度是$ O(1)$。

查找过程中，关键字的比较次数，取决于产生冲突的多少，产生的冲突少，查找效率就高，产生的冲突多，查找效率就低。因此，影响产生冲突多少的因素，也就是影响查找效率的因素。 
影响产生冲突多少有以下三个因素：

1. 哈希函数是否均匀；
2. 处理冲突的方法；
3. 哈希表的加载因子。（加载因子 = 填入表中记录个数/散列长度）

哈希表的加载因子和容量决定了在什么时候桶数（存储位置）不够，需要重新哈希。加载因子太大的话桶太多，遍历时效率变低；太小的话频繁 rehash，导致性能降低。所以加载因子的大小需要结合时间和空间效率考虑。

#### hashmap

> 简单来说，**HashMap由数组+链表组成的**，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的。
> 如果定位到的数组位置不含链表（当前entry的next指向null）,那么查找，添加等操作很快，仅需一次寻址即可
> 如果定位到的数组包含链表，对于添加操作，其时间复杂度为$O(n)$，首先遍历链表，存在即覆盖，否则新增；对于查找操作来讲，仍需遍历链表，然后通过key对象的equals方法逐一比对查找。
> 所以，性能考虑，**HashMap中的链表出现越少，性能才会越好。**

HashMap的主干是一个Entry数组。Entry是HashMap的基本组成单元，每一个Entry包含一个key-value键值对。

<img src="/img/hash/hashmap.png"/>

```java
//HashMap的主干数组，可以看到就是一个Entry数组，初始值为空数组{}，主干数组的长度一定是2的次幂。
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;

//Entry是HashMap中的一个静态内部类
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;//存储指向下一个Entry的引用，单链表结构
    int hash;//对key的hashcode值进行hash运算后得到的值，存储在Entry，避免重复计算
	
    //Creates new entry.
	Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    } 
```

```java
//重要字段
//实际存储的key-value键值对的个数
transient int size;

/**阈值，当table == {}时，该值为初始容量（初始容量默认为16）；当table被填充了，也就是为table分配内存空间后，threshold一般为 capacity*loadFactory。HashMap在进行扩容时需要参考threshold */
int threshold;

/**负载因子，代表了table的填充度，默认是0.75
为了减缓哈希冲突，如果初始桶为16，等到满16个元素才扩容，某些桶里可能就有不止一个元素了。所以加载因子默认为0.75，也就是说大小为16的HashMap，到了第13个元素，就会扩容成32。
*/
final float loadFactor;

/**HashMap被改变的次数，由于HashMap非线程安全，在对HashMap进行迭代时，
如果期间其他线程的参与导致HashMap的结构发生变化了（比如put，remove等操作），
需要抛出异常ConcurrentModificationException*/
transient int modCount;
```

```java
/*
HashMap有4个构造器，其他构造器如果用户没有传入initialCapacity 和loadFactor这两个参数，会使用默认值
initialCapacity默认为16，loadFactory默认为0.75
在常规构造器中，没有为数组table分配内存空间（有一个入参为指定Map的构造器例外），而是在执行put操作的时候才真正构建table数组
*/
public HashMap(int initialCapacity, float loadFactor) {
　　　　　//此处对传入的初始容量进行校验，最大不能超过MAXIMUM_CAPACITY = 1<<30(2^30)
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);

        this.loadFactor = loadFactor;
        threshold = initialCapacity;
　　　　　
        init();//init方法在HashMap中没有实际实现，不过在其子类如 linkedHashMap中就会有对应实现
    }

```

```java
public V put(K key, V value) {
        //如果table数组为空数组{}，进行数组填充（为table分配实际内存空间），入参为threshold，
        //此时threshold为initialCapacity 默认是1<<4(2^4=16)
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
       //如果key为null，存储位置为table[0]或table[0]的冲突链上
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);//对key的hashcode进一步计算，确保散列均匀
        int i = indexFor(hash, table.length);//获取在table中的实际位置
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        //如果该对应数据已存在，执行覆盖操作。用新value替换旧value，并返回旧value
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }
        modCount++;//保证并发访问时，若HashMap内部结构发生变化，快速响应失败
        addEntry(hash, key, value, i);//新增一个entry
        return null;
    }

```

inflateTable这个方法用于为主干数组table在内存中分配存储空间，通过roundUpToPowerOf2(toSize)可以确保capacity为大于或等于toSize的最接近toSize的二次幂，

比如toSize=13,则capacity=16;to_size=16,capacity=16;to_size=17,capacity=32.

```java
private void inflateTable(int toSize) {
    int capacity = roundUpToPowerOf2(toSize);//capacity一定是2的次幂
    /**此处为threshold赋值，取capacity*loadFactor和MAXIMUM_CAPACITY+1的最小值，
        capaticy一定不会超过MAXIMUM_CAPACITY，除非loadFactor大于1 */
    threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);
    table = new Entry[capacity];
    initHashSeedAsNeeded(capacity);
    }

//当发生哈希冲突并且size大于阈值的时候，需要进行数组扩容。
//扩容时，需要新建一个长度为之前数组2倍的新的数组，然后将当前的Entry数组中的元素全部传输过去
void addEntry(int hash, K key, V value, int bucketIndex) {
    if ((size >= threshold) && (null != table[bucketIndex])) {
        resize(2 * table.length);//当size超过临界阈值threshold，并且即将发生哈希冲突时进行扩容
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }

    createEntry(hash, key, value, bucketIndex);
}
```

最终存储位置的确定流程：

<img src="/img/hash/2.png"/>



```java
//对key的hashcode进一步进行计算以及二进制位的调整等来保证最终获取的存储位置尽量分布均匀
final int hash(Object k) {
    int h = hashSeed;
    if (0 != h && k instanceof String) return sun.misc.Hashing.stringHash32((String) k);
    h ^= k.hashCode();
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}

//返回数组下标
//h&（length-1）保证获取的index一定在数组范围内，如默认容量16，length-1=15，h=18,转换成二进制计算为index=2。位运算对计算机来说，位运算性能更高一些
static int indexFor(int h, int length) {
    return h & (length-1);
}
```

#### 为何HashMap的数组长度一定是2的次幂

HashMap的数组长度一定保持2的次幂，保证length-1低位全为1，而扩容后只有一位差异，也就是多出了最左位的1，这样在通过 h&(length-1)的时候，只要h对应的最左边的那一个差异位为0，就能保证得到的新的数组索引和老数组索引一致(大大减少了之前已经散列良好的老数组的数据位置重新调换)

如果不是2的次幂，也就是低位不是全为1此时，h的低位部分不再具有唯一性了，哈希冲突的几率会变的更大。

#### hashmap与hashtable区别

|                      | hashmap                                               | hashtable                               |
| :------------------: | :---------------------------------------------------- | :-------------------------------------- |
| **初始化和扩容数据** | 初始容量为 16，扩容大小为2n                           | 初始容量为 11，扩容大小为2n+1           |
|     **线程安全**     | 线程不安全，方法不是Synchronize的要提供外同步，效率高 | 线程安全，方法是是Synchronize的，效率低 |
|     **遍历方式**     | 通过实现 Iterator接口实现                             | 通过实现 Enumeration, Iterator 来遍历   |
|  **对于null的处理**  | 允许有null的键和值，当key为null的时候，会放在第0位    | 不允许有null的键和值                    |
|   **contains方法**   | containsValue和containsKey                            | contains                                |
|  **计算hash值方式**  | (h = key.hashCode()) ^ (h >>> 16)                     | hashCode()                              |
| **解决hash冲突方式** | 链表+红黑树                                           | 链表                                    |

#### JDK1.8中HashMap的性能优化

假如一个数组槽位上链上数据过多（即拉链过长的情况）导致性能下降该怎么办？

JDK1.8在JDK1.7的基础上针对增加了红黑树来进行优化。即当链表超过8时，链表就转换为红黑树，利用红黑树快速增删改查的特点提高HashMap的性能，其中会用到红黑树的插入、删除、查找等算法。
**HashMap put方法逻辑图（JDK1.8）**

<img src="/img/hash/put.png"/>

### RSA

#### 对称加密与非对称加密

对称加密：加密和解密的秘钥使用的是同一个，发送方和接收方事先商量好

缺点：每对用户每次使用对称加密算法时，都需要使用其他人不知道的唯一秘钥，这会使得收、发双方所拥有的钥匙数量巨大，密钥管理成为双方的负担；密钥必须通过见面协商，而无法直接通过网络进行交换。

<img src="/img/hash/duichen.jpg"/>

非对称加密（公钥加密）：需要两个密钥：公开密钥（publickey）和私有密钥（privatekey）。由于公钥是对所有人公开的，所以需要保证数据被公钥加密后无法反推，即单向计算容易，逆向反推困难。模运算即是这样的单向函数。

<img src="/img/hash/feiduichen.jpg"/>

> 模运算，例如，已知$3^3mod\ 7 = 6$，反之已知$3^xmod\ 7 = 6$，求$x$很困难，只能一个数一个数带入尝试。而对于大数来说更不现实。

#### RSA的加密解密过程

RSA属于非对称加密算法。

原始数据$m（message）$，加密密钥$e（encrypt）$，密文$c（cipher）$，解密密钥$d（decrypt）$

<img src="/img/hash/rsa.jpg" style="zoom:50%;" />

变换后得

<img src="/img/hash/ed.jpg" style="zoom: 33%;" />

如何选$e，d$是关键。

> **互质**，又称互素。若$N$个整数的最大公因子是1，则称这$N$个整数互质。
> **欧拉函数**，用于计算在小于等于$n$的正整数之中，有多少个数与$n$构成互质关系，以$φ(n)$表示
> 计算8的欧拉函数，和8互质的 **1**、**3**、**5**、**7**，$φ(8) = 4$
> - 如果$n$是质数的某一个次方，即$ n = p^k $($p$为质数，$k$为大于等于1的整数)，则$φ(n) = φ(p^k) = p^k - p^(k-1)$。比如$φ(8) = φ(2^3) =2^3 - 2^2 = 8 -4 = 4$
> - 如果$n$是质数，则 $φ(n)=n-1$。因为质数与小于它的每一个数，都构成互质关系。比如5与1、2、3、4都构成互质关系。
> - 如果$n$可以分解成两个互质的整数之积，即 $n = p * k $，则$φ(n) = φ(p * k) = φ(p)*φ(k)$。比如$φ(56) = φ(8) * φ(7) = 4 * 6 = 24$
> - 求$φ(n)$很困难，只能通过质因数分解。对大数求$φ(n)$计算上不可行。
> **欧拉定理**，如果两个正整数m和n互质，那么m的φ(n)次方减去1，可以被n整除。$m^{φ(n)}mod\ n\equiv 1$

对欧拉定理变形，$m^{φ(n)}\equiv 1 (mod\ n)$

$m^{kφ(n)}\equiv 1 (mod\ n)$

$m^{kφ(n)+1}\equiv m (mod\ n)$

$m^{kφ(n)+1}mod\ n\equiv m $

可得，$ed = kφ(n)+1$，$d = \frac{kφ(n)+1}{e}$。通过选取$k，n，e$，来计算解密密钥$d$

RSA算法具体的加密过程如下：

假设Alice想通过一个不可靠的媒体接受Bob的一条私人消息，他可以用下面的方式产生一个公钥和私钥。

- 随意选择两个大的质数$p$和$q$，$p$不等于$q$，计算$N = p*q$.
- 根据欧拉函数，求得$φ(N)=φ(p)φ(q)=(p-1)(q-1)$。
- 选择一个小于$φ(N)$的整数$e$，使$e$与$φ(N)$互质。并求得$d$。
- 将$p$和$q$的记录销毁。

其中$(N，e)$是公钥，$(N，d)$是私钥。

#### 应用

RSA的应用包括数字签名，数字证书，ssh，https等。实际应用中，由于公钥加密计算量大，速度慢，常和对称加密一起使用，公钥加密算法常被用作最初建立连接，真正数据传输过程用对称加密算法处理。

<img src="/img/hash/https.jpg" style="zoom: 67%;" />

- ssh（Secure Shell）

简单说，SSH是一种网络协议，存在多种实现，既有商业实现，也有开源实现，采用了非对称加密技术(RSA)加密了所有传输的数据，用于计算机之间的加密登录。如果一个用户从本地计算机，使用SSH协议登录另一台远程计算机，我们就可以认为，这种登录是安全的，即使被中途截获，密码也不会泄露。

由于公私钥是唯一的一对，在客户端保障自己私钥安全的情况下，服务端通过公钥就可以完全确定客户端的真实性，所以要实现公钥登陆，要先生成一个公私密钥对。通过 `ssh-keygen -t rsa` 命令来生成密钥对，默认会保存到 `~/.ssh` 目录。使用`ssh-copy-id`命令将公钥复制到远程主机。ssh-copy-id会将公钥写到远程主机的 `~/ .ssh/authorized_key` 文件中

- https

HTTPS 默认工作在 TCP 协议443端口，它的工作流程一般如以下方式：

1、TCP 三次同步握手

2、客户端验证服务器数字证书

3、DH 算法协商对称加密算法的密钥、hash 算法的密钥

4、SSL 安全加密隧道协商完成

5、网页以加密的方式传输，用协商的对称加密算法和密钥加密，保证数据机密性；用协商的hash算法进行数据完整性保护，保证数据不被篡改。

> **HTTP 与 HTTPS 区别**
>
> HTTP 明文传输，数据都是未加密的，安全性较差，HTTPS（SSL+HTTP） 数据传输过程是加密的，安全性较好。
> 使用 HTTPS 协议需要到 CA（Certificate Authority，数字证书认证机构） 申请证书，一般免费证书较少，因而需要一定费用。证书颁发机构如：Symantec、Comodo、GoDaddy 和 GlobalSign 等。
> HTTP 页面响应速度比 HTTPS 快，主要是因为 HTTP 使用 TCP 三次握手建立连接，客户端和服务器需要交换 3 个包，而 HTTPS除了 TCP 的三个包，还要加上 ssl 握手需要的 9 个包，所以一共是 12 个包。
> http 和 https 使用的是完全不同的连接方式，用的端口也不一样，前者是 80，后者是 443。
> HTTPS 其实就是建构在 SSL/TLS 之上的 HTTP 协议，所以，要比较 HTTPS 比 HTTP 要更耗费服务器资源。

### MD5算法/数字摘要

用于提供消息的完整性性保护。算法的输出由四个32位分组组成，将这四个32位分组级联后将生成一个128位散列值，即16字节。又称数字摘要/数字指纹。

> 不可逆，从结果无法反推原始数据
> 高度离散性，输出结果无法预测，哪怕输入改变一个bit输出也完全不同
> 抗碰撞性，想找到两个产生完全一致的md5的数据非常困难

应用场景：

- 用户密码保护，储存密码时不记录密码本身，只记录密码的md5结果，只有用户自己知道密码明文

- 文件完整性校验，文件较大，网络传输的不可靠因素导致传输不完整，或被篡改，校验接收端与发送端的文件一致

- 数字签名，发布程序同时发布md5，防止植入木马。

> 经过发送方私钥加密的数字摘要，即“数字签名”，数字签名能确定消息确实是由发送方签名并发送出，因为别人假冒不了发送方的签名。还能能验证数据的完整性。

- 云盘秒传，数据库中搜索是否存在相同的md5，如果存在则使用现有的即可。

大概步骤如下：

填充：如果输入信息的长度(bit)对512求余的结果不等于448，就需要填充使得对512求余的结果等于448。填充的方法是填充一个1和n个0。即便原始长度满足也必须进行。填充完后，信息的长度就为N*512+448(bit)；

记录信息长度：用64位来存储填充前信息长度。这64位加在第一步结果的后面，这样信息长度就变为N\*512+448+64=(N+1)\*512位。

装入标准幻数（四个整数）：标准的幻数（物理顺序）是（A=(01234567) 16 ，B=(89ABCDEF) 16 ，C=(FEDCBA98) 16 ，D=(76543210) 16 ）。在程序中定义是小端，高位在前（A=0X67452301L，B=0XEFCDAB89L，C=0X98BADCFEL，D=0X10325476L）。

四轮循环运算：循环的次数是分组的个数（N+1）

