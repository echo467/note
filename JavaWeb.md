JavaWeb

### 基本工具

+ Tomcat，一种免费开源的web应用服务器，运行web应用
+ Maven，一种项目管理工具，可以用来管理配置jar包

### servlet

#### 简介

+ sun公司用来开发动态web的一门技术

+ java中有一个接口叫servlet，要开发一个servlet程序需要两步
  + 编写一个类，**实现servlet接口**
  + 把开发好的Java类部署到web服务器上
  
+ 我们把**实现了servlet接口的Java程序叫做servlet**

+ 构建

  + 编写一个类实现servlet接口（可以直接继承HttpServlet接口）

  + 编写Servlet的映射（在web.xml文件中，先添加web框架）

    ```xml
     <!--注册Servlet-->
        <servlet>
            <servlet-name>hello</servlet-name>
             <!--自己编写servlet类-->
            <servlet-class>com.kuang.servlet.HelloServlet</servlet-class>
        </servlet>
        <!--Servlet的请求路径-->
        <servlet-mapping>
            <servlet-name>hello</servlet-name>
            <url-pattern>/hello</url-pattern>
        </servlet-mapping>
    ```

#### 原理

+ servlet是由Web服务器（tomcat）调用，服务器会生成一个request对象存放请求的参数，和一个response对象；并交给底层的servlet的service方法处理，并将响应结果放到response中返回给客户端
+ ![image-20201108150132864](https://gitee.com/ly10208/images/raw/master/img/20201210204904.png)

#### ServletContex

+ web容器启动时，会为每个web程序创建一个ServletContex对象，代表当前的web应用

##### 共享数据

+ 在一个Servlet中保存的数据，可以在另外一个servlet中拿到

+ ```java
  ServletContext context = this.getServletContext();//获取当前web的servlet对象
  context.setAttribute("username","zhangsan"); //存放数据
  
  context.getAttribute("username");//其他地方取出数据
  ```

##### 获取初始化参数

+ ```xml
  <!--配置一些web应用初始化参数-->
      <context-param>
          <param-name>url</param-name>
          <param-value>jdbc:mysql://localhost:3306/mybatis</param-value>
      </context-param>
  ```

+ ```java
   ServletContext context = this.getServletContext();
   String url = context.getInitParameter("url");
  ```

##### 请求转发

+ ```java
   ServletContext context = this.getServletContext();
  //RequestDispatcher requestDispatcher = context.getRequestDispatcher("/gp"); //转发的请求路径
  //requestDispatcher.forward(req,resp); //调用forward实现请求转发；
   context.getRequestDispatcher("/gp").forward(req,resp);//转跳到gp
  ```

##### 读取资源文件

+ Properties

  - 在java目录下新建properties
  - 在resources目录下新建properties

  发现：都被打包到了同一个路径下：classes，我们俗称这个路径为classpath:

+ ```java
  InputStream is = this.getServletContext().getResourceAsStream("/WEB-INF/classes/com/kuang/servlet/aa.properties");
  
  Properties prop = new Properties();
  prop.load(is);
  String user = prop.getProperty("username");
  String pwd = prop.getProperty("password");
  ```

#### HttpServletRequest

##### 获取参数，请求转发

 ```java
//解决输入乱码
req.setCharacterEncoding("utf-8"); 
//获取参数
 String username = req.getParameter("username");
 String password = req.getParameter("password");
 String[] hobbys = req.getParameterValues("hobbys");
//请求转发，转跳到/success.jsp页面
 req.getRequestDispatcher("/success.jsp").forward(req,resp);
 ```

#### HttpServletResponse

+ 处理返回给client的东西，向浏览器输出信息

##### 状态码

##### 文件下载

+ 流程
  + 要获取下载文件的路径
  + 获取要下载的文件名
  + 设置想办法让浏览器能够支持下载我们需要的东西
  + 获取下载文件的输入流
  + 创建缓冲区
  + 获取OutputStream对象
  + 将FileOutputStream流写入到buffer缓冲区
  + 使用OutputStream将缓冲区中的数据输出到客户端
+ 代码

```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    // 1. 要获取下载文件的路径
    String realPath = "F:\\班级管理\\西开【19525】\\2、代码\\JavaWeb\\javaweb-02-servlet\\response\\target\\classes\\秦疆.png";
    System.out.println("下载文件的路径："+realPath);
    // 2. 下载的文件名是啥？
    String fileName = realPath.substring(realPath.lastIndexOf("\\") + 1);//转义
    // 3. 设置想办法让浏览器能够支持(Content-Disposition)下载我们需要的东西,中文文件名URLEncoder.encode编码，否则有可能乱码
    resp.setHeader("Content-Disposition","attachment;filename="+URLEncoder.encode(fileName,"UTF-8"));
    // 4. 获取下载文件的输入流
    FileInputStream in = new FileInputStream(realPath);
    // 5. 创建缓冲区
    int len = 0;
    byte[] buffer = new byte[1024];
    // 6. 获取OutputStream对象
    ServletOutputStream out = resp.getOutputStream();
    // 7. 将FileOutputStream流写入到buffer缓冲区,使用OutputStream将缓冲区中的数据输出到客户端！
    while ((len=in.read(buffer))>0){
        out.write(buffer,0,len);
    }
    in.close();
    out.close();
}
```

##### 重定向

```java
resp.sendRedirect("/r/img");   //重定向
```

### jsp

#### 简介

+ java server pages，java服务端页面，和servlet一样是动态web技术
+ jsp是html语言中镶嵌java代码，为用户提供动态数据

#### 原理

+ jsp代码最终会被转换成一个Java类，**本质上就是一个servlet**
+ 在转换成java类的过程中，Java代码不变，html代码转换成 `out.write("<html>\r\n");`输出到前端

#### 9大内置对象

##### out对象

+ 向客户端、浏览器输出数据，并且管理缓冲区

##### request对象

+ 封装从客户端到服务器发出的请求信息
+ `request.setAttribute`保存数据时，只在**一次请求**中保存数据，请求转发携带这个数据

##### response对象

+ 封装了服务器的响应信息，可以设置http头和cookie信息

##### PageContex对象

+ 提供了对jsp页面所有对象的访问
+ `pageContext.setAttribute` 保存的数据只在**一个页面**中有效

##### session对象

+ 从一个客户打开浏览器并连接到服务器开始，到客户关闭浏览器离 开这个服务器结束(或者超时)，被称为一个会话
+ 用来保存会话信息。也就是说，可以实现在同一用户的不同请求之间共享数。
+ `session.setAttribute` 保存的数据在**一次会话**中有效

##### application对象

+ 代表当前的应用程序，存在于服务器的内存空间中。应用一旦启动便会自动生成一个application对象。如果应用没有被关闭，此application对象便一直会存在。直到应用被关闭，application的生命周期比session更长。
+ 为**多个用户共享全局信息**。比如当前的在线人数等。
+ `application.setAttribute` 保存的数据**全部用户**有效

##### config对象

+ 表示当前jsp程序的配置信息，获取服务器的配置信息

##### exception对象

+ jsp引擎在执行代码时抛出的异常

##### page对象

+ 类似与java编程中的this指针，他指向了当前jsp页面本身

### JavaBean

+ 实体类，一般用来和数据库的字段做映射  ORM；
+ JavaBean有特定的写法
  + 必须要有一个无参构造
  + 属性必须私有化
  + 必须有对应的get/set方法；

### Filter

+ 过滤器，用来过滤网站的数据

#### 开发

##### 实现Filter接口

```java
public class CharacterEncodingFilter implements Filter {
    //初始化：web服务器启动，就以及初始化了，随时等待过滤对象出现！
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("CharacterEncodingFilter初始化");
    }
    //Chain : 链
    /*
    1. 过滤中的所有代码，在过滤特定请求的时候都会执行
    2. 必须要让过滤器继续同行
        chain.doFilter(request,response);
     */
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        request.setCharacterEncoding("utf-8");
        response.setCharacterEncoding("utf-8");
        response.setContentType("text/html;charset=UTF-8");

        System.out.println("CharacterEncodingFilter执行前....");
        chain.doFilter(request,response); //让我们的请求继续走，如果不写，程序到这里就被拦截停止！
        System.out.println("CharacterEncodingFilter执行后....");
    }
    //销毁：web服务器关闭的时候，过滤会销毁
    public void destroy() {
        System.out.println("CharacterEncodingFilter销毁");
    }
}
```

##### 在web.xml中配置filter

```xml
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>com.kuang.filter.CharacterEncodingFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <!--只要是 /servlet的任何请求，会经过这个过滤器-->
    <url-pattern>/servlet/*</url-pattern>
    <!--<url-pattern>/*</url-pattern>-->
</filter-mapping>
```





