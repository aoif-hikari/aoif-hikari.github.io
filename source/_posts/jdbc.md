---
title: JDBC
tags: java
---

> Java DataBase Connectivity, Java连接数据库，是SUN公司制定的一套接口, java.sql.*;
>
> 后期将集成到Mybatis框架中。

<!-- more -->

## 使用JDBC：

从官网下载对应的驱动jar包，将其配置到环境变量classpath。

classpath=.;D:\course\06-JDBC\resources\MySql Connector Java 5.1.23\mysql-connector-java-5.1.23-bin.jar

### 编程

```java
public static void main(String[] args) {
    Connection conn = null;
    Statement stmt = null;
    try{
        // 1、注册驱动
        Driver driver = new com.mysql.jdbc.Driver();	//多态，父类型引用指向子类型对象
        DriverManager.registerDriver(driver); //静态方法

        //注册驱动的另一种方法
        //Class.forName(driver);
        //不用接受返回值，只想用类加载这个操作，执行静态代码块完成注册驱动

        // 2、获取连接
        /*
				url包括哪几部分:协议，IP，Port，资源名				
				eg：http://180.101.49.11:80/index.html
			http:// 通信协议; 180.101.49.11 IP地址; 80 端口号; index.html 资源名
			*/
        // 
        String url = "jdbc:mysql://127.0.0.1:3306/mydatabase";
        String user = "root";
        String password = "146";
        // static Connection getConnection(String url, String user, String password)
        conn = DriverManager.getConnection(url,user,password); 

        //System.out.println("数据库连接对象" + conn);	
        //数据库连接对象com.mysql.jdbc.JDBC4Connection@1ae369b7

        // 3、获取数据库操作对象
        // Statement createStatement() 创建一个 Statement 对象来将 SQL 语句发送到数据库。
        stmt = conn.createStatement();

        // 4、执行sql语句
        // int executeUpdate(String sql) 
        // 专门执行DML语句,返回值是“影响数据库中的记录条数”
        // 这里的sql语句不用;
        int count = stmt.executeUpdate("update dept set dname = '销售部',
                                       loc = '合肥' where deptno = 20;");
        //System.out.println(count == 1 ? "保存成功":"保存失败");
		// 5、处理查询结果集
        } catch(SQLException e) {
            e.printStackTrace();
        } finally {
            // 6、释放资源, 从小到大依次关闭
            // 释放Statement
            if(stmt != null) {
                try	{
                    stmt.close();	
                }
                catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            //释放Connection
            if(conn != null) {
                try	{
                    conn.close();	
                }
                catch (SQLException e) {
                    e.printStackTrace();
                }
			}
		}
	}
```

### 处理返回结果

executeUpdate：insert/update/delete, 返回结果对表文件操作时，受影响行数

executeQuery：查询命令【select  *  from 表名】，返回结果是查询命令得到【临时表】，ResultSet实例对象。

```java
ResultSet rs = null;
rs = ps.executeQuery();
//rs = stmt.executeQuery("select empno,ename,sal from emp");
//遍历查询到的结果
while(rs.next()){
    //按顺序取
    String empno = rs.getString(1);
    String ename = rs.getString(2);
    String sal = rs.getString(3);
    System.out.println(empno + "," + ename + "," + sal);
	//按名称取出
    String empno = rs.getString("empno");
    String ename = rs.getString("ename");
    String sal = rs.getString("sal");
    System.out.println(empno + "," + ename + "," + sal);
	//类型+顺序
    int empno = rs.getInt(1);
    String ename = rs.getString(2);
    double sal = rs.getDouble(3);
    System.out.println(empno + "," + ename + "," + (sal + 100));
	//类型+名称
    int empno = rs.getInt("empno");
    String ename = rs.getString("ename");
    double sal = rs.getDouble("sal");
    System.out.println(empno + "," + ename + "," + (sal + 200));
}
```

### 使用动态资源绑定器传参

```java
ResourceBundle bundle = ResourceBundle.getBundle("jdbc"); //.properties后缀不用写
String driver = bundle.getString("driver");
String url = bundle.getString("url");
String user = bundle.getString("user");
String password = bundle.getString("password");
//...
Connection conn = null;
conn = DriverManager.getConnection(url,user,password);
```

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mydatabase
user=root
password=146
```

### sql注入

问题：用户输入语句含有sql语句关键字，完成了sql语句的拼接，参与编译，导致原sql语句含义被扭曲。

解决方法：用户提供的信息不参与编译过程，用java.sql.PreparedStatement（继承java.sql.Statement）

   * PreparedStatement属于预编译的数据库操作对象，原理：预先对sql语句的框架进行编译，再给sql语句传“值”，使用较多。
   * PreparedStatement效率高。只用预编译一次，可执行多次。在编译阶段做安全检查（ps.setString()）Statement执行一次编译一次
   * 要求支持sql语句拼接时使用Statement。（如升/降序是在语句末尾拼接asc/desc）

```java
//使用PreparedStatement模拟登录，防止sql注入现象
Connection conn = null;
// Statement stat = null;
PreparedStatement ps = null;
ResultSet rs = null;
try {
    // 1、注册驱动
    Class.forName("com.mysql.jdbc.Driver");
    // 2、获取连接
    conn = DriverManager.getConnection(
        "jdbc:mysql://localhost:3306/mydatabase",
        "root",
        "146");
    // 3、获取预编译的数据库操作对象
    // sql语句的框架中，一个?，表示一个占位符，一个?将来接收一个值。注意：?不用单引号括起来
    String sql = "select * from t_user where userName = ? and userPassword = ?";
    // 程序执行到此处，会发送sql语句框架给DBMS，DBMS对sql语句框架进行预编译。
    ps = conn.prepareStatement(sql);
    // 给占位符?传值，第一个?的下标是1，第二个?的下标是2（JDBC中下标都从1开始）
    ps.setString(1,userLoginInfo.get("userName"));
    ps.setString(2,userLoginInfo.get("userPassword"));
    // 4、执行sql语句
    rs = ps.executeQuery();
    // 5、处理结果集
    if(rs.next()) {
        loginSuccess = true;
    }
} catch (ClassNotFoundException e) {
    e.printStackTrace();
} catch (SQLException throwables) {
    throwables.printStackTrace();
} finally {
    // 6、释放资源
    if (rs != null) {
        try {
            rs.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }
    if (ps != null) {
        try {
            ps.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }
    if (conn != null) {
        try {
            conn.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }
}
```

```java
//升降序排列时需要sql注入
import java.sql.*;
import java.util.Scanner;

Scanner s = new Scanner(System.in);
System.out.println("请输入desc或者asc");
String keyWords = s.nextLine();

Connection conn = null;
Statement stmt = null;
ResultSet rs = null;

try {
    Class.forName("com.mysql.jdbc.Driver");

    conn = DriverManager.getConnection(
        "jdbc:mysql://localhost:3306/mydatabase",
        "root",
        "146");
    
	//createStatement可以完成拼接
    stmt = conn.createStatement();
    String sql = "select ename from emp order by ename " + keyWords;
    rs = stmt.executeQuery(sql);

    while(rs.next()){
        System.out.println(rs.getString("ename"));
    }

} catch (ClassNotFoundException e) {
    e.printStackTrace();
} catch (SQLException throwables) {
    throwables.printStackTrace();
} finally {
    if (rs != null) {
        try {
            rs.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }
    if (stmt != null) {
        try {
            rs.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }
    if (conn != null) {
        try {
            rs.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }
}
}
```

### JDBC事务自动提交

即执行任一条DML语句，则自动提交一次。

```java
//conn.setAutoCommit(false); // 开启事务
//conn.commit(); // 提交事务
//conn.rollback(); // 回滚

public static void main(String[] args) {
    Connection conn = null;
    PreparedStatement ps = null;
    try {
        // 注册驱动
        Class.forName("com.mysql.jdbc.Driver");

        // 获取连接
        conn = DriverManager.getConnection(
            "jdbc:mysql://localhost:3306/mydatabase",
            "root",
            "146");

        // ---------------将自动提交改为手动提交------------------------------
        conn.setAutoCommit(false); // 开启事务
        //-----------------------------------------------------------------

        // 获取预编译的数据库操作对象
        String sql = "update t_act set balance = ? where actno = ? ";
        ps = conn.prepareStatement(sql);
        ps.setInt(1,10000);
        ps.setDouble(2,111);

        // 执行sql语句
        int count = ps.executeUpdate();

        /*
            String s = null;
            s.toString();
            */

        ps.setInt(1,10000);
        ps.setDouble(2,222);
        count += ps.executeUpdate();

        System.out.println(count == 2 ? "转账成功" : "转账失败");

        // -----------程序能执行到此处，说明没有异常，事务结束，手动提交数据----------------------
        conn.commit(); // 提交事务
        //-------------------------------------------------------------------------------
        
    } catch (Exception e) {
        // --------遇到异常，回滚-------------------
        if (conn != null) {
            try {
                conn.rollback(); // 回滚
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
        e.printStackTrace();
        //-----------------------------------------
    }  finally {
        // 释放资源
        if (ps != null) {
            try {
                ps.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
        if (conn != null) {
            try {
                conn.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
    }
}
```

### 锁

行级锁（悲观锁）:for update 当前事务结束之前，行结果被锁住，无法修改。事务排队进行，不允许并发
乐观锁：多线程并发，事务不需要排队，都可以修改，需要版本号。

### DAO与Entity

DAO(Data Access Object) 数据访问对象，将数据库操作都封装起来。包括：

1）实体类Dept：用于存放与传输对象数据。

2）数据库连接和关闭工具类JdbcUtil： 避免了数据库连接和关闭代码的重复使用，方便修改。

3）DAO 实现类DeptDao： 针对不同数据库给出DAO接口定义方法的具体实现。

```java
// 实体类（一张表对应一个实体，用于描述表结构，一个实例对象对应表中一个数据行，属性与表中字段保持一致）
package com.bjpowernode.entity;
public class Dept {
    private Integer deptNo;
    private String  dname;
    private String  loc;

    public Integer getDeptNo() {return deptNo;}
    public void setDeptNo(Integer deptNo) {this.deptNo = deptNo;}
    public String getDname() {return dname;}
    public void setDname(String dname) {this.dname = dname;}
    public String getLoc() {return loc;}
    public void setLoc(String loc) {this.loc = loc;}
    
    public Dept() {
    }
    public Dept(Integer deptNo, String dname, String loc) {
        this.deptNo = deptNo;
        this.dname = dname;
        this.loc = loc;
    }
}

//数据库连接和关闭工具类
public class JdbcUtil {
    private  Connection con = null;//类文件属性，可以在类文件中所有的方法中使用
    private  PreparedStatement ps=null;//类文件属性，可以在类文件中所有的方法中使用
    //静态语句块 static{}
    //在当前类文件第一次被加载到JVM时，JVM将会自动调用当前类文件静态语句块
    static{
        //1.注册数据库服务器提供的Driver接口实现类
        try {
            Class.forName("com.mysql.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
    //封装Connection对象创建细节 不需要考虑使用对象创建细节
    public  Connection  createCon(){
        try {
            con = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/bjpowernode", "root", "123");
        } catch (SQLException e) {
            e.printStackTrace();
            System.out.println("Connection对象创建失败");
        }
        return con;
    }
    //封装PreparedStatement对象创建细节
    public PreparedStatement createStatement(String sql){
        Connection con = createCon();
        try {
            ps = con.prepareStatement(sql);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return ps;
    }
    //封装PreparedStatement对象与Connection对象销毁细节
    public void close(){
        if(ps!=null){
            try {
                ps.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if(con!=null){
            try {
                con.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    //封装PreparedStatement对象与Connection对象与ResultSet对象销毁细节
    public void close(ResultSet rs){
        if(rs!=null){
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        close();

    }
}

// DAO接口实现（命名为表名+Dao，实现CRUD功能）
public class DeptDao {
    private  JdbcUtil util = new JdbcUtil();
    //添加数据行
    public int add(String deptNo,String dname,String loc){
        String sql="insert into dept (deptNo,dname,loc) values(?,?,?)";
        int result=0;
        PreparedStatement ps = util.createStatement(sql);
        try {
            ps.setInt(1, Integer.valueOf(deptNo));
            ps.setString(2, dname);
            ps.setString(3, loc);
            result=ps.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            util.close();
        }
        return result;
    }

    //删除数据行
    public int delete(String deptNo){
        String sql ="delete from dept where deptno=?";
        PreparedStatement ps = util.createStatement(sql);
        int result = 0;
        try {
            ps.setInt(1, Integer.valueOf(deptNo));
            result = ps.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            util.close();
        }
        return result;
    }

    //更新数据行
    public int update(String deptNo,String dname,String loc){
        String  sql ="update dept set dname=?,loc=? where deptno=?";
        PreparedStatement ps = util.createStatement(sql);
        int result=0;
        try {
            ps.setString(1, dname);
            ps.setString(2, loc);
            ps.setInt(3, Integer.valueOf(deptNo));
            result = ps.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
           util.close();
        }
        return result;
    }

    //查询数据行
    public List findAll(){
        String sql ="select * from dept";
        PreparedStatement ps = util.createStatement(sql);
        ResultSet rs = null;
        List list = new ArrayList();
        try {
            rs = ps.executeQuery();
            //将是临时表数据行转换为实体类实例对象保管
            while(rs.next()){
                int deptNo = rs.getInt("deptno");
                String dname = rs.getString("dname");
                String loc = rs.getString("loc");
                Dept dept = new Dept(deptNo, dname, loc);
                list.add(dept);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            util.close(rs);
        }
        return list;
    }
}
```

