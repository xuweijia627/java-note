### spring容器初始化流程
1. 根据提供的配置文件路径(classpath:applicationContext.xml)给configLocations赋值。2. 执行refresh()方法，销毁原有的容器，重新初始化

### spring bean的生命周期
说明：bean的创建是在容器中通过 BeanDefinition来完成的。将一个对象转换成 BeanDefinition，再放入Map中，但这里将BeanDefinition再放入Map中并不代表Bean被创建了，只是被初始化了，被标准化为BeanDefinition。
第一阶段：标准化，创建BeanDefinition。各种各样的对象要交给spring管理，那么需要一个标准，将这个标准创建好放入Map中。

第二阶段：创建Bean
