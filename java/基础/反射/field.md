#### Field

##### field 属性

- `getName()`：返回字段名称，例如，`"name"`；
- `getType()`：返回字段类型，也是一个`Class`实例，例如，`String.class`；
- `getModifiers()`：返回字段的修饰符，它是一个`int`，不同的bit表示不同的含义。

实例：

```java
package _field;
public class Person {
    private int age;
    public String name;
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    private void setAge(int age) {
        this.age = age;
    }
}
```

```java
private static void logFieldMsg(Field field) {
    System.out.println("current field: " + field);
    System.out.println("\tname = " + field.getName());
    System.out.println("\ttype = " + field.getType());
    System.out.println("\tmodifiers = " + field.getModifiers());
}

public static void main(String[] args) {
    Class pCls = Person.class;
    Field []fields = pCls.getDeclaredFields();
    for (Field field : fields) {
        logFieldMsg(field);
    }
}
```

输出结果：

```java
/**
 * current field: private int _field.Person.age
 *     name = age
 *     type = int
 *     modifiers = 2
 * current field: public java.lang.String _field.Person.name
 *     name = name
 *     type = class java.lang.String
 *     modifiers = 1
 */
```



##### 获取 field

- Field getField(name)：根据字段名获取**某个public**的field（包括父类）
- Field[] getFields()：获取所有**public**的field（包括父类）
- Field getDeclaredField(name)：根据字段名获取当前类的某个field（不包括父类）
- Field[] getDeclaredFields()：获取当前类的所有field（不包括父类）



##### 使用 field

针对 private 变量，应使用 setAccessible(true) 来指示 可修改其值

```java
Field ageField = pCls.getDeclaredField("age");
ageField.setAccessible(true);
ageField.set(person, 22);
```

