## 对象

#### Integer

##### 生成对象

- Integer i = 100：

  > 使用自动装箱，将整数值 `100` 转换为 `Integer` 对象。这种方式会利用对象池（缓存池）技术，在范围 `-128` 到 `127` 内复用已有的 `Integer` 对象，或者在超出范围时创建新的对象

- Integer i = new Integer(100):

  > 显式地通过构造函数创建一个新的 `Integer` 对象，该对象的值为 `100`。无论整数值是否在 `-128` 到 `127` 范围内，都会创建一个新的对象

源码：

```java
// 当执行Integer i = numb;时，将会翻译成Integer i = Integet.valueOf(numb);
public static Integer valueOf(int i) {
    // low = -128;  h = 127(也可通过VM设置)
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache. [i + (-IntegerCache.low)];//cache = new Integer[(high - low) + 1];
    return new Integer(i);
}
```



```java
// 堆上生成不同对象
Integer i = new Integer(100);
Integer j = new Integer(100);
print(i == j);  // true

// 自动拆箱
Integer i = new Integer(100);
int j = 100；	// 基本类型 int 直接存储数据值：指向常量池中对象
print(i == j);   // true；自动装/拆箱，Integer -> int,实际上是两个int进行比较


Integer i = 128;
Integer j = 128;
System.out.print(i == j); //false
```



### 对象方法

#### equals

- 自反性，对称性，传递性，一致性，非空性；
- java中所有类都继承Object，其中Object中的原始equals通过对比堆中地址来比较；
- 继承与Object的 String，Integer，Date等对象重写了equals方法，而不是比较堆中地址了！

```java
// 直接对比对象地址
public boolean equals(Object obj) {
    return (this == obj);
}

// 例子
class MyObject { // 单纯继承 object
}


Set<MyObject> objs = new Set<MyObject>();  // 通过 equals 判断是否相等（set不添加重复节点）
objs.add(new MyObject());
objs.add(new MyObject()); 				   // set 长度值为：2
```

两个新建的实例，由于equals() 比较为 false，则被判定为 不等；尽管两者内容同时为空。



#### Object.hashCode

- 若两个对象通过equals对比返回true，那么两者的产生的Integer 类型的 hashCode也当相等；
- 若两个对象 equals 返回 false，但hashcode 不必相等；

```java
public native int hashCode();
```

计算方法：

- Boolean：true：false ? 1231 : 1237;

- Character: ascii 值

- Integer：依整数值

- Double：

  ```java
  public static int hashCode(double value) {
      long bits = doubleToLongBits(value);//其中doubleToLongBits使用native doubleToRawLongBits
      return (int)(bits ^ (bits >>> 32));
  }
  ```

- String

  ```java
  public int hashCode() {
      int h = hash;//hash值默认为0
      if (h == 0 && value.length > 0) {   //value为char[],其值为传入字符串字符
          char val[] = value;
          for (int i = 0; i < value.length; i++) {
              h = 31 * h + val[i];        //实则是将字符的ascll值相加
          }
          hash = h;
      }
      return h;
  }
  ```

  

#### Object.toString

```java
// 直接输出对象并非获得的是对象地址，而是对象名 + 对象 hashCode 
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```



#### hashcode & equals

**在重写 equals() 时候，确定重写 hashCode() **,使其满足条件：

- equals == true：有相同 hashCode；
- equals == false：尽量满足不同 hashCode；

```java
public class SimpleBean {
    private String name;
    private int    age;

    public SimpleBean(String name,int age){
        this.name = name;
        this.age  = age;
    }
	
    // 省去 set() / get() 方法

    @Override
    public boolean equals(Object obj) {
        SimpleBean bean = (SimpleBean)obj;
        return this.age == bean.getAge() && this.name.equals(bean.getName());
    }

    @Override
    public int hashCode() {
        int hashCode = name.hashCode();
        hashCode += Integer.hashCode(age);
        return hashCode;
    }

	@Override
    public String toString() {
        return "name = " + name + "; age = " + age + "\n";
    }
}
```



#### 克隆

##### Object.clone()

Object.clone() 会检测出对象的大小，并为新对象创建足够的内存空间，然后将对象所有的二进制位复制到新对象中，即按位复制。

当然这是针对非组合对象，即对象只包含基本类型。



##### Cloneable 接口

1. 可能有个对某父类型的向上转型的引用，但并不知道是否可以克隆该对象；

   ```java
   if (myReference instanceof Cloneable) //
   ```

2. 并不希望所有的对象都是可以克隆的。因此Object.clone() 可以验证一个类是否实现了Cloneable接口。如果没有，便会抛出 CloneNotSupportedException.



##### 引用类型拷贝

```java
class CPU implements Cloneable{
    long memory;
    public CPU(long _memory) {
        this.memory = _memory;
    }

    @Override
    public CPU clone() {
        try {
            return (CPU)super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }
}

class GPU implements Cloneable {
    double refreshRate;
    public GPU(double _rate) {
        this.refreshRate = _rate;
    }

    @Override
    public GPU clone() {
        try {
            return (GPU) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }
}

public class PC {
    private CPU cpu;
    private GPU gpu;
    @Override
    public PC clone() {
        PC pc;
        try {
            pc = (PC)super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
        if (pc != null) {
            pc.cpu = pc.cpu.clone();
            pc.gpu = pc.gpu.clone();
        }
        return pc;
    }
}
```



##### 控制可克隆性

```java
class Ordinary {}  // 未重写clone()

// 抛出异常：重写clone，但未实现 Cloneable
class WrongClone extends Ordinary {
	@Override 
    public Object clone() throws CloneNotSupportedException {
		return super.clone();
	}
}
```

关闭克隆：通过抛出异常

```java
class NoMore {
	@Override public Object clone() throws CloneNotSupportedException {
		throw new CloneNotSupportedException();
	}
}
class TryMore extends NoMore {
    @Override public Object clone() throws CloneSupportedException {
        return super.Clone(); // 调用 NoMore 的 clone 函数，抛出异常
    }
}
```



