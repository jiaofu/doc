Aop
==
[TOC]
#简介
   AOP的源码中用到了两种动态代理来实现拦截切入功能：jdk动态代理和cglib动态代理。两种方法同时存在，各有优劣。jdk动态代理是由java内部的反射机制来实现的，cglib动态代理底层则是借助asm来实现的。总的来说，反射机制在生成类的过程中比较高效，而asm在生成类之后的相关执行过程中比较高效（可以通过将asm生成的类进行缓存，这样解决asm生成类过程低效问题）。还有一点必须注意：jdk动态代理的应用前提，必须是目标类基于统一的接口。如果没有上述前提，jdk动态代理不能应用。由此可以看出，jdk动态代理有一定的局限性，cglib这种第三方类库实现的动态代理应用更加广泛，且在效率上更有优势。
#切入点
##execution表达式
"execution(public int com.atguigu.spring.aop.impl.ArithmeticCalculator.*(int, int))")  
execution(<修饰符模式>? <返回类型模式> <方法名模式>(<参数模式>) <异常模式>?)  除了返回类型模式、方法名模式和参数模式外，其它项都是可选的.
<修饰符模式>   <异常模式> 是有可无的
一般的为  execution (* com.sample.service.impl..*.*(..))
 整个表达式可以分为五个部分：
> * 1、execution(): 表达式主体。
> * 2、第一个*号：表示返回类型，*号表示所有的类型。
> * 3、包名：表示需要拦截的包名，后面的两个句点表示当前包和当前包的所有子包，com.sample.service.impl包、子孙包下所有类的方法。
> * 4、第二个*号：表示类名，*号表示所有的类。
> * 5、*(..):最后这个星号表示方法名，*号表示所有的方法，后面括弧里面表示方法的参数，两个句点表示任何参数
## 组合
Pointcut可以有下列方式来定义或者通过&& || 和!的方式进行组合.
@args()
@target()
@within()
@annotation()
## 例子
``` python
   1)通过方法签名定义切点
 execution(public * *(..))
匹配所有目标类的public方法，但不匹配SmartSeller和protected voidshowGoods()方法。第一个*代表返回类型，第二个*代表方法名，而..代表任意入参的方法；

 execution(* *To(..))
匹配目标类所有以To为后缀的方法。它匹配NaiveWaiter和NaughtyWaiter的greetTo()和serveTo()方法。第一个*代表返回类型，而*To代表任意以To为后缀的方法；

2)通过类定义切点
 execution(*com.baobaotao.Waiter.*(..))
匹配Waiter接口的所有方法，它匹配NaiveWaiter和NaughtyWaiter类的greetTo()和serveTo()方法。第一个*代表返回任意类型，com.baobaotao.Waiter.*代表Waiter接口中的所有方法；

 execution(*com.baobaotao.Waiter+.*(..))
匹配Waiter接口及其所有实现类的方法，它不但匹配NaiveWaiter和NaughtyWaiter类的greetTo()和serveTo()这两个Waiter接口定义的方法，同时还匹配NaiveWaiter#smile()和NaughtyWaiter#joke()这两个不在Waiter接口中定义的方法。

3)通过类包定义切点
在类名模式串中，“.*”表示包下的所有类，而“..*”表示包、子孙包下的所有类。
 execution(* com.baobaotao.*(..))
匹配com.baobaotao包下所有类的所有方法；

 execution(* com.baobaotao..*(..))
匹配com.baobaotao包、子孙包下所有类的所有方法，如com.baobaotao.dao，com.baobaotao.servier以及com.baobaotao.dao.user包下的所有类的所有方法都匹配。“..”出现在类名中时，后面必须跟“*”，表示包、子孙包下的所有类；

 execution(* com..*.*Dao.find*(..))
匹配包名前缀为com的任何包下类名后缀为Dao的方法，方法名必须以find为前缀。如com.baobaotao.UserDao#findByUserId()、com.baobaotao.dao.ForumDao#findById()的方法都匹配切点。

4)通过方法入参定义切点

切点表达式中方法入参部分比较复杂，可以使用“*”和“..”通配符，其中“*”表示任意类型的参数，而“..”表示任意类型参数且参数个数不限。
 execution(* joke(String,int)))
匹 配joke(String,int)方法，且joke()方法的第一个入参是String，第二个入参是int。它匹配NaughtyWaiter#joke(String,int)方法。如果方法中的入参类型是java.lang包下的类，可以直接使用类名，否则必须使用全限定类名，如joke(java.util.List,int)；

 execution(* joke(String,*)))
匹 配目标类中的joke()方法，该方法第一个入参为String，第二个入参可以是任意类型，如joke(Strings1,String s2)和joke(String s1,double d2)都匹配，但joke(String s1,doubled2,String s3)则不匹配；
 execution(* joke(String,..)))
 
匹配目标类中的joke()方法，该方法第 一个入参为String，后面可以有任意个入参且入参类型不限，如joke(Strings1)、joke(String s1,String s2)和joke(String s1,double d2,Strings3)都匹配。
 execution(* joke(Object+)))
 
匹 配目标类中的joke()方法，方法拥有一个入参，且入参是Object类型或该类的子类。它匹配joke(Strings1)和joke(Client c)。如果我们定义的切点是execution(*joke(Object))，则只匹配joke(Object object)而不匹配joke(Stringcc)或joke(Client c)。

args()和@args()
args()函数的入参是类名，@args()函数的入参必须是注解类的类名。虽然args()允许在类名后使用+通配符后缀，但该通配符在此处没有意义：添加和不添加效果都一样。
1)args()
该函数接受一个类名，表示目标类方法入参对象按类型匹配于指定类时，切点匹配，如下面的例子：
args(com.baobaotao.Waiter)
表 示运行时入参是Waiter类型的方法，它和execution(**(com.baobaotao.Waiter))区别在于后者是针对类方法的签名而言的，而前者则针对运行时的入参类型而言。如args(com.baobaotao.Waiter)既匹配于addWaiter(Waiterwaiter)，也匹配于addNaiveWaiter(NaiveWaiter naiveWaiter)，而execution(**(com.baobaotao.Waiter))只匹配addWaiter(Waiterwaiter)方法；实际上，args(com.baobaotao.Waiter)等价于execution(**(com.baobaotao.Waiter+))，当然也等价于args(com.baobaotao.Waiter+)。
2)@args()

该函数接受一个注解类的类名，当方法的运行时入参对象标注发指定的注解时，方法匹配切点。这个切点函数的匹配规则不太容易理解，我们通过以下示意图对此进行详细讲解：
         
```
#通知
## 通知类型 
> * @Before：前置通知，在方法前通知；
> * @After :后置通知，在方法执行后通知；
> * @AfterRunning：返回通知，在方法返回结果之后通知；
> * @AfterThrowing：异常通知，在方法抛出异常之后通知；
> * @Around：环绕通知，围绕着方法执行；
## 方法接口
### JoinPoint 
> * java.lang.Object[] getArgs()：获取连接点方法运行时的入参列表； 
> > * Signature getSignature() ：获取连接点的方法签名对象； 
> * java.lang.Object getTarget() ：获取连接点所在的目标对象； 
> * java.lang.Object getThis() ：获取代理对象本身；
### ProceedingJoinPoint 
> *  ProceedingJoinPoint继承JoinPoint子接口，它新增了两个用于执行连接点方法的方法：
> *  java.lang.Object proceed() throws java.lang.Throwable：通过反射执行目标对象的连接点处的方法；
> *  java.lang.Object proceed(java.lang.Object[] args) throws java.lang.Throwable：
## 通知例子
```java
  package com.abc.advice;

import java.util.Arrays;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class AdviceTest {
    @Around("execution(* com.abc.service.*.many*(..))")
    public Object process(ProceedingJoinPoint point) throws Throwable {
        System.out.println("@Around：执行目标方法之前...");
        //访问目标方法的参数：
        Object[] args = point.getArgs();
        if (args != null && args.length > 0 && args[0].getClass() == String.class) {
            args[0] = "改变后的参数1";
        }
        //用改变后的参数执行目标方法
        Object returnValue = point.proceed(args);
        System.out.println("@Around：执行目标方法之后...");
        System.out.println("@Around：被织入的目标对象为：" + point.getTarget());
        return "原返回值：" + returnValue + "，这是返回结果的后缀";
    }
    
    @Before("execution(* com.abc.service.*.many*(..))")
    public void permissionCheck(JoinPoint point) {
        System.out.println("@Before：模拟权限检查...");
        System.out.println("@Before：目标方法为：" + 
                point.getSignature().getDeclaringTypeName() + 
                "." + point.getSignature().getName());
        System.out.println("@Before：参数为：" + Arrays.toString(point.getArgs()));
        System.out.println("@Before：被织入的目标对象为：" + point.getTarget());
    }
    
    @AfterReturning(pointcut="execution(* com.abc.service.*.many*(..))", 
        returning="returnValue")
    public void log(JoinPoint point, Object returnValue) {
        System.out.println("@AfterReturning：模拟日志记录功能...");
        System.out.println("@AfterReturning：目标方法为：" + 
                point.getSignature().getDeclaringTypeName() + 
                "." + point.getSignature().getName());
        System.out.println("@AfterReturning：参数为：" + 
                Arrays.toString(point.getArgs()));
        System.out.println("@AfterReturning：返回值为：" + returnValue);
        System.out.println("@AfterReturning：被织入的目标对象为：" + point.getTarget());
        
    }
    
    @After("execution(* com.abc.service.*.many*(..))")
    public void releaseResource(JoinPoint point) {
        System.out.println("@After：模拟释放资源...");
        System.out.println("@After：目标方法为：" + 
                point.getSignature().getDeclaringTypeName() + 
                "." + point.getSignature().getName());
        System.out.println("@After：参数为：" + Arrays.toString(point.getArgs()));
        System.out.println("@After：被织入的目标对象为：" + point.getTarget());
    }
}

@Around：执行目标方法之前...
@Before：模拟权限检查...
@Before：目标方法为：com.abc.service.AdviceManager.manyAdvices
@Before：参数为：[改变后的参数1, bb]
@Before：被织入的目标对象为：com.abc.service.AdviceManager@1dfc617e方法：manyAdvices
@Around：执行目标方法之后...
@Around：被织入的目标对象为：com.abc.service.AdviceManager@1dfc617e
@After：模拟释放资源...
@After：目标方法为：com.abc.service.AdviceManager.manyAdvices
@After：参数为：[改变后的参数1, bb]
@After：被织入的目标对象为：com.abc.service.AdviceManager@1dfc617e
@AfterReturning：模拟日志记录功能...
@AfterReturning：目标方法为：com.abc.service.AdviceManager.manyAdvices
@AfterReturning：参数为：[改变后的参数1, bb]
@AfterReturning：返回值为：原返回值：改变后的参数1 、 bb，这是返回结果的后缀
@AfterReturning：被织入的目标对象为：com.abc.service.AdviceManager@1dfc617e
Test方法中调用切点方法的返回值：原返回值：改变后的参数1 、bb，这是返回结果的后缀
```
## 获取注解的例子
### 通过反射机制获取切入点目标类的方法
``` java
public void invoke(JoinPoint joinPoint) throws Throwable{  
  
    //登录拦截  
 
  MethodInvocationProceedingJoinPoint methodPoint = (MethodInvocationProceedingJoinPoint)joinPoint; 
  
Field proxy = methodPoint.getClass().getDeclaredField("methodInvocation");  
  
proxy.setAccessible(true);  
  
ReflectiveMethodInvocation j = (ReflectiveMethodInvocation) proxy.get(methodPoint);  
  
Method method = j.getMethod();  
  
Login login = method.getAnnotation(Login.class);  
  
```
### 通过JoinPoint的getTarget()获取连接点所在的目标对象
``` java
public void invoke(JoinPoint joinPoint) throws Throwable{  
  
//拦截的实体类  
  
Object target = joinPoint.getTarget();  
  
//拦截的方法名称  
  
String methodName = joinPoint.getSignature().getName();  
  
//拦截的方法参数  
  
Object[] argsa = joinPoint.getArgs();  
  
//拦截的放参数类型  
  
Class[] parameterTypes = ((MethodSignature)joinPoint.getSignature()).getMethod().getParameterTypes();  
  
Method method = target.getClass().getMethod(methodName, parameterTypes);  
  
Login login = method.getAnnotation(Login.class);  
  
｝  
```
### 通过JoinPoint的getSignature()获取连接点的方法签名对象
``` java
public void invoke(JoinPoint joinPoint) throws Throwable{  
  
MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();  
  
Method method = methodSignature.getMethod();  
  
Login login = method.getAnnotation(Login.class);  
  
｝
```
#AOP代理（AOP Proxy）
：AOP框架创建的对象，包含通知。在Spring中，AOP代理可以是JDK动态代理或CGLIB代理。
#目标对象（Target Object）
包含连接点的对象，也被称作被通知或被代理对象。
#引入（Introduction）
添加方法或字段到被通知的类。Spring允许引入新的接口到任何被通知的对象。例如，你可以使用一个引入使任何对象实现IsModified接口，来简化缓存
#连接点（Joinpoint）
程序执行过程中明确的点，如方法的调用或特定的异常被抛出。
#方面（Aspect）
一个关注点的模块化，这个关注点实现可能另外横切多个对象。事务管理是J2EE应用中一个很好的横切关注点例子。方面用Spring的Advisor或拦截器实现。
#编织（Weaving）
组装方面来创建一个被通知对象。这可以在编译时完成（例如使用AspectJ编译器），也可以在运行时完成。Spring和其他纯Java AOP框架一样，在运行时完成织入
#例子
#注意事项



