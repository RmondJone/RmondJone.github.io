---
title: "Java的3种代理模式"
date: 2023-04-10T17:43:52+08:00
draft: false
categories: ["Java"]
tags: ["Java"]
---

**Java的代理模式简介**

代理(Proxy)是一种设计模式,提供了对目标对象另外的访问方式;即通过代理对象访问目标对象.这样做的好处是:**可以在目标对象实现的基础上,增强额外的功能操作,即扩展目标对象的功能**
这里使用到编程中的一个思想:不要随意去修改别人已经写好的代码或者方法,如果需改修改,可以通过代理的方式来扩展该方法，Spring中AOP切面编程就是代理的一个典型例子。

举个例子来说明代理的作用:假设我们想邀请一位明星,那么并不是直接连接明星,而是联系明星的经纪人,来达到同样的目的.明星就是一个目标对象,他只要负责活动中的节目,而其他琐碎的事情就交给他的代理人(经纪人)来解决.这就是代理思想在现实中的一个例子

![](/images/java_proxy.webp)

代理模式的关键点是:**代理对象与目标对象，代理对象是对目标对象的扩展，并会调用目标对象**

## 一、静态代理
创建一个接口，然后创建被代理的类实现该接口并且实现该接口中的抽象方法。之后再创建一个代理类，同时使其也实现这个接口。在代理类中持有一个被代理对象的引用，而后在代理类方法中调用该对象的方法。

```java
//接口
public interface HelloInterface {
    void sayHello();
}

//目标对象
public class Hello implements HelloInterface{
    @Override
    public void sayHello() {
        System.out.println("Hello zhanghao!");
    }
}

//代理类
public class HelloProxy implements HelloInterface{
    private HelloInterface helloInterface = new Hello();
    @Override
    public void sayHello() {
        //在不修改目标对象的一行代码前提下，代理方法实现了对目标对象的拓展
        System.out.println("Before invoke sayHello" );
        helloInterface.sayHello();
        System.out.println("After invoke sayHello");
    }
}

public static void main(String[] args) {
    HelloProxy helloProxy = new HelloProxy();
    helloProxy.sayHello();
}
```

### 静态代理总结
* 可以做到在不修改目标对象的功能前提下，对目标功能扩展。
* 因为代理对象需要与目标对象实现一样的接口，所以会有很多代理类，类太多，同时一旦接口增加方法，目标对象与代理对象都要维护。

## 二、JDK动态代理

为了解决上述静态代理产生的问题，Java动态代理技术油然而生。动态代理有以下几个特点：

* 代理对象不需要实现接口
* 代理对象的生成，无需手动声明，**通过Java反射机制动态生成**

关于Java反射机制可以参考我之前写的文章:
[《Java反射的使用以及原理深入》](/posts/java/java_reflex)

JDK动态代理的实现主要使用了下面2个关键类，在后续章节会详细介绍其原理，这里只介绍他们的使用方法：
* java.lang.reflect.Proxy
* java.lang.reflect.InvocationHandler

下面我们使用这2个类来实现一个动态代理，代码如下：

```java
// 目标类接口
public interface IHelloService {

    /**
     * 方法1
     * @param userName
     * @return
     */
    String sayHello(String userName);

    /**
     * 方法2
     * @param userName
     * @return
     */
    String sayByeBye(String userName);

}
// 目标类
public class HelloService implements IHelloService {

    @Override
    public String sayHello(String userName) {
        System.out.println(userName + " hello");
        return userName + " hello";
    }

    @Override
    public String sayByeBye(String userName) {
        System.out.println(userName + " ByeBye");
        return userName + " ByeBye";
    }
}
// 中间类
public class JavaProxyInvocationHandler implements InvocationHandler {

    /**
     * 中间类持有目标类对象的引用,这里会构成一种静态代理关系
     */
    private Object obj ;

    /**
     * 有参构造器,传入委托类的对象
     * @param obj 目标类的对象
     */
    public JavaProxyInvocationHandler(Object obj){
        this.obj = obj;
    }

    /**
     * 动态生成代理类对象,Proxy.newProxyInstance
     * @return 返回代理类的实例
     */
    public Object newProxyInstance() {
        return Proxy.newProxyInstance(
                //指定代理对象的类加载器
                obj.getClass().getClassLoader(),
                //代理对象需要实现的接口，可以同时指定多个接口
                obj.getClass().getInterfaces(),
                //方法调用的实际处理者，代理对象的方法调用都会转发到这里
                this);
    }


    /**
     *
     * @param proxy 代理对象
     * @param method 代理方法
     * @param args 方法的参数
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("invoke before");
        Object result = method.invoke(obj, args);
        System.out.println("invoke after");
        return result;
    }
}
// 测试动态代理类
public class MainJavaProxy {
    public static void main(String[] args) {
        JavaProxyInvocationHandler proxyInvocationHandler = new JavaProxyInvocationHandler(new HelloService());
        IHelloService helloService = (IHelloService) proxyInvocationHandler.newProxyInstance();
        helloService.sayByeBye("paopao");
        helloService.sayHello("yupao");
    }

}

```

### JDK动态代理总结
* 代理对象不需要实现接口
* 代理对象的生成，无需手动声明，**通过Java反射机制动态生成**
* 从上面的例子中可以看出，JDK动态代理有一个明显的缺陷，就是**目标类必须实现一个接口**，否则无法调用Proxy.newProxyInstance方法动态生成代理对象

##  三、CGLib动态代理
CGLib动态代理主要是为了解决JDK动态代理的痛点，是一个三方的框架。**当目标类没有实现接口时**，我们就要考虑使用这种动态代理技术来实现目标类的代理。

### CGLib实现原理
CGLib 代理是针对类来实现代理的，原理是对指定的目标类生成一个子类并重写其中业务方法来实现代理。

* 查找目标类上的所有非final的public类型的方法(final的不能被重写)
* 将这些方法的定义转成字节码
* 将组成的字节码转换成相应的代理的Class对象然后通过反射获得代理类的实例对象
* 实现 MethodInterceptor 接口,用来处理对代理类上所有方法的请求

```java
// 目标类
public class CglibHelloClass {
    /**
     * 方法1
     * @param userName
     * @return
     */
    public String sayHello(String userName){
        System.out.println("目标对象的方法执行了");
        return userName + " sayHello";
    }

    public String sayByeBye(String userName){
        System.out.println("目标对象的方法执行了");
        return userName + " sayByeBye";
    }

}
/**
 * CglibInterceptor 用于对方法调用拦截以及回调
 *
 */
public class CglibInterceptor implements MethodInterceptor {
    /**
     * CGLIB 增强类对象，代理类对象是由 Enhancer 类创建的，
     * Enhancer 是 CGLIB 的字节码增强器，可以很方便的对类进行拓展
     */
    private Enhancer enhancer = new Enhancer();

    /**
     *
     * @param obj  被代理的对象
     * @param method 代理的方法
     * @param args 方法的参数
     * @param proxy CGLIB方法代理对象
     * @return  cglib生成用来代替Method对象的一个对象，使用MethodProxy比调用JDK自身的Method直接执行方法效率会有提升
     * @throws Throwable
     */
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("方法调用之前");
        Object o = proxy.invokeSuper(obj, args);
        System.out.println("方法调用之后");
        return o;
    }


    /**
     * 使用动态代理创建一个代理对象
     * @param c
     * @return
     */
    public  Object newProxyInstance(Class<?> c) {
        /**
         * 设置产生的代理对象的父类,增强类型
         */
        enhancer.setSuperclass(c);
        /**
         * 定义代理逻辑对象为当前对象，要求当前对象实现 MethodInterceptor 接口
         */
        enhancer.setCallback(this);
        /**
         * 使用默认无参数的构造函数创建目标对象,这是一个前提,被代理的类要提供无参构造方法
         */
        return enhancer.create();
    }
}

//测试类
public class MainCglibProxy {
    public static void main(String[] args) {
        CglibInterceptor cglibProxy = new CglibInterceptor();
        CglibHelloClass cglibHelloClass = (CglibHelloClass) cglibProxy.newProxyInstance(CglibHelloClass.class);
        cglibHelloClass.sayHello("isole");
        cglibHelloClass.sayByeBye("sss");
    }
}
```

### CGLib动态代理总结

* 目标类无需实现接口
* CGLib动态代理无法实现目标类中被final标记的方法的代理
* CGLib动态代理在创建代理对象时所花费的时间却比JDK多得多，所以对于单例的对象，因为无需频繁创建对象，用CGLib合适，反之，使用JDK方式要更为合适一些。

## 四、JDK动态代理源码解析
JDK动态代理的关键在于代理对象的动态实现，所以我们只需着重关注Proxy类的静态方法newProxyInstance的源码即可，由于本人从事Android开发，所以这边的源码和JDK中的源码有一些出入，Android中做了一些优化处理：
```java
 @CallerSensitive
    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
        Objects.requireNonNull(h);

        final Class<?>[] intfs = interfaces.clone();
        // Android-removed: SecurityManager calls
        /*
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }
        */

        /*
         * Look up or generate the designated proxy class.
         */
        Class<?> cl = getProxyClass0(loader, intfs);

        /*
         * Invoke its constructor with the designated invocation handler.
         */
        try {
            // Android-removed: SecurityManager / permission checks.
            /*
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }
            */

            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            if (!Modifier.isPublic(cl.getModifiers())) {
                // BEGIN Android-changed: Excluded AccessController.doPrivileged call.
                /*
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
                */

                cons.setAccessible(true);
                // END Android-removed: Excluded AccessController.doPrivileged call.
            }
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
    }
```
从上诉代码可以看到，**Android中去除了安全性校验，如果构造方法为私有直接调用setAccessible(true)而不进行AccessController处理**。而生成代理类主要是getProxyClass0(loader, intfs),下面我们深入这个方法
```java
    private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
        proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());
    /**
     * Generate a proxy class.  Must call the checkProxyAccess method
     * to perform permission checks before calling this.
     */
    private static Class<?> getProxyClass0(ClassLoader loader,
                                           Class<?>... interfaces) {
        if (interfaces.length > 65535) {
            throw new IllegalArgumentException("interface limit exceeded");
        }

        // If the proxy class defined by the given loader implementing
        // the given interfaces exists, this will simply return the cached copy;
        // otherwise, it will create the proxy class via the ProxyClassFactory
        return proxyClassCache.get(loader, interfaces);
    }
```
可以看到这边使用了弱引用缓存，如果之前生成过代理类Class对象，则直接返回，没有则会通过ProxyClassFactory工厂去生成代理类Class对象。关于WeakCache源码这边不再深入，感兴趣的可以自己去看，底层使用了ConcurrentMap来进行缓存。下面我们进一步深入到ProxyClassFactory源码
```
    private static final class ProxyClassFactory
        implements BiFunction<ClassLoader, Class<?>[], Class<?>>
    {
        // prefix for all proxy class names
        private static final String proxyClassNamePrefix = "$Proxy";

        // next number to use for generation of unique proxy class names
        private static final AtomicLong nextUniqueNumber = new AtomicLong();

        @Override
        public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {

            Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
            for (Class<?> intf : interfaces) {
                /*
                 * Verify that the class loader resolves the name of this
                 * interface to the same Class object.
                 */
                Class<?> interfaceClass = null;
                try {
                    interfaceClass = Class.forName(intf.getName(), false, loader);
                } catch (ClassNotFoundException e) {
                }
                if (interfaceClass != intf) {
                    throw new IllegalArgumentException(
                        intf + " is not visible from class loader");
                }
                /*
                 * Verify that the Class object actually represents an
                 * interface.
                 */
                if (!interfaceClass.isInterface()) {
                    throw new IllegalArgumentException(
                        interfaceClass.getName() + " is not an interface");
                }
                /*
                 * Verify that this interface is not a duplicate.
                 */
                if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
                    throw new IllegalArgumentException(
                        "repeated interface: " + interfaceClass.getName());
                }
            }

            String proxyPkg = null;     // package to define proxy class in
            int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

            /*
             * Record the package of a non-public proxy interface so that the
             * proxy class will be defined in the same package.  Verify that
             * all non-public proxy interfaces are in the same package.
             */
            for (Class<?> intf : interfaces) {
                int flags = intf.getModifiers();
                if (!Modifier.isPublic(flags)) {
                    accessFlags = Modifier.FINAL;
                    String name = intf.getName();
                    int n = name.lastIndexOf('.');
                    String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                    if (proxyPkg == null) {
                        proxyPkg = pkg;
                    } else if (!pkg.equals(proxyPkg)) {
                        throw new IllegalArgumentException(
                            "non-public interfaces from different packages");
                    }
                }
            }

            if (proxyPkg == null) {
                // if no non-public proxy interfaces, use the default package.
                proxyPkg = "";
            }

            {
                // Android-changed: Generate the proxy directly instead of calling
                // through to ProxyGenerator.
                List<Method> methods = getMethods(interfaces);
                Collections.sort(methods, ORDER_BY_SIGNATURE_AND_SUBTYPE);
                validateReturnTypes(methods);
                List<Class<?>[]> exceptions = deduplicateAndGetExceptions(methods);

                Method[] methodsArray = methods.toArray(new Method[methods.size()]);
                Class<?>[][] exceptionsArray = exceptions.toArray(new Class<?>[exceptions.size()][]);

                /*
                 * Choose a name for the proxy class to generate.
                 */
                long num = nextUniqueNumber.getAndIncrement();
                String proxyName = proxyPkg + proxyClassNamePrefix + num;

                return generateProxy(proxyName, interfaces, loader, methodsArray,
                                     exceptionsArray);
            }
        }
    }
```
从上诉代码，可以看到Android再次对这边进行了魔改，原本JDK中需要通过ProxyGenerator.generateProxyClass方法来生成字节码文件，进而通过字节码文件生成Class对象的流程，这边**Android使用了一个generateProxy的native函数替代**。

具体为什么使用这个native函数替代？前文说到JDK动态代理有一个明显的缺陷就是目标类必须实现接口，这里使用这个native函数就是为了打破这个限制，通过这个native函数生成的代理类可以实现对**目标对象非接口实现方法的代理**
```java
    @FastNative
    private static native Class<?> generateProxy(String name, Class<?>[] interfaces,
                                                 ClassLoader loader, Method[] methods,
                                                 Class<?>[][] exceptions);
```