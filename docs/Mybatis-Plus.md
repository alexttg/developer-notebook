#Mybatis-Plus框架入门级教程

###1、框架背景
> （简称 MP）是一个 MyBatis(opens new window)的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

##集成SpringBoot
####引入相关依赖

```
<!--  mybatis-plus-->
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>mybatis-plus-boot-starter</artifactId>
  <version>3.5.2</version>
</dependency>
<!-- web-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!--lombok-->
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <version>1.18.4</version>
</dependency>
<!--相关驱动包-->
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
</dependency>
```

####配置yml
```
server:
  port: -1

spring:
  application:
    name: mybatis-plus-demo
  #mysql
  datasource:
    url: jdbc:mysql://localhost:3306/demo?useUnicode=true&characterEncoding=utf8&allowMultiQueries=true
    username: root
    password: root
    driverClassName: com.mysql.jdbc.Driver
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    maxPoolPreparedStatementPerConnectionSize: 20
    filters: stat,wall,log4j
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
    initialSize: 15
    minIdle: 15
    maxActive: 16
    type: com.alibaba.druid.pool.DruidDataSource

mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  mapper-locations: classpath*:mapper/*.xml
```
#### 配置Mybatis-plus包扫描路径
![](https://foruda.gitee.com/images/1685611862162394327/d869ca66_5094274.png)


**这个路径必须是项目mapper文件的存放路径**

```
@SpringBootApplication
@MapperScan("com.example.mybatisplus.mapper")
public class MybatisPlusApplication {

    public static void main(String[] args) {
        SpringApplication.run(MybatisPlusApplication.class, args);
    }

}
```

#### 配置Mybatis-Plus插件

```
@Configuration
public class MybatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        //添加乐观锁插件
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        //添加分页插件
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor());
        return interceptor;
    }
    
}
```
#### 配置Entity
```
@Data
@Builder
@TableName("user")
@AllArgsConstructor
@NoArgsConstructor
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String account;
    private Integer age;
    private String mobile;
    private String email;
    private String name;
    @Version
    private Integer version;
}
```
`@TableName` 指定表名，不指定则为类名，`@TableId`指定ID递增方式，规则如下，一般为`AUTO`

```

public enum IdType {
    /**
     * 数据库ID自增
     * <p>该类型请确保数据库设置了 ID自增 否则无效</p>
     */
    AUTO(0),
    /**
     * 该类型为未设置主键类型(注解里等于跟随全局,全局里约等于 INPUT)
     */
    NONE(1),
    /**
     * 用户输入ID
     * <p>该类型可以通过自己注册自动填充插件进行填充</p>
     */
    INPUT(2),

    /* 以下2种类型、只有当插入对象ID 为空，才自动填充。 */
    /**
     * 分配ID (主键类型为number或string）,
     * 默认实现类 {@link com.baomidou.mybatisplus.core.incrementer.DefaultIdentifierGenerator}(雪花算法)
     *
     * @since 3.3.0
     */
    ASSIGN_ID(3),
    /**
     * 分配UUID (主键类型为 string)
     * 默认实现类 {@link com.baomidou.mybatisplus.core.incrementer.DefaultIdentifierGenerator}(UUID.replace("-",""))
     */
    ASSIGN_UUID(4);
}
```

####配置Service

	public interface UserService  extends IService<User> {}
	
如果不需要mybatis-plus IService 中封装的公共方法，那么这步可以省略

```
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
}
```

**配置Mapper Xml**

xml文件的存放路径，需要yml配置的一致（需要让），如果不需要手写sql，那么这步可以省略

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mybatisplus.mapper.UserMapper">
</mapper>
```
###Mybtais-Plus 构造器使用相关Demo
**新增**

```
class MybatisPlusApplicationTests {

    @Autowired
    UserMapper userMapper;

    @Autowired
    UserService userService;

    /**
     * 单个新增
     */
    @Test
   public void insert() {
        User jack = User.builder()
                .account("25235262")
                .age(22)
                .mobile("111782721")
                .name("jack")
                .email("3232131336@gmail.com")
                .build();
        userMapper.insert(jack);
    }
}
```
SQL 执行日志

```
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@2520aa05] will not be managed by Spring
==>  Preparing: INSERT INTO user ( account, age, mobile, email, name ) VALUES ( ?, ?, ?, ?, ? )
==> Parameters: 25235262(String), 22(Integer), 111782721(String), 3232131336@gmail.com(String), jack(String)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@5423a17]
2022-12-08 19:25:21.213  INFO 48976 --- [ionShutdownHook] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} closed
```
**批量新增**

```
class MybatisPlusApplicationTests {

    @Autowired
    UserMapper userMapper;

    @Autowired
    UserService userService;

    /**
     * 
     * 批量新增
     */
    @Test
    void saveBatch() {
        User jack = User.builder()
                .account("25235262")
                .age(22)
                .mobile("111782721")
                .name("jack")
                .email("3232131336@gmail.com")
                .build();
        User molly = User.builder()
                .account("8488828")
                .age(12)
                .mobile("78878876")
                .name("molly")
                .email("399hys6222@gmail.com")
                .build();
        userService.saveBatch(Arrays.asList(jack,molly));
    }
}
```
SQL 执行日志

```
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@6996bbc4] will be managed by Spring
==>  Preparing: INSERT INTO user ( account, age, mobile, email, name ) VALUES ( ?, ?, ?, ?, ? )
==> Parameters: 25235262(String), 22(Integer), 111782721(String), 3232131336@gmail.com(String), jack(String)
==> Parameters: 8488828(String), 12(Integer), 78878876(String), 399hys6222@gmail.com(String), molly(String)
2022-12-09 11:05:50.796  INFO 59682 --- [ionShutdownHook] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} closed
```
**通过主键id修改**

```
@Test
void updateById() {
    User build = User.builder()
            .id(1L)
            .account("123")
            .age(31)
            .build();
    userMapper.updateById(build);
}
```
**自定义条件修改**

```
@Test
void update() {
    User build = User.builder()
            .email("123@gmail.com")
            .age(31)
            .build();
    userMapper.update(build,Wrappers.lambdaQuery(User.class).eq(User::getAge,11));
}
```
SQL 执行日志

```
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@3caf5c96] will not be managed by Spring
==>  Preparing: UPDATE user SET age=?, email=? WHERE (age = ?)
==> Parameters: 31(Integer), 123(String), 11(Integer)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@6c5ae8fd]
2022-12-08 19:12:27.753  INFO 48712 --- [ionShutdownHook] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} closed
```
**根据id删除**

```
@Test
    void deleteById() {
    userMapper.deleteById(1);
}
```
SQL 执行日志

```
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@5529522f] will not be managed by Spring
==>  Preparing: DELETE FROM user WHERE id=?
==> Parameters: 1(Integer)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@42576db9]
2022-12-09 11:13:57.860  INFO 59805 --- [ionShutdownHook] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} closed
```

**构造条件删除**

```
@Test
    void delete() {
    userMapper.delete(Wrappers.lambdaQuery(User.class).le(User::getAge,10));
}
```
SQL 执行日志

```
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@5af38a4a] will not be managed by Spring
==>  Preparing: DELETE FROM user WHERE (age <= ?)
==> Parameters: 10(Integer)
<==    Updates: 1
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@166e5a6d]
2022-12-09 11:17:31.279  INFO 59870 --- [ionShutdownHook] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} closed
```
**通过主键id查询**

```
@Test
void selectById() {
    User user = userMapper.selectById(1);
}
```
SQL 执行日志

```
==>  Preparing: SELECT id,account,age,mobile,email,name,version FROM user WHERE id=?
==> Parameters: 1(Integer)
<==      Total: 0
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@23bd047f]
null
2022-12-09 11:19:00.466  INFO 59890 --- [ionShutdownHook] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} closed
```
**构造条件查询**

```
@Test
void selectList() {

   List<User> users =  userMapper.selectList( Wrappers.lambdaQuery(User.class)
                  .eq(User::getAge,30)
                  .ne(User::getAccount,"jack"));
}
```
SQL 执行日志

```
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@63c84d31] will not be managed by Spring
==>  Preparing: SELECT id,account,age,mobile,email,name,version FROM user WHERE (age = ? AND account <> ?)
==> Parameters: 11(Integer), 3131313(String)
<==      Total: 0
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@77a2688d]
2022-12-09 11:24:35.295  INFO 60020 --- [ionShutdownHook] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} closed
```
**构造条件分页查询**

```
    @Test
    void selectPage() {
        //当前页面1，每页10条
        Page<User> page = new Page<>(1, 10);
        Page<User> userPage = userMapper.selectPage(page, Wrappers.lambdaQuery(User.class)
                .eq(User::getAge, 11)
                .ne(User::getAccount, "3131313"));
        //总页数
        long pages = userPage.getPages();
        //数据集
        List<User> records = userPage.getRecords();
        //当前页
        long current = userPage.getCurrent();
        //总条数
        long total = userPage.getTotal();
    }
```
SQL 执行日志

```
JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@1aaf46e6] will not be managed by Spring
==>  Preparing: SELECT COUNT(*) AS total FROM user WHERE (age = ? AND account <> ?)
==> Parameters: 11(Integer), 123(String)
<==    Columns: total
<==        Row: 3
<==      Total: 1
==>  Preparing: SELECT id,account,age,mobile,email,name,version FROM user WHERE (age = ? AND account <> ?) LIMIT ?
==> Parameters: 11(Integer), 123(String), 10(Long)
<==    Columns: id, account, age, mobile, email, name, version
<==        Row: 8, 25235262, 11, 111782721, 3232131336@gmail.com, jack, null
<==        Row: 9, 8488828, 11, 78878876, 399hys6222@gmail.com, molly, null
<==        Row: 13, 25235262, 11, 111782721, 3232131336@gmail.com, jack, 1
<==      Total: 3
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@606b53a3]
2022-12-09 11:33:36.049  INFO 60188 --- [ionShutdownHook] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} closed
```



