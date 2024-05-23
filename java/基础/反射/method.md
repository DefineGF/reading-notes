#### Method

##### Method属性

- `getName()`：方法名称
- `getReturnType()`：方法返回值类型，Class 类型
- `getParameterTypes()`：方法的参数类型，Class数组，
- `getModifiers()`：返回方法的修饰符，它是一个`int`



##### 获取Method

- `Method getMethod(name, Class...)`：获取某个`public`的`Method`（包括父类）
- `Method getMethod(name, Class...)`：获取某个`public`的`Method`（包括父类）
- `Method[] getMethods()`：获取所有`public`的`Method`（包括父类）
- `Method getDeclaredMethod(name, Class...)`：获取当前类的某个`Method`（不包括父类）
- `Method[] getDeclaredMethods()`：获取当前类的所有`Method`（不包括父类）



##### 调用实例

```java
Method setAgeMethod = person.getClass().getDeclaredMethod("setAge", int.class);
setAgeMethod.setAccessible(true);
setAgeMethod.invoke(person, 22);
```



