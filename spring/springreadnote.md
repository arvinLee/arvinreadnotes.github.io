#1 IOC 容器
##1.1 容器概述
###1.1.1 实例化容器
####1.1.1.1 构建xml格式的beans配置文件
1、在一个xml中import其他配置文件时，强烈推荐使用绝对路径，虽然相对路径也可以，但是如果是classpath类型的资源，如果classpath配置改变，如果使用相对路径可能会导致只想错误的文件或路径。  
2、在使用绝对路径的时候，也可以将绝对路径写入配置文件中，间接引用该绝对路径。 
##1.2 Bean 概述
###1.2.1 Bean 命名
####1.2.1.1 Id与Name
1、通常一个bean只有一个唯一标识，但是如果需要多个，其他的标识可以叫做别名；  
2、id属性在3.1版本之前为xsd:ID格式，3.1之后改为xsd:string格式；  
3、name属性可以使用空格、逗号 (,), 分好(;)分割，同时指定多个值。  
4、在被其他bean引用时（ref），id和name都可以作为ref的值。  
5、在同一配置文件下，id和name是不能重复的，否则启动时抛出异常；在不同文件下，id和name默认是不检查是否重复的，但是根据实例化顺序，相同id或name的bean，后实例化的bean会覆盖之前实例化的bean；可以通过DefaultListableBeanFactory的setAllowBeanDefinitionOverriding设置不可覆盖来检查所有配置文件中是否有重复的id或name。  
6、如果在配置文件中没有指定id和name，容器会自动生成一个name，名称为类的完全限定名+\#+序号（org.asaopen.service.HelloService\#0），序号是根据相同class出现的次数编排的。
###1.2.2 实例化bean
####1.2.2.1 根据xml配置文件实例化bean
1、构造方法模式：通过new操作符调用class对应的构造方法  
2、静态工厂方法模式：，通过配置&lt;bean/&gt;的factory-method属性，调用class的static工厂模式方法创建对象；如果&lt;bean/&gt;的class属性为静态内部类，应该使用静态内部类的名称（com.example.Foo$Bar，Bar是Foo的静态内部类）  
3、实例化工厂模式：&lt;bean/&gt;的class置空，factory-bean属性引用定义的工厂bean，factory-method指定工厂bean对应的工厂方法。一个工厂类可以有一个以上的工厂方法。
##1.3 依赖-Dependencies
###1.3.1 依赖注入-Dependency Injection
####1.3.1.1 构造器注入
1、调用构造方法注入或者调用静态工厂方法注入是相似的。  
2、如果构造器参数之间没有继承关系或实现相同接口，不会混淆，可以只通过ref参数引用对应的bean即可。如下：  
​	
```xml
<beans>
    <bean id="foo" class="x.y.Foo">
        <constructor-arg ref="bar"/>
        <constructor-arg ref="baz"/>
    </bean>

    <bean id="bar" class="x.y.Bar"/>

    <bean id="baz" class="x.y.Baz"/>
</beans>
```

**如果构造器的两个参数有继承关系或者实现同一个接口，那么按照这两个参数在配置文件中的顺序依次注入。**  
3、如果通过value属性注入基础类型参数，spring无法知道value对应的基础类型，可以通过type属性来标识参数的具体类型，(测试发现，也可以不传type属性，spring会根据类型和顺序判断注入参数)如下：  

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```
4、也可以通过index属性标识是第几个参数，index从0开始。

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```
5、也可以通过参数名注入参数，但参数名注入是有限制的，需要在编译程序时打开调试模式（即在编译时使用“javac –g:vars”在class文件中生成变量调试信息，从而能获取参数名字，默认是不包含变量调试信息的，获取不到参数名字）或在构造器上使用@ConstructorProperties（java.beans.ConstructorProperties）注解来指定参数名。(eclipse可以通过勾选Properties>Java Compiler>add variable attributes to generated class filer，来实现控制变量调试信息的生成)

```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>

package examples;
	public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }

}
```

####1.3.1.2 Setter方法注入
1、Setter注入是先调用无参的构造方法或者无参静态工厂方法创建实例后，调用setter 方法注入的。  
2、一般通过构造器注入必须的依赖，通过setter方法注入可选的依赖，但是可以在setter方法上添加@Required 注解，使属性成为必须的依赖。  
3、setter方法注入可以重新注入，覆盖依赖。
####1.3.1.3 依赖解析过程 Dependency resolution process
1、IOC容器创建的时候，会校验每个bean的配置  
2、bean创建的时候，通过setter注入 properties  
3、单例的bean或者预实例化的bean(默认)在重启创建的时候被实例化，其他(懒加载)的bean只有在被用到的时候才被实例化    
4、构造器注入如果出现循环注入，可以通过setter注入解决  
###1.3.2 依赖与配置细节 Dependencies and configuration in detail
####1.3.2.1 直接值 Straight values (primitives, Strings, and so on)
1、通过 &lt;property/&gt; 的value属性来讲bean的属性和构造参数以可读的方式体现，Spring的**conversion service**会将这些值转换为对应的类型  
2、也可以通过使用命名空间p(p-namespace)来简化xml配置  

	<beans xmlns="http://www.springframework.org/schema/beans"
	    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xmlns:p="http://www.springframework.org/schema/p"
	    xsi:schemaLocation="http://www.springframework.org/schema/beans
	    http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
	        destroy-method="close"
	        p:driverClassName="com.mysql.jdbc.Driver"
	        p:url="jdbc:mysql://localhost:3306/mydb"
	        p:username="root"
	        p:password="masterkaoli"/>
	
	</beans>

3、Spring容器还支持通过 **PropertyEditor**机制，实例化一个**java.util.Properties**实例

	<bean id="mappings"
	    class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
	
	    <!-- typed as a java.util.Properties -->
	    <property name="properties">
	        <value>
	            jdbc.driver.className=com.mysql.jdbc.Driver
	            jdbc.url=jdbc:mysql://localhost:3306/mydb
	        </value>
	    </property>
	</bean>
4、idref 元素，是将其他bean的id作为字符串value通过&lt;constructor-arg/&gt;或者&lt;property/&gt;传给对应的bean，所有类似于constructor-arg/&gt;或者&lt;property/&gt;的value属性，但是idref可以在部署的时候校验id对应的bean是否存在

	<bean id="theTargetBean" class="..."/>

	<bean id="theClientBean" class="...">
	    <property name="targetName">
	        <idref bean="theTargetBean" />
	    </property>
	</bean>

等价于

	<bean id="theTargetBean" class="..." />

	<bean id="client" class="...">
	    <property name="targetName" value="theTargetBean" />
	</bean>

**注：idref元素的local属性可以指定是该xml中定义的bean，但是该属性在Spring4.0中已经过期，不再支持。**

5、关联其他bean References to other beans (collaborators)，是通过ref元素或者属性来实现的，ref元素有三个属性，bean, local(4.0过期), or parent。bean属性的值可以目标bean的id或者name；如果有相同id的bean存在父容器需要bean注入，可以通过parent属性来指定。  
6、内部beans(Inner beans)，是在&lt;constructor-arg/&gt;或者&lt;property/&gt;元素内部的bean元素，内部bean不需要定义id或者name，就算定义了，也会被忽略。内部bean除了注入到包围他的bean中外不能被注入到其他合作的bean中，也不能独立的访问他们。  
7、通过&lt;list/&gt;,&lt;set/&gt;,&lt;map/&gt;,&lt;props/&gt;等元素，你可以设置Java集合类型的参数，例如List, Set, Map, and Properties。  
​	
	<bean id="moreComplexObject" class="example.ComplexObject">
	    <!-- results in a setAdminEmails(java.util.Properties) call -->
	    <property name="adminEmails">
	        <props>
	            <prop key="administrator">administrator@example.org</prop>
	            <prop key="support">support@example.org</prop>
	            <prop key="development">development@example.org</prop>
	        </props>
	    </property>
	    <!-- results in a setSomeList(java.util.List) call -->
	    <property name="someList">
	        <list>
	            <value>a list element followed by a reference</value>
	            <ref bean="myDataSource" />
	        </list>
	    </property>
	    <!-- results in a setSomeMap(java.util.Map) call -->
	    <property name="someMap">
	        <map>
	            <entry key="an entry" value="just some string"/>
	            <entry key ="a ref" value-ref="myDataSource"/>
	        </map>
	    </property>
	    <!-- results in a setSomeSet(java.util.Set) call -->
	    <property name="someSet">
	        <set>
	            <value>just some string</value>
	            <ref bean="myDataSource" />
	        </set>
	    </property>
	</bean>
集合的值也可以还是集合元素。  
8、如果有继承关系的bean，可以通过集合的merge="true"的属性，来合并父bean中和子bean中集合的值，如果有序的集合，类似List类型，父bean的属性值排在子bean的属性值之前；Map或者Properties相同的key，value会被覆盖。

	<beans>
	    <bean id="parent" abstract="true" class="example.ComplexObject">
	        <property name="adminEmails">
	            <props>
	                <prop key="administrator">administrator@example.com</prop>
	                <prop key="support">support@example.com</prop>
	            </props>
	        </property>
	    </bean>
	    <bean id="child" parent="parent">
	        <property name="adminEmails">
	            <!-- the merge is specified on the child collection definition -->
	            <props merge="true">
	                <prop key="sales">sales@example.com</prop>
	                <prop key="support">support@example.co.uk</prop>
	            </props>
	        </property>
	    </bean>
	<beans>
注意，不能合并不同类型的集合属性，比如不能将list或者map；merge属性应该是child的属性，在parent上设置merge属性是没有意义的。  
9、Null and empty string values 
​	
	<bean class="ExampleBean">
	    <property name="email" value=""/>
	</bean>
	
	<bean class="ExampleBean">
	    <property name="email">
	        <null/>
	    </property>
	</bean>

10、通过p或c命名空间，可以简化xml配置
​	
	<beans xmlns="http://www.springframework.org/schema/beans"
	    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xmlns:p="http://www.springframework.org/schema/p"
		xmlns:c="http://www.springframework.org/schema/c"
	    xsi:schemaLocation="http://www.springframework.org/schema/beans
	        http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	    <bean name="p-namespace" class="com.example.ExampleBean"
	        p:email="foo@bar.com"  p:spouse-ref="jane"/>
	
		<!-- c-namespace declaration -->
		<bean id="foo" class="x.y.Foo" c:bar-ref="bar" c:baz-ref="baz" c:email="foo@bar.com"/>
		
		<!-- c-namespace index declaration -->
		<bean id="foo" class="x.y.Foo" c:_0-ref="bar" c:_1-ref="baz"/>
	</beans>
可以通过驼峰命名法来定义ref，例如p:spouseRef，但是这个容易和以Ref结尾的属性名冲突。  
11、复合属性名
​	
	<bean id="foo" class="foo.Bar">
	    <property name="fred.bob.sammy" value="123" />
	</bean>
上面例子标识，foo有一个fred属性，fred有一个bob属性，bob有一个sammy属性，但是fred、bob在foo构造完成后不能为null。
###1.3.3 使用depends-on
如果两个没有依赖关系的bean，需要按照一定顺序实例化，可以使用&lt;bean/&gt;的depends-on属性，depends-on属性可以通过逗号(,)、空格或者分号(;)定义多个bean需要在该bean实例化之前完成实例化。

	<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
	    <property name="manager" ref="manager" />
	</bean>
	
	<bean id="manager" class="ManagerBean" />
	<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
###1.3.4 懒加载bean Lazy-initialized beans
默认情况，ApplicationContext会在初始化阶段立即创建并配置所有的单例bean。可以通过&lt;bean/&gt;的lazy-init属性使单例bean在使用时才被创建，但是如果一个懒加载bean被一个非懒加载bean引用作为依赖，那么该懒加载bean也会在立即创建，然后注入到非懒加载bean。

同时也可以通过&lt;beans/&gt;元素的default-lazy-init属性，从容器基本控制bean的懒加载。
###1.3.5 自动装配 Autowiring collaborators
Spring可以通过配置自动装配Bean的依赖  
1、Autowiring的好处  
（1）Autowiring可以显著减少properties or constructor arguments的配置  
（2）自动装配可以根据你对象的变化自动更新配置，进而不用显示的去修改配置文件中的properties or constructor arguments  
2、可以通过&lt;bean/&gt;元素的autowire属性开控制bean的自动装配模式或关闭自动装备。可以通过&lt;beans/&gt;的default-autowire属性，配置整个xml内的bean的自动装配模式或关闭自动装备。  
3、自动装配模式 Autowiring modes
（1）no，默认为不进行自动装配，依赖必须通过ref属性或元素注入  
（2）byName，根据property名字进行自动装配，spring会查找有相同id/name的bean，调用setter方法，进行自动装配。  
（3）byType，spring容器会查找和property的class类型一样的唯一的一个bean，调用setter方法进行注入；如果容器中同一class类型出现多个bean，则会抛出异常  
（4）constructor，与byType类型，只是调用构造方法进行注入  
4、byType和constructor模式，可以注入数组或集合类型的属性，这个时候所有匹配数组或集合的类型的bean都会被添加进对应的数组或集合中，如果是Map类型属性，beand的id或name会被作为key，bean对象会被作为value。  
​	
	public class FlyBehaviorDisplay {
		private List<FlyBehavior> flyBehavior;
	
		public void setFlyBehavior(List<FlyBehavior> flyBehavior) {
			this.flyBehavior = flyBehavior;
		}
		
		public void performFly(){
			for(FlyBehavior behavior : flyBehavior){
				behavior.fly();
			}
		}
	}

xml配置

	<beans xmlns="http://www.springframework.org/schema/beans"
	    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xsi:schemaLocation="http://www.springframework.org/schema/beans
	        http://www.springframework.org/schema/beans/spring-beans.xsd" >
		<bean id="flyWithWings" class="org.asaopen.behavior.impl.FlyWithWings"/>
		<bean id="flyWithRocket" class="org.asaopen.behavior.impl.FlyWithRocket" />
	
	    <bean id="flyDisplay" class="org.asaopen.service.FlyBehaviorDisplay" autowire="byType"/>
	</beans>
5、自动装配的限制和缺点  
（1）property 和 constructor-arg 的明确依赖总是会覆盖自动装配。而且，不能自动装配基础数据类型、Strings、Classes和这里类型的集合或数组。  
（2）自动装配不够精确，对于对象之间的依赖关系显示的不够明确。  
（3）自动装配信息不能够通过工具生成文档。  
（4）如果类型一样的bean存在多个，虽然对于集合或数组是没有问题的，但是对于只需要单个依赖对象的bean的时候，spring就会抛出异常。  

6、对于一些自动装配的缺点，可以有以下几种选择：
（1）放弃自动装配  
（2）通过将&lt;bean/&gt;的autowire-candidate设置为false，让该bean在自动装配是不成为候选者。  
（3）通过将&lt;bean/&gt;的primary 设置为true，让该bean成为自动装配的主候选者。  
（4）通过注解实现更细粒度的配置  

7、可以通过&lt;beans/&gt;的default-autowire-candidates属性，通过bean名称的模式匹配来限制自动装配的候选者；但是如果&lt;bean/&gt;的autowire-candidate设置为false，模式匹配将不生效。  
###1.3.6 方法注入 Method injection
####1.3.6.1 使用场景
假设单例bean A在某个方法的调用中，每次需要一个非单例的bean B，如果bean B也是交给容器管理的，通过属性注入的方式将bean B注入到bean A中，是没有办法实现在每次调用方法的时候，得到一个新的bean B的。
这个时候，可以放弃控制反转(IOC)，通过实现ApplicationContextAware 接口，调用容器的getBean方法，获取一个新的bean B实例，代码如下：

	// a class that uses a stateful Command-style class to perform some processing
	package fiona.apple;
	
	// Spring-API imports
	import org.springframework.beans.BeansException;
	import org.springframework.context.ApplicationContext;
	import org.springframework.context.ApplicationContextAware;
	
	public class CommandManager implements ApplicationContextAware {
	
	    private ApplicationContext applicationContext;
	
	    public Object process(Map commandState) {
	        // grab a new instance of the appropriate Command
	        Command command = createCommand();
	        // set the state on the (hopefully brand new) Command instance
	        command.setState(commandState);
	        return command.execute();
	    }
	
	    protected Command createCommand() {
	        // notice the Spring API dependency!
	        return this.applicationContext.getBean("command", Command.class);
	    }
	
	    public void setApplicationContext(
	            ApplicationContext applicationContext) throws BeansException {
	        this.applicationContext = applicationContext;
	    }
	}
但是这种方式不是很令人满意，因为业务代码耦合了Spring框架。这个时候，方法注入(Method Injection)可以避免这种耦合。
####1.3.6.2 查找方法式注入(Lookup method injection，不知道翻译的对不对)
1、查找方法式注入，是容器重写它管理的beans的方法，该方法返回的查找结果是容器中一个其他命名bean。Spring框架通过cglib动态生成了一个重写了对应的方法的子类来实现方法注入。  
2、注意事项  
（1）为了能够动态生成子类，需要重写方法的bean不能是final类，方法不能是final方法。  
（2）查找方法式注入不能和工厂方法以及@Bean methods in configuration classes一起工作，因为此时容易不能控制创建实例也就不能再运行时创建一个子类。  
3、配置示例
	package fiona.apple;

	// no more Spring imports!

	public abstract class CommandManager {

	    public Object process(Object commandState) {
	        // grab a new instance of the appropriate Command interface
	        Command command = createCommand();
	        // set the state on the (hopefully brand new) Command instance
	        command.setState(commandState);
	        return command.execute();
	    }
	
	    // okay... but where is the implementation of this method?
	    protected abstract Command createCommand();
	}

xml

	<!-- a stateful bean deployed as a prototype (non-singleton) -->
	<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
	    <!-- inject dependencies here as required -->
	</bean>
	
	<!-- commandProcessor uses statefulCommandHelper -->
	<bean id="commandManager" class="fiona.apple.CommandManager">
	    <lookup-method name="createCommand" bean="myCommand"/>
	</bean>

需要注入的方法最好依照下面的格式(也可以不是抽象方法，如果是抽象方法，生成的子类实现抽象方法，如果是非抽象方法，生成的子类重写该方法)：
​	
	<public|protected> [abstract] <return-type> theMethodName(no-arguments);

这里commandManager会是一个通过cglib生成的动态代理子类。
4、以注解方法配置Lookup method injection
​	
	public abstract class CommandManager {

	    public Object process(Object commandState) {
	        Command command = createCommand();
	        command.setState(commandState);
	        return command.execute();
	    }
	
	    @Lookup("myCommand")
	    protected abstract Command createCommand();
	}
或者更简单

	public abstract class CommandManager {

	    public Object process(Object commandState) {
	        MyCommand command = createCommand();
	        command.setState(commandState);
	        return command.execute();
	    }
	
	    @Lookup
	    protected abstract MyCommand createCommand();
	}

####1.3.6.3 强制方法替换 Arbitrary method replacement
直接上例子：

	public class MyValueCalculator {

	    public String computeValue(String input) {
	        // some real code...
	    }
	
	    // some other methods...
	
	}

实现org.springframework.beans.factory.support.MethodReplace接口的类提供一个新的方法：

	/**
	 * meant to be used to override the existing computeValue(String)
	 * implementation in MyValueCalculator
	 */
	public class ReplacementComputeValue implements MethodReplacer {
	
	    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
	        // get the input value, work with it, and return a computed result
	        String input = (String) args[0];
	        ...
	        return ...;
	    }
	}
xml配置
​	
	<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
	    <!-- arbitrary method replacement -->
	    <replaced-method name="computeValue" replacer="replacementComputeValue">
	        <arg-type>java.lang.String</arg-type>
	    </replaced-method>
	</bean>
	
	<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>

当你执行myValueCalculator.computeValue(args)的时候，Spring会通过cglib创建一个MyValueCalculator的子类，然后调用replacementComputeValue.reimplement(args)方法，原MyValueCalculator的computeValue方法不再执行。  
###1.4 Bean范围(生命周期、Bean scopes)
####1.4.1 Bean scopes类型
1、singleton：默认scope，bean对应的实例一个IOC容器只有一个  
2、prototype：bean对应的实例可以有任意多个  
3、request：bean的生命周期与http request的生命周期一致，只在spring web容器中可用  
4、session：bean的生命周期与http session的生命周期一致，只在spring web容器中可用  
5、globalSession：bean的生命周期与global http session的生命周期一致，通常应用在Portlet context中(单独登录，统一门户)，只在spring web容器中可用  
6、application：bean的生命周期与ServletContext的生命周期一致，只在spring web容器中可用  
7、websocket：bean的生命周期与WebSocket的生命周期一致，只在spring web容器中可用  
###1.4.2 单例 The singleton scope
当你定义一个bean的时候，如果这个bean的scope是singleton，spring的IOC容器会创建一个实例对象，然后这一个实例被存在缓存中，所有通过Id、Name对这个bean的访问，依赖，得到的返回值都是被缓存的实例对象。 

Spring单例的实现和设计模式中单例的实现是截然不同的，设计模式中的单例是通过硬编码的形式，保证一个ClassLoader只有一个特定class的实例；spring的单例可以描述为一个容器一个bean，spring默认的scope就是singleton。  
###1.4.3 原型 The prototype scope
当你定义一个bean的时候，如果这个bean的scope是prototype，通过ID、Name对这个bean的访问，每次得到的都是一个新的对象，有一个规则：**有状态的bean用prototype，无状态的bean用singleton**。 

和其他scope相比，Spring不会管理prototype bean的这个生命周期：容器实例化、配置、组装一个prototype对象，把它交给client后就没有更多的记录。尽管初始化生命周期的方法被调用，但是没有销毁生命周期的方便可被调用。client必须自己清理prototype-scoped对象，释放prototype-scoped对象持有的资源。To get the Spring container to release resources held by prototype-scoped beans, try using a custom bean **post-processor**, which holds a reference to beans that need to be cleaned up。  

在某些方面，对于prototype-scoped bean来说，Spring container的角色就像是java中的new 操作符。(For details on the lifecycle of a bean in the Spring container, see Section 7.6.1, “Lifecycle callbacks”.)  
###1.4.4 Request, session, global session, application, and WebSocket scopes
request, session, globalSession, application, and websocket scopes 只有在web类型的容器实现中才是可用的，如果在非web项目中使用这些scope，会抛出unknown bean scope异常。
####1.4.4.1 Request scope 
xml配置如下：

	<bean id="loginAction" class="com.foo.LoginAction" scope="request"/>
注解配置如下：

	@RequestScope
	@Component
	public class LoginAction {
	    // ...
	}
由于每一次http请求，Spring容器都会创建一个新的loginAction对象，所以你可以在LoginAction实例中有很多状态字段，并且不用担心并发问题。
####1.4.4.2 Session scope
xml配置如下：

	<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>
注解配置如下：
​	
	@SessionScope
	@Component
	public class UserPreferences {
	    // ...
	}
####1.4.4.3 Global session scope
xml配置如下：

	<bean id="userPreferences" class="com.foo.UserPreferences" scope="globalSession"/>
globalSession scope和session scope很类似，但是globalSession 只应用在portlet-based的web项目中。如果你在普通的web项目中使用globalSession，事实上spring会使用session scope。
####1.4.4.4 Application scope
xml配置如下：

	<bean id="appPreferences" class="com.foo.AppPreferences" scope="application"/>
注解配置如下：

```java
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```
####1.4.4.5 Scoped beans as dependencies 作为依赖的bean
如果想要将一个HTTP request(session) scoped的bean注入到一个更长生命周期的bean中，需要使用AOP proxy去替代需要注入的bean。也就是说，你需要注入一个暴露了与较短生命周期bean相同接口的代理对象，而且这个对象能够找到真正的目标对象并且委托方法调用到实际对象上。 

注：  
1、You may also use <aop:scoped-proxy/> between beans that are scoped as singleton, with the reference then going through an intermediate proxy that is serializable and therefore able to re-obtain the target singleton bean on deserialization.   
针对singleton scope使用<aop:scoped-proxy/>，然后通过一个可序列化的中间代理，因此能够在反序列化时重新获得目标单例bean。 


