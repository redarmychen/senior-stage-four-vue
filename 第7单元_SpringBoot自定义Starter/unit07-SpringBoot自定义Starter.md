第7单元 SpringBoot自定义Starter
==================

【授课重点】
============

1.  SpringBoot starter介绍；
2.  SpringBoot starter实现步骤；
3.  SpringBoot starter文件上传案例
4.  SpringBoot starter的使用
5.  SpringBoot的常用注解

【考核要求】
============

1.  掌握starter原理及开发步骤
2.  掌握SpringBoot的床用注解

【教学内容】
============

7.1 Spring Boot Starter简介
--------

>  Starter是Spring Boot中的一个非常重要的概念，Starter相当于模块，它能将模块所需的依赖整合起来并对模块内的Bean根据环境（ 条件）进行自动配置。使用者只需要依赖相应功能的Starter，无需做过多的配置和依赖，Spring Boot就能自动扫描并加载相应的模块。

```
1.它整合了这个模块需要的依赖库；
2.提供对模块的配置项给使用者；
3.提供自动配置类对模块内的Bean进行自动装配；
```

## 7.2 Starter的开发步骤

```css
1.新建Maven项目，在项目的POM文件中定义使用的依赖；
2.新建配置类，写好配置项和默认的配置值，指明配置项前缀；
3.新建自动装配类，使用@Configuration和@Bean来进行自动装配；
4.新建spring.factories文件，指定Starter的自动装配类；
```

## 7.3 Starter的开发案例

完成文件批量上传服务starter的开发：zhanggm-uploader-springboot-starter

### 7.3.1 创建SpringBoot项目

项目名称：zhanggm-uploader-springboot-starter

```xml
<!-- 添加父坐标 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.5.RELEASE</version>
</parent>
```

### 7.3.2 添加相关依赖

```xml
<!-- 自动配置 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-autoconfigure</artifactId>
</dependency>
<!--配置处理-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
<!--spring-webmvc-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.4.RELEASE</version>
    <scope>compile</scope>
</dependency>
<!--spring-lombok-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
<!--zhanggm-common-utils工具类工程-->
<dependency>
    <groupId>com.zhanggm.common</groupId>
    <artifactId>zhanggm-common-utils</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
<!--日志-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

### 7.3.3 编写文件服务Service

**文件上传返回的结果类**

```java
@Data
@AllArgsConstructor
public class FileResult {
    /** 文件名称 **/
    private String fileName;
    /** 文件访问服务地址 **/
    private String fileDomain;
    /** 文件访问Url **/
    private String fileUrl;
}
```

**属性文件配置类**

```java
@Setter
@Getter
@ConfigurationProperties(prefix = "file")
public class FileProperties {
    private String path;
    private String domain;
    private boolean enable;
}
```

**文件服务类**

```java

public class FileService {

    private Logger logger = LoggerFactory.getLogger(getClass());


    private FileProperties properties;

    public FileService(FileProperties properties) {
        this.properties = properties;
        logger.info("fileService enable:{}",properties.isEnable());
        logger.info("fileService upload path:{}",properties.getPath());
        logger.info("fileService domain:{}",properties.getDomain());
    }

    /**
     * 文件上传
     * @param files
     * @return
     */
    public List<FileResult> upload(MultipartFile files[]){
        List<FileResult> fileList = new ArrayList<>();
        for (MultipartFile file : files){
            //文件名称
            String originalFilename = file.getOriginalFilename();
            //调用工具类方法
            String extName = FileUtil.getExtName(originalFilename);
            String fileName = UUID.randomUUID()+extName;
            //保存文件
            File saveFile = new File(properties.getPath(),fileName);
            try {
                file.transferTo(saveFile);
                FileResult fileResult = new FileResult(fileName,properties.getDomain(),properties.getDomain()+fileName);
                fileList.add(fileResult);
            } catch (IOException e) {
                e.printStackTrace();
                continue;
            }
        }
        return fileList;
    }
}
```

**自动配置类**

```java

@Configuration
@EnableConfigurationProperties(FileProperties.class)
public class FileAutoConfiguration {

    @Autowired
    private FileProperties fileProperties;

    @Bean
    @ConditionalOnProperty(prefix = "file",value = "enable",havingValue = "true")
    public FileService fileService(){
        return new FileService(fileProperties);
    }

}
```

### 7.3.4 指定Starter的自动装配类 

新建文件：resources/META-INF/spring.factories

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.f1704.springboot.dubbo.starter.DubboServerAutoConfiguration,com.f1704.springboot.dubbo.starter.FileAutoConfiguration
```

### 7.3.5 安装Starter到Maven仓库 

```shell
mvn install
```

## 7.4 使用starter

1. 添加starter的依赖
2. 添加starter相关配置
3. 使用starter服务

## 7.5 SpringBoot注解

 https://www.cnblogs.com/wudimanong/p/10457211.html 