

#### 实现简易的 SimpleAtomicInteger

##### 使用 synchronized

```java
public class SimpleAtomicInteger {
    private int value;

    public SimpleAtomicInteger(int initialValue) {
        value = initialValue;
    }

    public synchronized int get() {
        return value;
    }

    public synchronized void set(int newValue) {
        value = newValue;
    }

    public synchronized int getAndIncrement() {
        int oldValue = value;
        value++;
        return oldValue;
    }

    public synchronized int incrementAndGet() {
        value++;
        return value;
    }
}
```

##### 使用 CAS

```java
import sun.misc.Unsafe;
import java.lang.reflect.Field;

public class SimpleAtomicInteger {
    private static final Unsafe unsafe;
    private static final long valueOffset;

    static {
        try {
            Field unsafeField = Unsafe.class.getDeclaredField("theUnsafe");
            unsafeField.setAccessible(true);
            unsafe = (Unsafe) unsafeField.get(null);
            valueOffset = unsafe.objectFieldOffset(SimpleAtomicInteger.class.getDeclaredField("value"));
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    private volatile int value;

    public SimpleAtomicInteger(int initialValue) {
        value = initialValue;
    }

    public int get() {
        return value;
    }

    public void set(int newValue) {
        value = newValue;
    }

    public int getAndIncrement() {
        int oldValue;
        do {
            oldValue = value;
        } while (!compareAndSwap(oldValue, oldValue + 1));
        return oldValue;
    }

    public int incrementAndGet() {
        int newValue;
        do {
            newValue = value + 1;
        } while (!compareAndSwap(value, newValue));
        return newValue;
    }

    private boolean compareAndSwap(int expectedValue, int newValue) {
        return unsafe.compareAndSwapInt(this, valueOffset, expectedValue, newValue);
    }
}
```

方法：public final native boolean compareAndSwapInt(Object obj, long offset, int expectedValue, int newValue);

- `obj`：要操作的对象
- `offset`：对象内存中字段的偏移量
- `expectedValue`：期望的值
- `newValue`：要设置的新值