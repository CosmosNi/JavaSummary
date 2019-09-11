## 策略模式

在策略模式（Strategy Pattern）中，一个类的行为或其算法可以在运行时更改。这种类型的设计模式属于行为型模式。

在策略模式中，我们创建表示各种策略的对象和一个行为随着策略对象改变而改变的 context 对象。

策略对象改变 context 对象的执行算法。

- 优点： 算法可以自由切换。 避免使用多重条件判断。 扩展性良好。

- 缺点： 策略类会增多。 所有策略类都需要对外暴露。



以下demo即为基于SpringBoot实现的案例

1. 首先定义一个抽象的处理类

```java
public abstract class AbstractHandler {

    /**
     * 子类不同的操作
     * @return
     */
    public abstract String doOperation();

    /**
     * 根据需要定义一些共有的方法
     */
//    public void commonMethod() {
//        System.out.println("------------------------");
//    }
}

```

2. 定义具体的子类处理

```java
@Component("cycle")
public class CycleHandler extends AbstractHandler {
    @Override
    public String doOperation() {
        return "画了一个圆";
    }
}


@Component("square")
public class SquareHandler extends AbstractHandler {
    @Override
    public String doOperation() {
        return "画了一个正方形";
    }
}

```



3. 定义处理类的上下文，用来选择具体的处理类

```java
@Component
public class HandlerContext {

    private Map<String, AbstractHandler> handlerMap = new ConcurrentHashMap<>();

    /**
     * 上下文注入处理类
     *
     * @param handlerMap
     */
    @Autowired
    public HandlerContext(Map<String, AbstractHandler> handlerMap) {
        this.handlerMap.clear();
        handlerMap.forEach((k, v) -> this.handlerMap.put(k, v));
    }


    /**
     * 获取具体的子处理器
     * @param handleType
     * @return
     */
    public AbstractHandler getInstance(String handleType) {
        return handlerMap.get(handleType);
    }
}

```

4. 测试

```java
@SpringBootApplication
@RestController
public class DemoApplication {
    @Autowired
    private HandlerContext handlerContext;

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }


    @GetMapping("/test")
    public String test(String type) {

        return handlerContext.getInstance(type).doOperation();
    }
}

```



