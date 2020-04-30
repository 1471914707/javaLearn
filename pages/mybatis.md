#{}是静态替换变量有注入风险，${}是利用PreparedStatement的？替换，相对于加''，经过预编译，是安全的，防止Sql注入。

select|update|delete|insert|resultMap|parameterMap|sql|include|selectKey|

动态标签trim|where|set|foreach|if|choose|when|otherwise|bind

> bind的作用是拼接字符串等操作，可以应对改数据库，sql注入
>
> <!-- and user_name like concat('%',#{userName},'%') -->               
>
> <bind name="userNameLike" value=" '%' + userName + '%' "/>
>
> and user_name like #{userNameLike}

不同的xml写一样的namespace和id，可以但会覆盖，因为用的map<namespace+id，object>，没有写namespace的话相同id则会报错。

**mybatis的executor执行器**：（严格限制在SqlSession生命周期范围）

1. **SimpleExecutor**：执行一次update或select，就开启一个Statement对象，用完立即关闭。
2. **ReuseExecutor**：执行update或select，以sql作为key查找Statement对象，存在就使用，不存在就创建，使用完不关闭，而是放在Map<String, Statement>重复利用。
3. **BatchExecutor**：执行update（jdbc批处理不支持select），将所有sql添加到批处理中(addBatch)，等待统一执行（executeBatch），它缓存多个Statement对象，先addBatch，再executeBatch

**拦截器**只能拦截四种类型的接口：Executor、StatementHander、ParameterHandler、ResultSetHandler。想要支持其他接口得重写configuration，可以对这四个接口所有方法进行拦截。

**缓存**：

* mybatis默认开启一级缓存，一级缓存是查询数据库前的最后一层缓存，使用PerpetualCache（即HashMap缓存），利用自动生成的cacheKey查询出数据，update/delete/insert提交回滚事务时会删除缓存。SqlSession级别的缓存。关闭方式有配置文件，插件，<insert ...flushCache="true">。
* 二级缓存比较复杂，涉及事务脏读，需要在**mapping.xml配置<cache />来开启。mapper级别的缓存。两个Mapper的namespace如果相同，那么这两个Mapper执行的sql查询会被缓存在同一个二级缓存中。要开启二级缓存需要在config配置文件中设置cacheEnabled属性为true。