### 1. 简介

- **概念**：Java Database Connectivity
- **本质**：是官方（sun公司）定义的一套操作所有关系型数据库的规则接口，并由数据库厂商去实现这套接口，提供数据库驱动的jar包。

我们可以使用这套接口（JDBC）去编程，而真正执行的代码是驱动中的实现类。（多态）

### 2. 快速入门

```java
public class TestJdbc {
    public static void main(String[] args) throws Exception {
        //1. 注册驱动
        Class.forName("com.mysql.jdbc.Driver");

        //2. 获取连接对象
        Connection conn =DriverManager.getConnection("jdbc:mysql://localhost:3306/zmall?useSSL=false","root","password");

        //3. SQL
        String sql ="update user set role=0 where username=\"admin\"";

        //4. 获取执行SQL对象
        Statement stmt=conn.createStatement();

        //5. 执行sql
        int count= stmt.executeUpdate(sql);

        System.out.println(count);

        //6. 释放资源
        stmt.close();
        conn.close();
    }
}
```



### 3. 常用类

#### 3.1 DriverManager

> 驱动管理对象

**1. 注册驱动**

上面程序使用`Class.forName("com.mysql.jdbc.Driver")`注册驱动，是因为Driver类中含有一个静态代码块，调用DriverManager的**registerDriver**的方法注册驱动。

```java
//Driver源码
static {
    try {
        DriverManager.registerDriver(new Driver());
    } catch (SQLException var1) {
        throw new RuntimeException("Can't register driver!");
    }
}
```

同时，5.x版本之后可以省略注册驱动的步骤，这是因为在驱动jar包中包含一个`META-INF\services\java.sql.Driver`文件，可以帮助我们自动注册驱动。

**2. 获取数据库连接**

```java
static Connection getConnection(String url, String user, String password);
```

#### 3.2 Connection

> 数据库连接对象

**1. 获取执行SQL的对象**

```java
Statement createStatement() throws SQLException;

PreparedStatement prepareStatement(String sql)throws SQLException;
```

**2. 管理事务**

```java
//开启事务，设置参数为false开启事务
void setAutoCommit(boolean autoCommit) throws SQLException;

//提交事务
void commit() throws SQLException;

//事务回滚
void rollback() throws SQLException;
```

#### 3.3 Statement

> 执行SQL的对象，执行静态SQL语句并返回运算结果

```java
//执行任意的SQL（不常用）
boolean execute(String sql) throws SQLException;

//执行DML（增删改）、DDL（创建）语句，并返回影响行数
int executeUpdate(String sql) throws SQLException;

//执行DQL(查询)语句
ResultSet executeQuery(String sql) throws SQLException;
```

#### 3.4 ResultSet

> 结果集对象

- **next()**：游标向下移动一行；

```java
boolean next() throws SQLException;
```

- **getXxx(param)**：获取数据。

其中参数<int>代表参数列的标号，从1开始；<String>类型代表列的名称。

```java
Connection conn= DriverManager.getConnection("jdbc:mysql:///zmall?useSSL=false","root","password");

Statement stmt=conn.createStatement();
String sql="select * from user";

ResultSet rs=stmt.executeQuery(sql);

while (rs.next()){
    System.out.print("username: "+rs.getString("username")+" password: ");
    System.out.println(rs.getString("password"));
}
```

#### 3.5 PreparedStatement

> 继承Statement，执行预编译的SQL语句

**1. 解决SQL注入问题**

```java
//SQL的参数使用占位符`?`
String sql="select * from user where username=? and password=?";
//获取对象
PreparedStatement pstmt=conn.prepareStatement(sql);
//传递参数，以1开始
pstmt.setString(1,"admin");
pstmt.setString(2,"admin");
//执行SQL语句
ResultSet rs=pstmt.executeQuery();
```



### 4. 事务控制

```java
 public static void main(String[] args){
        Connection conn=null;
        try {
            conn= DriverManager.getConnection("jdbc:mysql:///zmall?useSSL=false"
                    ,"root","password");

            //开启事务
            conn.setAutoCommit(false);

            String sql="update user set role = ? where username=?";
            PreparedStatement pstmt1=conn.prepareStatement(sql);
            PreparedStatement pstmt2=conn.prepareStatement(sql);
            //赋值操作，index从1开始
            pstmt1.setString(2,"admin");
            ......
            pstmt1.executeQuery();
            //制造异常
            int a=3/0;
            pstmt2.executeQuery();

            //提交事务
            conn.commit();
        }catch (Exception e){
            //事务回滚
            try {
                if (conn!=null) conn.rollback();    //判断是否存在连接
            } catch (SQLException e1) {
                e1.printStackTrace();
            }
        }
    }
```



### 5. 数据库连接池

> 存放数据库连接的容器（集合），节约资源，高效访问。

- 标准接口：javax.sql包下的 DataSource，包含两个主要方法，获取连接和归还连接；
- 一般由数据库厂家实现，常用的两个数据库连接池技术：C3P0和Druid。

**C3P0**

引入jar包，并配置文件`c3p0.properties`或`c3p0-config.xml`放置类路径下。

```java
//创建连接池，可传参数，选择配置文件中的其他配置
DataSource ds=new ComboPooledDataSource();

//获取连接池
Connection cnn=ds.getConnection();

//归还连接
conn.close();
```

**Druid**

导入jar包，配置文件`druid.properties`。可以是任意名称、任意路径。

```properties
url=jdbc:mysql:///zmall?useSSL=false
driverClassName=com.mysql.jdbc.Driver
username=root
password=root
```

| 属性            | 默认            | 描述                                                         |
| :-------------- | :-------------- | :----------------------------------------------------------- |
| username        |                 | 连接数据库的用户名                                           |
| password        |                 | 连接数据库的密码                                             |
| jdbcUrl         |                 | 同DBCP中的jdbcUrl属性                                        |
| driverClassName | 根据url自动识别 | 这一项可配可不配，如果不配置druid会根据url自动识别dbType，然后选择相应的driverClassName |
| initialSize     | 0               | 初始化时建立物理连接的个数。初始化发生在显示调用init方法，或者第一次getConnection时 |
| maxActive       | 8               | 最大连接池数量(Maximum number of Connections a pool will maintain at any given time) |
| maxIdle         | 8               | 已经不再使用，配置了也没效果                                 |
| minIdle         |                 | 最小连接池数量                                               |
| maxWait         |                 | 获取连接时最大等待时间，单位毫秒。配置了maxWait之后，缺省启用公平锁，并发效率会有所下降，如果需要可以通过配置useUnfairLock属性为true使用非公平锁。 |

```java
//加载配置
Properties pro=new Properties();
InputStream is=TestDruid.class.getClassLoader().getResourceAsStream("jdbc.properties");
pro.load(is);
//通过工厂创建连接池对象
DataSource ds= DruidDataSourceFactory.createDataSource(pro);
//获取连接
Connection conn=ds.getConnection();
```



### 6. Spring JDBC

> Spring框架对jdbc的简单封装

需要创建JdbcTemplate对象，依赖于数据源DataSource。

```java
dbcTemplate template=new JdbcTemplate(dataSource);
String sql="update user set username=? where id=?";

int count=template.update(sql, "admin", 1);
```

**常用方法**

| 方法名           | 描述                       |
| ---------------- | -------------------------- |
| update()         | 执行增删改语句（DML）      |
| queryForMap()    | 查询结果封装成Map集合      |
| queryForList()   | 查询结果封装成List集合     |
| query()          | 查询结果封装成JavaBean对象 |
| queryForObject() | 查询结果并封装成对象       |


