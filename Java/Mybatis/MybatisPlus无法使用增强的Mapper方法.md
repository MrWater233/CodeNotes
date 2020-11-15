# 一.起因

这次用MybatisPlus重构一个Mybatis项目，但是无法使用BaseMapper提供的增强方法

报错如下：

```txt
org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): com.xhiteam.usercenter.mapper.TestDataMapper.selectById
```

# 二.原因

经过再三确认，MybatisPlus的基础配置是没有配错的，经过断断续续好几天的问题查询终于找到了原因

原因是默认容器默认注入了了Mybatis的`SqlSessionFactory`而没有使用MybatisPlus的`MybatisSqlSessionFactoryBean`

既然原因找到了，那么接下来的的解决就相对简单了

# 三.解决方法

使用MybatisPlus提供的`MybatisSqlSessionFactoryBean`

因为使用了自定义的SqlSession，MybatisPlus的配置文件会失效，所以得在配置中设置

```java
@Configuration
public class MybatisPlusConfiguration {

	/**
	 * 将SqlSessionFactory配置为MybatisPlus的MybatisSqlSessionFactoryBean
	 *
	 * @param dataSource 默认配置文件中的数据源
	 * @return MybatisSqlSessionFactoryBean
	 */
	@Bean("mpSqlSessionFactory")
	public MybatisSqlSessionFactoryBean setSqlSessionFactory(DataSource dataSource) {
		MybatisSqlSessionFactoryBean bean = new MybatisSqlSessionFactoryBean();
		// 设置数据源
		bean.setDataSource(dataSource);
		// 简化PO的引用
		bean.setTypeAliasesPackage("com.xhiteam.usercenter.serve.model.po");
		// 设置全局配置
		bean.setGlobalConfig(this.globalConfig());
		return bean;
	}

	/**
	 * 设置全局配置
	 *
	 * @return 全局配置
	 */
	public GlobalConfig globalConfig() {
		GlobalConfig globalConfig = new GlobalConfig();
		GlobalConfig.DbConfig dbConfig = new GlobalConfig.DbConfig();
		// 设置逻辑删除字段
		dbConfig.setLogicDeleteField("deleted");
		// 设置更新策略
		dbConfig.setUpdateStrategy(FieldStrategy.NOT_EMPTY);
		globalConfig.setDbConfig(dbConfig);
		return globalConfig;
	}

}
```

