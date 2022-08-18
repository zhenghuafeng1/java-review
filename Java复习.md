# Java复习

# spring

## spring框架

spring核心是IOC（控制反转）、AOP（面向切面编程）、DI（依赖注入）

- IOC控制反转是指创建对象的控制权的转移，由编程人员主动把控，转移到spring容器中，并且由容器根据配置文件去创建实例和管理各个实例之间的依赖关系。最直观的表达就是，IOC让对象的创建不用去new，可以由spring根据我们提供的配置文件自动生成，我们直接从spring容器中获取即可。

- DI依赖注入指应用程序运行时依赖ioc容器来动态注入对象需要的外部资源。<mark>具体怎么进行依赖？</mark>

- AOP用于将与业务无关的，但却对多个对象产生影响的公共行为和逻辑，抽取并封装为一个可重用的模块。 SpringAOP使用的动态代理方式，所谓动态代理就是说AOP框架不会去修改字节码，而是每次运行时在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理。

### 如何解决循环依赖

使用三级缓存

- 一级缓存：singletonObjects，也叫单例池，存放已经经历了完整生命周期的Bean对象。

- 二级缓存：earlySingletonObjects，存放早起暴露出来的Bean对象，Bean的生命周期未结束（属性还未填充完整）。

- 三级缓存：singletonFactories，Map<Stirng,ObjectFactory<?>> ，存放可以生产Bean的工厂。

### 动态代理

1. jdk动态代理只提供接口代理，不支持类代理，核心InvocationHandler接口和Proxy类，InvocationHandler通过invoke()方法反射来调用目标类中的代码，动态地将横切逻辑和业务编织在一起，Proxy利用InvocationHandler动态创建一个符合某一接口的实例，生成目标类的代理对象。

2. CGLIB是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中特定方法并添加增强代码，从而实现AOP。CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么就无法使用CGLIB做动态代理。

当被代理类有实现接口的情况下，那么使用的是jdk动态代理，否则使用的是CGLIB动态代理

#### jdk动态代理和cglib动态区别

- jdk动态代理是实现了呗代理对象的接口，cglib是继承了呗代理对象。

- jdk和cglib都是在运行时期生成字节码，jdk是直接写class字节码，cglib使用asm框架写class。

- jdk调用代理方法是通过反射机制调用，cglib是通过FastClass机制直接调用方法。

## spring的生命周期

1. 实例化一个Bean，也就是我们通常说的new

2. 按照spring上下文对实例化的Bean进行配置，也就是IOC注入。

3. 如果这个Bean实现了BeanNameAware接口，会调用它实现的setBeanName方法，传递的是spring配置文件中Bean的ID。

4. 如果这个Bean实现了BeanFactoryAware接口，会调用它实现的setBeanFactory()，传递的是spring工厂本身（可以用这个方法获取到其他Bean）

5. 如果这个Bean实现了ApplicationContextAware接口，会调用setApplicationContext方法，传入spring上下文。

6. 如果这个Bean关联了BeanPostProcessor接口，将会调用postProcessBeforeInitialization方法，BeanPostProcessor经常被用作是Bean内容的更改，并且由于这个实在Bean初始化结束时调用After方法，也可用于内存或缓存技术.

7. 如果这个Bean在spring配置文件中配置了init-method属性会自动调用其配置初始化方法。

8. 如果这个Bean关联了BeanPostProcessor接口，将会调用postAfterInitialication方法。

注：以上工作完成后就可以使用这个Bean，这个Bean是一个single的，所以一般情况下我们调用同一个ID的Bean会是在内容地址相同的实例。

   9.当Bean不再需要时，会经历清理阶段，如果Bean实现了DisposableBean接口，会调用其实现的destroy方法。

    10.最后，如果这个Bean的spring配置中配置了destroy0method属性，会自动调用其配置的销毁方法。

### Bean创建周期

大致可以分为5个阶段：创建前准备、创建实例化、依赖注入、容器缓存和销毁实例；

一、创建前准备

![](C:\Users\admin\AppData\Roaming\marktext\images\2022-08-11-15-01-09-image.png)

> 实例化BeanFactoryPostProcessor
> 
> <mark>BeanFactoryPostProcessor会比Bean先实例化</mark>
> 
>                             |
> 
> 实例化BeanPostProcessor
> 
>                             |
> 
> 实例化BeanPostProcessor实现类
> 
>                             |
> 
> 实例化InstantiationAwareBeanPostProcessorAdapter实现类（实例化感知的Bwan后置处理器）
> 
>                             |
> 
> 执行InstantiationAwareBeanPostProcessor的postProcessBeforeInstantiation()方法（在实例化之前的后置处理）
> 
> <mark>在目标Bean被实例化之前先执行此处理器的逻辑</mark>

这个阶段的主要作用是bean在开始加载前，要在上下文和配置中去解析和查找bean有关的扩展实现比如init-method、destroy-method、BeanfactoryPostProcessor等一些bean在加载过程中的前置和后置的一些扩展实现。

二、创建实例阶段

![](C:\Users\admin\AppData\Roaming\marktext\images\2022-08-11-15-05-58-image.png)

> 执行Bean的构造器
> 
> <mark>正式创建Bean</mark>
> 
>             |
> 
> 执行InstantiationAwareBeanPostProcessor的postProcessPropertyValues()方法
> 
> <mark>当依赖注入发生前，Java对象以及创建，但是SpringBean还不对外可用</mark>

这个阶段的主要作用是通过反射去创建bean的实例对象，并且会扫描和解析bean声明的一些属性。

三、依赖注入阶段

![](C:\Users\admin\AppData\Roaming\marktext\images\2022-08-11-15-07-30-image.png)

> 为Bean注入属性
> 
>             
> 
> 调用BeanNameAware的setBeanName()方法
> 
>                                    |  ——————> <mark>先设置Bean的name再创建Bean的Factory</mark>
> 
> 调用BeanFactoryAware的setBeanFactory()方法
> 
>                                    |
> 
> 执行BeanPostProcessor的PostProcessBeforeInitialization()方法（后置处理器再实例化之前的方法）
> 
> **所有初始化动作（如afterPropertiesSet、init-method等）回调之前，都将把BeanPostProcessor赋值给新创建的Bean**
> 
>                                    |
> 
> 调用InitializingBean的afterPropertieSet()方法
> 
> **设置好所有的Bean属性之后，由BeanFactory调用**

如果被实例化的bean存在依赖其他bean对象的情况，则需要对这些依赖的bean进行对象注入，在这个阶段会触发一些扩展调用，比如BeanPostProcessors：用来实现bean初始化前后的扩展回调，以及BeanFactoryAware等等。

四、容器缓存阶段

![](C:\Users\admin\AppData\Roaming\marktext\images\2022-08-11-15-21-05-image.png)

> 调用Bean的init-method方法（在配置中属性指定的方法）
> 
>                                    |
> 
> 执行BeanPostProcessor的postProcessAfterInitialization()方法（Bean后置处理器再实例化之后执行）
> 
>                                    |
> 
> 执行InstantiationAwareBeanPostProcessor的postProcessAfterInitialization()方法（初始化感知后置处理器再实例化之后执行）

容器缓存阶段的主要作用就是把bean保存到容器以及spring的缓存中，到了这个阶段的bean就可以被开发者使用，这个阶段涉及到的操作，常见的像：init-method这个属性配置的一些方法会在这个阶段被调用以及BeanPostProcessors的后置处理方法。

五、销毁实例阶段

![](C:\Users\admin\AppData\Roaming\marktext\images\2022-08-11-15-24-29-image.png)

> 调用DiposibleBean的destory()方法
> 
> <mark>被BeanFactory调用，最终销毁单例对象</mark>
> 
>                         |
> 
> 调用Bean的destory-method方法（在配置中属性指定的方法）

当spring的应用上下文被关闭的时候，这个上下文中的所有的bean会被销毁，如果bean实现了DisposableBean接口或者配置了destory-method属性会在这个阶段被调用。

## spring支持的作用域

1. singleton：默认作用域，单例bean，每个容器中只有一个bean的实例。

2. prototype：每次请求都会为bean创建实例。

3. request：为每一个request请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。

4. session：与request范围类似，同一个session会话共享一个实例，不同会话使用不同的实例。

5. global-session：全局作用域，所有会话共享一个实例。如果想要声明让所有会话共享的存储变量的话，那么这全局变量需要存储在global-session中。

## spring框架中用到的设计模式

1. 工厂模式：BeanFactory就是简单工厂模式的体现，用来创建对象的实例。

2. 单例模式：Bean默认为单例模式。

3. 代理模式：spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术。

4. 模板方法：用来解决代码重复的问题。比如RestTemplate、JmsTemplate、JpaTemplate。

5. 观察者模式：定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动更新，如spring中listener的实现

## spring对象默认是单例的还是多例的？单例bean线程安全问题？

1. spring中对象默认是单例的，但是也可以配置为多例。

2. 单例bean对象对应的类存在可变的成员变量并且其中存在改变这个变量的线程时，多线程操作该bean对象时会出现线程安全问题。
   
   原因：多线程操作如果改变成员变量，其他线程无法访问该bean对象，造成数据混乱。
   
   解决办法：
   
   1.在bean对象中避免定义可变成员变量；
   
   2.在bean对象中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在ThreadLocal中。

## spring事务的实现方式和实现原理

spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。真正的数据库层的事务提交和回滚是通过binLog或者redoLog实现的

<mark>binLog、redoLog是什么？</mark>

spring事务实现主要有两种方法：

1. 编程式，bginTransaction、commit、rollback等事务管理相关的方法。

2. 声明式，利用注解Transactional或者aop配置。

## spring常用注解

1. @Component、@Controller、@Service、@Repository：用于实例化对象

2. @Scope：设置spring对象的作用域

3. @PostConstruct：在spring容器启动时执行该方法、@PreDestroy：用于服务器卸载serviet时运行。

4. @Value：简单属性的依赖注入。

5. @Autowired：对象属性的依赖注入。

6. @Qualifier：要和@Autowired联合使用，代表在按照类型匹配的基础上再按照名称匹配。

7. @Resource：按照属性名称依赖注入。

8. @ComponentScan：组件扫描。

9. @Bean：标在方法上，用于将方法的返回值对象放入容器中。

10. @PropertySource：用于引入其他的properties配置文件。

11. @Import：导入其他类的内容。

12. @Configuration：被标注的类会被spring认为是配置类，在spring启动的时候会自动扫描并加载所有配置类，然后将配置类中bean放入容器。

13. @TransactionalL表示当前类中的方法具有事务管理功能。

## spring事务传播行为（propagation）

例子：有A方法，B方法两个方法，A调用B

- REQUIRED（默认事务传播类型）：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务。

- SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。如果A中有事务，则B方法的事务加入A事务中，成为一个事务（一起成功，一起失败），如果A中没有事务，那么B就以非事务方式运行（执行完直接提交）

- MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。如果A中有事务，则B方法的事务加入A事务中，成为一个事务（一起成功，一起失败）；如果A中没有事务，B中有事务，那么B就直接抛异常了，意思是B必须要支持回滚的事务中运行。

- REQUIRES_NEW：创建新事务，无论当前存不存在事务，都创建新事务。B会新建一个事务，A和B事务互不干扰，他们出现问题回滚的时候，也都只回滚自己的事务。

- NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。被调用者B会以非事务方式运行（直接提交），如果当前有事务，也就是A中有事务，A会被挂起（不执行，等待B执行完，返回）；A和B出现异常需要回滚，互不影响。

- NEVER：以非事务方式执行，如果当前存在事务，则跑出异常。就是B从不以事务方式运行 ；A中不能有事务，如果没有，B就以非事务方式执行，如果A存在事务，那么直接抛异常。

- NESTED：如此当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。如果A中没有事务，那么B创建一个事务执行，如果A中也有事务，那么B会会把事务嵌套在里面。

## spring隔离级别（ISOLATION）

1. DEFAULT：默认隔离级别，使用数据库默认的事务隔离级别。

2. READ_UNCOMMITTED：读未提交，允许另外一个事务可以看到这个事务未提交的数据。最低的隔离级别，会产生脏读、不可重复读和幻读。

3. READ_COMMITTED：读已提交，保证一个事务修改的数据提交后才能被另一事务读取，而且能看到该事务对已有记录的更新。解决脏读问题。

4. REPEATABLE_READ：可重复读，保证一个事务修改的数据提交后才能被另一事务读取，但是不能看到该事务对已有记录的更新，行锁。解决脏读和不可重复读。

5. SERIALIZABLE：可串行化，一个事务在执行的过程中完全看不到其他事务对数据库所做的更新。表锁。除了防止脏读，不可重复读外，还避免了幻像读。

### 概念说明

- 脏读：脏读就是指当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问 这个数据，然后使用了这个未提交的数据。
  如：有两个事务A和B
  事务A通过where id =1，查询到age = 18，然后事务B同时去update这个id = 1 的数据 把age = 20，但是未提交
  此时事务A再次查询 id= 1时结果为age = 20，两次查询结果不一致。
  那么，这种在一个事务里面，由于其他的时候修改了数据并且没有提交，而导致了前后两次读取数据不一致的情况，这种事务并发的问题，我们把它定义成脏读。

- 不可重复读：是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两 次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的，因此称为是不 可重复读。
  如：有两个事务A和B
  事务A通过where id =1，查询到age = 18，然后事务B同时去update这个id = 1 的数据 把age = 20，已提交
  此时事务A再次查询 id= 1时结果为age = 20，两次查询结果不一致。
  这种第一个事务读取到了其他事务已提交的数据导致前后两次读取数据不一致的情况，就像这里，age 到底是等于 16 还是 18，那么这种事务并发带来的问题，我们把它叫做不可重复读。(发生在update操作)。

- 幻读：是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。 同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象 发生了幻觉一样
  如：有两个事务A和B
  事务A通过where age=18，查询到一条数据，然后事务B又新增了一条age = 18的数据
  此时事务A再次查询 age=18时，结果有两条数据，两次查询结果条数不一致。
  那么，一个事务前后两次读取数据数据不一致，是由于其他事务插入数据造成的，这种情况我们把它叫做幻读（发生在insert、delete操作）。

# springMVC

## springMVC主要组件

- 前端控制器 DispatchServlet：接收请求、响应结果，相当于转发器，有了DispatcherServlet就减少了其他组件之间的耦合度。

- 处理器映射器 HandlerMapping：根据请求的URL来查找Handler。

- 处理器适配器 HandlerAdapter：负责执行Handler。

- 处理器Handler：处理业务逻辑的java类。

- 视图解析器 ViewResolver：进行视图的解析，根据视图逻辑名将ModelAndView解析成真正的视图（view）。

- 视图View：View是一个接口，它的实现类支持不同的视图类型，如jsp，freemarker，pdf等等。  

## springMVC的执行流程以及各个组件的作用

1. 用户发送请求到前端控制器（DispatcherServlet）

2. 前端控制器（DispatchServlet）收到请求调用处理器映射器（HandlerMapping），去查找处理器（Handler）。

3. 处理器映射器（HandlerMapping）找到具体的处理器（可以根据xml、注解进行查找），生成处理器对象以及处理拦截器（如果有则生成）一并返回给DispatcherServlet。

4. 前端控制器（DispatcherServlet）调用处理器适配器（HandlerAdapter）去执行Handler。

5. Handler执行完成给适配器返回ModelAndView，ModelAndView是springmvc框架的一个底层对象，包括Model和view。

6. 前端控制器请求视图解析器去进行视图解析，根据逻辑视图名解析成真正的视图(jsp)。

7. 视图解析器向前端控制器返回View。

8. 前端控制器进行视图渲染，视图渲染将模型数据(在ModelAndView对象中)填充到request域。

9. 前端控制器向用户响应结果。

## springMVC统一异常处理

- 方法一：创建一个自定义异常处理器（实现HandlerExceptionResolver接口），并实现里面的异常处理方法，然后将这个类交给spring容器管理。

- 方法二：在类上加注解（@ControllerAdvice）表名这是一个全局异常处理类；在方法上加注解（@ExceptionHandler），在ExceptionHandler中有一个value属性，可以指定可以处理的异常类型。  

# springboot

## springboot启动器starter

1. 什么是starter：starter启动器，可以通过启动器集成其他的技术，比如web，mybatis，redis等等，可以提供对应技术的开发和运行环境。

2. starter执行原理：一、springboot在启动的时候会去扫描jar包中的一个名为spring.factories。二、根据文件中的配置，去加载自动配置类。配置文件格式是key=value，value中配置了很多需要spring加载的类。三、spring会去加载这些自动配置类，spring读取后，就会创建这些类的对象，放到spring容器中，后期就会从spring容器中获取这类对象。

3. springboot中常用的启动器：spring-boot-starter-web，提供web技术支持；spring-boot-starter-test；spring-boot-starter-jdbc；spring-boot-starter-jpa；spring-boot-starter-redis等等。

## springboot自动装配

### 注解

1. @SpringBootConfiguration：其本质是@Configuration，定义该类是一个配置类，功能等同于xml配置文件。@Configuration和@Bean两个注解可以创建一个简单的Spring配置类，可以用来替代相应的xml配置文件，可以理解为创建了IOC容器。

2. @EnableAutoConfiguration：其本质就是@import导入所有符合自动配置条件的bean定义加载到ioc容器。@EnableAutoConfiguration会根据类路径中的jar依赖为项目进行自动配置。

### 如何完成自动配置

主要注解为@EnableAutoConfiguration。

注解内部使用的@Import，导入了AutoConfigurationImportSelector类，这类内部提供了一个方法selectImports，这个方法会扫描导入的所有jar包下的spring.factories文件，解析其中的key=value，将列表中的类创建，并放到Spring容器中。

# mybatis

## ${}和#{}的区别

${}是字符替换，mybatis在处理时，直接替换成变量的值；

#{}是占位符，预编译处理，mybatis在处理时，会将sql中的#{}替换为？号，调用PreparedStatement的set方法来赋值；

## resultType和ResultMap的区别

如果数据库结果集中的列名和要封装实体的属性名完全一致的话用resultType；

如果数据库结果集中的列名和要封装实体的属性名有不一致的情况用resultMap属性，通过resultMap手动建立对象关系映射，resultMap要配置一下表和类的一一对应关系。

## mybatis缓存机制

mybatis有两级缓存，一级缓存是SqlSession级别的，默认开启，无法关闭；二级缓存是Mapper级别的，二级缓存默认是没有开启的但是手动开启。

1. 一级缓存：基础PerpetualCache的HashMap本地缓存，其存储作用域为session，当session flush或close之后，session中的所有cache就将清空。

2. 二级缓存其存储作用域为Mapper，使用二级缓存属性类需要实现serializable序列化接口。

3. 对于缓存数据更新机制，当某一个作用域(一级缓存session/二级缓存namespaces)的进行了crud操作后，默认该作用域下所有select中的缓存将被clear。

# Redis

## redis的存储结构

- String，字符串类型，一个key对应一个value。

- Hash，散列，是一个键值（key=>value）对集合。string类型的field和value的映射表，特别适合用于存储对象。

- List，列表，是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列边或者尾部。

- Set，集合，是string类型的无序集合。

- Sorted set，有序集合，和set一样也是string类型的元素的集合，且不允许重复的成员。不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的，但分数却可以重复。

## redis适合的场景

1. 会话缓存

2. 全页缓存

3. 队列

4. 排行榜/计数器

5. 发布/订阅

## redis的优点

1. 因为是纯内存操作，redis的性能非常出色，每秒可以处理超过10万次读写操作，是已知性能最快的key-value数据库。redis支持事务、持久化。

2. 多线程操作，避免了频繁的上下文切换。

3. 采用了飞阻塞I/O多路复用机制。I/O多路复用就是只有单个线程，通过跟踪每个I/O流的状态，来管理多个I/O流。

# redis的缺点

### 缓存和数据库双写一致性问题

解决方案：

1. 编写删除缓存的接口，在更新数据库的同时，调用删除缓存的接口删除缓存中的数据。这么做会有耦合高以及调用接口失败的情况。

2. 消息队列：activeMq，消息通知

### 缓存的并发竞争问题

解决方案：

1. 最简单的方式就是自卑一个分布式锁，大家去抢锁，抢到锁就做set操作即可。

### 缓存雪崩问题

即缓存同一时间大面积的失效，这个时候又来了一波请求，结果请求都去查询数据库，从而导致数据库连接异常。

解决方案：

1. 给缓存的失效时间加上一个随机值，避免集体失效。

2. 使用互斥锁，但是该方案的吞吐量明显下降了。

3. 搭建redis集群。

4. 设置过期标志更新缓存：当缓存过期则通知线程后台更新

### 缓存击穿

即黑客故意请求缓存中不存在的数据，导致所有的请求都怼到数据库上，从而数据库连接异常。

解决方案：

1. 利用互斥锁，缓存失效的时候就先去获得锁，得到锁了，再去请求数据库，没得到锁则休眠一段时间重试。

2. 采用异步更新策略，无论key是否取到值，都直接返回，value值中维护一个缓存失效时间，缓存如果过期，异步起一个线程去读数据库更新缓存。

## redis的持久化

redis提供两种持久化方法，分别是RDB和AOF

1. RDB：就是在不同的时间点，将redis存储的数据生成快照并存储到磁盘等介质上。redis会单独创建一个子进程（使用fork函数）来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作，这就确保了极高的性能。

2. AOF：将redis执行过的所有写指令记录下来，在下次redis重新启动时，只要把这些写指令从前到后再重复执行一遍，就可以实现数据恢复了。持久化过程：一、客户端请求写命令会被append追加到AOF缓冲区。二、AOF缓冲区根据AOF持久化策略【always，everysec，no】将操作sync同步到磁盘的AOF文件中。三、AOF文件大小超过重写策略或手动重写时，会对AOF文件rewrite重写，压缩AOF文件容量。四、redis服务重启时，会重新load加载AOF文件中的写操作达到数据恢复目的。

## RDB和AOF优缺点

### RDB优点

1. RDB是一个非常紧凑的文件，它保存了redis在某个时间点上的数据集。这种文件非常适合用于进行备份和灾难恢复。

2. 生成RDB文件的时候，redis主进程会fork一个子进程来处理所有保存工作，主进程不需要进行任何磁盘IO操作。

3. RDB在恢复大数据集时的速度比AOF的恢复速度要快。

### RDB缺点

1. RDB方式数据没办法做到实时持久化或秒级持久化因为bgsave每次运行都要执行fork操作创建子进程，属于重量级操作，频繁执行成本过高。

2. RDB文件使用特点二进制格式保存，redis版本演进过程中有多个格式的RDB版本，存在老版本redis服务无法兼容新版RDB格式的问题（版本不兼容）。

3. 在一定间隔时间做一次备份，如果redis意外down掉，就会丢失最后一次快照后的所有修改。

### AOF优点

1. 备份机制更稳健，丢失数据概率更低。

2. 可读的日志文本，考验处理误操作。

### AOF缺点

1. 比起EDB占用更多的磁盘空间。

2. 恢复备份速度要慢。

3. 每次读写都同步的话，有一定的性能压力。

4. 存在个别bug，造成恢复不能。

## redis分布式锁

### 单体应用

对并发操作进行加锁操作，保证对数据的操作具有原子性

- synchronized

- ReentrantLock

### 分布式应用

方案一：使用set命令：set key value [EX seconds] [PX milliseconds] [NX|XX]

> EX seconds ——设置指定的到期时间（以秒为单位）
> 
> PX milliseconds——设置指定的到期时间（以毫秒为单位）
> 
> NX——仅在键不存在时设置键
> 
> XX——只有键存在时才设置

方案二：setnx+exprie

方案三：setnx+value（系统时间+过期时间）

方案四：使用lua脚本

方案五：Redisson开源框架

方案六：RedLock

## redis单线程还是多线程

redis5之前是单线程，指的是工作线程为单线程。工作机制：读IO流、计算（把数据插入内存）、writeIO（返回结果）。

### 理解

一次完整的redis请求事件有多个阶段（客户端到服务器的网络连接--->redis读写事件发生--->redis服务端的数据处理（单线程）--->数据返回）。平时所说的redis单线程模型，本质上指的是服务端的数据处理阶段，不牵扯网络连接和数据返回。

1. 客户端到服务器的网络连接：客户端和服务器是socket通信方式，socket服务端监听可同时接受多个客户端请求，这里与redis无关，仅仅做网络连接。

2. redis读写事件发生并向服务端发送请求：redis的客户端与服务端通信是基于TCP连接，完成上一个阶段的网络连接，redis客户端开始真正向服务器发起读写事件，此时redis客户端开始向建立的网络流中推送数据，服务端可以理解为给每一个网络连接创建一个线程同时接收客户端的请求数据。

3. redis服务端的数据处理：服务端完成第二阶段的数据接收，接下来开始依据接收到的数据做逻辑处理，然后得到处理后的数据。

4. 数据返回

## IO多路复用

# rabbitmq

## 为什么要用rabbitmq

1. 在分布式系统下具备异步，削峰，负载均衡等一系列高级功能。

2. 拥有持久化的机制，进程消息，队列中的信息也可以保存下来。

3. 实现消费者和生产者之间的解耦。

4. 对于高并发场景下，利用消息队列可以使得同步访问变为串行访问，达到一定量的限流，利于数据库的操作。

5. 可以使用消息队列达到异步下单的效果，排队中，后台进行逻辑下单。

## 使用rabbitmq的场景

1. 服务间异步通信

2. 顺序消费

3. 定时任务

4. 请求削峰

## 如何确保消息正确地发送至rabbitmq？如何确保消息接收方消费了消息？

发送方确认模式：

> 将信道设置成confirm模式（发送方确认模式），则所有在信道上发布的消息都会被指派一个唯一的ID。
> 
> 一旦消息被投递到目的队列后，或者消息被写入磁盘后（可持久化消息），信道会发送一个确认给生产者（包含消息唯一ID）。
> 
> 如果rabbitmq发生内部错误从而导致消息丢失，会发送一条nack（notacknowledged，未确认）消息。

接收发确认机制：

> 消费者接收每一条消息后都必须进行确认（消息接收和消息确认是两个不同的操作）。只有消费者确认了消息，rabbitmq才能安全地从队列中删除。
> 
> 这里并没有用到超时机制，rabbitmq仅通过Consumer的连接终端来确认是否需要重新发送消息。也就是说，只要连接不中断，rabbitmq给了Consumer足够长的时间来处理消息。保证数据的最终一致性。
> 
> 下面罗列几种特殊情况：
> 
> 1. 如果消费接收到消息，在确认之前就断开了连接或取消订阅，rabbitmq会认为消息没有被分发，然后重新分发给下一个订阅的消费者。（可能存在消息重复消费的隐患，需要去重）。
> 
> 2. 如果消费者接收到消息却没有确认消息，连接也未断开，则rabbitmq认为该消费者繁忙，将不会给该消费者分发更多的消息。

## 如何避免消息重复投递或者重复消费

在消息生成时，mq内部针对每条生产者发送的消息生成一个inner-msg-id。作为去重的依据（消息投递失败并重传），避免重复的消息进入队列；在消息消费时，要求消息体重必须要有一个bizId（对于统一业务全局唯一，如支付ID、订单ID、帖子ID等）作为去重的依据，避免同一条消息被重复消费。

## 消息基于什么传输

由于TCP连接的创建和销毁开销较大，且并发数受系统资源限制，会造成性能瓶颈。rabbitmq使用信道的方式来传输数据。信道是建立在真实的TCP连接内的虚拟连接，且每条TCP连接上的信道数量没有限制。

## 消息如何分发

若该队列至少有一个消费者订阅，消息将以循环的方式发送给消费者。每条消息只会分发给一个订阅的消费者（前提是消费者能够正常处理消息并进行确认）。通过路由可实现多消费的功能。

## 消息怎么路由

消息提供方--->路由--->一至多个队列消息发布到交换器时，消息将拥有一个路由键，在消息创建时设定。通过队列路由键，可以吧队列绑定到交换器上。消息到达交换器后，rabbitmq会将消息的路由键与队列的路由键进行匹配（针对不同的交换器有不同的路由规则）；

常用的交换器主要分为以下三种：

> - fanout：如果交换器收到消息，将会广播到所有绑定的队列上
> 
> - direct：如果路由键完全匹配，消息就被投递到相应的队列
> 
> - topic：可以使来自不同源头的消息能够到达同一个队列。使用topic交换器时，可以使用通配符。

## 如何确保消息不丢失

消息持久化，前提是队列必须持久化。

rabbitmq确保持久性消息能从服务器中恢复的方式是，将它们写入磁盘上的一个持久化日志文件，当发布一条持久性消息到持久交换器上时，rabbitmq会在消息提交到日志文件后才发送响应。一旦消费者从持久化队列中消费了一条持久化消息，rabbitmq会在持久化日志中吧这条消息标记为等待垃圾收集。如果持久化消息在背消费之前rabbitmq重启，那么rabbitmq会自动重建交换器和队列（以及绑定），并重新发布持久化日志稳重的消息到合适的队列。

## mq的缺点

1. 系统可用性降低

2. 系统复杂性提高

3. 一致性问题

# mysql

## 存储引擎

MyISAM存储引擎 

> 特点是不支持事务、外键、表锁和全文索引，拥有较高的插入、查询速度。
> 
> 每个MyISAM在磁盘上存储成三个文件。文件名都和表名相同，扩展名分别是.frm（存储表定义）、.MYD（MYData，存储数据）、.MYI（MYIndex，存储索引）。这里特别要注意的是MyISAM不缓存数据文件，值缓存索引文件。

InnoDB存储引擎

> InnoDB是事务型数据库的首选引擎，支持事务安全表（ACID），支持行所和外键，是mysql默认引擎。

两种存储引擎的区别。

1. InnoDB支持事务，MyISAM不支持，这一点是非常之重要。事务是一种高级的处理方式，如在一些列增删改中只要哪个出错还可以回滚还原，而MyISAM就不可以了。

2. MyISAM适合查询以及插入为主的应用，InnoDB适合频繁修改以及涉及到安全性较高的应用。

3. InnoDB支持外键，MyISAM不支持。

4. MySQL 在 5.1 之前版本默认存储引擎是 MyISAM，5.1 之后版本默认存储引擎是 InnoDB。

5. InnoDB不支持FULLTEXT类型的索引。

6. InnoDB中不保存表的行数，如select count(*) from table时，InnoDB需要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的读出保存好的行数即可。注意的是，当count(*)语句包含where条件时MyISAM也需要扫描整个表。

7. 对于自增长的字段，InnoDB中必须包含只有该字段的索引，但是在MyISAM表中可以和其他字段一起建立联合索引。

8. 清空整个表时，InnoDB是一行一行的删除，效率非常慢。MyISAM则会重建表。

9. InnoDB支持行锁（某些情况下还是锁整表，如 update table set a=1 where user like '%lee%'。

## 数据库事务

### 事务特性

原子性（Atomicity）：即不可分割性，事务要么全部被执行，要么就全部不被执行。

一致性（Correspondence）：事务的执行使得数据库从一种正确状态转换成另一种正确状态。

隔离性（Isolation）：在事务正确提交之前，不允许把该事务对数据的如何改变提供给任何其他事务。

持久性（Durability）：事务正确提交后，其结果见永久保存在数据库中，即使事务提交后有了其他故障，事务的处理结果也会得到保存。

### 隔离级别

1、读未提交（read Uncommited）：

在该隔离级别，所有的事务都可以读取到别的事务中为提交的数据，会产生脏读问题。

2、读已提交（read commited）：

这是大多数数据库默认的隔离级别，但不是mysql的默认隔离级别；这个隔离级别满足了简单的隔离要求：一个事务只能看见以及提交事务所做的改变，所以会避免脏读问题；

由于一个事务可以看到别的事务以及提交的数据，于是随之而来产生了不可重复读和虚读等问题。

3、可重复读（Repeatable read）：

这是mysql的默认隔离级别，它确保了一个事务中多个实例在并发读取数据的时候会读取到一样的数据；不过理论上这会导致另一个棘手的问题：幻读（Phantom Read）。指当用户读取某一范围的数据行时，另一个事务又在该范围内插入了新行，当用户在读取该范围的数据行时，会发现有新的”幻影“行，InnoDB和Falcon存储引擎通过多版本并发控制（MVCC）机制解决了该问题。

4、可串行化（serializable）：

事务的最高级别，通过强制事务排序，使之不可能相互冲突，从而解决幻读问题。它是在每个读的数据行上加上共享锁。在这个级别，可能导致大量的超时现象和锁竞争。

## 索引

索引存储在内存中，为服务器存储引擎为了快速找到记录的一种数据结构。索引的主要作用是加快数据查找速度，提高数据库的性能。

## 索引的分类

1. 普通索引：最基本的索引，没有任何限制。

2. 唯一索引：与普通索引类似，不同的就是索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。

3. 主键索引：它是一种特殊的唯一索引，用于唯一标识数据表中的某一条记录，不允许有空值，一般用primary key来约束。

4. 联合索引（符合索引）：多个字段上建立的索引，能够加速符合查询条件的检索。

5. 全文索引：不支持中文全文检索，可以通过扩展MySQL，添加中文全文检索或为中文内容表提供一个对应的英文索引表的方式来支持中文。

## B树



## B+树
