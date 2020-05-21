第三单元 SpringBoot集成MyBatis
==================

【授课重点】
============

1.  SpringBoot日志配置使用；
2.  SpringBoot单元测试；
3.  SpringBoot集成MyBatis；
4.  SpringBoot集成FreeMarker；

【考核要求】
============

1.  掌握SpringBoot日志配置和使用
2.  能编写测试类，测试自己写的service
3.  了解定时器的应用场景，会配置、编写定时器。
4.  了解Servlet在SpringBoot框架中的配置步骤
5.  了解过滤器和拦截器的区别和使用。

【教学内容】
============

3.1 课程导入
--------

>  生产环境一旦出现问题，日志可以帮我们快速定位问题，解决问题 。

```java
try {
    //调用某服务
} catch(Exception e) {
    logger.error("错误信息", e);
}
```

>  定时器的应用场景非常多，比如我们购买火车票，下单后30分钟后没有付款，订单会取消，这时候我们写一个定时器，1分钟检查一下未付款的订单，如果超过30分钟，就执行订单取消操作。

```java
//查询未付款的订单
List<Order> orderList = orderService.getOrderListByStatus(1);
Date nowDate = new Date();
orderList.forEach(order->{
    //如果超过30分钟，则取消订单
    if(nowDate.getTime()-order.getCreateTime().getTime()>1000*60*30){
        orderService.cancelOrderById(order.getId());
    }
})
```



>  Servlet、过滤器、拦截器是web开发的核心。

![](media/330611-20171023144517066-24770749.png) 

 **SpringMVC的机制**是由同一个Servlet来分发请求给不同的Controller。

**过滤器**放在web资源之前，可以在请求抵达它所应用的web资源(可以是一个Servlet、一个Jsp页面，甚至是一个HTML页面)之前截获进入的请求。

 **SpringMVC 中的Interceptor 拦截器**的主要作用是拦截用户的请求并进行相应的处理。比如通过它来进行\**权限验证\**，或者是来\**判断用户是否登陆\**，或者是像12306 那样子\**判断当前时间是否是购票时间。\**  

3.2 SpringBoot日志配置使用
---------

 Spring Boot默认使用LogBack日志系统， LogBack默认将日志打印到控制台上。  

 新建的Spring Boot项目一般都会引用`spring-boot-starter`或者`spring-boot-starter-web`，而这两个起步依赖中都已经包含了对于`spring-boot-starter-logging`的依赖，所以，无需额外添加依赖。 

**日志配置：application.properties**

```properties
#日志配置
logging.level.com.gaofei.springboot.dao=info
logging.file.path=D:\\log\\
```

**Logger使用**

```java
@RestController
public class TestController {
    //private Logger logger = LoggerFactory.getLogger(getClass());
    private static Logger logger = LoggerFactory.getLogger(DemoController.class);
    
    @RequestMapping("hello")
    public Object hello(HttpServletRequest request){
        map.put("requestURI",request.getRequestURI());
        /** 打印日志 **/
        logger.info("requestURI:{}",request.getRequestURI());
        return map;
    }
}
```

## 3.3 SpringBoot单元测试

导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

创建测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class AppTests {
    
    private Logger logger = LoggerFactory.getLogger(getClass());
    
    @Autowired
    private DemoService demoService;
    
    @Test
    public void demoTest(){
        String zhangsan = demoService.hello("zhangsan");
        logger.info(zhangsan);
    }
    
}
```

## 3.4 Freemarker简介

 ![Figure](media/overview.png) 

​		FreeMarker是一款模板引擎： 即一种基于模板和要改变的数据， 并用来生成输出文本(HTML网页，电子邮件，配置文件，源代码等)的通用工具。 它不是面向最终用户的，而是一个Java类库，是一款程序员可以嵌入他们所开发产品的组件。 

 http://freemarker.foofun.cn/ 

## 3.5 SpringBoot集成FreeMarker

**导入依赖**

```xml
<!-- freemarker -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
```

**添加配置**

```properties
# FreeMarker Mvc配置
# 编码格式
spring.freemarker.charset=UTF-8
# freemarker模板后缀 默认是 .ftl
spring.freemarker.suffix=.html
#模板加载路径,默认路径是 classpath:/templates/
spring.freemarker.template-loader-path=classpath:/templates
#Content-Type值
spring.freemarker.content-type=text/html;charset=utf-8
#禁用模板缓存
spring.freemarker.cache=false
#数字格式化
spring.freemarker.settings.number_format=0.##
```

**编写Controller**

```java
@Controller
public class UserController {

    @Autowired
    private UserDao userDao;

    @RequestMapping("list")
    public String getPageInfo(User user, ModelMap modelMap
  			@RequestParam(value = "pageNum",defaultValue = "1") Integer pageNum,
            @RequestParam(value = "pageSize",defaultValue = "2") Integer pageSize){
        //分页插件
        PageHelper.startPage(pageNum,pageSize);
        List<User> userList = userDao.select(user);
        modelMap.addAttribute("pageInfo",new PageInfo<>(userList));
        return "list";
    }
}
```

**编辑模板**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Freemarker模板</title>
</head>
<body>
    <table>
        <#list pageInfo.list as item>
            <tr>
                <td>${item.id}</td>
                <td>${item.username}</td>
            </tr>
        </#list>
    </table>
</body>
</html>
```

**启动项目，访问列表页**

## 3.6FreeMarker语法

 https://www.cnblogs.com/Jimc/p/9791507.html 