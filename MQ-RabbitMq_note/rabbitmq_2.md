# RabbitMQ

## 第二种模型 work queue

> Work queues，也被称为（Task queues），任务模型。
> 		当消息处理比较耗时的时候，可能生产消息的速度会远远大于消息的消费速度。长此以往，消息就会堆积越来越多，无法及时处理。此时就可以使用work 模型：让多个消费者绑定到一个队列，共同消费队列中的消息。队列中的消息一旦消费，就会消失，因此任务是不会被重复执行的。

> ![1633508010965](D:\XIE_Tips\MQ-RabbitMq_note\images\1633508010965.png)

> 角色：
>
> ​		P：生产者：任务的发布者
> ​		C1：消费者-1，领取任务并且完成任务，假设完成速度较慢
> ​		C2：消费者-2：领取任务并完成任务，假设完成速度快

> 生产者：
>
> ```java
> channel.queueDeclare("hello", true, false, false, null);
> for (int i = 0; i < 10; i++) {
>   channel.basicPublish("", "hello", null, (i+"====>:我是消息").getBytes());
> }
> 
> ```
>
> 消费者 1 - 2 - 3....
>
> ```java
> channel.queueDeclare("hello",true,false,false,null);
> channel.basicConsume("hello",true,new DefaultConsumer(channel){
>   @Override
>   public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
>     System.out.println("消费者1: "+new String(body));
>   }
> });
> 
> ```
>
> 测试结果
>
> ​		总结:默认情况下，RabbitMQ将按顺序将每个消息发送给下一个使用者。平均而言，每个消费者都会收到相同数量的消息。这种分发消息的方式称为循环。

### 4.5.1  消息自动确认机制

> Doing a task can take a few seconds. You may wonder what happens if one of the consumers 
> starts a long task and dies with it only partly done. With our current code,once RabbitMQ delivers a message to the consumer it immediately marks it for deletion. In this case, if you kill a worker we will lose the message it was just processing. We'llalso lose all the messages that  were dispatched to this particular worker but were not yet handled. But we don't want to lose any tasks. If a worker dies,we'd like the task to be delivered to another worker.
>
> 完成一项任务可能需要几秒钟。您可能想知道，如果其中一个消费者开始了一项长期任务，但只完成了一部分就死了，会发生什么情况。在我们当前的代码中，一旦RabbitMQ将消息传递给使用者，它就会立即将其标记为删除。在这种情况下，如果您杀死一个worker，我们将丢失它刚刚处理的消息。我们还将丢失发送给该特定工作进程但尚未处理的所有消息。
>
> **但我们不想失去任何任务。如果一个worker死了，我们希望把任务交给另一个工人。**

> 生产者不变
>
> 消费者：
>
> ```java
> 消费之1
> package workquene;
> 
> import com.rabbitmq.client.*;
> import utils.RabbitMQUtils;
> 
> import java.io.IOException;
> 
> public class Customer1 {
>     public static void main(String[] args) throws IOException {
> 
>         //获取连接
>         Connection connection = RabbitMQUtils.getConnection();
>         final Channel channel = connection.createChannel();
>         channel.basicQos(1);//每一次只能消费一个消息
>         channel.queueDeclare("work",true,false,false,null);
>         //参数1:队列名称  参数2:消息自动确认 true  消费者自动向rabbitmq确认消息消费  false 不会自动确认消息
>         channel.basicConsume("work",false,new DefaultConsumer(channel){
>             @Override
>             public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
>                 try{
>                     Thread.sleep(2000);
>                 }catch (Exception e){
>                     e.printStackTrace();
>                 }
>                 System.out.println("消费者-1: "+new String(body));
>                 // 参数1:确认队列中那个具体消息 参数2:是否开启多个消息同时确实
>                 channel.basicAck(envelope.getDeliveryTag(),false);
>             }
>         });
> 
> 
> 
>     }
> }
> 
> ```
>
> **主要修改每次消费只拿一条数据 不进行一次缓存多条**  
>
> **其次 取消自动确认机制   在每次消费完一条数据进行手动确认**
>
> 设置通道一次只能消费一个消息
> 关闭消息的自动确认,开启手动确认消息

## 第三种模型 fanout

> fanout 扇出 也称为广播
>
> ![1633509607965](D:\XIE_Tips\MQ-RabbitMq_note\images\1633509607965.png)

> **在广播模式下，消息发送流程是这样的：**
>
> **可以有多个消费者**
> 		**每个消费者有自己的queue（队列）**
> 		**每个队列都要绑定到Exchange（交换机）**
> 		**生产者发送的消息，只能发送到交换机，**
> 		**交换机来决定要发给哪个队列，生产者无法决定。**
> 		**交换机把消息发送给绑定过的所有队列**
> 		**队列的消费者都能拿到消息。实现一条消息被多个消费者消费**

> **区别于第二种模型：  这种模型是一份消息被多个消费者共享所消费**
>
> **而第三种模型是  每个消费者都各自消费一份 并非共享消费**

> 生产者：
>
> ```java
> package fanout;
> 
> import com.rabbitmq.client.Channel;
> import com.rabbitmq.client.Connection;
> import utils.RabbitMQUtils;
> 
> import java.io.IOException;
> 
> public class Provider {
> 
>     public static void main(String[] args) throws IOException {
>         //获取连接对象
>         Connection connection = RabbitMQUtils.getConnection();
>         Channel channel = connection.createChannel();
> 
>         //将通道声明指定交换机   //参数1: 交换机名称    参数2: 交换机类型  fanout 广播类型
>         channel.exchangeDeclare("logs","fanout");
> 
>         //发送消息
>         channel.basicPublish("logs","",null,"fanout type message".getBytes());
> 
>         //释放资源
>         RabbitMQUtils.closeConnectionAndChanel(channel,connection);
> 
>     }
> }
> 
> 
> ```
>
> 多个消费者
>
> ```java
> package fanout;
> 
> import com.rabbitmq.client.*;
> import utils.RabbitMQUtils;
> 
> import java.io.IOException;
> 
> public class Customer1 {
>     public static void main(String[] args) throws IOException {
>         //获取连接对象
>         Connection connection = RabbitMQUtils.getConnection();
>         Channel channel = connection.createChannel();
> 
>         //通道绑定交换机
>         channel.exchangeDeclare("logs","fanout");
> 
>         //临时队列
>         String queueName = channel.queueDeclare().getQueue();
> 
>         //绑定交换机和队列
>         channel.queueBind(queueName,"logs","");
> 
>         //消费消息
>         channel.basicConsume(queueName,true,new DefaultConsumer(channel){
>             @Override
>             public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
>                 System.out.println("消费者1: "+new String(body));
>             }
>         });
> 
> 
> 
>     }
> }
> 
> ```



## 第四种模型 Routing -direct

### Routing 订阅模型  -- direct

> **在Fanout模式中，一条消息，会被所有订阅的队列都消费。**
> 但是，在某些场景下，**我们希望不同的消息被不同的队列消费。**
> **这时就要用到Direct类型的Exchange。**

> ​	**在Direct模型下：队列与交换机的绑定，不能是任意绑定了，而是要指定一个RoutingKey（路由key）消息的发送方在 向 Exchange发送消息时，也必须指定消息的 RoutingKey。**
>
> ​	**Exchange不再把消息交给每一个绑定的队列，而是根据消息的Routing Key进行判断，只有队列的Routingkey与消息的 Routing key完全一致，才会接收到消息**

> ![1633524823844](D:\XIE_Tips\MQ-RabbitMq_note\images\1633524823844.png)

> ​	P：生产者，向Exchange发送消息，发送消息时，会指定一个routing key。
> ​	X：Exchange（交换机），接收生产者的消息，然后把消息递交给 与routing key完全匹配的队列
> ​	C1：消费者，其所在队列指定了需要routing key 为 error 的消息
> ​	C2：消费者，其所在队列指定了需要routing key 为 info、error、warning 的消息

> 生产者：
>
> ```java
> package direct;
> 
> import com.rabbitmq.client.Channel;
> import com.rabbitmq.client.Connection;
> import utils.RabbitMQUtils;
> 
> import java.io.IOException;
> 
> public class Provider {
>     public static void main(String[] args) throws IOException {
>         //获取连接对象
>         Connection connection = RabbitMQUtils.getConnection();
>         //获取连接通道对象
>         Channel channel = connection.createChannel();
>         String exchangeName = "logs_direct";
>         //通过通道声明交换机  参数1:交换机名称  参数2:direct  路由模式
>         channel.exchangeDeclare(exchangeName,"direct");
>         //发送消息
>         String routingkey = "info";
>         channel.basicPublish(exchangeName,routingkey,null,("这是direct模型发布的基于route key: ["+routingkey+"] 发送的消息").getBytes());
> 
>         //关闭资源
>         RabbitMQUtils.closeConnectionAndChanel(channel,connection);
>     }
> }
> 
> 
> ```
>
> 消费者：
>
> ```java
> package direct;
> 
> import com.rabbitmq.client.*;
> import utils.RabbitMQUtils;
> 
> import java.io.IOException;
> 
> /**
>  * @author Administrator
>  */
> public class Consumer1 {
> 
>     public static void main(String[] args) throws IOException {
> 
>         Connection connection = RabbitMQUtils.getConnection();
> 
>         Channel channel = connection.createChannel();
> 
>         //声明通道所对应交换机的名称以及类型
>         channel.exchangeDeclare("log_direct", BuiltinExchangeType.DIRECT);
> 
>         //创建临时队列
>         String temqueue = channel.queueDeclare().getQueue();
> 
>         //通道    绑定临时队列  以及指定的路由键  交换机
>         channel.queueBind(temqueue,"log_direct","error");
> 
>         //消费消息
>         channel.basicConsume(temqueue,true,new DefaultConsumer(channel){
>             @Override
>             public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
>                 System.out.println("consumer1----------"+new String(body));
>             }
>         });
> 
>     }
> 
> }
> 
> 
> ```
>
> 

## 第五种模型  Routing -- topic

> ​	Topic类型的Exchange与Direct相比，都是可以根据RoutingKey把消息路由到不同的队列。只不过**Topic类型Exchange可以让队列在绑定Routing key的时候使用通配符！这种模型Routingkey** 
> **一般都是由一个或多个单词组成，多个单词之间以”.”分割，例如： item.insert**

> #### 统配符
> 		* (star) can substitute for exactly one word.    匹配不多不少恰好1个词
> 		# (hash) can substitute for zero or more words.  匹配零个、一个或多个词
> ##### 如:
> 		audit.#    匹配audit、audit.irs 、或者audit.irs.corporate等
> 	audit.*   只能匹配 audit.irs

> 生产者：
>
> ```java
> package topic;
> 
> import com.rabbitmq.client.BuiltinExchangeType;
> import com.rabbitmq.client.Channel;
> import com.rabbitmq.client.Connection;
> import org.junit.jupiter.api.Test;
> import utils.RabbitMQUtils;
> 
> import java.io.IOException;
> 
> /**
>  * @author Administrator
>  */
> public class Provider {
>     @Test
>     public void sendMessage() throws IOException {
> 
>         Connection connection = RabbitMQUtils.getConnection();
> 
>         Channel channel = connection.createChannel();
> 
>         //通道声明 交换机名称以及类型
>         channel.exchangeDeclare("log_topic", BuiltinExchangeType.TOPIC);
> 
> 
>         String routingKey = "user.save.delete";
> 
>         channel.basicPublish("log_topic",routingKey,null,("这是路由模型topic:["+routingKey+"]").getBytes());
> 
>         //释放资源
>         RabbitMQUtils.closeChannelAndConnection(channel,connection);
> 
>     }
> 
> 
> }
> 
> ```

> 消费者
>
> ```java
> package topic;
> 
> import com.rabbitmq.client.*;
> import utils.RabbitMQUtils;
> 
> import java.io.IOException;
> 
> /**
>  * @author Administrator
>  */
> public class Consumer1 {
> 
>     public static void main(String[] args) throws IOException {
> 
>         Connection connection = RabbitMQUtils.getConnection();
> 
>         Channel channel = connection.createChannel();
> 
>         //通道声明 交换机名称以及类型
>         channel.exchangeDeclare("log_topic", BuiltinExchangeType.TOPIC);
> 
>         //创建临时队列
>         String tempqueue = channel.queueDeclare().getQueue();
> 
>         //通道绑定  指定临时队列  交换机名称  以及  匹配路由
>         channel.queueBind(tempqueue,"log_topic","user.*");
> 
>         //消费消息
>         channel.basicConsume(tempqueue,true,new DefaultConsumer(channel){
> 
>             @Override
>             public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
>                 System.out.println("conseumer1---"+new String(body));
>             }
>         });
> 
>     }
> 
> }
> 
> ```
>
> 

> 枚举类型模仿
>
> ```java
> /**
>  * Enum for built-in exchange types.
>  */
> public enum BuiltinExchangeType {
> 
>     DIRECT("direct"), FANOUT("fanout"), TOPIC("topic"), HEADERS("headers");
> 
>     private final String type;
> 
>     BuiltinExchangeType(String type) {
>         this.type = type;
>     }
> 
>     public String getType() {
>         return type;
>     }
> }
> 
> ```