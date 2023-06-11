# 	 RabbitMq 学习笔记



## MQ的概念

### 1.1 什么是MQ

1. MQ（message queue） 消息队列
2. MQ是一种非常常见的上下游"逻辑解耦+物理解耦"的消息通信服务

### 1.2MQ功能

1. 流量消峰
2. 应用解耦
3. 异步处理

## MQ分类

+ ActiveMQ
+ Kafka
+ RocketMQ
+ RabbitMQ  

## RabbitMQ

### 四大核心概念

+ 生产者
+ 交换机
+ 队列
+ 消费者

## 安装

### docker

```
docker pull rabbitmq
```

**需要注意**的是`-p 5673:5672` 解释：-p 外网端口：docker的内部端口 ，你们可以改成自己的外网端口号，我这里映射的外网端口是5672那么程序连接端口就是用5672

```
 docker run -d -p 5672:5672 -p 15672:15672 --name myRabbitmq rabbitmq:management
```

查看容器

```
docker ps
```

进入容器

```
docker exec -it 容器id/容器name /bin/bash
```



## 简单使用

### 生产者

1. 首先需要创建队列名字
2. 然后创建连接工厂
3. 输入用户名 密码 地址等信息
4. 创建连接
5. 获取信道
6. 通过获取的信道创建队列
   1. 第一个参数是 队列名称
   2. 第二个参数是 是否持久化
   3.  该队列是否只对一个消费者进行消费,是否进行共享 true表示多个消费者消费
   4. 是否自动删除 最后一个消费者断开连接后 是否自动删除队列 fa不自动删除
   5. 其他参数
7. 通过信道传输信息
   1. 第一个参数 发送到那个交换机
   2. 路由的key是那个 本次是队列的名称
   3. 其他的参数信息
   4. 发送消息的消息体

~~~java
package com.it.rabbitmq.one;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import javax.xml.soap.SAAJResult;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;

/**
 * Created with IntelliJ IDEA.
 *
 * @Author: 爱吃山楂的小天
 * @Date: 2022/08/10/10:41
 * @Description: 生产者
 */
public class Producer {

    //队列名称
    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 工厂IP 连接RabbitMQ的队列
        factory.setHost("1.15.85.110");
        //用户名
        factory.setUsername("acszdxt");
        //密码
        factory.setPassword("acszdxt");
        //创建连接
        Connection connection = factory.newConnection();
        //获取信道
        Channel channel = connection.createChannel();
        //创建队列
        /**
         * 第一个参数队列名称
         * 是否需要保存消息(持久化,默认情况存在内存
         * 该队列是否只对一个消费者进行消费,是否进行共享 ,true可以多个消费者消费 false,只能一个消费者消费
         * 最后一个消费者断开连接是否自动删除
         * 其他参数
         */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        // 发消息
        String message = "Hello world";

        //用信道发送消息
        /**
         *  交换机名称
         *  路由key值是那个 本次是队列名称
         *  其他参数
         *  消息体
         */
        channel.basicPublish("",QUEUE_NAME,null,message.getBytes(StandardCharsets.UTF_8));
        System.out.println("消息发送完毕");
    }
}

~~~

### 消费者

1. 首先需要创建队列名字
2. 然后创建连接工厂
3. 输入用户名 密码 地址等信息
4. 创建连接
5. 获取信道
6. 通过信道收消息
   1. 消息的队列名称
   2. 消费成功之后是否自动应答 true自动
   3. *当一个消息发送过来后的回调接口*
   4. 消费取消的回调

~~~java
package com.it.rabbitmq.one;

import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * Created with IntelliJ IDEA.
 *
 * @Author: 爱吃山楂的小天
 * @Date: 2022/08/10/11:02
 * @Description: 消费者
 */
public class Consumer {
    //队列名称 要根生产者的队列名称一样
    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 工厂IP 连接RabbitMQ的队列
        factory.setHost("1.15.85.110");
        //用户名
        factory.setUsername("acszdxt");
        //密码
        factory.setPassword("acszdxt");
        //创建连接
        Connection connection = factory.newConnection();
        //创建信道
        Channel channel = connection.createChannel();
        //
        DeliverCallback deliverCallback = (consumerTag, message) -> {

            System.out.println(new String(message.getBody()));
        };

        // 取消消费回调
        CancelCallback cancelCallback = consumerTag -> {
            System.out.println("消费者取消消费");
        };

        /**
         * 消费者收消息
         * 队列名称
         * 消费成功后是否要自动应答
         * 消费成功的回调
         * 消费者取消消费的回调
         */
        channel.basicConsume(QUEUE_NAME, true, deliverCallback,cancelCallback);
    }
}

~~~

~~~java
/**
 * Created with IntelliJ IDEA.
 *
 * @Author: 爱吃山楂的小天
 * @Date: 2022/08/10/16:54
 * @Description:
 */
public class Consumer02 {

    public static final String QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMqUtils.getChannel();
        System.out.println("c2等待处理消息时间较长");
        //采用手动应答
        boolean autoAck = false;

        DeliverCallback deliverCallback = (consumerTag, message) -> {

            try {
                Thread.sleep(30000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }

            System.out.println("接收的消息" + new String(message.getBody()));

            //手动应答
            /**
             * 消息的标记
             * 是否批量应答
             */
            channel.basicAck(message.getEnvelope().getDeliveryTag(),false);
        };
        CancelCallback cancelCallback = consumerTag -> {
            System.out.println(consumerTag + "消息被消费者取消的回调");
        };

        channel.basicConsume(QUEUE_NAME,autoAck,deliverCallback,cancelCallback);
    }
}

~~~

![image-20220810171421060](C:\Users\22446\AppData\Roaming\Typora\typora-user-images\image-20220810171421060.png)

![image-20220810171434040](C:\Users\22446\AppData\Roaming\Typora\typora-user-images\image-20220810171434040.png)

![image-20220810171447816](C:\Users\22446\AppData\Roaming\Typora\typora-user-images\image-20220810171447816.png)

手动应答消息不会丢失



##  Hello world

### 引入依赖

~~~xml
<dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.7.3</version>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.11.0</version>
        </dependency>
~~~

### 生产者代码

~~~java
/**
 * Created with IntelliJ IDEA.
 *
 * @Author: 爱吃山楂的小天
 * @Date: 2022/08/10/10:41
 * @Description: 发信息
 */
public class Producer {

    //队列名称
    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 工厂IP 连接RabbitMQ的队列
        factory.setHost("1.15.85.110");
        //用户名
        factory.setUsername("acszdxt");
        //密码
        factory.setPassword("Acszdxt!096");
        //创建连接
        Connection connection = factory.newConnection();
        //获取信道
        Channel channel = connection.createChannel();
        //创建队列
        /**
         * 第一个参数队列名称
         * 是否需要保存消息(持久化,默认情况存在内存
         * 该队列是否只对一个消费者进行消费,是否进行共享 ,true可以多个消费者消费 false,只能一个消费者消费
         * 最后一个消费者断开连接是否自动删除
         * 其他参数
         */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        // 发消息
        String message = "Hello world";

        //用信道发送消息
        /**
         *  交换机名称
         *  路由key值是那个 本次是队列名称
         *  其他参数
         *  消息体
         */
        channel.basicPublish("",QUEUE_NAME,null,message.getBytes(StandardCharsets.UTF_8));
        System.out.println("消息发送完毕");
    }
}

~~~

### 消费者代码

~~~java

/**
 * Created with IntelliJ IDEA.
 *
 * @Author: 爱吃山楂的小天
 * @Date: 2022/08/10/11:02
 * @Description: 消费者
 */
public class Consumer {
    //队列名称 要根生产者的队列名称一样
    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        // 创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 工厂IP 连接RabbitMQ的队列
        factory.setHost("1.15.85.110");
        //用户名
        factory.setUsername("acszdxt");
        //密码
        factory.setPassword("Acszdxt!096");
        //创建连接
        Connection connection = factory.newConnection();
        //创建信道
        Channel channel = connection.createChannel();
        //
        DeliverCallback deliverCallback = (consumerTag, message) -> {

            System.out.println(new String(message.getBody()));
        };

        // 取消消费回调
        CancelCallback cancelCallback = consumerTag -> {
            System.out.println("消费者取消消费");
        };

        /**
         * 消费者收消息
         * 队列名称
         * 消费成功后是否要自动应答
         * 消费成功的回调
         * 消费者取消消费的回调
         */
        channel.basicConsume(QUEUE_NAME, true, deliverCallback,cancelCallback);
    }
}
~~~

## Work Queues 工作队列 轮训工作模式

![image-20220810142431064](C:\Users\22446\AppData\Roaming\Typora\typora-user-images\image-20220810142431064.png)

### 生产者代码

~~~java
/**
 * Created with IntelliJ IDEA.
 *
 * @Author: 爱吃山楂的小天
 * @Date: 2022/08/10/14:56
 * @Description: 生产者
 */
public class Producer {

    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = RabbitMqUtils.getChannel();
        //队列声明
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()) {
            String message = scanner.next();
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println("消息发送完成" + message);
        }
    }
}
~~~



### 工作队列

可通过idea运行多次

```java
**
 * Created with IntelliJ IDEA.
 *
 * @Author: 爱吃山楂的小天
 * @Date: 2022/08/10/14:44
 * @Description: 工作线程（消费者）
 */
public class Worker01 {

    public static final String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMqUtils.getChannel();

        DeliverCallback deliverCallback = (consumerTag, message) -> {
            System.out.println("接收的消息" + new String(message.getBody()));
        };
        CancelCallback cancelCallback = consumerTag -> {
            System.out.println(consumerTag + "消息被消费者取消的回调");
        };
        //接收消息
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);

    }
}
```

## 消息应答

消费者完成一个任务可能需要一段时间，如果其中一个消费者处理一个长的任务并仅只完成了部分突然它挂掉了，会发生什么情况。RabbitMQ 一旦向消费者传递了一条消息，便立即将该消息标记为删除。在这种情况下，突然有个消费者挂掉了，我们将丢失正在处理的消息。以及后续发送给该消费这的消息，因为它无法接收到。

为了保证消息在发送过程中不丢失，引入消息应答机制，消息应答就是：**消费者在接收到消息并且处理该消息之后，告诉 rabbitmq 它已经处理了，rabbitmq 可以把该消息删除了。**

### 自动应答

消息发送后立即被认为已经传送成功，这种模式需要在**高吞吐量和数据传输安全性方面做权衡**,因为这种模式如果消息在接收到之前，消费者那边出现连接或者 channel 关闭，那么消息就丢失 了,当然另一方面这种模式消费者那边可以传递过载的消息，**没有对传递的消息数量进行限制**，当然这样有可能使得消费者这边由于接收太多还来不及处理的消息，导致这些消息的积压，最终使 得内存耗尽，最终这些消费者线程被操作系统杀死，**所以这种模式仅适用在消费者可以高效并以 某种速率能够处理这些消息的情况下使用。**

### 手动消息应答的方法

- Channel.basicAck(用于肯定确认)

  RabbitMQ 已知道该消息并且成功的处理消息，可以将其丢弃了

- Channel.basicNack(用于否定确认)

- Channel.basicReject(用于否定确认)

  与 Channel.basicNack 相比少一个参数，不处理该消息了直接拒绝，可以将其丢弃了

### **Multiple 的解释：**

手动应答的好处是可以批量应答并且减少网络拥堵true 代表批量应答 channel 上未应答的消息

- 比如说 channel 上有传送 tag 的消息 5,6,7,8 当前 tag 是8 那么此时5-8 的这些还未应答的消息都会被确认收到消息应答
- false 同上面相比只会应答 tag=8 的消息 5,6,7 这三个消息依然不会被确认收到消息应答

手动应答代码

channel.basicAck

1. 第一个参数是 消息的标记
2. 第二个参数是否 批量应答

```
channel.basicAck(message.getEnvelope().getDeliveryTag(),false);
```



### 消息自动重新入队

如果消费者由于某些原因失去连接(其通道已关闭，连接已关闭或 TCP 连接丢失)，导致消息未发送 ACK 确认，RabbitMQ 将了解到消息未完全处理，并将对其重新排队。如果此时其他消费者可以处理，它将很快将其重新分发给另一个消费者。这样，即使某个消费者偶尔死亡，也可以确保不会丢失任何消息。

![RabbitMQ-00000019](https://testingcf.jsdelivr.net/gh/oddfar/static/img/RabbitMQ/RabbitMQ-00000019.png)

## 消息持久化

当 RabbitMQ 服务停掉以后，消息生产者发送过来的消息不丢失要如何保障？默认情况下 RabbitMQ 退出或由于某种原因崩溃时，它忽视队列和消息，除非告知它不要这样做。确保消息不会丢失需要做两件事：**我们需要将队列和消息都标记为持久化。**

### 队列持久化

之前我们创建的队列都是非持久化的，rabbitmq 如果重启的化，该队列就会被删除掉，如果要队列实现持久化需要在声明队列的时候把 durable 参数设置为持久化

~~~
boolean durable = true;
     channel.queueDeclare(QUEUE_NAME,durable,false,false,null);
~~~

### 消息持久化

添加MessageProperties.PERSISTENT_TEXT_PLAIN属性（保存在磁盘）

~~~
channel.basicPublish("",QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes("UTF-8"));

~~~

将消息标记为持久化并不能完全保证不会丢失消息。尽管它告诉 RabbitMQ 将消息保存到磁盘，但是这里依然存在当消息刚准备存储在磁盘的时候 但是还没有存储完，消息还在缓存的一个间隔点。此时并没 有真正写入磁盘。持久性保证并不强，但是对于我们的简单任务队列而言，这已经绰绰有余了。

### 不公平分发

在最开始的时候我们学习到 RabbitMQ 分发消息采用的轮训分发，但是在某种场景下这种策略并不是很好，比方说有两个消费者在处理任务，其中有个**消费者 1** 处理任务的速度非常快，而另外一个**消费者 2** 处理速度却很慢，这个时候我们还是采用轮训分发的化就会到这处理速度快的这个消费者很大一部分时间处于空闲状态，而处理慢的那个消费者一直在干活，这种分配方式在这种情况下其实就不太好，但是 RabbitMQ 并不知道这种情况它依然很公平的进行分发。

为了避免这种情况，**在消费者中消费之前**，我们可以设置参数 `channel.basicQos(1);`

```
//不公平分发
int prefetchCount = 1;
channel.basicQos(prefetchCount);
//采用手动应答
boolean autoAck = false;
channel.basicConsume(TASK_QUEUE_NAME, autoAck, deliverCallback, cancelCallback);

```

### 预取值分发

带权的消息分发

本身消息的发送就是异步发送的，所以在任何时候，channel 上肯定不止只有一个消息另外来自消费 者的手动确认本质上也是异步的。因此这里就存在一个未确认的消息缓冲区，因此希望开发人员能**限制此缓冲区的大小**，**以避免缓冲区里面无限制的未确认消息问题**。这个时候就可以通过使用 basic.qos 方法设 置“预取计数”值来完成的。

该值定义通道上允许的未确认消息的最大数量。一旦数量达到配置的数量， RabbitMQ 将停止在通道上传递更多消息，除非至少有一个未处理的消息被确认，例如，假设在通道上有未确认的消息 5、6、7，8，并且通道的预取计数设置为 4，此时RabbitMQ 将不会在该通道上再传递任何消息，除非至少有一个未应答的消息被 ack。比方说 tag=6 这个消息刚刚被确认 ACK，RabbitMQ 将会感知 这个情况到并再发送一条消息。消息应答和 QoS 预取值对用户吞吐量有重大影响。

通常，增加预取将提高 向消费者传递消息的速度。**虽然自动应答传输消息速率是最佳的，但是，在这种情况下已传递但尚未处理的消息的数量也会增加，从而增加了消费者的 RAM 消耗**(随机存取存储器)应该小心使用具有无限预处理的自动确认模式或手动确认模式，消费者消费了大量的消息如果没有确认的话，会导致消费者连接节点的 内存消耗变大，所以找到合适的预取值是一个反复试验的过程，不同的负载该值取值也不同 100 到 300 范 围内的值通常可提供最佳的吞吐量，并且不会给消费者带来太大的风险。

预取值为 1 是最保守的。当然这将使吞吐量变得很低，特别是消费者连接延迟很严重的情况下，特别是在消费者连接等待时间较长的环境 中。对于大多数应用来说，稍微高一点的值将是最佳的。

# 发布确认

### 发布确认逻辑

生产者将信道设置成 confirm 模式，一旦信道进入 confirm 模式，所有在该信道上面发布的消息都将会被指派一个唯一的 ID(从 1 开始)，一旦消息被投递到所有匹配的队列之后，broker 就会发送一个确认给生产者(包含消息的唯一 ID)，这就使得生产者知道消息已经正确到达目的队列了，如果消息和队列是可持久化的，那么确认消息会在将消息写入磁盘之后发出，broker 回传给生产者的确认消息中 delivery-tag 域包含了确认消息的序列号，此外 broker 也可以设置basic.ack 的 multiple 域，表示到这个序列号之前的所有消息都已经得到了处理。

confirm 模式最大的好处在于他是异步的，一旦发布一条消息，生产者应用程序就可以在等信道返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者应用便可以通过回调方法来处理该确认消息，如果RabbitMQ 因为自身内部错误导致消息丢失，就会发送一条 nack 消息， 生产者应用程序同样可以在回调方法中处理该 nack 消息。

### 发布确认的策略

开启发布确认的方法:

发布确认默认是没有开启的，如果要开启需要调用方法 confirmSelect，每当你要想使用发布确认，都需要在 channel 上调用该方法

```java
//开启发布确认
channel.confirmSelect();
```

###  单个确认发布

这是一种简单的确认方式，它是一种**同步确认发布**的方式，也就是发布一个消息之后只有它被确认发布，后续的消息才能继续发布，`waitForConfirmsOrDie(long)` 这个方法只有在消息被确认的时候才返回，如果在指定时间范围内这个消息没有被确认那么它将抛出异常。

这种确认方式有一个最大的缺点就是：**发布速度特别的慢**，因为如果没有确认发布的消息就会阻塞所有后续消息的发布，这种方式最多提供每秒不超过数百条发布消息的吞吐量。当然对于某些应用程序来说这可能已经足够了。

```
    public static  void publishMessageIndividually() throws IOException, TimeoutException, InterruptedException {
        // 队列名称
        String uuid = UUID.randomUUID().toString();

        Channel channel = RabbitMqUtils.getChannel();
        //队列声明
        channel.queueDeclare(uuid,true,false,false,null);
        // 开启发布确认
        channel.confirmSelect();
        // 开始时间
        long begin = System.currentTimeMillis();
        //批量发消息
        for (int i = 0; i < MESSAGE_NUM; i++) {
            String msg = i + "";
            channel.basicPublish("",uuid, MessageProperties.PERSISTENT_TEXT_PLAIN,msg.getBytes());
            boolean flag = channel.waitForConfirms();
            if (flag) {
                System.out.println("消息发送成功");
            }

        }
        long end = System.currentTimeMillis();
        System.out.println("单个发布耗时:" +(end-begin));
    }
```



### 批量确认发布

上面那种方式非常慢，与单个等待确认消息相比，先发布一批消息然后一起确认可以极大地提高吞吐量，当然这种方式的缺点就是：当发生故障导致发布出现问题时，不知道是哪个消息出 问题了，我们必须将整个批处理保存在内存中，以记录重要的信息而后重新发布消息。当然这种方案仍然是同步的，也一样阻塞消息的发布。

步骤

- 先开启发布确认
- 创建安全线程map
- 创建 发布成功失败的确认回调 
  - 消息确认成功发布后在回调删除map里面的值
- 给信道添加监听器
- 发送消息
  - 发送时记录发送的消息

```
    public static  void publishMessageAsync() throws IOException, TimeoutException, InterruptedException {
        // 队列名称
        String uuid = UUID.randomUUID().toString();

        Channel channel = RabbitMqUtils.getChannel();
        //队列声明
        channel.queueDeclare(uuid,true,false,false,null);
        // 开启发布确认
        channel.confirmSelect();

        /**
         * 线程安全有序的哈希表 适用于高并发
         * 1 轻松的将序号与消息关联
         * 2 轻松的删除条目 只需给到序号
         * 3 支持高并发
         */
        ConcurrentSkipListMap<Long,String> outConfirms = new ConcurrentSkipListMap<>();

        // 开始时间
        long begin = System.currentTimeMillis();

        // 消息确认发送成功 回调
        /**
         *  tag 消息的标记
         *  是否为批量确认
         */
        ConfirmCallback ackCallback = (tag,multiple) -> {
            // 删除已经确认的消息
            // 如果是批量
            if (multiple) {
                // 删除确认到消息
                ConcurrentNavigableMap<Long, String> confirmed = outConfirms.headMap(tag);
                confirmed.clear();
            } else {
                outConfirms.remove(tag);
            }
            System.out.println("发送的消息"+tag);

        };
        // 消息确认发送失败 回调
        ConfirmCallback nackCallback = (tag,multiple) -> {
            String str = outConfirms.get(tag);
            System.out.println("未发送的消息"+str);
            // 打印未确认的消息
        };
        // 准备消息的监听器 监听哪些消息成功哪些失败
        channel.addConfirmListener(ackCallback,nackCallback); //异步

        for (int i = 0; i < MESSAGE_NUM; i++) {
            String msg = i + "";
            channel.basicPublish("",uuid, MessageProperties.PERSISTENT_TEXT_PLAIN,msg.getBytes());

            // 记录以及发出的消息
            outConfirms.put(channel.getNextPublishSeqNo(),msg);
        }
        long end = System.currentTimeMillis();
        System.out.println("异步发布耗时:" +(end-begin));

    }
```

### 异步确认发布

- 先创建队列

  ```
  //队列声明
  channel.queueDeclare(uuid,true,false,false,null);
  ```

- 开启发布确认

  ```
  // 开启发布确认
  channel.confirmSelect();
  ```

- 创建一个线程安全的并且支持高并发的map

  ```
   /**
           * 线程安全有序的哈希表 适用于高并发
           * 1 轻松的将序号与消息关联
           * 2 轻松的删除条目 只需给到序号
           * 3 支持高并发
           */
          ConcurrentSkipListMap<Long,String> outConfirms = new ConcurrentSkipListMap<>();
  
  ```

- 创建消息返送成功监听器回调

  ```
   ConfirmCallback ackCallback = (tag,multiple) -> {
              // 删除已经确认的消息
              // 如果是批量
              if (multiple) {
                  // 删除确认到消息
                  ConcurrentNavigableMap<Long, String> confirmed = outConfirms.headMap(tag);
                  confirmed.clear();
              } else {
                  outConfirms.remove(tag);
              }
              System.out.println("发送的消息"+tag);
  
          };  // 消息确认发送失败 回调
          ConfirmCallback nackCallback = (tag,multiple) -> {
              String str = outConfirms.get(tag);
              System.out.println("未发送的消息"+str);
              // 打印未确认的消息
          };
  ```

- 创建消息返送失败监听器回调

  ```
    // 消息确认发送失败 回调
          ConfirmCallback nackCallback = (tag,multiple) -> {
              String str = outConfirms.get(tag);
              System.out.println("未发送的消息"+str);
              // 打印未确认的消息
          };
  ```

- 消息的监听器 监听哪些消息成功哪些失败

  ```
   channel.addConfirmListener(ackCallback,nackCallback); //异步
  ```

- 最后发送消息

## 死心队列

### 发送方

设置发送消息过期时间

```
 // 死心消息 设置ttl 10 秒
        AMQP.BasicProperties properties = new AMQP.BasicProperties()
                .builder().expiration("10000").build();
        for (int i = 0; i < 10; i++) {
            String str =  "info"+ i;
            channel.basicPublish(NORMAL_EXCHANGE,"zs",properties,str.getBytes());
        }
```

### 接收方

创建死信队列交换机 和正常交换机队列

```
 channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
 channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);
 channel.queueDeclare(NORMAL_QUEUE,false,false,false,arguments);
 channel.queueDeclare(DEAD_QUEUE,false,false,false,null);

```

创建正常队列时要设置死信交换机 

需要创建一个map集合 把信息添加到map里面 但是key不能改变

```
		Map<String, Object> arguments = new HashMap<>();
		//过期时间
        //arguments.put("x-message-ttl",10000);
        // 正常队列设置死信交换机
        arguments.put("x-dead-letter-exchange",DEAD_EXCHANGE);
        // 设置死信key
        arguments.put("x-dead-letter-routing-key","lisi");
```

然后在创建正常队列时把map集合放最后一个参数里面

### 参数

```
x-message-ttl 过期时间
	arguments.put("x-message-ttl",10000);
x-dead-letter-exchange 正常队列设置死信交换机
	arguments.put("x-dead-letter-exchange",DEAD_EXCHANGE);
设置死信key
	arguments.put("x-dead-letter-routing-key","lisi");
x-max-length 设置队列最大长度
	arguments.put("x-max-length","6");
```

## 延迟发送消息

先安装rabbitmq延迟插件

声明交换机队列

交换机

```
 @Bean
    public CustomExchange delayedExchange(){
        Map<String,Object> arguments = new HashMap<>(1);
        arguments.put("x-delayed-type","direct");
        /**
         *  交换机名称
         *  交换机类型
         * 是否需要持久化
         * 是否要自动删除
         * 其他参数
         */
        return new CustomExchange(DELAYED_EXCHANGE,"x-delayed-message",true,false,arguments);
    }
```

队列

```
 @Bean
    public Queue delayedQueue(){
        return  new Queue(DELAYED_QUEUE);
    }
```

绑定

```
@Bean
    public Binding delayedQueueBindingDelayedExchange(
            @Qualifier("delayedExchange")CustomExchange customExchange,
            @Qualifier("delayedQueue") Queue queue
            ){
return BindingBuilder.bind(queue).to(customExchange).with(DELAYED_ROUTING_KEY).noargs();
    }
```

### 发送方

```
    // 基于插件
    @GetMapping("sendDelayMsg/{msg}/{delayTime}")
    public void sendMsg(@PathVariable String msg ,@PathVariable Integer delayTime){
        log.info("当前时间:{},发送消息给延迟队列:{}",new Date().toString(),msg);

        rabbitTemplate.convertAndSend("delayed_exchange","delayed.routing_key",msg,message -> {
            // 设置延迟时间时间
            message.getMessageProperties().setDelay(delayTime);
            return message;
        });

    }
```

### 接收方

```
 @RabbitListener(queues = "delayed_queue")
    public void receiveDelayQueue(Message message){
        String msg = new String(message.getBody());
        log.info("当前时间:{} 收到延迟队列的消息{}",new Date().toString(),msg);
    }
```

## 发布确认高级

配置文件开启交换机未收到消息回调

```
  publisher-confirm-type: correlated
```

配置文件开启队列未收到消息回调

```
publisher-returns: true
```

配置类 需要实现二个方法RabbitTemplate.ConfirmCallback,RabbitTemplate.ReturnCallback

实现完成后需要注入 通过 @PostConstruct 

```
 @Autowired
    private RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void init() {
        rabbitTemplate.setConfirmCallback(this);
        rabbitTemplate.setReturnCallback(this);
    }
```

交换机未收到发送发消息回调

```
/**
     * 交换机确认回调方法
     * 1.发消息交换机接收到了回调
     * 1.1 correlationData 保存回调消息的ID及相关信息
     * 1.2交换机收到消息ack = true
     * 1.3 cause null
     * 2.发消息交换机接收失败了回调
     * 2.1 correlationData 保存回调消息的ID及相关信息
     * 2.2交换机收到消息 ack = false
     * 2.3 cause 失败的原因
     */

    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String s) {
        String id = correlationData != null ? correlationData.getId() : "";
        String string = correlationData != null ? Arrays.toString(Objects.requireNonNull(correlationData.getReturnedMessage()).getBody()) : "";
        if (ack) {
            log.info("交换机收id为{}的{}", id, string);
        } else {
            log.info("交换机没有收到id为{}的{},原因是{}", id, s, string);
        }
    }
```

队列未收到返送发消息回调

```
    @Override
    public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
        log.error("消息{}被交换机{}退回原因{},路由key{}", new String(message.getBody()), exchange, replyText, routingKey);
    }
```

消息接收

```
    @RabbitListener(queues = ConfirmConfig.CONFIRM_QUEUE)
    public void receiveConfirmMessage(Message message){
        String msg = new String(message.getBody());
        log.info("当前时间:{} 收到消息{}",new Date().toString(),msg);
    }
```

消息发送

发送要创建 CorrelationData类指定id

```
    @GetMapping("sendMessage/{msg}")
    public void sendMessage(@PathVariable String msg){
        CorrelationData correlationData = new CorrelationData("1");
        correlationData.setReturnedMessage(new Message(msg.getBytes(),null));
        log.info("当前时间:{},发送消息:{}",new Date().toString(),msg);

        rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE,ConfirmConfig.CONFIRM_KEY,msg,correlationData);
        rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE,ConfirmConfig.CONFIRM_KEY+12,msg,correlationData);
    }
```

## 备份交换机

### 声明备份交换机备份队列

```

    @Bean
    public FanoutExchange backupExchange(){
        return new FanoutExchange(BACKUP_EXCHANGE);
    }
	@Bean
    public Queue backupQueue(){
        return QueueBuilder.durable(BACKUP_QUEUE).build();
    }
    @Bean
    public Queue warringQueue(){
        return QueueBuilder.durable(WARRING_QUEUE).build();
    }
```

### 绑定队列

```
@Bean
    public Binding backupBindingBackupExchange(
            @Qualifier("backupExchange") FanoutExchange fanoutExchange,
            @Qualifier("backupQueue") Queue queue
    ){
        return BindingBuilder.bind(queue).to(fanoutExchange);
    }
    @Bean
    public Binding warringBindingExchange(
            @Qualifier("backupExchange") FanoutExchange fanoutExchange,
            @Qualifier("warringQueue") Queue queue
    ){
        return BindingBuilder.bind(queue).to(fanoutExchange);
    }
```

### 修改原始交换机

在原来的交换机上加上alternate-exchange属性 值是备份交换机名字

```
@Bean
    public DirectExchange confirmExchange(){
            return ExchangeBuilder.directExchange(CONFIRM_EXCHANGE).withArgument("alternate-exchange",BACKUP_EXCHANGE).build();
    }
```

## 优先级队列

先设置队列的优先级 优先级0-255数字越大优先级越高推荐0-10

```
@Bean
    public Queue confirmQueue(){
        return QueueBuilder.durable(CONFIRM_QUEUE).maxPriority(10).build();
    }
```

发送消息时指定消息优先级

```
rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE,ConfirmConfig.CONFIRM_KEY+12,msg, messagePostProcessor ->{
            messagePostProcessor.getMessageProperties().setPriority(10);
            return messagePostProcessor;
        },correlationData);
```

## 惰性队列

声明队列时加上lazy()

```
  @Bean
    public Queue confirmQueue(){
        return QueueBuilder.durable(CONFIRM_QUEUE).maxPriority(10).lazy().build();
    }
```

