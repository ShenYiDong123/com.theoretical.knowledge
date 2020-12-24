### 版本号 ###
version=1.0-20201224-001

### 基本介绍 ###
总结原理


### Spring相关 ###

**  Spring Bean的生命周期 **
1. 实例化一个Bean，也就是我们常说的new。 
IOC
2. 按照Spring上下文对实例化的Bean进行配置，也就是IOC注入。 
setBeanName
 3. 如果这个Bean已经实现了BeanNameAware接口，会调用它实现的setBeanName(String) 方法，此处传递的就是Spring配置文件中Bean的id值 
BeanFactoryAware
 4. 如果这个 Bean 已经实现了 BeanFactoryAware 接口，会调用它实现的 setBeanFactory， setBeanFactory(BeanFactory)传递的是Spring工厂自身（可以用这个方式来获取其它Bean， 只需在Spring配置文件中配置一个普通的Bean就可以）。 
ApplicationContextAware
 5. 如果这个Bean已经实现了ApplicationContextAware接口，会调用 setApplicationContext(ApplicationContext)方法，传入Spring上下文（同样这个方式也 可以实现步骤4的内容，但比4更好，因为ApplicationContext是BeanFactory的子接 口，有更多的实现方法） 
postProcessBeforeInitialization
 6. 如果这个Bean关联了BeanPostProcessor接口，将会调用 postProcessBeforeInitialization(Object obj, String s)方法，BeanPostProcessor经常被用 作是Bean内容的更改，并且由于这个是在Bean初始化结束时调用那个的方法，也可以被应 用于内存或缓存技术。 
init-method
 7. 如果Bean在Spring配置文件中配置了init-method属性会自动调用其配置的初始化方法。 
postProcessAfterInitialization 
8. 如果这个Bean关联了BeanPostProcessor接口，将会调用 postProcessAfterInitialization(Object obj, String s)方法。 注：以上工作完成以后就可以应用这个 Bean 了，那这个 Bean 是一个 Singleton 的，所以一 般情况下我们调用同一个id 的Bean会是在内容地址相同的实例，当然在Spring配置文件中 也可以配置非Singleton。 
Destroy
 9. 当 Bean 不再需要时，会经过清理阶段，如果 Bean 实现了 DisposableBean 这个接口，会调 用那个其实现的destroy()方法； 
destroy-method
 10. 后，如果这个 Bean 的 Spring 配置中配置了 destroy-method 属性，会自动调用其配置的 销毁方法。 

** Spring基于xml注入bean的几种方式 **
set方法注入
构造器方法注入
静态方法
实例方法

** @Autowired和@Resource之间的区别 **
`@Autowired`默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在（可以设置它required属性为false）
`@Resource`默认是按照名称来装配注入的，只有当找不到与名称匹配的bean才会按照类型来装配注入

** Spring事务的实现方式和实现原理 **


**  spring事务支持声明式事务和编程式事务 **
编程式事务使用的是transationTemplate beginTransaction()、commit()、rollback()
声明式事务建立在AOP上
声明式事务最大的优势是不用在业务代码中掺杂其他的代码，只需添加注解，使业务代码不受污染
配置文件配置事务管理器org.springframework.jdbc.datasource.DataSourceTransactionManager



**  **


**  **


**  **


**  **


**  **


**  **


**  **


**  **


**  **


**  **


**  **


