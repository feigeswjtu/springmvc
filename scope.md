Scope 描述了Spring创建一个bean的行为，Spring的Scope有以下几种，通过@Scope注解实现。

1. Singleton: 表示一个容器中只有一个Bean实例，这个配置为Spring的Bean的默认配置。
2. Prototype: 每次注入时，都会新建一个对象。
3. Requst: Web项目中，给每一个http request新建一个示例。
4. Session: Web项目中，给每一个http session新建一个示例。
5. GlobalSession: 这个只是在portal应用中有用，给每一个global http session新建一个bean。
我们常用的只有前两者Singleton和Prototype，下面举两个例子说明:

两个bean:
```java
@Service
@Scope("prototype")
public class DemoPrototypeService {
}
```

```java
 @Service
 public class DemoSingletonService {
 }
```

配置类:
```java
@Configuration
@ComponentScan("ch2.scope")
public class Config {
}
```

运行类:

```java
public class Main {
    public static void main(String []args){
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
        DemoSingletonService s1 = context.getBean(DemoSingletonService.class);
        DemoSingletonService s2 = context.getBean(DemoSingletonService.class);

        DemoPrototypeService p1 = context.getBean(DemoPrototypeService.class);
        DemoPrototypeService p2 = context.getBean(DemoPrototypeService.class);

        System.out.println("singleton例子, s1==s2: " + s1.equals(s2));
        System.out.println("prototype例子, p1==p2: " + s1.equals(s2));
    }
}
```

执行结果:
```data
singleton例子, s1==s2: true
prototype例子, p1==p2: false
```
