#Gradle Build 基础语法之Groovy编程
Gradle Build脚本是用Groovy语言写的。Groovy运行在JVM虚拟机上，它填补了java程序员需要的脚本语言功能。
<br>关于Groovy语言构建Gradle Build脚本，我们不需要了解很多就可以做很多事情。Gradle提供了它自己的Groovy版本，也就是说我们甚至不需要安装Groovy。我们只需要将Groovy代码放到build.gradle中即可，然后运行gradle命令build.gradle中的代码。
	
	task groovy <<{}
	println "Hello Groovy!"
 task定义了一个任务，这样运行gradle tasks就可以列出它。而打印语句由于没有在特定的块里面，只要gradle.build运行就会执行。即gradle groovy就可以看见输出Hello Groovy!
###兼容大部分的java
大部分的Java语法，语句，也是有效的Groovy语法，语句。我们可以写java，然后访问一些标准库:
		
	class JavaGreeter {
    public static void sayHello() {
        System.out.println("Hello Java!");
    }
	}

	JavaGreeter greeter = new JavaGreeter()
	greeter.sayHello()
这样只要运行脚本，就会输出Hello Java！
###定义变量
Groovy是动态语言，所以它的变量是动态类型，也就是说，只有到语句被解析时，变量才会有意义，或者对应的类型才确定，所以在声明它时不需要有确定的类型。

	def foo = 6.5
可以通过变量名.class来得知对应变量的类型：
	
	println "foo is of type: ${foo.class} and has value: $foo"
	foo = "a string"
	println "foo is now of type: ${foo.class} and has value: $foo"
两次的类型，分别是class java.math.BigDecimal和java.lang.String。
###字符串中的变量引用 
类似文本串的格式化，Groovy可以使用'$foo'的形式将变量值插入到字符串中：
		
	println "foo has value: $foo"
对应的输出为：foo has value: 6.5.
<br>当然，还可以插入代码:
		
	println "Let's do some math. 5+6 = ${5+6}" 
对应输出：Let's do some math. 5+6 = 11.
<br>对应的语法就是${code expr}.
###方法
`println`关键字展开后就是`System.out.println`，而且调用时不需要加`()`，也不需要以分号结尾。有没有分号，是可选的。对于括号，无论方法需要多少个参数，只要参数的使用不产生歧义，就可以不加括号。
<br>而且，方法不需要return，最后的语句结果就是返回值。输入参数的类型，也不需要指定，只要参数重写了`+`的方法，就可以。			
		
	def doubleIt(n){
		n + n
	}
	foo = 5
	println "doubleIt($foo) = ${doubleIt(foo)}"
	foo = "foobar"
	println "doubleIt($foo) = ${doubleIt(foo)}"
由于字符串也重写了`+`，所以输出为foobarfoobar。
<br>下面展示方法参数的例子：
		
	def noArgs(){
		println "Called the no args function"
	}
	
	def oneArg(x){
		println "Called the 1 arg function with $x"
		x
	}
	
	def twoArgs(x,y){
		println "Called the 2 arg function with $x and $y"
		x + y
	}
	
	oneArg 500
	twoArgs 200,300
	noArgs()
	//noArgs // Doesn't work,variable or method?
	//twoArgs oneArg 500, 200 // Also doesn't work as it's ambiguous,(	twoArgs oneArg) is oneArg variable as 500?
	twoArgs oneArg(500), 200 // Fixed the ambiguity
以上的例子，混淆的地方都是某个符号到底是方法名还是变量名，在解析时无法分辨。还有就是`twoArgs oneArg(500), 200 `,若去掉括号，那么oneArg以为你要给它两个参数，显然会报错。
##闭包
所谓的闭包，有两个特性，第一个是可以赋值给某个变量，第二是，可以获得闭包方法所在的上下文变量引用。
		
	def foo = "One million dollars"
	def myClosure = {
		println "Hello from a closure"
		println "The value of foo is $foo"
	}
	
	myClosure()
	def bar = myClosure
	def baz = bar
	baz()
上面展示了基本的闭包方法概念，本身作为变量值，以及捕获周边的变量值。当然，foo可以随时改变，那么输出的也会改变。闭包的语法，大括号里的是方法体，然后右边的等于，相当于赋值给了闭包变量。那么参数的话，可以引用全局的，例如foo变量，也可以像lambda表达式一样，使用输入参数：
		
	def doubleIt = {x -> x + x}
	println "10+10 = ${doubleIt(10)}"
###闭包作为函数参数
既然闭包方法可以赋值给变量，那么显然就可以作为函数的参数了：
		
	def applyTwice(func, arg){
    func(func(arg))
	}

	foo = 5
	def fooDoubledTwice = applyTwice(doubleIt, foo)
	println "Applying doubleIt twice to $foo equals $fooDoubledTwice"
以上的这些特性，就是所谓的[“first class functions”](https://en.wikipedia.org/wiki/First-class_function)，意味着，它可以作为函数参数，返回值，以及存储在变量，或者其他数据结构中。下面的例子，就是在闭包中再定义一个闭包返回(可以让外面的方法访问方法或者闭包内部的变量)：
		
	def myClosure = {
    println "Hello from a closure"
    def var = "I'm var in myClosure"
    def innerClosure = {
    	println "visit $var"
    }
    innerClosure
	}
	def func = myClosure()
	func()
###列表处理
定义列表，直接上[]就可以（好像python）：
		
	def myList = ["Gradle","Groovy","Android"]
	println "The type of myList is ${myList.class}"
最终myList对应的是java的`java.util.ArrayList`类型。然后，可以结合闭包方法快速进行列表中元素的迭代处理：
		
	def printItem = {item -> println "List item: $item"}
	myList.each(printItem)
gradle支持这种["高阶函数"](https://zh.wikipedia.org/wiki/%E9%AB%98%E9%98%B6%E5%87%BD%E6%95%B0)的特性。
<br>根据之前描述的函数调用方法，和闭包生成方法，其实，可以直接使用如下的形式：
		
	myList.each{println "Compactly printing each list item: $it"}
若只有一个参数，直接使用it即可引用。
###闭包作为方法的最后一个参数
当闭包作为方法的最后一个参数时，可以直接使用如下引用方法：
	
	task foo <<{}
	def fooFunc(name,c){
 		c(name)
	}
	fooFunc("jason"){
		println "$it"
	}
##类和闭包
这里的类定义可以像java一样，也可以如下定义和使用：
		
	class GroovyGreeter {
    String greeting = "Default greeting"
    def printGreeting(){println "Greeting: $greeting"}
	}

	def myGroovyGreeter = new GroovyGreeter()

	myGroovyGreeter.printGreeting()
	myGroovyGreeter.greeting = "My custom greeting"
	myGroovyGreeter.printGreeting()
Groovy中，闭包可以有一个delegate对象，在闭包的方法体中用到的变量和方法都可以在这个对象中去查找并使用：
		
	def greetingClosure = {
    greeting = "Setting the greeting from a closure"
    printGreeting()
	}

	// greetingClosure() // This doesn't work, because `greeting` isn't defined
	greetingClosure.delegate = myGroovyGreeter
	greetingClosure() // This works as `greeting` is a property of the delegate
仔细查看List接口，好像没有each方法，原来，Groovy为特定的类增加了静态方法，方法是：
[`each(List<T> self, Closure closure)`](http://docs.groovy-lang.org/latest/html/api/org/codehaus/groovy/runtime/DefaultGroovyMethods.html#each(java.util.List,%20groovy.lang.Closure))，而你调用的时候，第一个参数就是调用的list，不需要你传入。这个特性很像C#的扩展方法，静态方法的第一个参数是某种类型，那么这个方法就会被加入到该类型中。
<br>[学习资料一](http://learnxinyminutes.com/docs/groovy/),[资料二](http://groovy.codehaus.org/User+Guide).
##Task
就像闭包可以有一个上下文的委托对象，整个build 脚本有一个project的委托对象。在Gradle DSL中的关键词，比如属性，方法等，都是在这个project对象中定义。
例如，在project对象中有一个task的方法，用来声明task对象。它接受要创建的task的名称，以及一个配置闭包，后者会在稍后介绍，先看简单的声明：

	project.task("myTask1")
由于整个脚本的委托对象就是project一个，所以可以省略project前缀，直接声明:

	task("myTask2")
之前，我们知道方法的括号可以省略，所以还可以如下定义:

	task "myTask3"
我们好像还可以再偷懒一点，那就是把双引号也省略掉：

	task myTask4
这个特性，可以查看[stackoverflow](http://stackoverflow.com/questions/27584463/understing-the-groovy-syntax-in-a-gradle-task-definition)上的解答。
<br>声明好了task后，它就变成了我可以访问的对象，那么，就可以直接引用它们，设置属性:

	myTask4.description = "This is what's shown in the task list"
	myTask4.group = "This is the heading for this task in the task list,"
这样，你只要执行gradle tasks，就可以发现myTask4有自己的介绍，并且属于单独的一组。当然，你可以为不同的task设置相同的group，这样就可以归纳在一个组下。
###Task的actions
Task对象最重要的属性就是actions 列表，所谓的action，就是可以执行的闭包。我们可以使用特定的方法来向actions属性增加闭包。比如，添加到actions表尾：

	myTask4.doLast {println "Do this last (doLast)"}
添加到表头：

	myTask4.doFirst {println "Do this first (doFirst)"}
我们也可以使用`leftShift`操作符进行添加，以下两种方式等价：

	myTask4.leftShift {println "Do this even more last (leftShift)"}
	myTask4 << {println "Do this last of all (<<)"}
执行`gradle myTask4`后，输出的顺序如下：

	Do this first (doFirst)
	Do this last (doLast)
	Do this even more last (leftShift)
	Do this last of all (<<)
所以`<<`也是添加到末尾。
<br>刚才有提到，task方法还可以接受一个闭包，如下：

	task myTask5 << {
		println "Here's how to declare a task and give it an action in one stroke"
	}
这样就可以简单得一笔生成可以执行的task。
<br>除了在声明task之后为它们设置属性，我们可以配置一个configuration闭包，然后初始化变量:
	
	task myTask6 {
		description "Here's a task with a configuration block"
		group "Some group"
		doLast {
			println "Here's the action!"
		}
	}
关于configuration 闭包（以下简称cc），有两个重要的点。
<br>1.cc在执行时，委托对象就是task本身，所以你可以直接设置它里面的属性。<br>2.task对象里面的属性，都拥有一个同名的set方法，比如group是个属性，但是也有一个set的方法叫做group.
	
	task myTask7 {
		description("Description")//Function call works
		    //description "Description" // This is identical to the line above
		 group = "Some group" // Assignment also works
		  doLast { // We can also omit the parentheses, because Groovy syntax
        println "Here's the action"
    	}
	}
还有种创建方式（太多创建方式，感觉不太妙），那就是在方法参数中，直接初始化：

	task myTask8(description: "Another description") << {
		println "Doing something"
	}
大多数时候，属性在设么地方设置，完全取决于可读性，对应的task创建[create api](http://gradle.org/docs/current/javadoc/org/gradle/api/tasks/TaskContainer.html#create(java.util.Map)
)中有详解。但是，除了一种情况，那就是task type的设置，必须要用上面这种key:value的方式设置，至于type是什么，之后会介绍。
##Task 依赖
task之间是可以有关系的，下面讨论用法：
<br>1.denpendsOn：若A任务依赖于B，则B必须在A运行之前运行:

	task putOnSocks {
    doLast {
        println "Putting on Socks."
    }
	}

	task putOnShoes {
    dependsOn "putOnSocks"
    doLast {
        println "Putting on Shoes."
    }
	}
而且，你是无法使用gradle tasks看见putOnSocks，gradle认为这只是个协助任务，你需要运行gradle tasks --all 才可以看见全部任务。
<br>2.finalizedBy: A任务运行后需要运行B任务:
	
	task eatBreakfast {
    finalizedBy "brushYourTeeth"
    doLast{
        println "Om nom nom breakfast!"
    }
	}

	task brushYourTeeth {
	    doLast {
        println "Brushie Brushie Brushie."
    	}
	}	
<br>3.shouldRunAfter ,当A和B会同时运行，那么先运行B，这种关系很弱，A和B是完全可以独立运行的，而不依赖于对方。
	
	task A {
    shouldRunAfter "B"
    doLast{
        println "Om nom nom breakfast!"
    }
	}

	task B {
	    doLast {
        println "Brushie Brushie Brushie."
    	}
	}	
<br>4.mustRunAfter ,和3类似，查看[解释](https://blog.safaribooksonline.com/2013/08/16/gradle-task-ordering/).
	`putOnShoes.mustRunAfter takeShower`

<br>5.依赖多个任务:
	
	task getReady {
    // Remember that when assigning a collection to a property, we need the
    // equals sign
    dependsOn = ["takeShower", "eatBreakfast", "putOnShoes"]
	}
<br>5.高级用法，声明依赖于已"putOn"开头的任务:

	task getEquipped {
    dependsOn tasks.matching{ task -> task.name.startsWith("putOn")}
    doLast {
        println "All geared up!"
    }
	}
关于tasks等更多概念，查看[文档](https://docs.gradle.org/current/userguide/more_about_tasks.html).
##文件的Copy和Archive
在Gradle中，拷贝文件非常简单，只要定义一个task，然后设置type为"Copy"即可，下面是定义一个空的拷贝task：
	
	task copyTask(type: Copy)
上面的这个task，什么也不做，对于copy，我们至少需要知道cppy源和目标，如下：
	
	task copyImages(type: Copy){
		from 'images'
		into 'build'
	}
上面copyTask的意思就是拷贝images目录的内容到build目录中去，当时目录本身不拷贝。
###定制拷贝任务
有时候拷贝的目标是特定扩展的文件，或者命名有规律的文件。Gradle中，可以使用通配符来实现拷贝文件的过滤，结合from，into，include，exclude等一起作用，称为CopySpec，下面的task只拷贝以".jpeg"结尾的文件：
	
	task copyJpegs(type: Copy) {
    from 'images'
    include '*.jpg'
    into 'build'
	}
我们可以定制多层级的CopySpecs，结合使用，比如，我们想在某文件夹下包含自定文件，而又要排除掉特定文件。而且来自同一个文件目录下的文件，也可以被分配到不同的文件下。比如下面的代码：

	task copyImageFolders(type: Copy) {
    from('images') {
        include '*.jpg'
        into 'jpeg'
    }

    from('images') {
        include '*.gif'
        into 'gif'
    }

    into 'build'
	}
分别将images下的jpg和gif放到build/jpeg,build/gif目录下。
<br>删除文件夹：
	
	task deleteBuild(type: Delete){
		delete 'build'
	}	
###文件归档
文件压缩为zip档：
	
	task zipImages(type: Zip){
		baseName = 'images'
   	   destinationDir = file('build')
       from 'images'
	}	
其中baseName是归档后的名字，from是归档源文件夹，destinationDir是目标文件存放的目录，引用了java的file对象来创建它。
<br>压缩后分别放入不同的子文件夹：
	
	task zipImageFolders(type: Zip) {
    baseName = 'images'
    destinationDir = file('build')
    from('images') {
        include '*.jpg'
        into 'jpeg'
    }

    from('images') {
        include '*.gif'
        into 'gif'
    }
	}	
##为Gradle脚本传参数	
在开发时，经常碰到一些情况，那就是脚本的部分内容需要改变。Gradle允许从脚本外的地方给project对象添加参数。主要有两种方法，一个是添加gradle.properties文件，另一个是命令行参数。
<br>我们先创建一个task:

	task printGreeting <<{
		println greeting
	}		
显然，直接运行上面的任务肯定报错，因为greeting这个属性没有被定义。那么我们想办法定义并传递这个参数给脚本对象：
	
	1.创建gradle.properties文件，然后在里面添加
	    greeting = "Hello from a properties file"
	2.在运行前，直接添加在命令行上:
	    gradle -Pgreeting = "Hello from a properties file" pG
当上面两种情况同时出现时，命令行的定义将覆盖文件中的定义。
<br>还有一种扩展属性的方法，如下:
	
	ext {//ext block可以立即设置多个属性值
		greeting = "Hello from inside the build script"
	}
	project.ext.prop1 = "foo"
	task doStuff {
    	ext.prop2 = "bar"
	}
	subprojects { ext.${prop3} = false }
	ext.isSnapshot = version.endsWith("-SNAPSHOT")
	if (isSnapshot) {
   	 // do snapshot stuff
	}
相当于动态生成属性，供其它地方调用。[参考资料](https://docs.gradle.org/current/userguide/writing_build_scripts.html#sec:extra_properties).
##自定义Task类型
自定义Task类型，就是继承并实现Task：
	
	class MyTask extends DefaultTask {}
其中DefaultTask实现了Task的接口，当然，MyTask只是一个定义的简单例子。
	
	class HelloTask extends DefaultTask{
		@TaskAction
		void doAction(){
			println 'Hello World'
		}
	}
	task hello(type:HelloTask)
我们以java方式定义了一个action方法，这样就像之前的action一样被执行。
	
	class HelloNameTask extends DefaultTask {
    String firstName

    @TaskAction
    void doAction() {
        println "Hello, $firstName"
    }
	}
	task hellName(type:HelloNameTask){
		firstName = 'Jeremy'
	}
上面的类直接定义了一个属性变量，然后在[配置阶段](https://docs.gradle.org/current/userguide/build_lifecycle.html)初始化它。
##Task日志输出
log在Gradle中相当于Ui界面，用来知道运行的情况。Gradle 定义了6个级别的log，输出方式就是log4j，过滤输出，会包含上一层的级别输出。有两个级别是Gradle特定的，QUIET和LIFECYCLE。

Level | Used for
------|----------
ERROR | Errormessages
QUIET |Important information messages
WARNING|Warning messages
LIFECYCLE	|Progress information messages
INFO |	Information messages
DEBUG	| Debug messages

在命令行上过滤日志输出：

Option	|Outputs Log Levels
--------|----------------
no logging options|	LIFECYCLE and higher
-q or --quiet	|QUIET and higher
-i or --info|	INFO and higher
-d or --debug	|DEBUG and higher (that is, all log messages)
运行出错时，添加-s or --stacktrace，或者 -S or --full-stacktrace来跟踪错误异常。
###输出日志
正常情况下，println的输出定向为标准输出，等级为QUIET。Gradle也提供了logger变量，来定制输出日志的等级：

	logger.quiet('An info log message which is always logged.')
	logger.error('An error log message.')
	logger.warn('A warning log message.')
	logger.lifecycle('A lifecycle info log message.')
	logger.info('An info log message.')
	logger.debug('A debug log message.')
	logger.trace('A trace log message.')
###重定向标准输出
	
	logging.captureStandardOutput LogLevel.INFO
	println 'A message which is logged at INFO level'
###定制Gradle logs输出格式
	
	useLogger(new CustomEventLogger())

	class CustomEventLogger extends BuildAdapter implements 	TaskExecutionListener {

    public void beforeExecute(Task task) {
        println "[$task.name]"
    }

    public void afterExecute(Task task, TaskState state) {
        println()
    }
    
    public void buildFinished(BuildResult result) {
        println 'build completed'
        if (result.failure != null) {
            result.failure.printStackTrace()
        }
    }
	}
实现BuildAdapter,TaskExecutionListener后，设置到Gradle.useLogger()方法中来替换默认实现。[参考资料](https://docs.gradle.org/current/userguide/logging.html)	.
##Build Lifecycle
在build过程中，由三部分构成
	
	Initialization，Gradle支持单个和多个项目的构建，那么在这个阶段，gradle决定哪些project进入这个构建过程，然后为每个项目生成一个project对象。
	Configuration，在这个阶段，所有包含的projects会运行配置部分的代码。
	Execution，gradle执行在配置阶段生成和配置的tasks执行action。
###Initailization
除了build.gradle文件，还有一个settings文件，在多项目构建中，在root project中必须要设置一个settings.gradle文件，然后在里面定义要build的projects。
settings.gradle
	
	println 'This is executed during the initialization phase.'
build.gradle
	
	println 'This is executed during the configuration phase.begin'

	task configured {
    println 'This is also executed during the configuration phase.'
	}

	task test << {
    println 'This is executed during the execution phase.'
	}

	task testBoth {
    doFirst {
      println 'This is executed first during the execution phase.'
    }
    doLast {
      println 'This is executed last during the execution phase.'
    }
    println 'This is executed during the configuration phase as well.'
	}
		println 'This is executed during the configuration phase.end'

输出结果：

	>gradle test testBoth
	This is executed during the initialization phase.
    This is executed during the configuration phase.begin
	This is also executed during the configuration phase.
	This is executed during the configuration phase as well.
	This is executed during the configuration phase.end
	:test
	This is executed during the execution phase.
	:testBoth
	This is executed first during the execution phase.
	This is executed last during the execution phase.

	BUILD SUCCESSFUL

	Total time: 1 secs			
以上可以看出，虽然configured task没有执行，但是它的配置闭包还是会被执行一次。
<br>其他的部分，查看[文档](https://docs.gradle.org/current/userguide/build_lifecycle.html).	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
