# Sentinel源码解析（一）: Entry的构建

## 一、Entry介绍

- Entry是sentinel中用来表示是资源是否可以继续执行一个凭证，可以理解为token。
- 使用SphU.entry()时，会返回一个entry，如果抛出了一个异常BlockException，说明资源被保护了。
- 使用 SphO.entry() 时，资源被保护了会返回false，反之true。


## 二、组成

先看Entry类的几个关键字段含义:
```java
public abstract class Entry implements AutoCloseable {

    private static final Object[] OBJECTS0 = new Object[0];

    /**
     * 当前entry的创建时间，毫秒值，用来计算响应时间rt
     */
    private long createTime;

    /**
     * 当前Entry所关联的node，会在NodeSelectorSlot插槽中设置，主要是记录了当前Context下的统计信息
     */
    private Node curNode;

    /**
     * context的请求源节点，通常是服务的消费端，如果存在的话，在ClusterBuilderSlot的entry方法中设置
     * 保存了在这个上下文中，所有的调用的统计信息和
     */
    private Node originNode;
    private Throwable error;

    /**
     * 当前Entry所关联的资源包装器
     */
    protected ResourceWrapper resourceWrapper;
    }
```
#### [Entry](../sentinel-core/src/main/java/com/alibaba/csp/sentinel/Entry.java)

- createTime：当前entry的创建时间，毫秒值，用来计算响应时间rt。
- curNode：当前Entry所关联的node，会在NodeSelectorSlot插槽中设置，主要是记录了当前Context下的统计信息。
- originNode：context的请求源节点，通常是服务的消费端，如果存在的话，在ClusterBuilderSlot的entry方法中设置
- resourceWrapper：当前Entry所关联的资源包装器

#### [CtEntry](../sentinel-core/src/main/java/com/alibaba/csp/sentinel/CtEntry.java)
CtEntry 是 Entry的子类，主要保存了实体之间的关系、调用链、上下文信息。

- parent：entry的父entry，用于在同一个context上下文中，多次调用entry方法，保存entry之间的关系。
- child：entry的子entry，与parent相反
- chain：entry中的插槽链([slotchain的详细解析](./slotchain.md))

#### [AsyncEntry](../sentinel-core/src/main/java/com/alibaba/csp/sentinel/AsyncEntry.java)
AsyncEntry是CtEntry的子类，用于对异步执行的资源进行监控

## 三、Entry源码分析
接下来看看对应Entry的源码
- [CEntry.java源码解析](../sentinel-core/src/main/java/com/alibaba/csp/sentinel/CtEntry.java)
```java
    CtEntry(ResourceWrapper resourceWrapper, ProcessorSlot<Object> chain, Context context) {
        //调用父类构造方法初始化资源包装器
        super(resourceWrapper);
        //设置插槽链
        this.chain = chain;
        //设置上下文
        this.context = context;

        setUpEntryFor(context);
    }
    
    private void setUpEntryFor(Context context) {
        // The entry should not be associated to NullContext.
        if (context instanceof NullContext) {
            return;
        }
        //获取context的当前的entry，并设为parent
        this.parent = context.getCurEntry();
        if (parent != null) {
            ////如果parent存在，并把parent的entry的child设为当前的entry
            ((CtEntry)parent).child = this;
        }
        //设置context的当前entry
        context.setCurEntry(this);
    }
```
```s```


#### 参考资料:
- https://www.jianshu.com/p/3ae0733a3783  