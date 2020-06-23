## Spring的@Import和@Bean注解应用详解

### *为什么需要@Bean和@Import*
当我们设计一个class，注册给ioc容器来管理对象的生命周期时，会使用@Component, @Repository , @Controller , @Service标识到。但是这样模式下，只能限于自己编写的类。如果依赖到一个第三方库的class，也想交由Ioc机制来管理和使用，由于我们没有源码无法加入@Component等注解声明，就需要@Bean或@Import来把三方class注册(导入)给Ioc容器了。我们也可以在大量的开源库中看到使用，来解决开源库工具类可以用于Spring容器模式中也可以用于其他对象管理模式的使用场景。

### @Bean注解说明
1. @Bean就放在方法上，Spring的@Bean注解用于告诉方法，产生一个Bean对象，然后这个Bean对象交给Spring管理。 
2. @Bean注解的另一个好处就是能够动态获取一个Bean对象，能够根据配置参数在方法的实现给出不同的Bean对象。
3. 标识了@Bean注解的方法，Spring只会调用一次，随后这个Spring将会将这个Bean对象放在自己的IOC容器中。
4. @Bean注解在返回实例的方法上，如果未通过@Bean指定bean的名称，则默认与标注的方法名相同。
5. @Bean注解默认作用域为单例singleton作用域，可通过@Scope(“prototype”)设置为原型作用域。
6. @Bean注解注册bean,同时可以指定初始化和销毁方法，如 @Bean(name="testBean",initMethod="start",destroyMethod="cleanUp")。
7. 示例:
```java
class A{
        @Bean
        public PersonDao getPersonDao(){
            return new PersonDao();
        }
    }
```

### *@Import注解说明*

1. @Import只能用在类上 ，@Import通过快速导入的方式实现把实例加入spring的IOC容器中。
2. @Import通过快速导入的方式实现把实例加入spring的IOC容器中。从上文我们知道尤其适用于导入三方包，它有三种使用方法，因此会比@Bean更加便捷。

3. @Import三种方式
- 第一种：直接填对应的class数组，class数组可以有0到多个。
例如：
```java 
@Import({ 类名.class , 类名.class... })
@Import({Jt808Starter.class, Jt808Config.class, SimpleBeanConfig.class}) 
```

- 第二种：class实现ImportSelector接口并在接口selectImports方法中返回class名称数组(0..n)
```java
//创建Myclass类并实现ImportSelector接口，通过全类名string数组给其返回值
public class MyClass implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        return new String[]{"com.sun.Test.TestDemo1","com.sun.Test.TestDemo2" };
    }
}
//使用时Import MyClass,间接注册了selectImports方法返回的class数组
@Import({MyClass.class})
public class TestDemo {
    //...
}
```
- 第三种：实现接口ImportBeanDefinitionRegistrar方式。
 这种主式类似于第二种ImportSelector用法，相对于第二种可以更灵活自定义Bean的描述信息注册给Ioc容器，如示例：
 ```java
public class MyRegclass2 implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) {
        //指定bean定义信息（包括bean的类型、作用域...）
        RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(TestDemo2.class);
        //注册一个bean指定bean名字（id）
        beanDefinitionRegistry.registerBeanDefinition("TestDemo2",rootBeanDefinition);
    }
}

//使用时Import MyRegclass2,间接注册了selectImports方法返回的class数组
@Import({MyRegclass2.class})
public class TestDemo2 {
    //...
}
 ```
4. @Import注解的三种使用方式总结
- 第一种用法：@Import（{ 要导入的容器中的class } ）：容器会自动注册这个组件，id默认是全类名。
- 第二种用法：ImportSelector：返回需要导入的组件的全类名string数组，SpringBoot很多官方包里用的特别多。 
- 第三种用法：ImportBeanDefinitionRegistrar：手动注册Bean描述信息到容器。


### *补充：Spring的注解分类*

1. 一类注解是用于注册Bean到ioc容器
我们把spring ioc容器当成一个工具箱，通过各类注解来往箱子里装东西，也就是说，此时注解的作就是告诉加载机制，只有标识是"工具"的class才会被装(注册\声明)进去。用于注册Bean的注解： 比如@Component , @Repository , @ Controller , @Service , @Configration这些注解就是用于注册Bean，放进IOC容器中，交给spring ioc容器来管理对象的生命周期。

2. 一类注解是用于标识如何使用Bean
用于使用Bean的注解：比如@Autowired , @Resource，@contionalxxxx等注解，一般为了表示如何查找、匹配的声明。