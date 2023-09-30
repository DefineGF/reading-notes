##### 常用函数

```java
String DBDRIVER="com.mysql.jdbc.Driver";
String DBURL="jdbc:mysql://localhost:3306/javaweb?useSSL=false";
Class.forName(DBDRIVER);    //注意先注册一下

Connection connection = DriverManager.getConnection(DBURL,DBUSER,DBPASSWORD);
connection.close();

String addSql="insert into commodity(name,price) values(?,?)";
PreparedStatement pstmt = connection.prepareStatement(addSql);
pstmt.setString(1, commodity.getName()); // commodity 为对象实例
pstmt.setDouble(2, commodity.getPrice());
pstmt.executeUpdate();
```



##### 修改

```java
Connection connection = DBConnection.getConnection();
String sqlString="update commodity set name=?,price=? where id=?";

PreparedStatement pstmt = connection.prepareStatement(sqlString);
pstmt.setString(1, commodity.getName());
pstmt.setDouble(2, commodity.getPrice());
pstmt.setInt(3, commodity.getId());
pstmt.executeUpdate();
```



##### 删除

```java
Connection connection = DBConnection.getConnection();
String sqlString="delete from commodity where id=?";
PreparedStatement pstmt=null;
try {
    pstmt=connection.prepareStatement(sqlString);
    pstmt.setInt(1,commodity.getId());
    pstmt.executeUpdate();
}
```



##### 查找

```java
Connection connection = DBConnection.getConnection();
String sqlString="select *from commodity";

PreparedStatement pstmt = connection.prepareStatement(sqlString);

List<Commodity> commodities=new ArrayList<>();
ResultSet resultSet = pstmt.executeQuery();
while(resultSet.next()){
    Commodity tempCom=new Commodity();
    tempCom.setId(resultSet.getInt("id")); // == getInt(int row); 比如 getInt(1)
    tempCom.setName(resultSet.getString("name"));
    tempCom.setPrice(resultSet.getDouble("price"));
    commodities.add(tempCom);
}
```

