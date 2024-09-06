# java - Lombok 原理与实现 - Sumkor's Blog - SegmentFault 思否
本文主要包含以下内容：

1.  Lombok 的实现机制分析。
2.  插入式注解处理器的说明及使用。
3.  动手实现 lombok 的 @Getter 和 @Setter 注解。
4.  配置 IDEA 以调试 Java 编译过程。

1\. Lombok
----------

官网介绍：

> Project Lombok is a java library that automatically plugs into your editor and build tools, spicing up your java.  
> Never write another getter or equals method again, with one annotation your class has a fully featured builder, Automate your logging variables, and much more.

在 Maven 中添加 Lombok 依赖：

```
<dependency\>
    <groupId\>org.projectlombok</groupId\>
    <artifactId\>lombok</artifactId\>
    <version\>1.18.20</version\>
    <scope\>provided</scope\>
</dependency\>
```

简单的例子如下，通过添加注解的方式，自动生成 getter/setter 方法。

```
package com.sumkor;

import lombok.Getter;
import lombok.Setter;

@Setter
@Getter
public class MyTest {

    private String value;

    public static void main(String\[\] args) {
        MyTest myTest \= new MyTest();
        myTest.setValue("hello");
        System.out.println(myTest.getValue());
    }
}
```

编译后的代码如下：

```
package com.sumkor;

public class MyTest {
    private String value;

    public MyTest() {
    }

    public static void main(String\[\] args) {
        MyTest myTest \= new MyTest();
        myTest.setValue("hello");
        System.out.println(myTest.getValue());
    }

    public void setValue(String value) {
        this.value = value;
    }

    public String getValue() {
        return this.value;
    }
}
```

2\. Annotation Processor
------------------------

### 2.1 Javac 编译器

《深入理解 Java 虚拟机：JVM 高级特性与最佳实践》中的第 10 章《前端编译与优化》对 Javac 编译器进行了介绍。

Javac 编译过程如下：

> 从 Javac 代码的总体结构来看，编译过程大致可以分为 1 个准备过程和 3 个处理过程，它们分别如下所示。
> 
> 1.  准备过程：初始化插入式注解处理器。
> 2.  解析与填充符号表过程，包括：词法、语法分析；填充符号表。
> 3.  插入式注解处理器的注解处理过程。
> 4.  分析与字节码生成过程。

![](https://segmentfault.com/img/remote/1460000041200282)

重点关注对插入式注解处理器的说明：

> JDK 5 之后，Java 语言提供了对注解（Annotations）的支持，注解在设计上原本是与普通的 Java 代码一样，都只会在程序运行期间发挥作用的。但在 JDK 6 中又提出并通过了 JSR-269 提案，该提案设计了一组被称为“插入式注解处理器”的标准 API，可以提前至编译期对代码中的特定注解进行处理，从而影响到前端编译器的工作过程。我们可以把插入式注解处理器看作是一组编译器的插件，当这些插件工作时，允许读取、修改、添加抽象语法树中的任意元素。如果这些插件在处理注解期间对语法树进行过修改，编译器将回到解析及填充符号表的过程重新处理，直到所有插入式注解处理器都没有再对语法树进行修改为止。

可以看到 Lombok 是基于插入式注解处理器来实现：

> 有了编译器注解处理的标准 API 后，程序员的代码才有可能干涉编译器的行为，由于语法树中的任意元素，甚至包括代码注释都可以在插件中被访问到，所以通过插入式注解处理器实现的插件在功能上有很大的发挥空间。只要有足够的创意，程序员能使用插入式注解处理器来实现许多原本只能在编码中由人工完成的事情。譬如 Java 著名的编码效率工具 Lombok，它可以通过注解来实现自动产生 getter/setter 方法、进行空置检查、生成受查异常表、产生 equals() 和 hashCode() 方法，等等，帮助开发人员消除 Java 的冗长代码，这些都是依赖插入式注解处理器来实现的。

### 2.2 Java 注解

Java 中的注解分为运行时注解和编译时注解，通过设置元注解 @Retention 中的 RetentionPolicy 指定注解的保留策略：

```
public enum RetentionPolicy {
    
    SOURCE,

    
    CLASS,

    
    RUNTIME
}
```

说明：

*   SOURCE：表示注解的信息会被编译器抛弃，不会留在 class 文件中，注解的信息只会留在源文件中。
*   CLASS：表示注解的信息被保留在 class 文件中。当程序编译时，但不会被虚拟机读取在运行的时候。
*   RUNTIME：表示注解的信息被保留在 class 文件中。当程序编译时，会被虚拟机保留在运行时。

日常开发中使用的注解都是 RUNTIME 类型的，可以在运行期被反射调用读取。

javax.annotation.Resource

```
@Target({TYPE, FIELD, METHOD})
@Retention(RUNTIME)
public @interface Resource
```

而 Lombok 中的注解是 SOURCE 类型的，只会在编译期间被插入式注解处理器读取。

lombok.Getter

```
@Target({ElementType.FIELD, ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface Getter
```

### 2.3 插入式注解处理器

自定义的注解处理器需要继承 AbstractProcessor 这个类，基本的框架大体如下：

```
package com.sumkor.processor;

import javax.annotation.processing.\*;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.TypeElement;
import java.util.Set;

@SupportedAnnotationTypes("com.sumkor.annotation.Getter")
@SupportedSourceVersion(SourceVersion.RELEASE\_8)
public class GetterProcessor extends AbstractProcessor {

    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        super.init(processingEnv);
    }

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        return true;
    }
}
```

其中：

*   `@SupportedAnnotationTypes` 代表了这个注解处理器对哪些注解感兴趣，可以使用星号`*`作为通配符代表对所有的注解都感兴趣。
*   `@SupportedSourceVersion` 指出这个注解处理器可以处理哪些版本的 Java 代码。
*   `init()` 用于获取编译阶段的一些环境信息。
*   `process()` 可以编写处理语法树的具体逻辑。如果不需要改变或添加抽象语法树中的内容，process() 方法就可以返回一个值为 false 的布尔值，通知编译器这个轮次中的代码未发生变化。

本文后续会实现具体的自定义的插入式注解处理器，并进行打断点调试。

### 2.4 Javac APT

利用插入式注解处理器在编译阶段修改语法树，需要用到 Javac 中的注解处理工具 APT(Annotation Processing Tool)，这是 Sun 为了帮助注解的处理过程而提供的工具，APT 被设计为操作 Java 源文件，而不是编译后的类。

本文使用的是 JDK 8，Javac 相关的源码存放在 tools.jar 中，要在程序中使用的话就必须把这个库放到类路径上。注意，到了 JDK 9 时，整个 JDK 所有的 Java 类库都采用模块化进行重构划分，Javac 编译器就被挪到了 jdk.compiler 模块，并且对该模块的访问进行了严格的限制。

![](https://segmentfault.com/img/remote/1460000041200283)

### 2.5 JCTree 语法树

com.sun.tools.javac.tree.JCTree 是语法树元素的基类，包含以下重要的子类：

*   JCStatement：声明语法树节点，常见的子类如下
    
    *   JCBlock：语句块语法树节点
    *   JCReturn：return语句语法树节点
    *   JCClassDecl：类定义语法树节点
    *   JCVariableDecl：字段/变量定义语法树节点
*   JCMethodDecl：方法定义语法树节点
*   JCModifiers：访问标志语法树节点
*   JCExpression：表达式语法树节点，常见的子类如下
    
    *   JCAssign：赋值语句语法树节点
    *   JCIdent：标识符语法树节点，可以是变量，类型，关键字等

JCTree 利用的是访问者模式，将数据与数据的处理进行解耦。部分源码如下：

```
public abstract class JCTree implements Tree, Cloneable, DiagnosticPosition {

    public int pos \= -1;

    public abstract void accept(JCTree.Visitor visitor);

}
```

利用访问者 TreeTranslator，可以访问 JCTree 上的类定义节点 JCClassDecl，进而可以获取类中的成员变量、方法等节点并进行修改。

编码过程中，可以利用 javax.annotation.processing.Messager 来打印编译过程的相关信息。  
注意，Messager 的 printMessage 方法在打印 log 的时候会自动过滤重复的 log 信息。

比起打印日志，利用 IDEA 工具对编译过程进行 debug，对 JCTree 语法树会有更为直观的认识。  
文末提供了在 IDEA 中调试插入式注解处理器的配置。

3\. 动手实现
--------

分别创建两个项目，用于实现和验证 @Getter 和 @Setter 注解。

创建项目 lombok-processor，包含自定义注解和插入式注解处理器。  
创建项目 lombok-app，该项目依赖了 lombok-processor 项目，使用其中的自定义注解进行测试。

### 3.1 processor 项目

项目整体结构如下：  
![](https://segmentfault.com/img/remote/1460000041200284)

#### Maven 配置

由于需要在编译阶段修改 Java 语法树，需要调用语法树相关的 API，因此将 JDK 目录下的 tools.jar 引入当前项目。

```
<dependency\>
    <groupId\>com.sun</groupId\>
    <artifactId\>tools</artifactId\>
    <version\>1.8</version\>
    <scope\>system</scope\>
    <systemPath\>${java.home}/../lib/tools.jar</systemPath\>
</dependency\>
```

lombok-processor 项目采用 Java SPI 机制，使其自定义的插入式注解处理器对 lombok-app 项目生效。由于 lombok-processor 项目在编译期间需要排除掉自身的插入式注解处理器，因此配置 maven resource 以过滤掉 SPI 文件，等到打包的时候，再将 SPI 文件加入 lombok-processor 项目的 jar 包中。  
此外，为了方便调试，将 lombok-processor 项目的源码也发布到本地仓库中。

完整的 maven build 配置如下：

```
<build\>
    
    <resources\>
        <resource\>
            <directory\>src/main/resources</directory\>
            <excludes\>
                <exclude\>META-INF/\*\*/\*</exclude\>
            </excludes\>
        </resource\>
    </resources\>

    <plugins\>
        <plugin\>
            <groupId\>org.apache.maven.plugins</groupId\>
            <artifactId\>maven-compiler-plugin</artifactId\>
            <version\>3.1</version\>
            <configuration\>
                <source\>${maven.compiler.target}</source\>
                <target\>${maven.compiler.target}</target\>
            </configuration\>
        </plugin\>
        
        <plugin\>
            <groupId\>org.apache.maven.plugins</groupId\>
            <artifactId\>maven-resources-plugin</artifactId\>
            <version\>2.6</version\>
            <executions\>
                <execution\>
                    <id\>process-META</id\>
                    <phase\>prepare-package</phase\>
                    <goals\>
                        <goal\>copy-resources</goal\>
                    </goals\>
                    <configuration\>
                        <outputDirectory\>target/classes</outputDirectory\>
                        <resources\>
                            <resource\>
                                <directory\>${basedir}/src/main/resources/</directory\>
                                <includes\>
                                    <include\>\*\*/\*</include\>
                                </includes\>
                            </resource\>
                        </resources\>
                    </configuration\>
                </execution\>
            </executions\>
        </plugin\>
        
        <plugin\>
            <groupId\>org.apache.maven.plugins</groupId\>
            <artifactId\>maven-source-plugin</artifactId\>
            <version\>3.0.1</version\>
            <configuration\>
                <attach\>true</attach\>
            </configuration\>
            <executions\>
                <execution\>
                    <phase\>compile</phase\>
                    <goals\>
                        <goal\>jar</goal\>
                    </goals\>
                </execution\>
            </executions\>
        </plugin\>
    </plugins\>
</build\>
```

#### 自定义注解

自定义注解主要使用了两个元注解：

*   `@Target({ElementType.TYPE})` 表示是对类的注解。
*   `@Retention(RetentionPolicy.SOURCE)`表示这个注解只在编译期起作用，在运行时将不存在。

自定义 @Getter 注解：

```
package com.sumkor.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.TYPE}) 
@Retention(RetentionPolicy.SOURCE) 
public @interface Getter {
}
```

自定义 @Setter 注解：

```
package com.sumkor.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.SOURCE)
public @interface Setter {
}
```

#### 自定义注解处理器

##### BaseProcessor

定义抽象的基类 BaseProcessor，用于统一获取编译阶段的工具类，如 JavacTrees、TreeMaker 等。  
由于本项目需要在 IDEA 中进行调试和运行，因此引入 IDEA 环境的 ProcessingEnvironment。

```
package com.sumkor.processor;

import com.sun.tools.javac.api.JavacTrees;
import com.sun.tools.javac.processing.JavacProcessingEnvironment;
import com.sun.tools.javac.tree.TreeMaker;
import com.sun.tools.javac.util.Context;
import com.sun.tools.javac.util.Names;

import javax.annotation.processing.AbstractProcessor;
import javax.annotation.processing.Messager;
import javax.annotation.processing.ProcessingEnvironment;
import java.lang.reflect.Method;

public abstract class BaseProcessor extends AbstractProcessor {

    protected Messager messager;   
    protected JavacTrees trees;    
    protected TreeMaker treeMaker; 
    protected Names names;         

    
    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        processingEnv = jbUnwrap(ProcessingEnvironment.class, processingEnv);
        super.init(processingEnv);
        this.messager = processingEnv.getMessager();
        this.trees = JavacTrees.instance(processingEnv);
        Context context \= ((JavacProcessingEnvironment) processingEnv).getContext();
        this.treeMaker = TreeMaker.instance(context);
        this.names = Names.instance(context);
    }

    
    private static <T> T jbUnwrap(Class<? extends T> iface, T wrapper) {
        T unwrapped \= null;
        try {
            final Class<?> apiWrappers = wrapper.getClass().getClassLoader().loadClass("org.jetbrains.jps.javac.APIWrappers");
            final Method unwrapMethod \= apiWrappers.getDeclaredMethod("unwrap", Class.class, Object.class);
            unwrapped = iface.cast(unwrapMethod.invoke(null, iface, wrapper));
        }
        catch (Throwable ignored) {}
        return unwrapped != null? unwrapped : wrapper;
    }

}
```

##### GetterProcessor

自定义注解 @Getter 对应的注解处理器如下，代码流程：

1.  获取被 @Getter 注解修饰的类。
2.  找到类上的所有成员变量。
3.  为成员变量构造 getter 方法。

难点在于对 JavacTrees 和 TreeMaker 相关 API 的使用上，文中关键代码均有注释，方便理解。

```
package com.sumkor.processor;

import com.sumkor.annotation.Getter;
import com.sun.source.tree.Tree;
import com.sun.tools.javac.code.Flags;
import com.sun.tools.javac.tree.JCTree;
import com.sun.tools.javac.tree.TreeTranslator;
import com.sun.tools.javac.util.List;
import com.sun.tools.javac.util.ListBuffer;
import com.sun.tools.javac.util.Name;

import javax.annotation.processing.ProcessingEnvironment;
import javax.annotation.processing.RoundEnvironment;
import javax.annotation.processing.SupportedAnnotationTypes;
import javax.annotation.processing.SupportedSourceVersion;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.Element;
import javax.lang.model.element.TypeElement;
import javax.tools.Diagnostic;
import java.util.Set;

@SupportedAnnotationTypes("com.sumkor.annotation.Getter")
@SupportedSourceVersion(SourceVersion.RELEASE\_8)
public class GetterProcessor extends BaseProcessor {

    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        super.init(processingEnv);
        messager.printMessage(Diagnostic.Kind.NOTE, "========= GetterProcessor init =========");
    }

    
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        
        Set<? extends Element\> set = roundEnv.getElementsAnnotatedWith(Getter.class);
        set.forEach(element -> {
            
            JCTree jcTree \= trees.getTree(element);
            jcTree.accept(new TreeTranslator() {
                
                @Override
                public void visitClassDef(JCTree.JCClassDecl jcClassDecl) {
                    List<JCTree.JCVariableDecl> jcVariableDeclList = List.nil();
                    for (JCTree tree : jcClassDecl.defs) {
                        
                        if (tree.getKind().equals(Tree.Kind.VARIABLE)) {
                            JCTree.JCVariableDecl jcVariableDecl \= (JCTree.JCVariableDecl) tree;
                            jcVariableDeclList = jcVariableDeclList.append(jcVariableDecl);
                        }
                    }
                    
                    jcVariableDeclList.forEach(jcVariableDecl -> {
                        messager.printMessage(Diagnostic.Kind.NOTE, jcVariableDecl.getName() + " has been processed");
                        jcClassDecl.defs = jcClassDecl.defs.prepend(makeGetterMethodDecl(jcVariableDecl));
                    });
                    super.visitClassDef(jcClassDecl);
                }

            });
        });
        return true;
    }

    
    private JCTree.JCMethodDecl makeGetterMethodDecl(JCTree.JCVariableDecl jcVariableDecl) {
        ListBuffer<JCTree.JCStatement> statements = new ListBuffer<>();
        
        statements.append(treeMaker.Return(treeMaker.Select(treeMaker.Ident(names.fromString("this")), jcVariableDecl.getName())));
        
        JCTree.JCBlock body \= treeMaker.Block(0, statements.toList());
        
        return treeMaker.MethodDef(treeMaker.Modifiers(Flags.PUBLIC), getNewMethodName(jcVariableDecl.getName()), jcVariableDecl.vartype, List.nil(), List.nil(), List.nil(), body, null);
    }

    
    private Name getNewMethodName(Name name) {
        String s \= name.toString();
        return names.fromString("get" + s.substring(0, 1).toUpperCase() + s.substring(1, name.length()));
    }

}
```

##### SetterProcessor

自定义注解 @Setter 对应的注解处理器 SetterProcessor，整体流程与 GetterProcessor 差不多：

1.  获取被 @Setter 注解修饰的类。
2.  找到类上的所有成员变量。
3.  为成员变量构造 setter 方法。

重点关注构造 setter 方法的逻辑：

```
private JCTree.JCMethodDecl makeSetterMethodDecl(JCTree.JCVariableDecl jcVariableDecl) {
    ListBuffer<JCTree.JCStatement> statements = new ListBuffer<>();
    
    JCTree.JCExpressionStatement aThis \= makeAssignment(treeMaker.Select(treeMaker.Ident(names.fromString("this")), jcVariableDecl.getName()), treeMaker.Ident(jcVariableDecl.getName()));
    statements.append(aThis);
    
    JCTree.JCBlock block \= treeMaker.Block(0, statements.toList());

    
    treeMaker.pos = jcVariableDecl.pos;

    
    JCTree.JCVariableDecl param \= treeMaker.VarDef(treeMaker.Modifiers(Flags.PARAMETER), jcVariableDecl.getName(), jcVariableDecl.vartype, null);
    List<JCTree.JCVariableDecl> parameters = List.of(param);
    
    JCTree.JCExpression methodType \= treeMaker.Type(new Type.JCVoidType());

    
    return treeMaker.MethodDef(treeMaker.Modifiers(Flags.PUBLIC), getNewMethodName(jcVariableDecl.getName()), methodType, List.nil(), parameters, List.nil(), block, null);
}

private JCTree.JCExpressionStatement makeAssignment(JCTree.JCExpression lhs, JCTree.JCExpression rhs) {
    return treeMaker.Exec(
            treeMaker.Assign(lhs, rhs)
    );
}

private Name getNewMethodName(Name name) {
    String s \= name.toString();
    return names.fromString("set" + s.substring(0, 1).toUpperCase() + s.substring(1, name.length()));
}
```

### 3.2 app 项目

项目整体结构如下，这里 IDEA 的标红提示是由于无法识别自定义注解导致的，不影响项目运行。  
![](https://segmentfault.com/img/remote/1460000041200285)
  
在 maven 中引入 lombok-processor 项目：

```
<dependency\>
    <groupId\>com.sumkor</groupId\>
    <artifactId\>lombok-processor</artifactId\>
    <version\>1.0-SNAPSHOT</version\>
</dependency\>
```

编写测试类，引入自定义的注解 @Getter 和 @Setter，可以看到 IDEA 虽然标红提示找不到方法，但是可以正常通过编译和运行。  
![](https://segmentfault.com/img/remote/1460000041200286)

4\. 调试
------

### 4.1 IDEA 配置

使用 Attach Remote JVM 的方式，对 lombok-app 项目的编译过程进行调试。

1.  创建一个远程调试，指定端口为 5005。

```
\-agentlib:jdwp=transport\=dt\_socket,server=y,suspend=n,address=5005
```

![](https://segmentfault.com/img/remote/1460000041200287)

2.  在菜单中选择：Help -> Edit Custom VM Options，添加以下内容并重启 IDEA。这里的端口是上一步中选择需要 attach 的端口。

```
\-Dcompiler.process.debug.port\=5005
```

3.  打开 IDEA 的 Debug Build Process。注意，这个选项在每次 IDEA 重启后都会默认关闭。

![](https://segmentfault.com/img/remote/1460000041200288)

### 4.2 开始调试

1.  在注解处理器中添加断点，推荐断点打在 AbstractProcessor#init 中。
2.  执行 mvn clean 操作，清除掉上一次构建完成的内容。
3.  执行 ctrl + F9 操作进行 lombok-app 项目构建。  
    ![](https://segmentfault.com/img/remote/1460000041200289)
      
    在状态栏可以看到构建过程在等待 debugger 连接：  
    ![](https://segmentfault.com/img/remote/1460000041200290)
    
4.  运行刚才创建的远程调试。  
    ![](https://segmentfault.com/img/remote/1460000041200291)
      
    可以看到进入了 AbstractProcessor#init 中的断点。  
    ![](https://segmentfault.com/img/remote/1460000041200292)
    对自定义的注解处理器进行调试，观察 TreeMaker 对语法树的修改结果。  
    ![](https://segmentfault.com/img/remote/1460000041200293)
    
    ### 4.3 问题解决
    

在编写 SetterProcessor#process 方法的时候，如果缺少 `treeMaker.pos = jcVariableDecl.pos;` 这一行代码，在编译过程会报错提示：`java.lang.AssertionError: Value of x -1`，具体编译信息如下：

```
Executing pre-compile tasks...
Loading Ant configuration...
Running Ant tasks...
Running 'before' tasks
Checking sources
Copying resources... \[lombok\-app\]
Parsing java... \[lombok\-app\]
java: ========= GetterProcessor init =========
java: value has been processed
java: ========= SetterProcessor init =========
java: 编译器 (1.8.0\_91) 中出现异常错误。如果在 Bug Database (http:
java: java.lang.AssertionError: Value of x -1
java:     at com.sun.tools.javac.util.Assert.error(Assert.java:133)
java:     at com.sun.tools.javac.util.Assert.check(Assert.java:94)
java:     at com.sun.tools.javac.util.Bits.incl(Bits.java:186)
java:     at com.sun.tools.javac.comp.Flow$AssignAnalyzer.initParam(Flow.java:1858)
java:     at com.sun.tools.javac.comp.Flow$AssignAnalyzer.visitMethodDef(Flow.java:1807)
java:     at com.sun.tools.javac.tree.JCTree$JCMethodDecl.accept(JCTree.java:778)
java:     at com.sun.tools.javac.tree.TreeScanner.scan(TreeScanner.java:49)
java:     at com.sun.tools.javac.comp.Flow$BaseAnalyzer.scan(Flow.java:404)
java:     at com.sun.tools.javac.comp.Flow$AssignAnalyzer.scan(Flow.java:1382)
java:     at com.sun.tools.javac.comp.Flow$AssignAnalyzer.visitClassDef(Flow.java:1749)
java:     at com.sun.tools.javac.tree.JCTree$JCClassDecl.accept(JCTree.java:693)
java:     at com.sun.tools.javac.comp.Flow$AssignAnalyzer.analyzeTree(Flow.java:2446)
java:     at com.sun.tools.javac.comp.Flow$AssignAnalyzer.analyzeTree(Flow.java:2429)
java:     at com.sun.tools.javac.comp.Flow.analyzeTree(Flow.java:211)
java:     at com.sun.tools.javac.main.JavaCompiler.flow(JavaCompiler.java:1327)
java:     at com.sun.tools.javac.main.JavaCompiler.flow(JavaCompiler.java:1296)
java:     at com.sun.tools.javac.main.JavaCompiler.compile2(JavaCompiler.java:901)
java:     at com.sun.tools.javac.main.JavaCompiler.compile(JavaCompiler.java:860)
java:     at com.sun.tools.javac.main.Main.compile(Main.java:523)
java:     at com.sun.tools.javac.api.JavacTaskImpl.doCall(JavacTaskImpl.java:129)
java:     at com.sun.tools.javac.api.JavacTaskImpl.call(JavacTaskImpl.java:138)
java:     at org.jetbrains.jps.javac.JavacMain.compile(JavacMain.java:238)
java:     at org.jetbrains.jps.incremental.java.JavaBuilder.lambda$compileJava$2(JavaBuilder.java:514)
java:     at org.jetbrains.jps.incremental.java.JavaBuilder.invokeJavac(JavaBuilder.java:560)
java:     at org.jetbrains.jps.incremental.java.JavaBuilder.compileJava(JavaBuilder.java:512)
java:     at org.jetbrains.jps.incremental.java.JavaBuilder.compile(JavaBuilder.java:355)
java:     at org.jetbrains.jps.incremental.java.JavaBuilder.doBuild(JavaBuilder.java:280)
java:     at org.jetbrains.jps.incremental.java.JavaBuilder.build(JavaBuilder.java:234)
java:     at org.jetbrains.jps.incremental.IncProjectBuilder.runModuleLevelBuilders(IncProjectBuilder.java:1485)
java:     at org.jetbrains.jps.incremental.IncProjectBuilder.runBuildersForChunk(IncProjectBuilder.java:1123)
java:     at org.jetbrains.jps.incremental.IncProjectBuilder.buildTargetsChunk(IncProjectBuilder.java:1268)
java:     at org.jetbrains.jps.incremental.IncProjectBuilder.buildChunkIfAffected(IncProjectBuilder.java:1088)
java:     at org.jetbrains.jps.incremental.IncProjectBuilder.buildChunks(IncProjectBuilder.java:854)
java:     at org.jetbrains.jps.incremental.IncProjectBuilder.runBuild(IncProjectBuilder.java:441)
java:     at org.jetbrains.jps.incremental.IncProjectBuilder.build(IncProjectBuilder.java:190)
java:     at org.jetbrains.jps.cmdline.BuildRunner.runBuild(BuildRunner.java:132)
java:     at org.jetbrains.jps.cmdline.BuildSession.runBuild(BuildSession.java:318)
java:     at org.jetbrains.jps.cmdline.BuildSession.run(BuildSession.java:146)
java:     at org.jetbrains.jps.cmdline.BuildMain$MyMessageHandler.lambda$channelRead0$0(BuildMain.java:218)
java:     at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
java:     at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
java:     at java.lang.Thread.run(Thread.java:745)
java: Compilation failed: internal java compiler error
Checking dependencies... \[lombok\-app\]
Dependency analysis found 0 affected files
Errors occurred while compiling module 'lombok-app'
javac 1.8.0\_91 was used to compile java sources
Finished, saving caches...
Compilation failed: errors: 1; warnings: 0
Executing post-compile tasks...
Loading Ant configuration...
Running Ant tasks...
Synchronizing output directories...
```

从异常堆栈信息可知是 `com.sun.tools.javac.util.Bits.incl(Bits.java:186)` 出现报错，定位到异常位置：

com.sun.tools.javac.util.Bits#incl

```
public void incl(int var1) {
    Assert.check(this.currentState != Bits.BitsState.UNKNOWN);
    Assert.check(var1 >= 0, "Value of x " + var1); 
    this.sizeTo((var1 >>> 5) + 1);
    this.bits\[var1 >>> 5\] |= 1 << (var1 & 31);
    this.currentState = Bits.BitsState.NORMAL;
}
```

打断点分析异常链路，可以定位到是 SetterProcessor#process 方法导致的报错。导致问题的核心值是 JCTree 的 pos 字段，该字段用于指明当前语法树节点在语法树中的位置，而使用 TreeMaker 生成的 pos 都为固定值，需要将此字段设置为所解析元素的 pos 即可（修改后的 SetterProcessor#process 方法见上一节）。

5\. 参考
------

*   [Lombok原理分析与功能实现](https://link.segmentfault.com/?enc=ASOdUW3WAXRTVKM4cPbO%2Fw%3D%3D.c5aqrIL1E4hkozUnlsl8VK0uSWsizTyZVqNslkxyMidxWwsRQvZjcQAVKhRUnNmLgEdDBF9yi4DMKMx3K2tYgA%3D%3D)
*   [Java-JSR-269-插入式注解处理器](https://link.segmentfault.com/?enc=chwCNO8mqAcGD7%2B6iBVPRg%3D%3D.S7gUT0u7N6qsuh8CWn0PQ2TDbewNn8j59lSSGd3%2FqYSUgjfp%2BiQJ9bsQ2W7CI2NDSXePQiLegKB%2FF3ijW4HfweQYyCYGoOs3NN%2BwMiNVLrModfrXuqJi6FpFZvECHavHrst1WyxPaxqW28cFitMBery8ufVmw3FkbeAL0oTZrCk%3D)
*   [Lombok经常用，但是你知道它的原理是什么吗？](https://link.segmentfault.com/?enc=A7untsy61mUAaCbOmOtr2A%3D%3D.tzkUxDBkmhfUk66CS3S7I5WhZwczmboe%2Byo9raHl%2FoIzv%2BcG2fW%2F1FD7khEvkS5b)
*   [jsr269 java.lang.AssertionError: Value of x -1](https://link.segmentfault.com/?enc=Q7VEKyVMBPwBk7FVMV3Rbw%3D%3D.B%2FnyAc0VyANhnwp%2F5JIkqHM%2B%2FRabgvX17eDsAmDrgjoKnjmjJjY7twjGnLUl%2FgXCPt5jHEKBUyG%2Bb4YXlTot0w%3D%3D)

* * *

作者：Sumkor  
链接：[https://segmentfault.com/a/11...](https://segmentfault.com/a/1190000041200280)