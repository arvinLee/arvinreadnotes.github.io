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
