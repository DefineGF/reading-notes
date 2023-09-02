

#### virtual和abstract

用来修饰方法时必须添加public；



##### 差异

- virtual方法必须要有实现；而abstract方法一定不能实现；
- virtual可以被子类实现，abstract方法必须被子类实现；
- 只有抽象类才有抽象方法，因此拥有abstract方法的类必须被abstract修饰；
- 无法创建abstract实例，只能被继承；