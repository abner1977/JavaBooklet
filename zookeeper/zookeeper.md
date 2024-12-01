# zookeeper

## 一、定义

Zookeeper是一个分布式开源框架，用来协调分布式应用。



## 二、应用场景

1、作为注册中心（命名服务）结合RPC调用框架。

2、发布-订阅（watcher），对zk节点监控，发生改变，进行事件通知。

3、负载均衡实现。

4、分布式通知。

5、zookeeper分布式锁。

6、分布式配置中心。



## 三、数据结构

树状存储结构

*zookeeper 为了保证 高吞吐和低延迟，在内存中维护了这个树状的目录结构，这种特性使得 zookeeper 不能用于存放大量的数据，每个节点的存放数据上限为 1M。*

节点名称不允许重复。

**节点包括：**

persistent:持久化节点

persistent_SEQUENTIAL（[sɪ'kwenʃ(ə)l] ）:顺序自动编号持久化节点，这种节点会根据当前已存在的节点数自动加 1

ephemeral（[ɪ'fem(ə)rəl] ）:临时节点，客户端session超时这类节点就会被自动删除

ephemeral_sequential:临时自动编号节点



## 四、安装

**windows：**

环境要求:必须要有jdk环境,本次讲课使用jdk1.8

1.安装jdk

2.安装Zookeeper. 在官网http://zookeeper.apache.org/下载zookeeper.我下载的是zookeeper-3.4.6版本。

解压zookeeper-3.4.6至D:\machine\zookeeper-3.4.6.

在D:\machine 新建data及log目录。

3.ZooKeeper的安装模式分为三种，分别为：单机模式（stand-alone）、集群模式和集群伪分布模式。ZooKeeper 单机模式的安装相对比较简单，如果第一次接触ZooKeeper的话，建议安装ZooKeeper单机模式或者集群伪分布模式。

安装单击模式。 至D:\machine\zookeeper-3.4.6\conf 复制 zoo_sample.cfg 并粘贴到当前目录下，命名zoo.cfg.





## 五、命令

使用 zkCli.sh -server 127.0.0.1:2181 连接到 ZooKeeper 服务，连接成功后，系统会输出 ZooKeeper 的相关环境以及配置信息。

命令行工具的一些简单操作如下：

\1. 显示根目录下、文件： ls / 使用 ls 命令来查看当前 ZooKeeper 中所包含的内容

\2. 显示根目录下、文件： ls2 / 查看当前节点数据并能看到更新次数等数据

\3. 创建文件，并设置初始内容： create /zk "test" 创建一个新的 znode节点“ zk ”以及与它关联的字符串

\4. 获取文件内容： get /zk 确认 znode 是否包含我们所创建的字符串

\5. 修改文件内容： set /zk "zkbak" 对 zk 所关联的字符串进行设置

\6. 删除文件： delete /zk 将刚才创建的 znode 删除

\7. 退出客户端： quit

\8. 帮助命令： help



## 六、JAVA操作zookeeper





## 七、watcher节点监听

接口类Watcher用于表示一个标准的事件处理器

包含KeeperState和EventType两个枚举类，分别代表了通知状态和事件类型，同时定义了事件的回调方法：process（WatchedEvent event）



当/zk-book这个节点的数据发生变更时，服务端会发送给客户端一个“ZNode数据内容变更”事件，客户端只能够接收到信息。





## 八、分布式锁

### ***1、数据库实现分布式锁。***

参考连接：https://honeypps.com/architect/distribute-lock-based-on-database/

**方式1：**建立一个锁表，添加唯一性约束。当想获得锁的时候，插入数据，释放锁的时候，删除数据。

多个请求的话，只有一个成功，其他都会插入失败。

**<u>优缺点：</u>**

1. 这种锁没有失效时间，一旦释放锁的操作失败就会导致锁记录一直在数据库中，其它线程无法获得锁。这个缺陷也很好解决，比如可以做一个定时任务去定时清理。
2. 这种锁的可靠性依赖于数据库。建议设置备库，避免单点，进一步提高可靠性。
3. 这种锁是非阻塞的，因为插入数据失败之后会直接报错，想要获得锁就需要再次操作。如果需要阻塞式的，可以弄个for循环、while循环之类的，直至INSERT成功再返回。
4. 这种锁也是非可重入的，因为同一个线程在没有释放锁之前无法再次获得锁，因为数据库中已经存在同一份记录了。想要实现可重入锁，可以在数据库中添加一些字段，比如获得锁的主机信息、线程信息等，那么在再次获得锁的时候可以先查询数据，如果当前的主机信息和线程信息等能被查到的话，可以直接把锁分配给它。

**2、乐观锁**

version 版本概念，更新的时候，获取开始查询的version，判断是否一致，一致则更新。

作用于同一行数据记录上，这就导致一个明显的缺点，在一些特殊场景，如大促、秒杀等活动开展的时候，大量的请求同时请求同一条记录的行锁，会对数据库产生很大的写压力。乐观锁比较适合并发量不高，并且写操作不频繁的场景。

**3、悲观锁**

查询语句后面增加FOR UPDATE，数据库会在查询过程中给数据库表增加悲观锁，也称排他锁。

悲观锁中，每一次行数据的访问都是独占的，只有当正在访问该行数据的请求事务提交以后，其他请求才能依次访问该数据，否则将阻塞等待锁的获取。

悲观锁可以严格保证数据访问的安全。但是缺点也明显，即每次请求都会额外产生加锁的开销且未获取到锁的请求将会阻塞等待锁的获取，在高并发环境下，容易造成大量请求阻塞，影响系统可用性。另外，悲观锁使用不当还可能产生死锁的情况。

### 2、redis实现

[具体实现以及原理分析]: https://www.jianshu.com/p/47fd7f86c848



### 3、zookeeper实现

临时节点+事件通知 实现

```java
package com.abner.test;

import org.I0Itec.zkclient.ZkClient;

public  abstract  class ZookeeperAbstractLock implements  Lock {

    // zk连接地址
    private static final String CONNECTSTRING = "127.0.0.1:2181";
    // 创建zk连接
    protected ZkClient zkClient = new ZkClient(CONNECTSTRING);
    protected static final String PATH = "/lock";

    public void getLock() {
        if (tryLock()) {
            System.out.println("##获取lock锁的资源####");
        } else {
            // 等待
            waitLock();
            // 重新获取锁资源
            getLock();
        }

    }

    // 获取锁资源
    abstract boolean tryLock();

    // 等待
    abstract void waitLock();

    public void unLock() {
        if (zkClient != null) {
            zkClient.close();
            System.out.println("释放锁资源...");
        }
    }

}

```

```java
package com.abner.test;

import org.I0Itec.zkclient.IZkDataListener;

import java.util.concurrent.CountDownLatch;

public class ZookeeperDistrbuteLock  extends ZookeeperAbstractLock  {
    private CountDownLatch countDownLatch = null;
    @Override
    boolean tryLock() {
        try {
            zkClient.createEphemeral(PATH);
            return true;
        } catch (Exception e) {
            return false;
        }

    }

    @Override
    void waitLock() {
        IZkDataListener iZkDataListener=new IZkDataListener() {
            public void handleDataChange(String s, Object o) throws Exception {
                // 唤醒被等待的线程
                if (countDownLatch != null) {
                    countDownLatch.countDown();
                }

            }
            public void handleDataDeleted(String s) throws Exception {

            }
        };
        // 注册事件
        zkClient.subscribeDataChanges(PATH, iZkDataListener);
        if (zkClient.exists(PATH)) {
            countDownLatch = new CountDownLatch(1);
            try {
                countDownLatch.await();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        // 删除监听
        zkClient.unsubscribeDataChanges(PATH, iZkDataListener);


    }
}

```

```java
package com.abner.test;

public class OrderService implements Runnable {
    private OrderNumGenerator orderNumGenerator = new OrderNumGenerator();
    private Lock lock = new ZookeeperDistrbuteLock();
    public void run() {
        getNumber();
    }
    public void getNumber() {
        try {
            lock.getLock();
            String number = orderNumGenerator.getNumber();
            System.out.println(Thread.currentThread().getName() + ",生成订单ID:" + number);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unLock();
        }
    }
    public static void main(String[] args) {
        System.out.println("####生成唯一订单号###");
        for (int i = 0; i < 100; i++) {
            new Thread( new OrderService()).start();
        }
    }
}

```



## 九、zookeeper实现负载均衡

思路：使用Zookeeper实现负载均衡原理，服务器端将启动的服务注册到，zk注册中心上，采用临时节点。客户端从zk节点上获取最新服务节点信息，本地使用负载均衡算法，随机分配服务器。

取模算法   调用次数/服务器集群数量

step1:服务器注册至zk 

step2:客户端监听节点。获取节点下服务器信息。

step3:客户端根据服务器信息，采用取模算法，发送请求。





## 十、zookeeper选举

三台主机会尝试创建master节点（临时节点），谁创建成功了，就是master，向外提供。其他两台就是slave。

所有slave必须关注master的删除事件（临时节点，如果服务器宕机了，Zookeeper会自动把master节点删除）。如果master宕机了，会进行新一轮的master选举



• Zookeeper的核心是**原子广播**，这个机制保证了各个Server之间的同步。实现这个机制的协议叫做<u>*<u>*Za**b协**</u>
 　  议。Zab协议有两种模式，它们分别是恢复模式（选主）和广播模式（同步）**</u>。当服务启动或者在领导者
 　　崩溃后，Zab就进入了恢复模式，当领导者被选举出来，且大多数Server完成了和leader的状态同步以后
 　  ，恢复模式就结束了。状态同步保证了leader和Server具有相同的系统状态



为了保证事务的顺序一致性，zookeeper采用了**递增的事务id号（zxid）来标识事务**。所有的提议（
 　　proposal）都在被提出的时候加上了zxid。实现中zxid是一个64位的数字，它高32位是epoch用来标识
 　  leader关系是否改变，每次一个leader被选出来，它都会有一个新的epoch，标识当前属于那个leader的
 　　统治时期。低32位用于递增计数。
 　• 每个Server在工作过程中有三种状态：
 　　　LOOKING：当前Server不知道leader是谁，正在搜寻
 　　　LEADING：当前Server即为选举出来的leader
 　　　FOLLOWING：leader已经选举出来，当前Server与之同步

## 十一、羊群效应

**一个特定的znode 改变的时候ZooKeper 触发了所有watches 的事件。导致大量的通知需要被发送，将会导致在这个通知期间的其他操作提交的延迟**

解决：

1. 客户端调用create()方法创建名为“_locknode_/guid-lock-”的节点，需要注意的是，这里节点的创建类型需要设置为EPHEMERAL_SEQUENTIAL。
2. 客户端调用getChildren(“_locknode_”)方法来**获取所有已经创建的子节点**，注意，**这里不注册任何Watcher**。
3. 客户端获取到所有子节点path之后，**如果发现自己在步骤1中创建的节点序号最小，那么就认为这个客户端获得了锁**。
4. 如果在步骤3中发现自己并非所有子节点中最小的，说明自己还没有获取到锁。此时**客户端需要找到比自己小的那个节点，然后对其调用exist()方法，同时注册事件监听。**
5. 之后当这个被关注的节点被移除了，客户端会收到相应的通知。这个时候客户端需要再次调用getChildren(“_locknode_”)方法来获取所有已经创建的子节点，确保自己确实是最小的节点了，然后进入步骤3。















