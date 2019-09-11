# 代理模式

定义：代理模式即给某个对象提供一个代理对象，并由代理对象控制对原对象的引用。简单来说，就是现实生活的中介，就以房产中介来说，客户委托中介寻找房源，具体的繁杂手续均由中介处理。

- 中介隔离
- 开闭原则，增加功能



代理可分为静态代理以及动态代理。此处主要介绍动态代理。基于jdk以及cglib两种方式。

- jdk：

```java
public class DynamicProxy implements InvocationHandler {

    private Object object;

    public DynamicProxy(Object object) {
        this.object = object;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("正在执行：");
        Object result = method.invoke(object, args);
        System.out.println(result);
        System.out.println("执行完毕！");
        return null;
    }
}

```



- cglib:

```java
public class CglibProxy implements MethodInterceptor {

    private Object target;

    /**
     * 相当于JDK动态代理中的绑定
     */
    public Object getInstance(Object target) {
        //给业务对象赋值
        this.target = target;
        //创建加强器，用来创建动态代理类
        Enhancer enhancer = new Enhancer();
        //为加强器指定要代理的业务类（即：为下面生成的代理类指定父类）
        enhancer.setSuperclass(this.target.getClass());
        //设置回调：对于代理类上所有方法的调用，都会调用CallBack，而Callback则需要实现intercept()方法进行拦
        enhancer.setCallback(this);
        // 创建动态代理类对象并返回
        return enhancer.create();
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("预处理——————");
        Objepublic class CglibProxy implements MethodInterceptor {

    private Object target;

    /**
     * 相当于JDK动态代理中的绑定
     */
    public Object getInstance(Object target) {
        //给业务对象赋值
        this.target = target;
        //创建加强器，用来创建动态代理类
        Enhancer enhancer = new Enhancer();
        //为加强器指定要代理的业务类（即：为下面生成的代理类指定父类）
        enhancer.setSuperclass(this.target.getClass());
        //设置回调：对于代理类上所有方法的调用，都会调用CallBack，而Callback则需要实现intercept()方法进行拦
        enhancer.setCallback(this);
        // 创建动态代理类对象并返回
        return enhancer.create();
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("预处理——————");
        Object result = methodProxy.invokeSuper(o, objects);
        System.out.println(result);
        System.out.println("调用后操作——————");
        return result;
    }ct result = methodProxy.invokeSuper(o, objects);
        System.out.println(result);
        System.out.println("调用后操作——————");
        return result;
    }public class CglibProxy implements MethodInterceptor {

    private Object target;

    /**
     * 相当于JDK动态代理中的绑定
     */
    public Object getInstance(Object target) {
        //给业务对象赋值
        this.target = target;
        //创建加强器，用来创建动态代理类
        Enhancer enhancer = new Enhancer();
        //为加强器指定要代理的业务类（即：为下面生成的代理类指定父类）
        enhancer.setSuperclass(this.target.getClass());
        //设置回调：对于代理类上所有方法的调用，都会调用CallBack，而Callback则需要实现intercept()方法进行拦
        enhancer.setCallback(this);
        // 创建动态代理类对象并返回
        return enhancer.create();
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("预处理——————");
        Object result = methodProxy.invokeSuper(o, objects);
        System.out.println(result);
        System.out.println("调用后操作——————");
        return result;
    }
```



- 代理测试类

```java
public class DynamicProxyDriver {

    /**
     * jdk代理对象
     */
    static People getPersonProxy(People people) {
        return (People) Proxy.newProxyInstance(people.getClass().getClassLoader(), people.getClass().getInterfaces(), new DynamicProxy(people));
    }

    /**
     * cglib 代理对象
     */
    static People getCglibProxy(People people) {
        return (People) new CglibProxy().getInstance(people);
    }

    public static void main(String[] args) {
        People people = new Student();
        People proxyPeople = getPersonProxy(people);
        proxyPeople.doOperation("张三", "提交作业！");

        People cglibPeople = getCglibProxy(people);
        cglibPeople.doOperation("张三", "提交作业！");
    }


    interface People {
        String doOperation(String name, String action);
    }

    static class Student implements People {


        @Override
        public String doOperation(String name, String action) {
            return name + "正在" + action;
        }

    }
}

```

