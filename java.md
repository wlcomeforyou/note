1. ArrayList和LinkedList的区别

2. HashMap的put流程

3. JVM哪些是共享去，哪些可以作为gc root

4. spring容器的创建流程
  a.首先会扫描所有的beanDefinition对象，并放入全局的map中
  b.然后筛选出非懒加载的单例BeanDefinition对象进行创建，多例对象是在bean使用通过BeanDefinition创建
  c.创建Bean包括了合并BeanDefinition、推断构造方法、实例化、属性填充、初始化前、初始化、初始化后等步骤，AOP在初始化后

5. spring的事务机制
  对于使用了@Transactional注解的Bean,spring会创建一个代理对象，当调用该代理对象方法时，会判断该方法上是否有@Transactional注解，但如果有则使用事务管理器创建一个数据库连接，并将该数据库连接的autocmmmit属性设置为false，当该方法运行成功无异常时提交事务，若出现异常并且是需要回滚的异常时会进行回滚操作，否则一样会进行事务提交

6. @Transactional失效
  a. 注解加在了非public方法上(Spring AOP 代理源码中判断方法是否是public修饰，不是则无法获取注解的配置信息)
  b. 注解使用失效
  c. 异常被catch，没有抛出
  d. 非注解方法调用注解方法
