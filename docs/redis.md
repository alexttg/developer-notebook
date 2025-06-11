#Redis运行Lua脚本
> 在一些互联网项目中，难免会设计到一些原子性操作，例如实时交易量的累加，分布式锁的实现，那么采用lua脚本可以帮助我们完成一个原子性的操作

Redis会使用相同的 Lua 解释器来运行所有命令。Redis 还保证以原子方式执行脚本：在执行脚本时不会执行其他脚本或 Redis 命令。

使用Lua脚本的好处有

● 避免多次网络IO请求，当业务场景需要多次操作Redis时可以封装Lua脚本，减少网络开销

● 多指令原子化，Redis使用独特的解释器将多个操作指令封装成同一指令执行

###在Redis中基本用法

```
EVAL script numkeys key [key ...] arg [arg ...]
```
例

	EVAL "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 value1 value2
	
参数说明：

1. `EVAL` ：Lua执行关键字

2. `KEYS[1]`：传入key数组中第一个元素也就是“key1”
3. `KEYS[2]`：传入key数组中第二个元素也就是“key2”
4. `ARGV[1]`：传入argv数组中第一个元素也就是“value1”
5. `ARGV[2]`：传入argv数组中第二个元素也就是“value2”
6. 最后的参数 2 为 key值数组的长度

![](https://foruda.gitee.com/images/1685612099841323833/945651be_5094274.png)

###Lua操作Redis基本数据结构

####Lua操作String

存

	EVAL "redis.call('SET',KEYS[1],ARGV[1])" 1 stringinser 123
取

	EVAL "return redis.call('GET',KEYS[1])" 1 stringinser
####Lua操作Map
存

	EVAL "redis.call('HSET',ARGV[1],KEYS[1],ARGV[2])" 1 mapinsert test 123
取

	EVAL "return redis.call('HGET',ARGV[1],KEYS[1])" 1 mapinsert test
####Lua操作List
存

	EVAL "redis.call('LPUSH',KEYS[1],ARGV[1])" 1 listinsert 123
取

	EVAL "return redis.call('LRANGE',KEYS[1],0,-1)" 1 listinsert
####Lua操作Set
存

	EVAL "redis.call('SADD',KEYS[1],ARGV[1])" 1 setinsert 123
取

	EVAL "return redis.call('SMEMBERS',KEYS[1])" 1 setinsert
	
####Lua操作Sorted Set
存

	EVAL "redis.call('ZADD',KEYS[1],ARGV[1],ARGV[2])" 1 zsetinsert 0 123
取

	EVAL "return redis.call('ZRANGE',KEYS[1],0,-1)" 1 zsetinsert
	
###Lua操作Redis特殊数据结构
####Lua操作Geo （计算上海到广州的直线距离）
存

	EVAL "return redis.call('GEOADD',KEYS[1],ARGV[1],ARGV[2],ARGV[3],ARGV[4],ARGV[5],ARGV[6])" 1 drawing 121.472644 31.231706 Shanghai 113.280637 23.125178 Guangzhou	

取

	EVAL "return redis.call('GEODIST',KEYS[1],ARGV[1],ARGV[2])" 1 drawing Shanghai Guangzhou

####Lua操作Bitmap（修改Redis值内存偏移量）

```
set bigtmap_a  a
EVAL "redis.call('SETBIT',KEYS[1],ARGV[1],ARGV[2]) redis.call('SETBIT',KEYS[1],ARGV[3],ARGV[4]) " 1 bigtmap_a 6 1 7 0
get bigtmap_a
```
Lua操作HyperLogLog（统计站点uv）
将访问页面货接口的每个用户信息添加到 HyperLogLog 中。

	EVAL "redis.call('PFADD',KEYS[1],ARGV[1]) redis.call('PFADD',KEYS[1],ARGV[2]) redis.call('PFADD',KEYS[1],ARGV[3]) " 1 uv user1 user2  user1
统计日活

	EVAL "return redis.call('PFCOUNT',KEYS[1]) " 1 uv
###Lua实现分布式锁
 lock
 
 	EVAL "local key=KEYS[1] local value = ARGV[1] if redis.call('SETNX',key,value) >0 then redis.call('SETEX', key,ARGV[2], value) return 0 end  return 1" 1 lock uid123 60

unlock

	EVAL "if redis.call('GET',KEYS[1]) ==ARGV[1] then redis.call('DEL', KEYS[1]) return 0 end return 1" 1 lock uid123
## 在Spring中运行Lua
 在实际开发中一般会使用Jedis或RedisTemplate等Redis的Java客户端库配合 `EVALSHA`命令运行Lua，不会使用`EVAL`，后面会说它们详细区别
####引入maven依赖

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>fastjson</artifactId>
	<version>2.0.7</version>
</dependency>
``` 
#### 初始化RedisTemplate至Spring容器


```
@Bean
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
    RedisTemplate<String, Object> template = new RedisTemplate<>();
    //使用fastjson作为序列化方式
    FastJsonRedisSerializer<Object> serializer = new FastJsonRedisSerializer<>(Object.class);
    //设置hash类型序列化方式
    template.setHashKeySerializer(new StringRedisSerializer());
    template.setHashValueSerializer(serializer);
    //设置string 序列化方式
    template.setKeySerializer(new StringRedisSerializer());
    template.setValueSerializer(serializer);
    template.setConnectionFactory(redisConnectionFactory);
    return template;
    }
```
####使用RedisTemplate运行Lua
RedisTemplate 做的事情主要是封装了Redis执行Lua的原生指令，为以下两种

```
EVALSHA sha1 numkeys [key [key ...]] [arg [arg ...]]
EVAL script numkeys [key [key ...]] [arg [arg ...]]

```

**EVALSHA 与 EVAL 的区别是前者会将Lua脚本缓存至Reids内存中并返回一串sha1值，下次执行相同命令时，便会通过相同的sha1值找到对应命令执行（若脚本改变了一个字符这sha1值也会改变）**

编写Lua脚本
> 第一次执行`EVALSHA`命令后将脚本缓存会返回一串与之对应的sha1值

```
#以下为分布式锁加锁
local key = KEYS[1] 
local value = ARGV[1]
local time = ARGV[2]
if redis.call('SETNX', key, value) > 0
  then
  redis.call('SETEX', key, time, value)
  return 0
end
return 1
```

例：

`3ac9e3871efe141e7b889378431957fc0a9878f8`  为sha1值

**运行以下命令**：

	EVALSHA 3ac9e3871efe141e7b889378431957fc0a9878f8  1 demo_key 1 60
运行以上命令相当于运行上面的Lua脚本

运行结果

![图片](https://foruda.gitee.com/images/1685612138649308888/574d76c6_5094274.png)

####在代码中运行上述Lua脚本

```
 /**
     * 使用类加载器获取lua脚本
     */
    private final ClassPathResource classPathResource = new ClassPathResource("lua/lock.lua");

    public void synchronizedByLua(String key, long value, long time) {
        RedisScript<Long> redisScript = RedisScript.of(classPathResource, Long.class);
        //sha1
        String sha1 = redisScript.getSha1();
        System.out.println(sha1);
        //执行Lua
        Long execute = redisTemplate.execute(redisScript, Lists.newArrayList(key), value, time);

        System.out.println(execute);
    }

```

