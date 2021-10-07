# RabbitMQ

## 1. MQ引言

### 1.1什么是MQ

> MQ(Message Quene) :  翻译为消息队列,通过典型的生产者和消费者模型,生产者不断向消息队列中生产消息，消费者不断的从队列中获取消息。因为**消息的生产和消费都是异步的**，而且只关心消息的发送和接收，**没有业务逻辑的侵入,轻松的实现系统间解耦**。别名为 **消息中间件**通过利用高效可靠的消息传递机制进行平台无关的数据交流，并基于数据通信来进行分布式系统的集成。

### 1.2 MQ有哪些

> 当今市面上有很多主流的消息中间件，如老牌的ActiveMQ、RabbitMQ，炙手可热的Kafka，阿里巴巴自主开发RocketMQ等。

### 1.3 不同MQ的特点

> 1.ActiveMQ
> 		ActiveMQ 是Apache出品，最流行的，能力强劲的开源消息总线。它是一个完全支持JMS规范的的消息		中间件。**丰富的API**,多种集群架构模式让ActiveMQ在业界成为老牌的消息中间件,在中小型企业 颇受欢		迎!
>
> 2.Kafka
> 		Kafka是LinkedIn开源的分布式发布-订阅消息系统，目前归属于Apache顶级项目。Kafka主要特点是基		于Pull的模式来处理消息消费，**追求高吞吐量，一开始的目的就是用于日志收集和传输**。0.8版本开始支		持复制，不支持事务，**对消息的重复、丢失、错误没有严格要求，适合产生大量数据的互联网服务的数		据收集业务**。
>
> 3.RocketMQ
> 		RocketMQ是阿里开源的消息中间件，它是**纯Java开发**，具有高吞吐量、高可用性、适合大规模分布式		系统应用的特点。**RocketMQ思路起源于Kafka**，但并不是Kafka的一个Copy，它对消息的可靠传输及		事务性做了优化，目前在阿里集团被广泛应用于交   易、充值、流计算、消息推送、日志流式处理、		binglog分发等场景。
>
> 4.RabbitMQ
> 		RabbitMQ是使用Erlang语言开发的开源消息队列系统，基于AMQP协议来实现。AMQP的主要特征是面		向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。**AMQP协议更多用在企业系统内对数		据一致性、稳定性和可靠性要求很高的场景，对性能和吞吐量的要求还在其次**。
>
> RabbitMQ比Kafka可靠，Kafka更适合IO高吞吐的处理，一般应用在大数据日志处理或对实时性（少量延迟），可靠性（少量丢数据）要求稍低的场景使用，比如ELK日志收集。		

## 2. RabbitMQ引言

### 2.1 RabbitMQ

> **基于AMQP协议，erlang语言开发**，是部署最广泛的开源消息中间件,是最受欢迎的开源消息中间件之一。

> AMQP 协议
>  	AMQP（advanced message queuing protocol）在2003年时被提出，最早用于解决金融领不同平台之	  	间的消息传递交互问题。顾名思义， AMQP是一种协议，更准确的说是一种binary wire-levelprotocol（链	接协议）。这是其和JMS的本质差别，AMQP不从API层进行限定，而是**直接定义网络交换的数据格式**。这	**使得实现了AMQP的provider天然性就是跨平台的**。以下是AMQP协议模型:

![](D:\XIE_Tips\MQ-RabbitMq_note\images\20201030175435428.png)

### 2.2 下载安装包进行安装

> windwos版：https://www.cnblogs.com/vaiyanzi/p/9531607.html



## 3. RabbitMQ配置

### 3.1 RabbitMQ管理命令行

> 1.服务启动相关
>
> 	systemctl start|restart|stop|status rabbitmq-server
>
> 2.管理命令行  用来在不使用web管理界面情况下命令操作RabbitMQ
>
> 	rabbitmqctl  help  可以查看更多命令
>
> 3.插件管理命令行
>
> 	rabbitmq-plugins enable|list|disable 

### 3.2 web管理界面介绍

> ![1633444304614](D:\XIE_Tips\MQ-RabbitMq_note\images\1633444304614.png)

> connections：无论生产者还是消费者，都需要与RabbitMQ建立连接后才可以完成消息的生产和消费，在这里可以查看连接情况`
>
> channels：通道，建立连接后，会形成通道，消息的投递获取依赖通道。
>
> Exchanges：交换机，用来实现消息的路由
>
> Queues：队列，即消息队列，消息存放在队列中，等待消费，消费后被移除队列。

#### 3.2.2 Admin用户和虚拟主机管理

##### 3.2.2.1 添加用户

> ![1633444411645](D:\XIE_Tips\MQ-RabbitMq_note\images\1633444411645.png)

> 上面的Tags选项，其实是指定用户的角色，可选的有以下几个：
>
> 超级管理员(administrator)
>
> ​		可登陆管理控制台，可查看所有的信息，并且可以对用户，策略(policy)进行操作。
>
> 监控者(monitoring)
>
> ​		可登陆管理控制台，同时可以查看rabbitmq节点的相关信息(进程数，内存使用情况，磁盘使用情况等)
>
> 策略制定者(policymaker)
>
> ​		可登陆管理控制台, 同时可以对policy进行管理。但无法查看节点的相关信息(上图红框标识的部分)。
>
> 普通管理者(management)
>
> ​		仅可登陆管理控制台，无法看到节点信息，也无法对策略进行管理。其他无法登陆管理控制台，通常就是普通的生产者和消费者。
>

##### 3.2.2.2 创建虚拟主机

> **虚拟主机**
> 		为了让各个用户可以互不干扰的工作，RabbitMQ添加了虚拟主机（Virtual Hosts）的概念。其实就是一个独立的访问路径，不同用户使用不同路径，各自有自己的队列、交换机，互相不会影响。**相当于关系型中的数据库**

> ![1633444747828](D:\XIE_Tips\MQ-RabbitMq_note\images\1633444747828.png)

## 4. RabbitMQ的第一个程序

### 4.1 AMQP协议

> ![1633444851102](D:\XIE_Tips\MQ-RabbitMq_note\images\1633444851102.png)

### 4.2 RabbitMQ支持的消息模型

> ![](D:\XIE_Tips\MQ-RabbitMq_note\images\2020103018025840.png)

> ![](D:\XIE_Tips\MQ-RabbitMq_note\images\20201030180315852.png)

### 4.3 引入依赖

> <dependency>
>     <groupId>com.rabbitmq</groupId>
>     <artifactId>amqp-client</artifactId>
>     <version>5.7.2</version>
> </dependency>

### 4.4 第一种模型 （直连）

> ![1633445220256](D:\XIE_Tips\MQ-RabbitMq_note\images\1633445220256.png)

> 在上图的模型中，有以下概念：
>
> ​	P：生产者，也就是要发送消息的程序
> ​	C：消费者：消息的接受者，会一直等待消息到来。
> ​	queue：消息队列，图中红色部分。类似一个邮箱，可以缓存消息；生产者向其中投递消息，消费者从其中取出消息。

4.4.1 编写生产者

> ```java
> package helloword;
> 
> import com.rabbitmq.client.Channel;
> import com.rabbitmq.client.Connection;
> import com.rabbitmq.client.MessageProperties;
> import org.junit.Test;
> import utils.RabbitMQUtils;
> 
> import java.io.IOException;
> import java.util.concurrent.TimeoutException;
> 
> public class Provider {
> 
>     //生产消息
>     @Test
>     public void testSendMessage() throws IOException, TimeoutException {
> /*
>         //创建连接mq的连接工厂对象
>         ConnectionFactory connectionFactory = new ConnectionFactory();
>         //设置连接rabbitmq主机
>         connectionFactory.setHost("192.168.11.143");
>         //设置端口号
>         connectionFactory.setPort(5672);
>         //设置连接那个虚拟主机
>         connectionFactory.setVirtualHost("/ems");
>         //设置访问虚拟主机的用户名和密码
>         connectionFactory.setUsername("ems");
>         connectionFactory.setPassword("123");
> 
>         //获取连接对象
>         Connection connection = connectionFactory.newConnection();*/
> 
>         //通过工具类获取连接对象
>         Connection connection = RabbitMQUtils.getConnection();
> 
>         //获取连接中通道
>         Channel channel = connection.createChannel();
> 
>         //通道绑定对应消息队列
>         //参数1:  队列名称 如果队列不存在自动创建
>         //参数2:  用来定义队列特性是否要持久化 true 持久化队列   false 不持久化
>         //参数3:  exclusive 是否独占队列  true 独占队列   false  不独占
>         //参数4:  autoDelete: 是否在消费完成后自动删除队列  true 自动删除  false 不自动删除
>         //参数5:  额外附加参数
>         channel.queueDeclare("hello",true,false,false,null);
> 
>         //发布消息
> 
>         //参数1: 交换机名称 参数2:队列名称  参数3:传递消息额外设置  参数4:消息的具体内容
>         channel.basicPublish("","hello", MessageProperties.PERSISTENT_TEXT_PLAIN,"hello rabbitmq".getBytes());
> 
> 
>         /*channel.close();
>         connection.close();*/
> 
>         //调用工具类
>         RabbitMQUtils.closeConnectionAndChanel(channel,connection);
> 
> 
>     }
> }
> ```

4.4.2编写消费者

> ```java
> 
> package helloword;
> 
> import com.rabbitmq.client.Channel;
> import com.rabbitmq.client.Connection;
> import com.rabbitmq.client.MessageProperties;
> import org.junit.Test;
> import utils.RabbitMQUtils;
> 
> import java.io.IOException;
> import java.util.concurrent.TimeoutException;
> 
> public class Provider {
> 
>     //生产消息
>     @Test
>     public void testSendMessage() throws IOException, TimeoutException {
> /*
>         //创建连接mq的连接工厂对象
>         ConnectionFactory connectionFactory = new ConnectionFactory();
>         //设置连接rabbitmq主机
>         connectionFactory.setHost("192.168.11.134");
>         //设置端口号
>         connectionFactory.setPort(5672);
>         //设置连接那个虚拟主机
>         connectionFactory.setVirtualHost("/ems");
>         //设置访问虚拟主机的用户名和密码
>         connectionFactory.setUsername("admin");
>         connectionFactory.setPassword("admin");
> 
>         //获取连接对象
>         Connection connection = connectionFactory.newConnection();*/
> 
>         //通过工具类获取连接对象
>         Connection connection = RabbitMQUtils.getConnection();
> 
>         //获取连接中通道
>         Channel channel = connection.createChannel();
> 
>         //通道绑定对应消息队列
>         //参数1:  队列名称 如果队列不存在自动创建
>         //参数2:  用来定义队列特性是否要持久化 true 持久化队列   false 不持久化
>         //参数3:  exclusive 是否独占队列  true 独占队列   false  不独占
>         //参数4:  autoDelete: 是否在消费完成后自动删除队列  true 自动删除  false 不自动删除
>         //参数5:  额外附加参数
>         channel.queueDeclare("hello",true,false,false,null);
> 
>         //发布消息
> 
>         //参数1: 交换机名称 参数2:队列名称  参数3:传递消息额外设置  参数4:消息的具体内容
>         channel.basicPublish("","hello", MessageProperties.PERSISTENT_TEXT_PLAIN,"hello rabbitmq".getBytes());
> 
> 
>         /*channel.close();
>         connection.close();*/
> 
>         //调用工具类
>         RabbitMQUtils.closeConnectionAndChanel(channel,connection);
> 
> 
>     }
> }
> 
> ```

4.4.3 参数的说明

> channel.queueDeclare("hello",true,false,false,null);
> 					'参数1':用来声明通道对应的队列
> 					'参数2':用来指定是否持久化队列
> 					'参数3':用来指定是否独占队列
> 					'参数4':用来指定是否自动删除队列
> 					'参数5':对队列的额外配置

4.4.4 工具类的封装 

> ```java
> package utils;
> 
> 
> import com.rabbitmq.client.Channel;
> import com.rabbitmq.client.Connection;
> import com.rabbitmq.client.ConnectionFactory;
> 
> import java.util.Properties;
> 
> public class RabbitMQUtils {
> 
>     private static ConnectionFactory connectionFactory;
>     private static Properties properties;
>     static{
>         //重量级资源  类加载执行之执行一次
>         connectionFactory = new ConnectionFactory();
>         connectionFactory.setHost("192.168.42.134");
>         connectionFactory.setPort(5672);
>         connectionFactory.setVirtualHost("/");
>         connectionFactory.setUsername("admin");
>         connectionFactory.setPassword("admin");
> 
>     }
> 
>     //定义提供连接对象的方法
>     public static Connection getConnection() {
>         try {
>             return connectionFactory.newConnection();
>         } catch (Exception e) {
>             e.printStackTrace();
>         }
>         return null;
>     }
> 
>     //关闭通道和关闭连接工具方法
>     public static void closeConnectionAndChanel(Channel channel, Connection conn) {
>         try {
>             if(channel!=null) channel.close();
>             if(conn!=null)   conn.close();
>         } catch (Exception e) {
>             e.printStackTrace();
> 
>         }
>     }
> 
>     public static void main(String[] args) {
>         //System.out.println("RabbitMQUtils.getConnection() = " + RabbitMQUtils.getConnection());
>     }
> }
> ```
>
> 

