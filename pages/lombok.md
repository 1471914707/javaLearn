[原文路径](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485385&idx=2&sn=a7c3fb4485ffd8c019e5541e9b1580cd&chksm=cea24802f9d5c1144eee0da52cfc0cc5e8ee3590990de3bb642df4d4b2a8cd07f12dd54947b9&token=1381785723&lang=zh_CN#rd)

* **val**: 用在局部变量上，有点像js的val。
* **@cleanup**: 释放资源

```java
    try {
        @Cleanup InputStream inputStream = new FileInputStream(args[0]);
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
```

* **@NonNull**: 用于方法参数上，遇到空的值会抛出空指针。
* **@ToString**: 自动覆盖toString方法，例如@ToString(exclude=”id”)排除id属性，或者@ToString(callSuper=true, includeFieldNames=true)调用父类的toString方法，包含所有属性
* **@EqualsAndHashCode**： 用在类上，自动生成equals方法和hashCode方法**
* **@NoArgsConstructor, @RequiredArgsConstructor and @AllArgsConstructor**：用在类上，自动生成无参构造和使用所有参数的构造函数以及把所有@NonNull属性作为参数的构造函数，如果指定staticName = “of”参数，同时还会生成一个返回类对象的静态工厂方法，比使用构造函数方便很多
* **@Data**： 注解在类上，相当于同时使用了`@ToString`、`@EqualsAndHashCode`、`@Getter`、`@Setter`和`@RequiredArgsConstrutor`这些注解
* **@Value**：用在类上，是@Data的不可变形式，相当于为属性添加final声明，只提供getter方法，而不提供setter方法
* **@Bulider**: 用在类上，链式生成类
* **@Log**: @CommonsLog、@Log、@Log4j、@Log4j2、@Slf4j、@XSlf4j
* **@SneakyThrows**：自动抛受检异常，而无需显式在方法上使用throws语句
* **@Synchronized**：用在方法上，将方法声明为同步的，并自动加锁，而锁对象是一个私有的属性$lock或$LOCK
* **@Getter(lazy=true)**：可以替代经典的Double Check Lock样板代码

