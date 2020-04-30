* 相对于spring更加纯注解开发，减少配置和大量样板代码。
* 与spring生态系统的天然结合。
* 可以使用内置工具(如Maven和Gradle)开发管理jar和测试Spring Boot应用程序。
* 提供内嵌的HTTP服务器。
* ~~提供命令行接口工具~~。

**1. 读取配置文件信息**：

* **@Value("${property}")**  不被推荐了。
* **@ConfigurationProperties(prefix = "library")** 与bean绑定，加注解@Component。
* **@EnableConfigurationProperties(ProfileProperties.class)** 用于启动类上， 配置类用@ConfigurationProperties("my-profile")，这个可以加@notnull之类的校验。
* **@PropertySource("classpath:website.properties")** 读取指定文件，加@Component。

**2.全局异常**：

* @ControllerAdvice(assignableTypes = {ExceptionController.class})用于类、@ExceptionHandler(value = Exception.class)用于方法。
* @ResponseStatus(code = HttpStatus.NOT_FOUND) 返回特定的码，用于异常类。
* 方法里throw new ResponseStatusException，没效果。

**3. 过滤器**：

* @Configuration类下生成FilterRegistrationBean<MyFilter>的@Bean

```java
@Bean
    public FilterRegistrationBean<MyFilter> setUpMyFilter() {
        FilterRegistrationBean<MyFilter> filterRegistrationBean = new                FilterRegistrationBean<>();
        filterRegistrationBean.setOrder(2);
        filterRegistrationBean.setFilter(myFilter);
        filterRegistrationBean.setUrlPatterns(new ArrayList<>(Arrays.asList("/api/*")));

        return filterRegistrationBean;
    }
```

多个过滤器按指定顺序执行的话需要用上面的方法。

* @WebFilter(filterName = "MyFilterWithAnnotation", urlPatterns = "/api/*")，启动类要加@ServletComponentScan。

这里提下过滤器和拦截器：拦截器是基于Java的反射机制，AOP差不多，过滤器基于函数回调，灵活程度上是拦截器更强大，因为拦截器能除了能做过滤器的事情，还可以在控制器执行前后和视图渲染前后处理逻辑。过滤器主要针对URL地址做编码，过滤没用的参数和安全登录校验。

**4.参数校验**：

springboot引进来的基础包上已经包含所要用的校验注解，在controller上加注解@Validated，方法上无论传的参数还是对象，都在前面加@Valid，单独参数除了加@Valid，还需要直接加@NotNull之类的校验规则。值得一提的是，因为之前有同事说处理参数校验出错的时候，不用在代理里处理bindResult，其实这种说法并不对，因为不单独处理的话时候会抛出异常MethodArgumentNotValidException，如果使用全局异常处理的话，就可以集中处理这类问题。

除了contoller之外，service这些也可以使用

```java
@Service
@Validated
public class PersonService {

    public void validatePerson(@Valid Person person){
        // do something
    }
}
```

* 自定义校验，实现的一个注解@interface，再实现一个具体类实现ConstraintValidator接口。
* 校验组，例@NotNull(groups = DeletePersonGroup.class)，DeletePersonGroup是静态内部接口，在方法参数上使用@Validated(AddPersonGroup.class)即可生效，它的作用是同一个字段可能在不同场景有不同的校验规则，这个时候就可以根据校验组灵活使用。