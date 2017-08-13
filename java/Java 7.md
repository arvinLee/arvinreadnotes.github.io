[TOC]



#Java 7

## 一 Java 7 语法新特性

​	OpenJDK的Coin项目的目的就是为了收集对Java语言的语法进行增强的建议。

### 1.1 在switch语句中使用字符串

​	原理可以通过反编译switch代码可以看出来，一个字符串的switch语句被换成了两个switch语句，第一个switch语句是根据字符串的hashcode确定一个int型变量，第二个switch语句是根据确定的int值来确定具体执行的代码块。

### 1.2 数值字面量的改进

#### 1.2.1 二进制证书字面量

​	八进制：012 = 10

​	十六进制：0x12 = 18

​	二进制：0b11 = 3（新增二进制字面量）

#### 1.2.2 在数值字面量中使用下划线

​	在java 7中，数值字面量，不管是整数还是浮点数，都允许在数字之间插入任意多个下划线，方便阅读。

​	下划线只能出现在数字中间。

### 1.3 优化的异常处理

​	重要改动：

​	1、一个是支持在一个子catch字句中同时捕获多个异常

​	2、在捕获并重新抛出异常时的异常类型更加精确。

#### 1.3.1 异常的基础知识

​	1、受检异常（checked exception）

​	优点：在编译时就必须处理异常，防止意外的忽略错误。

​	缺点：为了通过编译，要写大量异常处理代码。

​	2、非受检异常，包括运行时异常和Error（unchecked exception）

​	3、异常声明是API的一部分。

​	在一个公共方法的声明中使用throws关键字来声明七可能抛出的异常的时候，这些异常就成为这个公开方法得一部分，属于开发API。在维护这个公开API的时候，这些异常可能会对API的演化造成阻碍，使得编写代码是不得不考虑向后兼容性。

​	例如一个API抛出了A异常，该API的使用者肯定已经进行了try-catch-finally处理，后续如果增加、修改异常，API的使用者都必须重新捕获处理异常，否则代码无法通过编译。

​	因此要谨慎考虑每个公共方法所声明的异常。所以推荐使用非受检异常，但是非受检异常也要在api文档中进行说明。

#### 1.3.2 创建自己的异常

​	1、精心设计异常的层次结构

​	异常的层次结构与程序的类层次结构是相对应的，不同的抽象层次上的代码应该只声明抛出同一个层次上的相关异常。

​	比如web项目分为展现层、服务层、数据访问层。与之对应的异常也应该按照这个层次结构来进行划分。这么做的好处是工作于某个抽象层次上的开发人员不需要去了解其他层次的细节。

​	当一个异常抛出的时候，如果没有被捕获，就会一直沿着调用栈向上传递，直到被上层方法捕获或者最终由Java虚拟机来处理。这种传递方式会使这个异常跨越多个抽象层次的边界。如果上次代码不需要关注底层异常，那么一个异常在跨越抽象层次边界的时候，需要进行包装。包装只有的异常才是上层代码需要关注的。

​	包装异常的时候，一个典型的做法就是为每个层次定义一个基本的异常类，这个层次的所有公开方法在声明异常的时候都使用这个异常类。

​	2、异常类中包含足够的信息

​	异常类中应该包含当前的输入条件等等信息，方便准确的定位问题。

​	3、异常与错误提示

​	要区分异常与展示给用户的错误提示。通常来说，异常值得是程序的内部错误，与异常相关的信息，主要是共开发人员调用时使用。这些信息对于最终用户是没有意义的。因此，程序需要保证在直接与用户交互的代码层次上，捕获所有的异常，并生成项目的错误提示。比如在一个Servlet中，要确保在产生HTTP响应的时候捕获全部的异常，以避免用户看到一个包含异常堆栈信息的错误页面。

#### 1.3.3 处理异常

​	1、如果在出现异常的方法中又恢复办法（例如有默认处理办法）则可以在该方法中处理。

​	2、如果在出现异常的方法中没有恢复办法，则需要往上抛出异常。

​	3、当异常到达抽象层次边界时，需要进行包装后再往上传递。

​	4、finally与return

​	finally是在return后面的表达式运行后执行的（此时并没有将结果返回给调用者，而是先把要返回的至保存起来，不管finally中的代码是什么样的，返回的值都不会改变，依然是之前保存的值，注意引用类型变量），所以函数返回值是在finally执行前确定的。

​	finally中最好不要包含return，否则程序会提前退出，返回值不是try或catch中保存的返回值。	

​	5、消失的异常

​	在通过try-catch-finally处理异常时，如果finally中也抛出了异常，这个异常会往上传递，那么之前try语句中抛出的异常就会丢失。

​	Java 7中为Throwable类增加了addSuppressed方法，防止异常的丢失。

​	另外可以自己实现防止异常丢失的代码，即记录下catch抛出的异常，然后如果finally中抛出了异常，把记录的异常包装后一起抛出。

#### 1.3.4 Java 7的异常处理新特性 

​	1、一个catch子句捕获多个异常

​	Java 7改进了catch子句的语法，允许杂其中指定多种异常，每个异常类型之间使用“|”来分割。

```java
public void excute(){
  try{
    
  }catch(ExceptionA | ExceptionB ab){
    same handle.....
  }catch(ExcetpionC c){
    other handle.....
  }
}
```

​	ExceptionB不能是ExceptionA的子类。编译器会把该语法转换成多个catch子句。

​	2、更加精确的异常抛出

​	在进行异常处理的时候，如果遇到当前代码无法处理的异常，应该把异常重新抛出，交由调用栈的上层代码来处理。在重新抛出异常的时候，需要判断异常的类型。Java 7对重新抛出异常时的异常类型做了更加精确的判断，以保证抛出的异常的确是可以被抛出的。

​	异常被catch后重新抛出是，Java 7做了精确判断，如果在Java 7中编译不会通过，但是在Java 6版本中编译可以通过。

```java
public void excute(){
	try {
		throw new InterruptedIOException("");
    } catch (IOException e) {
		try {
        	throw e;
      	} catch (FileNotFoundException e1) {
			
      }
    }
}
```

### 1.4 try-catch-resources语句

​	Java 7对try语句进行了增强，继承自AutoCloseable接口的Java类，可以通过try语句自动关闭，如果关闭资源时出现异常，try语句的异常会被抛出，并且关闭资源的异常会被作为抑制异常添加到try语句抛出的异常中。

```java
public void excute(){
  	try(FileInputStrem in = new FileInputStrem("filepath")){
      	//流操作
  	}catch(IOException e){
      	//处理异常
  	}
}
```

​	try-catch-resources还可以同时管理多个资源

```java
public void excute(){
  	try(FileInputStream in = new FileInputStream("filepath");
       FileOutputStream out = new FileOutputStream("outfilepath")){
      	//流操作
  	}catch(IOException e){
      	//处理异常
  	}
}
```

### 1.5 优化变长参数的方法调用

​	Java 7之前，泛型可变参数方法，如果调用方传递的是不可具体化的类型，例如List\<String\>，编译器就会警告。

```java
public void exceute(){
  	List<String> list1 = new ArrayList<>();
  	List<String> list2 = new ArrayList<>();
  	test(list1,list2);//Java 6编译器会警告，类型不安全
  	test2(list1,list2);
}

public <T> void test(T... args){
  	System.out.println(args.length);
}

//Java 7提供了SafeVarargs注解，来标识参数可变的方法是类型安全的，但是SafeVarargs只能用在static或final方法中
@SafeVarargs
public <T> static void test2(T... args){
  	System.out.println(args.length);
}
```

## 二 Java 语言的动态性

### 2.1 脚步语言支持API

​	"多语言开发"，根据需求和语言本身特性选择合适的编程语言，以快速高效的解决问题。不同语言编写的代码可以同时运行在同一个Java虚拟机上，这些脚本语言与Java语言之间的交互，是由脚本语言支持API来完成的。

#### 2.1.1 脚本引擎

​	脚本的执行需要由该脚本语言对应的脚本引擎来完成。

```java
public void excute() throws ScriptException{
  	ScriptEngineManager engineManager = new ScriptEngineManager();
  	//过getEngineByExtension（"js"）和getEngineByMimeType（"text/javascript"）
  	ScriptEngine engine = engineManager.getEngineByName("JavaScript");
  	if(engine != null){
      	Object result = engine.eval("var a = 8*8;");
      	System.out.println(result);
  	}
}
```

#### 2.1.2 语言绑定

​	感觉更像是全局变量绑定，语言绑定对象，实际上就是一个Map，用于存放一些全局性的变量。ScriptEngine的get和put方法是使用了默认的语言绑定对象，用户也可以使用自己的语言绑定对象（实现javax.script.Bindings接口）。通过这种方式，就完成了Java与脚本语言之间的双向数据交互。

​	也可以使用自定义的数据绑定对象：

```java
public void useCustomBingding() throws ScriptException{
  	ScriptEngineManager engineManager = new ScriptEngineManager();
  	ScriptEngine engine = engineManager.getEngineByName("JavaScript");
  	if(engine != null){
      	Bindings bindings = new SimpleBindings();
      	bindings.put("num",8);
      	Object result = engine.eval("num*num;",bindings);
      	System.out.println(result);
  	}
}
```

#### 2.1.3 脚本执行上下文

​	脚本引擎通过上下文对象来获取与脚本执行相关的信息。

​	1、输入输出

​	默认情况下，脚本的输入输出都是发生在标准控制台中，通过ScriptContext的setReader和setErrorWriter方法可以分别设置脚本执行时的数据输入来源和产生错误时出错信息的输出目的。

```java
public void scriptToFile() throws IOException,ScriptException{
  	ScriptEngineManager engineManager = new ScriptEngineManager();
  	ScriptEngine engine = engineManager.getEngineByName("JavaScript");
  	ScriptContext context = engine.getContext();
  	context.setWriter(new FileWriter("output.txt"));
  	engine.eval("print('hello world')");
}
```

​	2、自定义属性

​	ScriptContext有与ServletContext中类似的获取和设置属性的方法，即setAttribute和getAttribute，ScriptContext中的属性是由作用域之分的。不同的作用域的区别在于查询属性时的优先级不同。优先级高的作用域中的属性会隐藏优先级低的作用域中的同名属性。

​	ScriptContext所包含的作用域是固定的，开发人员不能自定义作用域。通过getScopes方法可以得到所有可用的作用域列表。ScriptContext.ENGINE_SCOPE表示的作用域对应的是当前的脚本引擎，而ScriptContext.GLOBAL_SCOPE表示的作用域对应的是从同一引擎工厂这种创建出来的所有脚本引擎对象。前者优先级较高。

```java
public void scriptContextAttribute() throws IOException,ScriptException{
  	ScriptEngineManager engineManager = new ScriptEngineManager();
  	ScriptEngine engine = engineManager.getEngineByName("JavaScript");
  	ScriptContext context = engine.getContext();
  	context.setAttribute("name","Alex",ScriptContext.GLOBAL_SCOPE);
  	context.setAttribute("name","Bob",ScriptContext.ENGINE_SCOPE);
  	context.getAttribute("name");//值为Bob
}
```

​	3、语言绑定对象

​	脚本执行上下文中也可以设置语言绑定对象。与属性一样，有作用域之分。

```java
public void scriptContextBingdings() throws IOException,ScriptException{
  	ScriptEngineManager engineManager = new ScriptEngineManager();
  	ScriptEngine engine = engineManager.getEngineByName("JavaScript");
  	ScriptContext context = engine.getContext();
  	Bindings bindings1 = engine.crateBdindings();
  	bindings1.put("name","Alex");
  	Bindings bindings2 = engine.crateBdindings();
  	bindings2.put("name","Bob");
  	context.setBindings(bindings1,ScriptContext.GLOBAL_SCOPE);
  	context.setBindings(bindings2,ScriptContext.ENGINE_SCOPE);
  	engine.eval("print('name')")//值为Bob
}
```

​	脚本引擎中的put和get方法所操作的实际上就是ScriptContext中作用域为ENGINE_SCOPE的语言绑定对象。

```java
public void useScriptContextValues() throws IOException,ScriptException{
  	ScriptEngineManager engineManager = new ScriptEngineManager();
  	ScriptEngine engine = engineManager.getEngineByName("JavaScript");
  	ScriptContext context = engine.getContext();
  	Bindings bindings = context.getBindings(ScriptContext.ENGINE_SCOPE);
  	bindings.put("name","Alex");
  	engine.eval("print('name')")//值为Bob
}
```

脚本上下问的setAttribute实际上是向语言绑定对象中添加数据的。

#### 2.1.4 脚本的编译

​	脚本语言一般是解释执行的，脚本引擎在运行时需要先解释脚本之后再运行。一般来说，通过解释执行的方式来运行脚本的速度比编译之后再运行会慢一些。当一段脚本需要被多次重复执行时，可以先对脚本进行编译。实现javax.script.Compilable接口的引擎，支持对脚本进行编译。

​	Java SE中自带的JavaScript脚本引擎室支持对脚本进行编译的。

````java
public void compile(){
  	ScriptEngineManager engineManager = new ScriptEngineManager();
  	ScriptEngine engine = engineManager.getEngineByName("JavaScript");
  	Compilable compile = (Compilable)engine;
  	CompiledScript = compile.compile("print('hello world')");
  	script.eval();
}
````

#### 2.1.5 方法调用




