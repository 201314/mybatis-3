DefaultSqlSession 地址

关闭了support事物、关闭myatis一级缓存
1. 11162

2. 11258

3. 11331

4. 11403


开启了support事物、关闭myatis一级缓存
1. 10497
2. 10497

3. 14152
4. 14152

事物拦截器
TransactionInterceptor:invokeWithinTransaction

org.springframework.transaction.interceptor.TransactionAspectSupport#createTransactionIfNecessary
org.springframework.transaction.support.AbstractPlatformTransactionManager#getTransaction

开启了support事物、开启myatis一级缓存
1. DefaultSqlSessionFactory 10486 (new) <==> SqlSessionHolder 14246(new)  DefaultSqlSession 14233(new)  

2. DefaultSqlSessionFactory 10486  SqlSessionHolder 14181 DefaultSqlSession 14394  

3. DefaultSqlSessionFactory 10479  SqlSessionHolder 14181 DefaultSqlSession 14180  

4. 无疾而终

有事物：
Creating a new SqlSession
Registering transaction synchronization for SqlSession
Releasing transactional SqlSession
Transaction synchronization committing SqlSession
Transaction synchronization deregistering SqlSession
Transaction synchronization closing SqlSession  -- 必须有事务的按以上

Transaction synchronization resuming SqlSession
Fetched SqlSession
Releasing transactional SqlSession     -- 没有事务的按以上

无事务：
Creating a new SqlSession
SqlSession was not registered for synchronization because synchronization is not active
Closing non transactional SqlSession

# SpringBoot 自动装配 
@Mapper
MybatisAutoConfiguration.AutoConfiguredMapperScannerRegistrar#registerBeanDefinitions
@MapperScan
MapperScannerRegistrar#registerBeanDefinitions

# 注册dao bean
MapperScannerConfigurer#postProcessBeanDefinitionRegistry
ClassPathBeanDefinitionScanner#scan
ClassPathMapperScanner#doScan --> processBeanDefinitions

# 注入SqlSessionFactory
SqlSessionFactoryBean

--> MapperFactoryBean 
--> SqlSessionDaoSupport 
--> SqlSessionTemplate (SqlSessionInterceptor getSqlSession) 

--> SqlSessionUtils#getSqlSession 获取Session
--> DefaultSqlSessionFactory#openSession
--> DefaultSqlSession(SqlSession) 执行SQL ，通过 完整的namespace.xml中SQL的ID 来执行对应SQL
    getMapper ==> Configuration#getMapper ==> MapperRegistry#getMapper ==> MapperProxyFactory#newInstance(SqlSession) 获取MapperProxy 的代理类,实际最终是调用SqlSession

SqlSessionTemplate 线程安全
DefaultSqlSession 线程不安全

#  org.apache.ibatis.annotations
由 MapperAnnotationBuilder 解析Mapper注解

# org.apache.ibatis.binding
绑定 -- 重要

# org.apache.ibatis.builder
XML解析器
BaseBuilder --> XMLConfigBuilder(mybatis.xml 解析)、XMLStatementBuilder(组装 MappedStatement)、XMLMapperBuilder(解析/mapper/节点下parameterMap、resultMap、sql)、XMLScriptBuilder、SqlSourceBuilder、MapperBuilderAssistant

# org.apache.ibatis.cache
二级缓存
Cache --> LruCache（等一些自定义缓存）、PerpetualCache、ScheduledCache、SerializedCache、LoggingCache、SynchronizedCache、BlockingCache
CacheKey: 每个参数的hashCode之和做为checkSum,每个参数的hashCode*multiplier做为总hashCode
最终key格式为 hashcode:checkSum:每个参数:每个参数

# 脚本执行器测试工具
org.apache.ibatis.jdbc

# 数据源
org.apache.ibatis.datasource

PooledDataSource 有连接池的数据源 
UnpooledDataSource 无连接池的数据源
JndiDataSourceFactory 提供JNDI形式的数据源


# org.apache.ibatis.executor 
SQL执行器
Executor         --> SimpleExecutor、BatchExecutor、ReuseExecutor、CachingExecutor --> BaseExecutor
StatementHandler --> (根据RoutingStatementHandler选择)PreparedStatementHandler、SimpleStatementHandler、CallableStatementHandler(存储过程用) --> BaseStatementHandler 
ResultSetHandler --> DefaultResultSetHandler

结果提取器
ResultExtractor --> 将List中内容转换为目标类型

以查询、PreparedStatement为例
BaseExecutor#query --> BaseExecutor#queryFromDatabase --> SimpleExecutor#doQuery --> Configuration#newStatementHandler(所有拦截器进行拦截,关键类org.apache.ibatis.plugin.Plugin#wrap)
--> PreparedStatementHandler#query --> DefaultResultSetHandler#handleResultSets

# org.apache.ibatis.executor.keygen
主键生成器
KeyGenerator --> SelectKeyGenerator、Jdbc3KeyGenerator、NoKeyGenerator

# org.apache.ibatis.executor.loader
代理
ProxyFactory --> JavassistProxyFactory 、 CglibProxyFactory

#org.apache.ibatis.io
VFS --> DefaultVFS、JBoss6VFS 查找代码或配置中使用的Class

# 动态SQL
org.apache.ibatis.jdbc

SQL 脚本执行器: ScriptRunner

# org.apache.ibatis.logging
从try中尝试是否能找到log实现
LogFactory

# org.apache.ibatis.mapping 此处好几个类使用了建造者模式
BoundSql 		 拥有实际的SQL可能带?参数 、RowBounds
MappedStatement  

# org.apache.ibatis.parsing
GenericTokenParser  --> 解析${x} 的核心实现类
PropertyParser的VariableTokenHandler --> 解析${x:11} 如果x不存在，则返回默认值11,需要开启

# org.apache.ibatis.plugin 拦截器
以下中使用
Configuration#newStatementHandler 
Configuration#newResultSetHandler
Configuration#newParameterHandler

# org.apache.ibatis.reflection
ParamNameResolver @Mapper标记的类入参解析
SystemMetaObject
MetaObject

# org.apache.ibatis.reflection.factory 创建mybaits使用过程中需要的对象
# org.apache.ibatis.reflection.wrapper 包装对象，操作对象属性

# org.apache.ibatis.scripting.xmltags
动态script SQL  == DynamicSqlSource
SqlNode --> ChooseSqlNode、ForEachSqlNode、IfSqlNode、SetSqlNode、TrimSqlNode、StaticTextSqlNode、TextSqlNode、VarDeclSqlNode、WhereSqlNode
            MixedSqlNode == 混合SqlNode并生成SQL，当每一小段标签SQL完成时，塞入该Node

# org.apache.ibatis.session
SqlSessionFactoryBuilder#build(Reader reader, String environment, Properties properties)
	XMLConfigBuilder#parser 负责解析XML

# org.apache.ibatis.type
入参和执行结果类型转换















