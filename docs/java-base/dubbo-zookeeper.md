# Dubbo 注册中心

## 什么是注册中心

dubbo通过注册中心实现了分布式环境中各个服务之间的的注册与发现。是各个节点的纽带。

动态加入：服务提供者通过注册中心可以动态的将自己暴露给消费者，不用逐一通知。

动态发现：消费者可以动态的感知新的配置、路由和新的服务提供者，无需重启加载配置。

动态调整：注册中心支持参数的动态调整，新参数自动更新到所有的服务节点。

统一配置：配置中心化，避免每个服务的配置不一致问题。

## 工作流程

![](https://note.youdao.com/yws/public/resource/7f5bc7c68877511de7441b2db224ad2f/xmlnote/FF583CB8200E4CB2BE540637ED0F1A2B/8443)



0. 服务容器负责启动，加载，运行服务提供者。

1. 服务提供者在启动时，向注册中心注册自己提供的服务。

2. 服务消费者在启动时，向注册中心订阅自己所需的服务。

3. 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。

4. 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。

5. 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

   



## Zookeeper原理

zookeeper是树形结构的注册中心，dubbo使用zookeeper作为注册中心，只会创建持久节点和临时持久节点。

示例：/dubbo/com.foo.BarService/providers 

dubbo：根节点

com.foo.BarService：接口名称

providers：四种服务目录（consumer、routers、configurations）  

![](https://note.youdao.com/yws/public/resource/7f5bc7c68877511de7441b2db224ad2f/xmlnote/DCD491AA94464CADBB4D17444A77CBF9/8445)

type：持久化节点

url：临时节点

- 服务提供者启动时: 向 /dubbo/com.foo.BarService/providers 目录下写入自己的 URL 地 址
- 服务消费者启动时: 订阅 /dubbo/com.foo.BarService/providers 目录下的提供者 URL 地 址。并向 /dubbo/com.foo.BarService/consumers 目录下写入自己的 URL 地址
- 监控中心启动时: 订阅 /dubbo/com.foo.BarService 目录下的所有提供者和消费者 URL 地 址。
  

## zookeeper注册中心工作流程

服务提供方启动

	服务提供者在启动的时候，会在ZooKeeper上注册服务。所谓注册服务，其实就是在ZooKeeper的/dubbo/com.foo.BarService/providers节点下创建一个子节点，并写入自己的URL地址（包括ip地址和端口号），这就代表了com.foo.BarService这个服务的一个提供者。

服务消费者启动

	服务消费者在启动的时候，会向ZooKeeper注册中心订阅自己的服务。其实，就是读取并订阅ZooKeeper上/dubbo/com.foo.BarService/providers节点下的所有子节点，并解析出所有提供者的URL地址来作为该服务地址列表。
	
	同时，服务消费者还会在ZooKeeper的/dubbo/com.foo.BarService/consumers节点下创建一个临时节点，并写入自己的URL地址，这就代表了com.foo.BarService这个服务的一个消费者。

消费者远程调用提供者

	服务消费者，从提供者地址列表中，基于软负载均衡算法，选一个提供者进行调用，如果调用失败，再选另一个提供者调用。

增加服务提供者

	增加提供者，也就是在providers下面新建子节点。一旦服务提供方有变动，zookeeper就会把最新的服务列表推送给消费者。

减少服务提供者

	所有提供者在ZooKeeper上创建的节点都是临时节点，利用的是临时节点的生命周期和客户端会话相关的特性，因此一旦提供者所在的机器出现故障导致该提供者无法对外提供服务时，该临时节点就会自动从ZooKeeper上删除，同样，zookeeper会把最新的服务列表推送给消费者。

ZooKeeper宕机之后

	消费者每次调用服务提供方是不经过ZooKeeper的，消费者只是从zookeeper那里获取服务提供方地址列表。所以当zookeeper宕机之后，不会影响消费者调用服务提供者，影响的是zookeeper宕机之后如果提供者有变动，增加或者减少，无法把最新的服务提供者地址列表推送给消费者，所以消费者感知不到。


## 订阅与发布（Zookeeper实现）

### 发布：

服务提供者和消费者都需要把自己注册到注册中心。发布的本质实际上就是在zookeeper对应的节点上创建目录。

### 订阅：

订阅通常由pull和push两种。duboo采用的是第一启动拉取，后续接收事件重新拉取数据。

服务暴露时候，会监听configurators用于监听动态配置；消费者会监听providers、routers、configurators这三个目录，分别对应服务提供者、路由和动态配置变更通知。

#### Zookeeper中的两个重要机制

**session：**

每个zk客户端与zk连接时会创建一个session，在设置的sessionTimeOut内，客户端会与zk进行心跳包的定时发送，从而感知每个客户端是否宕机，如果创建某个临时Znode的对应session销毁时，相应的临时节点也会被zk删除。

**watcher：**

监听机制，监听某个Znode 当该znode发生变化时，会回调该watcher，但是这个watcher是一次性的，下次需要监听时还得再注册一次。



**具体原理：**

zookeeper注册中心采用的是`事件通知`+`客户端拉取`的方式，客户端第一次连接上注册中心时，会获取对应目录下的全量数据。并在订阅的节点上注册一个watcher，客户端与注册中心保持tcp长连接，后续每个节点数据变更的时候，注册中心的回调主动通知客户端，客户端接到通知后，把对应节点数据都拉取过来。

## 缓存机制

注册中心模块在 `AbstractRegistry` 类中实现通用的缓存机制。这里的缓存可以分为两类，内存服务缓存以及磁盘文件缓存。

#### 内存服务缓存：

`AbstractRegistry`使用一个 `ConcurrentMap`保存相关信息

```java
private final ConcurrentMap<URL, Map<String, List<URL>>> notified = new ConcurrentHashMap<>();
```

这个集合中 key 为消费者的 URL，而 value 为一个 Map 集合。这个内层 Map 集合使用服务目录作为 key，分别为 providers，routers，configurators，consumers 四类，value 则是对应服务列表集合。

#### 文件缓存：
`AbstractRegistry`使用一个 `fiel`保存相关信息

```java
private File file;
```

由于服务重启就会导致内存缓存消失，所以额外增加磁盘文件缓存。

文件缓存默认位置位于 `${user.home}/.dubbo/`文件夹，文件名为`dubbo-registry-${application.name}-${register_address}.cache`。可以设置 `dubbo.registry.file` 配置信息从而修改默认配置，实现源码如下：

```java
String filename = url.getParameter(Constants.FILE_KEY, System.getProperty("user.home") + "/.dubbo/dubbo-registry-" + url.getParameter(Constants.APPLICATION_KEY) + "-" + url.getAddress() + ".cache");
```

#### 缓存的加载：

dubbo 程序初始化的时候，`AbstractRegistry` 构造函数将会从本地磁盘文件中将数据读取到 `Properties` 对象实例中，后续都将会先写入 `Properties`，最后再将里面信息再写入文件。

