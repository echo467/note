



Mybatis官方文档：https://mybatis.org/mybatis-3/zh/index.html

### 1. 简介

+ 优秀的**持久层**框架
+ 支持定制SQL、存储过程和高级映射
+ 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集
+ 可以使用简单的 **XML 或注解**来配置和映射原生类型、接口和 Java 的 POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录

### 2. mybatis配置

#### 2.1 maven导jar包

jar包的导入可以在父目录下的pom.xml中导入

```xml
<!-- 引入mybatis的jar包 -->
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.6</version>
</dependency>
<!-- 配置mysql连接驱动 -->
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.22</version>
</dependency>
```

#### 2.2  mybatis核心配置文件

resources路径下的mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">   
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?serverTimezone=GMT%2B8"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="com/ly/mapper/userMapper.xml"/>
    </mappers>
</configuration>
```

#### 2.3 mybatis工具类

```java
public class MybatisUtil {
    private static SqlSessionFactory sqlSessionFactory;
    static {
        try {
            //mybatis核心配置文件
            String resource = "mybatis-config.xml"; 
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public static SqlSession getSqlSession (){
        return sqlSessionFactory.openSession();
    }
}
```

#### 2.4 实体类entity

与数据库的字段对应，是一个JavaBean

```java
@Data    //引入Lombok后，一个注释即可省略get、set方法和有参无参构造方法
public class User {
    private int id;
    private String name;
    private String password;
}
```

#### 2.5 接口及其映射

+ userMapper接口，用来供上层调用操作数据库

```java
public interface userMapper {
    User seletAll(int id);
}
```

+ mapper映射，实际上相当于userMapper接口的实现类

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace配置userMapper接口的路径
    resultType要写完整路径,除非使用别名 -->
<mapper namespace="com.ly.mapper.userMapper">
    <select id="seletAll" resultType="com.ly.entity.User">
       select * from mybatis.user where id = #{id}
    </select>
</mapper>
```

#### 2.6 注意事项

##### 项目结构

![image-20201115105203298](images.assets\image-20201115105203298.png)

+ userMapper.xml文件即可以放到java路径下，又可以放到resources路径下

+ 当放到java路径下时，maven不会自动打包java路径下的.xml文件，需要在**子项目下的pom.xm**l文件中配置资源路径，**resources与java两个路径都得配置**

  ```xml
  <build>
          <resources>
              <resource>
                  <directory>src/main/resources</directory>
                  <includes>
                      <!-- 原本resources路径也要配置上-->
                      <include>*.xml</include>
                  </includes>
              </resource>
              <resource>
                  <directory>src/main/java</directory>
                  <includes>
  <!--                    <include>*.xml</include>-->
                      <include>**/*.xml</include>
                  </includes>
              </resource>
          </resources>
      </build>
  ```

+ 当放到resources路径下时，最好与userMapper接口的路径一样

  ```
  src/main/java/com/ly/mapper/userMapper.java
  src/main/resourrces/com/ly/mapper/userMapper.xml
  ```

##### 注解开发

+ 上面的mapper映射文件可以通过注解来代替

  ```java
  @Select("select * from user where id = #{id}")
  User seletAll(@Param("id")int id);
  ```

+ 但是注册mapper时，只能通过package或class方式来注册，不能使用resources注册

  ```xml
  <mappers>
      <!--        <mapper resource="com/ly/mapper/userMapper.xml"/>-->
      <!--        <mapper class="com.ly.mapper.userMapper" />-->
      <package name="com.ly.mapper"/>
  </mappers>
  ```

### 3. 原理

#### 3.1 原理图

![image-20201115135240916](images.assets\image-20201115135240916.png)

#### 3.2 解析

+ SqlSessionFactoryBuild通过给定的mybatis-config.xml文件，构建出SqlSessionFactory；
+ SqlSessionFactory通过openSession、closeSession方法，构建销毁SqlSession；
+ SqlSession通过getMapper方法，获取到Mapper类
+ 调用Mapper类的接口操作数据库。

#### 3.3 作用域

+ SqlSessionFactoryBuilder，用来创建SqlSessionFactory，创建完成之后就不再需要。最佳作用域使方法作用域（也就是局部方法变量）

+ SqlSessionFactory，相当于一个数据库连接池，一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例，作用域全局。可使用单例模式或静态单例模式。

+ SqlSession，连接到连接池的一个请求，非线程安全不可共享，使用完成需要关闭。作用域是请求或方法作用域

  ![image-20201115140914868](images.assets\image-20201115140914868.png)

### 4.  配置解析

#### 4.1  核心配置文件

+ mybatis-config.xml，是mybatis的配置文件

  ``` xml
  configuration（配置）
  properties（属性）
  settings（设置）
  typeAliases（类型别名）
  typeHandlers（类型处理器）
  objectFactory（对象工厂）
  plugins（插件）
  environments（环境配置）
  environment（环境变量）
  transactionManager（事务管理器）
  dataSource（数据源）
  databaseIdProvider（数据库厂商标识）
  mappers（映射器）
  ```

+ 实例

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE configuration
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>
      <environments default="development">   
          <environment id="development">
              <transactionManager type="JDBC"/>
              <dataSource type="POOLED">
                  <property name="driver" value="com.mysql.jdbc.Driver"/>
                  <property name="url" value="jdbc:mysql://localhost:3306/mybatis?serverTimezone=GMT%2B8"/>
                  <property name="username" value="root"/>
                  <property name="password" value="123456"/>
              </dataSource>
          </environment>
      </environments>
      <mappers>
          <mapper resource="com/ly/mapper/userMapper.xml"/>
      </mappers>
  </configuration>
  ```

#### 4.2 环境配置（environments）

+ environments 下面可以配置多个 environment，也就是说可以配置多套环境（开发、运维、测试之类）
+ 通过 default 字段选择当前使用哪一套环境
+ 每一套环境中，需要选择事务管理器、连接池、和数据库驱动、地址、用户名、密码

#### 4.3 属性（properties）

+ 我们可以通过properties属性来实现引用配置文件

+ 这些属性都是**可外部配置且可动态替换的**，既可以在典型的 Java 属性文件(db.properties)中配置，亦可通过 properties 元素的子元素来传递。

+ 编写配置文件 db.properties

  ```properties
  driver=com.mysql.jdbc.Driver
  url=jdbc:mysql://localhost:3306/mybatis?serverTimezone=GMT%2B8
  username=root
  password=123456
  ```

+ 核心配置文件中映入

  ```xml
  <properties resource="db.properties">
      <property name="username" value="root"/>
      <property name="password" value="111111"/>
  </properties>
  <environments default="development">
      <environment id="development">
          <transactionManager type="JDBC"/>
          <dataSource type="POOLED">
              <property name="driver" value="${driver}"/>
              <property name="url" value="${url}"/>
              <property name="username" value="${username}"/>
              <property name="password" value="${password}"/>
          </dataSource>
      </environment>
  </environments>
  
  ```

+ 总结

  + 可以直接引入外部文件
  + 可以在其中增加一些属性配置
  + 如果两个文件有同一个字段，优先使用**外部配置文件**的！

#### 4.4 类型别名（typeAliases)

+ 我们注意到在mapper映射文件中 resultType 需要设置详细的包名，可以在核心配置中添加别名代替

+ 给实体类别名

  ``` xml
   <!--可以给实体类起别名-->
  <typeAliases>
      <typeAlias type="com.ly.entity.User" alias="User" />
  </typeAliases>
  ```

+ 给一整个包下的实体类别名

  ```xml
  <typeAliases>
      <package name="com.ly.entity"/>
  </typeAliases>
  ```

  MyBatis 扫描实体类的包，它的默认别名就为这个类的 **类名**，首字母小写！

+ 注解加别名

  ```java
  @Alias("user")
  public class User {}
  ```

#### 4.5 设置（setting）

+ 这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。 

+ 比如说开启日志、开启缓存等

  ```xml
  <!-- 日志实现类型 log4j、stdout_logging(控制台).... -->
  <settings>
      <setting name="logImpl" value="STDOUT_LOGGING"/>
      <!--        <setting name="logImpl" value="LOG4J"/>-->
  </settings>
  ```

#### 4.6 映射（mapper）

+ MapperRegistry：注册绑定我们的Mapper.xml文件；

+ 方式一：推荐使用

  ```xml
  <!--使用resource路径注册mapper.xml文件-->
  <mappers>
      <mapper resource="com/ly/mapper/userMapper.xml"/>
  </mappers>
  ```

+ 方式二：使用class文件绑定

  ```xml
  <mappers>
      <mapper class="com.ly.mapper.userMapper" />
  </mappers>
  ```

  注意：

  - 接口和他的Mapper配置文件必须同名！
  - 接口和他的Mapper配置文件**编译之后**必须在同一个包下！

+ 方式三：使用扫描包进行注入绑定

  ```xml
  <mappers>
      <package name="com.ly.mapper"/>
  </mappers>
  ```

  注意：

  - 接口和他的Mapper配置文件必须同名！
  - 接口和他的Mapper配置文件**编译之后**必须在同一个包下！

### 5. 映射器

#### 5.1 xml映射文件

+ mapper.xml，等效于mapper接口的实现类

+ 标签

  ```xml
  select   映射查询语句
  insert   映射插入语句
  update   映射更新语句
  delete   映射删除语句
  sql      定义可被其它语句引用的可重用语句块（存储过程）
  resultMap 描述如何从数据库结果集中加载对象，是最复杂也是最强大的元素
  cache      该命名空间（一个mapper文件）的缓存配置
  cache-ref  引用其他命名空间的缓存配置
  ```

#### 5.2 select

+ 用来查询

  ```xml
  <select id="seletAll" resultType="com.ly.entity.User" >
      select id,name,password as pwd from mybatis.user where id = #{id}
  </select>
  ```

+ 属性
  + id  命名空间中的唯一标识符，用来引用这条语句
  + parameterType  传入这条语句的参数的类型，一般不需要设置，mybatis会自动判断
  + resultType 希望得到的返回结果的类型或别名，如果返回的是集合，那应该设置为**集合包含的类型**，而不是集合本身的类型
  + resultMap **自定义返回结果类型**的引用，不能和resultType同时使用

#### 5.3 insert、update、delete

+ 用来插入、修改、删除记录
+ 一般只用带一个id属性就可以

#### 5.4  resultMap

+ 当数据库的字段名和java实体类的属性名不一致时，需要使用结果集映射，否则该属性将为空

```
结果集映射
id  name  password
id  name  pwd
```

+ 需要预先建立一个结果集映射，然后再使用该映射集

```xml
<!--定义一个结果集映射-->
<resultMap id="UserMap" type="User">
    <!--column数据库中的字段，property实体类中的属性-->
    <result column="id" property="id"/>
    <result column="name" property="name"/>
    <result column="password" property="pwd"/>
</resultMap>

<!--使用该结果集映射-->
<select id="getUserById" resultMap="UserMap">
     select * from mybatis.user where id = #{id}
</select>
```

上面UserMap甚至只需要定义字段与属性对应不上的映射就行

+ 映射时，如果有复杂的属性（对象、集合），则需要单独处理
+ 常用标签
  + id ，映射关系组的主键（可以不设置）
  + result，定义一组映射关系
  + association，关联一个对象
  + collection，集合多个对象

#### 5.5 多对一

+ 使用association
+ 相关属性
  + property  ： resultMap映射对象的属性
  + colum      :    数据库中的字段
  + javaType :    property的类名
  + select      :    加载复杂类型属性的映射语句的 ID，并将cloum中的数据作为参数传递给 select 语句

##### 实体类

```java
@Data
public class Student {
    private int sid;
    private String sname;
    //每个学生都有一个老师
    private Teacher teacher;
}
```

```java
@Data
public class Teacher {
    private int tid;
    private String tname;
}
```

##### 按结果嵌套

联表查询

```xml
<!--按照结果嵌套处理-->
<select id="getStudent2" resultMap="StudentTeacher2">
    select s.id sid,s.name sname,t.name tname
    from student s,teacher t
    where s.tid = t.id;
</select>

<resultMap id="StudentTeacher2" type="Student">
    <result property="sid" column="sid"/>
    <result property="sname" column="sname"/>
    <association property="teacher" javaType="Teacher">
        <result property="tname" column="tname"/>
    </association>
</resultMap>
```

##### 按照查询嵌套

子查询

```xml
<!--
    思路:
        1. 查询所有的学生信息
        2. 根据查询出来的学生的tid，寻找对应的老师！  子查询
    -->
<select id="getStudent" resultMap="StudentTeacher">
    select * from student
</select>

<resultMap id="StudentTeacher" type="Student">
    <result property="sid" column="id"/>
    <result property="sname" column="name"/>
    <!--复杂的属性，我们需要单独处理 对象： association 集合： collection -->
    <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"/>
</resultMap>

<select id="getTeacher" resultType="Teacher">
    select * from teacher where id = #{id}
</select>

```

#### 5.6 一对多

+ 使用collection
+ 相关属性
  + property  ： resultMap映射对象的属性
  + ofType     :    集合中对象的类型
  + colum      :    数据库中的字段
  + javaType :    property的类名
  + select      :    加载复杂类型属性的映射语句的 ID，并将cloum中的数据作为参数传递给 select 语句

##### 实体类

```java
@Data
public class Student {
    private int id;
    private String name;
    private int tid;
}
```

```java
@Data
public class Teacher {
    private int id;
    private String name;
    //一个老师拥有多个学生
    private List<Student> students;
}
```

##### 按照结果嵌套

```xml
<!--按结果嵌套查询-->
<select id="getTeacher" resultMap="TeacherStudent">
    select s.id sid, s.name sname, t.name tname,t.id tid
    from student s,teacher t
    where s.tid = t.id and t.id = #{tid}
</select>

<resultMap id="TeacherStudent" type="Teacher">
    <result property="id" column="tid"/>
    <result property="name" column="tname"/>
    <!--复杂的属性，我们需要单独处理 对象： association 集合： collection
        javaType="" 指定属性的类型！
        集合中的泛型信息，我们使用ofType获取
        -->
    <collection property="students" ofType="Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <result property="tid" column="tid"/>
    </collection>
</resultMap>
```

##### 按照查询嵌套

```xml
<select id="getTeacher2" resultMap="TeacherStudent2">
    select * from mybatis.teacher where id = #{tid}
</select>

<resultMap id="TeacherStudent2" type="Teacher">
    <collection property="students" javaType="ArrayList" ofType="Student" select="getStudentByTeacherId" column="id"/>
</resultMap>

<select id="getStudentByTeacherId" resultType="Student">
    select * from mybatis.student where tid = #{tid}
</select>
```

### 6. 动态SQL

#### 6.1 概念

+ 动态SQL就是**根据不同的条件生成不同的SQL语句**，实现SQL**语句拼接**

+ 本质还是SQL语句 ， 只是我们可以在SQL层面，去执行一个逻辑代码

+ 常用标签

  ```xml
  <where> </where>   充当sql语句的where条件
  
  <if  test=" "> </if>   
  
  <choose>
    <when></when>
    <otherwise></otherwise>
  </choose> 
  
  <trim>         
    <where></where>
    <set></set>
  </trim>
  
  <foreach> </foreach>
  ```

#### 6.2  if标签

```xml
<select id="findUser" resultType="user">
  SELECT * FROM user where
  <if test="id != null">
      id = #{id}
  </if>
  <if test="name != null">
      AND name like #{name}
  </if>
</select> 
```

+ <if test="表达式">  语句   </if>
+ 如果test中的表达式为真，则带上后面的语句否则不带
+ 但是上面的sql语句，当id为空，拿么不为空时，sql语句会变成` SELECT * FROM user where AND name like #{name} ` 。会报错

#### 6.3 where标签

 ```xml
<select id="findUser" resultType="user">
  SELECT * FROM user
  <where>
    <if test="id != null">
         id = #{id}
    </if>
    <if test="name != null">
        AND name like #{name}
    </if>
  </where>
</select>  
 ```

+ *where* 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。
+ 而且，若子句的开头为 “AND” 或 “OR”，*where* 元素也会将它们去除。

#### 6.4 choose标签

```xml
<select id="findUser" resultType="user">
  SELECT * FROM user
  <where>
    <choose>
        <when test="id != null">
           id = #{id}
        </when>
        <when test="name != null">
           name like #{name}
        </when>
        <otherwise>
           password = #{pwd}
        </otherwise>
    </choose>
  </where>
</select> 
```

+ 有时候，我们不想使用所有的条件，而只是想从多个条件中选择一个使用。
+ 针对这种情况，MyBatis 提供了 choose 元素，它有点像 Java 中的 switch 语句。
+ 当when后面的test满足时，即结束匹配；若都不满足，则执行otherwise

#### 6.5 foreach标签

```xml
<select id="findUser" resultType="user">
    SELECT * FROM user where id in
    <foreach item="item" index="index" collection="list"
             open="(" separator="," close=")"
             #{item}
    </foreach>
</select> 
```

+ 对集合进行遍历（尤其是在构建 IN 条件语句的时候）
+ 可以指定一个集合（list），声明可以在元素体内使用的集合项（item）和索引（index）变量。
+ 它也允许你指定开头与结尾的字符串以及集合项迭代之间的分隔符

### 7. 缓存

#### 7.1 mybatis缓存

+ MyBatis包含一个非常强大的查询缓存功能，可以将用户经常查询的数据放到内存中，下次查取直接走内存就行
+ 可以减小和数据库的交互次数，提高查询效率，解决高并发系统性能问题
+ 经常查询并且不经常改变的数据，可以使用缓存。
+ mybatis有两级缓存，默认开启一级缓存，关闭二级缓存

#### 7.2 一级缓存

+ 一级缓存的是 SqlSession，且只在一次SqlSession中有效（也就是在拿到连接到关闭连接的阶段）
+ 缓存失效
  + 查询不同的东西
  + 增删改操作
  + 手动清理

#### 7.3 二级缓存

+ 二级缓存是全局缓存，基于namespace，也就是一个mapper.xml文件一个缓存。
+ 工作机制
  + 一个会话查询一条数据，这个数据就会被放在当前会话的一级缓存中；
  + 如果当前会话关闭了，这个会话对应的一级缓存就没了；但是一级缓存中的数据被保存到二级缓存中；

+ 开启步骤

  + 开启全局缓存

    ```xml
    <!--显示的开启全局缓存-->
    <setting name="cacheEnabled" value="true"/>
    ```

  + 在要使用二级缓存的Mapper中开启

    ```xml
    <!--在当前Mapper.xml中使用二级缓存-->
    <cache/>
    ```

    也可以定义参数

    ```xml
    <!--在当前Mapper.xml中使用二级缓存-->
    <!--先进先出策略、刷新时间60s、大小512条、只读-->
    <cache  eviction="FIFO"          
           flushInterval="60000"
           size="512"
           readOnly="true"/>
    ```

+ 小结

  + 所有的数据都会先放在一级缓存中；
  + 只有当会话提交，或者关闭的时候，才会提交到二级缓冲中！

#### 7.4 缓存原理

![image-20201116214035087](images.assets\image-20201116214035087.png)

