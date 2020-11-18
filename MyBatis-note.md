### MyBatis的优势
说明：MyBatis是一个半自动的持久层框架，对于开发人员而言，核心sql需要自己编写和优化，sql与java代码分开，功能边界清晰，一个专注业务，一个专注数据库。
* 易上手
* 参数绑定
* 结果集映射
* sql与代码解耦
### MyBatis核心流程三大阶段
* 初始化阶段：读取mybatis-config.xml配置文件，构建SqlSessionFactory
* 代理阶段：获得SqlSession，通过SqlSession获得Mapper接口对象
* 数据处理阶段：通过SqlSession完成SQL的解析，参数的绑定，SQL的执行，结果集映射。

### MyBatis 源码底层实现原理
