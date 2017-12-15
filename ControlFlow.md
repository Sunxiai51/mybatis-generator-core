# MBG控制流

本文档大致描述了MBG工作的控制流，为学习MBG源码提供一个指引与参考。

version: MyBatis Generator Core 1.3.5

考虑到叙述的简洁性，文档中省略了一些内容，例如xml解析过程、额外包的引入与对应classLoader的初始化、callback方法的扩展定义与调用等，这些内容不影响理解主要流程，有兴趣的读者可以自行学习，欢迎补充。

## ShellRunner - 程序入口

该类有一个main方法，能够通过命令行调用MBG来生成文件。
1. 参数校验
2. 解析传入参数与配置文件，生成Configuration *(调用了`ConfigurationParser`)*
3. 初始化`MyBatisGenerator`，调用其generate方法创建需要的文件

### ConfigurationParser - 配置解析器

该类的parseConfiguration方法可以根据参数和配置文件生成Configuration对象，该对象用于后续的创建过程。
1. 通过DocumentBuilder将配置文件解析为Document
2. 根据DocumentType调用不同的方法，返回Configuration对象：
	- 如果是IBATOR_CONFIG，调用parseIbatorConfiguration *(调用了IbatorConfigurationParser)*
	- 如果是MYBATIS_GENERATOR_CONFIG，调用parseMyBatisGeneratorConfiguration *(调用了MyBatisGeneratorConfigurationParser，该类的parseConfiguration方法进行Configuration中属性和Document中配置的映射)*

### MyBatisGenerator - MBG核心类

该类的generate方法是MBG核心功能。
1. 初始化工作，清空一些辅助集合，根据Configuration获取需要生成的`Context`，将所配置的classPathEntry加入externalClassLoader等等
2. database introspection *(调用了每个context的introspectTables方法)*
3. code generation *(调用了每个context的generateFiles方法)*
4. merging/saving generated files

#### Context - 上下文

是MBG里的配置单元，通常一个context对应一个数据源。

该类源码中有如下注释：
```java
// Methods should be called in this order:
//
// 1. getIntrospectionSteps()
// 2. introspectTables()
// 3. getGenerationSteps()
// 4. generateFiles()
//
```

- 该类的introspectTables方法进行该数据源的database introspection
	1. 获取JavaTypeResolver
	2. 逐个扫描配置文件中的TableConfiguration:
		1. 检索传入参数，如果该table本次不需要创建，跳过
		2. 如果该table没有配置任何需要创建的内容，添加Warning，跳过
		3. 创建该tableConfiguration对应的IntrospectedTable列表 *(调用了`DatabaseIntrospector`的introspectTables方法)*
		4. 将列表添加至context的introspectedTables

- 该类的generateFiles方法进行该数据源的code generation:
	1. 初始化PluginAggregator
	2. TODO...

##### DatabaseIntrospector
