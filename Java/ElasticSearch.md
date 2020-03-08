![images](https://raw.githubusercontent.com/MrWater233/PictureHost/master/ElasticSearch%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

# SpringBoot集成

> SpringBoot默认使用SpringData操作ElasticSearch

## Jest

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

## SpringData ElasticSearch

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

**以6.8.1为例的docker安装：**

```shell
docker pull elasticsearch:6.8.1
docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9200:9200 -p 9300:9300 --name es 镜像ID
```

#### 容器无法启动问题

[ElasticSearch容器无法启动问题](https://blog.csdn.net/qq_43268365/article/details/88234308?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)

### 配置

> 需要配置Client的节点信息：clusterNodes、cluster-name是否启动repositorie

```yml
spring:
  data:
    elasticsearch:
      repositories:
        enabled: true
      cluster-nodes: xxx:9300 #部署的url接9300端口
      cluster-name: docker-cluster #进入xxx:9200端口可以查出，如下图所示
```

![images](https://raw.githubusercontent.com/MrWater233/PictureHost/master/20200229223353.png)

### 使用

#### ElasticsearchRepository

具体使用方法参照[官方文档](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.4.RELEASE/reference/html/#elasticsearch.repositories)

1. 创建需要存储的entity

   加入`@Document(indexName = "test", type = "book")`注解，指定索引和类型名

   `com.springbootelasticsearch.entity.Book`

   ```java
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   @Document(indexName = "demo", type = "book")
   public class Book {
       @Id
       private Integer id;
       private String bookName;
       private String author;
       private String description;
   }
   ```

2. 创建ElasticsearchRepository接口

   也可以通过`@Query(查询表达式)` `或者jpa命名规范`来自定义查找方法

   [jpa自定义查询表达式](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.4.RELEASE/reference/html/#elasticsearch.query-methods.criterions)

   `com.springbootelasticsearch.repository.BookRepository`

   ```java
   /**
    * 泛型参数：1.操作实体类型 2.实体主键类型
    */
   @Repository
   public interface BookRepository extends ElasticsearchRepository<Book, Integer> {
       /**
        * 自定义按照书名进行分页模糊查找
        *
        * @param bookName
        * @param pageable
        * @return
        */
       Page<Book> findByBookNameLike(String bookName, Pageable pageable);
   
       /**
        * 按照书名或者内容查找
        *
        * @param bookName
        * @param description
        * @param pageable
        * @return
        */
       Page<Book> findByBookNameLikeOrDescriptionLike(String bookName, String description, Pageable pageable);
   }
   ```

3. service层

   **注意**：jpa的分页从第0页开始
   
   `com.springbootelasticsearch.service.Impl.BookServiceImpl`
   
   ```java
   @Service
   @Slf4j
   public class BookServiceImpl implements BookService {
       @Autowired
       BookRepository bookRepository;
   
       /**
        * 保存书籍
        *
        * @param book
        * @return
        */
       public Integer createBooks(Book book) {
           Book save = bookRepository.index(book);
           return save.getId();
       }
   
       /**
        * 根据书名进行模糊分页搜索
        *
        * @param bookName
        * @return
        */
       public List<Book> searchBooks(String bookName, Integer pageNo, Integer pageSize) {
        Pageable pageable = PageRequest.of(pageNo, pageSize);
           return bookRepository.findByBookNameLike(bookName, pageable).getContent();
       }
   
       /**
        * 按照书名或者描述进行模糊分页查找
        *
        * @param content
        * @param pageNo
        * @param pageSize
        * @return
        */
       public List<Book> searchByNameOrDes(String content, Integer pageNo, Integer pageSize) {
           Pageable pageable = PageRequest.of(pageNo, pageSize);
           return bookRepository.findByBookNameLikeOrDescriptionLike(content, content, pageable).getContent();
       }
   ```
   
4. Controller层

   `com.springbootelasticsearch.controller.BookController`

   ```java
   @RestController
   public class BookController {
       @Autowired
       BookService bookService;
   
       @PostMapping("/api/book")
       public Integer createBook(Book book) {
           System.out.println(book);
           return bookService.createBooks(book);
       }
   
       @GetMapping("/api/book/search")
       public List<Book> searchBooks(@RequestParam("bookName") String bookName,
                                     @RequestParam("pageNo") Integer pageNo,
                                     @RequestParam("pageSize") Integer pageSize) {
           return bookService.searchBooks(bookName, pageNo, pageSize);
       }
   
       @GetMapping("/api/book/searchtoo")
       public List<Book> searchByNameOrDes(@RequestParam("content") String content,
                                           @RequestParam("pageNo") Integer pageNo,
                                           @RequestParam("pageSize") Integer pageSize) {
           return bookService.searchByNameOrDes(content,pageNo,pageSize);
       }
   }
   ```

   

