---
title: HashMap、HashTable、ArrayList、LinkedList、Vector 区别
date: 2019-03-21 12:02:51
tags:
---

## HashTable 和 HashMap 区别

1.继承不同
```java
public class Hashtable extends Dictionary implements Map
public class HashMap extends AbstractMap implements Map
```
<!-- more -->

2.Hashtable 中的方法是同步的，而HashMap中的方法在缺省情况下是非同步的。在多线程并发的环境下，可以直接使用Hashtable，但是要使用HashMap的话就要自己增加同步处理了。

3.Hashtable中，key和value都不允许出现null值。
  在HashMap中，null可以作为键，这样的键只有一个；可以有一个或多个键所对应的值为null。当get()方法返回null值时，即可以表示 HashMap中没有该键，也可以表示该键所对应的值为null。因此，在HashMap中不能由get()方法来判断HashMap中是否存在某个键， 而应该用containsKey()方法来判断。

4.两个遍历方式的内部实现上不同。
  Hashtable、HashMap都使用了 Iterator。而由于历史原因，Hashtable还使用了Enumeration的方式 。
  
5.两者计算hash的方法不同
  Hashtable计算hash是直接使用key的hashcode对table数组的长度直接进行取模。
```java
int hash = key.hashCode();
int index = (hash & 0x7FFFFFFF) % tab.length;
```
  HashMap计算hash对key的hashcode进行了二次hash，以获得更好的散列值，然后对table数组长度取摸。
```java
static int hash(int h) {
        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }

 static int indexFor(int h, int length) {
        return h & (length-1);
    }
```
6.Hashtable和HashMap它们两个内部实现方式的数组的初始大小和扩容的方式。HashTable中hash数组默认大小是11，增加的方式是 old*2+1。HashMap中hash数组的默认大小是16，而且一定是2的指数。

## ArrayList、Vector 区别

1.两个类都实现了List接口（List接口继承了Collection接口），他们都是有序集合，都可以按位置索引号取出某个元素，并且其中的数据是允许重复的，这是HashSet之类的集合的最大不同处，HashSet之类的集合不可以按索引号去检索其中的元素，也不允许有重复的元素。

2.同步性，Vector是线程安全的，也就是说是它的方法之间是线程同步的，而ArrayList是线程序不安全的，它的方法之间是线程不同步的。如果只有一个线程会访问到集合，那最好是使用ArrayList，因为它不考虑线程安全，效率会高些；如果有多个线程会访问到集合，那最好是使用Vector，因为不需要我们自己再去考虑和编写线程安全的代码。

3.数据增长：ArrayList与Vector都有一个初始的容量大小，当存储进它们里面的元素的个数超过了容量时，就需要增加ArrayList与Vector的存储空间，Vector默认增长为原来两倍，而ArrayList的增长策略在文档中没有明确规定（从源代码看到的是增长为原来的1.5倍）。ArrayList与Vector都可以设置初始的空间大小，Vector还可以设置增长的空间大小，而ArrayList没有提供设置增长空间的方法。

## ArrayList、LinkedList 区别

1.ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。

2.对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。

3.对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据。
增加元素到列表任意位置:
由于实现的不同，ArrayList和LinkedList在这个方法上存在一定的性能差异，由于ArrayList是基于数组实现的，而数组是一块连续的内存空间，如果在数组的任意位置插入元素，必然导致在该位置后的所有元素需要重新排列，因此，其效率相对会比较低。
每次插入操作，都会进行一次数组复制。而这个操作在增加元素到List尾端的时候是不存在的，大量的数组重组操作会导致系统性能低下。并且插入元素在List中的位置越是靠前，数组重组的开销也越大。
对LinkedList来说，在List的尾端插入数据与在任意位置插入数据是一样的，不会因为插入的位置靠前而导致插入的方法性能降低。
在ArrayList的每一次有效的元素删除操作后，都要进行数组的重组。并且删除的位置越靠前，数组重组时的开销越大。
在LinkedList的实现中，首先要通过循环找到要删除的元素。如果要删除的位置处于List的前半段，则从前往后找；若其位置处于后半段，则从后往前找。因此无论要删除较为靠前或者靠后的元素都是非常高效的；但要移除List中间的元素却几乎要遍历完半个List，在List拥有大量元素的情况下，效率很低。

## HashSet 和 HashMap、Hashtable的 区别

除开HashMap和Hashtable外，还有一个hash集合HashSet，有所区别的是HashSet不是key value结构，仅仅是存储不重复的元素，相当于简化版的HashMap，只是包含HashMap中的key而已

通过查看源码也证实了这一点，HashSet内部就是使用HashMap实现，只不过HashSet里面的HashMap所有的value都是同一个Object而已，因此HashSet也是非线程安全的，至于HashSet和Hashtable的区别，HashSet就是个简化的HashMap的，所以你懂的
下面是HashSet几个主要方法的实现

```java
  private transient HashMap<E,Object> map;
  private static final Object PRESENT = new Object();
  
  public HashSet() {
    map = new HashMap<E,Object>();
    }
 public boolean contains(Object o) {
    return map.containsKey(o);
    }
 public boolean add(E e) {
    return map.put(e, PRESENT)==null;
    }
 public boolean add(E e) {
    return map.put(e, PRESENT)==null;
    }
 public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
    }


 public void clear() {
    map.clear();
    }
```

## Spring 框架中都用到了哪些设计模式 ？

*代理模式，在AOP中被使用最多。
*单例模式，在Spring配置文件中定义bean的时候默认的是单例模式。
*工厂模式, BeanFactory用来创建对象的实例。
*模板方法， 用来解决重复性代码。
*前端控制器，Spring提供了DispatcherSerclet来对请求进行分发。
*视图帮助，Spring提供了一系列的JSP标签。
*依赖注入，它是惯穿于BeanFactory/ApplicationContext接口的核心理念。

## Spring Bean 的生命周期 ？

*指Spring中bean元素被实例化，和被销毁的过程。我们通过init-method属性指定初始化方法； 通过destroy-method方法指定销毁方法。
*注意：只有作用域为Singleton的时候才会有效。

## Java 中实现依赖注入的三种方式？

*构造器注入
*set方法注入
*接口注入

## 什么是控制反转（IOC），什么是依赖注入（DI）？

*IOC：就是对象之间的依赖关系由容器来创建，对象之间的关系本来是由我们开发者自己创建和维护的，在我们使用Spring框架后，对象之间的关系由容器来创建和维护，将开发者做的事让容器做，这就是控制反转。BeanFactory接口是Spring Ioc容器的核心接口。
*DI：我们在使用Spring容器的时候，容器通过调用set方法或者是构造器来建立对象之间的依赖关系。
*控制反转是目标，依赖注入是我们实现控制反转的一种手段。

## Spring AOP（面向切面）编程的原理 ？

*AOP面向切面编程，它是一种思想。它就是针对业务处理过程中的切面进行提取，以达到优化代码的目的，减少重复代码的目的。 就比如，在编写业务逻辑代码的时候，我们习惯性的都要写：日志记录，事物控制，以及权限控制等，每一个子模块都要写这些代码，代码明显存在重复。这时候，我们运用面向切面的编程思想，采用横切技术，将代码中重复的部分，不影响主业务逻辑的部分抽取出来，放在某个地方进行集中式的管理，调用。 形成日志切面，事物控制切面，权限控制切面。 这样，我们就只需要关系业务的逻辑处理，即提高了工作的效率，又使得代码变的简洁优雅。这就是面向切面的编程思想，它是面向对象编程思想的一种扩展。
*AOP的使用场景： 缓存、权限管理、内容传递、错误处理、懒加载、记录跟踪、优化、校准、调试、持久化、资源池、同步管理、事物控制等。 AOP的相关概念： 切面（Aspect） 连接点(JoinPoint) 通知（Advice） 切入点（Pointcut） 代理（Proxy）： 织入（WeaVing）
*Spring AOP的编程原理？ 代理机制 JDK的动态代理：只能用于实现了接口的类产生代理。 Cglib代理：针对没有实现接口的类产生代理，应用的是底层的字节码增强技术，生成当前类的子类对象。

## 什么是Spring MVC ？

*Spring MVC是一个基于MVC架构的用来简化web应用程序开发的应用开发框架，它是Spring的一部分，它和Struts2一样都属于表现层的框架。
*MVC（Model模型 View 视图 Controller 控制器）：这是一种软件架构思想，是一种开发模式，将软件划分为三种不同类型的模块，分别是模型，视图，和控制器。 模型：用于封装业务逻辑处理（java类）； 视图：用于数据展现和操作界面（Servlet）； 控制器：用于协调视图和模型（jsp）； 处理流程：视图将请求发送给控制器，由控制器选择对应的模型来处理；模型将处理结果交给控制器，控制器选择合适的视图来展现处理结果。hex
