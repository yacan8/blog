---
title: Java使用Nashorn，调用Promise实现服务端渲染
date: 2018-01-20 15:21:41
tags:
- java
- nashorn
- promise
---

Nashorn JavaScript引擎是Java SE 8 的一部分，并且和其它独立的引擎例如 [Google V8](https://code.google.com/p/v8/)（用于Google Chrome和Node.js的引擎）互相竞争。Nashorn通过在JVM上，以原生方式运行动态的JavaScript代码来扩展Java的功能。
## 使用Nashorn
Java代码中简单的HelloWorld如下所示：
```java
ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");
engine.eval("print('Hello World!');");
```
为了在Java中执行JavaScript，你首先要通过javax.script包创建脚本引擎。
JavaScript代码既可以通过传递JavaScript代码字符串，也可以传递指向你的JS脚本文件的`FileReader`来执行：
```java
ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");
engine.eval(new FileReader("script.js"));
```
Nashorn JavaScript基于 [ECMAScript 5.1](http://es5.github.io/)，但是它的后续版本会对ES6提供支持：
> Nashorn的当前策略遵循ECMAScript规范。当我们在JDK8中发布它时，它将基于ECMAScript 5.1。Nashorn未来的主要发布基于ECMAScript 6。
## 在Java中调用JavaScript函数
Nashorn 支持从Java代码中直接调用定义在脚本文件中的JavaScript函数。你可以将Java对象传递为函数参数，并且从函数返回数据来调用Java方法。
```js
var fun1 = function(name) {
    print('Hi there from Javascript, ' + name);
    return "greetings from javascript";
};

var fun2 = function (object) {
    print("JS Class Definition: " + Object.prototype.toString.call(object));
};
```
`ScriptEngine`内置了`invokeFunction`方法，提调用javascript函数并返回结果。
```java
ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");
engine.eval(new FileReader("script.js"));

Object result = invocable.invokeFunction("fun1", "Peter Parker");
System.out.println(result);
System.out.println(result.getClass());

// Hi there from Javascript, Peter Parker
// greetings from javascript
// class java.lang.String
```
##在JavaScript中调用Java方法
在JavaScript中调用Java方法十分容易。我们首先需要定义一个Java静态方法。
```java
static String fun1(String name) {
    System.out.format("Hi there from Java, %s", name);
    return "greetings from java";
}
```
Java类可以通过`Java.type`API扩展在JavaScript中引用。它就和Java代码中的import类似。只要定义了Java类型，我们就可以自然地调用静态方法fun1()，然后像sout打印信息。由于方法是静态的，我们不需要首先创建实例。
```js
var MyJavaClass = Java.type('my.package.MyJavaClass');

var result = MyJavaClass.fun1('John Doe');
print(result);

// Hi there from Java, John Doe
// greetings from java
```
在使用JavaScript原生类型调用Java方法时，Nashorn 如何处理类型转换？让我们通过简单的例子来弄清楚。
下面的Java方法简单打印了方法参数的实际类型：
```java
static void fun2(Object object) {
    System.out.println(object.getClass());
}
```
为了理解背后如何处理类型转换，我们使用不同的JavaScript类型来调用这个方法：
```js
MyJavaClass.fun2(123);
// class java.lang.Integer

MyJavaClass.fun2(49.99);
// class java.lang.Double

MyJavaClass.fun2(true);
// class java.lang.Boolean

MyJavaClass.fun2("hi there")
// class java.lang.String

MyJavaClass.fun2(new Number(23));
// class jdk.nashorn.internal.objects.NativeNumber

MyJavaClass.fun2(new Date());
// class jdk.nashorn.internal.objects.NativeDate

MyJavaClass.fun2(new RegExp());
// class jdk.nashorn.internal.objects.NativeRegExp

MyJavaClass.fun2({foo: 'bar'});
// class jdk.nashorn.internal.scripts.JO4
```
JavaScript原始类型转换为合适的Java包装类，而JavaScript原生对象会使用内部的适配器类来表示。要记住`jdk.nashorn.internal`中的类可能会有所变化，所以不应该在客户端面向这些类来编程。
##ScriptObjectMirror
在向Java传递原生JavaScript对象时，你可以使用`ScriptObjectMirror`类，它实际上是底层JavaScript对象的Java表示。`ScriptObjectMirror`实现了`Map`接口，位于`jdk.nashorn.api`中。这个包中的类可以用于客户端代码。

下面的例子将参数类型从`Object`改为`ScriptObjectMirror`，所以我们可以从传入的JavaScript对象中获得一些信息。
```java
static void fun3(ScriptObjectMirror mirror) {
    System.out.println(mirror.getClassName() + ": " +
        Arrays.toString(mirror.getOwnKeys(true)));
}
```
当向这个方法传递对象（哈希表）时，在Java端可以访问其属性：
```js
MyJavaClass.fun3({
    foo: 'bar',
    bar: 'foo'
});

// Object: [foo, bar]
```
我们也可以在Java中调用JavaScript的成员函数。让我们首先定义JavaScript `Person`类型，带有属性`firstName` 和 `lastName`，以及方法`getFullName`。
```js
function Person(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.getFullName = function() {
        return this.firstName + " " + this.lastName;
    }
}
```
JavaScript方法`getFullName`可以通过`callMember()`在`ScriptObjectMirror`上调用。
```java
static void fun4(ScriptObjectMirror person) {
    System.out.println("Full Name is: " + person.callMember("getFullName"));
}
```
当向Java方法传递新的`Person`时，我们会在控制台看到预期的结果：
```
var person1 = new Person("Peter", "Parker");
MyJavaClass.fun4(person1);

// Full Name is: Peter Parker
```
## 实战
通过使用promise实现服务端渲染。
### polyfill
由于当前Nashorn基于es5，不支持部分es6对象，我们需要引入polyfill文件，[nashorn-polyfill](https://www.npmjs.com/package/nashorn-polyfill)。
该polyfill通过java与JavaScript的结合使用，使Nashorn支持`console`, `process`, `Blob`, `Promise`等对象和`setTimeout`, `clearTimeout`, `setInterval`, `clearInterval`等函数。
### Nashorn工具类
实例化工具类，通过该类操作Javascript对象。
```java
public class NashornHelper {

    /**
     * 用于本类的日志
     */
    private static final Logger       logger                       = LoggerFactory.getLogger(NashornHelper.class);

    private static final String       JAVASCRIPT_DIR               = "static"; // js文件目录

    private static final String       LIB_DIR                      = JAVASCRIPT_DIR + File.separator + "lib";

    private static final String[]     VENDOR_FILE_NAME             = {"vendor.js"}; // webpack打包的三方库，如react，lodash

    private static final String       SRC_DIR                      = JAVASCRIPT_DIR + File.separator + "src"; // 文件目录

    private static final String       POLYFILL_FILE_NAME           = "nashorn-polyfill.js";

    private final NashornScriptEngine engine;

    private static NashornHelper      nashornHelper;

    private static ScriptContext            sc                        = new SimpleScriptContext();

    private static ScheduledExecutorService globalScheduledThreadPool = Executors.newScheduledThreadPool(20);

    // 单例模式
    public static synchronized NashornHelper getInstance() {
        if (nashornHelper == null) {
            long start = System.currentTimeMillis();
            nashornHelper = new NashornHelper();
            long end = System.currentTimeMillis();
            logger.error("init nashornHelper cost time {}ms", (end - start));
        }
        return nashornHelper;
    }

    private NashornHelper(){
        long start = System.currentTimeMillis();
        engine = (NashornScriptEngine) new ScriptEngineManager().getEngineByName("nashorn");
        Bindings bindings = new SimpleBindings();
        bindings.put("logger", logger); // 向nashorn引擎注入logger对象
        sc.setBindings(engine.createBindings(), ScriptContext.ENGINE_SCOPE);
        sc.getBindings(ScriptContext.ENGINE_SCOPE).putAll(bindings);
        sc.setAttribute("__IS_SSR__", true, ScriptContext.ENGINE_SCOPE);
        sc.setAttribute("__NASHORN_POLYFILL_TIMER__", globalScheduledThreadPool, ScriptContext.ENGINE_SCOPE);
        engine.setBindings(sc.getBindings(ScriptContext.ENGINE_SCOPE), ScriptContext.ENGINE_SCOPE);

        long end = System.currentTimeMillis();
        logger.info("init nashornHelper cost time {}ms", (end - start));

        try {  // 执行js文件
            engine.eval(read(LIB_DIR + File.separator + POLYFILL_FILE_NAME));
            for (String fileName : NashornHelper.VENDOR_FILE_NAME) {
                engine.eval(read(SRC_DIR + File.separator + fileName));
            }
            engine.eval(read(SRC_DIR + File.separator + "app.js"));
        } catch (ScriptException e) {
            logger.error("load javascript failed.", e);
        }
    }
    // 获取Nashorn作用域下的对象
    public ScriptObjectMirror getGlobalGlobalMirrorObject(String objectName) {
        return (ScriptObjectMirror) engine.getBindings(ScriptContext.ENGINE_SCOPE).get(objectName);
    }
    // 调用全局方法
    public Object callRender(String methodName, Object... input) {
        try {
            return engine.invokeFunction(methodName, input);
        } catch (ScriptException e) {
            logger.error("run javascript failed.", e);
            return null;
        } catch (NoSuchMethodException e) {
            logger.error("no such method.", e);
            return null;
        }
    }
    // 读取文件
    private Reader read(String path) {
        InputStream in = getClass().getClassLoader().getResourceAsStream(path);
        return new InputStreamReader(in);
    }
```
实例化工具类
```java
NashornHelper engine = NashornHelper.getInstance();
```
执行调用javasript，java中获取`promise`对象
```java
ScriptObjectMirror promise = (ScriptObjectMirror) engine.callRender("ssr_render");
```
执行`promise`的`then`方法，等待执行完成并回调
```java
    promise.callMember("then", fnResolve);
    ScriptObjectMirror nashornEventLoop = engine.getGlobalGlobalMirrorObject("nashornEventLoop");

    nashornEventLoop.callMember("process"); // 执行nashornEventLoops.process()使主线程执行回调函数
    int i = 0;
    int jsWaitTimeout = 1000 * 60;
    int interval = 200; // 等待时间间隔
    int totalWaitTime = 0; // 实际等待时间
    while (!promiseResolved && totalWaitTime < jsWaitTimeout) {
        nashornEventLoop.callMember("process");
        try {
            Thread.sleep(interval);
        } catch (InterruptedException e) {
        }
        totalWaitTime = totalWaitTime + interval;
        if (interval < 500) interval = interval * 2;
        i = i + 1;
    }
```
回调函数
```java
  private Consumer<Object> fnResolve = object -> {
    synchronized (promiseLock) {
        html = (String) object;
        promiseResolved = true;
    }
  };
```
最后结果已字符串形式存在`html`中，可将其渲染到页面中.
## 最后
适用场景非浏览器渲染页面，如java离线渲染前端页面到Pdf。
实战 [github地址](https://github.com/yacan8/react-ssr-demo), [前端js项目地址](https://github.com/yacan8/react-mobx-todo)
