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

### MyBatis 源码底层实现原理
