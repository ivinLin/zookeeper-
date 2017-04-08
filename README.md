# zookeeper集群搭建

## 下载和运行
### 下载 zookeeper
从Apache官方网站下载一个ZooKeeper 的最近稳定版本。
下载地址：http://Hadoop.apache.org/zookeeper/releases.html

作为国内用户来说，选择最近的的源文件服务器所在地，能够节省不少的时间。
可以选择在这里下载：http://labs.renren.com/apache-mirror//hadoop/zookeeper/

下载下来后，解压缩到相应目录中。

### 下载 JDK
zookeeper 的运行依赖 java 环境。如果 mac 机器下没有安装 JDK，可在 oracle 官网上进行下载。具体的下载安装教程可参考：http://www.cnblogs.com/quickcodes/p/5127101.html。

JDK 安装好之后，需要对环境变量进行配置，具体配置教程可参考：http://www.cnblogs.com/quickcodes/p/5398709.html

### 运行 zookeeper
进入到 zookeeper 的解压缩文件目录，进到bin目录下，运行服务程序：
    
    ProdeMacBook-Pro:bin ivin$ ./zkServer.sh start
    ZooKeeper JMX enabled by default
    Using config: /Users/ivin/program/zookeeper-2/bin/../conf/zoo.cfg
    Starting zookeeper ... STARTED
启动成功。

其他相关的命令为：
    
    1. 启动ZK服务:       bin/zkServer.sh start
    2. 查看ZK服务状态:   bin/zkServer.sh status
    3. 停止ZK服务:       bin/zkServer.sh stop
    4. 重启ZK服务:       bin/zkServer.sh restart

服务器启动成功之后，可以使用客户端连接服务器。
    
    ProdeMacBook-Pro:bin ivin$ ./zkCli.sh -server 127.0.0.1:2181
    Connecting to 127.0.0.1:2181
    WATCHER::

    WatchedEvent state:SyncConnected type:None path:null
    [zk: 127.0.0.1:2181(CONNECTED) 0] 
连接上之后，即可使用相关的命令进行操作。

## zookeeper配置
zookeeper 的配置文件在 conf 目录下，文件名为 zoo.cfg（需要将默认的 zoo_sample.cfg 文件名修改为 zoo.cfg）。

### 单机配置
具体的配置相对比较简单，单机配置 DEMO 如下：
    
    # ZK中的一个时间单元。ZK中所有时间都是以这个时间单元为基础，进行整数倍配置的。
    # 例如，session的最小超时时间是2*tickTime
    tickTime=2000
    
    # Follower在启动过程中，会从Leader同步所有最新数据，然后确定自己能够对外服务的起始状态。
    # Leader允许F在 initLimit 时间内完成这个工作。通常情况下，我们不用太在意这个参数的设置。
    # 如果ZK集群的数据量确实很大了，F在启动的时候，从Leader上同步数据的时间也会相应变长，
    # 因此在这种情况下，有必要适当调大这个参数
    initLimit=10
    
    # 在运行过程中，Leader负责与ZK集群中所有机器进行通信，例如通过一些心跳检测机制，来检测机器的存活状态。
    # 如果L发出心跳包在syncLimit之后，还没有从F那里收到响应，那么就认为这个F已经不在线了。
    # 注意：不要把这个参数设置得过大，否则可能会掩盖一些问题。
    syncLimit=5
    
    # 存储快照文件snapshot的目录。默认情况下，事务日志也会存储在这里。
    # 建议同时配置参数dataLogDir, 事务日志的写性能直接影响zk性能数，更新事务日志将被存储到默认位置（临时目录）
    dataDir=/Users/ivin/program/zoo-data
    
    # 事务日志输出目录。尽量给事务日志的输出配置单独的磁盘或是挂载点，这将极大的提升ZK性能
    dataLogDir=/Users/ivin/program/zoo-data
 
    # 客户端连接server的端口，即对外服务端口，一般设置为2181吧
    clientPort=2181
    
    # 默认情况下，Leader是会接受客户端连接，并提供正常的读写服务。
    # 但是，如果你想让Leader专注于集群中机器的协调，那么可以将这个参数设置为no，这样一来，会大大提高写操作的性能。
    leaderServes=no
    
    # 用于记录所有请求的log，一般调试过程中可以使用，但是生产环境不建议使用，会严重影响性能。
    # traceFile=/Users/ivin/program/zoo-data/
    
    # 单个客户端与单台服务器之间的连接数的限制，是ip级别的，默认是60，如果设置为0，那么表明不作任何限制。
    # 请注意这个限制的使用范围，仅仅是单台客户端机器与单台ZK服务器之间的连接数限制，
    # 不是针对指定客户端IP，也不是ZK集群的连接数限制，也不是单台ZK对所有客户端的连接数限制。
    maxClientCnxns=0
    
### 集群配置
在同一机器下，通过绑定不同的端口，启动多个zookeeper程序实现伪集群。

1. 首先需要有三个 zookeeper 程序包，可分别命名为：zk1,zk2,zk3
2. 每个程序包下面的配置文件为：
        
        tickTime=2000
        initLimit=10
        syncLimit=5
        dataDir=/Users/ivin/program/zoo-data/zk1

        # 每个包的端口号不一样，分别为2181，2182，2183
        clientPort=2181 
        
        # server.A=B：C：D
        #其中 A 是一个数字，表示这个是第几号服务器；必须在上面的数据目录下新建一个 myid 文件，文件内容即为 A。
        # B 是这个服务器的 ip 地址；
        # C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；
        # D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号
        server.1001=127.0.0.1:2187:3888
        server.1002=127.0.0.1:2188:3888
        server.1003=127.0.0.1:2189:3888
        
3. 在dataDir=/Users/ivin/program/zoo-data/目录下新建三个目录，分别是：zk1,zk2,zk3
4. 在每个目录下新建文件名为 myid 的文件，文件内容即为上面配置文件的server.A中的 A。如上，zk1下的 myid 值为1001；zk2下的 myid 值为1002；zk3下的 myid 值为1003
5. 分别启动三个程序包中的服务器，启动起来后，集群即搭建好了。

#### 遇到问题（1天时间啊才搞定。。。）
三个 zk 程序包配置好之后，都可以启动起来，但通过./zkServer.sh status 查看，其中一个一直提示：
    
    ProdeMacBook-Pro:bin ivin$ ../../zookeeper-3/bin/zkServer.sh status
    ZooKeeper JMX enabled by default
    Using config: /Users/ivin/program/zookeeper-3/bin/../conf/zoo.cfg
    Error contacting service. It is probably not running.
表明这个服务没有正常启动起来。另外两个正常，一个是 leader，一个是 follower。

==打开 zookeeper.out 文件（在 bin 目录下）查看具体的启动信息，发现有一个端口绑定的错误。==

**解决办法**：将配置文件中的集群配置绑定的端口尽量改成不同。
原先的配置是：
    
    server.1001=127.0.0.1:2187:3888
    server.1002=127.0.0.1:2188:3888
    server.1003=127.0.0.1:2189:3888
修改成：
    
    server.1001=127.0.0.1:2187:9881
    server.1002=127.0.0.1:2188:9882
    server.1003=127.0.0.1:2189:9883
问题解决。所以，++及时查看错误日志文件才是解决问题的王道++！

## zookeeper 常用四字命令
1. echo stat|nc 127.0.0.1 2181 ：
查看哪个节点被选择作为follower或者leader
2. echo ruok|nc 127.0.0.1 2181 ： 测试是否启动了该Server，若回复imok表示已经启动。
3. echo dump| nc 127.0.0.1 2181 ：列出未经处理的会话和临时节点。
4. echo kill | nc 127.0.0.1 2181 ：关掉server
5. echo conf | nc 127.0.0.1 2181 ：输出相关服务配置的详细信息。
6. echo cons | nc 127.0.0.1 2181 ：列出所有连接到服务器的客户端的完全的连接 / 会话的详细信息。
7. echo envi |nc 127.0.0.1 2181  ：输出关于服务环境的详细信息（区别于 conf 命令）。
8. echo reqs | nc 127.0.0.1 2181 ： 列出未经处理的请求。
9. echo wchs | nc 127.0.0.1 2181 ：列出服务器 watch 的详细信息。
10. echo wchc | nc 127.0.0.1 2181 ：通过 session 列出服务器 watch 的详细信息，它的输出是一个与 watch 相关的会话的列表。
11. echo wchp | nc 127.0.0.1 2181 ：通过路径列出服务器 watch 的详细信息。它输出一个与 session 相关的路径。
