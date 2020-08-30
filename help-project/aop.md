# 热门框架原理源码

## 手写Spring AOP

### AOP分析

#### AOP框架的责任

- AOP是什么？

  Aspect Oriented Programming面向切面编程，在不改变类的代码的情况下，对类方法进行功能增强（代理类自身方法之间的调用不会进行增强）

- AOP框架要做什么？

  AOP框架中要面向用户提供AOP功能，让用户可以通过AOP技术实现对类方法进行功能增强

#### AOP元素分析

- Advice：通知，增强的功能																		用户提供 框架使用
- Joint points：连接点，可选的方法点 													   框架提供 用户使用                                                
- Pointcut：切入点，选择切入的方法点                                                      用户提供  框架使用
- Aspect：切面，选择的（多个）方法点+增强的功能                                用户提供  框架使用
- Introduction：引入：添加新的方法、属性到已存在的类中                     
- Weaving：织入（编织）：不改变原类的代码，加入功能增强                 框架实现

思考：这些东西，哪些是用户提供的，哪些是AOP框架要写好的？

#### AOP分析

根据AOP的定义：在不改变类的代码的情况下，对类方法进行功能增强。

| 进行功能增强：功能               | Advice通知            |
| -------------------------------- | --------------------- |
| 对类方法增强：可选择要增强的方法 | Pointcuts 切入点      |
| 不改变原类的代码，实现增强       | Weaving  织入（编织） |

Advice+Pointcuts=Aspect

#### 图解AOP

![image-20200829232936139](D:\code\spring-aop\help-project\image-20200829232936139.png)

#### AOP概念-特定分析

##### Advice

- 用户性：由用户提供增强功能逻辑代码
- 变化性 ：不同的增强需求，会有不同的逻辑
- 可选时机：可选择在方法功能前、后、异常时进行功能增强
- 多重性：同一个切入点上可以有多重增强

##### Pointcut

- 用户性：由用户来指定
- 变化的：用户可灵活指定
- 多点性：用户可以选择在多个点上进行增强

##### Weaving

- 无侵入性，不改原类代码
- 在AOP框架中实现

#### Advice设计

- Advice是由用户来提供，我们来使用，它是多变的。

  - ​	我们如何能认识用户提供的东西？用户在我们写好框架后使用我们的框架。
  - 如何让我们的代码隔绝用户提供的多变？

  我们定义一套标准接口，用户通过实现接口来提供他们不同的逻辑。

  **重要设计原则：如何应对变化，面向接口编程**

- 定义Advice接口

  接口没有定义任何方法

  ![image-20200830061708565](D:\code\spring-aop\help-project\img\image-20200830061708565.png)

- Advice的特定：可选择时机，可选择在方法功能前、后、异常时进行功能增强

  -  有的advice是在方法执行前进行增强；-》前置增强
  - 有的是在方法执行后进行增强；-》后置增强
  - 有的会是之前、之后都进行增强；-》环绕增强
  - 有的则只是在方法执行抛出异常时进行功能增强处理；-》异常处理增强

  **问：我们需要做什么？**

  定义标准接口方法，让用户可以实现它，提供各种增强。

- 前置增强：在方法执行前进行增强

  - ​	它可能需要什么参数？

    目的是对方法进行增强，应该需要的是方法相关的信息；我们使用它时，能给入它的好像也就只有当前要执行方法的信息。

  - 运行时方法有哪些信息？

    -   方法本身：Method
    - 方法所属的对象：Object
    - 方法的参数：Object[]

  - 前置增强的返回值是什么？

  ​          在方法执行前进行增强，不需要返回值

  #### Advice设计-后置增强分析

- 后置增强：在方法执行后进行增强

  - 它可能需要什么参数？

    -    方法本身：Method
    - 方法所属的对象：Object
    - 方法的参数：Object[]
    - 方法的返回值：Object

  - 它的返回值是什么？

    在方法执行后进行增强，不需要返回值！

#### Advice设计-环绕增强分析

- 环绕增强：包裹方法今星期增强

  - ​	它可能需要什么参数？
    - ​     方法本身：Method
    - 方法所属的对象：Object
    - 方法的参数：Object[]
  - 它的返回值是什么？

  方法被他包裹，也即方法的将由它来执行，他需要返回方法的返回值。Object

  #### 经前面的分析，我们一共需要定义三个方法

  - 思考：是把这个三个定义到一个接口中，换是分开三个接口定义？

  分三个接口，换可通过类型来区分不同的Advice

  ![image-20200830063323809](D:\code\spring-aop\help-project\img\image-20200830063323809.png)

![image-20200830063337477](D:\code\spring-aop\help-project\img\image-20200830063337477.png)

![image-20200830063355178](D:\code\spring-aop\help-project\img\image-20200830063355178.png)

![image-20200830065027878](D:\code\spring-aop\help-project\img\image-20200830065027878.png)![image-20200830065039599](D:\code\spring-aop\help-project\img\image-20200830065039599.png)

![image-20200830065152491](D:\code\spring-aop\help-project\img\image-20200830065152491.png)

#### Pointcut分析

- Pointcut的特定
  - 用户性：由用户来指定
  - 变化性：用户可灵活指定
  - 多点性：用户可以选择在多个点上进行增强

- 我们需要做什么？

  为用户提供一个东西，让他们可以灵活的指定多个方法点，而我们又能懂！

  思考：切入点是由用户来指定在哪些方法上进行增强，那么这个哪些方法点如何表示，能满足上面的特定？

- 分析

  -  指定哪些方法，是不是一个描述信息？

  - 如何来指定一个方法？

    XX类的XX方法

  - 重载怎么办？ 

     加上参数类型

  - 总结：其实就是一个完整的方法签名！

    ```java
    com.dn.spring.aop.Girl.dbj(Boy,Time)
    com.dn.spring.aop.Girl.dbj(Boy,Girl,Time)
    ```

  - 如何做到多点性，灵活性？在一个描述中指定一类方法？

    - 某个包下的某个类的某个方法
    - 某个包下的所有类中的所有方法
    - 某个包下的所有类中的do开头的方法
    - 某个包下的service结尾的类中的do开头的方法
    - 某个包下的及其子包下的以service结尾的类中的do开头的方法

    我们需要一个表达式，能灵活描述这些信息的表达式

  - 要表达的哪些信息？

     包名.类名.方法名（参数类型）

  - 每部分的要求是怎样的？
    -   包名：有父子特定，要能模糊匹配
    - 类名：要能模糊匹配

  - 这个表达式将被我们用来决定是否需要对某个类的某个方法进行增强，这个决定过程应该是怎样的？

    匹配类，匹配方法

  - 一个表达式如果不好实现，分成多个表达式进行组合是否容易些？

    是的，可以这个考虑

  - 我们掌握的表达式有哪些？他们是否能满足这里的需要？

    -  正则表达式？

    - Ant Path表达式？

    - AspectJ的pointcut表达式？

      ```java
      execution(* com.xyz.service.AccountService.*(..))
      ```

      正则表达式是可以的。AspectJ本就是切面编程组件，也是可以的。

      execution(modifiers-pattern? ret-type-pattern  declaring-type-pattern?name-pattern(param-pattern) throws-pattern?)

  - ​     该来对Pointcu进行接口、类设计了

    -  切点应该有什么属性？

      切点定义表达式

    - 切点应对外提供什么行为（方法）？

    - 切点将被我们用来做什么？

       对类、方法进行匹配

      切点应提供匹配类，匹配方法的行为

    - 如果在我们的设计的框架中要能灵活扩展切点的实现，我们该如何设计？

      这又是一个要支持可多变的问题，像通知一样，我们来定义一套标准接口、定义好基本行为，面向接口编程，屏蔽掉具体的实现。

  - 该来对Pointcut进行接口、类设计了

    -  切点应该有什么属性？

      切点表达式

    - 切点应对外提供什么行为（方法）？

    - 切点将被我们用来做什么？

       对类、方法进行匹配

      切点应该提供匹配类，匹配方法的行为。

    - 如果在我们设计的框架中要能灵活扩展切点的实现方式，我们该如何设计？

      这又是一个要支持可多变的问题，像通知一样，我们来定义一套标准，定义好基本行为，面向接口编程，屏蔽掉具体的实现。

      无论哪种试下，都实现匹配类、匹配方法的接口

- Pointcut标准接口

  ![image-20200830074629352](D:\code\spring-aop\help-project\img\image-20200830074629352.png)

![image-20200830074721043](D:\code\spring-aop\help-project\img\image-20200830074721043.png)

![image-20200830074731226](D:\code\spring-aop\help-project\img\image-20200830074731226.png)

#### AspectJExpressionPointcut实现

- 实现步骤

  -  引入AspectJ的jar

    ![image-20200830074913690](D:\code\spring-aop\help-project\img\image-20200830074913690.png)

  - 掌握AspectJ的api使用，我们只使用它的切点表达式解析匹配部分

    - ​       入口：org.aspectj.weaver.tools.PointcutParse  获得切点解析器：

      ![image-20200830075046194](D:\code\spring-aop\help-project\img\image-20200830075046194.png)

    - 解析表达式，得到org.aspectj.weaver.tools.PointcutExpression

       ![image-20200830075131114](D:\code\spring-aop\help-project\img\image-20200830075131114.png)

    - 用PointcutExpression匹配类，不可靠。没关系，通过方法匹配来正确匹配。

      ![image-20200830075222964](D:\code\spring-aop\help-project\img\image-20200830075222964.png)

    - 用PointcuExpression匹配方法，可靠

      ![image-20200830075253647](D:\code\spring-aop\help-project\img\image-20200830075253647.png)

#### Advice Pointcut使用

- 用户该如何使用我们提供的东西？

  ![image-20200830075454745](D:\code\spring-aop\help-project\img\image-20200830075454745.png)

  ![image-20200830075504928](D:\code\spring-aop\help-project\img\image-20200830075504928.png)![image-20200830075513519](D:\code\spring-aop\help-project\img\image-20200830075513519.png)

#### Advisor扩展说明

- 扩展不同的Advisor实现

    ![image-20200830075547722](D:\code\spring-aop\help-project\img\image-20200830075547722.png)

- 换可这样设计

  ![image-20200830075602781](D:\code\spring-aop\help-project\img\image-20200830075602781.png)

- #### 不同Advice的执行顺序是什么？

  - 正常结束

  ```java
  @RestController
  @RequestMapping("/")
  class DemoOrder{
  
  
      @RequestMapping("/demo")
      public String demo(String test) {
          System.out.println("正在执行demo方法");
          return "demo方法返回值";
      }
  
  }
  
  
  
  @Component
  @Aspect
  public class AdviceOrderConfig {
  
  
      @Before("execution(* edu.dongnao.courseware.spring.aop.adviceorder.DemoOrder.demo(..))")
      public void beforeProcess() {
          System.out.println("进入before方法");
       }
  
      @AfterReturning(pointcut="execution(* edu.dongnao.courseware.spring.aop.adviceorder.DemoOrder.demo(..))",
      returning = "retValue")
      public void afterReturningProcess(Object retValue) {
          System.out.println("进入afterReturningProcess方法:"+retValue);
      }
  
  
      @AfterThrowing(pointcut="execution(* edu.dongnao.courseware.spring.aop.adviceorder.DemoOrder.demo(..))",
              throwing = "ex")
      public void afterThrowingProcess(RuntimeException ex) {
          System.out.println("进入afterThrowingProcess方法:"+ Arrays.toString(ex.getStackTrace()));
      }
  
  
      @After("execution(* edu.dongnao.courseware.spring.aop.adviceorder.DemoOrder.demo(..))")
      public void afterProcess() {
          System.out.println("进入after方法");
      }
  
  
      @Around("execution(* edu.dongnao.courseware.spring.aop.adviceorder.DemoOrder.demo(..))")
      public void aroundProcess(ProceedingJoinPoint joinPoint) throws Throwable {
          System.out.println("进入aroundProcess方法，在执行切入点的方法前");
          try {
              Object retValue = joinPoint.proceed();
          } catch (Exception exception) {
          }
          System.out.println("进入aroundProcess方法，在执行切入点的方法后");
      }
  
  }
     
  ```

  ```
  #没有异常的调用结果
  进入aroundProcess方法，在执行切入点的方法前
  进入before方法
  正在执行demo方法
  进入afterReturningProcess方法:demo方法返回值
  进入after方法
  进入aroundProcess方法，在执行切入点的方法后
  ```

  执行顺序：around-》before-》方法执行-》afterReturning-》after-》around

  - 抛出异常

  ```java
  @RestController
  @RequestMapping("/")
  class DemoOrder{
  
      @RequestMapping("/demo")
      public String demo(String test) {
          System.out.println("正在执行demo方法");
          throw new RuntimeException("执行demo方法抛出异常");
      }
  
  }
  ```

  ```
  #有异常抛出的结果
  进入aroundProcess方法，在执行切入点的方法前
  进入before方法
  正在执行demo方法
  进入afterThrowingProcess方法:执行demo方法抛出异常
  进入after方法
  进入aroundProcess方法，在执行切入点的方法后：
  ```

  执行顺序：around-》before-》方法执行-》afterThrowing-》after-》around

  总结：1）after类似于finally，不管没有没有异常都会执行

  ​            2）around的优先级最高，第一个执行和最后结束

  ​		    3）afterReturning正常返回时执行，afterThrowing异常返回时执行

- #### spring AOP和Aspect AOP有什么区别？



































