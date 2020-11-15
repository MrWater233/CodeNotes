# 一.基础配置

引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

yml配置

```yaml
spring:
  redis:
    database: 0
    host: 127.0.0.1
    port: 6379
    password: 123456
```

# 二.两种Template

`SpringData Redis`提供了两种template对Redis的一些常用操作进行封装

他们分别是

- `RedisTemplate`
- `StringRedisTemplate`

## 2.1 区别

他们的主要区别是使用的序列化类不同

- `RedisTemplate`使用的是`JdkSerializationRedisSerializer`

  ```java
  @Autowired
  private RedisTemplate<String,Object> redisTemplate;
  
  @Test
  public void setValue() {
      redisTemplate.opsForValue().set("test1","redis");
  }
  ```

  ![image-20201109144624089](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20201109144624.png)

- `StringRedisTemplate`使用的是`StringRedisSerializer`

  ```java
  @Autowired
  private StringRedisTemplate stringRedisTemplate;
  
  @Test
  void setStringValue() {
      stringRedisTemplate.opsForValue().set("test2","redis");
  }
  ```

  ![image-20201109144706669](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20201109144706.png)

## 2.2 使用场景

- 当需要从Redis中存取的Value为字符串，那么推荐使用`StringRedisTemplate`
- 当需要从Redis中存取的Value为复杂的对象类型时，推荐使用`RedisTemplate`

# 三.配置序列化

由于`RedisTemplate`使用的是JDK默认的序列化器

当我们使用Redis管理工具查看时不容易阅读

可以通过配置来更改他的序列化器。这里将key设置为字符串序列化器，value设置为json序列化器

```java
@Configuration
public class RedisConfig {

	@Bean
	public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
		RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
		redisTemplate.setConnectionFactory(redisConnectionFactory);
		// 设置key的序列化方式
		redisTemplate.setKeySerializer(new StringRedisSerializer());
		// 设置value的序列化方式
		redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
		return redisTemplate;
	}

}
```

# 四.Redis中的数据结构

> 因为Redis是一个K-V数据库，所以这里的数据结构指的是Value的数据结构
>
> 这里为了简化，使用`StringRedisTemplate`进行测试

## 4.1 字符串（String）

> Redis中最基本的数据结构，采用单Key单Value（字符串）的方式存储
>
> Redis采用SDS（简单动态字符串）来实现存储

常用操作（下面的`key`指key的名称）：

```java
@SpringBootTest
public class RedisStringTest {

	@Autowired
	private StringRedisTemplate stringRedisTemplate;

	/**
	 * 设置K-V.如果存在相同的key会覆盖value
	 */
	@Test
	void set() {
		stringRedisTemplate.opsForValue().set("key", "value");
	}

	/**
	 * 当key不存在时添加K-V
	 */
	@Test
	void setIfAbsent() {
		stringRedisTemplate.opsForValue().setIfAbsent("key", "value");
	}

	/**
	 * 当key存在时设置value
	 */
	@Test
	void setIfPresent() {
		stringRedisTemplate.opsForValue().setIfPresent("key", "value1");
	}

	/**
	 * 设置K-V并设置超时时间
	 */
	@Test
	void setWithTime() {
		stringRedisTemplate.opsForValue().set("key", "value", Duration.ofSeconds(60L));
	}

	/**
	 * 批量设置K-V
	 */
	@Test
	void multiSet() {
		Map<String, String> map = new HashMap<>();
		map.put("key1", "value");
		map.put("key2", "value");
		stringRedisTemplate.opsForValue().multiSet(map);
	}

	/**
	 * 获取字符串
	 */
	@Test
	void get() {
		String value = (String) stringRedisTemplate.opsForValue().get("key");
		System.out.println(value);
	}

	/**
	 * 批量获取字符串value
	 */
	@Test
	void multiGet() {
		List<String> keys = new LinkedList<>();
		keys.add("key1");
		keys.add("key2");
		List<String> values = stringRedisTemplate.opsForValue().multiGet(keys);
		values.forEach(System.out::println);
	}

	/**
	 * 获取字符串value的大小
	 */
	@Test
	void getSize() {
		Long size = stringRedisTemplate.opsForValue().size("key");
		// 5
		System.out.println(size);
	}

	/**
	 * 删除
	 */
	@Test
	void delete() {
		stringRedisTemplate.delete("key");
	}

}
```

## 4.2 链表（List）

> 链表的左右都可以添加元素
>
> 如果值全部都被移除，那么key也会被移除

常用操作（下面的`list`指key的名称）：

```java
@SpringBootTest
public class RedisListTest {

	@Autowired
	StringRedisTemplate stringRedisTemplate;

	/**
	 * 从链表左侧开始加入元素
	 */
	@Test
	void leftPush() {
		stringRedisTemplate.opsForList().leftPush("list", "v1");
	}

	/**
	 * 从链表左侧批量插入元素。插入后链表顺序为：v4 v3 ...
	 */
	@Test
	void leftPushAll() {
		stringRedisTemplate.opsForList().leftPushAll("list", "v3", "v4");
	}

	/**
	 * 从链表右侧开始加入元素
	 */
	@Test
	void rightPush() {
		stringRedisTemplate.opsForList().rightPush("list", "v2");
	}

	/**
	 * 从链表右侧开始批量插入元素。插入后链表顺序为：... v5 v6
	 */
	@Test
	void rightPushAll() {
		stringRedisTemplate.opsForList().rightPushAll("list", "v5", "v6");
	}

	/**
	 * 弹出最左边的元素
	 */
	@Test
	void leftPop() {
		String value = stringRedisTemplate.opsForList().leftPop("list");
		System.out.println(value);
	}

	/**
	 * 弹出最右边的元素
	 */
	@Test
	void rightPop() {
		String value = stringRedisTemplate.opsForList().rightPop("list");
		System.out.println(value);
	}

	/**
	 * 获取指定索引下标的value，从0开始
	 */
	@Test
	void getIndex() {
		String value = stringRedisTemplate.opsForList().index("list", 0L);
		System.out.println(value);
	}

	/**
	 * 获取指定索引下标范围的值(闭区间)。如果是0,-1默认为整个list，因为-1指的是最后一个元素
	 */
	@Test
	void getRange() {
		List<String> values = stringRedisTemplate.opsForList().range("list", 0L, 1L);
		values.forEach(System.out::println);
	}

	/**
	 * 只保留指定区间内的元素。-1指最后一个元素
	 */
	@Test
	void setIndex() {
		stringRedisTemplate.opsForList().trim("list", 1, -1);
	}

	/**
	 * 从左到右查找，删除1个值为"v1"的元素
	 */
	@Test
	void delete() {
		stringRedisTemplate.opsForList().remove("list", 1, "v1");
	}

	/**
	 * 获取链表元素个数
	 */
	@Test
	void getSize() {
		Long size = stringRedisTemplate.opsForList().size("list");
		System.out.println(size);
	}
}
```

## 4.3 集合（Set）

> 无重复元素

