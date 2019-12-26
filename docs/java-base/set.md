# set

## HashSet

> 存取无序，元素唯一：

​	示例：

```java
HashSet<String> hs = new HashSet<>();
		hs.add("a");
		hs.add("c");
		hs.add("b");
		hs.add("a");
		hs.add("d");
		for (int i = 0; i < 100; i++) {
			System.out.println(hs);
		}
```

输出结果：[a, b, c, d]

​		HashSet在存储元素的时候会调用元素的hashCode值，看哈希值与已经存在的hashSet的元素的哈希值是否相同，如果不同就直接添加元素；如果相同，则继续条用元素的equals()和哈希值相同的元素一次比较。返回为true就重复不添加；如果false就步重复添加。

- 源码

  hashSet.add()的 `final V putVal(int hash, K key, V value, boolean onlyIfAbsent,               boolean evict)` 方法，即实现了上述的比较过程。

**故：保证元素唯一性需要让元素重写两个方法：equals()和 hashCode();**

​	示例：

- 在重写equals()和hashCode()之前：

```java
public static void testHashSet2() {
		HashSet<Person> hs =  new HashSet<>();
		hs.add(new Person("lisi",2));
		hs.add(new Person("lisi",2));
		hs.add(new Person("lisi",2));
		System.out.println(hs);

	}
```

输入结果： `[Person{name='lisi', age=2}, Person{name='lisi', age=2}, Person{name='lisi', age=2}]`

- Person重写equals()和hashCode()之后：
输出结果：`[Person{name='lisi', age=2}]`

## TreeSet

> 元素唯一，给元素排序

- 问题:为什么使用TreeSet存入字符串,字符串默认输出是按升序排列的?

  因为字符串实现了一个接口,叫做Comparable 接口.字符串重写了该接口的compareTo 方法,所以String对象具备了比较性.那么同样道理,自定义元素(例如Person类,Book类)想要存入TreeSet集合,就需要实现该接口,也就是要让自定义对象具备比较性.

  存入TreeSet集合中的元素要具备比较性.

**实现的方式如下：**

方式一：让元素所在类实现Comparable接口，并重谢CompareTo()方法，并根据CompareTo()的返回值来进行添加元素

* 返回正数：往二叉树的右边添加
* 返回负数：往二叉树的左边添加
* 返回 0 ： 说明重复，不添加

示例：

```
public static void testTreeSet() {
		TreeSet<Person> ts = new TreeSet<>();
		ts.add(new Person("zhangsan", 9));
		ts.add(new Person("lisi", 10));
		ts.add(new Person("wangwu", 15));
		System.out.println(ts);
	}
	
	
public class Person  implements Comparable<Person>{
	private String name;
	private int age;

	/**
	 * 按年龄大小后按姓名
	 * @param o
	 * @return
	 */
	@Override
	public int compareTo(Person o) {
		int num = this.age - o.age;
		return num ==0 ? this.name.compareTo(o.name) : num  ;
	}
}
```

结果：`[Person{name='zhangsan', age=9}, Person{name='lisi', age=10}, Person{name='wangwu', age=15}]`



方式二：在使用treeset构造函数的时候，直接传入一个比较器，不用Person实现Comparable接口

```java
public static void testTreeSet() {
		TreeSet<Person> ts = new TreeSet<>(new Comparator<Person>() {	
        //3）：按照姓名长度排（先比较姓名长度 后比较姓名内容 后比较年龄）
        @Override
        public int compare(Person o1, Person o2) {//o1是即将存入的元素  o2是已经存入集合的元素
            //比较长度为主要条件
            int length = o1.getName().length() - o2.getName().length();	
            //比较内容为次要条件
            int num = length == 0 ? o1.getName().compareTo(o2.getName()) : length;	
            return num == 0 ? o1.getAge() - o2.getAge() : num;
        
    });
		ts.add(new Person("zhangsan", 9));
		ts.add(new Person("lisi", 10));
		ts.add(new Person("wangwu", 15));
		System.out.println(ts);
	}
```



## LinkedHashSet

会保存插入的顺序。



**引用一句大神的话：**

看到array，就要想到角标。

看到link，就要想到first，last。

看到hash，就要想到hashCode,equals.

看到tree，就要想到两个接口。Comparable，Comparator。