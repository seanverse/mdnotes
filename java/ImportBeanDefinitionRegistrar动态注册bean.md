
### 简介
- 1、ImportBeanDefinitionRegistrar类只能通过其他类@Import的方式来加载，通常是启动类或配置类。  
- 2、使用@Import，如果括号中的类是ImportBeanDefinitionRegistrar的实现类，则会调用接口方法，将其中要注册的类注册成bean。实现该接口的类拥有注册bean的能力。

### 手动把一个类注册成bean
1. 首先写一个类，最终要把它注册为bean。
```java
  public class HelloService {

  }
```
2. 自定义ImportBeanDefinitionRegistrar实现类手动注册bean。
```java
  public class HelloImportBeanDefinitionRegistrar 
          implements ImportBeanDefinitionRegistrar {

      /**   
       * @Description AnnotationMetadata:当前类的注解信息；
       * BeanDefinitionRegistry：注册类，其registerBeanDefinition()可以注册bean
       * @Date 2019/6/12
       * @Param [importingClassMetadata, registry]
       * @return void
       **/
      @Override
      public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata,
                                          BeanDefinitionRegistry registry) {
  
          //扫描注解
          Map<String, Object> annotationAttributes = importingClassMetadata
              .getAnnotationAttributes(ComponentScan.class.getName());
          String[] basePackages = (String[]) annotationAttributes.get("basePackages");
  
          //扫描类
          ClassPathBeanDefinitionScanner scanner =
                  new ClassPathBeanDefinitionScanner(registry, false);
          TypeFilter helloServiceFilter = new AssignableTypeFilter(HelloService.class);
          
          scanner.addIncludeFilter(helloServiceFilter);
          scanner.scan(basePackages);
      }
  
  }
  ```

3. 最后定义一个配置类发现一下上面的ImportBeanDefinitionRegistrar实现类。
```java
  @Configuration
  @ComponentScan("com.haien.import2.domain")
  @Import(HelloImportBeanDefinitionRegistrar.class)
  public class HelloConfiguration {
  
  }
  //测试：

  @RunWith(SpringRunner.class)
  @SpringBootTest
  @ContextConfiguration(classes = {HelloConfiguration.class}) //表示只需要这一个文件
  public class DemoApplicationTest2 {
  
      @Resource
      HelloService helloService;
  
      /**
       * @Description 扫描到helloService
       * @Date 2019/6/11
       * @Param []
       * @return void
       **/
      @Test
      public void contextLoads(){
          System.out.println(helloService.getClass());
      }
  }
```
### 自己写一个注解实现@Component的功能
1. 目标：自己写一个注解@Mapper，配合其他类，实现被注解类可以被注册成bean的功能。
```java
@Mapper：

  @Documented
  @Inherited
  @Retention(RetentionPolicy.RUNTIME)
  @Target({ElementType.TYPE,ElementType.FIELD,ElementType.METHOD,ElementType.PARAMETER})
  public @interface Mapper {
  }
```
2. 注释于类上：
```java
  @Mapper
  public class CountryMapper {
  }
```
3. 实现ImportBeanDefinitionRegistrar接口，重写registerBeanDefinitions方法，手动注册bean；同时实现一些Aware接口，以便获取Spring的一些数据。
```java
  public class MapperAutoConfiguredMyBatisRegistrar implements
          ImportBeanDefinitionRegistrar,ResourceLoaderAware,BeanFactoryAware {
      
      private ResourceLoader resourceLoader;
      private BeanFactory beanFactory;
  
      /**
       * @Description 注册bean，但我们并不知道需要注册哪些bean，所以需要借助
       * ClassPathBeanDefinitionScanner扫描器，扫描我们需要注册的bean
       * @Date 2019/6/11
       * @Param [annotationMetadata, beanDefinitionRegistry]
       * @return void
       **/
      @Override
      public void registerBeanDefinitions(AnnotationMetadata annotationMetadata,
                                          BeanDefinitionRegistry registry) {
          
          MyClasssPathBeanDefinitionScanner scanner=
                  new MyClasssPathBeanDefinitionScanner(registry,false);
          scanner.setResourceLoader(resourceLoader);
          scanner.registerFilters();
          scanner.doScan("com.haien.import1.domain");
      }
  
      @Override
      public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
          this.beanFactory=beanFactory;
      }
  
      @Override
      public void setResourceLoader(ResourceLoader resourceLoader) {
          this.resourceLoader=resourceLoader;
      }
  
  }
```
4. 以上我们还借助了扫描器ClassPathBeanDefinitionScanner，通过它来获取我们需要注册的bean。
```java
  public class MyClasssPathBeanDefinitionScanner extends ClassPathBeanDefinitionScanner {

      public MyClasssPathBeanDefinitionScanner(BeanDefinitionRegistry registry,
                                               boolean useDefaultFilters) {
          super(registry, useDefaultFilters);
      }
  
      /**
       * @Description 注册条件过滤器，将@Mapper注解加入允许扫描的过滤器中，
       * 如果加入排除扫描的过滤器集合excludeFilter中，则不会扫描
       * @Date 2019/6/11
       * @Param []
       * @return void
       **/
      protected void registerFilters(){
          addIncludeFilter(new AnnotationTypeFilter(Mapper.class));
      }
  
      @Override
      protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
          return super.doScan(basePackages);
      }
  }

  //测试：

  @RunWith(SpringRunner.class)
  @SpringBootTest(classes = MapperAutoConfig.class)
  //只将MapperAutoConfig类纳入测试环境的Spring容器中，
  //或@ContextConfiguration(classes = {MapperAutoConfig.class})
  public class DemoApplicationTest {
  
      @Resource
      CountryMapper countryMapper;
  
      /**
       * @Description 扫描不到CountryMapper
       * @Date 2019/6/11
       * @Param []
       * @return void
       **/
      @Test
      public void contextLoads(){
          System.out.println(countryMapper.getClass());
      }
  }
```