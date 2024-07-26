#  可变字符串类型的方法

StringBuffer和StringBuilder的API方法完全相同。

StringBuffer和StringBuilder的内部有一个value数组，这个数组没有final修饰，可以被修改、被扩容，这个数组就相当于是字符串的缓冲区。围绕这个字符串缓冲区的操作：

- 增：append，insert
- 删：delete，deleteCharAt
- 改：setCharAt，setLength，replace，reverse
- 查：indexOf，lastIndexOf，charAt



# collection接口的方法:

- ###### 增加: 

  - **add**:对象.add("要添加的东西")
  - **addAll**:对象.addAll(另一个对象)

- ###### 删除:

  - **remove**:对象.remove("要删除的元素")

  - **removeIf**:(**"需要Predicate接口"**,需要编写一个匿名内部类来实现**Predicate**接口)

    -  Predicate p = new Predicate(){
                  @Override
                  public boolean test(Object o) {//这里o就相当于集合中的元素
                      //在test方法中要编写 元素满足xx条件才会被删除
                      String s = (String) o;
                      return s.contains("o");
                  }
              };

      ​	对象.removeIf(p)

  - **clear**:对象.clear //清空所有

  - **removeAll**:输出+对象1.removeAll(对象2)  //删除对象2中包含的所有元素

  - **retainAll**:输出+对象1.retainAll(对象2)  //删除对象1和对象2的非交集部分

- ###### 改:

  - **无**

- ###### 查:

  - **contains:**输出+对象1.contains(要查的东西)  //判断集合中是否有 要查的东西	
  - **containsAll:**输出+对象1.containsAll(对象2) //判断集合1中与没有集合2的所有元素
  - **size():**输出+对象1.size() // 查看对象1的元素个数
  - **isEmpty:**输出+对象.isEmpty()  //查看对象中是否有元素

- ###### 遍历:

  - **方式一:**直接foreach（最简洁，最推荐的方法）"增强for循环---iter"

    - for (Object obj : 对象) {
                  String str = (String) obj;
                  System.out.println(str + "的长度：" + str.length());
              }

  - **方式二:**将集合的元素放到一个数组中返回，然后再遍历

    - Object[] objects = 对象.toArray();
              for (int i = 0; i < objects.length; i++) {
                  System.out.println(objects[i]);
              }

  - **方式三：**迭代器遍历  迭代器:重复访问集合的元素的工具

    - Iterator i= 对象.iterator();

      while(i.hasNext()){
                  Object obj = i.next();
                  System.out.println(obj);

​		

# Collection的子接口：Set

Set子接口的实现类有一个共同特征：元素不可重复。

常用的实现类：HashSet、LinkedHashSet、TreeSet等。

- HashSet：元素不可重复，完全无序。（这里的无序，是指完全没有规律）
- LinkedHashSet：元素不可重复，元素有顺序，按照添加顺序排列，因为底层有一个双向链表。
- TreeSet：元素不可重复，元素有大小顺序。依赖于Comparable接口 或 Comparator接口。

TreeSet:   

Comparable接口（自然比较器，默认比较器）

Comparator接口（定制比较器)



**增删改查偏离与Collection一样**



# Collection的子接口：List

List系列的集合有一个共同特征：元素可以重复，元素可以通过下标进行访问。无论底层是不是数组，都可以通过下标进行访问，我们称为有序的（有下标顺序）。

List的常见实现类有：ArrayList、LinkedList、Vector、Stack等。



除了Collection父接口的方法之外，List子接口又增加了一些方法

- 添加：
  - add（元素）
  - add（下标，元素） //指定下标位置添加元素
  - addAll（集合）
  - addAll（下标，集合） //指定下标位置，插入一组元素
  
- 删除：
  - clear
  
  - remove（元素/下标）
  
  - removeAll（集合）//删除集合中包含的所有元素.
  
  - removeIf (条件)
  
  - Predicate p = new Predicate(){
        @Override
        public boolean test(Object o) {//这里o就相当于集合中的元素
            //在test方法中要编写 元素满足xx条件才会被删除
            String s = (String) o;
            return s.contains("o");
        }
    };
  
  - retainAll（集合）//删除集合1和集合2的非交集部分
  
    
  
- 修改：
  - set（下标，新元素）
  - replaceAll（UnaryOperator对象）//首字母大写
    - UnaryOperator u = new UnaryOperator() {
                  @Override
                  public Object apply(Object oldElement) {
                      String str = (String) oldElement;
                     
      //写法一return (char)(str.charAt(0)-32) + str.substring(1);
      //写法二return Character.toUpperCase(str.charAt(0)) + str.substring(1);
      //写法三return str.substring(0,1).toUpperCase() + str.substring(1);
                  }
              };
  
- 查询：
  - **contains:**输出+对象1.contains(要查的东西)  //判断集合中是否有 要查的东西	
  - **containsAll:**输出+对象1.containsAll(对象2) //判断集合1中与没有集合2的所有元素
  - **size():**输出+对象1.size() // 查看对象1的元素个数
  - **isEmpty:**输出+对象.isEmpty()  //查看对象中是否有元素
  - **get:**输出+对象.get（下标）返回元素
  - **indexOf:**输出+对象.indexOf(元素) 从前往后返回下标
  - **lastIndexOf:**输出+对象.lastIndexOf(元素)从后往前返回下标
  
- 遍历：
  - 直接foreach（最简洁，最推荐的方法）
  - 将元素放到Object[]数组中返回，然后遍历数组（不太推荐）
  - 用Iterator迭代器遍历：Iterator迭代器只能从头开始遍历。
  - 用ListIterator迭代器遍历：
    - （1）ListIterator迭代器可以从任意位置开始遍历
    - （2）ListIterator迭代器可以从左往右，也可以从右往左
    - （3）在遍历过程中支持对集合进行增、删、改、查

![image-20240616170802287](C:\Users\zero\Desktop\image-20240616170802287.png)

# 动态数组源码

特点:

- 元素是连续存储的,根据索引操作元素率极高
- 数组容量!=元素个数,数组容量>=元素个数.
- 当数组满时,需要扩容.
- 在中间插入或删除元素,需要移动元素.

# 双向链表源码

- 结点类型是指由data(数据)和link(关系)两个不跟的类型.这里的数据是指元素本身.关系是指与当前节点有直接关系的其他结点的地址.
- 绝大部跟数据结构都需要结点类型,例如:链表、树、图等.

```java
单链表的结点: data next
    
    class Node<E>{
        E data;
        Node<E>next;
    }
```

```java
双链表的结点:previous data next
    
    class Node<E>{
        Node<E>previous;
        E data;
        Node<E>next
    }
```

```java
二叉树的结点: left data right
    
    class TreeNode<E>{
        TreeNode<E>parent;
        E data;
        TreeNode<E>left;
        TreeNode<E>right;
    }
```

# List接口各种实现类的区别

- ArrayList:底层是动态数组
- LinkedList:底层是双向链表
- Vector:底层是动态数组
- Stack:是Vector的子类,底层是动态数组,但是它又扩展了几个特殊的方法,使得Stack称为一种新的数据结构:栈结构.遵循先出后进的原则.

JDK中很多成对类型的特点:**旧的线程安全**,**新的线程不安全**

- StringBuffer(古老)   StringBuilder(较新)

- Vector(古老)  ArrayList(较新)

- Hashtable(古老)  HashMap(较新)

集合中,古老的集合命名都没什么规律,而新版的集合命名有规律:

- 凡是List接口的实现类,都是以List结尾
- 凡是Set接口的实现类,都是以Set结尾
- 凡是Map接口的实现类,都是以Map结尾

# 动态**数组(数组)**与**LinkedList(链表)**的区别

- 空间的要求

  - 数组的元素是"连续"存储,在堆中开辟一整块**连续**的存储空间.**对空间要求较高.**

  - 链表的元素不需要"连续"存储.他们可以在堆中不同的位置,

    上一个元素要找下一个元素是靠点中的previous,next等关系.**对空间要求比较低**

- 底层存储元素的特点

  - **数据结构:**
    - **需要扩容**,从非末尾位置插入或删除时,需要移动元素.

  - **链表结构**
    - **不需要扩容**,从非末尾位置插入和删除时,不需要移动过元素,只需要修改前后的关系.

  - 效率问题
    - 测试 10000个对象,任意位置插入元素,测试结果是ArrayList比LinkedList快

# 列表、队列、栈、集等数据结构

## 1.列表与集

**列表**：**是指所有的List接口的实现类**。这个系列的集合元素都是可重复的，可以通过下标进行操作。

- 如果底层是数组，那么下标操作是非常快的，时间复杂度O(1)
- 如果底层是链表，那么下标操作是较慢的，时间复杂度O(n)

**集**：**是指所有的Set接口的实现类**。这个系列的集合都是不可重复的，不可以通过下标进行操作。

- HashSet：完全无序。底层是哈希表结构。
- LinkedHashSet：底层是双向链表+哈希表结构。双向链表按照元素的添加顺序连接元素。
- TreeSet：底层是红黑树。按照元素的大小顺序排列。一边添加一边排序，依赖于Comparable或Comparator接口。

## 2、栈

栈：是指先进后出（或后进先出）的一种数据结构。先进后出（First In Last Out，缩写FILO）后进先出（Last In First Out，缩写LIFO)

- Stack：底层是数组
- LinkedList：底层是链表。

栈结构的方法：

- push(元素)：压栈，添加元素到栈顶
- pop()：弹栈，弹走栈顶元素
- peek()：查看栈顶元素
- search(目标对象)：查看目标对象从栈顶开始数，是第几个元素，不存在返回-1

## 3、队列

队列：

- **普通队列**：先进先出（First In First Out，缩写FIFO）。普通队列有一个公共的接口：Queue。
- **双端队列**：两头都可以进出。双端队列也有一个公共的接口：Deque（double ended queue，简称deque）。Deque又继承了Queue接口。
- **LinkedList实现了List接口，同时也实现Deque接口**。换句话说，LinkedList中的方法集List，Queue，Deque，栈等数据结构的所有方法于一身。

**普通队列的方法：**





# Map

 Map集合的特点:

- Map系列的集合是用来存储键对(key,value)
  - Map<K,V>:这里K代表key的类型,V代表value的类型;
- key,不允许重复,value允许重复;
- key不允许修改,value允许修改;



###### 小总结:

- List:元素可重复,可以修改
- Set:元素不可重复,不可修改;
- Map:key不可重复,不可修改,value,可以重复,可以修改;

## Map接口的方法

- 增
  - put(key,value)
  - putAll(另一个Map集合)
- 删
  - remove(key):根据key删除一对键值对
  - removeAll(key,value) 必须key和value都匹配上,才删除一对键值对
- 改
  - replace(key,新value)根据key直接覆盖value
  - replace(key,旧value,新value)必须key和value都匹配上,才会用新Value覆盖旧Value
  - replaceAll(BiFunction<?super K,?super V,?exends V>function)需要写一个apply抽象方法,抽象方法的形参分别是Key和旧Value,抽象方法的返回值是新Value,
- 查
  - boolean containsKey(key)
  - boolean containsValue(value)
- 遍历
  - Map接口的所有实现类都没有实现Iterable接口,所以都不支持直接使用foreach(增强for循环)或Iterable迭代器便利.
  - 必须将Map集合结构转换为Collection集合才可以使用Foreach或iterator迭代器遍历;
  - 遍历所有的key :set<k>keyset()
  - 遍历所有的value:collection<V>values();
  - 遍历所有的键值对:Set<Map.Entry<K,V>>entrySet()

**遍历所有的key**

```java
Set<Integer> keys = map.keySet();
        for (Integer key : keys) {
            System.out.println(key);
        }
```

**遍历所有的Value**

```java
Collection<String> values = map.values();
        for (String value : values) {
            System.out.println(value);
        }
```

**遍历所有的键值对**

```java
Set<Map.Entry<Integer, String>> entries = map.entrySet();
        for (Map.Entry<Integer, String> entry : entries) {
            System.out.println("key:" + entry.getKey());
            System.out.println("value:" + entry.getValue());
        }
    }
}
```

## Map接口的实现类

- HashMap:

  - 完全无序

  - 允许key和value为null

  - 线程不安全,效率高

- linkedHashMap:双向链表+哈希表   称为链式哈希表
  - 会保留键值对的添加顺序
- TreeMap:红黑树
  - 按照key的大小循序
  - 依赖于comparable和comparator接口
- Hashtable:古老的哈希表
  - 完全无序
  - 不允许key和value为null
  - 线程安全,效率低
- Properties:属性表,它是Hashtable的子类
  - key,value都是固定String类型,通常用于存储系统属性的键值对

# Set与Map的关系

- HashSet:底层就是一个HashMap
- linkedHashSet:底层就是一个LinkedHashMap
- TreeSet:底层是一个TreeMap

# 调用泛型方法

```java
【修饰符】<泛型字母列表>返回值类型 方法名(【形参列表】){
	方法体语句
}

例如
public static <T> List<T> asList(T... a)
```

# 使用泛型类或泛型接口

```java
【修饰符】class 类名 <泛型字母列表>{
    
}
```

```java
【修饰符】 interface 接口名<泛型字母列表>{
    
}
```

- 泛型类

```java
public class ArrayList<E>
```

```java
ArrayList<String>list = new ArrayList<>();
```

- 泛型接口

```java
public interface collection<E>
```

```java
public interface Iterator<E>
```

```java
public interface Comparator<T>
    要重写compare方法
```

```java
public interface comparable<T>
    要重写compareTo方法
```

# 泛型通配符

- 类名<?> 或接口名<?>代表的是任意类型
- 类名<?extebds上限>或 接口名<?extends上限>:?代表是<=上限的类型
- 类名<?super下限>或 接口名<?super 下限>:?代表的是>=下限的类型

## 小细节

- 指定泛型类型时,不能指定基本数据类型,只能指定为引用数据类型
- 泛型字母可以是多个,多个之间使用逗号分隔,泛型字母通常使用单个大写字母表示此时它们的类型是未知的.

# 编写多线程的程序

- 继承Thread类
- 实现Runnable接口
- 实现Callable接口
- 线程池

## 继承Thread类

Thread类是一个线程类,属于java.lang包

步骤:

- 定义一个类继承Thread类
- 必须重写父类方法:public void run();
- 在run方法中,编写这个线程类对象要完成的任务
- 在测试类的main中,常见这个自定义线程的对象
- 调用该对象的start()方法,这个方法是从父类Thread继承的.

## 实现Runnable接口

Runnable接口也是属于Java.lang包

步骤:

- 定义一个类实现Runnable
- 必须重写父接口的一个方法:public void run()
- 在run方法中,编写这个线程类对象要完成的任务
- 在测试类的main中,创建这个自定义线程类的对象
- 创建类的main,创建这个自定义线程的对象
- 创建一个Thread类的对象
- 调用Thread类的对象的start()方法

## 思考题:有直接继承Thread类的方式,为什么还要有实现Runnable接口的方式?

- 类继承有单继承的限制
- 实现接口没有限制,可以多实现

## 可以使用匿名内部类的方式

```java
public static void main(String[] args)
    new Thread(){
    public void run{
        完成的任务
    }
}.start();
```

# Thread类的一些方法

- public void run():此线程要执行的任务在此处定义代码;
- public void start() 执行此线程,java虚拟机会调用run方法
- public static Thread currentThread():返回当前正在执行这句语句的线程对象的引用
- public String getName():获取当前线程名称.默认是Thread-0,Thread-1
- public void setName:设置线程的名称
- public final int gerPriority():设置线程的优先级
- public final void serPriority()改变线程的优先级
- public final boolean isAlive():测试线程是否处于活跃状态.如果线程已经启动且尚未终止,则为活动状态.

## 系列方法

public static void sleep(lang millis):使当前正在执行的线程以指定的毫秒数暂停(暂时停止执行)

public static void yield()只是让当前线程暂停一下,让系统的线程调度器重新调度一次,希望优先级与当前线程相同或更高的其他线程能后获得机会,但是这个不能保证,完全有可能的情况是,当某个线程调用了yield方法暂停后,线程调度器又将其调度出来重新执行.

void join():执行这句代码的线程A要等待该线程(调用join方法的线程B)终止之后才能继续.

void join(long millis):等待该线程终止的时间最长为millis毫秒.如果millis时间到将不再等待

void join(long millis,int nanos)等待该线程终止时间最长为millis毫秒+nanos纳秒

public void interrupt():中断线程.要通过这个方法制造interrupttedException异常的话,前提是这个线程正在执行可能发生异常的方法

wait:让当前线程等待



public void setDaemon(true):将指定的线程设置为守护线程.必须在线程启动之前设置.否则会报异常

public boolean isDaemon:判断线程是不是守护线程

# 线程的安全问题

## 什么情况下会出现线程不安全

- 多个线程
- 共享资源:这时多个线程需要"竞争"资源
- 修改共享资源:不是只读,有写操作

如果上面三个条件都满足了.那么就有不安全隐患,就必须警惕一些问题

## 什么样的数据可以被共享

- 堆的方法区的数据可以被共享
  - 堆:对象的实例变量,但是必须是同一个对象的实例变量.多个线程必须使用同一个对象
  - 方法区:类的静态变量

- 其他内存中的数据不可被共享
  - 虚拟机栈
  - 本地方法栈
  - 程序计数器

## 同步锁原理

java中也是用加锁的方式来解决线程的不安全问题.Java中是给代码加锁.好比是给某个人上厕所的整个过程加锁.一旦某段代码加锁之后,他只能被一个线程执行,其他线程想执行,就只能排队

**同步锁的原理是:**

- java中所有的对象都有一个对象头,对象头中有一个标记位供线程使用
- 每个java对象都可以当同步锁对象
- 竞争同一个共享数据的多个线程,必须使用同一个同步锁对象

## 同步方法与同步代码块

同步锁对象的使用方式:(关键字:**synchronized**)

- 同步代码块
- 同步方法

同步锁锁代码时,需要考虑范围:

- 如果锁的范围太大:会导致其他线程没有机会,只有一个线程有机会使用共享数据
- 如果锁的范围太小:会导致线程的安全问题没有彻底解决
- 锁的代码范围应该是单词任务的所有代码,这些代码具有不可分割的原子性

同步代码块的语法格式

```java
synchronized(同步锁对象名){
    //需要加锁的代码
}
```

同步方法的语法格式

```java
【其他修饰符】 synchronized 返回值类型 方法名(【形参列表】){
    //需要加锁的代码
}
```

- 如果是非静态方法,他的同步锁对象只能是this,
- 如果是静态方法,他的同步锁对象只能是当前类的Class对象

# 单例设计模式

什么是单例类

- 某个类的对象在整个系统运行期间有且只有1个

设计模式

- 解决某种问题时,代码的套路,模板

单例设计模式

- 设计单例类的套路,模板,经验

如何设计单例类

- 创建对象与构造器有关=>构造器私有化
- 在单例类的内部 创建这个唯一的对象供别人使用

根据创建对象的时机不同,可以将单例类的写法分为两大派系:

- 饿汉式单例类:饿(恶)饥不择食,很着急,上来就new对象.恶:蛮不讲理,上来就new对象
- 懒汉式单例类:懒:不到万不得已,绝对不new.

饿汉式写法一:

```java
public enum SingleOne {
    INSTANCE
}
```

饿汉式写法二:

```java
public class SingleTwo {
    public static final SingleTwo INSTANCE = new SingleTwo();
    private SingleTwo(){

    }
}
```

饿汉式写法三:
```java
public class SingleThree {
    private static final SingleThree instance = new SingleThree();
    private SingleThree(){

    }

    public static SingleThree getInstance(){
        return instance;
    }
}
```

懒汉式写法一:

```java
public class SingleFour {
    private static SingleFour instance;//这里不new

    private SingleFour(){

    }

    //当外面调用getInstance()时，说明确实需要这个类的对象，此时再new
    public static synchronized SingleFour getInstance(){
        if(instance == null) {
            instance = new SingleFour();
        }
        return instance;
    }
}

```

懒汉式写法二:

```java
public class SingleFive {
    private SingleFive(){
    }

    /*
    虽然Inner是SingleFive的静态内部类，但是它也是一个类，它有自己的独立的字节码文件。
    它的加载，需要用到这个类才加载。
     */
    private static class Inner{
        static SingleFive instance = new SingleFive();
    }

    //Inner类的加载和初始化，是在调用getInstance方法时，用到Inner类，才初始化，才new了SingleFive的对象
    public static SingleFive getInstance(){
        return Inner.instance;
    }
}
```

# 生产者与消费者的问题

## 简介

当一个或一些线程,负责往数据的缓冲区增加数据,这些线程被称为生产者线程

而另一个或另一些线程,负责从数据缓冲区减少数据,这些线程被称为消费者线程

当数据缓冲区"空"的时候,消费者线程应该停下来等待,等生产者线程生产恶新的数据后才能继续消费.

反过来,当数据缓冲区满了之后,生产者线程应该停下来等待,等消费者线程消耗了数据才能继续生产

这样的场景中设计到了两个问题:

- 线程的安全问题=>加锁
  - 多个线程
  - 有共享的数据缓冲区
  - 对数据缓冲区有修改操作
- 线程的协作问题=>线程通信机制来解决,即等待与唤醒机制
  - wait:让当前线程等待
  - notify/notifyAll:唤醒正在等待的某个/所有线程
  - 当wait或notify/notifyAll方法不是由 监视器 (同步锁对象) 调用时,就会发生异常

## sleep方法和wait方法有什么区别

- 关于锁的态度
  - sleep方法会让当前进程休眠,但是不会释放持有的锁对象
  - wait方法也会让当前线程休眠,但是休眠的同时会释放持有的锁对象
- 其他区别
  - sleep是一个静态方法,由Thread类名调用.他在Thread线程类中声明
    - sleep必须指定时间
  - wait是一个非静态方法,由同步锁对象调用.他在Object类中声明.因为同步锁对象通常不是线程类本身的对象,是其他类的对象当同步锁对象,而其他类可能是任意类型,所以这个方法只能放到Object类中,只有Object类方法才是所有类都能继承的方法
    - wait可以指定时间,也可以不指定时间.如果不指定时间,必须由notify/notifyAll唤醒

# 线程的生命周期

线程的生命周期有五个状态:新建(new)->就绪(Runnable)->运行(running)->阻塞(Blocked)->死亡(Dead).

cpu需要在多条线程之间切换,于是线程状态会多次在运行、阻塞、就绪之间切换

​                          阻塞

新建  ->     就绪 <->    运行  ->   死亡

新版的线程的生命周期状态是由Thread类的内部枚举类State定义





​                    定时等待状态

新建  ->     就绪 <->    运行  ->   死亡

​        等待监视器锁              无线等待

## 死锁

死锁:两个线程互相持有对方想要的资源(锁资源或其他资源)不放手.
