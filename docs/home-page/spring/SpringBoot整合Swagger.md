# Spring Boot整合Swagger-UI实现在线API文档

## 关于 Swagger

Swagger 是一个用于生成、描述和调用 RESTful 接口的 Web 服务。

![image-20230424191601954](https://github.com/shmily0021/Blog-note-gitalk/blob/main/img/Swagger/swagger01.png?raw=true)

换句话说，Swagger 就是将项目中想要暴露的接口展示在页面上，开发者可以直接进行接口调用和测试，能在很大程度上提升开发的效率。

比如说，一个后端程序员写了一个登录接口，想要测试自己写的接口是否符合预期的话，就得先模拟用户登录的行为，包括正常的行为（输入正确的用户名和密码）和异常的行为（输入错误的用户名和密码），这就要命了。

但有了 Swagger 后，可以通过简单的配置生成接口的展示页面，把接口的请求参数、返回结果通过可视化的形式展示出来，并且提供了便捷的测试服务。

- 前端程序员可以通过接口展示页面查看需要传递的请求参数和返回的数据格式，不需要后端程序员再编写接口文档了；
- 后端程序员可以通过接口展示页面测试验证自己的接口是否符合预期，降低了开发阶段的调试成本。

> Swagger 官网地址：https://swagger.io/

Swagger 定义了一套规范，你只需要按照它的规范去定义接口及接口相关的信息，然后通过 Swagger 衍生出来的一系列工具，就可以生成各种格式的接口文档，甚至还可以生成多种语言的客户端和服务端代码，以及在线接口调试页面等等。

那只要及时更新 Swagger 的描述文件，就可以自动生成接口文档了，做到调用端代码、服务端代码以及接口文档的一致性。

## 整合 Swagger-UI

> Swagger-UI 是一套 HTML/CSS/JS 框架，用于渲染 Swagger 文档，以便提供美观的 API 文档界面。
>
> 也就是说，Swagger-UI 是 Swagger 提供的一套可视化渲染组件，支持在线导入描述文件和本地部署UI项目。

第一步，在 pom.xml 文件中添加 Swagger 的 starter。

```text
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-boot-starter</artifactId>
    <version>3.0.0</version>
</dependency>
```

为什么是springfox-boot-starter而不是swagger-boot-starter呢？

- Swagger 是一种规范。
- springfox-swagger 是一个基于 Spring 生态系统的，Swagger 规范的实现。
- springfox-boot-starter 是 springfox 针对 Spring Boot 项目提供的一个 starter，简化 Swagger 依赖的导入，否则我们就需要在 pom.xml 文件中添加 springfox-swagger、springfox-swagger-ui 等多个依赖。

第二步，添加 Swagger 的 Java 配置。

```java
@Configuration // 声明Java配置类
@EnableOpenApi // 开启Swagger
public class SwaggerConfig {

    @Bean
    public Docket docket() {
        Docket docket = new Docket(DocumentationType.OAS_30)  //  OpenAPI 说明书
                .apiInfo(apiInfo()).enable(true) // 配置 API 文档基本信息，标题、描述、作者、版本等。
                .select()
                //apis： 添加swagger接口提取范围
                .apis(RequestHandlerSelectors.basePackage("top.codingmore.controller")) // 指定 API 的接口范围为 controller 控制器。 写控制器的包路径
                .paths(PathSelectors.any())  // 指定匹配所有的 URL。
                .build();

        return docket;
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("SpringBoot整合Swagger")
                .description("本项目是用来测试 SpringBoot 整合 Swagger 是否成功")
                .contact(new Contact("Shmily","https://shmily0021.github.io","shmily0021@yeah.net"))
                .version("v1.0")
                .build();
    }
}

```

1）@Configuration 注解通常用来声明一个 Java 配置类，取代了以往的 xml 配置文件，让配置变得更加的简单和直接。

2）@EnableOpenApi 注解表明开启 Swagger。

3）SwaggerConfig 类中包含了一个 @Bean 注解声明的方法 `docket()`，该方法会被 Spring 的 AnnotationConfigApplicationContext 或 AnnotationConfigWebApplicationContext 类进行扫描，然后添加到 Spring 容器当中。

简单描述一下 Swagger 的配置内容：

- `new Docket(DocumentationType.OAS_30)`，使用 3.0 版本的 Swagger API。OAS 是 OpenAPI Specification 的简称，翻译成中文就是 OpenAPI 说明书，Swagger 遵循的就是这套规范。
- `apiInfo(apiInfo())`，配置 API 文档基本信息，标题、描述、作者、版本等。
- `apis(RequestHandlerSelectors.basePackage("top.codingmore.controller"))` 指定 API 的接口范围为 controller 控制器。
- `paths(PathSelectors.any())` 指定匹配所有的 URL。

第三步，添加控制器类。

```java
@Api(tags = "测试 Swagger")
@RestController
@RequestMapping("/swagger")
public class SwaggerController {

    @ApiOperation("测试")
    @RequestMapping("/test")
    public String test() {
        return "Hello Swagger！";
    }
}
```

1）@Api注解，用在类上，该注解将控制器标注为一个 Swagger 资源。该注解有 3 个属性：

- tags，具有相同标签的 API 会被归在一组内展示
- value，如果 tags 没有定义，value 将作为 API 的 tags 使用。
- description，已废弃

2）@ApiOperation 注解，用在方法上，描述这个方法是做什么用的。该注解有 4 个属性：

- value 操作的简单说明，长度为120个字母，60个汉字。
- notes 对操作的详细说明。
- httpMethod HTTP请求的动作名，可选值有："GET", "HEAD", "POST", "PUT", "DELETE", "OPTIONS" and "PATCH"。
- code 默认为200，有效值必须符合标准的HTTP Status Code Definitions。

3）@RestController 注解，用在类上，是@ResponseBody和@Controller的组合注解，如果方法要返回 JSON 的话，可省去 @ResponseBody 注解。

4）@RequestMapping 注解，可用在类（父路径）和方法（子路径）上，主要用来定义 API 的请求路径和请求类型。该注解有 6 个属性：

- value，指定请求的实际地址
- method，指定请求的method类型， GET、POST、PUT、DELETE等
- consumes，指定处理请求的提交内容类型（Content-Type），例如 application/json, text/html
- produces，指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回
- params，指定request中必须包含某些参数值
- headers，指定request中必须包含某些指定的header值

第四步，启动服务，在浏览器中输入 `http://localhost:8080/swagger-ui/` 就可以访问 Swagger 生成的 API 文档了。

![image-20230424200730074](https://github.com/shmily0021/Blog-note-gitalk/blob/main/img/Swagger/swagger02.png?raw=true)

点开 get 请求的面板，点击「try it out」再点击「excute」可以查看接口返回的数据。

![image-20230424200910299](https://github.com/shmily0021/Blog-note-gitalk/blob/main/img/Swagger/swagger03.png?raw=true)

## 版本不兼容

在 Spring Boot 整合 Swagger 的过程中，我发现一个大 bug，Spring Boot 2.6.7 版本和 springfox 3.0.0 版本不兼容，启动的时候直接就报错了。

一路跟踪下来，发现 GitHub 上确认有人在 Spring Boot 仓库下提到了这个 bug。

> https://github.com/spring-projects/spring-boot/issues/28794

Spring Boot 说这是 springfox 的 bug。

有人提到的解决方案是切换到 SpringDoc。这样就需要切换注解 `@Api → @Tag`，`@ApiOperation(value = "foo", notes = "bar") → @Operation(summary = "foo", description = "bar")`，对旧项目不是很友好，如果是新项目的话，倒是可以直接尝试 SpringDoc。

还有人提出的解决方案是：

- 先将匹配策略调整为 ant-path-matcher（application.yml）。

```yaml
spring:
  mvc:
    path match:
      matching-strategy: ANT_PATH_MATCHER
```

- 然后在 Spring 容器中注入下面这个 bean，可以放在 SwaggerConfig 类中。

```java
public class SwaggerConfig {
    @Bean
    public static BeanPostProcessor springfoxHandlerProviderBeanPostProcessor() {
        return new BeanPostProcessor() {

            @Override
            public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
                if (bean instanceof WebMvcRequestHandlerProvider || bean instanceof WebFluxRequestHandlerProvider) {
                    customizeSpringfoxHandlerMappings(getHandlerMappings(bean));
                }
                return bean;
            }

            private <T extends RequestMappingInfoHandlerMapping> void customizeSpringfoxHandlerMappings(List<T> mappings) {
                List<T> copy = mappings.stream()
                        .filter(mapping -> mapping.getPatternParser() == null)
                        .collect(Collectors.toList());
                mappings.clear();
                mappings.addAll(copy);
            }

            @SuppressWarnings("unchecked")
            private List<RequestMappingInfoHandlerMapping> getHandlerMappings(Object bean) {
                try {
                    Field field = ReflectionUtils.findField(bean.getClass(), "handlerMappings");
                    field.setAccessible(true);
                    return (List<RequestMappingInfoHandlerMapping>) field.get(bean);
                } catch (IllegalArgumentException | IllegalAccessException e) {
                    throw new IllegalStateException(e);
                }
            }
        };
    }
}
```

> 解决方案地址：https://github.com/springfox/springfox/issues/3462

重新编译项目，就会发现错误消失了，我只能说GitHub 仓库的 issue 区都是大神！

查看 Swagger 接口文档，发现一切正常。

springfox 和 Spring 在 pathPatternsCondition 上产生了分歧，这两个步骤就是用来消除这个分歧的。

除此之外，还有另外一种保守的做法，直接将 Spring Boot 的版本回退到更低的版本，比如说 2.4.5。