# Spring Boot整合Knife4j，美化强化丑陋的Swagger

一般在使用 Spring Boot 开发前后端分离项目的时候，都会用到 <a href="https://shmily0021.github.io/#/docs/home-page/spring/SpringBoot%E6%95%B4%E5%90%88Swagger">Swagger</a> （戳链接详细了解）。

但随着系统功能的不断增加，接口数量的爆炸式增长，Swagger 的使用体验就会变得越来越差，比如请求参数为 JSON 的时候没办法格式化，返回结果没办法折叠，还有就是没有提供搜索功能。

## 关于 Knife4j

Knife4j 的前身是 swagger-bootstrap-ui，是 springfox-swagger-ui 的增强 UI 实现。swagger-bootstrap-ui 采用的是前端 UI 混合后端 Java 代码的打包方式，在微服务的场景下显得非常臃肿，改良后的 Knife4j 更加小巧、轻量，并且功能更加强大。

springfox-swagger-ui 的界面长这个样子，说实话，确实略显丑陋。

官方文档：

> [https://doc.xiaominfo.com/knife4j/documentation/open](https://doc.xiaominfo.com/knife4j/documentation/)

码云地址：

> [https://gitee.com/xiaoym/knife4jopen in new window](https://gitee.com/xiaoym/knife4j)

示例地址：

> https://gitee.com/xiaoym/swagger-bootstrap-ui-demo

## SpringBoot整合 Knife4j

Knife4j 完全遵循了 Swagger 的使用方式，所以可以无缝切换。

第一步，在 pom.xml 文件中添加 Knife4j 的依赖（**不需要再引入 springfox-boot-starter**了，因为 Knife4j 的 starter 里面已经加入过了）。

```text
<dependency>
    <groupId>com.github.xiaoymin</groupId>
    <artifactId>knife4j-spring-boot-starter</artifactId>
    <!--在引用时请在maven中央仓库搜索3.X最新版本号-->
    <version>3.0.3</version>
</dependency>
```

第二步，配置类 SwaggerConfig 还是 Swagger 时期原来的配置。

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
                .apis(RequestHandlerSelectors.basePackage("com.ujiuye.controller")) // 指定 API 的接口范围为 controller 控制器。
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

第三步，新建测试控制器类 Knife4jController.java：

```java
@Api(tags = "测试 Knife4j")
@RestController
@RequestMapping("/knife4j")
public class Knife4jController {

    @ApiOperation("测试")
    @RequestMapping(value = "/test", method = RequestMethod.POST)
    public String test() {
        return "Hello knife4j!";
    }
}
```

第四步，由于 springfox 3.0.x 版本 和 Spring Boot 2.6.x 版本有冲突，所以还需要先解决这个 bug，一共分两步（在[Swaggeropen in new window](https://tobebetterjavaer.com/springboot/swagger.html) 那篇已经解释过了，这里不再赘述，但防止有小伙伴在学习的时候再次跳坑，这里就重复一下步骤）。

先在 application.yml 文件中加入：

```yaml
spring:
  mvc:
    path match:
      matching-strategy: ANT_PATH_MATCHER
```

再在 SwaggerConfig.java 中添加：

```java
@Configuration // 声明Java配置类
@EnableOpenApi // 开启Swagger
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

以上步骤均完成后，开始下一步，否则要么项目启动的时候报错，要么在文档中看不到测试的文档接口。

第五步，运行 Spring Boot 项目，浏览器地址栏输入以下地址访问 API 文档，查看效果。

> 访问地址（和 Swagger 不同）：http://localhost:8080/doc.html

![image-20230424210049040](https://github.com/shmily0021/Blog-note-gitalk/blob/main/img/Swagger/knife4j.png?raw=true)

## Knife4j 的功能特点

简单来介绍下 Knife4j 的 功能特点：

**1）支持登录认证**

Knife4j 和 Swagger 一样，也是支持头部登录认证的，点击「authorize」菜单，添加登录后的信息即可保持登录认证的 token。

如果某个 API 需要登录认证的话，就会把之前填写的信息带过来。

**2）支持 JSON 折叠**

Swagger 是不支持 JSON 折叠的，当返回的信息非常多的时候，界面就会显得非常的臃肿。Knife4j 则不同，可以对返回的 JSON 节点进行折叠。

![image-20230424210329045](https://github.com/shmily0021/Blog-note-gitalk/blob/main/img/Swagger/knife4j02.png?raw=true)

**3）离线文档**

Knife4j 支持把 API 文档导出为离线文档（支持 markdown 格式、HTML 格式、Word 格式），

![image-20230424210513072](https://github.com/shmily0021/Blog-note-gitalk/blob/main/img/Swagger/knife4j03.png?raw=true)

**4）全局参数**

当某些请求需要全局参数时，这个功能就很实用了，Knife4j 支持 header 和 query 两种方式。

**5）搜索 API 接口**

Swagger 是没有搜索功能的，当要测试的接口有很多的时候，当需要去找某一个 API 的时候就傻眼了，只能一个个去拖动滚动条去找。

更多功能请参考 ：<a href="https://doc.xiaominfo.com/docs/quick-start">官方文档</a>