# Swagger

学习目标：

- 了解Swagger的作用和概念

- 了解前后端分离

- 在SpringBoot中集成Swagger

  

前后端分离时代：

后端：后端控制层，服务层，数据访问层【后端团队】

前端：前端控制层，视图层【前端团队】

​	伪造后端数据，json。已经存在了，不需要后端，前端工程依旧能跑起来

前端后如何交互？==>API

前后端相对对立，松耦合；

前后端甚至可以部署在不同的服务器上；



产生一个生产问题：

前端后端集成联调，前端人员和后端人员无法做到“即时协商，尽早解决”，最终导致问题集中爆发

解决方案：

首选指定schema【计划的提纲】，实时更新最新API，降低集成的风险；

早些年：指定word计划文档；

前后端分离：

前端测试后端接口：postman

后端提供接口，需要实时更新最新的消息及改动！



### Swagger

号称世界上最流行的API框架；

RestFul Api 文档在线自动生成工具=>Api文档与API定义同步更新

直接运行，可以在线测试API接口；

支持多种语言：（Java，php....）

官网：https://swagger.io/

在项目中使用Swagger需要springbox

swagger2

ui

### springboot集成swagger

1、新建一个spring boot=web项目

2、导入相关依赖

```xml
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>
```

3、编写一个Hello工程

4、配置swagger==>config

```java

@Configuration
@EnableSwagger2  //开启swagger
public class SwaggerConfig {
}
//springboot2.5.6支持2.9.2swagger
//踩个坑：2.6.1 的 springboot 不支持 2.9.2 的 swagger 配置，报空指针错误。
//解决办法：降低springboot版本或者升级swagger版本。
```

5、测试运行:http://localhost:8080/swagger-ui.html

![image-20220504155345577](Swagger.assets/image-20220504155345577-1653916936469.png)

### 配置swagger

swagger的bean实例Docket；

```java
//配置Swagger 信息=apiInfo
    public ApiInfo apiInfo(){
        //作者信息
        Contact contact = new Contact("景昱", "null", "202525553@qq.com");
        return new ApiInfo(
                "景昱的SwaggerAPI文档",
                "只要学不死，就往死里学",
                "v1.0",
                "null",
                contact,
                "Apache 2.0",
                "http://www.apache.org/licenses/LICENSE-2.0" ,
                new ArrayList()
        );
    }
```

Swagger配置扫描接口

```java
 //配置了swagger的docket的bean实例
    @Bean
    public Docket docket(){
        return  new Docket(DocumentationType.SWAGGER_2)
        .apiInfo(apiInfo())
        .select()
        //RequestHandlerSelectors，配置要扫描接口方式
       //basePackage：指定要扫描的包
                //any():扫描全部
                //none():不扫描
                //withClassAnnotation：扫描类上的注解，参数是一个注解的反射对象
                //withMethodAnnotation:扫描方法上的注解
                //.apis(RequestHandlerSelectors.withMethodAnnotation())
        .apis(RequestHandlerSelectors.basePackage("com.jingyu.swagger.controller"))
                //paths 过滤路径
                .paths(PathSelectors.ant("/jingyu/**"))
        .build()
                ;
    }
```

配置是否启动Swagger

```java
  //配置了swagger的docket的bean实例
    @Bean
    public Docket docket(){
        return  new Docket(DocumentationType.SWAGGER_2)
        .apiInfo(apiInfo())
                .enable(false)//enable是否启动swagger，如果为false，则swagger不能在浏览器中访问
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.jingyu.swagger.controller"))
                .build()
                ;
    }
```

我只希望我的Swagger在生产环境中使用，在发布的时候不使用

判断是不是生产那环境 flag = false

注入enable（flag）

### 配置API文档分组

```java
 .groupName("景昱")
```

如何配置多个分组；多个Docket实例即可

```java
 @Bean
    public Docket docket1(){
        return new Docket(DocumentationType.SWAGGER_2).groupName("A");
    }

    @Bean
    public Docket docket2(){
        return new Docket(DocumentationType.SWAGGER_2).groupName("B");
    }

    @Bean
    public Docket docket3(){
        return new Docket(DocumentationType.SWAGGER_2).groupName("C");
    }
```

实体类配置

```java
package com.jingyu.swagger.pojo;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
//@Api(注释)
@ApiModel("用户实体类")
public class User {
    @ApiModelProperty("用户名")
    public String username;
    @ApiModelProperty("密码")
    public String password;
}

```

controller

```java
package com.jingyu.swagger.controller;

import com.jingyu.swagger.pojo.User;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiParam;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping(value = "/hello")
    public String hello(){
         return "hello";
    }
    //只要我们的接口中，返回值中存在的实体类，他就会被扫描到swagger中
    @PostMapping(value = "/user")
    public User user(){
        return new User();
    }
    //Operation接口,不是放在类上的，是方法
    @ApiOperation("hello控制类")
    @GetMapping(value = "/hello2")
    public String hello2( @ApiParam("用户名") String username){
        return "hello"+username;
    }

    @ApiOperation("post")
    @PostMapping(value = "/postt")
    public User postt( @ApiParam("用户名") User user){
        int a = Integer.parseInt("abc");
        return user;
    }
}

```

总结：

1.我们可以通过Swagger给一些比较难以理解的属性或者接口，增加注释信息

2.接口文档实时更新

3.可以在线测试

Swagger是一个优秀的工具

【注意点】在正式发布的时候，关闭Swagger！！！出于安全考虑。而且节省运行内存