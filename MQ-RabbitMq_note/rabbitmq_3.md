

# RabbitMQ

## SpringBoot中使用RabbitMQ

> #### 搭建初始环境
>
> 1. ##### 引入依赖
>
>    ```java
>    <dependency>
>      <groupId>org.springframework.boot</groupId>
>      <artifactId>spring-boot-starter-amqp</artifactId>
>    </dependency>
>    
>    ```
>
> 2. ##### 编写配置文件(application.yml)
>
>    ```java
>    spring:
>      application:
>        name: springboot-rabbitmq
>      rabbitmq:
>        host: 127.0.0.1
>        virtual-host: xxx
>        port: 5672
>        username: xps
>        password: 1234
>    ```
>
> 3. ##### 第一种hello模型的使用  （点对点）
>
>    **RabbitTemplate 在编写好配置文件 springboot会自动整合链接rabbitmq**
>
>    **提供了RabbitTemplate模板来使用 简化开发**  
>
>    **使用的时候直接在项目中注入即可**
>
>    ###### 3.1 生产者编写
>
>    ```java
>    package com.example;
>    
>    import org.junit.jupiter.api.Test;
>    import org.springframework.amqp.rabbit.core.RabbitTemplate;
>    import org.springframework.beans.factory.annotation.Autowired;
>    import org.springframework.boot.test.context.SpringBootTest;
>    
>    @SpringBootTest
>    class DemoApplicationTests {
>    
>    
>        @Autowired
>        private RabbitTemplate rabbitTemplate;
>        @Test
>        void contextLoads() {
>            rabbitTemplate.convertAndSend("hello","hello world"); 
>    // 生产端没有指定交换机只有routingKey和Object。
>    //消费方产生hello队列，放在默认的交换机(AMQP default)上。
>    //而默认的交换机有一个特点，只要你的routerKey的名字与这个
>    //交换机的队列有相同的名字，他就会自动路由上。 
>    //生产端routingKey 叫hello ，消费端生产hello队列。
>    //他们就路由上了
>        }
>    
>    }
>    
>    ```
>
>    ###### 3.2消费者
>
>    ```java
>    import org.springframework.amqp.rabbit.annotation.Queue;
>    import org.springframework.amqp.rabbit.annotation.RabbitHandler;
>    import org.springframework.amqp.rabbit.annotation.RabbitListener;
>    import org.springframework.stereotype.Component;
>    
>    @Component
>    // 生产端没有指定交换机只有routingKey和Object。
>    //消费方产生hello队列，放在默认的交换机(AMQP default)上。
>    //而默认的交换机有一个特点，只要你的routerKey的名字与这个
>    //交换机的队列有相同的名字，他就会自动路由上。 
>    //生产端routingKey 叫hello ，消费端生产hello队列。
>    //他们就路由上了
>    @RabbitListener(queuesToDeclare = @Queue(value = "hello"))
>    public class HelloCustomer {
>    
>        @RabbitHandler
>        public void receive1(String message){
>            System.out.println("message = " + message);
>        }
>    }
>    ```
>
> 4. ##### 第二中work模型的使用
>
>    ###### 4.1 生产者
>
>    ```java
>        //第二种 work 模型  1对多  轮询  还是直接发送消息到消息队列
>        @Test
>        void testWorkModel(){
>            for (int i = 0; i < 10; i++) {
>                rabbitTemplate.convertAndSend("work-1","work模型 消息发送 ......."+i);
>            }
>        }
>    
>    ```
>
>    ###### 4.2 消费者
>
>    ```java
>    package com.xieps.springbootintegerationrabbitmq.work;
>    
>    import org.springframework.amqp.rabbit.annotation.Queue;
>    import org.springframework.amqp.rabbit.annotation.RabbitHandler;
>    import org.springframework.amqp.rabbit.annotation.RabbitListener;
>    import org.springframework.stereotype.Component;
>    
>    /**
>     * @author Administrator
>     */
>    @Component
>    
>    public class WorkModelConsumerConfig {
>    
>        @RabbitListener(queuesToDeclare = @Queue("work-1"))
>        public void receive1(String message){
>            System.out.println("consumer1---"+message);
>        }
>    
>    
>        @RabbitListener(queuesToDeclare = @Queue("work-1"))
>        public void receive2(String message){
>            System.out.println("consumer2----"+message);
>        }
>    
>    }
>    
>    ```
>
>    ##### 5.第三种 fanout 广播模型
>
>    ###### 5.1生产者
>
>    ```java
>        /**
>         * fanout 广播模型  生产者发送消息到交换机  由交换机去分发到消息队列
>         */
>        @Test
>        public void testExchangeModel(){
>    
>            rabbitTemplate.convertAndSend("log-exchange","","fanout 广播 直接发送消息到交换机..");
>    
>        }
>    
>    ```
>
>    ###### 5.2消费者
>
>    ```java
>    package com.xieps.springbootintegerationrabbitmq.fanout;
>    
>    import com.rabbitmq.client.BuiltinExchangeType;
>    import org.springframework.amqp.rabbit.annotation.Exchange;
>    import org.springframework.amqp.rabbit.annotation.Queue;
>    import org.springframework.amqp.rabbit.annotation.QueueBinding;
>    import org.springframework.amqp.rabbit.annotation.RabbitListener;
>    import org.springframework.stereotype.Component;
>    
>    /**
>     * @author Administrator
>     */
>    @Component
>    public class FanoutModelConsumerConfig {
>    
>        @RabbitListener(bindings = {
>              @QueueBinding(
>                      value = @Queue, //采用默认创建的临时队列
>                      exchange = @Exchange(name = "log-exchange",type = "fanout")
>              )
>        })
>        public void receive(String message){
>            System.out.println("consumer1---"+message);
>        }
>    
>        @RabbitListener(bindings = {
>                @QueueBinding(
>                        value = @Queue,
>                        exchange = @Exchange(name = "log-exchange",type = "fanout")
>                )
>        })
>        public void receive2(String message){
>            System.out.println("consumer2*****"+message);
>        }
>    
>    }
>    
>    ```
>
>    ##### 6.第四种 路由模式 direct
>
>    ###### 6.1生产者
>
>    ```java
>        //第四种 路由模式  根据路由键 发送到指定的队列中
>        @Test
>        public void routeModel(){
>            rabbitTemplate.convertAndSend("log-route","info","路由模式 根据指定的key发送 消息队列根据key进行接收....");
>        }
>    ```
>
>    ###### 6.2消费者
>
>    ```java
>    package com.xieps.springbootintegerationrabbitmq.route;
>    
>    import org.springframework.amqp.rabbit.annotation.Exchange;
>    import org.springframework.amqp.rabbit.annotation.Queue;
>    import org.springframework.amqp.rabbit.annotation.QueueBinding;
>    import org.springframework.amqp.rabbit.annotation.RabbitListener;
>    import org.springframework.stereotype.Component;
>    
>    @Component
>    public class RouteModelConsumerConfig {
>    
>        @RabbitListener(bindings = {
>                @QueueBinding(
>                        value = @Queue,
>                        key = {"error","info"},
>                        exchange = @Exchange(name = "log-route",type = "direct")
>                )
>        })
>        public void receive1(String message){
>            System.out.println("consumer1----"+message);
>        }
>    
>        @RabbitListener(bindings = {
>                @QueueBinding(
>                        value = @Queue,
>                        key = {"error"},
>                        exchange = @Exchange(name="log-route",type = "direct")
>                )
>        })
>        public void receive2(String message){
>            System.out.println("consumer2*******"+message);
>        }
>    
>    }
>    
>    ```
>
>    ##### 7.动态路由 加入通配符 topic
>
>    ###### 7.1生产者
>
>    ```java
>        //第五种 动态路由  通过统配字符发送到指定的消息队列中
>        @Test
>        public void topicModel(){
>            rabbitTemplate.convertAndSend("log-topic","user.save.eeee","动态路由模式 通配符进行发送到指定消息队列");
>        }
>    ```
>
>    ###### 7.2消费者
>
>    ```java
>    package com.xieps.springbootintegerationrabbitmq.topic;
>    
>    import org.springframework.amqp.rabbit.annotation.Exchange;
>    import org.springframework.amqp.rabbit.annotation.Queue;
>    import org.springframework.amqp.rabbit.annotation.QueueBinding;
>    import org.springframework.amqp.rabbit.annotation.RabbitListener;
>    import org.springframework.stereotype.Component;
>    
>    @Component
>    public class TopicModelConsumerConfig {
>    
>        @RabbitListener(bindings = {
>                @QueueBinding(
>                        value = @Queue,
>                        key = {"user.save","user.*"},
>                        exchange = @Exchange(name = "log-topic",type = "topic")
>                )
>        })
>        public void receive1(String message){
>            System.out.println("consumer1++++++"+message);
>        }
>    
>        @RabbitListener(bindings = {
>                @QueueBinding(
>                        value = @Queue,
>                        key = {"user.#","user.save"},
>                        exchange = @Exchange(name = "log-topic",type = "topic")
>                )
>        })
>        public void receive2(String message){
>            System.out.println("consumer2//////////////"+message);
>        }
>    
>    }
>    
>    ```
>
>    
>
>    