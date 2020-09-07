在开发环境下可以用过`ResourceUtils`来访问类路径，他是通过**文件路径**来进行访问的

```java
//初始化模板文件
templates = new ArrayList<>();
PathMatchingResourcePatternResolver patternResolver = new PathMatchingResourcePatternResolver();
try {
    File directory = ResourceUtils.getFile("classpath:template");
    if (directory.isDirectory()) {
        File[] files = directory.listFiles();
        for (File file : files) {
            String name = file.getName();
            templates.add("template/" + name);
        }
    }
} catch (FileNotFoundException e) {
    log.error("templates init error");
    throw new BizException(XstarterStatusCode.TEMPLATES_INIT_ERROR);
}
```



但是在Jar包环境下不能通过文件路径进行访问资源文件，这时候就需要通过**文件流**的方式去读取文件

```java
//初始化模板文件
templates = new ArrayList<>();
PathMatchingResourcePatternResolver patternResolver = new PathMatchingResourcePatternResolver();
try {
    Resource[] resources = patternResolver.getResources("template/*.vm");
    for (Resource resource : resources) {
        String fileName = resource.getFilename();
        templates.add("template/" + fileName);
    }
} catch (IOException e) {
    log.error("templates init error");
    throw new BizException(XstarterStatusCode.TEMPLATES_INIT_ERROR);
}
```



参考资料

1.https://stackoverflow.com/questions/25869428/classpath-resource-not-found-when-running-as-jar

2.https://blog.csdn.net/hero272285642/article/details/85119778?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param