## 单例模式

###### 饿汉式

```java
public class Singleton{ 
     private static Singleton instance = new Singleton(); 
     private Singleton(){} 
     public static Singleton getInstance(){  
          return instance;  
      } 
}
```



###### 懒汉式

```java
public class Singleton {
     
    //类初始化时，不初始化这个对象(延时加载，真正用的时候再创建)
    private static Singleton instance;
     
    //构造器私有化
    private Singleton(){}
     
    //方法同步，调用效率低
    public static synchronized Singleton getInstance(){
        if(instance==null){
            instance=new Singleton();
        }
        return instance;
    }
}
```



###### 双重检查锁

```java
public class Singleton {
        private volatile static Singleton Singleton;
 
        private Singleton() {
        }
 
        public static Singleton newInstance() {
            if (Singleton == null) {
                synchronized (Singleton.class) {
                    if (Singleton == null) {
                        Singleton = new Singleton();
                    }
                }
            }
            return Singleton;
        }
    }
```



###### 静态内部类

```java
public class Singleton {
     
    private static class SingletonClassInstance{
        private static final Singleton instance=new Singleton();
    }
     
    private Singleton(){}
     
    public static Singleton getInstance(){
        return SingletonClassInstance.instance;
    }
     
}
```



###### 枚举类

```java
public enum Singleton {
     
    //枚举元素本身就是单例
    INSTANCE;
     
    //添加自己需要的操作
    public void singletonOperation(){     
    }
}
```



