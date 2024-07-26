# 工具类:

```java
public class JDBCUtils {
    static String user;
    static String password;
    static String driverClassName;
    static String url;

    static {
        FileInputStream fis = null;
        try {
            //1.读取配置文件
            Properties properties = new Properties();
            //创建流
            fis = new FileInputStream("文件的路径+名字");
            //加载流
            properties.load(fis);
            //读取数据
            user = properties.getProperty("user");
            password = properties.getProperty("password");
            driverClassName = properties.getProperty("driverClassName");
            url = properties.getProperty("url");
            System.out.println(user + " " + password + " " + driverClassName + " " + url);
        } catch (Exception e) {
            //终止程序运行
            throw new RuntimeException(e.getMessage());
        } finally {
            //关闭资源
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }

    //自定义方法
    public static Connection getConnection() {
        try {
            //获取Connection对象
            Class.forName(driverClassName);
            //通过driverManager获取Connection对象
            Connection connection = DriverManager.getConnection(url, user, password);
            return connection;
        } catch (Exception e) {
            //终止程序运行
            throw new RuntimeException(e.getMessage());
        }
    }

    //关闭资源
    public static void close(PreparedStatement ps, Connection connection) {
        if (ps != null) {
            try {
                ps.close();
            } catch (SQLException e) {
                throw new RuntimeException(e);
            }
        }
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {
                throw new RuntimeException(e);
            }
        }
    }
}

```

# 增,删,改:

```java
public class CRUDDemo {

    @Test
    public void Test1()throws Exception{
        //获取Connection对象
        Connection connection = JDBCUtils.getConnection();
        //写SQL
        String sql = "insert into xxx(x,x,...) values(?,?,...)";
        //预编译
        PreparedStatement ps = connection.prepareStatement(sql);
        ps.setInt(xxx);
        ps.setString(xxx);
        //执行sql语句
        int result = ps.executeUpdate();//该方法用来执行 增，删，改的sql语句
        System.out.println("共" + result + "行受到影响");
        //关闭资源
        JDBCUtils.close(ps,connection);
    }
    @Test
    public void Test2()throws Exception{
    //改
        Connection connection = JDBCUtils.getConnection();
        String sql = "update xxx set name=? where id=?";
        PreparedStatement ps = connection.prepareStatement(sql);
        ps.setInt(xxx);
        ps.setString(xxx);
        int result = ps.executeUpdate();
        System.out.println("共" + result + "行受到影响");

        JDBCUtils.close(ps,connection);
    }
    @Test
    public void Test3()throws Exception{
        //删
        Connection connection = JDBCUtils.getConnection();
        String sql = "delete from xxxx";//全删
        PreparedStatement ps = connection.prepareStatement(sql);
        int result = ps.executeUpdate();
        System.out.println("共" + result + "行受到影响");
        JDBCUtils.close(ps,connection);
    }
}
```



# 档案

```java
user=root
password=123321
driverClassName=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/db?serverTimezone=UTC
```

# 使用DBUtils增和遍历

```java
public class DBUtilsDemo {
    @Test
    public void Test1() throws Exception {
        //创造对象
        QueryRunner qr = new QueryRunner();
        //插入一条数据
        int result = qr.update(JDBCUtils.getConnection(),"insert into eee(id,name) values(?,?)",3,"张三");
        System.out.println("共有" + result + "条数据受到影响");
    }
    @Test
    public void Test2()throws Exception{
        //查询
        QueryRunner qr = new QueryRunner();
        eee eee = qr.query(JDBCUtils.getConnection(), "select id,name from eee where id=?", new BeanHandler<eee>(eee.class), 1);
        System.out.println(eee);

    }
    @Test
    public void Test3()throws Exception{
        //查询所有
        QueryRunner qr = new QueryRunner();
        List<eee> eee = qr.query(JDBCUtils.getConnection(), "select id,name from eee", new BeanListHandler<eee>(eee.class));
        for (com.atguigu.jdbc.eee eee1 : eee) {
            System.out.println(eee1);
        }
    }
```

