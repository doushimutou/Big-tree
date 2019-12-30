# CopyOnWriteArrayList

## ArrayList并发问题：

下面的代码片段，抛出`Exception in thread "pool-1-thread-2" java.util.ConcurrentModificationException` ;

```JAVA
public class COWTest {
	public static void main(String[] args) {
		ArrayList<String> arrayList = new ArrayList<>();
		arrayList.add("aaa");
		arrayList.add("bbb");
		arrayList.add("ccc");
		arrayList.add("ddd");
		Iterator<String> iter = arrayList.iterator();
		ExecutorService service = Executors.newFixedThreadPool(5);
        //模拟多线程下的读
		for (int i = 0; i <5 ; i++) {
			service.execute(() -> {
				arrayList.iterator();
				while (iter.hasNext()) {
					System.err.println(iter.next());
				}
			});
		}

		// 执行5个任务, 模拟多线程下的写的操作
		for (int i = 0; i < 5; i++) {
			service.execute(()->{
				arrayList.add("111");
			});
		}
		System.err.println(Arrays.toString(arrayList.toArray()));
	}
}

```

同样的代码把容器换成CopyOnWriteArratList,不在抛异常，

```java
CopyOnWriteArrayList<String> copyOnWriteArrayList = new CopyOnWriteArrayList<>();
		copyOnWriteArrayList.add("aaa");
		copyOnWriteArrayList.add("bbb");
		copyOnWriteArrayList.add("ccc");
		copyOnWriteArrayList.add("ddd");
		Iterator<String> iterator = copyOnWriteArrayList.iterator();
		ExecutorService service = Executors.newFixedThreadPool(5);

		for (int i = 0; i <5 ; i++) {
			service.execute( ()->{
				while (iterator.hasNext()){
					System.out.println(iterator.next());
				}
			});
		}
		for (int i = 0 ;i<5 ;i++){
			service.execute( ()->{
				copyOnWriteArrayList.add("222");
			});
		}
		System.out.println(copyOnWriteArrayList);

```



## CopyOnWriteArrayList初始化：

构造函数：

```
    
    public CopyOnWriteArrayList() {
        setArray(new Object[0]);
    }

    public CopyOnWriteArrayList(Collection<? extends E> c) {
        Object[] elements;
        if (c.getClass() == CopyOnWriteArrayList.class)
            elements = ((CopyOnWriteArrayList<?>)c).getArray();
        else {
            elements = c.toArray();
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elements.getClass() != Object[].class)
                elements = Arrays.copyOf(elements, elements.length, Object[].class);
        }
        setArray(elements);
    }


    public CopyOnWriteArrayList(E[] toCopyIn) {
        setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
    }
```

 从构造函数看出，无论那种构造方法，都会创建一个Object类型的数组，然后赋值给成员array。



## add、remove解析

### add():

```java
public boolean add(E e) {
    	//获取独占锁
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            //获取 array
            Object[] elements = getArray();
            //复制array到新数组，添加元素到新数组
            int len = elements.length;
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            //使用新数组替换添加前的数组
            setArray(newElements);
            return true;
        } finally {
            //释放锁
            lock.unlock();
        }
    }
```

首先使用上面的两行代码加上了锁，保证同一时间只能有一个线程在添加元素。

然后使用Arrays.copyOf(...)方法复制出另一个新的数组，而且新的数组的长度比原来数组的长度+1，副本复制完毕，新添加的元素也赋值添加完毕，最后又把新的副本数组赋值给了旧的数组，最后在finally语句块中将锁释放。
在添加元素的时候，首先**复制了一个快照**，然后对这个快照进行添加，而不是直接在原来的数组上。

### romve():

```java
    public E remove(int index) {
        //获取独占锁
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            //获取数组
            Object[] elements = getArray();
            int len = elements.length;
            //获取指定元素
            E oldValue = get(elements, index);
            int numMoved = len - index - 1;
            //如果要删除最后一个
            if (numMoved == 0)
                setArray(Arrays.copyOf(elements, len - 1));
            else {
                //分两次复制删除后剩余的元素到新数组中
                Object[] newElements = new Object[len - 1];
                System.arraycopy(elements, 0, newElements, 0, index);
                System.arraycopy(elements, index + 1, newElements, index,
                                 numMoved);
                //使用新数组替换老数组
                setArray(newElements);
            }
            return oldValue;
        } finally {
            //释放独占锁
            lock.unlock();
        }
    }
```

删除元素，很简单，就是判断要删除的元素是否最后一个，如果最后一个直接在复制副本数组的时候，复制长度为旧数组的length-1即可；但是如果不是最后一个元素，就先复制旧的数组的index前面元素到新数组中，然后再复制旧数组中index后面的元素到数组中，最后再把新数组复制给旧数组的引用。
———————————————

## 弱一致性

在遍历期间，其他线程对list进行了增删改的操作，但是操作与读取的是两个list，这就是弱一致性。



## `CopyOnWriteArrayList`使用场景

- 1、读多写少（白名单，黑名单，商品类目的访问和更新场景），为什么？因为写的时候会复制新集合
- 2、集合不大，为什么？因为写的时候会复制新集合
- 实时性要求不高，为什么，因为有可能会读取到旧的集合数据








