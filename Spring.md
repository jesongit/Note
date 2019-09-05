# Spring

## Spring 概念
1. spring 是开源的轻量级框架
2. spring 是一站式框架
    - 在 javaEE 三层结构中提供了不同的解决技术
    > web层：SpringMVC  service层：ioc dao层：jdbcTemplate
3. spring 核心
   - aop：面向切面编程，扩展功能不是修改源代码实现
   - ioc：控制反转
    > 控制反转：把对象创建交给Spring配置创建

## Spring IOC
### IOC 底层原理
#### 使用技术
- xml配置文件
- dom4j 解决 xml
- 工厂设计模式
- 反射
#### 实现原理
~~~Java
public class User {
    public void add() {};
}
// 调用方法
User user = new User();
user.add()
// 缺点耦合度太高。当类名、方法名发生改变需要修改多处
------------------------------------------------------------

// 使用工厂模式解耦合
public class UserService {
    public void add() {}
}
public class Factory {
    // 提供返回UserService对象的方法
    public static UserService getService() {
        return new UserService();
    }
}
public class UserServlet {
    UserService s = Factory.getService();
    s.add(); 
}
// 问题：servlet 和 工厂出现了新的耦合
------------------------------------------------------------

// IOC 原理
public class UserService {
    public void add() {}
}
// 1. 创建配置文件
<bean id = "userService" class = "xx.xxx.UserService"/>

// 2. 创建工厂类 解析xml + 反射
public class UserFactory {
    //  返回 UserService 对象的方法
    public static UserService getService() {
        // 解析xml文件
        String classValue = "";  // 使用dom4j得到
        // 反射创建类对象
        Class clazz = Class.forName(classValue);
        // 创建类对象
        Userservice service = clazz.newInstance();
        return service;
    }
}

// 调用方法得到userService
public class UserServlet {
    UserService userService = UserFactoryu.getService();
}
~~~
### 示例
1. 导入 jar 包（最基本的包）
    - commons-logging
    - log4j
    - spring-beans
    - spring-core
    - spring-context
    - spring-expression
2. 创建类、方法
3. 配置 xml 文件
    - 配置文件**位置和名字不固定**
    > 建议放在 src 下，官方建议名称applicationContext.xml
    - 引入约束（IDEA 会自动引入）
    - 配置对象创建

## Spring 的 Bean 管理
### Bean 实例化的三种方式
1. **配置文件**
~~~xml
    <!-- 无参构造配置 主要方式-->
    <bean id="user" class="xin.jeson.User"/>
    <!-- 静态工厂配置 -->
    <bean id="user" class="xin.jeson.UserFactoryByStatic" factory-method="getUser"/>
    <!-- 实例工厂配置 -->
    <bean id="userFactory" class="xin.jeson.UserFactory"/>
    <bean id="user" factory-bean="userFactory" factory-method="getUser"/>
~~~
2. **调用方法**
~~~Java
// 加载 xml
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        User user = (User)context.getBean("user");  // 获取对象
        System.out.println(user);
~~~
3. **无参构造方法创建**
   > 必须存在无参构造，默认存在，但如果声明了有参构造默认就没有无参构造
4. **使用静态工厂创建**
~~~Java
// 构建工厂类
public class UserFactoryByStatic {
    public static User getUser() {
        return new User();
    }
}
~~~
4. **使用实例工厂创建**
~~~Java
// 构建工厂类
public class UserFactory {
    public User getUser() {
        return new User();
    }
}
~~~
### Bean 标签常用属性
1. **id**：Bean 在使用时的名字
2. **class**：Bean 类的**全路径**
3. **name**：和 id 功能一样（但是 name 可以包含特殊符号，向下兼容保留属性）
4. **scope**：设置类的**作用范围**
    - **singleton** ：**默认值**，单例的
    - **protoype**  ：多例的
    - **request**        ：Web 项目中，将对象存入 request 中
    - **session**        ：Web 项目中，将对象存入 session 中
    - **globalSession**  ：Web 项目中，Porlet 环境中存在，否则相当于 session
### 属性注入
~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- xmlns:p 只有使用 P 命名空间注入才需要此约束 -->

    <bean id="userDao" class="xin.jeson.UserDao"/>

    <!-- 使用 setter 注入 (主要的注入方式) -->
    <bean id="user" class="xin.jeson.User">
        <property name="name" value="Jeson"/>
        <property name="userDao" ref="userDao"/>
    </bean>
    <!-- 使用有参构造注入 -->
    <bean id="user" class="xin.jeson.User">
        <constructor-arg name="name" value="Jeson"/>
        <constructor-arg ref="userDao"/>
    </bean>-->
    <!-- P 命名空间注入 -->
    <bean id="user" class="xin.jeson.User" p:name="Jeson" p:userDao-ref="userDao"/>
</beans>
    <!-- 另有注解注入 -->
~~~
### 注解注入
> **需要导入 spring-aop jar 包**
> **配置文件中需要加入 context 约束**
~~~xml
// applicationContext.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 开启扫描注解 -->
    <context:component-scan base-package="xin"/>
    <!-- 多个包可以逗号隔开，如果包太多可以直接写最大的包名字，会包含所有子包 -->

    <!-- 仅扫描属性上面的注释 -->
    <context:annotation-config/>
</beans>
~~~
#### 创建对象有四个注解
 - @Component
 - @Controller
 - @Service
 - @Repository
#### 单/多实例
@Scope(value="prototype")
#### 注解注入属性
~~~Java
@Component(value="userDao")
public class UserDao {
    public void show() {
        System.out.println("dao");
    }
}
@Service(value="userService")
public class UserService {

    @Autowried               // 根据类的类型名字找类
    private UserDao userDao;

    @Resource(name="userDao") // 根据定义类的名字找类
    private UserDao userDao;

    public void show() {
        System.out.println("service");
        userDao.show();
    }
}
~~~
##### 注入属性的 3 种方式
1. Field 注入
~~~Java
@Autowried               // 同上示例
private UserDao userDao;
~~~
2. 构造器注入（Spring 4.x 推荐）
~~~Java
private UserDao userDao;
@Autowired
public UserService(UserDao userDao) {
    this.userDao = userDao;
}
~~~
3. Setter 注入（Spring 3.x 推荐）
~~~Java
private UserDao userDao;
@Autowired
public void setUserDao(UserDao userDao) {
    this.userDao = userDao;
}
~~~
#### 配置文件和注解混合使用
> 配置文件中创建对象，使用注解进行属性注入

## AOP
### AOP 概述
1. AOP 即面向切面编程
2. AOP 采取横向抽取机制
3. AOP 使用动态代理实现
### AOP 原理
~~~Java
public class User {
    public void addUser() {
        // TODO
    }
}
// 扩展功能：添加日志功能（什么时候添加的用户）

// 方案一
直接在 TODO 中修改源代码，添加日志逻辑

// 方案二 纵向抽取机制
public class BaseUser {
    public void writeLog() {
        // TODO
    }
}
public class User extend BaseUser{
    public void addUser() {
        // TODO
        // 调用父类方法实现日志功能
        super.writeLog();
        // 当父类方法名改变，仍然需要修改源代码
    }
}

// 方案三 横向抽取机制(AOP)
// 情况一（有接口）
public interface Dao {
    public void addUser();
}
public class DaoImpl implements Dao {
    public void addUser() {
        // TODO 旧逻辑
    }
}
// 使用动态代理的方式创建接口实现类的代理对象
// 即创建与 DaoImpl 类的平级对象，增强旧逻辑功能
// 实现与 DaoImpl 相同的功能
// Jdk 动态代理
// 情况二 无接口
// 没有接口情况，同样使用动态代理
// 使用子类的代理对象调用父类的方法完成增强
// Cglib 动态代理
~~~
### AOP 操作术语
~~~Java
public class User {
    public void add() {

    }
    public void update() {

    }
    public void findAll() {

    }
}
~~~
1. Joinpiont（连接点）
    > 可以被增强的方法,如 add()等
2. **Pointcut（切入点）**
    > 实际被增强的方法
3. **Adcice（通知/增强）**
    > 即扩展（增强）的功能（逻辑）
    - **前置通知**：方法之前执行
    - **后置通知**：方法之后执行
    - **异常通知**：方法出现异常时执行
    - **最终通知**：后置之后执行
    - **环绕通知**：方法之前之后都执行
    > 前置-> 环绕-> 后置 -> 最终
4. **Aspect（切面）**
    > 把增强应用到方法上称之为切面
5. Introduction（引介）
   > 可以用于动态添加属性或方法
6. Target（目标对象）
   > 增强逻辑所在的类
7. Weaving（织入）
    > 增强的过程
8. Proxy（代理）
   > 织入后的类称之为代理类
### AOP 操作
> 使用 Aspect 框架实现
> 只有 Spring 2.x 以上支持
#### 准备
1. 导入 Jar 包
   - **aopalliance**
   - **aspectjweaver**
   - **spring-aop**
   - **spring-sepects**
2. 添加约束
   - aop
   - spring-aop
#### 使用表达式配置切入点
~~~
execution(<访问修饰符>?<返回类型><方法名>(<参数>)<异常>)
例如:
execution(* xin.jeson.User.show(..)) // 增强 User 类中的show方法
execution(* xin.jeson.User.*(..))    // 增强 User 类中的所有方法
execution(* *.*(..))                 // 增强所有类中的所有方法
execution(* xx*(..))                 // 增强所有 xx 开头的方法
~~~
#### 使用 Aspect 实现 AOP 的两种方式

1. **xml 配置文件进行增强**
~~~Java
// 需要被增强的类
public class User {
    public void show() {
        System.out.println("User");
    }
}
// 增强逻辑的类
public class MyUser {
    // 前置通知
    public void beforeShow() {
        System.out.println("Before");
    }
    // 后置通知
    public void afterShow() {
        System.out.println("After");
    }
    // 环绕通知
    public void aroundShow(proceedingJoinPoint proceedingJoinPoint) {
        // 方法之前
        System.out.println("BeforeByAround");

        // 执行被增强的方法
        proceedingJoinPoint.proceed();

        // 方法之后
        System.out.println("AfterByAround");
    }
}
~~~
~~~ xml
<!-- 配置对象 -->
<bean id="user" class="xin.jeson.User"/>
<bean id="myUser" class="xin.jeson.MyUser"/>
<!-- 配置 AOP 操作 -->
<aop:config>
    <!-- 配置切入点 -->
    <aop:pointcut expression="execution(* xin.jeson.User.*(..))" id="pointcut"/>
    <!-- 配置切面 -->
    <aop:aspect ref="myUser">
        <!-- 将 myUser 中的方法配置到切入点 -->
        <aop:before method="beforeShow" pointcut-ref="pointcut"/>
        <aop:after-returning method="afterShow" pointcut-ref="pointcut"/>
        <aop:around method="aroundShow" pointcut-ref="pointcut"/>
    </aop:aspect>
</aop:config>
~~~
2. 使用注解进行 AOP 操作
~~~xml
<!-- 开启 AOP 操作 -->
<aop:aspectj-autoproxy/>
~~~
~~~Java
// 需要被增强的类
@Component(value="user")
public class User {
    public void show() {
        System.out.println("User");
    }
}
// 增强逻辑的类
@Component(value="myUser")
@Aspect
public class MyUser {
    // 前置通知
    @Before(value="execution(* xin.jeson.User.show(..))")
    public void beforeShow() {
        System.out.println("Before");
    }

    // 后置通知
    @AfterReturning(value="execution(* xin.jeson.User.show(..))")
    public void afterShow() {
        System.out.println("After");
    }
    // 环绕通知
    @Around(value="execution(* xin.jeson.User.show(..))")
    public void aroundShow(proceedingJoinPoint proceedingJoinPoint) {
        // 方法之前
        System.out.println("BeforeByAround");

        // 执行被增强的方法
        proceedingJoinPoint.proceed();

        // 方法之后
        System.out.println("AfterByAround");
    }
    // @After 最终通知
}
~~~

## Spring 整合 Web 项目
> ApplicationContext 类处于 Action 层，每产生一个 Action 会存在效率问题
### 实现原理
1. ServletContext 对象（只有唯一的一个对象）
2. 监听器
> 在服务器启动时，为每个项目创建一个 ServletContext 对象
> 监听到 ServeletContext 创建时，加载 Spring 配置文件并创建配置好的对象
> 将对象放到 ServeletContext 对象中 (setArrtibute方法)
> 获取对象(getAttribute方法)
### 配置监听器
> 需要导入 spring-web jar 包
~~~xml
// web.xml
<listener>
    <listener-class> org.springframework.web.context.ContextLoaderListener </listener-class>
</listener>
~~~
### 指定 spring 配置文件位置
~~~xml
// web.xml
// 默认会找\WEB-INF\applicationContext.xml
<context-parm>
    <param-name> contextConfigLocation </param-name>
    <param-value> classpath:applicationContext.xml </param-value>
</context-parm>
~~~

## Spring 的 JdbcTemplate 操作
> Spring 对不同的持久化技术都进行了封装 (Dao 层技术)
### 准备
1. 导入 Jar 包
   - spring-jdbc
   - spring-tx
   - jdbc（数据库驱动的 jar 包，如：mysql-connector）
### 数据库操作
~~~Java
// 1。 创建对象，设置数据库信息
DriverManagerDataSource dataSource = new DriverManagerDataSource();
dataSourrce.setDriverClassName("com.mysql.jdbc.Driver");
dataSource.setUrl("jdbc:mysql:///test");
dataSource.setUsername("root");
dataSource.setPardword("root");
// 2. 创建 jdbcTemplate 对象，设置数据源
JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
// 3. 使用 jdbcTemplate 的方法进行操作
// 增加数据
String sql = "insert into table values(?,?)";
int rows = jdbcTemplate.updae(sql, "Jeson", "100");
System.out.println(rows);
// 删除数据
String sql = "delete from table where xx=?";
int rows = jdbcTemplate.update(sql, "xxx");
System.out.println(rows);
// 修改数据
String sql = "update table set xx=? where xx=?";
int rows = jdbcTemplate.update(sql, "xxx", "xxx");
System.out.println(rows);
// 查询数据
// 查询返回一个值
String sql = "seletc count(*) from table";
int count = jdbcTemplate.queryForObject(sql, Integer.class);
System.out.println(count);
// 查询返回对象
class MyRowMapper implements RowMapper<User> {
    @Override
    public User mapRow(ResultSet rs, int num) throws SQLException {
        // 从结果集得到数据并封装到自己定义的类中
        String username = rs.getString("username");
        String password = rs.getString("password");

        User user = new User();
        user.setUseranme(username);
        user.setPassword(password);
        return user;
    }
}
String sql = "select * from table where xx=?";
User user = jdbcTemplate.queryForObject(sql, new MyRowMapper(), "xx");
System.out.println(user);
// 查询返回 List
String sql = "select * from talbe";
List list = jdbcTemplate.query(sql, new MyRowMapper());
Syste.out.println(list);
~~~
### Spring 配置连接池和 Dao 使用 JdbcTemplate
#### Spring 配置 c3p0 连接池
> jar包：c3p0 和 machange-commons-java
~~~xml
<!-- 配置连接池 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <!-- 注入属性 -->
    <property name="driverClass" value="com.mysql.jdbc.Driver"/>
    <property name="jdbcUrl" value="jdbc:mysql///test"/>
    <property naem="user" value="root"/>
    <property name="password" value="root"/>
</bean>
~~~
#### Dao 使用 JdacTemplate
~~~xml
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <!-- 注入数据源 -->
    <property name="dataSource" ref="dataSource"/>
</bean>
~~~
~~~Java
@Component(value="userDao")
public class UserDao {

    @Autowired
    private JdbcTemplate jdbcTemplate;
    public void show() {
        String sql = "select * from talble";
        List list = jdbcTemplate.queryForByObject(sql, new MyRowMapper());
        // TODO
    }
}
~~~

## Spring 事务管理
### 事务概念
    - 什么是事务
    - 事务特性
      - 原子性
      - 一致性
      - 隔离性
      - 持久性
    - 读的问题
      - 设置隔离级别
### API 介绍
- 事务管理器
> 针对不从 Dao 层框架，提供了不用的实现类
### 编程式事务管理（不用）
### 声明式事务管理
#### 基于 xml 配置文件
~~~xml
<!-- 已经配置好了 JdbcTemplate 等相关配置 -->
<!-- 1. 配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
<!-- 2. 配置事务增强 -->
<tx:advice id="txAdvice" transaction-maager="transactionManager">
    <!-- 事务操作 -->
    <tx:attributes>
        <!-- 设置事务操作方法匹配规则 -->
        <tx:method name="xxx*" propagation="REQUIRED"/>
        <!-- 以上意思为 以 xxx 开头的方法都可以匹配到 -->
    </tx:attributes>
</tx:advice>
<!-- 3. 配置切面 -->
<aop:config>
    <aop:pointcut expression="execution(* *.*(..))" id="pointcut"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut"/>
</aop:config>
~~~
#### 基于注解实现
~~~xml
<!-- 已经配置好了 JdbcTemplate 等相关配置 -->
<!-- 1. 配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
<!-- 2. 开启事务注解 -->
<tx:annotation-driven transaction-manager="transactionManager"/>
~~~
~~~Java
@Transactional  // 在需要开启事务的类上注解
public class Class {}
~~~

## SSH 框架整合
### Struts2 和 Spring 整合
> Jar 包：Struts2-spring-plugin 和 struts 相关 jar 包
~~~Java
public class UserAction extend ActionSupport {

    @Override
    public String execute() throws Exception {
        return NONE;
    }
}
~~~
~~~xml
<!-- spring 配置文件 -->
<!-- c3p0 连接池配置（默认写好） -->

<!-- 配置 Action 的对象 -->
<bean id="userAction" class="xx.xxx.UserAction" scope="prototype"/>

<!-- struts 配置文件 -->
<struts>
    <package name="xxx" extends="struts-default" namespace="/">
        <!-- 与 spring 整合后 class 只需要写其 id -->
        <action name="userAction" class"userAction" />
    </package>
</struts>
~~~
### Hibernate 和 Spring 整合
> 不需要数据库信息的配置
~~~xml
<!-- c3p0 连接池配置（默认写好） -->
<!-- 配置事务管理器 -->
<bean id="transactionManager" calss="...HibernateTransactionManager">
    <!-- 注入 sessionFactory （也可以注入 dataSource） -->
    <property name="sessionFactory" ref="sessionFactory"/>
</bean>
<!-- 开启事务注解 （同上）-->
<!-- sessionFactory 创建交给 Spring 管理（Spring 中有封装好的类 -->
<bean id="sessionFactory" class="...LocalSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <!-- 指定 Hibernate 配置文件（无配置文件不需要写这个）-->
    <property name="configLocations" value="classpath:hibernate.cfg.xml"/>
    <!-- 也可以直接将 hibernate 配置写在此处，减少配置文件 -->
    <!-- 以下为无 hibernate 配置文件 示例 -->
    <!-- 配置 hibernate 基本信息 -->
    <property name="hibernateProperties">
        <props>
            <prop key="hibernate.show_sql"> true </prop>
            <prop key="hibernate.format_sql"> true </prop>
            <prop key="hibernate.dialet"> org.hibernate.dialect.MySQLDialect </prop>
        </props>
    </property>
    <!-- 配置映射文件 -->
    <property name="mappingResources">
        <List>
            <value> xin/jeson/entity/User.hbm.xml </value>
            <!-- .... -->
        </List>
    </property>
</bean>
~~~
### HibernateTemplate 的使用
~~~xml
<!-- 包含以上整合内容 -->
<!-- 创建 HibernateTemplate 对象 -->
<bean id="hibernateTemplate" class"...">
    <property name="sessionFactory" ref="sessionFactory/>
</bean>
~~~
~~~Java
@Component
public class User {
    // TODO
}
public interface UserDao {
    public void add();
}
@Repository
@Transactional  // 开启事务后正常运行
public class UserDaoImpl implements UserDao {
    private HibernateTemplate hibernateTemplate;
    @Autowired
    public UserDaoImpl(HibernateTemplate hibernateTemplate) {
        this.hibernateTemplate = hibernateTemplate;
    }
    public void add() {
        // 此时无法成功更新数据库，需要使用事务提交
        hibernateTemplate.save(entity);  // 添加操作
    }
}
@Service
public class UserService {
    private UserDao userDao;
    @Autowired
    public UserService(UserDaoImpl userDaoImpl) {
        this.userDao = userDaoImpl;
    }
}
@Controller
public class UserAction {
    private UserService userService;
    @Autowired
    public UserAction(UserService userService) {
        this.userService = userService;
    }
}
~~~
~~~Java
// Hibernate crud 操作
// 添加操作
Serializable save(Object entity)
// 修改操作
void update(Object entity)
// 删除操作
void delete(Object entity)
// 查询操作
<T> T get(Class<T> entityClass, Serializable id)
<T> T load(Class<T> entityClass, Serializable id)
List find(Sring queryString, Object... values)

// 示例
// 1. get 方法：根据 id 查询
User user = hibernateTemplate.get(User.class, id)
// 2. find 查询所有 (HQL 语句查询)
List<User> list = (List<User>)hibernateTemplate.find("from User")
// 3. find 查询 username = jeson 的语句
List<User> list = (List<User>)hibernateTemplate.find("from User where username=?", "jeson")

~~~

## Spring 分模块开发
1. 在 Spring 里面配置多个内容，造成配置混乱，不利于维护
    - 可将部分配置使用单个文件配置
    - 在 Spring 核心文件中引用即可
~~~xml
// 引用格式
<!-- 引入其他配置文件 -->
<import resource="classpath:xx.xml"/>
~~~