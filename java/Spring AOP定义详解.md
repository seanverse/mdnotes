# Spring 注解 之 AOP基于@Aspect的AOP配置

>Spring AOP面向切面编程，可以用来配置事务、做日志、权限验证、在用户请求时做一些处理等等。用@Aspect做一个切面，就可以直接实现。

## 1.首先定义一个切面类，加上@Component  @Aspect这两个注解

```java
@Component
@Aspect
public class LogAspect {
    private static final Logger logger = LoggerFactory.getLogger(LogAspect.class);
    private static final ObjectMapper OBJECT_MAPPER = new ObjectMapper();
}
```

## 2.定义切点

```java
private final String POINT_CUT = "execution(public * com.xhx.springboot.controller.*.*(..))";
@Pointcut(POINT_CUT)
public void pointCut(){}
```

>切点表达式中，..两个点表明多个，* 代表一个。上面表达式代表切入com.xhx.springboot.controller包下的所有类的所有方法，方法参数不限，返回类型不限。 其中访问修饰符可以不写，不能用*，第一个 * 代表返回类型不限，第二个 * 表示所有类，第三个*表示所有方法，.. 两个点表示方法里的参数不限。然后用@Pointcut切点注解，想在一个空方法上面，一会儿在Advice通知中，直接调用这个空方法就行了，也可以把切点表达式卸载Advice通知中的，单独定义出来主要是为了好管理。

 

## 3.Advice，通知增强，主要包括五个注解Before,After,AfterReturning,AfterThrowing,Around，下面代码中关键地方都有注释，我都列出来了。

   - @Before  在切点方法之前执行
   - @After  在切点方法之后执行
   - @AfterReturning 切点方法返回后执行
   - @AfterThrowing 切点方法抛异常执行
   - @Around 属于环绕增强，能控制切点执行前，执行后，，用这个注解后，程序抛异常，会影响@AfterThrowing这个注解
    
```java
package com.xhx.springboot.config;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.*;
import org.aspectj.lang.reflect.SourceLocation;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.util.Arrays;

@Component
@Aspect
public class LogAspect {

    private static final Logger logger = LoggerFactory.getLogger(LogAspect.class);
    private static final ObjectMapper OBJECT_MAPPER = new ObjectMapper();

    private final String POINT_CUT = "execution(public * com.xhx.springboot.controller.*.*(..))";

    @Pointcut(POINT_CUT)
    public void pointCut(){}

    @Before(value = "pointCut()")
    public void before(JoinPoint joinPoint){
        logger.info("@Before通知执行");
        //获取目标方法参数信息
        Object[] args = joinPoint.getArgs();
        Arrays.stream(args).forEach(arg->{ // 大大
            try {
                logger.info(OBJECT_MAPPER.writeValueAsString(arg));
            } catch (JsonProcessingException e) {
            logger.info(arg.toString());
            }
        });

        //aop代理对象
        Object aThis = joinPoint.getThis();
        logger.info(aThis.toString()); //com.xhx.springboot.controller.HelloController@69fbbcdd

        //被代理对象
        Object target = joinPoint.getTarget();
        logger.info(target.toString()); //com.xhx.springboot.controller.HelloController@69fbbcdd

        //获取连接点的方法签名对象
        Signature signature = joinPoint.getSignature();
        logger.info(signature.toLongString()); //public java.lang.String com.xhx.springboot.controller.HelloController.getName(java.lang.String)
        logger.info(signature.toShortString()); //HelloController.getName(..)
        logger.info(signature.toString()); //String com.xhx.springboot.controller.HelloController.getName(String)
        //获取方法名
        logger.info(signature.getName()); //getName
        //获取声明类型名
        logger.info(signature.getDeclaringTypeName()); //com.xhx.springboot.controller.HelloController
        //获取声明类型 方法所在类的class对象
        logger.info(signature.getDeclaringType().toString()); //class com.xhx.springboot.controller.HelloController
        //和getDeclaringTypeName()一样
        logger.info(signature.getDeclaringType().getName());//com.xhx.springboot.controller.HelloController

        //连接点类型
        String kind = joinPoint.getKind();
        logger.info(kind);//method-execution

        //返回连接点方法所在类文件中的位置 打印报异常
        SourceLocation sourceLocation = joinPoint.getSourceLocation();
        logger.info(sourceLocation.toString());
        //logger.info(sourceLocation.getFileName());
        //logger.info(sourceLocation.getLine()+"");
        //logger.info(sourceLocation.getWithinType().toString()); //class com.xhx.springboot.controller.HelloController

        ///返回连接点静态部分
        JoinPoint.StaticPart staticPart = joinPoint.getStaticPart();
        logger.info(staticPart.toLongString()); //execution(public java.lang.String com.xhx.springboot.controller.HelloController.getName(java.lang.String))


        //attributes可以获取request信息 session信息等
        ServletRequestAttributes attributes =
        (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        logger.info(request.getRequestURL().toString()); //http://127.0.0.1:8080/hello/getName
        logger.info(request.getRemoteAddr()); //127.0.0.1
        logger.info(request.getMethod()); //GET

        logger.info("before通知执行结束");
    }


    /**
    * 后置返回
    * 如果第一个参数为JoinPoint，则第二个参数为返回值的信息
    * 如果第一个参数不为JoinPoint，则第一个参数为returning中对应的参数
    * returning：限定了只有目标方法返回值与通知方法参数类型匹配时才能执行后置返回通知，否则不执行，
    * 参数为Object类型将匹配任何目标返回值
    */
    @AfterReturning(value = POINT_CUT,returning = "result")
    public void doAfterReturningAdvice1(JoinPoint joinPoint,Object result){
        logger.info("第一个后置返回通知的返回值："+result);
    }

    @AfterReturning(value = POINT_CUT,returning = "result",argNames = "result")
    public void doAfterReturningAdvice2(String result){
        logger.info("第二个后置返回通知的返回值："+result);
    }
    
    //第一个后置返回通知的返回值：姓名是大大
    //第二个后置返回通知的返回值：姓名是大大
    //第一个后置返回通知的返回值：{name=小小, id=1}


    /**
    * 后置异常通知
    * 定义一个名字，该名字用于匹配通知实现方法的一个参数名，当目标方法抛出异常返回后，将把目标方法抛出的异常传给通知方法；
    * throwing:限定了只有目标方法抛出的异常与通知方法相应参数异常类型时才能执行后置异常通知，否则不执行，
    * 对于throwing对应的通知方法参数为Throwable类型将匹配任何异常。
    * @param joinPoint
    * @param exception
    */
    @AfterThrowing(value = POINT_CUT,throwing = "exception")
    public void doAfterThrowingAdvice(JoinPoint joinPoint,Throwable exception){
        logger.info(joinPoint.getSignature().getName());
        if(exception instanceof NullPointerException){
            logger.info("发生了空指针异常!!!!!");
        }
    }

    @After(value = POINT_CUT)
    public void doAfterAdvice(JoinPoint joinPoint){
        logger.info("后置通知执行了!");
    }

    /**
    * 环绕通知：
    * 注意:Spring AOP的环绕通知会影响到AfterThrowing通知的运行,不要同时使用
    *
    * 环绕通知非常强大，可以决定目标方法是否执行，什么时候执行，执行时是否需要替换方法参数，执行完毕是否需要替换返回值。
    * 环绕通知第一个参数必须是org.aspectj.lang.ProceedingJoinPoint类型
    */
    @Around(value = POINT_CUT)
    public Object doAroundAdvice(ProceedingJoinPoint proceedingJoinPoint){
        logger.info("@Around环绕通知："+proceedingJoinPoint.getSignature().toString());
        Object obj = null;
        try {
            obj = proceedingJoinPoint.proceed(); //可以加参数
            logger.info(obj.toString());
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        logger.info("@Around环绕通知执行结束");
        return obj;
    }
}
```

**执行前后顺序是这样**

1. @Around环绕通知
2. @Before通知执行
3. @Before通知执行结束
4. @Around环绕通知执行结束
5. @After后置通知执行了!
6. @AfterReturning第一个后置返回通知的返回值：18
    

- org.aspectj.lang.JoinPoint ：方法中的参数JoinPoint为连接点对象，它可以获取当前切入的方法的参数、代理类等信息，因此可以记录一些信息，验证一些信息等。
- org.aspectj.lang.ProceedingJoinPoint： 为JoinPoint的子类，多了两个方法
- public Object proceed() throws Throwable; //调用下一个advice或者执行目标方法，返回值为目标方法返回值，因此可以通过更改返回值，修改方法的返回值
- public Object proceed(Object[] args) throws Throwable; //参数为目标方法的参数 因此可以通过修改参数改变方法入参
 

可以用下面的方式获取request、session等对象
```java
//attributes可以获取request信息 session信息等
ServletRequestAttributes attributes =
(ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
HttpServletRequest request = attributes.getRequest();
 ```

## 下面介绍一下切点表达式：

### 1. execution(方法修饰符 返回类型 方法全限定名(参数)) 主要用来匹配整个方法签名和返回值的

"execution(public * com.xhx.springboot.controller.*.*(..))"

**只能匹配一级路径, ..可以匹配多级，可以是包路径，也可以匹配多个参数, + 只能放在类后面，表明本类及所有子类   还可以按下面这么玩，所有get开头的，第一个参数是Long类型的。
    @Pointcut("execution(* *..get*(Long,..))")
### 2. within(类路径)   用来限定类，同样可以使用匹配符
下面用来表示com.xhx.springboot包及其子包下的所有类方法：**"within(com.xhx.springboot..*)"**

### 3. this与target

this与target在用法上有些重合，理解上有对比性。

this表示当前切入点表达式所指代的方法的对象的实例，即代理对象是否满足this类型

target表示当前切入点表达式所指代的方法的目标对象的实例   即是否是为target类做的代理

如果当前对象生成的代理对象符合this指定的类型，则进行切面，target是匹配业务对象为指定类型的类，则进行切面。

生成代理对象时会有两种方法，一个是CGLIB一个是jdk动态代理。用下面三个例子进行说明：
- this(SomeInterface)或target(SomeInterface)：

这种情况下，无论是对于Jdk代理还是Cglib代理，其目标对象和代理对象都是实现SomeInterface接口的（Cglib生成的目标对象的子类也是实现了SomeInterface接口的），因而this和target语义都是符合的，此时这两个表达式的效果一样；
- this(SomeObject)或target(SomeObject): 

这里SomeObject没实现任何接口：这种情况下，Spring会使用Cglib代理生成SomeObject的代理类对象，由于代理类是SomeObject的子类，子类的对象也是符合SomeObject类型的，因而this将会被匹配，而对于target，由于目标对象本身就是SomeObject类型，因而这两个表达式的效果一样；
- this(SomeObject)或target(SomeObject):

这里SomeObject实现了某个接口：对于这种情况，虽然表达式中指定的是一种具体的对象类型，但由于其实现了某个接口，因而Spring默认会使用Jdk代理为其生成代理对象，Jdk代理生成的代理对象与目标对象实现的是同一个接口，但代理对象与目标对象还是不同的对象，由于代理对象不是SomeObject类型的，因而此时是不符合this语义的，而由于目标对象就是SomeObject类型，因而target语义是符合的，此时this和target的效果就产生了区别；这里如果强制Spring使用Cglib代理，因而生成的代理对象都是SomeObject子类的对象，其是SomeObject类型的，因而this和target的语义都符合，其效果就是一致的。
 

### 4. args(paramType)

args无论其类路径或者是方法名是什么,表达式的作用是匹配指定参数类型和指定参数数量的方法，类型用全路径     

args(java.lang.String,..,java.lang.Integer)

## 5. @within(annotationType) 匹配带有指定注解的类，，within为配置指定类型的类实例
下面匹配含有 @Component注解的类
```java
"@within(org.springframework.stereotype.Component)"
```
## 6. @annotation(annotationType) 匹配带有指定注解的方法

## 7. @args(annotationType)
@args表示使用指定注解标注的类作为某个方法的参数时该方法将会被匹配
可以使用&&、||、!、三种运算符来组合切点表达式，表示与或非的关系。
@Around(value = "pointcut1() || pointcut2()")

```java
public interface ValidTradePayRpc {
Result<Boolean> validTradePay(@NotNull @Valid TradePayPO tradePayPO);
}
public class TradePayPO implements Serializable {2	
    @NotBlank
    @Length(max = 15)
    /** 业务交易编号(订单编号) */
    private String businessTradeNo;
	/**
	* 业务渠道：1-订阅，2-CA
    * @see BusinessTypeEnum
    *
    * */
    @NotNull
    @Range(min = 1, max = 2)
    private Integer businessType;

    /** 商户名称(商家) */
    @NotBlank
    @Length(max = 50)
    private String merchantName;
    	
    /** 订单标题（即商品名称），粗略描述用户的支付目的。如“喜士多（浦东店）消费”*/
    @NotBlank
    @Length(max = 256)
    private String orderSubject;
	
    /** 订单描述（即商品描述），可以对交易或商品进行一个详细地描述，比如填写"购买商品2件共15.00元"*/
    @NotBlank
    @Length(max = 128)
    private String orderBody;

}
```