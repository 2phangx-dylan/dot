## JDBC

- JDBC全称为Java Database Connectivity；

- 是Java语言中用来规范客户端程序如何来访问数据库的应用程序接口（API），提供了诸如查询、更新数据库数据的方法。JDBC也是Sun Microsystems的商标。我们通常说的JDBC是面向关系型数据库的。

## Built The Connection

### 基本流程

- **JDBCDemo01、JDBCDemo02**中显示了最基本的创建数据库连接并使用的示例，**Emp**为容器用于存储表数据：
  1. 导入驱动**jar**包，将项目需要的**mysql-connector-java-5.1.37-bin.jar**放入到目录中的**lib**包里，并添加；
  2. 注册驱动，使用**Class.forName("com.mysql.jdbc.Driver")**，**Class**类中的静态方法**forName(String class)**，加载**mysql-connector-java-5.1.37-bin.jar**中的**com.mysql.jdbc.Driver**类到内存中；
  3. 程序与数据库主要通过**Connection**的对象进行数据的交互，通过**java.sql.DriverManager**中的静态方法**getConnection()**，传入数据库的url、数据库登录用户名、数据库登录密码，即可获取到访问目标数据库的**Connection**的对象，<u>该类的对象在使用后需要进行资源释放</u>；
  4. 定义**sql**语句；
  5. **Statement**是执行**sql**语句的对象，通过**Connection**的对象调用**createStatement()**方法获得，<u>该类的对象在使用后需要进行资源释放</u>；
  6. **Statement**通过方法**executeUpdate(String sql)**可以执行**sql**语句进行相关的**DML**操作，方法返回一个**Int**类型数据，表示有多少行受影响；
  7. **Statement**通过方法**executeQuery(String sql)**可以执行**sql**语句进行相关的**DQL**操作，方法返回一个**ResultSet**，用于存储数据；
  8. **ResultSet**中提供了**next()**方法返回一个布尔值，类似于迭代器，当指针到达数据表末后一位（读取不到数据时），返回**false**，其余情况下能读取到数据返回**true**，<u>该类的对象在使用后需要进行资源释放</u>；
  9. **ResultSet**中提供了一系列**getXXX(String str)**的方法，用于获取表数据。
  10. 可以直接对获取到的数据进行使用，也可以创建一个用于封装数据的容器，通过容器对封装在其中的数据进行调用。

#### JDBCDemo01

```java
package cn.itcast.connector.jdbc;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;

public class JDBCDemo01 {
    public static void main(String[] args) {
        Connection conn = null;
        Statement stmt = null;
        try {
            // 1.导入驱动jar包
            // 2.注册驱动
            Class.forName("com.mysql.jdbc.Driver");
            // 3.获取数据库连接对象
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/db3", "root", "root");
            // 4.定义sql语句
            String sql = "UPDATE account SET balance = 500 WHERE id = 1";
            // 5.获取执行sql的对象Statement
            stmt = conn.createStatement();
            // 6.执行sql
            int count = stmt.executeUpdate(sql);
            // 7.处理结果
            System.out.println("受影响的行数为：" + count);

        } catch (ClassNotFoundException | SQLException e) {
            e.printStackTrace();
        } finally {
            // 8.释放资源
            if (stmt != null) {
                try {
                    stmt.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (conn != null) {
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

#### JDBCDemo02

```java
package cn.itcast.connector.jdbc;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

@SuppressWarnings("all")
public class JDBCDemo02 {
    public List<Emp> findAll() {
        List<Emp> empList = new ArrayList<Emp>();

        Connection conn = null;
        Statement stmt = null;
        ResultSet rs = null;

        try {
            Emp emp = null;

            Class.forName("com.mysql.jdbc.Driver");
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/db3", "root", "root");
            String sql = "SELECT * FROM emp";
            stmt = conn.createStatement();
            rs = stmt.executeQuery(sql);

            while (rs.next()) {
                int id = rs.getInt("id");
                String name = rs.getString("name");
                String gender = rs.getString("gender");
                double salary = rs.getDouble("salary");
                Date join_date = rs.getDate("join_date");
                int dept_id = rs.getInt("dept_id");

                emp = new Emp();

                emp.setId(id);
                emp.setName(name);
                emp.setGender(gender);
                emp.setSalary(salary);
                emp.setJoin_date(join_date);
                emp.setDept_id(dept_id);

                empList.add(emp);
            }
        } catch (ClassNotFoundException | SQLException e) {
            e.printStackTrace();
        } finally {
            if (rs != null) {
                try {
                    rs.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (stmt != null) {
                try {
                    stmt.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (conn != null) {
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
        return empList;
    }

    public static void main(String[] args) {
        List<Emp> list = new JDBCDemo02().findAll();
        for (Emp emp : list) {
            System.out.println(emp);
        }
    }
}
```

#### Emp

```java
package cn.itcast.connector.jdbc;

import java.util.Date;

public class Emp {
    private int id;
    private String name;
    private String gender;
    private double salary;
    private Date join_date;
    private int dept_id;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gentder) {
        this.gender = gentder;
    }

    public double getSalary() {
        return salary;
    }

    public void setSalary(double salary) {
        this.salary = salary;
    }

    public Date getJoin_date() {
        return join_date;
    }

    public void setJoin_date(Date join_date) {
        this.join_date = join_date;
    }

    public int getDept_id() {
        return dept_id;
    }

    public void setDept_id(int dept_id) {
        this.dept_id = dept_id;
    }

    @Override
    public String toString() {
        return "Emp{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", gentder='" + gender + '\'' +
                ", salary=" + salary +
                ", join_date=" + join_date +
                ", dept_id=" + dept_id +
                '}';
    }
}
```

### 注意事项

- 注意：使用**Statement**可能会造成**sql**注入问题：
  - 使用**Statement**需要传入拼接数据的**sql**语句，在拼接**sql**语句的时如果遇到一些特殊的关键字，会造成安全性问题；
  - 官方提供了一个方法，使用**PreparedStatement**代替**Statement**进行数据库查询：
    1. **sql**语句需要在获取**PreparedStatement**对象的时候一并作为参数传入**Connection**对象的**prepareStatement(String sql)**中；
    2. 其中**sql**语句里的参数使用"?"代替；
    3. 调用**PreparedStatement**中一系列的**setXXX(int parameter, XXX xxx)**进行参数设置，索引从1开始；
    4. 执行增删改查语句的时候，只需要调用**PreparedStatement**中的**excuteUpdate()**、**excuteQuery()**即可，不再需要传入参数。

#### JDBCDemo04

```java
package cn.itcast.connector.jdbc;

import cn.itcast.connector.util.JDBCUtils;

import java.sql.*;
import java.util.Scanner;

/**
 * 依赖配置文件：D:\IdeaProjects\basic-code\03-jdbc\src\jdbc.properties
 * 当前查询的目标表位于数据库db5
 *
 * 1. 通过键盘录入用户名和密码
 * 2. 判断用户是否登录成功
 */
public class JDBCDemo04 {
    public static void main(String[] args) {
        // 1.从控制台得到用户键入的用户名和密码
        Scanner sc = new Scanner(System.in);
        System.out.print("请键入你的用户名：");
        String username = sc.nextLine();
        System.out.print("请键入你的密码：");
        String password = sc.nextLine();

        // 2.判断用户键入的信息是否正确
        boolean login = new JDBCDemo04().login_b(username, password);
        // 3.返回用户登录结果
        if (login) {
            System.out.println("登陆成功！");
        } else {
            System.out.println("用户名或密码错误！");
        }
    }

    /**
     * 用于判断用户是否登录成功，使用Statement执行sql语句
     *
     * @param username 用户名
     * @param password 登录密码
     * @return 登录结果
     */
    private boolean login_a(String username, String password) {
        if (username == null || password == null) {
            return false;
        }
        Connection conn = null;
        Statement stmt = null;
        ResultSet rs = null;

        try {
            String sql = "SELECT * FROM user WHERE username = '" + username + "' AND password = '" + password + "'";

            conn = JDBCUtils.getConnection();
            stmt = conn.createStatement();
            rs = stmt.executeQuery(sql);
            return rs.next();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JDBCUtils.close(rs, stmt, conn);
        }

        return false;
    }
    /**
     * 用于判断用户是否登录成功，使用PreparedStatement执行sql语句
     *
     * @param username 用户名
     * @param password 登录密码
     * @return 登录结果
     */

    private boolean login_b(String username, String password) {
        if (username == null || password == null) {
            return false;
        }

        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;

        try {
            connection = JDBCUtils.getConnection();

            String sql = "select * from user where username = ? and password = ?";

            preparedStatement = connection.prepareStatement(sql);
            preparedStatement.setString(1, username);
            preparedStatement.setString(2, password);

            resultSet = preparedStatement.executeQuery();

            return resultSet.next();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JDBCUtils.close(resultSet, preparedStatement, connection);
        }
        return false;
    }
}
```

#### JDBCUtils

```java
package cn.itcast.connector.util;

import java.io.IOException;
import java.io.InputStream;
import java.sql.*;
import java.util.Properties;

public class JDBCUtils {
    private static String url;
    private static String user;
    private static String password;
    private static String driver;

    static {
        try {
            Properties pro = new Properties();

            InputStream is = JDBCUtils.class.getClassLoader().getResourceAsStream("jdbc.properties");

            pro.load(is);

            url = pro.getProperty("url");
            user = pro.getProperty("user");
            password = pro.getProperty("password");
            driver = pro.getProperty("driver");

            Class.forName(driver);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    /**
     * 用于获取Connection对象
     *
     * @return 返回Connection对象
     * @throws SQLException
     */
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(url, user, password);
    }

    /**
     * 用于释放资源
     *
     * @param stmt Statement对象
     * @param conn Connection对象
     */
    public static void close(Statement stmt, Connection conn) {
        if (stmt != null) {
            try {
                stmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 用于释放资源
     *
     * @param rs   ResultSet对象
     * @param stmt Statement对象
     * @param conn Connection对象
     */
    public static void close(ResultSet rs, Statement stmt, Connection conn) {
        if (rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (stmt != null) {
            try {
                stmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (conn != null) {
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```

#### jdbc.properties

```properties
# DB Configuration:
url=jdbc:mysql://localhost:3306/db5
user=root
password=root
driver=com.mysql.jdbc.Driver
```

## Transaction

- 事务管理对于数据库来说至关重要，Connection中提供了对数据库进行事务管理的方法：
  - 开启事务：setAutoCommit(boolean autoCommit)
    - 该方法默认参数为true，需要开启事务则设置参数为false
    - 建议在创建连接后开启事务
  - 提交事务：commit()
    - 当所有的sql语句都执行完后，调用commit()进行事务的提交
  - 回滚事务：rollback()
    - 当程序出现任何的异常时，都需要进行事务的回滚操作
    - catch体中抓取的异常最好为Exception
    - 因为是在连接建立之后开启的事务，所以需要判断连接是否正常建立，再判断是否需要回滚

## DataSource

### c3p0

- 依赖**c3p0-0.9.5.5.jar**和**mchange-commons-java-0.2.19.jar**，可以去**Maven**网站上下载，挂个**vpn**吧；
- 不要忘记了仍旧需要**mysql-connector-java-8.0.16.jar**，终究是要获取数据库驱动的；
- 每次连接数据库都要创建关闭**Connection**对象会浪费系统的资源，**Java**提供了**DataSource**接口，供各大数据库厂家创建他们的连接池对象，但一般会使用第三方的连接池对象，因为这个接口很简单，常用的有**c3p0**和**Druid**；
- 有一个莫大的疑点是：由数据库连接池创建的**Connection**对象调用**close()**方法的时候，连接会归还给连接池，但是为什么，对象还是那个**Connection**的对象，调用的方法却不是那个可以把连接对象关闭的方法，日后解决。（使用了代理的方式增强了close()方法）

#### CPDSDemo

```java
package cn.itcast.c3p0;

import cn.itcast.connector.util.JDBCUtils;
import com.mchange.v2.c3p0.ComboPooledDataSource;

import java.sql.*;

/**
 * 每次连接数据库都要创建关闭Connection对象会浪费系统的资源，Java提供了DataSource接口，
 * 供各大数据库厂家创建他们的连接池对象，但一般会使用第三方的连接池对象，因为这个接口很
 * 简单，常用的有c3p0和Druid。
 */
public class CPDSDemo {
    public static void main(String[] args) {

        // 创建连接池对象
        ComboPooledDataSource cpds = new ComboPooledDataSource();

        Connection cn = null;
        PreparedStatement pstat = null;
        ResultSet rs = null;

        try {
            // 从连接池获取Connection对象
            cn = cpds.getConnection();
            String sql = "select * from user";
            // 传入sql语句获取preparedStatement对象
            pstat = cn.prepareStatement(sql);
            // 获取ResultSet对象
            rs = pstat.executeQuery();
			// 获取数据并打印到控制台
            while (rs.next()) {
                int id = rs.getInt("id");
                String username = rs.getString("username");
                String password = rs.getString("password");

                System.out.println(username + "'s id is " + id + " with password " + password);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JDBCUtils.close(rs, pstat, cn);
        }
    }
}
```

#### c3p0.xml

```xml
<c3p0-config>
    <default-config>
        <!-- DB Configuration -->
        <property name="classDriver">com.mysql.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://localhost:3306/db5</property>
        <property name="user">root</property>
        <property name="password">root</property>

        <!-- 初始化连接池数量、最大连接对象数量 -->
        <property name="initialPoolSize">5</property>
        <property name="maxPoolSize">10</property>
        <property name="checkoutTimeout">1000</property>
    </default-config>

    <!-- This app is massive! -->
    <named-config name="intergalactoApp">
        <property name="acquireIncrement">50</property>
        <property name="initialPoolSize">100</property>
        <property name="minPoolSize">50</property>
        <property name="maxPoolSize">1000</property>

        <!-- intergalactoApp adopts a different approach to configuring statement caching -->
        <property name="maxStatements">0</property>
        <property name="maxStatementsPerConnection">5</property>

        <!-- he's important, but there's only one of him -->
        <user-overrides user="master-of-the-universe">
            <property name="acquireIncrement">1</property>
            <property name="initialPoolSize">1</property>
            <property name="minPoolSize">1</property>
            <property name="maxPoolSize">5</property>
            <property name="maxStatementsPerConnection">50</property>
        </user-overrides>
    </named-config>
</c3p0-config>
```

### druid

- 由阿里巴巴提供的数据库连接池，它的配置文件为properties类型的，名字可以自定义，因此需要在创建连接池对象的时候指定文件；

#### DDSFDemo

```java
package cn.itcast.druid;

import cn.itcast.connector.util.JDBCUtils;
import com.alibaba.druid.pool.DruidDataSourceFactory;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.Properties;

/**
 * 演示了数据库连接池Druid的使用方式。
 */
public class DDSFDemo {
    public static void main(String[] args) throws Exception {
        // 配置文件需要解析到Properties中
        Properties properties = new Properties();
        properties.load(DruidDataSourceFactory.class.getClassLoader().getResourceAsStream("druid.properties"));

        // 使用DruidDataSourceFactory中的createDataSource获取DataSource
        DataSource ds = DruidDataSourceFactory.createDataSource(properties);
       
        Connection cn = null;
        PreparedStatement pstat = null;
        ResultSet rs = null;

        try {
            // 从连接池获取Connection对象
            cn = ds.getConnection();
            String sql = "select * from user";
            // 传入sql语句获取preparedStatement对象
            pstat = cn.prepareStatement(sql);
            // 获取ResultSet对象
            rs = pstat.executeQuery();

            while (rs.next()) {
                int id = rs.getInt("id");
                String username = rs.getString("username");
                String password = rs.getString("password");

                System.out.println(username + "'s id is " + id + " with password " + password);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JDBCUtils.close(rs, pstat, cn);
        }
    }
}
```

#### druid.properties

```properties
# DB Configuration:
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/db5
username=root
password=root
initialSize=5
maxActive=10
maxWait=3000
```

## JdbcTemplate

- **Spring**框架对**JDBC**的简单封装。提供了一个**JDBCTemplate**对象简化了**JDBC**的开发。
  - 仍旧需要提供**druid.properties**，因为需要向**JDBCTemplate**提供**DataSource**对象；
  - 那么就需要把以下的**jar**包添加到项目或模块的**lib**中：
    1. **spring-beans-5.2.5.RELEASE.jar**
    2. **spring-core-5.2.5.RELEASE.jar**
    3. **spring-jdbc-5.2.5.RELEASE.jar**
    4. **spring-tx-5.2.5.RELEASE.jar**
    5. **commons-logging-1.1.1.jardruid-1.1.20.jar**

### **simple**

#### NormalJdbcTemplate

```java
package jdbc;

import org.junit.Test;
import org.springframework.jdbc.core.JdbcTemplate;
import util.JDBCUtils;

import java.util.List;
import java.util.Map;

/**
 * JBDCTemplate中基本的方法。
 */
public class NormalJdbcTemplate {
    private JdbcTemplate jdbcTemplate = new JdbcTemplate(JDBCUtils.getDataSource());

    /**
     * update(String sql);
     *
     * 测试DML语句，用于增删改表数据。
     */
    @Test
    public void test_a() {
        String sql = "update emp set salary = 6000 where id = 1";
        jdbcTemplate.update(sql);
    }

    /**
     * queryForMap(sql[, Object... args]);
     *
     * 此方法只会返回一个Map集合，如果需要返回多个，使用queryForList(sql[, Object... args]);
     */
    @Test
    public void test_b() {
        String sql = "select * from emp where id = ?";
        Map<String, Object> queryForMap = jdbcTemplate.queryForMap(sql, 4);
        System.out.println(queryForMap);
    }

    /**
     * queryForList(sql[, Object... args]);
     *
     * 此方法返回一个List<Map<String, Object>>，自己体会。
     */
    @Test
    public void test_c() {
        String sql = "select * from emp where id = ? or gender = ?";
        List<Map<String, Object>> queryForList = jdbcTemplate.queryForList(sql, 5, "男");
        for (Map<String, Object> stringObjectMap : queryForList) {
            System.out.println(stringObjectMap);
        }
    }

    /**
     * public <T> T queryForObject(String sql, Class<T> requiredType);
     *
     * 此方法一般使用聚合函数的时候调用，需要传入sql语句，以及期望的返回值类型的类对象Class。
     */
    @Test
    public void test_d() {
        String sql = "select count(id) from emp";
        Long queryForObject = jdbcTemplate.queryForObject(sql, Long.class);
        System.out.println(queryForObject);
    }
}
```

#### pojo: Emp

```java
package pojo;

import java.sql.Date;

/**
 * 了解到实体类的名字要和数据库表的字段名一致，另外数据库表如果使用下划线，对应的数据表类中的成员属性名使用大写。
 */
public class Emp {
    private Integer id;
    private String name;
    private String gender;
    private Double salary;
    private Date joinDate;
    private Integer deptId;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public Double getSalary() {
        return salary;
    }

    public void setSalary(Double salary) {
        this.salary = salary;
    }

    public Date getJoinDate() {
        return joinDate;
    }

    public void setJoinDate(Date joinDate) {
        this.joinDate = joinDate;
    }

    public Integer getDeptId() {
        return deptId;
    }

    public void setDeptId(Integer deptId) {
        this.deptId = deptId;
    }

    @Override
    public String toString() {
        return "Emp{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", gender='" + gender + '\'' +
                ", salary=" + salary +
                ", joinDate=" + joinDate +
                ", deptId=" + deptId +
                '}';
    }
}
```

#### util: JDBCUtils

```java
package util;

import com.alibaba.druid.pool.DruidDataSourceFactory;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.Properties;

/**
 * 工具类，数据库连接池。本类只是提供工具类产生DataSource和Connection对象，不编写close()方法。
 */
public class JDBCUtils {
    private static DataSource ds = null;

    static {
        try {
            Properties properties = new Properties();
            properties.load(JDBCUtils.class.getClassLoader().getResourceAsStream("druid.properties"));
            ds = DruidDataSourceFactory.createDataSource(properties);
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    public static Connection getConnection() {
        try {
            return ds.getConnection();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;
    }

    public static DataSource getDataSource() {
        return ds;
    }
    public static void close() {
        // 使用JDBCTemplate不需要定义关闭的方法，JDBCTemplate会自动帮程序处理并释放。
    }
}
```

### complex

#### ImportantJdbcTemplate

```java
package jdbc;

import org.junit.Test;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import pojo.Emp;
import util.JDBCUtils;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

public class ImportantJdbcTemplate {
    private JdbcTemplate jdbcTemplate = new JdbcTemplate(JDBCUtils.getDataSource());

    /**
     * public <T> List<T> query(String sql, RowMapper<T> rowMapper)
     *
     * 可以在query中实现RowMapper接口，只需要覆写里面的一个方法，可以使用lambda表达式。
     *
     * RowMapper中的mapRow方法中提供一行的数据供给用户获取值，该方法要求返回一个T对象，返回的对象会在
     * 整个过程中被封装成一个List<T>，作为query的返回值。
     *
     * 但同样免不了需要从ResultSet中获取值再给T进行set的操作，操作麻烦。
     */
    @Test
    public void test_a() {
        String sql = "select * from emp";
        List<Emp> empList = jdbcTemplate.query(sql, new RowMapper<Emp>() {
            @Override
            public Emp mapRow(ResultSet rs, int i) throws SQLException {
                Emp emp = new Emp();

                emp.setId(rs.getInt("id"));
                emp.setName(rs.getString("name"));
                emp.setGender(rs.getString("gender"));
                emp.setSalary(rs.getDouble("salary"));
                emp.setJoinDate(rs.getDate("join_date"));
                emp.setDeptId(rs.getInt("dept_id"));

                return emp;
            }
        });
        for (Emp emp : empList) {
            System.out.println(emp);
        }
    }

    /**
     * public class BeanPropertyRowMapper<T> implements RowMapper<T>;
     * public <T> List<T> query(String sql, RowMapper<T> rowMapper);
     *
     * 第二种我们采用接口的实现类，只需要知道T是什么，并传入T.class作为其有参构造的参数，即可获取相应的List<T>。
     *
     * 注意的是，pojo中的实现类的成员变量名必须要在符合规则的情况下，与数据表中的字段形成映射。
     */
    @Test
    public void test_b() {
        String sql = "select * from emp";
        List<Emp> empList = jdbcTemplate.query(sql, new BeanPropertyRowMapper<Emp>(Emp.class));
        for (Emp emp : empList) {
            System.out.println(emp);
        }
    }
}
```