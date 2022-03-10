# AOP概念和作用

1. 什么是AOP?
   - 在软件业，AOP为Aspect Oriented Programming的缩写，意为：面向切面编程，通过预编译方式和运行期间动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率
2. AOP的作用及优势
   - 作用：在程序运行期间，不修改源码对已有方法进行增强；
   - 优势：
     1. 减少重复代码；
     2. 提高开发效率；
     3. 维护方便。
3. AOP的实现方式
   - 使用动态代理技术。
4. 【个人观点】AOP的意义所在
   - 不仅可以实现对一个类中的方法进行动态代理，而且可以做到对一个包中的所有类中的方法进行统一的动态代理；
   - 传统的动态代理操作，往往需要建立一个动态代理类或在业务层中进行动态代理操作，但这都涉及到了修改源码的问题，且一个动态代理类中，如果想要对多个业务层的类进行代理操作，需要编写多个动态代理的方法，效率是不高的；
   - 而通过使用AOP，可以使一套代码通过简单的xml配置或配置类，实现对包中所有类的所有方法的动态代理，大大减少了代码重复，提高了开发的效率，同时也使得后期的维护更加方便。

# 动态代理实现事务管理

1. 基于接口的动态代理
   - 提供者：JDK官方的Proxy类。
   - 要求：被代理类最少实现一个接口。
2. 基于子类的动态代理
   - 提供者：第三方CGLib，如果出现asmxxxx异常，需要导入asm.jar。
   - 要求：被代理类不能final修饰的类（最终类）。

## 关于动态代理的使用细节

1. 事务必须是在同一个Connection对象下，才可以使用动态代理进行管理；
2. Spring JdbcTemplate每调用一次update()或query()方法都会向DataSource要一个新的connection数据库连接对象；
3. 由于上一条的原因，在Spring JdbcTemplate中，使用JdbcTemplate.getDataSource().getConnection()得到的connection对象，与在dao中使用的connection对象不是同一个connection对象；前者connection对象调用方法setAutoCommit(flase)，对后者connection对象的增删改查（CURD）操作没有的任何影响；
4. 如果dao中使用的是JdbcTemplate进行CURD操作了，我们将无法保证使用JdbcTemplate总是获取到同一个connection，那么只能作以下选择：
   1. 创建一个connection的Bean，在dao中使用connection对象来自行编写CURD操作，采用动态代理中编写事务控制代码的方式，对事务进行管理；
   2. **将事务交由Spring管理，采用动态代理中编写事务控制代码的方式，对事务进行管理。**
5. Spring的事务管理依赖于TransactionTemplate，TransactionTemplate的创建依赖于DataSourceTransactionManager，DataSourceTransactionManager则依赖于DataSource；
6. TransactionTemplate可以调用方法execute()，使用匿名内部类的方式覆写TransactionCallback类中的doInTransaction方法，同时作为参数传入execute()，事务已交由Spring控制，仅需要将业务层代码编写在doInTransaction方法中即可。

## TransactionManager的事务管理

### 动态代理前的准备事项

- 此TransactionManager管理类为用户自定义的类，可以使用此类对单例connection对象进行事务管理；
- 使用Spring Ioc配置类创建connection的Bean，此connection对象时单例的；
- TransactionManager的对象中release()方法中，需要设置connection.setAutoCommit(true)，因为此Connection连接对象的下一次的CURD操作未必是需要进行事务管理的（例：只是执行一条sql语句），所以需要在release()中重新设置事务提交的方式。（尽管多数都是统一对业务层实现类中所有方法进行代理）
- 在Intellij IDEA中${username}使用特殊含义的，避免在@Value注解中使用。



#### ConnectionConfig.java

- 用于配置唯一的Connection对象进行事务管理

```java
package cn.dylanphang.aopuserdefinedconnection.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
/**
 * 使用此配置类创建connection的Bean。
 *
 * 在Intellij IDEA中${username}使用特殊含义的，避免在@Value中使用。
 */
@Configuration
@PropertySource("classpath:druid.properties")
public class ConnectionConfig {

    @Value("${conn.driverClassName}")
    private String driver;
    @Value("${conn.url}")
    private String url;
    @Value("${conn.username}")
    private String username;
    @Value("${conn.password}")
    private String password;

    @Bean("connection")
    public Connection createConnection() {
        Connection conn = null;
        try {
            Class.forName(driver);
            conn = DriverManager.getConnection(url, username, password);
            return conn;
        } catch (ClassNotFoundException | SQLException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```
#### TransactionManager.java

- （UserDefined）用户自定义的TransactionManager类，配置唯一的Connection对象，用于事务管理。

```java
package cn.dylanphang.aopuserdefinedconnection.transaction;


import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.sql.Connection;
import java.sql.SQLException;

/**
 * 使用此类来控制事务。
 */
@Component("transactionManager")
public class TransactionManager {

    private Connection connection;

    @Autowired
    public void setConnection(Connection connection) {
        this.connection = connection;
    }

    public void beginTransaction() {
        try {
            connection.setAutoCommit(false);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public void commit() {
        try {
            connection.commit();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public void rollback() {
        try {
            connection.rollback();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public void release() {
        try {
            connection.setAutoCommit(true);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

### JDK官方的Proxy类创建动态代理

- 使用动态代理对UserService的实现类进行代理，在invoke()方法中调用TransactionManager中的方法来进行事务管理，详情参见代码；
- 方法中的Connection对象已经交由Spring管理了，即使Connection对象调用close()方法，也是没有办法销毁它的，此时Connection的生命周期参照Spring Ioc中单例Bean的生命周期；
- Connection对象close()方法的失效，猜测是Spring对该方法进行了代理，并在invoke中什么都不覆写。



#### UserServiceFactory.java

- 注入的UserService对象实际上是UserServiceImpl（实现类），接收Proxy.newProxyInstance返回值的时候，同样使用多态将Object类型强转为UserService类型；
- Proxy类会自动对invoke方法返回的Object对象进行类型的转换，会自动转为原本方法的返回值类型，因此在invoke中不需要考虑类型转换的问题。
- 实际上，可以将getUserService方法的返回值作为一个Bean，只需要在该方法上添加@Bean注解，并配上id，即可在表现层中直接获取该Bean对象。（AOP章节将采用此方式）

```java
package cn.dylanphang.aopuserdefinedconnection.proxy;

import cn.dylanphang.aopuserdefinedconnection.service.UserService;
import cn.dylanphang.aopuserdefinedconnection.transaction.TransactionManager;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.lang.reflect.Proxy;

@Component("userServiceFactory")
public class UserServiceFactory {

    @Autowired
    private UserService userService;
    
    @Autowired
    private TransactionManager transactionManager;
    
    public UserService getUserService() {
        return (UserService) Proxy.newProxyInstance(userService.getClass().getClassLoader(),
                userService.getClass().getInterfaces(),
                ((proxy, method, args) -> {
                    try {
                        // 1.开启事务
                        transactionManager.beginTransaction();
                        
                        // 2.执行CURD操作
                        Object result = method.invoke(userService, args);
                        
                        // 3.提交事务
                        transactionManager.commit();
                        return result;
                    } catch (Exception e) {
                        e.printStackTrace();
                        // 4.回滚事务
                        transactionManager.rollback();
                    } finally {
                        // 5.释放（实际上connection已经交由Spring管理了）
                        transactionManager.release();
                    }
                    return null;
                }));
    }
}
```



#### UserServiceClient.java

- 关于异常的抓取，我们已经在动态代理中使用try{...}catch{...}finally{...}将异常抓取了，因此在实际的表现层中，不需要再对可能出现异常的代码，进行异常抓取的操作；
- 但是Spring AOP则并不是主动将异常进行抓取，而且将异常抛出。

```java
package cn.dylanphang.aopuserdefinedconnection.client;

import cn.dylanphang.aopuserdefinedconnection.config.SpringConfiguration;
import cn.dylanphang.aopuserdefinedconnection.pojo.User;
import cn.dylanphang.aopuserdefinedconnection.proxy.UserServiceFactory;
import cn.dylanphang.aopuserdefinedconnection.service.UserService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
public class UserServiceClient {
    public static void main(String[] args) {

        // 1.加载Spring配置，加载Bean对象
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SpringConfiguration.class);

        // 2.使用Bean的id获取对象
        UserServiceFactory userServiceFactory = ac.getBean("userServiceFactory", UserServiceFactory.class);

        // 3.获取UserService的代理类
        UserService userService = userServiceFactory.getUserService();

        // *.第一次CURD操作
        User user = userService.findUserById(18);
        user.setMoney((float) 9000);
        userService.updateUser(user);

        // *.第二次CURD操作
        userService.transfer("mike", "tom", (float) 500); // transfer中手动引发了异常

        // *.第三次CURD操作
        List<User> users = userService.findAllUser();

        for (User insideUser : users) {
            System.out.println(insideUser);
        }

        // 4.释放资源
        ac.close();
    }
}
```

## TransactionTemplate的事务管理

### 动态代理前的准备事项

- 前面已经说过了，如果我们使用JdbcTemplate帮助我们进行CURD的操作，那么将不再可以使用自行编写TransactionManager类，因为我们无法保证Connection的唯一性，即参与事务管理的Connection对象并不一定是进行CURD操作的Connection对象。
- 如果此时需要进行事务管理，我们只能使用Spring为我们提供的事务管理类TransactionTemplate；

- 在Spring Ioc中配置TransacitonTemplate的Bean，即可在动态代理中使用TransactionTemplate对象了。

#### Ioc.xml

- 其中定义了如何获取DataSourceTransactionManager对象，以及如何通过此DataSourceTransactionManager对象进一步获取TransactionTemplate对象。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 告诉Spring创建容器时要扫描的包 -->
    <context:component-scan base-package="cn.dylanphang.proxy"></context:component-scan>

    <!-- 配置如何创建dataSource对象 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSourceFactory" factory-method="createDataSource">
        <constructor-arg>
            <props>
                <prop key="driverClassName">com.mysql.cj.jdbc.Driver</prop>
                <prop key="url">jdbc:mysql://localhost:3306/spring?serverTimezone=UTC</prop>
                <prop key="username">root</prop>
                <prop key="password">root</prop>
            </props>
        </constructor-arg>
    </bean>

    <!-- 配置如何创建JdbcTemplate对象 -->
    <bean id="JdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <constructor-arg name="dataSource" ref="dataSource"></constructor-arg>
    </bean>

    <!-- 配置如何创建TransactionManager对象，供动态代理使用 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>

    <!-- 配置如何创建TransactionTemplate对象，供动态代理使用 -->
    <bean id="transactionTemplate" class="org.springframework.transaction.support.TransactionTemplate">
        <property name="transactionManager" ref="transactionManager"></property>
    </bean>
</beans>
```

### JDK官方的Proxy类创建动态代理

#### UserServiceFactory.java

- 使用JDK官方的Proxy类创建动态代理，使用TransactionTemplate进行事务的管理。
- 执行TransactionTemplate执行事务管理，需要调用excute()方法，并传入一个TransactionCallback对象，可以使用匿名内部类，直接覆写TransactionCallback对象中的doInTransaction()方法，业务层代码编写于doInTransaction中即可。
- 可以通过TransactionTemplate对象对事务进行进一步的管理与设置。

```java
package cn.dylanphang.proxy.transaction;

import cn.dylanphang.proxy.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.TransactionCallback;
import org.springframework.transaction.support.TransactionTemplate;

import java.lang.reflect.Proxy;

@Component("userServiceFactory")
public class UserServiceFactory {
    private TransactionTemplate transactionTemplate;
    private UserService userService;

    @Autowired
    public void setTransactionTemplate(TransactionTemplate transactionTemplate) {
        this.transactionTemplate = transactionTemplate;
    }

    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }

    public UserService getUserService() {
        return (UserService) Proxy.newProxyInstance(userService.getClass().getClassLoader(),
                userService.getClass().getInterfaces(),
                (proxy, method, args) -> { // invoke(...)
                    // 设置事务传播属性
                    transactionTemplate.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
                    // 设置事务的隔离级别,设置为读已提交（默认是ISOLATION_DEFAULT:使用的是底层数据库的默认的隔离级别）
                    transactionTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
                    // 设置是否只读，默认是false
                    transactionTemplate.setReadOnly(true);
                    // 默认使用的是数据库底层的默认的事务的超时时间
                    transactionTemplate.setTimeout(30000);

                    return transactionTemplate.execute((transactionStatus) -> { // doInTransaction(...)
                        try {
                            return method.invoke(userService, args);
                        } catch (Exception e) {
                            transactionStatus.setRollbackOnly();
                        }
                        return null;
                    });
                });
    }
}
```

### CGLib的EnHancer类创建动态代理

#### UserServiceFactory.java

- 使用CGLib的EnHancer类创建动态代理，使用TransactionTemplate进行事务的管理。

```java
package cn.dylanphang.enhancer.transaction;

import cn.dylanphang.enhancer.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.stereotype.Component;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.support.TransactionTemplate;

@Component("userServiceFactory")
public class UserServiceFactory {
    private TransactionTemplate transactionTemplate;
    private UserService userService;

    @Autowired
    public void setTransactionTemplate(TransactionTemplate transactionTemplate) {
        this.transactionTemplate = transactionTemplate;
    }

    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }

    public UserService getUserService() {
        return (UserService) Enhancer.create(userService.getClass(),
                (MethodInterceptor) (proxy, method, args, methodProxy) -> { // intercept(...)
                    // 设置事务传播属性
                    transactionTemplate.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
                    // 设置事务的隔离级别,设置为读已提交（默认是ISOLATION_DEFAULT:使用的是底层数据库的默认的隔离级别）
                    transactionTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
                    // 设置是否只读，默认是false
                    transactionTemplate.setReadOnly(true);
                    // 默认使用的是数据库底层的默认的事务的超时时间
                    transactionTemplate.setTimeout(30000);

                    return transactionTemplate.execute((transactionStatus) -> { // doInTransaction(...)
                        try {
                            return method.invoke(userService, args);
                        } catch (Exception e) {
                            transactionStatus.setRollbackOnly();
                        }
                        return null;
                    });
                }
        );
    }
}
```

# AOP实现事务管理

## 关于AOP的使用细节

- 同样的，使用AOP进行事务管理的前提，是保证参与事务管理的Connection对象和进行CURD操作的Connection对象时同一个；
- 因此我们不能使用Spring JdbcTemplate，只能使用自定义的TransactionManager进行AOP的使用示例。

- 如果不使用环绕通知配置AOP，则AOP不会对出现异常的CURD语句进行异常的捕捉，它仅仅提供事务管理的功能，可以通过以下方法进行异常的捕捉：
  1. 在业务层使用try{...}catch{...}结构对实际调用的代码行进行异常的捕捉，但实际上在表现层会总会多次地调用CURD代码对数据库进行操作，不太推荐；
  2. 使用动态代理，对业务层的代码进行代理，在代理中完成异常的捕捉。

- 如果不适用环绕通知配置AOP，其他通知的执行顺序是不一致的，例：在最终通知中配置了setAutoCommit(true)，异常通知中配置了事务回滚操作rollback()，最终通知有可能会先于异常通知而将事务提交设置为TRUE，会导致异常通知中的事务回滚操作失败，从而产生异常。

- 推荐使用环绕通知的方式配置AOP，在通知类中编写环绕通知的方法，在方法体中完成异常的抓取与事务的管理。

## 关于AOP的相关术语

1. JoinPoint（连接点）
   - 所谓连接点是指那些被拦截到的点。在Spring中，这些点指的是方法，因为Spring只支持方法类型的连接点。
2. Pointcut（切入点）
   - 所谓切入点是指我们要对哪些Joinpoint进行拦截的定义。
3. Advice（通知/增强）
   - 所谓通知是指连接到Joinpoint之后所要做的事情就是通知。
   - 通知的类型：前置通知、后置通知、异常通知、最终通知、环绕通知。
4. Introduction（引介）
   - 引介是一种特殊的通知在不修改类代码的前提下，Introduction可以在运行期为类动态地添加一些方法或Field。
5. Target（目标对象）
   - 代理的目标对象。
6. Weaving（织入）
   - 是指把增强应用到目标对象来创建新的代理对象的过程。
   - Spring采用动态代理织入，而AspectJ采用编译期织入或类装载期织入。
7. Proxy（代理）
   - 一个类被AOP织入增强后，就产生一个结果代理类。
8. Aspect（切面）
   - 是切入点和通知（引介）的结合。

## AOP基于Xml文件的配置

### AOP配置标签

1. \<aop:config>
   - 作用：用于声明开始aop的配置，所有的aop配置代码都写在里面。

2. \<aop:aspect>
   - 作用：用于配置切面。
   - 属性：
     - id：给切面提供一个唯一标识；
     - ref：引用配置好的通知类bean的id。（通知类包括用于作事务管理的类）
3. \<aop:pointcut>
   - 作用：用于配置切入点表达式。就是指定对哪些类的哪些方法进行增强。
   - 属性：
     - expression：用于定义切入点表达式；
     - id：用于给切入点表达式提供一个唯一标识。
4. \<aop:before>
   - 作用：用于配置前置通知。指定增强的方法在切入点方法之前执行。
   - 属性：
     - method：用于指定通知类中的增强方法名称；
     - pointcut-ref：用于指定切入点表达式引用；
     - pointcut：用于指定切入点表达式。
5. \<aop:after-returning>
   - 作用：用于配置后置通知。
   - 属性：
     - method：用于指定通知类中的增强方法名称；
     - pointcut-ref：用于指定切入点表达式引用；
     - pointcut：用于指定切入点表达式。

6. \<aop:after-throwing>
   - 作用：用于配置异常通知。
   - 属性：
     - method：用于指定通知类中的增强方法名称；
     - pointcut-ref：用于指定切入点表达式引用；
     - pointcut：用于指定切入点表达式。
7. \<aop:after>
   - 作用：用于配置最终通知。
   - 属性：
     - method：用于指定通知类中的增强方法名称；
     - pointcut-ref：用于指定切入点表达式引用；
     - pointcut：用于指定切入点表达式。



- 关于切入点表达式说明
  - execution：匹配方法的执行（常用）
  - 表达式语法：execution([修饰符] 返回值类型 包名.类名.方法名.(参数))
  - 通常情况下，我们都是对业务层的方法进行增强，所以切入点表达式都是切到业务层实现类。
    - execution="\* cn.dylanphang.aop.service.impl.\*.\*(..)"
  - 写法说明及示例：

|                             规则                             |                             示例                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                          全匹配方式                          | "public void cn.dylanphang.aop.service.impl.UserServiceImpl.saveUser(cn.dylanphang.aop.pojo.User)" |
|                           省略模式                           | "void cn.dylanphang.aop.service.impl.UserServiceImpl.saveUser(cn.dylanphang.aop.pojo.User)" |
|              返回值可以使用\*号，表示任意返回值              | "\* cn.dylanphang.aop.service.impl.UserServiceImpl.saveUser(cn.dylanphang.aop.pojo.User)" |
| 包名可以使用\*号，表示任意包名，但是有几级包，就需要写几个\* | "\* \*.\*.\*.\*.\*.UserServiceImpl.saveUser(cn.dylanphang.aop.pojo.User)" |
|                 使用..来表示当前包，及其子包                 | "\* cn..UserServiceImpl.saveUser(cn.dylanphang.aop.pojo.User)" |
|                 类名可以使用\*号，表示任意类                 |      "\* cn..\*.saveUser(cn.dylanphang.aop.pojo.User)"       |
|               方法名可以使用\*号，表示任意方法               |         "\* cn..\*.\*(cn.dylanphang.aop.pojo.User)"          |
| 参数列表可以使用\*号，表示参数可以是任意数据类型，但是必须有参数 |                      "\* cn..\*.\*(\*)"                      |
|   参数列表可以使用..表示有无参数均可，有参数可以是任意类型   |                      "\* cn..\*.\*(..)"                      |
|                          全通配方式                          |                      "\* \*..\*.\*(..)"                      |

8. \<aop:around>

   - 作用：用于配置环绕通知。
   - 属性：
     - method：用于指定通知类中的增强方法名称；
     - pointcut-ref：用于指定切入点表达式引用；
     - pointcut：用于指定切入点表达式。

   - 说明：它是Spring框架为我们提供的一种可以在代码中手动控制增强代码什么时候执行的方式。
   - 注意：通常情况下，环绕通知都是独立使用的。AOP配置文件

#### aop.xml

- AOP的.xml配置，Intellij IDEA即使在配置正确的情况下，可能会出现错误报警，但配置是可用的；
- 以下配置没有使用环绕通知，通知的顺序是不一定的。如果我们在最终通知里进行了资源的释放，很可能导致后置通知或异常通知报错；

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 告诉Spring创建容器时要扫描的包 -->
    <context:component-scan base-package="cn.dylanphang.aop"/>
    
    <!-- 使用aop:config声明aop配置，配置代码都写在里面 -->
    <aop:config>
        <!-- 配置切入点表达式 -->
        <aop:pointcut id="targetPointcut" expression="execution(* cn.dylanphang.aop.service.impl.*.*(..))"/>
        <!-- 配置切面通知类 -->
        <aop:aspect id="transactionAspect" ref="transactionManager">
            <!-- 前置通知 -->
            <aop:before method="beginTransaction" pointcut-ref="targetPointcut"/>
            <!-- 后置通知 -->
            <aop:after-returning method="commit" pointcut-ref="targetPointcut"/>
            <!-- 异常通知 -->
            <aop:after-throwing method="rollback" pointcut-ref="targetPointcut"/>
            <!-- 最终通知 -->
            <aop:after method="release" pointcut-ref="targetPointcut"/>
        </aop:aspect>
    </aop:config>
</beans>
```



- 如果需要保证通知的顺序，务必要使用环绕通知。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 告诉Spring创建容器时要扫描的包 -->
    <context:component-scan base-package="cn.dylanphang.aop"/>
    
    <!-- 使用aop:config声明aop配置，配置代码都写在里面 -->
    <aop:config>
        <!-- 配置切入点表达式 -->
        <aop:pointcut id="targetPointcut" expression="execution(* cn.dylanphang.aop.service.impl.*.*(..))"/>
        <!-- 配置切面通知类 -->
        <aop:aspect id="transactionAspect" ref="transactionManager">
            <!-- 环绕通知 -->
            <aop:around method="transactionAround" pointcut-ref="targetPointcut"></aop:around>
        </aop:aspect>
    </aop:config>
</beans>
```



### AOP配置的类

#### ConnectionConfig.java

- 用于配置唯一的Connection对象进行事务管理。

```java
package cn.dylanphang.aop.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

@Component
@PropertySource("classpath:druid.properties")
public class ConnectionConfig {

    @Value("${conn.driverClassName}")
    private String driver;
    @Value("${conn.url}")
    private String url;
    @Value("${conn.username}")
    private String username;
    @Value("${conn.password}")
    private String password;

    @Bean("connection")
    public Connection createConnection() {
        Connection conn = null;
        try {
            Class.forName(driver);
            conn = DriverManager.getConnection(url, username, password);
            return conn;
        } catch (ClassNotFoundException | SQLException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

#### TransactionManager.java

- 配置在\<aop:aspect>中的通知类（事务管理类）TransactionManager，当\<aop:pointcut>中配置的业务层UserServiceImpl中的方法被调用时，执行\<aop:aspect>中的配置。

```java
package cn.dylanphang.aopannotation.transaction;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.sql.Connection;
import java.sql.SQLException;

/**
 * Spring框架为我们提供了一个接口：ProceedingJoinPoint，它可以作为环绕通知的方法参数。在环绕通知执行时，Spring框架会为我们提供该接口的实现类对象，我们直接使用即可。
 */
@Component("transactionManager")
@Aspect
public class TransactionManager {

    private Connection connection;

    @Autowired
    public void setConnection(Connection connection) {
        this.connection = connection;
    }

    public Object transactionAround(ProceedingJoinPoint proceedingJoinPoint) {
        try {
            // 1.开启事务
            this.beginTransaction();
            
			// 2.获取方法执行参数，并调用目标方法
            Object[] args = proceedingJoinPoint.getArgs();
            Object rtValue = proceedingJoinPoint.proceed(args);
			
            // 3.提交事务
            this.commit();
            return rtValue;
        } catch (Throwable throwable) {
            // 4.回滚事务
            this.rollback();
            throwable.printStackTrace();
        } finally {
            // 5.释放
            this.release();
        }
        return null;
    }

    public void beginTransaction() {
        try {
            connection.setAutoCommit(false);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public void commit() {
        try {
            connection.commit();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public void rollback() {
        try {
            connection.rollback();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public void release() {
        try {
            connection.setAutoCommit(true);
        } catch (SQLException e) {
            e.printStackTrace();
        }

    }
}
```

#### UserServiceImpl.java

- 配置在\<aop:pointcut>中的业务层UserServiceImpl类。

```java
package cn.dylanphang.aop.service.impl;

import cn.dylanphang.aop.dao.UserDao;
import cn.dylanphang.aop.pojo.User;
import cn.dylanphang.aop.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.List;

@Component("userService")
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;

    @Override
    public void transfer(String sourceName, String targetName, Float money) {
        User sourceUser = userDao.findByName(sourceName);
        User targetUser = userDao.findByName(targetName);

        sourceUser.setMoney(sourceUser.getMoney() - money);
        targetUser.setMoney(targetUser.getMoney() + money);

        userDao.update(sourceUser);
        int i = 1 / 0; // 模拟转账异常，异常前的事务都会成功提交，异常后的代码将不会执行。
        userDao.update(targetUser);
    }

    @Override
    public void saveUser(User user) {
        this.userDao.save(user);
    }

    @Override
    public void deleteUser(Integer id) {
        this.userDao.delete(id);
    }

    @Override
    public void updateUser(User user) {
        this.userDao.update(user);
    }

    @Override
    public User findUserById(Integer id) {
        return this.userDao.findById(id);
    }

    @Override
    public User findUserByName(String username) {
        return this.userDao.findByName(username);
    }

    @Override
    public List<User> findAllUser() {
        return this.userDao.findAll();
    }
}
```

#### UserServiceExceptionCatch.java

- AOP不会对业务层出现的异常进行捕捉，因此需要使用动态代理对业务层代码的异常进行捕捉处理；
- 如果不对UserServiceImpl进行异常捕捉，表现层中的其他CURD代码将会受到异常的影响，除非给每一个CURD代码都加上try{...}catch{...}结构，否则仍应该使用动态代理直接对业务层代码进行捕捉处理。

```java
package cn.dylanphang.aop.proxy;

import cn.dylanphang.aop.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

import java.lang.reflect.Proxy;

@Component("userServiceExceptionCatch")
public class UserServiceExceptionCatch {

    @Autowired
    private UserService userService;

    @Bean("userServiceProxy")
    public UserService getUserService() {
        return (UserService) Proxy.newProxyInstance(userService.getClass().getClassLoader(),
                userService.getClass().getInterfaces(),
                ((proxy, method, args) -> {
                    try {
                        return method.invoke(userService, args);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                    return null;
                }));
    }
}
```

#### UserServiceClient.java

- 直接在动态代理的方法上使用@Bean注解，Spring会自动加载并获取到代理对象，在表现层中直接使用id就能获取到目标代理对象；
- 关于getBean()方法，第二个参数传入什么类的字节码文件，就能获取到什么类型的返回值。可以采用多态，但类型一定要符合规则。

```java
package cn.dylanphang.aop.client;

import cn.dylanphang.aop.pojo.User;
import cn.dylanphang.aop.service.UserService;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
public class UserServiceClient {
    public static void main(String[] args) {

        ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("BeanForAop.xml");
        UserService userService = ac.getBean("userServiceProxy", UserService.class);

        // aop对异常进行了抛出操作，如果要后面的代码可用，需要进行异常的抓取。
        userService.transfer("mike", "tom", (float) 500);

        List<User> users = userService.findAllUser();
        for (User user : users) {
            System.out.println(user);
        }

        ac.close();
    }
}
```

## AOP基于注解的配置

### AOP常用注解

1. @Aspect
   - 作用：把当前类声明为切面类，相当于\<aop:aspect>。

2. @Before
   - 作用：把当前方法看成是前置通知，相当于\<aop:before>。
   - 属性：
     - value：用于指定切入点表达式，还可以指定切入点表达式的引用。
3. @AfterReturning
   - 作用：把当前方法看成是后置通知，相当于\<aop:after-returning>。
   - 属性：
     - value：用于指定切入点表达式，还可以指定切入点表达式的引用。
4. @AfterThrowing
   - 作用：把当前方法看成是异常通知，相当于\<aop:after-throwing>。
   - 属性：
     - value：用于指定切入点表达式，还可以指定切入点表达式的引用。
5. @After
   - 作用：把当前方法看成是最终通知，相当于\<aop:after>。
   - 属性：
     - value：用于指定切入点表达式，还可以指定切入点表达式的引用。
6. @Around
   - 作用：把当前方法看成是环绕通知，相当于\<aop:around>。
   - 属性：
     - value：用于指定切入点表达式，还可以指定切入点表达式的引用。
7. @Pointcut
   - 作用：指定切入点表达式
   - 属性：
     - value：指定表达式的内容。
8. @EnableAspectJAutoProxy
   - 作用：在不适用XML配置的时候，在SrpingConfiguration类上使用的注解，允许aop编程。

### AOP配置的类

#### TransactionManager.java

- 以下为通知类的完整范例，包括不使用环绕通知、使用环绕通知且使用@Pointcut与仅使用环绕通知。

```java
package cn.dylanphang.aopannotation.transaction;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.sql.Connection;
import java.sql.SQLException;

@Component("transactionManager")
@Aspect
public class TransactionManager {

    private Connection connection;

    @Autowired
    public void setConnection(Connection connection) {
        this.connection = connection;
    }

    @Pointcut("execution(* cn.dylanphang.aopannotation.service.impl.*.*(..))")
    public void pc() {}

    @Around("pc()")
    // @Around("execution(* cn.dylanphang.aopannotation.service.impl.*.*(..))")
    public Object transactionAround(ProceedingJoinPoint proceedingJoinPoint) {
        try {
            Object[] args = proceedingJoinPoint.getArgs();

            beginTransaction();

            Object rtValue = proceedingJoinPoint.proceed(args);

            commit();
            return rtValue;

        } catch (Throwable throwable) {
            rollback();
            throwable.printStackTrace();
        } finally {
            release();
        }
        return null;
    }
    
	// @Before("pc()")
    // @Before("execution(* cn.dylanphang.aopannotation.service.impl.*.*(..))")
    public void beginTransaction() {
        try {
            connection.setAutoCommit(false);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    
	// @AfterReturning("pc()")
    // @AfterReturning("execution(* cn.dylanphang.aopannotation.service.impl.*.*(..))")
    public void commit() {
        try {
            connection.commit();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
	
    // @AfterThrowing("pc()")
    // @AfterThrowing("execution(* cn.dylanphang.aopannotation.service.impl.*.*(..))")
    public void rollback() {
        try {
            connection.rollback();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
	// @After("pc()")
    // @After("execution(* cn.dylanphang.aopannotation.service.impl.*.*(..))")
    public void release() {
        try {
            connection.setAutoCommit(true);
        } catch (SQLException e) {
            e.printStackTrace();
        }

    }
}
```

#### SpringConfiguration.java

- 类中使用了@EnableAspectJAutoProxy注解，允许aop运行。

```java
package cn.dylanphang.aopannotation.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
import org.springframework.context.annotation.Import;

@Configuration
@ComponentScan({"cn.dylanphang.aopannotation"})
@Import({ConnectionConfig.class})
@EnableAspectJAutoProxy
public class SpringConfiguration {
}
```

#### ConnectionConfig.java

```java
package cn.dylanphang.aopannotation.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

@Configuration
@PropertySource("classpath:druid.properties")
public class ConnectionConfig {

    @Value("${conn.driverClassName}")
    private String driver;
    @Value("${conn.url}")
    private String url;
    @Value("${conn.username}")
    private String username;
    @Value("${conn.password}")
    private String password;

    @Bean("connection")
    public Connection createConnection() {
        Connection conn = null;
        try {
            Class.forName(driver);
            conn = DriverManager.getConnection(url, username, password);
            return conn;
        } catch (ClassNotFoundException | SQLException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

# Spring中的JdbcTemplate

## JdbcTemplate概述

- JdbcTemplate是Spring框架中的一个对象，是对样式Jdbc API对象的简单封装。除此之外，Spring框架还提供了很多其他的操作模板类。
  - 操作关系型数据库：
    - JdbcTemplate
    - HibernateTemplate
  - 操作nosql数据库：
    - RedisTemplate
  - 操作消息队列：
    - JmsTemplate

## DataSource数据库连接池的配置

- 数据库连接池的种类很多，有c3p0、dbcp、druid等等。Spring框架中也内置了数据库连接池。
- 要在Spring中使用数据库连接池，首先需要数据库配置文件，包括driver、url、username、password等连接信息，其次需要xml配置文件。



### 数据库连接参数配置

#### jdbc.properties

- 4个最基本的连接信息需要提供在配置文件中。

```properties
# 数据库连接参数配置
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring?serverTimezone=UTC
jdbc.username=root
jdbc.password=root
```



### 数据库连接池的配置

- 数据库配置文件关联到spring配置文件中，以我所知有三种方式：
  1. 通过\<context:property-placeholder>将配置文件配置到其中的location属性中；
  2. 通过\<bean>加载类org.springframework.context.support.PropertySourcesPlaceholderConfigurer，在其中的\<property>中配置属性name="location"、value="classpath:jdbc.properties"；
  3. 通过\<bean>加载类org.springframework.beans.factory.config.PropertyPlaceholderConfigurer，在其中的\<property>中配置属性name="location"、value="classpath:jdbc.properties"。
- 注意事项：
  - 以上配置中，第三条的类PropertyPlaceholderConfigurer已过时，不推荐使用；
  - "classpath:"关键字可省略不写，IDEA会自动提醒。



#### c3p0.xml

- 以下是使用c3p0配置数据库连接池的配置文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="cn.dylanphang.jdbctemplate"/>
    <!--<context:property-placeholder location="classpath:jdbc.properties"/>-->

    <!--<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">-->
        <!--<property name="location" value="jdbc.properties"/>-->
    <!--</bean>-->

    <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="location" value="classpath:jdbc.properties"/>
    </bean>

    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>
</beans>
```



#### dbcp.xml

- 以下是使用dbcp配置数据库连接池的配置文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="cn.dylanphang.jdbctemplate"/>
    <!--<context:property-placeholder location="classpath:jdbc.properties"/>-->

    <bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
        <property name="location" value="jdbc.properties"/>
    </bean>

    <!--<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">-->
        <!--<property name="location" value="classpath:jdbc.properties"/>-->
    <!--</bean>-->

    <bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>
</beans>
```



#### druid.xml

- 以下是使用druid配置数据库连接池的配置文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="cn.dylanphang.jdbctemplate"/>
    <!--<context:property-placeholder location="classpath:jdbc.properties"/>-->

    <!--<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">-->
    <!--<property name="location" value="jdbc.properties"/>-->
    <!--</bean>-->

    <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="location" value="classpath:jdbc.properties"/>
    </bean>

    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSourceFactory" factory-method="createDataSource">
        <constructor-arg>
            <props>
                <prop key="driverClassName">${jdbc.driver}</prop>
                <prop key="url">${jdbc.url}</prop>
                <prop key="username">${jdbc.username}</prop>
                <prop key="password">${jdbc.password}</prop>
            </props>
        </constructor-arg>
    </bean>

    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>
</beans>
```



#### jdbctemplate-built-in.xml

- 以下是使用Spring框架内置对象配置数据库连接池的配置文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="cn.dylanphang.jdbctemplate"/>
    <!--<context:property-placeholder location="classpath:jdbc.properties"/>-->

    <bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
        <property name="location" value="jdbc.properties"/>
    </bean>

    <!--<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">-->
    <!--<property name="location" value="classpath:jdbc.properties"/>-->
    <!--</bean>-->

    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>
</beans>
```



## JdbcTemplate的创建及配置

- 通过阅读JdbcTemplate的源码，可以发现配置JdbcTemplate对象的Bean可以，数据源DataSource既可以使用构造函数注入，也可以使用Setter方法注入；
- 构造器注入其实就是使用\<constructor-arg>标签进行配置，Setter方法注入就是使用\<property>标签进行配置。



### JdbcTemplate源码及xml配置文件

#### JdbcTemplate.java

```java
public JdbcTemplate() {
}

public JdbcTemplate(DataSource dataSource) { // 可以使用构造函数注入
    this.setDataSource(dataSource); // 可以使用Setter依赖注入
    this.afterPropertiesSet();
}

public JdbcTemplate(DataSource dataSource, boolean lazyInit) {
    this.setDataSource(dataSource);
    this.setLazyInit(lazyInit);
    this.afterPropertiesSet();
}
```



#### jdbcTemplate.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 告诉Spring创建容器时要扫描的包 -->
    <context:component-scan base-package="cn.dylanphang.proxy"></context:component-scan>

    <!-- 配置如何创建dataSource对象 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSourceFactory" factory-method="createDataSource">
        <constructor-arg>
            <props>
                <prop key="driverClassName">com.mysql.cj.jdbc.Driver</prop>
                <prop key="url">jdbc:mysql://localhost:3306/spring?serverTimezone=UTC</prop>
                <prop key="username">root</prop>
                <prop key="password">root</prop>
            </props>
        </constructor-arg>
    </bean>

    <!-- 配置如何创建jdbcTemplate对象 -->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <constructor-arg name="dataSource" ref="dataSource"></constructor-arg>
    </bean>
</beans>
```



### JdbcTemplate增删改查操作

- JdbcTemplate对于数据库的增删改查操作，主要用的方法有update()、query()和queryForObject()。
  - 增删改：update(String sql, [...])；
  - 查：query()可以执行多行查询；queryForObject()执行单行查询。
    - 两个方法的参数都依赖于RowMapper()的类，此类为抽象类，可以使用匿名内部类对其中的方法进行覆写。
    - 可以使用Spring为我们提供的BeanPropertyRowMapper()类，传入一个字节码.class文件即可，返回值类型和所提供的字节码类一致。
    - 对于不同的Java版本来说，写入字节码文件的位置不太相同，可以统一写成new BeanPropertyRowMapper\<User>(User.class)形式。



#### UserDaoImpl.java

- dao的实现类，其中使用JdbcTemplate对数据库进行了增删改查（CURD）等操作。

```java
package cn.dylanphang.jdbctemplate.dao.impl;

import cn.dylanphang.jdbctemplate.dao.UserDao;
import cn.dylanphang.jdbctemplate.pojo.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

import java.util.Date;
import java.util.List;

@Repository("userDao")
public class UserDaoImpl implements UserDao {

    private JdbcTemplate jdbcTemplate;

    // 注解可以放置在参数列表中
    public UserDaoImpl(@Autowired JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Override
    public void save(User user) {
        String username = user.getUsername();
        Date birthday = user.getBirthday();
        String gender = user.getGender();
        Float money = user.getMoney();

        jdbcTemplate.update("INSERT INTO user (username, birthday, gender, money) VALUES (?, ?, ?, ?)", username, birthday, gender, money);
    }

    @Override
    public void delete(Integer id) {
        jdbcTemplate.update("DELETE FROM user WHERE id=?", id);
    }

    @Override
    public void update(User user) {
        Integer id = user.getId();
        String username = user.getUsername();
        Date birthday = user.getBirthday();
        String gender = user.getGender();
        Float money = user.getMoney();

        jdbcTemplate.update("UPDATE user SET username=?, birthday=?, gender=?, money=? WHERE id=?", username, birthday, gender, money, id);

    }

    @Override
    public User findById(Integer id) {
        User user = jdbcTemplate.queryForObject("SELECT * FROM user WHERE id=?", new BeanPropertyRowMapper<>(User.class), id);
        return user;
    }

    @Override
    public User findByName(String username) {
        User user = jdbcTemplate.queryForObject("SELECT * FROM user WHERE username=?", new BeanPropertyRowMapper<>(User.class), username);
        return user;
    }

    @Override
    public List<User> findAll() {
        List<User> users = jdbcTemplate.query("SELECT * FROM user", new BeanPropertyRowMapper<>(User.class));
        return users;
    }
}
```


#### findByName.(Method)

- 关于queryForObject(...)中如何使用匿名内部类方式覆写RowMapper\<T>(Class\<T> class)对象。

```java
public User findByName(String username) {
    User user = jdbcTemplate.queryForObject("SELECT * FROM user WHERE username=?", new RowMapper<>(User.class) {
        @Override
        public Object mapRow(ResultSet resultSet, int i) throws SQLException {
            // 使用ResultSet获取值，存在User中，并返回User
            return null;
        }
    }, username);
    return user;
}
```


#### findAll.(Method)

- 关于query(...)中如何使用匿名内部类方式覆写RowMapper\<T>(Class\<T> class)对象。

```java
public List<User> findAll() {
    List<User> users = jdbcTemplate.query("SELECT * FROM user", new RowMapper<>(User.class) {
        @Override
        public Object mapRow(ResultSet resultSet, int i) throws SQLException {
            // 使用ResultSet获取值，存在User中，并创建List<User>保存结果集，最后返回结果集
            return null;
        }
    });

    return users;
```



### JdbcTemplate在dao中的使用方式

1. 如何在dao中使用JdbcTemplate，最简单的方式莫过于依赖注入：
  - 通过配置Spring IoC的xml文件，创建JdbcTemplate的bean，在配置文件中可以使用dao实现类中构造函数或Setter的方式将JdbcTemplate的bean注入到dao实现类中；
  - 通过配置jdbcConfig的配置类，在其中配置获取JdbcTemplate的方法，使用@Bean注解，即可在dao实现类中使用@Autowired将bean注入。
2. 其二方式，可以通过让dao的实现类继承JdbcDaoSupport类，达到获取JdbcTemplate对象的目的。

#### JdbcDaoSupport.java

- 需要明确的一点，JdbcDaoSupport的作用是提供数据库连接池DataSource给它，它帮助我们创建JdbcTemplate对象，它的子类可以通过父类方法getJdbcTemplate()的方式获取到JdbcTemplate对象，从而使用此对象进行增删改查。

- 作用原理，通过查看JdbcDaoSupport源码，dao实现类从JdbcDaoSupport中能继承下来setDataSource()方法，可以配置基于xml文件将此DataSource依赖注入，JdbcDaoSupport就可以使用此DataSource创建JdbcTemplate对象。
- 在xml中配置此dao实现类的bean，将DataSource注入到dao实现类中，即可在实现类里使用getJdbcTemplate()方法获取到jdbcTemplate对象。
- 但通过继承JdbcDaoSupport的方式，无法对dao进行注解的配置，即便我们拥有DataSource的配置类@Bean，也只能通过xml文件对dao实现类进行依赖注入的操作。（子类中使用父类的方法，会使用父类的属性，private访问修饰符修饰的字段是不会被继承下来的。）

```java
package org.springframework.jdbc.core.support;

import java.sql.Connection;
import javax.sql.DataSource;

import org.springframework.dao.support.DaoSupport;
import org.springframework.jdbc.CannotGetJdbcConnectionException;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DataSourceUtils;
import org.springframework.jdbc.support.SQLExceptionTranslator;
import org.springframework.lang.Nullable;
import org.springframework.util.Assert;

public abstract class JdbcDaoSupport extends DaoSupport {

   @Nullable
   private JdbcTemplate jdbcTemplate;

   /**
    * Set the JDBC DataSource to be used by this DAO.
    */
   public final void setDataSource(DataSource dataSource) {
      if (this.jdbcTemplate == null || dataSource != this.jdbcTemplate.getDataSource()) {
         this.jdbcTemplate = createJdbcTemplate(dataSource);
         initTemplateConfig();
      }
   }

   protected JdbcTemplate createJdbcTemplate(DataSource dataSource) {
      return new JdbcTemplate(dataSource);
   }

   @Nullable
   public final DataSource getDataSource() {
      return (this.jdbcTemplate != null ? this.jdbcTemplate.getDataSource() : null);
   }

   public final void setJdbcTemplate(@Nullable JdbcTemplate jdbcTemplate) {
      this.jdbcTemplate = jdbcTemplate;
      initTemplateConfig();
   }

   @Nullable
   public final JdbcTemplate getJdbcTemplate() {
     return this.jdbcTemplate;
   }

   protected void initTemplateConfig() {
   }

   @Override
   protected void checkDaoConfig() {
      if (this.jdbcTemplate == null) {
         throw new IllegalArgumentException("'dataSource' or 'jdbcTemplate' is required");
      }
   }

   protected final SQLExceptionTranslator getExceptionTranslator() {
      JdbcTemplate jdbcTemplate = getJdbcTemplate();
      Assert.state(jdbcTemplate != null, "No JdbcTemplate set");
      return jdbcTemplate.getExceptionTranslator();
   }

   protected final Connection getConnection() throws CannotGetJdbcConnectionException {
      DataSource dataSource = getDataSource();
      Assert.state(dataSource != null, "No DataSource set");
      return DataSourceUtils.getConnection(dataSource);
   }

   protected final void releaseConnection(Connection con) {
      DataSourceUtils.releaseConnection(con, getDataSource());
   }

}
```

#### druid.xml

- 使用继承JdbcDaoSupport的方式，需要在dao实现类中注入DataSource，JdbcDaoSupport会使用此DataSource创建JdbcTemplate对象，在dao实现类中使用getJdbcTemplate()获取jdbcTemplate对象；
- 可以直接选择注入JdbcTemplate，但是没有必要，因为如果我拥有JdbcTemplate的Bean，为什么不直接注入到目标dao实现类中。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="cn.dylanphang.jdbctemplate"/>

    <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="location" value="classpath:jdbc.properties"/>
    </bean>

    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSourceFactory" factory-method="createDataSource">
        <constructor-arg>
            <props>
                <prop key="driverClassName">${jdbc.driver}</prop>
                <prop key="url">${jdbc.url}</prop>
                <prop key="username">${jdbc.username}</prop>
                <prop key="password">${jdbc.password}</prop>
            </props>
        </constructor-arg>
    </bean>
    
	<!-- 使用继承JdbcDaoSupport的方式，需要在dao实现类中注入DataSource，JdbcDaoSupport会使用此DataSource创建JdbcTemplate对象 -->
    <!-- 也可以直接注入JdbcTemplate，但没必要 -->
    <bean id="userDao" class="cn.dylanphang.jdbctemplate.dao.impl.UserDaoImpl">
        <property name="dataSource" ref="dataSource"/>
    </bean>
</beans>
```

#### UserDaoImpl.java

- 此版dao实现类通过继承JdbcDaoSupport的方式，使用父类方法getJdbcTemplate()获取到jdbcTemplate对象，进行增删改查的操作。

```java
package cn.dylanphang.jdbctemplate.dao.impl;

import cn.dylanphang.jdbctemplate.dao.UserDao;
import cn.dylanphang.jdbctemplate.pojo.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.support.JdbcDaoSupport;
import org.springframework.stereotype.Repository;

import java.util.Date;
import java.util.List;

//@Repository("userDao")
public class UserDaoImpl extends JdbcDaoSupport implements UserDao {

    @Override
    public void save(User user) {
        String username = user.getUsername();
        Date birthday = user.getBirthday();
        String gender = user.getGender();
        Float money = user.getMoney();

        getJdbcTemplate().update("INSERT INTO user (username, birthday, gender, money) VALUES (?, ?, ?, ?)", username, birthday, gender, money);
    }

    @Override
    public void delete(Integer id) {
        getJdbcTemplate().update("DELETE FROM user WHERE id=?", id);
    }

    @Override
    public void update(User user) {
        Integer id = user.getId();
        String username = user.getUsername();
        Date birthday = user.getBirthday();
        String gender = user.getGender();
        Float money = user.getMoney();

        getJdbcTemplate().update("UPDATE user SET username=?, birthday=?, gender=?, money=? WHERE id=?", username, birthday, gender, money, id);

    }

    @Override
    public User findById(Integer id) {
        User user = getJdbcTemplate().queryForObject("SELECT * FROM user WHERE id=?", new BeanPropertyRowMapper<>(User.class), id);
        return user;
    }

    @Override
    public User findByName(String username) {
        User user = getJdbcTemplate().queryForObject("SELECT * FROM user WHERE username=?", new BeanPropertyRowMapper<>(User.class), username);
        return user;
    }

    @Override
    public List<User> findAll() {
        List<User> users = getJdbcTemplate().query("SELECT * FROM user", new BeanPropertyRowMapper<>(User.class));
        return users;
    }
}
```

# Spring中的事务管理

## Spring事务管理前言

1. JavaEE体系进行分层开发，事务处理位于业务层，Spring提供了分层设计业务层的事务处理解决方案。
2. Spring框架为我们提供了一组事务控制的接口。这组接口是在spring-tx-5.0.2.RELEASE.jar中的。
3. Spring的事务控制都在基于AOP的，它既可以使用编程的方式实现，也可以使用配置的方式实现。

## Spring中事务管理的API

1. PlatformTransactionManager
   - PlatformTransactionManager事务管理器，它给我们提供了常用的操作事务的方法。

```java
/*
在开发中我们都是使用它的实现类：
	1.使用Spring JDBC或iBatis进行持久化数据时使用：
	org.springframework.jdbc.datasource.DataSourceTransactionManager
	2.使用Hibernate版本进行持久化数据时使用：
	org.springframework.orm.hibernate5.HibernateTransactionManager
*/
// 获取事务状态信息
TransactionStatus getTransaction(TransactionDefinition definition);
// 提交事务
void commit(TransactionStatus status);
// 回滚事务
void rollback(TransactionStatus status);
```

2. TransactionDefinition
   - TransactionDefinition是事务的定义信息对象。

```java
// 获取事务对象名称
String getName();
// 获取事务隔离级别
int getIsolationLevel();
// 获取事务传播行为
int getPropagationBehavior();
// 获取事务超时时间
int getTimeout();
// 获取事务是否只读
// 读写型事务：增加、删除或修改，开启事务；只读型事务：执行查询，也开启事务。
boolean isReadOnly();
```

3. TransactionStatus
   - TransactionStatus此接口提供事务具体的运行状态，它描述了某个时间点上，事务对象的状态信息。

```java
// 刷新事务
void flush();
// 获取是否存在存储点
boolean hasSavepoint();
// 获取事务是否完成
boolean isCompleted();
// 获取事务是否为新的事务
boolean isNewTransaction();
// 获取事务是否回滚
boolean isRollbackOnly();
// 设置事务回滚
void setRollbackOnly();
```

### 事务的隔离级别

- 事务隔离级别反映事务提交并发访问时的处理态度：
  - ISOLATION_DEFAULT：默认级别，归属下列某一种；
  - ISOLATION_READ_UNCOMMITTED：可以读取未提交数据；
  - ISOLATION_READ_COMMITTED：只能读取已提交数据，解决脏读问题（Oracle默认级别）；
  - ISOLATION_REPEATABLE_READ：是否读取其他事务提交修改的数据，解决不可重复读的问题（MySQLD默认级别）；
  - ISOLATION_SERIALIZABLE：是否读取其他事务提交添加后的数据，解决幻读问题；

### 事务的传播行为

|     value     |                           comment                            |
| :-----------: | :----------------------------------------------------------: |
|   REQUIRED    | 如果当前没有事务，那就新建一个事务；如果已经存在一个事务，加入到这个事务中。默认值。 |
|   SUPPORTS    | 支持当前事务，如果当前没有事务，就以非事务方式执行（没有事务，也就是自动提交）。 |
|   MANDATORY   |        使用当前的事务，如果当前没有事务，就抛出异常。        |
|  REQUERS_NEW  |        新建事务，如果当前在事务中，就把当前事务挂起。        |
| NOT_SUPPORTED |  以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。  |
|     NEVER     |        以非事务方式运行，如果当前存在事务，抛出异常。        |
|    NESTED     | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行REQUIRED类似的操作。 |

### 超时时间

- 默认值是-1，没有超时限制。如果有，以秒为单位进行设置。

### 是否是只读事务

- 建议查询时设置为只读。

## TransactionManager事务管理

- Spring中的TransactionManager之前在TransactionTemplate中提及过：
  - TransactionTemplate可以帮助我们进行事务的管理，要创建TransactionTemplate需要传入一个DataSourceTransactionManager对象，创建这个对象需要传入一个数据源DataSource。
  - 之前的实例中，JdbcTemplate和TransationManager同使用一个数据源，我们通过TransactionManager对象创建了TransactionTemplate对象，并在proxy或enhancer动态代理中使用TransactionTemplate对象帮助我们进行事务的管理。
- Spring可以也可以通过只TransactionManager的方式实现事务的管理，有两种方式：
  - 通过xml文件配置TransactionManager对象，配合AOP实现对业务层的事务管理；
  - 通过注解的方式配置事务管理（不需要依赖AOP）；
  - 无论通过哪种方式，事务管理中的异常不会被捕获，需要自行捕获。

### 基于XML文件的配置

- 基于配置文件的方式，要使用到aop，所以务必记得添加依赖。

```xml
<!-- aop配置依赖spring-aspects(重要) -->
<dependency>	
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>5.2.6.RELEASE</version>
</dependency>
```

#### transaction.xml

- \<tx:attributes>中配置\<tx:method>标签内事务属性一览：

|      属性       |                             作用                             |
| :-------------: | :----------------------------------------------------------: |
|      name       |                指定方法名称，是业务核心方法。                |
|    read-only    |             是否是只读事务。默认false，不只读。              |
|    isolation    |    指定事务的隔离级别。默认值是使用数据库的默认隔离级别。    |
|   propagation   |                     指定事务的传播行为。                     |
|     timeout     |            指定超时时间。默认值为：-1。永不超时。            |
|  rollback-for   | 用于指定一个异常，当执行产生该异常时，事务回滚。产生其他异常，事务不回滚。没有默认值，任何异常都回滚。 |
| no-rollback-for | 用于指定一个异常，当产生该异常时，事务不回滚，产生其他异常时，事务回滚。没有默认值，任何异常都回滚。 |

- UserDao已经使用xml文件的方式配置了，不需要再UserDaoImpl中使用注解配置了，此版使用继承JdbcDaoSupport类的方式，所以不需要为UserDao注入JdbcTemplate了，直接注入DataSource即可。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:component-scan base-package="cn.dylanphang.xml"/>
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!-- 使用c3p0数据库连接池 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${jdbc.driver}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!-- UserDao使用继承JdbcDaoSupport类的方式，因此要给UserDao注入一个数据源 -->
    <bean id="userDao" class="cn.dylanphang.xml.dao.impl.UserDaoImpl">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 在模拟的Client中会调用业务层里手动引发的异常，导致转账过程出错，需要事务管理 -->
    <!-- 以下使用TransactionManager和AOP配置事务管理 -->

    <!-- 1.配置一个事务管理器 -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 2.配置事务通知引用事务管理器 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <!-- 3.配置事务属性 -->
            <tx:method name="transfer" read-only="false" propagation="REQUIRED"/>
            <tx:method name="*" read-only="true" propagation="SUPPORTS"/>
        </tx:attributes>
    </tx:advice>

    <!-- 4.配置切入点表达式和事务通知的对应关系 -->
    <aop:config>
        <aop:pointcut id="pc1" expression="execution(* cn.dylanphang.xml.service.impl.*.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pc1"/>
    </aop:config>
</beans>
```

#### UserServiceClient.java

- 事务管理中不会进行异常的处理，需要在程序中手动对可能出现异常的语句进行抓取，否则后续代码将不会被执行。

```java
package cn.dylanphang.xml.client;

import cn.dylanphang.xml.pojo.User;
import cn.dylanphang.xml.service.UserService;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
public class UserServiceClient {
    public static void main(String[] args) {

        ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("transaction.xml");

        UserService userService = ac.getBean("userService", UserService.class);


        // 1.转账，手动引发异常，因此需要事务管理
        try {
            userService.transfer("dylan", "hanna", (float) 500);
        } catch (Exception e) {
            System.out.println("Something wrong.");
        }

        // 2.查询表中所有数据
        List<User> users = userService.findAllUser();
        for (User user : users) {
            System.out.println(user);
        }
        ac.close();
    }
}
```



### 基于注解的配置

#### jdbc.properties

- 基于纯注解的配置，我们仍然依赖于jdbc的配置文件。

```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring?serverTimezone=UTC
jdbc.username=root
jdbc.password=root
```

#### SpringConfiguration.java

- 开启Spring对@Transactional注解的支持（位于UserServiceImpl.java中）：
  - 在配置类中添加@EnableTransactionManagement注解；
  - 在配置文件中添加\<tx:annotation-driven transaction-manager="transactionManager"/>。

```java
package cn.dylanphang.annotation.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@Configuration
@ComponentScan("cn.dylanphang.annotation")
@EnableTransactionManagement
@Import({JdbcConfig.class, TransactionConfig.class})
public class SpringConfiguration {
}
```

#### JdbcConfig.java

- 配置类中的@Bean都是可以在配置文件中进行配置的。

```java
package cn.dylanphang.annotation.config;

import com.mchange.v2.c3p0.ComboPooledDataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.jdbc.core.JdbcTemplate;

import javax.sql.DataSource;
import java.beans.PropertyVetoException;

@Configuration
@PropertySource("classpath:jdbc.properties")
public class JdbcConfig {

    @Value("${jdbc.driver}")
    private String driver;

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean("dataSource")
    public DataSource getDataSource() {
        try {
            ComboPooledDataSource comboPooledDataSource = new ComboPooledDataSource();

            comboPooledDataSource.setDriverClass(driver);
            comboPooledDataSource.setJdbcUrl(url);
            comboPooledDataSource.setUser(username);
            comboPooledDataSource.setPassword(password);

            return comboPooledDataSource;
        } catch (PropertyVetoException e) {
            e.printStackTrace();
        }
        return null;
    }

    @Bean("jdbcTemplate")
    public JdbcTemplate getJdbcTemplate() {
        return new JdbcTemplate(this.getDataSource());
    }
}
```

#### TransactionConfig.java

- 配置类中的@Bean都是可以在配置文件中进行配置的。

- 仍然需要获取到TransactionManager对象，进行事务管理。

```java
package cn.dylanphang.annotation.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.TransactionManager;

import javax.sql.DataSource;

@Configuration
@Import(JdbcConfig.class)
public class TransactionConfig {

    private DataSource dataSource;

    public TransactionConfig(@Autowired DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean("transactionManager")
    public TransactionManager getTransactionManager() {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

#### UserServiceImpl.java

- 最重要的一步是在UserService中加入@Transactional注解，并配置属性。

```java
package cn.dylanphang.annotation.service.impl;

import cn.dylanphang.annotation.dao.UserDao;
import cn.dylanphang.annotation.pojo.User;
import cn.dylanphang.annotation.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service("userService")
@Transactional(readOnly = true, propagation = Propagation.SUPPORTS)
public class UserServiceImpl implements UserService {

    private UserDao userDao;

    @Autowired
    public UserServiceImpl(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    @Transactional(readOnly = false, propagation = Propagation.REQUIRED)
    public void transfer(String sourceName, String targetName, Float money) {
        User sourceUser = userDao.findByName(sourceName);
        User targetUser = userDao.findByName(targetName);

        sourceUser.setMoney(sourceUser.getMoney() - money);
        targetUser.setMoney(targetUser.getMoney() + money);

        userDao.update(sourceUser);
        int i = 1 / 0; // 模拟转账异常，则异常前的事务都会成功提交，异常后的代码将不会执行。需要事务管理。
        userDao.update(targetUser);
    }

    @Override
    public void saveUser(User user) {
        this.userDao.save(user);
    }

    @Override
    public void deleteUser(Integer id) {
        this.userDao.delete(id);
    }

    @Override
    public void updateUser(User user) {
        this.userDao.update(user);
    }

    @Override
    public User findUserById(Integer id) {
        return this.userDao.findById(id);
    }

    @Override
    public User findUserByName(String username) {
        return this.userDao.findByName(username);
    }

    @Override
    public List<User> findAllUser() {
        return this.userDao.findAll();
    }
}
```

#### UserServiceClient.java

- 事务管理中不会进行异常的处理，需要在程序中手动对可能出现异常的语句进行抓取，否则后续代码将不会被执行。

```java
package cn.dylanphang.annotation.client;

import cn.dylanphang.annotation.config.SpringConfiguration;
import cn.dylanphang.annotation.pojo.User;
import cn.dylanphang.annotation.service.UserService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
public class UserServiceClient {
    public static void main(String[] args) {

        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SpringConfiguration.class);

        UserService userService = ac.getBean("userService", UserService.class);


        // 1.转账，手动引发异常，因此需要事务管理
        try {
            userService.transfer("dylan", "hanna", (float) 500);
        } catch (Exception e) {
            System.out.println("Something wrong.");
        }

        // 2.查询表中所有数据
        List<User> users = userService.findAllUser();
        for (User user : users) {
            System.out.println(user);
        }
        ac.close();
    }
}
```