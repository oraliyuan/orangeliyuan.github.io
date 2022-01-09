### 生命周期





bean 生命周期图

![image-20211227222931997](/Users/oraliyuan/Library/Application Support/typora-user-images/image-20211227222931997.png)



bean 创建的大致生命周期

Xxx.class→无参构造（推断构造方法）→普通对象→依赖注入（属性赋值）→初始化前（@PostConstruct）→初始化（InitializingBean）→初始化后（AOP）→代理对象→给代理对象target属性赋值为之前生成好的普通对象→Bean 







bean 扫描流程

![image-20211227223147428](/Users/oraliyuan/Library/Application Support/typora-user-images/image-20211227223147428.png)









扩展点：



BeanFactoryPostProcessor 扩展点

可以向beanfactory 添加beanDefinition

BeanDefinitionRegistryPostProcessor.postProcessBeanDefinitionRegistry()

可以拿到bean factory 并对其中的beanDefinition 做修改

BeanFactoryPostProcessor.postProcessBeanFactory()



实例化前（可能直接返回一个对象，而跳过Spring 的实例化）

InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation()



推断构造方法（）

SmartInstantiationAwareBeanPostProcessor.determineCandidateConstructors()



实例化后，处理合并后的BeanDefinition (这里对BeanDefinition做出修改会影响Bean 后续初始化的行为，比如设置properties，就会替代依赖注入？)

MergedBeanDefinitionPostProcessor.postProcessMergedBeanDefinition()



实例化之后，属性设置之前
InstantiationAwareBeanPostProcessor.postProcessAfterInstantiation



依赖注入

InstantiationAwareBeanPostProcessor.postProcessProperties()

InstantiationAwareBeanPostProcessor实现类AutowiredAnnotationBeanPostProcessor.postProcessProperties()  实现了@Autowired 注解



BeanNameAware
BeanClassLoaderAware
BeanFactoryAware



初始化前
BeanPostProcessor.postProcessBeforeInitialization()


初始化
InitializingBean.afterPropertiesSet()

初始化后
BeanPostProcessor.postProcessAfterInitialization() 



所有的非懒加载单例Bean都创建完了后调用

SmartInitializingSingleton.afterSingletonsInstantiated()



todo 自己画个图