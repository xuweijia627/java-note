### spring容器初始化流程
1. 根据提供的配置文件路径(classpath:applicationContext.xml)给configLocations赋值。2. 执行refresh()方法，销毁原有的容器，重新初始化

### spring bean的生命周期
说明：bean的创建是在容器中通过 BeanDefinition来完成的
