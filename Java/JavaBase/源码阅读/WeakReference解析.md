> 今天看`ThreadLocal`源码的时候
>
> 发现`ThreadLocalMap`这个中的`Entry`类继承了`WeakReference`

```java
/**
 * The entries in this hash map extend WeakReference, using
 * its main ref field as the key (which is always a
 * ThreadLocal object).  Note that null keys (i.e. entry.get()
 * == null) mean that the key is no longer referenced, so the
 * entry can be expunged from table.  Such entries are referred to
 * as "stale entries" in the code that follows.
 */
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```



那么`WeakReference`有什么用呢？

`WeakReference`如字面意思，弱引用， 当一个对象仅仅被weak reference（弱引用）指向, 而没有任何其他strong reference（强引用）指向的时候, 如果这时GC运行, 那么这个对象就会被回收，不论当前的内存空间是否足够，这个对象都会被回收。

而上述源码中`ThreadLocal`被弱引用



这里写个例子理解一下：

`User`

```java
public class User {
}
```

`Ref`

```java
public class Ref extends WeakReference<User> {
	/**
	 * 构造函数，弱引用指向参数referent
	 *
	 * @param referent
	 */
	public Ref(User referent) {
		super(referent);
	}
}
```

`Test`

```java
public class Test {
	public static void main(String[] args) {
		//弱引用指向User
		Ref ref = new Ref(new User());
		System.out.println(ref.get()==null);
		//垃圾回收运行
		System.gc();
		//线程休眠，给予足够的时间回收垃圾
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println(ref.get()==null);
	}
}
```

结果：

```text
false
true
```

