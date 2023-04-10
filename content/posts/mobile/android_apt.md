---
title: "Android中注解以及Android APT的基本使用"
date: 2023-04-10T16:26:30+08:00
draft: false
categories: ["移动端"]
tags: ["Android","APT","注解"]
---

## 一、什么是注解？
注解是代码里的特殊标记，这些标记可以在编译、类加载、运行时被读取，并执行相应的处理。通过使用注解，开发人员可以在不改变原有逻辑的情况下，在源文件中嵌入一些补充的信息。代码分析工具、 开发工具和部署工具可以通过这些补充信息进行验证、处理或者进行部署。
## 二、注解的分类
* **标准注解**

注解|描述
--|--
@Override|对覆盖超类中的方法进行标记，如果被标记的方法并没有实际覆盖超类中的方法，则编 译器会发出错误警告
@Deprecated|对不鼓励使用或者已过时的方法添加注解，当编程人员使用这些方法时，将会在编译 时显示提示信息
@SuppressWarnings|选择性地取消特定代码段中的警告
@SafeVarargs|JDK 7新增，用来声明使用了可变长度参数的方法，其在与泛型类一起使用时不会出现 类型安全问题

除了标准注解，还有元注解，它用来注解其他注解，从而创建新的注解
* **元注解**

注解|描述
--|--
@Targe|注解所修饰的对象范围
@Inherited|表示注解可以被继承
@Documented|表示这个注解应该被JavaDoc工具记录
@Retention|用来声明注解的保留策略
@Repeatable|JDK 8 新增，允许一个注解在同一声明类型（类、属性或方法）上多次使用

其中@Targe注解取值是一个ElementType类型的数组，其中有以下几种取值，对应不同的对象范围

* ElementType.TYPE：能修饰类、接口或枚举类型。
* ElementType.FIELD：能修饰成员变量。
* ElementType.METHOD：能修饰方法。
* ElementType.PARAMETER：能修饰参数。
* ElementType.CONSTRUCTOR：能修饰构造方法。
* ElementType.LOCAL_VARIABLE：能修饰局部变量。
* ElementType.ANNOTATION_TYPE：能修饰注解。
* ElementType.PACKAGE：能修饰包。
* ElementType.TYPE_PARAMETER：类型参数声明。
* ElementType.TYPE_USE：使用类型

其中@Retention注解有3种类型，分别表示不同级别的保留策略。

* RetentionPolicy.SOURCE：源码级注解。注解信息只会保留在.java源码中，源码在编译后，注解信息被丢弃，不会保留在.class中。
* RetentionPolicy.CLASS：编译时注解。注解信息会保留在.java 源码以及.class 中。当运行Java程序时， JVM会丢弃该注解信息，不会保留在JVM中
*  RetentionPolicy.RUNTIME：运行时注解。当运行Java程序时，JVM也会保留该注解信息，可以通过反射获取该注解信息

## 三、注解的定义
### 基本定义
定义新的注解类型使用@interface关键字，这与定义一个接口很像，如下所示
```java
public @interface Test{
  ....
}
```
定义完注解后则可以直接使用
```java
@Test
public class AnnotationTest{
  ....
}
```

### 定义成员变量

注解只有成员变量，没有方法。注解的成员变量在注解定义中以“无形参的方法”形式来声明，其“方法名”定义了该成员变量的名字，其返回值定义了该成员变量的类型：
```java
public @interface Test {
    String name();
    int age();
}
```
也可以在定义注解的成员变量时，使用default关键字为其指定默认值，如下所示：
```java
public @interface Test {
    String name() default "郭翰林";
    int age() default 28;
}
```
### 定义注解的范围

定义注解的范围主要由上面的几个**元注解**来限定使用范围。下面这个示例则限制了这个注解只能去标记变量，并且此注解是编译时注解。
```java
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Target(ElementType.FIELD)
public @interface Test {
    String name() default "郭翰林";
    int age() default 28;
}
```
## 四、注解处理器

**如果没有处理注解的工具，那么注解也不会有什么大的作用**。对于不同的注解有不同的注解处理器。 虽然注解处理器的编写会千变万化，但是其也有处理标准，比如：针对运行时注解会采用反射机制处理， 针对编译时注解会采用 AbstractProcessor 来处理。本节就针对前面讲到的运行时注解和编译时注解来编写注解处理器。

### 运行时的注解处理器

```java
public class AnnonationTest {
    @Test
    String defaultName;
    @Test(name = "111111")
    String name;

    public static void main(String[] args) {
        Field[] fields = AnnonationTest.class.getDeclaredFields();
        for (Field field : fields) {
            Test test = field.getAnnotation(Test.class);
            System.out.println(test.name());
        }
    }
}
//运行结果
郭翰林
111111
```
### 编译时的注解处理器

由于反射机制会有性能消耗，如果应用程序频繁的进行反射则会导致应用卡顿。所以类似于[Butterknife](https://github.com/JakeWharton/butterknife)、[Dagger](https://github.com/google/dagger)的依赖注入框架都采用了编译时的注解处理器。

编译时的注解处理器，主要通过**继承AbstractProcessor** 实现了process方法，来生成想要的逻辑代码（.java文件），触发编译时静态的存储在工程目录本地。

```java
@AutoService(Processor.class)
public class BindViewProcessor extends AbstractProcessor {
    private Messager messager;
    private Elements elements;

    /**
     * 注释：可以得到ProcessingEnviroment，
       ProcessingEnviroment提供很多有用的工具类Elements, Types 和 Filer
     * 时间：2021/2/18 0018 15:39
     * 作者：郭翰林
     *
     * @param processingEnv
     */
    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        super.init(processingEnv);
        messager = processingEnv.getMessager();
        elements = processingEnv.getElementUtils();
    }

    /**
     * 注释：指定这个注解处理器是注册给哪个注解的
     * 时间：2021/2/18 0018 15:39
     * 作者：郭翰林
     *
     * @return
     */
    @Override
    public Set<String> getSupportedAnnotationTypes() {
        HashSet<String> supportTypes = new LinkedHashSet<>();
        supportTypes.add(BindView.class.getCanonicalName());
        return supportTypes;
    }

    /**
     * 注释：指定使用的Java版本
     * 时间：2021/2/18 0018 15:39
     * 作者：郭翰林
     *
     * @return
     */
    @Override
    public SourceVersion getSupportedSourceVersion() {
        return SourceVersion.latestSupported();
    }

    /**
     * 注释：具体的处理过程
     * 时间：2021/2/18 0018 15:40
     * 作者：郭翰林
     * @param annotations
     * @param roundEnv
     * @return
     */
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        .....
        return false;
    }
}

```

（*PS：尽管Butterknife最后还是采用了反射，但是由于他使用了一个LinkHashMap记录了反射对象，理论上一个类里的注解只会被反射一次，所以性能消耗有限。*）

## 五、Android中的APT技术的基本使用

我们来模仿Butterknife的@BindView注解来阐述，Android业务场景中我们应该怎么去自定义我们的注解，以及配合APT技术来生成我们想要的模板代码，来简化我们日常开发中的逻辑。

### 第一步：创建自定义注解以及注解处理器Java模块

创建模块目录如下图所示，添加自定义注解以及自定义注解处理器。这里需要注意的是创建时**一定需要创建JAVA模块**，要不然后续无法通过annotationProcessor引用模块。

![](/images/android_apt_1.webp)

注解处理器模块的Gradle脚本引用如下：
```
apply plugin: 'java-library'

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation project(":apt-annotation")
    implementation 'com.squareup:javapoet:1.10.0'
    implementation 'com.google.auto.service:auto-service:1.0-rc6'
    annotationProcessor  'com.google.auto.service:auto-service:1.0-rc6'
}

sourceCompatibility = JavaVersion.VERSION_1_8
targetCompatibility = JavaVersion.VERSION_1_8
```
这里我们可以看到这边使用了2个三方库
库名|用途
--|--
com.squareup:javapoet|辅助生成JAVA代码的三方库
com.google.auto.service:auto-service|辅助生成META-INF/services/javax.annotation.processing.Processor文件

规定Java注解处理器模块，必须的在main 目录下新建 resources 资源文件夹，并 resources 中再建立META-INF/services目录 文件夹。最后在META-INF/services中创建 javax.annotation.processing.Processor文件，这个文件中的内容是 注解处理器的名称。为了省去麻烦的创建过程，我们使用了Google提供的AutoService库来帮助我们动态生成，下图可以看到动态生成的截图。
![](/images/android_apt_2.webp)

### 第二步：书写自定义注解以及自定义注解处理器
```java
/**
 * @author 郭翰林
 * @date 2021/2/18 0018 14:23
 * 注释:定义绑定视图注解
 */
@Retention(RetentionPolicy.CLASS)
@Target(ElementType.FIELD)
public @interface BindView {
    @IdRes int value();
}
```

```java
@AutoService(Processor.class)
public class BindViewProcessor extends AbstractProcessor {
    private Messager mMessager;
    private Elements mElements;
    private Map<String, ClassCreatorProxy> mProxyMap = new HashMap<>();

    /**
     * 注释：可以得到ProcessingEnviroment，ProcessingEnviroment提供很多有用的工具类Elements, Types 和 Filer
     * 时间：2021/2/18 0018 15:39
     * 作者：郭翰林
     *
     * @param processingEnv
     */
    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        super.init(processingEnv);
        mMessager = processingEnv.getMessager();
        mElements = processingEnv.getElementUtils();
    }

    /**
     * 注释：指定这个注解处理器是注册给哪个注解的
     * 时间：2021/2/18 0018 15:39
     * 作者：郭翰林
     *
     * @return
     */
    @Override
    public Set<String> getSupportedAnnotationTypes() {
        HashSet<String> supportTypes = new LinkedHashSet<>();
        supportTypes.add(BindView.class.getCanonicalName());
        return supportTypes;
    }

    /**
     * 注释：指定使用的Java版本
     * 时间：2021/2/18 0018 15:39
     * 作者：郭翰林
     *
     * @return
     */
    @Override
    public SourceVersion getSupportedSourceVersion() {
        return SourceVersion.latestSupported();
    }

    /**
     * 注释：具体的处理过程
     * 时间：2021/2/18 0018 15:40
     * 作者：郭翰林
     *
     * @param annotations
     * @param roundEnv
     * @return
     */
    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        mMessager.printMessage(Diagnostic.Kind.NOTE, "BindViewProcessor开始编译...");
        mProxyMap.clear();
        //得到所有的注解
        Set<? extends Element> elements = roundEnv.getElementsAnnotatedWith(BindView.class);
        for (Element element : elements) {
            VariableElement variableElement = (VariableElement) element;
            TypeElement classElement = (TypeElement) variableElement.getEnclosingElement();
            String fullClassName = classElement.getQualifiedName().toString();
            ClassCreatorProxy proxy = mProxyMap.get(fullClassName);
            if (proxy == null) {
                proxy = new ClassCreatorProxy(mElements, classElement);
                mProxyMap.put(fullClassName, proxy);
            }
            BindView bindAnnotation = variableElement.getAnnotation(BindView.class);
            int id = bindAnnotation.value();
            proxy.putElement(id, variableElement);
        }
        //通过javapoet生成
        for (String key : mProxyMap.keySet()) {
            ClassCreatorProxy proxyInfo = mProxyMap.get(key);
            JavaFile javaFile = JavaFile.builder(proxyInfo.getPackageName(), proxyInfo.generateJavaCode()).build();
            try {
                //　生成文件
                javaFile.writeTo(processingEnv.getFiler());
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        mMessager.printMessage(Diagnostic.Kind.NOTE, "BindViewProcessor编译结束...");
        return true;
    }
}

```

### 第三步：创建反射调用模块

在前面2步，完成之后其实，APT注解工作已经完成了，此时把注解运用到主模块代码中，编译即可看到APT自动生成的代码

![](/images/android_apt_3.webp)

但是生成的代码我们肯定是要用来在代码中调用的，所以需要创建一个反射调用模块来调用APT动态生成的代码，但是注意此时创建的是一个Android Lib而不是Java模块。

![](/images/android_apt_4.webp)

```java
public class BindViewTool {
    /**
     * 注释：通过反射调用Apt生成的代码以执行绑定视图逻辑
     * 时间：2021/2/18 0018 17:51
     * 作者：郭翰林
     *
     * @param activity
     */
    public static void bind(Activity activity) {
        Class clazz = activity.getClass();
        try {
            Class bindViewClass = Class.forName(clazz.getName() + "_ViewBinding");
            Method method = bindViewClass.getMethod("bind", activity.getClass());
            method.invoke(bindViewClass.newInstance(), activity);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}

```
```
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    api project(":apt-annotation")
    ....
}
```
从上面的依赖关系可以看到，这里把自定义注解模块也给引用了进去，主要是为了在主模块中引用时不用重复引用。

### 第四步:主模块中引用并验证

首先添加Gradle依赖
```
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    ....
    implementation project(":apt-library")
    annotationProcessor project(":apt-processor")
    ...
}
```
然后在需要使用的地方引用自定义注解
```
public class MainActivity extends AppCompatActivity {
    @BindView(value = R.id.toolbar)
    Toolbar toolbar;

    @BindView(value = R.id.fab)
    FloatingActionButton fab;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        setSupportActionBar(toolbar);
        BindViewTool.bind(this);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Snackbar.make(view, "Replace with your own action", Snackbar.LENGTH_LONG)
                        .setAction("Action", null).show();
            }
        });
    }
}

```
运行代码，可以看到自定义注解处理器生成的日志，以及生成的APT代码截图
![](/images/android_apt_5.webp)
![](/images/android_apt_6.webp)

Demo:[https://github.com/RmondJone/Android-APT](https://github.com/RmondJone/Android-APT)


我的个人博客：[https://rmondjone.github.io/](https://rmondjone.github.io/)

