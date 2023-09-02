##### 存放地址：data/data/&lt;package name&gt;/databases/



##### 数据类型

NULL:              这个值为空值

VARCHAR(n)：长度不固定且其最大长度为 n 的字串，n不能超过 4000。

CHAR(n)：       长度固定为n的字串，n不能超过 254。

INTEGER:         值被标识为整数,依据值的大小可以依次被存储为1,2,3,4,5,6,7,8.

REAL:                所有值都是浮动的数值,被存储为8字节的IEEE浮动标记序号.

TEXT: 		  值为文本字符串,使用数据库编码存储(TUTF-8, UTF-16BE or UTF-16-LE).

BLOB: 		  值是BLOB数据块，以输入的数据格式进行存储。如何输入就如何存储,不改变格式。（表示二进制类型）

DATE：		   包含了 年份、月份、日期。

TIME： 	           包含了 小时、分钟、秒。



##### 常用方法

| 方法名称                                                     | 方法表示含义     |
| ------------------------------------------------------------ | ---------------- |
| openOrCreateDatabase(String path,SQLiteDatabase.CursorFactory  factory) | 打开或创建数据库 |
| insert(String table,String nullColumnHack,ContentValues  values) | 插入一条记录     |
| delete(String table,String whereClause,String[]  whereArgs)  | 删除一条记录     |
| query(String table,String[] columns,String selection,String[]  selectionArgs,String groupBy,String having,String  orderBy) | 查询一条记录     |
| update(String table,ContentValues values,String whereClause,String[]  whereArgs) | 修改记录         |
| execSQL(String sql)                                          | 执行一条SQL语句  |
| close()                                                      | 关闭数据库       |



##### 实例

```java
public class MyDatabaseHelper extends SQLiteOpenHelper{
    //给出创建表的例子
    private static final String CREATE_GOOD_IN_CART_TABLE=
        "create table if not exists goodsInCart("
                +"id integer primary key autoincrement,"
                +"name text,"
                +"price real,"
                +"amount integer,"
                +"orderCode )";
    /**
    *
    * @param context
    * @param name     数据库名字
    * @param factory
    * @param version  版本
    */
    public MyDatabaseHelper(Context context, String name,SQLiteDatabase.CursorFactory factory, int version) {
        super(context, name, factory, version);
    }
    @Override
    public void onCreate(SQLiteDatabase sqLiteDatabase) {
        Log.e("MyDatabaseHelper","执行onCreate()");
        this.sqLiteDatabase=sqLiteDatabase;
        sqLiteDatabase.execSQL(CREATE_GOOD_IN_CART_TABLE);
    }
    public void createTable(String sqlString){
        sqLiteDatabase.execSQL(sqlString);
        Toast.makeText(mContext,"执行创建表",Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onUpgrade(SQLiteDatabase sqLiteDatabase, int i, int i1) {

    }
    
}
```

