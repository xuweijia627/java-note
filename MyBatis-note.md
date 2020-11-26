### MyBatis的优势
说明：MyBatis是一个半自动的持久层框架，对于开发人员而言，核心sql需要自己编写和优化，sql与java代码分开，功能边界清晰，一个专注业务，一个专注数据库。
* 易上手
* 参数绑定
* 结果集映射
* sql与代码解耦
### MyBatis核心流程三大阶段
* 初始化阶段：读取mybatis-config.xml配置文件，构建SqlSessionFactory，通过SqlSessionFactory创建SqlSession。
* 代理阶段：通过SqlSession获得Mapper接口动态代理，动态代理回调SqlSession中的某一个方法，SqlSession将执行方法转发给Executor。
* 数据处理阶段：Executor基于JDBC访问数据库，Executor通过反射将数据转为POJO返回给SqlSession，最后返回给调用者。

### MyBatis插件
MyBatis采用责任链模式，通过动态代理组织多个插件，通过插件可以改变MyBatis的默认行为（诸如sql重写之类的），MyBatis允许使用插件拦截四大对象：
* Executor：执行增删改查操作
* StatementHandler：处理sql语句预编译
* ParameterHandler：设置预编译参数
* ResultSetHandler：处理结果集

### MyBatis 多级缓存
* 一级缓存：即SqlSession级缓存，不同SqlSession之间互相隔离。创建SqlSession时，会创建其中的Executor，创建Executor会创建 Cache
* 二级缓存：namespace级缓存，在同一个命名空间，不同SqlSession可以共享二级缓存
### 常见问题
* MyBatis的优缺点
* MyBatis工作原理
* MyBatis 有哪些执行器？ 区别是什么？ SimpleExecutor, BatchExecutor，ReuseExecutor
* MyBatis 延迟加载原理
* #{}和${}的区别
