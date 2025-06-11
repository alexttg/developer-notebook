#Nacos
> Nacos 是一个基于云原生的动态服务发现、配置和管理平台，它可以帮助开发者更好地实现微服务架构中的动态服务发现和配置管理。

###如何安装

[官方文档](https://nacos.io/zh-cn/docs/quick-start.html)

**下载安装包**

通过 [Github](https://github.com/alibaba/nacos/releases) 选择你要下载 的压缩包，最好下载最新版本

**初始化数据**

创建数据库

```
CREATE DATABASE nacos_config DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

执行 Nacos 服务端包中的 SQL 脚本来初始化数据库，脚本文件位于路径 nacos/conf/nacos-mysql.sql。

> ${NACOS_HOME}：Nacos Server 所在的目录


	mysql -uroot -p nacos_config < ${NACOS_HOME}/conf/mysql-schema.sql

修改 application.properties，放开对应mysql注释填写自己mysql地址

```
db.num=1
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user=root
db.password.0=1234
```
###启动Nacos
进入Nacos的 bin 目录下，使用以下命令启动 Nacos

	sh startup.sh -m standalone

##集成Springboot

添加maven依赖

```
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    <version>2021.1</version>
</dependency>
```
在 `application.yml`文件中添加Nacos的地址与namespac空间命名的名称

```
nacos:
  config:
    bootstrap:
      #开启系统启动时预读取nacos的配置，用于满足@Value注入数据的场景
      enable: true
    # 配置所属命名空间的id,此处我们配置名称为dev的id，可以在命名空间列表查看id的值
    namespace: demo-namespace
    # 配置所属分组
    group: DEFAULT_GROUP
    # 配置ID
    data-id: demo
    # 配置文件类型,对应nacos配置页面的配置格式，默认是properties
    type: yaml
    # nacos服务器地址
    server-addr: 127.0.0.1:8848
    # 开启自动刷新nacos配置
    auto-refresh: true
    # 针对配置项同名的情况，是否允许nacos的配置覆盖本地的配置
    remote-first: true
  discovery:
    server-addr:127.0.0.1:8848

```

应用程序的启动类中添加 `@EnableDiscoveryClient `注解

```
@SpringBootApplication
@EnableDiscoveryClient
public class MinioDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(MinioDemoApplication.class, args);
    }

}

```