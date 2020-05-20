# 一.基本概念

ES通过**倒排索引**来进行检索

![images](https://raw.githubusercontent.com/MrWater233/PictureHost/master/ElasticSearch%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

![](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200513094757.png)

# 二.Docker环境的搭建

## 1.ElasticSearch

```shell
docker pull elasticsearch:6.8.1
docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9200:9200 -p 9300:9300 -p 5601:5601 --name elasticsearch elasticsearch:6.8.1
```

5601端口：kibana的端口

9300端口：ES节点之间通讯使用

9200端口：ES节点和外部通讯使用

[ElasticSearch容器无法启动问题](https://blog.csdn.net/qq_43268365/article/details/88234308?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)

访问端口`9200`

## 2.IK分词器安装

[Github地址](https://github.com/medcl/elasticsearch-analysis-ik)并在release中找到需要下载版本的地址

1. 进入容器

   ```shell
   docker exec -it elasticsearch /bin/bash
   ```

2. 进入`plugins`目录

   ```shell
   cd plugins/
   ```

3. 安装IK分词器6.8.1版本

   ```shell
   elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.8.1/elasticsearch-analysis-ik-6.8.1.zip
   ```

4. 重启容器

   ```shell
   docker restart elasticsearch
   ```

## 3.Kibana

```shell
docker pull kibana:6.8.1
docker run -it -d -e ELASTICSEARCH_URL=http://127.0.0.1:9200 --name kibana --network=container:elasticsearch kibana:6.8.1
```

`--network`指定容器共享elasticsearch容器的网络栈 (使用了--network 就不能使用-p 来暴露端口)

访问端口`5601`

## 4.Logstash

1. 搭建mysql的Docker环境，容器命名为`mysql`

2. 创建数据库`test`并执行以下语句

   ```sql
   DROP TABLE IF EXISTS `t_blog`;
   CREATE TABLE `t_blog`  (
     `id` int(11) NOT NULL AUTO_INCREMENT,
     `title` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
     `author` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
     `content` mediumtext CHARACTER SET utf8 COLLATE utf8_general_ci NULL,
     `create_time` timestamp(0) NOT NULL DEFAULT CURRENT_TIMESTAMP(0),
     `update_time` timestamp(0) NOT NULL DEFAULT CURRENT_TIMESTAMP(0) ON UPDATE CURRENT_TIMESTAMP(0),
     PRIMARY KEY (`id`) USING BTREE
   ) ENGINE = InnoDB AUTO_INCREMENT = 5 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
   
   INSERT INTO `t_blog` VALUES (1, 'Java从入门到精通', '小明', NULL, '2020-05-19 16:22:03', '2020-05-19 16:21:41');
   INSERT INTO `t_blog` VALUES (2, 'Spring实战', '小红', NULL, '2020-05-19 16:22:03', '2020-05-19 16:21:41');
   INSERT INTO `t_blog` VALUES (3, 'SpringBoot从入门到精通', '小方', NULL, '2020-05-19 16:22:03', '2020-05-19 16:21:41');
   INSERT INTO `t_blog` VALUES (4, 'Redis从入门到精通', '小绿', '集成Spring', '2020-05-19 16:22:03', '2020-05-19 16:21:41');
   ```

3. 拉取镜像

   ```shell
   docker pull logstash:6.8.1
   ```

4. 创建Dockerfile文件

   ```config
   FROM logstash:6.8.1
   
   #安装input插件
   RUN logstash-plugin install logstash-input-jdbc
   
   #安装output插件
   RUN logstash-plugin install logstash-output-elasticsearch
   
   #容器启动时执行的命令,指定容器内的mysql.conf.(CMD 能够被 docker run 后面跟的命令行参数替换)
   CMD ["-f","/etc/logstash/pipeline/mysql.conf"]
   ```

5. 创建镜像

   ```shell
   Docker build -t my_logstash .
   ```

6. 创建`/usr/docker/logstash/`挂载目录

7. 下载JDBC连接包并放入，这里放入为`mysql-connector-java-8.0.17.jar`

8. 创建`logstash.yml`（

   ```yaml
   http.host: "0.0.0.0"
   xpack.monitoring.elasticsearch.url: http://ESIP:9200
   ```

9. 创建`log4j2.properties`

   ```properties
   logger.elasticsearchoutput.name = logstash.outputs.elasticsearch
   logger.elasticsearchoutput.level = debug
   ```

10. 创建`pipelines.yml`

    ```yaml
    - pipeline.id: my-logstash
      path.config: "/etc/logstash/pipeline/*.conf"
      pipeline.workers: 3
    ```

11. 创建`mysql.conf`

    通过`docker inspect --format '{{ .NetworkSettings.IPAddress }}' 容器id`查看容器的IP地址并配置

    ```yaml
    input {
      jdbc {
    	# JDBC驱动包在容器内的位置
    	jdbc_driver_library => "/etc/logstash/pipeline/mysql-connector-java-8.0.17.jar"
    	# 要使用的驱动包类
        jdbc_driver_class => "com.mysql.jdbc.Driver"
    	# MYSQL数据库连接信息
        jdbc_connection_string => "jdbc:mysql://MysqlIP:3306/test"
        jdbc_user => "root"
        jdbc_password => "123456"
    	# 定时任务，多久执行一次查询，默认一分钟，如果没有延迟可以指定"* * * * *"
        schedule => "* * * * *"
    	# 清空上一次的sql_last_value记录，如果为真那么每次都相当于从头开始查询所有的数据库记录
        clean_run => false
    	# 执行的SQL语句，这样可以保证刚好在更新时插入的数据不会被遗漏
        statement => "select * from t_blog where update_time > :sql_last_value and update_time < NOW() order by update_time desc"
      }
    }
    
    output {
      elasticsearch {
    	# ES host:port
        hosts => "ESIP:9200"
    	# 索引
        index => "blog"
    	# _id
        document_id => "%{id}"
      }
    }
    ```

12. 启动容器

    ```shell
    docker run -d -v /usr/docker/logstash/:/etc/logstash/pipeline/ --name=logstash my_logstash
    ```

13. 查看日志

    ```shell
    docker logs -f -t --tail 10 logstash
    ```

14. 在kibana中

    通过`GET /blog/_stats`查看索引状态

    通过`POST /blog/_search{}`查看文档

# 三.RestAPI的使用

## 索引

1. 查看所有索引：`GET:/_all`
2. 创建索引：`PUT:/索引名`
3. 删除索引：`DELETE:/索引名`

## 数据

> 因为ES推荐一个index下只有一个type，所以统一用`_doc`

1. 插入数据：`PUT:/索引名/_doc/id`然后发送一段JSON数据
2. 根据获取数据：`GET:/索引名/_doc/id`
3. 搜索数据：`GET:/索引名/_doc/_search?q=first_name:john`

# 四.分词器

## 内置分词器类型

1. `Standard Analyzer`
   1. 默认分词器
   2. 按词切分
   3. 小写处理，并支持删除停止词
   4. 中文以单字分词
2. `Simple Analyzer`
   1. 按照非字母切分
   2. 小写处理
   3. 去除掉数字类型字符
3. `Whitespace Analyzer`
   1. 空白字符作为分隔符
   2. 不支持中文
4. `Language Analyzer`
   1. 不支持中文

## Kibana测试分词

`standard`

```json
POST _analyze
{
  "analyzer": "standard",
  "text": "Hello World"
}
```

`IK`

```json
//智能分词
POST _analyze
{
  "analyzer": "ik_smart",
  "text": "你好啊"
}
//最大词元分词
POST _analyze
{
  "analyzer": "ik_max_word",
  "text": "你好啊"
}
```

# 五.SpringBoot集成

> SpringBoot默认使用SpringData操作ElasticSearch

## 1.Jest

> 默认不生效，需要导入Jest工具包

### 依赖

```xml
<dependency>
    <groupId>io.searchbox</groupId>
    <artifactId>jest</artifactId>
    <version>5.3.3</version>
</dependency>
```

5.x的ElasticSearch使用5.x的Jest，详细看[Jest官方Github](https://github.com/searchbox-io/Jest)

### 配置

```yml
spring:
  elasticsearch:
    jest:
      uris: http://xxx:9200
```

### 使用

1. 创建一个entity

   `com.springbootelasticsearch.entity.Article`

   ```java
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   public class Article {
       //标识Jest主键
       @JestId
       private Integer id;
       private String author;
       private String title;
       private String content;
   }
   ```

2. 操作

   ```java
   @SpringBootTest
   class SpringbootElasticsearchApplicationTests {
       /**
        * 自动注入es操作对象
        */
       @Autowired
       JestClient jestClient;
   
       /**
        * 测试索引
        */
       @Test
       void contextLoads() {
           Article article = new Article(1,"张三","标题","内容");
           /**
            * 构建一个索引
            * 指定对象->指定索引->指定类型->指定id（类的注解中已经指定）
            */
           Index index = new Index.Builder(article).index("test").type("articles").build();
           /**
            * 执行
            */
           try {
               jestClient.execute(index);
           } catch (IOException e) {
               e.printStackTrace();
           }
       }
   
       /**
        * 测试搜索
        */
       @Test
       public void search(){
           //json的查询表达式，可在官方文档中查询阅读
           String json = "{\n" +
                   "    \"query\" : {\n" +
                   "        \"match\" : {\n" +
                   "            \"content\" : \"内容\"\n" +
                   "        }\n" +
                   "    }\n" +
                   "}";
           /**
            * 构建一个搜索
            * 搜索表达式->指定索引->指定类型
            */
           Search search = new Search.Builder(json).addIndex("test").addType("articles").build();
           /**
            * 执行
            */
           try {
               SearchResult result = jestClient.execute(search);
               System.out.println(result.getJsonString());
           } catch (IOException e) {
               e.printStackTrace();
           }
       }
   }
   ```

## 2.SpringData ElasticSearch

[官方文档](https://docs.spring.io/spring-data/elasticsearch/docs/4.0.0.RELEASE/reference/html/)

> 操作：
>
> 1.使用ElasticsearchTemplate操作
>
> 2.编写一个ElasticsearchRepository的子接口来操作

### 注意

#### 镜像的安装

<font color=red>应当注意springdata与elasticsearch的版本匹配，不然启动会报`NoNodeAvailableException`错误</font>

![images](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200229213659.png)

 具体查看[version](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.0.RC3/reference/html/#preface.versions)

### 配置

> 需要配置Client的节点信息：clusterNodes、cluster-name

#### ~~配置文件配置（已过时）~~

```yml
spring:
  data:
    elasticsearch:
      cluster-nodes: xxx:9300 #部署的url接9300端口
      cluster-name: docker-cluster #进入xxx:9200端口可以查出，如下图所示
  # 日期格式化
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
```

![images](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200229223353.png)

#### TransportClient（不推荐）

官方在ES7以后就不再推荐，将在ES8以后废除

`com.learn.springboot.es.config.TransportClientConfig`

```java
import org.elasticsearch.client.Client;
import org.elasticsearch.client.transport.TransportClient;
import org.elasticsearch.common.settings.Settings;
import org.elasticsearch.common.transport.TransportAddress;
import org.elasticsearch.transport.client.PreBuiltTransportClient;
import org.springframework.context.annotation.Bean;
import org.springframework.data.elasticsearch.config.ElasticsearchConfigurationSupport;

import java.net.InetAddress;
import java.net.UnknownHostException;

/**
 * @author WanJingmiao
 * @Description
 * @date 2020/5/20 14:59
 * @Version
 */
@Configuration
public class TransportClientConfig extends ElasticsearchConfigurationSupport {
    @Bean
    public Client elasticsearchClient() throws UnknownHostException {
        //获取方式同上
        Settings settings = Settings.builder().put("cluster.name", "docker-cluster").build();
        TransportClient client = new PreBuiltTransportClient(settings);
        client.addTransportAddress(new TransportAddress(InetAddress.getByName("xx.xx.xx.xx"), 9300));
        return client;
    }
}
```

#### High Level REST Client（推荐）

官方推荐使用的连接方式

`com.learn.springboot.es.config.RestClientConfig`

```java
import org.elasticsearch.client.RestHighLevelClient;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.elasticsearch.client.ClientConfiguration;
import org.springframework.data.elasticsearch.client.RestClients;
import org.springframework.data.elasticsearch.config.AbstractElasticsearchConfiguration;

/**
 * @author WanJingmiao
 * @Description
 * @date 2020/5/20 15:05
 * @Version
 */
@Configuration
public class RestClientConfig extends AbstractElasticsearchConfiguration {
    @Override
    public RestHighLevelClient elasticsearchClient() {
        final ClientConfiguration clientConfiguration = ClientConfiguration.builder()
                .connectedTo("xx.xx.xx.xx:9200")
                .build();

        return RestClients.create(clientConfiguration).rest();
    }
}
```

### 使用

具体使用方法参照[官方文档](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.4.RELEASE/reference/html/#elasticsearch.repositories)

1. 配置实体映射

   `@Documen`注解，指定索引、类型名（官方推荐一个索引一个类型，默认单个类型为doc）、使用线上ES的配置和不在Springboot项目启动的时候重新创建Index 

   `com.learn.springboot.es.entity.mysql.MysqlBlog`

   ```java
   import lombok.Data;
   import org.springframework.data.annotation.Id;
   import org.springframework.data.elasticsearch.annotations.DateFormat;
   import org.springframework.data.elasticsearch.annotations.Document;
   import org.springframework.data.elasticsearch.annotations.Field;
   import org.springframework.data.elasticsearch.annotations.FieldType;
   
   import java.util.Date;
   
   /**
    * @author WanJingmiao
    * @Description
    * @date 2020/5/20 11:46
    * @Version
    */
   @Data
   @Document(indexName = "blog", type = "doc", useServerConfiguration = true, createIndex = false)
   public class EsBlog {
       /**
        * 对应ES的id字段
        */
       @Id
       private Integer id;
       /**
        * 配置对应ES中的数据类型和分词器
        */
       @Field(type = FieldType.Text,analyzer = "ik_max_word")
       private String title;
       @Field(type = FieldType.Text,analyzer = "ik_max_word")
       private String author;
       @Field(type = FieldType.Text,analyzer = "ik_max_word")
       private String content;
       /**
        * 配置时间类型和自定义的日期格式化（三种不同的格式化）
        */
       @Field(type = FieldType.Date,format = DateFormat.custom,pattern = "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis")
       private Date createTime;
       @Field(type = FieldType.Date,format = DateFormat.custom,pattern = "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis")
       private Date updateTime;
   }
   ```

2. 配置Dao

   `com.learn.springboot.es.entity.mysql.MysqlBlog`

   ```java
   import com.learn.springboot.es.entity.es.EsBlog;
   import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
   
   /**
    * @author WanJingmiao
    * @Description 指定实体类和主键的数据类型
    * @date 2020/5/20 11:58
    * @Version
    */
   public interface EsBlogRepository extends ElasticsearchRepository<EsBlog,Integer> {
   }
   ```

3. 使用

   `com.learn.springboot.es.controller.DataController`

   ```java
   @RestController
   public class DataController {
       @Autowired
       EsBlogRepository esBlogRepository;
   
       @PostMapping("/search")
       public Object search(String keyword) {
           //构建返回对象
           Map<String, Object> map = new HashMap<>();
           //使用Spring内置的watch计算耗时
           StopWatch watch = new StopWatch();
           watch.start();
           
           //构建Bool合并查询语句
           BoolQueryBuilder builder = QueryBuilders.boolQuery();
           //matchPhrase为精确匹配
           builder.should(QueryBuilders.matchPhraseQuery("title", keyword));
           builder.should(QueryBuilders.matchPhraseQuery("content", keyword));
           //构建成可执行的String，即DSL语句
           String s = builder.toString();
           //执行查询，转为Page类型
           Page<EsBlog> search = (Page<EsBlog>) esBlogRepository.search(builder);
           //将结果转为list
           List<EsBlog> content = search.getContent();
           map.put("list", content);
           
           watch.stop();
           long totalTimeMillis = watch.getTotalTimeMillis();
           map.put("duration", totalTimeMillis);
           return map;
       }
   }
   ```

   

