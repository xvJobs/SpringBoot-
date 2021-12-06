# SpringBoot

## 1、为什么要使用Spring Boot

-   因为Spring、Spring MVC需要使用大量的配置文件
-   还需要配置各种对象，把使用的对象放入Spring容器中才能使用
-   需要了解其他框架配置规则。



-   SpringBoot等价于不需要配置文件的Spring+SpringMVC。常用的框架和第三方库都已经配置好了
-   Spring开发效率高，使用方便



### 1.1、JavaConfig

使用Java类作为xml配置文件的代替，是配置Spring容器的纯Java的方式。在这个Java类这可以创建对象，把对象放在Spring容器中（注入到容器）

使用连个注解：

1.  @Configuration：放在一个类上面，表示这个类是作为配置文件使用的。
2.  @Bean：声明对象，把对象注入到容器中。

```java
@Configuration //表示当前类是作为配置文件使用，用来配置容器。
public class SpringConfiguration{  //这个类就相当于beans.xml
	@Bean
    public Student createStudent() { //方法的返回值是对象，方法的返回值注入到容器中
        Student s1= new Student();
        s1.setName("陆时忤"); 
        s1.setAge(20);
        s1.setSex("男")；
    }
}
```

 

### 1.2、ImporResource

@ImportResource作用导入其他的xml配置文件，等价在xml

```xml
<import rescource=""/>
```

放在JavaConfign类的上面

属性：value/locations:指定配置文件的位置

```java
@Configuration          //多个配置文件用，号隔开
@ImportResource(value="classpath:applicationContext.xml")
public class SpringConfiguration{ 
```



### 1.3、PropertyResource

@PropertyResource：读取properties属性文件。使用属性配置文件可以实现外部化配置，

在程序代码之外提供数据。

步骤：

1.  在resource目录下，创建properties文件，使用k=v的格式提供数据
2.  在PropertyResource指定properties文件位置
3.  使用@Value(value=“${key}“)来获取v



创建config.properties文件

```properties
tiger_name=老虎
tiger_age=6
```

在JavaConfig类中指定properties配置文件位置

```java
@Configuration
@ImportResource(value="classpath:applicationContext.xml")
@PropertySource(value="classpath:config.properties") // 指定properties位置
@ComponentScan(basePackage="包名") //组件扫描器扫描包内的@Component
public class SpringConfiguration{ 
}
```

创建数据类Tiger

```java
@Component("tiger")
public class Tiger{
    @Value("${tiger_name}")
    private String name;
    @Value("${tiger_age}")
    private String age;
}
```



## 2、注解的使用

@SpringBootApplication
符合注解：由一下三个注解组成
@SpringBootConfiguration
@EnableAutoConnfiguration
@ComponentScan
    

### 1.@SpringBootConfiguration

```java
@Configutation
public @interface SpringBootConfiguration {
    @AliasFor(
    annotation = Configuration.class
    )
    boolean proxybeanMethods()default true;
}
//说明：使用了@SpringBootConfiguration注解标注类，可以作为配置文件使用，
//可以使用Bean声明对象，注入到容器
```

### 2.@EnableAutoConfiguration

启用自动配置，把Java对象配置好，注入到Spring容器中；把第三方对象/框架创建好

例如把mybatis对象创建号，放入容器中

### 3.@ComponentScan

组件扫描器，找到注解，根据注解的功能创建对象，给属性赋值等等。

默认扫描的包：@ComponentScan所在的类所在的包及子包。



### 4.@ConfigurationProperties

将整个文件映射成一个对象，用于自定义配置项比较多的情况

application.properties配置文件

```properties
web.name=百度 
web.website=www.baidu.com
```

```
@Component
@ConfigurationProperties(prefix="web")
public class webInfo(){
	private Strign name;
	private String website
}
```



## 3、Spring Boot核心配置文件

### 1.配置文件名称：application

扩展名有：prroperties(k=v); yml(k: v)

使用application.properties，application.yml (推荐使用第二个)

例1:application.properties设置端口和上下文

```properties
#设置端口号
server.port=8080
#设置访问应用上下文路径contextPath
server.servlet.context-path=/myboot
```

例2:application.yml

```yaml
server:
  port: 8080
  servlet:
    context-path: /myboot2
```



### 2.多环境配置

开发环境、测试环境、上线的环境

每个环境有不同的配置信息，例如端口，上下文见，数据库url，用户名，密码等等



使用多环境配置文件，可以方便的切换不同配置。

使用方式：创建多个配置文件，名称规则：application-环境名称.properties/yml



创建开发环境配置文件：application-dev.properties/yml

创建测试环境配置文件：application-test.properties/yml

创建上线环境配置文件：application-online.properties/yml



aaplication.yml文件配置调用的环境

```yaml
spring:
  profiles:
    active: dev/test/online
```



### 3.使用容器获取对象

通过SpringApplication.run(Application.class,args);返回容器



### 4.CommandLineRunner接口

如果需要在容器启动后执行一些服务，比如读取配置文件，数据库连接之类的。

SpringBoot给我们提供了两个接口来帮助我们实现郑重需求。

这两个接口分别为CommandLineRunner和ApplicationRunner

他们的执行时机为容器启动完成的时候，自动执行run()方法；

这两个接口中有一个run方法，我们只需要实现这个方法即可。

这两个接口不同之处在于：

Applicatrunner中run方法的参数为ApplicaArgument，

而CommandLineRunner接口中run方法的参数为String数组。



## 4、Web组件

三个内容：拦截器、Servlet、Filter

### 1.拦截器

```java
//添加拦截器对象，注入到容器中
@Configuration
public class config implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //创建拦截器对象
        HandlerInterceptor interceptor = new LoginInterceptor();
        //指定拦截请求uri地址
        String[] path = {"/user/**"};
        //指定不拦截的uri地址
        String[] excludePath = {"/user/login"};
        registry.addInterceptor(interceptor)
                .addPathPatterns(path)
                .excludePathPatterns(excludePath);
    }
}
```



### 2.Servlet

在SpringzBoot框架中使用Servlet对象

使用步骤：

1.  创建Servlet类；创建类继承HeepServlet
2.  注册Servlet，让框架能找到Servlet

例子：

1.  创建自定义的Servlet

    ```java
    //创建servlet类
    public class MyServlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest request, HttpServletResponse response) 		throws ServletException, IOException {
            response.setContentType("text/html;charset=utf-8");
            PrintWriter pw = response.getWriter();
            pw.println("执行的是Servlet方法");
            pw.close();
        }
    }
    ```

2.  注册Servlet

```java
@Configuration
public class WebApplicationConfig {
    //定义方法，注册Servlet对象
    @Bean
    public ServletRegistrationBean servletRegistrationBean(){
        //注册Servlet方法一
        //第一个参数是Servlet对象，第二个参数是uri地址
        ServletRegistrationBean bean = new ServletRegistrationBean(new MyServlet(),
                "/myServlet");

        //注册Servlet方法二
        ServletRegistrationBean beanSecond = new ServletRegistrationBean();
        beanSecond.setServlet(new MyServlet());
        beanSecond.addUrlMappings("/login","/test"); //可以指定多个uri地址

        return beanSecond;
    }

}
```



### 3.过滤器Filter

Filter是Servlet规范中的过滤器，可以处理请求，对其功能球的参数，属性进行调整，常常在过滤器中处理字符编码

在框架中使用过滤器：

1.  创建自定义过滤器
2.  注册过滤器

```java
//自定义过滤器
public class MyFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain 				filterChain) throws IOException, ServletException {
        System.out.println("执行了MyFilter，doFilter");
        filterChain.doFilter(servletRequest, servletResponse);
    }
}
```

```java
//注册Filter
@Configuration
public class WebApplicationConfig {
    
    @Bean
    public FilterRegistrationBean filterRegistrationBean(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new MyFilter());
        bean.addUrlPatterns("/user/*");
        return bean;
    }
}
```



### 4.字符集过滤器

CharacterEncodingFilter:解决post请求中乱码的问题

在SpringMVC框架中在web.xml注册过滤器，配置它的属性。

#### 1.自定义CharacterEncodingFilter

使用步骤：

1.  配置字符集过滤器

    ```
     //注册Filter
        @Bean
        public FilterRegistrationBean filterRegistrationsBean() {
            FilterRegistrationBean reg = new FilterRegistrationBean();
            
            //使用框架中的过滤器
            CharacterEncodingFilter filter = new CharacterEncodingFilter();
            //指定使用的编码方式
            filter.setEncoding("utf-8");
            //指定request、response都使用encoding的值
            filter.setForceEncoding(true);
            reg.setFilter(filter);
            //指定过滤的uri地址
            reg.addUrlPatterns("/*");
            return reg;
        }
    ```

2.  修改application.yml文件，关闭SpringBoot的过滤器，让自定义的过滤器启动

```yaml
#SrpingBoot中默认已经配置了CharacterEncodingFilter。默认编码格式为ISO-8859-1。
#设置enabled=false， 作用是关闭SpringBoot的过滤器，启动自定义的CharacterEncodingFilter
server:
  servlet:
    encoding:
      enabled: false
```



#### 2.修改SpringBoot过滤器的编码格式

修改application.yml文件（推荐使用这种方式）

```yaml
server:
  servlet:
    encoding:
      #让系统的CharacterEncodingFilter启动
      enabled: true
      #指定编码格式
      charset: utf-8
      #强制request、response都使用charset属性的编码方式
      force: true
```



## 5、ORM操作MySQL

使用MyBatis框架操作数据，在SpringBoot框架集成了Mybatis

使用步骤：

1.  mybatis起步依赖：完成mybatis对象自动配置，对象发给你在容器中;mybatis-spring-boot-starter
2.  pom.xml指定把src/main/java目录中的xml文件包含到classpath中
3.  创建实体类Student
4.  创建Dao接口StudentDao，创建一个查询学生的方法
5.  创建Dao接口对应的Mapper文件，xml文件，写sql语句
6.  创建Service层对象，创建StudentService接口和他的实现类。去dao对象的方法。完成数据库的操作
7.  创建Controller对象，访问Service
8.  写application配置文件配置数据库的连接信息

### 1.第一种方式：@Mapper

@Mapper：放在dao接口上面，创建此接口的代理对象

```java
//实体类Student
@Data
public class Student {
    private Integer id;
    private String name;
    private Integer age;

}
```

```java
//接口StudentDao
@Mapper
public interface StudentDao {
    List<Student> selectStudent(Integer id);
}
```

```xml
<!--Dao接口对应的Mapper文件-->
<mapper namespace="com.lushiwu.dao.StudentDao">
    <select id="selectStudent" resultType="com.lushiwu.model.Student">
        select id, name, age from student where id = #{id}
    </select>
</mapper>
```

```java
//Service层实现类
@Service
public class StudentServiceImpl implements StudentService {
    @Resource
    private StudentDao student;
    @Override
    public List<Student> queryStudent(Integer id) {
        List<Student> students = student.selectStudent(id);
        return students;
    }
}
```

```java
//Controller对象
@Controller
public class StudentController {

    @Resource
    private StudentService studentService;

    @RequestMapping("/student/query")
    @ResponseBody
    public List<Student> queryStudent(Integer id){
        List<Student> students = studentService.queryStudent(id);
        return students;
    }
}
```

```yaml
#数据库连接信息
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://139.196.16.182:3306/springdb?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2B8
    username: root
    password: 123
```



### 2.第二种方式：@MapperScan

```java
/**
*@MapperScan：扫描dao接口和Mapper文件
	basePackage：dao接口所在的包名， 是一个字符串数组可以扫描多个包
*/
@SpringBootApplication
@MapperScan("com.lushiwu.dao") //@MapperSacn({"com.lushiwu.dao","com.lushiwu.model.dao"})
public class Application {
}
```





### 3.第三种方式：Mapper文件和Dao接口分开管理

把Mapper文件放在resources目录下

1.  在resources目录中创建子目录，例如mapper

2.  把mapper文件发给你在mapper目录中

3.  在application.yml文件中。指定mapper文件的位置

    ```yaml
    mybatis:
      #指定mapper文件的位置
      mapper-locations: classpath:mapper/*.xml
      #指定mybatis日志  
      configuration:
        log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    
    ```

4.  pom.xml中指定resources目录中的文件，编译到目标目录中



### 4.事物

Spring框架中的事物：

1.  管理实务对象：事物管理器（接口、接口的实现类）

    例如：使用jdbc或mybatis访问数据库，使用事物管理器DataSourceTransactionManager

2.  声明式事物：在xml配置文件或使用注解说明事物控制内容

    控制事物：隔离级别、传播行为、超时时间

3.  事物处理方式：

    1.  Spring框架中的@Transactional
    2.  aspecj框架可以在xml配置文件中，声明事物控制的内容



SpringBoot中使用事物：上面两种方式都可以

1.  在业务方法的上面加入@Transactional,加入注解后，方法有事物功能了。

2.  明确在主启动类的上面，加入@EnableTransactionManager

    ```java
    /*
    * @Transactional:表示方法具有事物功能
    *   默认:使用库的隔离级别，REQUIRED 传播行为：超时时间 -1
    *   抛出运行时异常,回滚事物
    * */
    @Transactional
        @Override
        public int addStudent(Student student) {
            System.out.println("业务方法addStudent执行了");
            int rows = studentDao.insert(student);
            System.out.println("执行sql语句");
            
            return rows;
        }
    ```

    

## 6、接口架构风格——RESTful

### RestFul风格

#### 概念

RestFul就是一个资源定位及资源操作的风格。不是标准也不是协议，只是一种风格。基于这种风格设计的软件可以更简洁，更有层次、更易于实现缓存等机制。

#### 功能

资源：互联网所有的事物都可以被抽象为资源。

资源操作：使用Get、Post、Put、Delete，用不同方法对资源进行操作。

分别对应：查询，添加，修改，删除。

传统方式操作资源：通过不同的参数来实现不同的效果！方法单一，post和get

使用RESTful操作资源：可以通过不同的请求方式来实现不同的效果！如下：请求地址一样，但是功能可以不一样！



### RestFul的注解：

@PathVariable：从url中获取数据

@GetMapping：支持get请求支持

@PostMapping：支持post请求

@PutMapping：支持put请求

@DeleteMapping：支持delete请求

@RestController：符合注解，是@Controller和@ResponseBody组合；在类的上面使用RestController



### 在页面或ajax中，支持put、delete请求

在SpringMVC中有一个过滤器，支持post请求转为put、delete

过滤器：org.springframework.web.filter.HiddenHttpMethodFilter

作用：把请求中的post请求转为put、delete



#### 实现步骤

1.  application.yml：开启HiddenHttpMethodFilter过滤器

2.  在请求页面中，包含__method参数，他的值是put、delete，发起这个请求使用post方式

    ```yaml
    spring: 
      mvc:
        hiddenmethod:
          filter:
            enabled: true #启动支持put、delete请求
    ```

    ```html
    <form action="student/test" method="post">
        <input type="hidden" name="_method" value="put">
        <input type="submin" value="测试请求方式">
    </form>
    ```

    

## 总结

### 注解

Spring+SpringMVC+SpringBoot

#### 创建对象的：

@Controller：放在类的上面，创建控制器对象，注入到容器中

@ResController：发给你在类的上面，创建控制器对象，注入到容器中。

​	作用：复合注解@Controller，@ResposeBody，使用这个注解类，里面的控制器方法的返回值都是数据。

@Service：发给你在业务层的实现类上面，创建service对象，注入到容器

@Respository：放在Dao层的实现类上面，创建dao对象，放入到容器。没有使用这个注解，是因为现在使用Mybatis框架，dao对象是Mybatis通过代理生成的。不需要使用@Respsitory，所以没有使用

@Component：放在类的上面，创建此类的对象，放入到容器中。



#### 赋值的：

@Value：简单类型的赋值，例如在属性的上面使用@Value（“李四”）privata String name

​		还可以使用@Value，获取配置文件中的数据（properties或者yml）。

​		@Value（“${service.port}"）private Integer port



@Autowired：引用类型赋值自动注入，支持byName，byType，默认是byType。

​		放在属性的上面，也可以放在构造方法的上面，推荐是放在构造方法上面。

@Qualifer：给引用类型赋值，使用ByName方式。

​		@Autowired和@Quailfer都是Spring框架提供的



@Resource：来自jdk的定义，javax.anntation。实现类型的自动注入，支持byName，byType 默认是byName

​		如果byName失败，在使用byType注入。在属性上面使用

