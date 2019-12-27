# map

​	主要用于存储健值对，根据键得到值，因此不允许键重复(重复会覆盖)，但允许值重复。map<key,value>,key不允许重复。

### Hashmap

- Hashmap是一个最常用的Map，它根据键的HashCode值存储数据，根据键可以直接获取它的值，具有很快的访问速度。遍历时，取得数据的顺序是完全随机的；

- HashMap不支持线程的同步（非线程安全），即任一时刻可以有多个线程同时写HashMap，可能会导致数据的不一致；

- 在Map中插入、删除和定位元素，HashMap是最好的选择。

**几种构建hashmap的构造方法：**

  HashMap()：构建一个空的哈希映像

  HashMap(Map m)：构建一个哈希映像，并且添加映像m的所有映射

  HashMap(int initialCapacity)：构建一个拥有特定容量的空的哈希映像

  HashMap(int initialCapacity, float loadFactor)：构建一个拥有特定容量和加载因子的空的哈希映像

  

### hashtable

- HashTable与HashMap类似，它不允许记录的键或者值为空；

- 支持线程的同步（线程安全），即任一时刻只有一个线程能写HashTable，因此导致了Hashtable在写入时会比较慢。

### linkedhashmap

- LinkedHashMap是HashMap的一个子类；

- LinkedHashMap保存了记录的插入顺序，在用Iterator遍历LinkedHashMap时，先得到的记录肯定是先插入的；

在遍历的时候会比HashMap慢，不过有种情况例外，当HashMap容量很大，实际数据较少时，遍历起来可能会比LinkedHashMap慢，因为LinkedHashMap的遍历速度只和实际数据有关，和容量无关，而HashMap的遍历速度和他的容量有关。

### treemap

- TreeMap实现SortMap接口，能够把它保存的记录根据键排序，默认是按键值的升序排序，也可以指定排序的比较器。当用Iterator遍历TreeMap时，得到的记录是排过序的。

- TreeMap取出来的是排序后的键值对,非线程安全

  

**几种构建TreeMap的构造方法：**

TreeMap()：构建一个空的映像树

TreeMap(Map m)：构建一个映像树，并且添加映像m中所有元素

TreeMap(Comparator c)：构建一个映像树，并且使用特定的比较器对关键字进行排序

TreeMap(SortedMap s)：构建一个映像树，添加映像树s中所有映射，并且使用与有序映像s相同的比较器排序



### 总结：

在单线程、没有高并发的要求下，非线程安全的情况下，使用hashmap,初始化hashmap时候在知道容量的情况下，可以指定容量，避免扩容带来的性能消耗；

当需要对key或则value排序时且线程要求是非安全的情况下，优先考虑使用treemap；

当需要根据元素放入的顺序来去除的时候，且线程要求是非安全的情况下，优先考虑使用linkedHashmap;



### 生成环境遇到一个BUG:

需要根据根据一个条件来排序一个map里面的value,首先想到了treemap，可以实现排序功能；但是treemap默认安装key的大小排序，不是实际需要是要按照放入顺序排序；把treemap更改为linkedHashmap问题解决。



> 后续还会对hashmap、linkedhashmap、treemap 从源码角度分析为什么他们具有这些特性

