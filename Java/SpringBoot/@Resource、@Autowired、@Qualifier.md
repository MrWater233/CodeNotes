# @Autowired

Spring按照byType的方式在容器中查找注入，即找到同类型的类注入

当容器中存在多个像同类的时候就会报错

要求容器中必须存在对象，当把required的属性设置为false时允许null值的存在

# @Resource

默认通过byName（通过bean的name属性查找）的方式在容器中查找注入，若byName查找不到再通过byType的方式查找注入

# @Qualifier

只通过byName的方式查找