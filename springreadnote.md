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
2、静态工厂方法模式：，通过配置<bean/>的factory-method属性，调用class的static工厂模式方法创建对象；如果<bean/>的class属性为静态内部类，应该使用静态内部类的名称（com.example.Foo$Bar，Bar是Foo的静态内部类）  
3、实例化工厂模式：<bean/>的class置空，factory-bean属性引用定义的工厂bean，factory-method指定工厂bean对应的工厂方法。一个工厂类可以有一个以上的工厂方法。
##1.3 依赖-Dependencies
###1.3.1 依赖注入-Dependency Injection
####1.3.1.1 构造器注入
1、调用构造方法注入或者调用静态工厂方法注入是相似的。  
2、如果构造器参数之间没有继承关系或实现相同接口，不会混淆，可以只通过ref参数引用对应的bean即可。如下：  
	
	<beans>
	    <bean id="foo" class="x.y.Foo">
	        <constructor-arg ref="bar"/>
	        <constructor-arg ref="baz"/>
	    </bean>
	
	    <bean id="bar" class="x.y.Bar"/>
	
	    <bean id="baz" class="x.y.Baz"/>
	</beans>

**如果构造器的两个参数有继承关系或者实现同一个接口，那么按照这两个参数在配置文件中的顺序依次注入。**  
3、如果通过value属性注入基础类型参数，spring无法知道value对应的基础类型，可以通过type属性来标识参数的具体类型，(测试发现，也可以不传type属性，spring会根据类型和顺序判断注入参数)如下：  

	<bean id="exampleBean" class="examples.ExampleBean">
	    <constructor-arg type="int" value="7500000"/>
	    <constructor-arg type="java.lang.String" value="42"/>
	</bean>
4、也可以通过index属性标识是第几个参数，index从0开始。

	<bean id="exampleBean" class="examples.ExampleBean">
	    <constructor-arg index="0" value="7500000"/>
	    <constructor-arg index="1" value="42"/>
	</bean>
5、也可以通过参数名注入参数，但参数名注入是有限制的，需要在编译程序时打开调试模式（即在编译时使用“javac –g:vars”在class文件中生成变量调试信息，从而能获取参数名字，默认是不包含变量调试信息的，获取不到参数名字）或在构造器上使用@ConstructorProperties（java.beans.ConstructorProperties）注解来指定参数名。(eclipse可以通过勾选Properties>Java Compiler>add variable attributes to generated class filer，来实现控制变量调试信息的生成)

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

####1.3.1.2 Setter方法注入
1、Setter注入是先调用无参的构造方法或者无参静态工厂方法创建实例后，调用setter 方法注入的。
2、一般通过构造器注入必须的依赖，通过setter方法注入可选的依赖，但是可以在setter方法上添加@Required 注解，使属性成为必须的依赖。
3、setter方法注入可以重新注入，覆盖依赖。
####1.3.1.3 依赖解析过程 Dependency resolution process
1、IOC容器创建的时候，会校验每个bean的配置  
2、bean创建的时候，通过setter注入 properties  
3、单例的bean或者预实例化的bean(默认)在重启创建的时候被实例化，其他(懒加载)的bean只有在被用到的时候才被实例化    