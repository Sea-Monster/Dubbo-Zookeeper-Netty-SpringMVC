# Dubbo-Zookeeper-Netty-SpringMVC

1. dubbo-common 提供公共接口；
2. dubbo-provide是服务提供者（接口的实现）；
3. dubbo-consumer是服务器消费者（接口的调用）；
4. zookeeper管理注册中心（负载均衡，资源同步-当然前提是集群）；
5. springmvc写restful接口；
6. netty提供服务器，客户端的请求由spring的DispatcherServlet处理；

## 运行：  
1. 本地启动zookeeper，例如OSX系统，用homebrew装的zookeeper，则是：
brew services start zookeeper （作为一个后台服务）或者zkServer start

2. StartProvider的main方法;
3. 测试类ConsumerTest;
4. MyServer的main方法;
5. 在浏览器请求http://localhost:8080/项目路径/test/  ;

## 2017-01-25 dubbo 改为dubbo 集群：
参考：http://www.cnblogs.com/yjmyzz/p/4587663.html

1. 在一台机器上安装3个zookeeper，模拟3个zookeeper组成的集群，以我自己本机为例，分别解压到
/usr/local/seamonster/zookeeper-1
/usr/local/seamonster/zookeeper-2
/usr/local/seamonster/zookeeper-3

2. 创建每个目录下conf/zoo.cfg文件：
cp zoo_sample.cfg zoo.cfg

3. 修改zoo.cfg内容
/usr/local/seamonster/zookeeper-1/conf/zoo.cfg (只贴出新增/修改部分):
dataDir=/tmp/zookeeper1/data
clientPort=2181
server.1=localhost:2287:3387
server.2=localhost:2288:3388
server.3=localhost:2289:3389


/usr/local/seamonster/zookeeper-2/conf/zoo.cfg （只贴出新增/修改部分）：
dataDir=/tmp/zookeeper2/data
clientPort=2182
server.1=localhost:2287:3387
server.2=localhost:2288:3388
server.3=localhost:2289:3389

/usr/local/seamonster/zookeeper-3/conf/zoo.cfg （只贴出新增/修改部分）：
dataDir=/tmp/zookeeper3/data
clientPort=2183
server.1=localhost:2287:3387
server.2=localhost:2288:3388
server.3=localhost:2289:3389

注：因为是在一台机器上模拟集群，所以端口不能重复，这里用2181~2183，2287~2289，以及3387~3389相互错开。另外每个zk的instance，都需要设置独立的数据存储目录、日志存储目录，所以dataDir、dataLogDir这二个节点对应的目录，需要手动先创建好。

(存疑)另外还有一个灰常关键的设置，在每个zk server配置文件的dataDir所对应的目录下，必须创建一个名为myid的文件，其中的内容必须与zoo.cfg中server.x 中的x相同，即：

/tmp/zookeeper1/data/myid 中的内容为1，对应server.1中的1
/tmp/zookeeper2/data/myid 中的内容为2，对应server.2中的2
/tmp/zookeeper3/data/myid 中的内容为3，对应server.3中的3

生产环境中，分布式集群部署的步骤与上面基本相同，只不过因为各zk server分布在不同的机器，上述配置文件中的localhost换成各服务器的真实Ip即可。分布在不同的机器后，不存在端口冲突问题，可以让每个服务器的zk均采用相同的端口，这样管理起来比较方便。

4. 启动3个zookeeper：
/usr/local/seamonster/zookeeper-1/bin/zkServer.sh start
/usr/local/seamonster/zookeeper-2/bin/zkServer.sh start
/usr/local/seamonster/zookeeper-3/bin/zkServer.sh start

启用成功后，输入 jps 看下进程

20351 ZooKeeperMain
20791 QuorumPeerMain
20822 QuorumPeerMain
20865 QuorumPeerMain

5. 修改dubbo-provider.xml, dubbo-consumer.xml
<dubbo:registry protocol="zookeeper" address="127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183" />
其实就是几个地址都写在address，用逗号隔开

然后就可以继续之前的步骤2，3，4，5： （2 StartProvider的main方法）