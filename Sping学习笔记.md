#  Sping学习笔记

### spring框架概述

1. Spring是轻量级的开源的JavaEE框架

2. Spring可以解决企业应用的复杂性

3. Spring 有二个核心的部分：IOC 和 Aop 

   + IOC控制反转，把创建对象的过程交给Spring进行管理
   + Aop 面向切面编程，不修改源代码进行功能增强

4. Spring 特点 

   + 方便解耦，简化开发
+ Aop编程的支持
   + 方便程序测试
+ 方便集成各种优秀框架
   + 方便进行事务的操作
+ 降低API开发难度

### spring入门案例

1. 先创建工程

2. 导入jre包

   ```xml
   <dependencies>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-context</artifactId>
               <version>5.3.21</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-beans</artifactId>
               <version>5.3.21</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-core</artifactId>
               <version>5.3.21</version>
           </dependency>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-expression</artifactId>
               <version>5.3.21</version>
           </dependency>
           <dependency>
               <groupId>commons-logging</groupId>
               <artifactId>commons-logging</artifactId>
               <version>1.2</version>
           </dependency>
       </dependencies>
   ```

3. 创建一个类

   ```java
   /**
    * Created with IntelliJ IDEA.
    *
    * @Author: 爱吃山楂的小天
    * @Date: 2022/07/28/17:37
    * @Description:
    */
   public class User {
       public void add(){
           System.out.println("add...");
       }
   }
   
   ```

4. 创建spring配置文件，用配置文件创建对象

   + spring配置文件都是xml格式

   + 通过bean标签创建对象,IoC容器中bean的id标签不能重复

   + **id**是bean的唯一标识

   + **class** 用来定义类的全限定名（包名＋类名）。只有子类Bean不用定义该属性。

   + ```XML
     <!-- 配置User类对象-->
         <bean id="user" class="com.it.spring.User"></bean>
     ```

   + 先加载spring配置文件 创建Ioc容器

   + **user**表示bean配置文件里面的别名也就是id名

   +  **User.class** 表示创建出来的Bean的类型

   + ```java
       public static void main(String[] args) {
             //加载spring配置文件
             ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
             
             User user = context.getBean("user", User.class);
             user.add();
         }
     ```

### SpringIoC容器

**IOC**底层原理

**IOC**接口（BeanFactory）

**IOC**操作Bean（基于xml）

**IOC**操作Bean（基于注解）

#### 一、IOC容器

#####  1、什么是IOC（控制反转）

 a）把对象创建和对象之间的调用过程，交给Spring进行管理

 b）使用IOC目的：为了降低耦合度

##### 2、IOC底层

 a）xml解析、工厂模式、反射

##### 3、Spring提供的IOC容器实现的两种方式（两个接口）

 a）BeanFactory接口：IOC容器基本实现是Spring内部接口的使用接口，不提供给开发人员进行使用（加载配置文件时候不会创建对象，在获取对象时才会创建对象。）

 b）ApplicationContext接口：BeanFactory接口的子接口，提供更多更强大的功能，提供给开发人员使用（加载配置文件时候就会把在配置文件对象进行创建）推荐使用！

##### 4、ApplicationContext接口的实现类（具体根据API文档查看☺）

#### 二、IOC操作Bean

##### **1、IOC操作Bean管理**

 a）Bean管理就是两个操作：（1）Spring创建对象；（2）Spring注入属性

##### **2、基于xml方式创建对象**

```xml
<!--配置User对象创建-->
<bean id="user" class="com. it. spring.User">/ bean>
<!--    id属性 对象别名名称唯一-->
<!--    class属性 创建类全路径-->
<!--    name跟id属性差不多但是可以加特殊符号-->
<!--    创建对象时默认执行无参构造-->
</beans>
```

##### **3、基于xml方式注入属性**（DI：依赖注入（注入属性））

###### **使用set方法进行注入**

~~~
//（1）传统方式： 创建类，定义属性和对应的set方法
public class Book {
        //创建属性
        private String name;

        //创建属性对应的set方法
        public void setBname(String name) {
            this.name = name;
        }

   }
~~~

~~~
<!--（2）spring方式： set方法注入属性-->
 <bean id="user" class="com.it.spring.User" name="user2">
    <!--使用property完成属性注入
        name：类里面属性名称
        value：向属性注入的值
    -->
    
        <property name="name" value="张三"></property>
</bean>

~~~

###### **有参构造函数注入**

~~~
//（1）传统方式：创建类，构建有参函数
public class Orders {
    //属性
    private String oname;
    private String address;
    //有参数构造
    public Orders(String oname,String address) {
        this.oname = oname;
        this.address = address;
    }
  }

~~~

~~~
<!--（2）spring方式：有参数构造注入属性-->
<bean id="orders" class="com.atguigu.spring5.Orders">
    <constructor-arg name="oname" value="Hello"></constructor-arg>
    <constructor-arg name="address" value="China！"></constructor-arg>
</bean>
   
~~~

#####  **p名称空间注入（了解即可）**

~~~
<!--1、添加p名称空间在配置文件头部-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"		<!--在这里添加一行p-->

<!--2、在bean标签进行属性注入（算是set方式注入的简化操作）-->
    <bean id="book" class="com.atguigu.spring5.Book" p:bname="very" p:bauthor="good">
    </bean>

~~~

##### **注入空值和特殊符号**

~~~
<bean id="book" class="com.atguigu.spring5.Book">
    <!--（1）null值-->
    <property name="address">
        <null/><!--属性里边添加一个null标签-->
    </property>
    
    <!--（2）特殊符号赋值-->
     <!--属性值包含特殊符号
       a 把<>进行转义 &lt; &gt;
       b 把带特殊符号内容写到CDATA
      -->
        <property name="address">
            <value><![CDATA[<<南京>>]]></value>
        </property>
</bean>
~~~

##### **注入属性-外部bean**

~~~
public class UserService {//service类

    //创建UserDao类型属性，生成set方法
    private UserDao userDao;
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void add() {
        System.out.println("service add...............");
        userDao.update();//调用dao方法
    }
}

public class UserDaoImpl implements UserDao {//dao类

    @Override
    public void update() {
        System.out.println("dao update...........");
    }
}

~~~

~~~
<!--1 service和dao对象创建-->
<bean id="userService" class="com.atguigu.spring5.service.UserService">
    <!--注入userDao对象
        name属性：类里面属性名称
        ref属性：创建userDao对象bean标签id值
    -->
    <property name="userDao" ref="userDaoImpl"></property>
</bean>
<bean id="userDaoImpl" class="com.atguigu.spring5.dao.UserDaoImpl"></bean>

~~~

##### **基于XML方式注入内部bean和级联赋值**

###### 注入属性-内部bean

~~~
//部门类
public class Dept {
    private String dname;
    public void setDname(String dname) {
        this.dname = dname;
    }
}

//员工类
public class Emp {
    private String ename;
    private String gender;
    //员工属于某一个部门，使用对象形式表示
    private Dept dept;
    
    public void setDept(Dept dept) {
        this.dept = dept;
    }
    public void setEname(String ename) {
        this.ename = ename;
    }
    public void setGender(String gender) {
        this.gender = gender;
    }
}

~~~

~~~
<!--内部bean-->
    <bean id="emp" class="com.atguigu.spring5.bean.Emp">
        <!--设置两个普通属性-->
        <property name="ename" value="Andy"></property>
        <property name="gender" value="女"></property>
        <!--设置对象类型属性-->
        <property name="dept">
            <bean id="dept" class="com.atguigu.spring5.bean.Dept"><!--内部bean赋值-->
                <property name="dname" value="宣传部门"></property>
            </bean>
        </property>
    </bean>

~~~

~~~
<!--方式一：级联赋值-->
    <bean id="emp" class="com.atguigu.spring5.bean.Emp">
        <!--设置两个普通属性-->
        <property name="ename" value="Andy"></property>
        <property name="gender" value="女"></property>
        <!--级联赋值-->
        <property name="dept" ref="dept"></property>
    </bean>
    <bean id="dept" class="com.atguigu.spring5.bean.Dept">
        <property name="dname" value="公关部门"></property>
    </bean>

~~~

~~~
 //方式二：生成dept的get方法（get方法必须有！！）
    public Dept getDept() {
        return dept;
    }

~~~

~~~
 <!--级联赋值-->
    <bean id="emp" class="com.atguigu.spring5.bean.Emp">
        <!--设置两个普通属性-->
        <property name="ename" value="jams"></property>
        <property name="gender" value="男"></property>
        <!--级联赋值-->
        <property name="dept" ref="dept"></property>
        <property name="dept.dname" value="技术部门"></property>
    </bean>
    <bean id="dept" class="com.atguigu.spring5.bean.Dept">
    </bean>

~~~



##### **IOC 操作 Bean 管理——xml 注入集合属性**

~~~
//（1）创建类，定义数组、list、map、set 类型属性，生成对应 set 方法
public class Stu {
    //1 数组类型属性
    private String[] courses;
    //2 list集合类型属性
    private List<String> list;
    //3 map集合类型属性
    private Map<String,String> maps;
    //4 set集合类型属性
    private Set<String> sets;
    
    public void setSets(Set<String> sets) {
        this.sets = sets;
    }
    public void setCourses(String[] courses) {
        this.courses = courses;
    }
    public void setList(List<String> list) {
        this.list = list;
    }
    public void setMaps(Map<String, String> maps) {
        this.maps = maps;
    }

~~~

~~~
<!--（2）在 spring 配置文件进行配置-->
    <bean id="stu" class="com.atguigu.spring5.collectiontype.Stu">
        <!--数组类型属性注入-->
        <property name="courses">
            <array>
                <value>java课程</value>
                <value>数据库课程</value>
            </array>
        </property>
        <!--list类型属性注入-->
        <property name="list">
            <list>
                <value>张三</value>
                <value>小三</value>
            </list>
        </property>
        <!--map类型属性注入-->
        <property name="maps">
            <map>
                <entry key="JAVA" value="java"></entry>
                <entry key="PHP" value="php"></entry>
            </map>
        </property>
        <!--set类型属性注入-->
        <property name="sets">
            <set>
                <value>MySQL</value>
                <value>Redis</value>
            </set>
        </property>
</bean>

~~~

##### **在集合里面设置对象类型值**

~~~
  //学生所学多门课程
    private List<Course> courseList;//创建集合
    public void setCourseList(List<Course> courseList) {
        this.courseList = courseList;
    }

~~~

~~~

    <!--创建多个course对象-->
    <bean id="course1" class="com.atguigu.spring5.collectiontype.Course">
        <property name="cname" value="Spring5框架"></property>
    </bean>
    <bean id="course2" class="com.atguigu.spring5.collectiontype.Course">
        <property name="cname" value="MyBatis框架"></property>
    </bean>
    
   	<!--注入list集合类型，值是对象-->
       <property name="courseList">
           <list>
               <ref bean="course1"></ref>
               <ref bean="course2"></ref>
           </list>
       </property>

~~~

~~~
<!--第一步：在 spring 配置文件中引入名称空间 util-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util" <!--添加util名称空间-->
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">  <!--添加util名称空间-->
    
<!--第二步：使用 util 标签完成 list 集合注入提取-->
<!--把集合注入部分提取出来-->
 <!--1 提取list集合类型属性注入-->
    <util:list id="bookList">
        <value>易筋经</value>
        <value>九阴真经</value>
        <value>九阳神功</value>
    </util:list>

 <!--2 提取list集合类型属性注入使用-->
    <bean id="book" class="com.atguigu.spring5.collectiontype.Book" scope="prototype">
        <property name="list" ref="bookList"></property>
    </bean>

~~~

#### 三 Spring [IOC](https://so.csdn.net/so/search?q=IOC&spm=1001.2101.3001.7020)容器-Bean管理

##### **1、IOC 操作 Bean 管理（FactoryBean）**

 1、Spring 有两种类型 bean，一种普通 bean，另外一种工厂 bean（FactoryBean）

 2、普通 bean：在配置文件中定义 bean 类型就是返回类型

 3、工厂 bean：在配置文件定义 bean 类型可以和返回类型不一样 第一步 创建类，让这个类作为工厂 bean，实现接口 FactoryBean 第二步 实现接口里面的方法，在实现的方法中定义返回的 bean 类型

~~~
public class MyBean implements FactoryBean<Course> {

    //定义返回bean
    @Override
    public Course getObject() throws Exception {
        Course course = new Course();
        course.setCname("abc");
        return course;
    }
}

~~~

~~~
<bean id="myBean" class="com.atguigu.spring5.factorybean.MyBean">
</bean>

~~~

~~~
@Test
public void test3() {
 ApplicationContext context =
 new ClassPathXmlApplicationContext("bean3.xml");
 Course course = context.getBean("myBean", Course.class);//返回值类型可以不是定义的bean类型！
 System.out.println(course);
}

~~~

##### **2IOC 操作 Bean 管理（bean 作用域）**

 Spring 里面，默认情况下，bean 是单实例对象，下面进行作用域设置：

（1）在 spring 配置文件 bean 标签里面有属性（scope）用于设置单实例还是多实例

（2）scope 属性值 第一个值 默认值，singleton，表示是单实例对象 第二个值 prototype，表示是多实例对象

#####  **3、IOC 操作 Bean 管理（bean 生命周期）**

1、生命周期 ：从对象创建到对象销毁的过程

2、bean 生命周期

 （1）通过构造器创建 bean 实例（无参数构造）

 （2）为 bean 的属性设置值和对其他 bean 引用（调用 set 方法）

 （3）调用 bean 的初始化的方法（需要进行配置初始化的方法）

 （4）bean 可以使用了（对象获取到了）

 （5）当容器关闭时候，调用 bean 的销毁的方法（需要进行配置销毁的方法）

3、演示 bean 生命周期 ：

~~~
        public class Orders {
         //无参数构造
         public Orders() {
         System.out.println("第一步 执行无参数构造创建 bean 实例");
         }
         private String oname;
         public void setOname(String oname) {
         this.oname = oname;
         System.out.println("第二步 调用 set 方法设置属性值");
         }
         //创建执行的初始化的方法
         public void initMethod() {
         System.out.println("第三步 执行初始化的方法");
         }
         //创建执行的销毁的方法
         public void destroyMethod() {
         System.out.println("第五步 执行销毁的方法");
         }
        }

~~~

~~~
public class MyBeanPost implements BeanPostProcessor {//创建后置处理器实现类
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之前执行的方法");
        return bean;
    }
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之后执行的方法");
        return bean;
    }
}

~~~

~~~
<!--配置文件的bean参数配置-->
<bean id="orders" class="com.atguigu.spring5.bean.Orders" init-method="initMethod" destroy-method="destroyMethod">	<!--配置初始化方法和销毁方法-->
    <property name="oname" value="手机"></property><!--这里就是通过set方式（注入属性）赋值-->
</bean>

<!--配置后置处理器-->
<bean id="myBeanPost" class="com.atguigu.spring5.bean.MyBeanPost"></bean>

~~~

~~~
 @Test
 public void testBean3() {
// ApplicationContext context =
// new ClassPathXmlApplicationContext("bean4.xml");
 ClassPathXmlApplicationContext context =
 new ClassPathXmlApplicationContext("bean4.xml");
 Orders orders = context.getBean("orders", Orders.class);
 System.out.println("第四步 获取创建 bean 实例对象");
 System.out.println(orders);
 //手动让 bean 实例销毁
 context.close();
 }

~~~

#####  **4、IOC 操作 Bean 管理(外部属性文件)**

###### **方式一：直接配置数据库信息** ：

（1）配置Druid（德鲁伊）连接池 （2）引入Druid（德鲁伊）连接池依赖 jar 包

~~~
<!--直接配置连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="url" value="jdbc:mysql://localhost:3306/userDb"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
    </bean>

~~~

###### **方式二：引入外部属性文件配置数据库连接池**

（1）创建外部属性文件，properties 格式文件，写数据库信息（**jdbc.properties**）

~~~
    prop.driverClass=com.mysql.jdbc.Driver
    prop.url=jdbc:mysql://localhost:3306/userDb
    prop.userName=root
    prop.password=root

~~~

（2）把外部 properties 属性文件引入到 spring 配置文件中 —— 引入 context 名称空间

~~~
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd"><!--引入context名称空间-->
    
        <!--引入外部属性文件-->
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!--配置连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${prop.driverClass}"></property>
        <property name="url" value="${prop.url}"></property>
        <property name="username" value="${prop.userName}"></property>
        <property name="password" value="${prop.password}"></property>
    </bean>
    
</beans>

~~~

**5、IOC 操作 Bean 管理xml方式(自动装配)**

~~~
<!--    实现自动装配
        bean标签属性autowire
        auto wire属性二个常用值
        byName 根据属性名自动装配 注入bean的id值和属性名一致
        byType 根据属性类型自动装配
-->

    <bean id="user" class="com.it.spring.User" autowire="byName"></bean>
    <bean id="book" class="com.it.spring.Book"></bean>
</beans>
~~~

~~~
public class User {

    Book book;
    String name;

    public void setBook(Book book) {
        this.book = book;
    }
}

~~~

##### 5、[IOC](https://so.csdn.net/so/search?q=IOC&spm=1001.2101.3001.7020) 操作 Bean 管理(基于注解方式)

######  **1、什么是注解**

 （1）注解是代码特殊标记，格式：@注解名称(属性名称=属性值, 属性名称=属性值…)

 （2）使用注解，注解作用在类上面，方法上面，属性上面

 （3）使用注解目的：简化 [xml](https://so.csdn.net/so/search?q=xml&spm=1001.2101.3001.7020) 配置

###### **2、Spring 针对 Bean 管理中创建对象提供注解**

 下面四个注解功能是一样的，都可以用来创建 bean 实例

 （1）@[Component](https://so.csdn.net/so/search?q=Component&spm=1001.2101.3001.7020)

 （2）@Service

 （3）@Controller

 （4）@Repository

###### **3、基于注解方式实现对象创建**

 第一步 引入依赖 （引入**spring-aop jar包**）

 第二步 开启组件扫描

~~~
<!--开启组件扫描
 1 如果扫描多个包，多个包使用逗号隔开
 2 扫描包上层目录
-->
<context:component-scan base-package="com.atguigu"></context:component-scan>

~~~

第三步 创建类，在类上面添加创建对象注解

~~~
//在注解里面 value 属性值可以省略不写，
//默认值是类名称，首字母小写
//UserService -- userService
@Component(value = "userService") //注解等同于XML配置文件：<bean id="userService" class=".."/>
public class UserService {
 public void add() {
 System.out.println("service add.......");
 }
}

~~~

**开启组件扫描细节配置**

~~~
<!--示例 1
 use-default-filters="false" 表示现在不使用默认 filter，自己配置 filter
 context:include-filter ，设置扫描哪些内容
-->
<context:component-scan base-package="com.atguigu" use-defaultfilters="false">
 <context:include-filter type="annotation"

expression="org.springframework.stereotype.Controller"/><!--代表只扫描Controller注解的类-->
</context:component-scan>
<!--示例 2
 下面配置扫描包所有内容
 context:exclude-filter： 设置哪些内容不进行扫描
-->
<context:component-scan base-package="com.atguigu">
 <context:exclude-filter type="annotation"

expression="org.springframework.stereotype.Controller"/><!--表示Controller注解的类之外一切都进行扫描-->
</context:component-scan>

~~~

###### **基于注解方式实现属性注入**

（1）@Autowired：根据属性类型进行自动装配

~~~
@Service
public class UserService {

 @Autowired
 private UserDao userDao;
 public void add() {
 System.out.println("service add.......");
 userDao.add();
 }
}

@Repository
public class UserDaoImpl implements UserDao {
    @Override
    public void add() {
        System.out.println("dao add.....");
    }
}

~~~

@Qualifier：根据名称进行注入，这个@Qualifier 注解的使用，和上面@Autowired 一起使用

```java
@Autowired 
//根据名称进行注入（目的在于区别同一接口下有多个实现类，根据类型就无法选择，从而出错！）
@Qualifier(value = "userDaoImpl1") 
```

@Resource：可以根据类型注入，也可以根据名称注入（它属于javax包下的注解，不推荐使用！）

~~~
//@Resource //根据类型进行注入
@Resource(name = "userDaoImpl1") //根据名称进行注入
private UserDao userDao;

~~~

@Value：注入普通类型属性

~~~
@Value(value = "abc")
private String name
~~~

### Spring AOP

#### **1、AOP 基本概念**

 （1）[面向切面编程](https://so.csdn.net/so/search?q=面向切面编程&spm=1001.2101.3001.7020)（方面），利用 AOP 可以对业务逻辑的各个部分进行隔离，从而使得 业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

 （2）通俗描述：不通过修改源代码方式，在主干功能里面添加新功能

 （3）使用登录例子说明 AOP

#### **2、AOP（底层原理）**

 	AOP 底层使用动态代理 ，动态代理有两种情况：

> 第一种 有接口情况，使用 JDK 动态代理 ；创建**接口实现类代理对象**，增强类的方法
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200702135134128.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQ5NjE5MA==,size_16,color_FFFFFF,t_70)

> 第二种 没有接口情况，使用 CGLIB 动态代理；创建**子类的代理对象**，增强类的方法
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020070213514980.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTQ5NjE5MA==,size_16,color_FFFFFF,t_70)

#### 3、Aop（jDK动态代理）

 1）使用 JDK 动态代理，使用 Proxy 类里面的方法创建代理对象

```
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h)

```

 第一参数，类加载器

 第二参数，增强方法所在的类，这个类实现的接口，*支持多个接口*

 第三参数，实现这个接口 InvocationHandler，创建代理对象，写增强的部分

![S30213-16522277](D:\Application\QQ\qq消息数据\2244696539\FileRecv\MobileFile\S30213-16522277.png)

 2）编写 JDK 动态代理代码

~~~
//（1）创建接口，定义方法
public interface UserDao {
 public int add(int a,int b);
 public String update(String id);
}

~~~

~~~
//（2）创建接口实现类，实现方法
public class UserDaoImpl implements UserDao {
 @Override
 public int add(int a, int b) {
 return a+b;
 }
 @Override
 public String update(String id) {
 return id;
 }
}

~~~

~~~~
//（3）使用 Proxy 类创建接口代理对象
public class JDKProxy {
 public static void main(String[] args) {
 //创建接口实现类代理对象
 Class[] interfaces = {UserDao.class};
 UserDaoImpl userDao = new UserDaoImpl(); 
/** 第一参数，类加载器 
	第二参数，增强方法所在的类，这个类实现的接口，(支持多个接口)
	第三参数，实现这个接口 InvocationHandler，创建代理对象，写增强的部分  */
 UserDao dao =(UserDao)Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interfaces,
					new UserDaoProxy(userDao));
 int result = dao.add(1, 2);
 System.out.println("result:"+result);
 }
}

//创建代理对象代码
class UserDaoProxy implements  {
 //1 把创建的是谁的代理对象，把谁传递过来
 //有参数构造传递
 private Object obj;
 public UserDaoProxy(Object obj) {
 this.obj = obj;
 }
 //增强的逻辑
 @Override
 public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
 //方法之前
 System.out.println("方法之前执行...."+method.getName()+" :传递的参数..."+ Arrays.toString(args));
 //被增强的方法执行
 Object res = method.invoke(obj, args);
 //方法之后
 System.out.println("方法之后执行...."+obj);
 return res;
 }
}

~~~~

#### 4、 **AOP（术语）**

1. 连接点：类里面哪些方法可以被增强，这些方法称为连接点

2. 切入点：实际被真正增强的方法称为切入点

3. 通知（增强）：实际增强的逻辑部分称为通知，且分为以下五种类型：

   前置通知 2）后置通知 3）环绕通知 4）异常通知 5）最终通知

4. 切面：把通知应用到切入点过程

#### 5、**AOP操作**

- Spring 框架一般都是基于 AspectJ 实现 AOP 操作，AspectJ 不是 Spring 组成部分，独立 AOP 框架，一般把 AspectJ 和 Spirng 框架一起使 用，进行 AOP 操作

- 基于 AspectJ 实现 AOP 操作：1）基于 xml 配置文件实现 （2）基于注解方式实现（使用）

- 切入点表达式，如下：

  ```
  （1）切入点表达式作用：知道对哪个类里面的哪个方法进行增强 
  （2）语法结构： execution([权限修饰符] [返回类型] [类全路径] [方法名称]([参数列表]) )
  （3）例子如下：
      例 1：对 com.atguigu.dao.BookDao 类里面的 add 进行增强
  		execution(* com.atguigu.dao.BookDao.add(..))
   	例 2：对 com.atguigu.dao.BookDao 类里面的所有的方法进行增强
  		execution(* com.atguigu.dao.BookDao.* (..))
      例 3：对 com.atguigu.dao 包里面所有类，类里面所有方法进行增强
  		execution(* com.atguigu.dao.*.* (..))
  
  ```

#### 6 、**AOP 操作（AspectJ 注解）**

````
//1、创建类，在类里面定义方法
public class User {
 public void add() {
 System.out.println("add.......");
 }
}
//2、创建增强类（编写增强逻辑）
//（1）在增强类里面，创建方法，让不同方法代表不同通知类型
//增强的类
public class UserProxy {
 public void before() {//前置通知
 System.out.println("before......");
 }
}


````

````
<!--3、进行通知的配置-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!-- 开启注解扫描 -->
    <context:component-scan base-package="com.atguigu.spring5.aopanno"></context:component-scan>

    <!-- 开启Aspect生成代理对象-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
    <!-- 如果代理对象是接口实现类则需要加上这个配置-->
    <aop:config proxy-target-class="true"></aop:config>
</beans>

````



```
//增强的类
@Component
@Aspect  //生成代理对象
public class UserProxy {}

//被增强的类
@Component
public class User {}

```

```
//4、配置不同类型的通知
@Component
@Aspect  //生成代理对象
public class UserProxy {
      //相同切入点抽取
    @Pointcut(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void pointdemo() {

    }

    //前置通知
    //@Before注解表示作为前置通知
    @Before(value = "pointdemo()")//相同切入点抽取使用！
    public void before() {
        System.out.println("before.........");
    }

    //后置通知（返回通知）
    @AfterReturning(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void afterReturning() {
        System.out.println("afterReturning.........");
    }

    //最终通知
    @After(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void after() {
        System.out.println("after.........");
    }

    //异常通知
    @AfterThrowing(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void afterThrowing() {
        System.out.println("afterThrowing.........");
    }

    //环绕通知
    @Around(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕之前.........");

        //被增强的方法执行
        proceedingJoinPoint.proceed();

        System.out.println("环绕之后.........");
    }
}


```

#### 7 、**有多个增强类对同一个方法进行增强，设置增强类优先级**

```
//（1）在增强类上面添加注解 @Order(数字类型值)，数字类型值越小优先级越高
@Component
@Aspect
@Order(1)
public class PersonProxy{ }


```

#### 8、Aop 注解方式

```xml
  <aop:config>
<!--        配置切入点-->
        <aop:pointcut id="p" expression="execution(* com.it.spring.Aop1.UserImpl.add(..))"/>

<!--        切面-->
        <aop:aspect ref="userProxy">
            <aop:before method="before" pointcut-ref="p"></aop:before>
        </aop:aspect>
    </aop:config>
```

### JDBCTemplate

#### 1、创建dataSourece

~~~
    <bean id="dateSource" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close">
        <property name="url" value="jdbc:mysql:///yygh_authority"></property>
        <property name="username" value="root"></property>
        <property name="password" value="Acszdxt!096" ></property>
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    </bean>

~~~

#### 2、创建JdbcTemplate对象

~~~
    <bean id="JdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
<!--        注入dataSource-->
        <property name="dataSource" ref="dateSource"></property>
    </bean>

~~~

#### 3、操作数据库

~~~
    public void add(User user) {
        int update = jdbcTemplate.update("insert into user values (?,?)", user.getUser(), user.getName());

        System.out.println(1);

    }
    
    public void update(User user) {
        int update = jdbcTemplate.update("update user set name=? where user=?", user.getName(), user.getUser());
        System.out.println(update);
    }

    @Override
    public void delete(String user) {
        int update = jdbcTemplate.update("delete from user where user=?", user);
        System.out.println(update);
    }
~~~

### 事务

#### 事务传播级别

##### REQUIRED

PROPAGATION_REQUIRED：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的默认设置

~~~
@Transactional(propagation= Propagation.REQUIRED)
public void methodA(){
	methodB();
    // do something
}

@Transactional(propagation= Propagation.REQUIRED)
public void methodB(){
    // do something
}

~~~

调用methdoA，如果methodB发生异常，触发事务回滚，也会methodA中的也会回滚。

##### REQUIRES_NEW

PROPAGATION_REQUIRES_NEW：创建新事务，无论当前存不存在事务，都创建新事务。

~~~
@Transactional(propagation= Propagation.REQUIRED)
public void methodA(){
    // do something pre
	methodB();
    // do something post
}

@Transactional(propagation= Propagation.REQUIRES_NEW)
public void methodB(){
    // do something
}

~~~

调用methodA，会先开启事务1，执行A的something pre的代码。再调用methodB，methdoB会开启一个事务2，再执行自身的代码。最后在执行methodA的something post。如果method发生异常回滚，只是methodB中的代码回滚，不影响methodA中的代码。如果methodA发生异常回滚，只回滚methodA中的代码，不影响methodB中的代码。

简言之，不会影响别人，也不会被别人影响。

##### SUPPORTS

PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。

~~~
@Transactional(propagation= Propagation.REQUIRED)
public void methodA(){
	methodB();
    // do something
}

@Transactional(propagation= Propagation.SUPPORTS)
public void methodB(){
    // do something
}

~~~

如果调用methodA，再调用methodB，MehtodB会加入到MethodA的开启的当前事务中。

如果直接调用methodB，当前没有事务，就以非事务执行。

##### NOT_SUPPORTED

~~~
@Transactional(propagation= Propagation.REQUIRED)
public void methodA(){
	methodB();
    // do something
}

@Transactional(propagation= Propagation.NOT_SUPPORTED)
public void methodB(){
    // do something
}

~~~

调用methodA，再调用methodB，methodA开启的事务会被挂起，即在methodB中不齐作用，相当于没有事务，methodB内部抛出异常不会回滚。methodA内的代码发生异常会回滚。

直接调用methodB，不会开启事务。

##### NEVER

PROPAGATION_NEVER：以非事务方式执行操作，如果当前存在事务，则抛出异常。

~~~
@Transactional(propagation= Propagation.REQUIRED)
public void methodA(){
	methodB();
    // do something
}

@Transactional(propagation= Propagation.NEVER)
public void methodB(){
    // do something
}

~~~

##### MANDATORY

PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。

~~~
@Transactional(propagation= Propagation.REQUIRED)
public void methodA(){
	methodB();
    // do something
}

@Transactional(propagation= Propagation.MANDATORY)
public void methodB(){
    // do something
}

~~~

如果调用methodA，再调用methodB，MehtodB会加入到MethodA的开启的当前事务中。

如果直接调用methodB，当前没有事务，就会抛出异常。

##### NESTED

PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。

~~~
@Transactional(propagation= Propagation.REQUIRED)
public void methodA(){
    // do something pre
	methodB();
    // do something post
}

@Transactional(propagation= Propagation.NESTED)
public void methodB(){
    // do something
}

~~~

调用methodA，开启一个事务，执行something pre的代码，设置回滚点savepoint,再调用methodB的代码，如果methodB里抛出异常，此时回滚到之前的saveponint。再然后执行methodA里的something post的代码，最后提交或者回滚事务。
嵌套事务，外层的事务如果回滚，会导致内层的事务也回滚；但是内层的事务如果回滚，仅仅是回滚自己的代码，不影响外层的事务的代码。

#### 事务的四大特性（ACID）

##### 原子性

操作要么全部成功，要么全部失败回滚。

##### 一致性

事务执行前和执行后处于一致性状态。例如，转账前A、B共5000元，A、B之间转账后，两者之和仍应该是5000元。

##### 隔离性

事务之间互不干扰。

##### 持久性

事务一旦提交，数据的改变是永久性的，即使这时候数据库发生故障，数据也不会丢失。

#### 与事务隔离级别的相关问题

##### 脏读

​		A事务对一条记录进行修改，尚未提交，B事务已经看到了A的修改结果。若A发生回滚，B读到的数据就是错误的，这就是脏读。

##### 不可重复读

​		A事务对一条记录进行修改，尚未提交，B事务第一次查询该记录，看到的是修改之后的结果，此时A发生回滚，B事务又一次查询该记录，看到的是回滚后的结果。同一个事务内，B两次查询结果不一致，这就是不可重复读。

##### 幻读

​		A事务对所有记录进行修改，尚未提交，此时B事务创建了一条新记录，A、B都提交。A查看所有数据，发现有一条数据没有被修改，因为这是B事务新增的，就想看到了幻象一样，这就是幻读。

#### 事务的隔离级别

##### 读未提交（read uncommitted）

事务尚未提交，其他事务即可以看到该事务的修改结果。隔离级别最差，脏读、不可重复读、幻读都不能避免。

##### 读提交（read committed）

事务只能看到其他事务提交之后的数据。可避免脏读，不可重复读、幻读无法避免。

不可重复读原因：A事务修改，B事务查询，A提交前和提交后，B事务看到的数据是不一致的。

幻读原因：A事务修改，B事务新增，B事务提交前，A事务已经提交。B事务提交后，A发现仍有数据未修改。

##### 可重复读（repeatable read）-------innodb默认隔离级别

​		一个事务多次查询，无论其他事务对数据如何修改，看到的数据都是一致的。因为A事务查询数据时，若B同时在修改数据，A事务看到的永远是B事务执行前的数据。只有当A提交或者回滚之后，看到的才是最新的被B修改知乎的数据。可避免脏读、不可重复读，幻读无法避免。

##### 序列化（serializable）

事务顺序执行，可避免脏读、不可重复读、幻读，但效率最差。因为A事务执行时，其他事务必须等待。
