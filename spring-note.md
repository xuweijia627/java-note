### spring容器初始化流程
1. 根据提供的配置文件路径(classpath:applicationContext.xml)给configLocations赋值。2. 执行refresh()方法，销毁原有的容器，重新初始化

### spring bean的生命周期
说明：bean的创建是在容器中通过 BeanDefinition来完成的。将一个对象转换成 BeanDefinition，再放入Map中，但这里将BeanDefinition再放入Map中并不代表Bean被创建了，只是被初始化了，被标准化为BeanDefinition。
第一阶段：标准化，创建BeanDefinition。各种各样的对象要交给spring管理，那么需要一个标准，将这个标准创建好放入Map中。

第二阶段：创建Bean，放入单例池中(singletonObjects)
### spring AOP
Cglib vs 动态代理
* Cglib底层用了ASM框架，来操作字节码生成新的类
* 动态代理只能对接口进行代理，如果要代理一个普通类，没有接口就没法使用
* 动态代理提供了Proxy类来创建代理，Cglib提供类Enhancer来创建代理
* 都可以用于AOP的实现
### spring 事务
事务说明：逻辑上的一组操作，对于每一个执行单元要么全部成功，要么全部失败。

事务传播行为

* PROPAGATION_REQUIRED       如果外层有事务在外层事务中运行，外层没有事务就新建一个事务
* PROPAGATION_REQUIRES_NEW   无论外层有没有事务都新建一个事务
* PROPAGATION_SUPPORTS       如果外层有事务在外层事务中运行，外层没有事务就以非事务方式运行
* PROPAGATION_NOT_SUPPORTED  无论外层有没有事务都以非事务方式运行
* PROPAGATION_NEVER          不支持事务，如果外层有事务就抛异常
* PROPAGATION_NESTED         当前事务回滚到某一个点，避免所有嵌套事务都回滚，即各自回滚各自的
* PROPAGATION_MANDATORY      必须在事务中运行，如果当前没有事务就抛异常

