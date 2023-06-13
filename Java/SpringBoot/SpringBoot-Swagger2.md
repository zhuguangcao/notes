### SpringBoot-API文档化 Swagger2

>一、maven配置

```java
<!--swagger 依赖-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<!--swagger UI展示页面-->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

>二、使用

1、增加swagger配置类

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class Swagger2 {
    //http://localhost:8088/swagger-ui.html
    //配置
    @Bean
    public Docket createRestApi (){
        return new Docket(
                //指定swagger api类型
                DocumentationType.SWAGGER_2)
                //配置api文档网站整体描述信息
                .apiInfo(apiInfo())
                //controller源码文件目录
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.springboot.b2c.controller"))
                //匹配所有的controller类
                .paths(PathSelectors.any())
                //构建
                .build();
    }
    //配置文档网站整体描述信息
    private ApiInfo apiInfo(){
        return new ApiInfoBuilder()
                //标题
                .title("restful api文档")
                //联系人信息
                .contact(new Contact("horace",
                        "http://www.baidu.com",
                        "xxx@email.com"))
                //描述
                .description("学习b2c单机架构网站api文档")
                //版本
                .version("1.0.0.1")
                //团队提供服务网站
                .termsOfServiceUrl("localhost:8088")
                //构建
                .build();
    }
}
```

2、整个api文档的说明书写方法

其中apiInfo方法，主要用来配置整个api网站的展示信息

```java
// 配置文档网站整体描述信息
private ApiInfo apiInfo(){
    return new ApiInfoBuilder()
            //标题
            .title("restful api文档")
            //联系人信息
            .contact(new Contact("horace",
                    "http://www.baidu.com",
                    "xxx@email.com"))
            //描述
            .description("学习b2c单机架构网站api文档")
            //版本
            .version("1.0.0.1")
            //团队提供服务网站
            .termsOfServiceUrl("localhost:8088")
            //构建
            .build();
}
```

**3、单个方法和参数的书写方法**

**（1）在类层级别使用的**

@ApiIgnore 在class之前使用的注解，有这个注解，这个接口类将不显示在api网站

参数：

- 无

@Api()：用在请求的类上，表示对类的说明，也代表了这个类是swagger2的资源

参数：

- tags：说明该类的作用，参数是个数组，可以填多个。
- value="该参数没什么意义，在UI界面上不显示，所以不用配置"
- description = "用户基本信息操作"

**（2）在方法层级使用**

@ApiOperation()：用于方法，表示一个http请求访问该方法的操作

参数：

- value="方法的用途和作用"
- notes="方法的注意事项和备注"
- tags：说明该方法的作用，参数是个数组，可以填多个

格式：tags={"作用1","作用2"}

- httpMethod：http请求方式GET POST PUT等

@ApiParam()：用于方法，参数，字段说明 表示对参数的要求和说明

参数：

- name="参数名称"
- value="参数的简要说明"
- defaultValue="参数默认值"
- required="true" 表示属性是否必填，默认为false

@ApiResponses：用于请求的方法上，根据响应码表示不同响应

一个@ApiResponses包含多个@ApiResponse

@ApiResponse：用在请求的方法上，表示不同的响应

参数：

- code="404" 表示响应码(int型)，可自定义
- message="状态码对应的响应信息

实例：

```java
import com.springboot.b2c.pojo.com.springboot.b2c.pojo.bo.UserBo;
import com.springboot.b2c.service.UserService;
import com.springboot.b2c.utils.JSONResult;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiParam;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@Api(value = "用户登录注册", tags = {"主要包含用户注册和登录接口"})
@RestController
public class PassportController {

    @Autowired
    private UserService userService;

    @ApiOperation(value = "检测用户名存在", notes = "检测用户名是否存在，如果不存在返回正确", httpMethod = "GET")
    @ApiParam(value = "用户名", name = "username", defaultValue = "", required = true)
    @GetMapping("/usernameIsExist")
    public JSONResult usernameIsExist(@RequestParam String username) {
        //判断参数是否为空
        if (StringUtils.isBlank(username)) {
            return JSONResult.errorMsg("用户名为空");
        }
        //查找用户是否存在
        Boolean isExist = userService.queryUsernameIsExist(username);
        if (isExist) {
            return JSONResult.errorMsg("账号已存在");
        }
        return JSONResult.ok();
    }
}
```

**3、在POJO层级使用**

@ApiModel()：用于响应实体类上，用于说明实体作用

参数：

- description="描述实体的作用"

@ApiModelProperty：用在属性上，描述实体类的属性

参数：

- value="用户名" 描述参数的意义
- name="name" 参数的变量名
- required=true 参数是否必选

```java
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

@ApiModel(value = "用户注册对象", description = "用户注册动作，客户端发送给服务端的数据对象")
public class UserBo {
    @ApiModelProperty(value = "用户名", name = "username", example = "138888888888", required = true)
    private String username;
    @ApiModelProperty(value = "密码", name = "password", example = "123456", required = true)
    private String password;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

