title: RabbitMQ教程
author: rentie
tags:
  - mq
categories:
  - 队列
date: 2020-09-11 09:31:00
---
MQ全称为Message Queue，消息队列是应用程序和应用程序之间的通信方法；在项目中，可将一些无需即时返回且耗时的操作提取出来，进行异步处理，而这种异步处理的方式大大的节省了服务器的请求响应时间，从而提高了系统的吞吐量。MQ消息中间件的用处：<font color=red>系统解耦、异步调用、流量削峰</font>

<!--more-->

### 消息队列产品介绍
目前市面上成熟主流的MQ有Kafka 、RocketMQ、RabbitMQ，我们这里对每款MQ做一个简单介绍。
#### Kafka
Apache下的一个子项目，使用scala实现的一个高性能分布式Publish/Subscribe消息队列系统。
``` bash
快速持久化：通过磁盘顺序读写与零拷贝机制，可以在O(1)的系统开销下进行消息持久化。
高吞吐：在一台普通的服务器上既可以达到10W/s的吞吐速率。
高堆积：支持topic下消费者较长时间离线，消息堆积量大。
完全的分布式系统：Broker、Producer、Consumer都原生自动支持分布式，依赖zookeeper自动实现复杂均衡。
支持Hadoop数据并行加载：对于像Hadoop的一样的日志数据和离线分析系统，但又要求实时处理的限制，这是一个可行的解决方案。
```
   
#### RocketMQ
RocketMQ的前身是Metaq，当Metaq3.0发布时，产品名称改为RocketMQ。RocketMQ是一款分布式、队列模型的消息中  间件，具有以下特点 ：
``` bash
能够保证严格的消息顺序。
提供丰富的消息拉取模式。
高效的订阅者水平扩展能力。
实时的消息订阅机制。
支持事务消息。
亿级消息堆积能力。
```
#### RabbitMQ
使用Erlang编写的一个开源的消息队列，本身支持很多的协议：AMQP，XMPP, SMTP,STOMP，也正是如此，使的它变的非常重量级，更适合于企业级的开发。同时实现了Broker架构，核心思想是生产者不会将消息直接发送给队列，消息在发送给客户端时先在中心队列排队。对路由(Routing)，负载均衡(Load balance)、数据持久化都有很好的支持。多用于进行企业级的ESB整合。
### RabbitMQ介绍
RabbitMQ是由erlang语言开发，基于AMQP（Advanced Message Queue 高级消息队列协议）协议实现的消息队列，它是一种应用程序之间的通信方法，消息队列在分布式系统开发中应用非常广泛。
RabbitMQ官方地址：http://www.rabbitmq.com/
RabbitMQ提供了6种模式：简单模式，work模式，Publish/Subscribe发布与订阅模式，Routing路由模式，Topics主题模式，RPC远程调用模式（远程调用，不太算MQ；不作介绍）；
官网对应模式介绍：https://www.rabbitmq.com/getstarted.html

![upload successful](/images/pasted-0.png)
### 安装及配置RabbitMQ
#### 安装说明
 自行百度
#### 用户及Virtual Hosts配置
##### 用户角色
RabbitMQ在安装好后，可以访问http://localhost:15672 ；其自带了guest/guest的用户名和密码；如果需要创建自定义用户；那么也可以登录管理界面后，如下操作：

![upload successful](/images/pasted-1.png)
角色说明
``` bash
1、超级管理员(administrator)
可登陆管理控制台，可查看所有的信息，并且可以对用户，策略(policy)进行操作。
2、监控者(monitoring)
可登陆管理控制台，同时可以查看rabbitmq节点的相关信息(进程数，内存使用情况，磁盘使用情况等)
3、策略制定者(policymaker)
可登陆管理控制台, 同时可以对policy进行管理。但无法查看节点的相关信息(上图红框标识的部分)。
4、普通管理者(management)
仅可登陆管理控制台，无法看到节点信息，也无法对策略进行管理。
5、其他
无法登陆管理控制台，通常就是普通的生产者和消费者。
```
##### Virtual Hosts配置
像mysql拥有数据库的概念并且可以指定用户对库和表等操作的权限。RabbitMQ也有类似的权限管理；在RabbitMQ中可以虚拟消息服务器Virtual Host，每个Virtual Hosts相当于一个相对独立的RabbitMQ服务器，每个VirtualHost之间是相互隔离的。exchange、queue、message不能互通。 相当于mysql的db。Virtual Name一般以/开头。
###### 创建Virtual Hosts

![upload successful](/images/pasted-2.png)
###### 设置Virtual Hosts权限

![upload successful](/images/pasted-3.png)

![upload successful](/images/pasted-4.png)
参数说明
``` bash
user：用户名
configure ：一个正则表达式，用户对符合该正则表达式的所有资源拥有 configure 操作的权限
write：一个正则表达式，用户对符合该正则表达式的所有资源拥有 write 操作的权限
read：一个正则表达式，用户对符合该正则表达式的所有资源拥有 read 操作的权限
```
### RabbitMQ进阶学习
#### Work queues工作队列模式
##### 模式说明

![upload successful](/images/pasted-6.png)
Work Queues与入门程序的简单模式相比，多了一个或一些消费端，多个消费端共同消费同一个队列中的消息。
**应用场景：**对于任务过重或任务较多情况，使用工作队列模式使用多个消费者可以提高任务处理的速度。
##### 代码实现
Work Queues与入门程序的简单模式的代码是几乎一样的；可以完全复制，并复制多一个消费者进行多个消费者同时消费消息的测试
###### 生产者
``` bash
/**
 * RabbitMQ入门案例
 * 生产者开发-完成工作队列消息发送
 * @author Steven
 * @description com.itheima.mq.work
 */
public class WorkProducer {
    public static void main(String[] args) throws Exception{
        Connection connection = ConnectionUtils.getConnection();
        //8、创建频道-channel = connection.createChannel()
        Channel channel = connection.createChannel();
        //连续发10条消息
        for (int i = 0; i < 10; i++) {
            //9、声明队列-channel.queueDeclare(名称，是否持久化，是否独占本连接,是否自动删除,附加参数)
            channel.queueDeclare("work_queue",true,false,false,null);
            //10、创建消息-String m = xxx
            String message = "hello,欢迎来到深圳黑马！" + i;
            //11、消息发送-channel.basicPublish(交换机[默认Default Exchage],路由key[简单模式可以传递队列名称],消息其它属性,消息内容)
            channel.basicPublish("","work_queue",null,message.getBytes("utf-8"));
        }
        //12、关闭资源-channel.close();connection.close()
        channel.close();
        connection.close();
    }
}
```
###### 消费者ONE

``` bash
/**
 * RabbitMQ入门案例
 * 消费者开发-完成工作队列消息接收
 * @author Steven
 * @description com.itheima.mq.work
 */
public class WorkConsumerOne {
    public static void main(String[] args) throws Exception{
        Connection connection = ConnectionUtils.getConnection();
        //8、创建频道-channel = connection.createChannel()
        Channel channel = connection.createChannel();
        //9、声明队列-channel.queueDeclare(名称，是否持久化，是否独占本连接,是否自动删除,附加参数)
        channel.queueDeclare("work_queue",true,false,false,null);

        //创建消费者
        Consumer callback = new DefaultConsumer(channel){
            /**
             * @param consumerTag 消费者标签，在channel.basicConsume时候可以指定
             * @param envelope 消息包的内容，可从中获取消息id，消息routingkey，交换机，消息和重传标志(收到消息失败后是否需要重新发送)
             * @param properties  属性信息(生产者的发送时指定)
             * @param body 消息内容
             * @throws IOException
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                //路由的key
                String routingKey = envelope.getRoutingKey();
                //获取交换机信息
                String exchange = envelope.getExchange();
                //获取消息ID
                long deliveryTag = envelope.getDeliveryTag();
                //获取消息信息
                String message = new String(body,"utf-8");
                System.out.println(
                        "routingKey:" + routingKey +
                        ",exchange:" + exchange +
                        ",deliveryTag:" + deliveryTag +
                        ",message:" + message);
            }
        };
        /**
         * 消息消费
         * 参数1：队列名称
         * 参数2：是否自动应答，true为自动应答[mq接收到回复会删除消息]，设置为false则需要手动应答
         * 参数3：消息接收到后回调
         */
        channel.basicConsume("work_queue",true,callback);

        //注意，此处不建议关闭资源，让程序一直处于读取消息
    }
}
```
###### 消费者TWO
``` bash
/**
 * RabbitMQ入门案例
 * 消费者开发-完成工作队列消息接收
 * @author Steven
 * @description com.itheima.mq.work
 */
public class WorkConsumerTwo {
    public static void main(String[] args) throws Exception{
        Connection connection = ConnectionUtils.getConnection();
        //8、创建频道-channel = connection.createChannel()
        Channel channel = connection.createChannel();
        //9、声明队列-channel.queueDeclare(名称，是否持久化，是否独占本连接,是否自动删除,附加参数)
        channel.queueDeclare("work_queue",true,false,false,null);

        //创建消费者
        Consumer callback = new DefaultConsumer(channel){
            /**
             * @param consumerTag 消费者标签，在channel.basicConsume时候可以指定
             * @param envelope 消息包的内容，可从中获取消息id，消息routingkey，交换机，消息和重传标志(收到消息失败后是否需要重新发送)
             * @param properties  属性信息(生产者的发送时指定)
             * @param body 消息内容
             * @throws IOException
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                //路由的key
                String routingKey = envelope.getRoutingKey();
                //获取交换机信息
                String exchange = envelope.getExchange();
                //获取消息ID
                long deliveryTag = envelope.getDeliveryTag();
                //获取消息信息
                String message = new String(body,"utf-8");
                System.out.println(
                        "routingKey:" + routingKey +
                        ",exchange:" + exchange +
                        ",deliveryTag:" + deliveryTag +
                        ",message:" + message);
            }
        };
        /**
         * 消息消费
         * 参数1：队列名称
         * 参数2：是否自动应答，true为自动应答[mq接收到回复会删除消息]，设置为false则需要手动应答
         * 参数3：消息接收到后回调
         */
        channel.basicConsume("work_queue",true,callback);

        //注意，此处不建议关闭资源，让程序一直处于读取消息
    }
}
```
##### 测试与查看结果
启动两个消费者，然后再启动生产者发送消息；到IDEA的两个消费者对应的控制台查看是否竞争性的接收到消息。

![upload successful](/images/pasted-7.png)

![upload successful](/images/pasted-8.png)
##### 小结
在一个队列中如果有多个消费者，那么消费者之间对于同一个消息的关系是**竞争**的关系。
#### Publish/Subscribe发布订阅模式
##### 模式说明

![upload successful](/images/pasted-9.png)
在发布订阅模型中，多了一个x(exchange)角色，而且过程略有变化。
``` bash
P：生产者，也就是要发送消息的程序，但是不再发送到队列中，而是发给X（交换机）
C：消费者，消息的接受者，会一直等待消息到来。
Queue：消息队列，接收消息、缓存消息。
Exchange：交换机，图中的X。一方面，接收生产者发送的消息。另一方面，知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。Exchange有常见以下3种类型：
	Fanout：广播，将消息交给所有绑定到交换机的队列
	Direct：定向，把消息交给符合指定routing key 的队列
	Topic：通配符，把消息交给符合routing pattern（路由模式） 的队列
```
**Exchange（交换机）只负责转发消息，不具备存储消息的能力**，因此如果没有任何队列与Exchange**绑定**，或者没有符合路由规则的队列，那么消息会丢失！

![upload successful](/images/pasted-10.png)
##### 代码实现
**注意:**<br>
1.声明交换机- channel.exchangeDeclare(交换机名字,交换机类型)<br>
2.发送消息时，要发给交换机，而不是发给队列
###### 生产者
``` bash
/**
 * RabbitMQ入门案例
 * 生产者开发-完成广播消息发送
 * @author Steven
 * @description com.itheima.mq.fanout
 */
public class FanoutProducer {
    public static void main(String[] args) throws Exception{
        Connection connection = ConnectionUtils.getConnection();
        //8、创建频道-channel = connection.createChannel()
        Channel channel = connection.createChannel();

        //声明交换机- channel.exchangeDeclare(交换机名字,交换机类型)
        channel.exchangeDeclare("fanout_exchange", BuiltinExchangeType.FANOUT);
        //连续发10条消息
        for (int i = 0; i < 10; i++) {
            //10、创建消息-String m = xxx
            String message = "hello,欢迎来到深圳黑马！" + i;
            //11、消息发送-channel.basicPublish(交换机[默认Default Exchage],路由key[简单模式可以传递队列名称],消息其它属性,消息内容)
            channel.basicPublish("fanout_exchange","",null,message.getBytes("utf-8"));
        }
        //12、关闭资源-channel.close();connection.close()
        channel.close();
        connection.close();
    }
}
```
执行完生产者后,发现交换机多了一行,但是不见消息内容,这是为什么呢?
消息丢失了!!!因为交换机没有存储消息的能力,在 rabbitmq 中只有队列存储消息的能力.因为这时还没有队列,所以就会丢失;<br>
**结论:** 消息发送到了一个没有绑定队列的交换机时,消息就会丢失!

![upload successful](/images/pasted-11.png)

###### 消费者ONE
绑定交换机的代码在消费者中，和之前的工作队列模式代码一样，只不过多了一行绑定交换机.
队列需要绑定指定的交换机- channel.queueBind(队列名, 交换机名, 路由key,广播消息设置为空串)
``` bash
/**
 * RabbitMQ入门案例
 * 消费者开发-完成广播消息接收
 * @author Steven
 * @description com.itheima.mq.simple
 */
public class FanoutConsumerOne {
    public static void main(String[] args) throws Exception{
        Connection connection = ConnectionUtils.getConnection();
        //8、创建频道-channel = connection.createChannel()
        Channel channel = connection.createChannel();
        //9、声明队列-channel.queueDeclare(名称，是否持久化，是否独占本连接,是否自动删除,附加参数)
        channel.queueDeclare("fanout_queue1",true,false,false,null);
        //队列绑定交换机-channel.queueBind(队列名, 交换机名, 路由key[广播消息设置为空串])
        channel.queueBind("fanout_queue1", "fanout_exchange", "");

        //创建消费者
        Consumer callback = new DefaultConsumer(channel){
            /**
             * @param consumerTag 消费者标签，在channel.basicConsume时候可以指定
             * @param envelope 消息包的内容，可从中获取消息id，消息routingkey，交换机，消息和重传标志(收到消息失败后是否需要重新发送)
             * @param properties  属性信息(生产者的发送时指定)
             * @param body 消息内容
             * @throws IOException
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                //路由的key
                String routingKey = envelope.getRoutingKey();
                //获取交换机信息
                String exchange = envelope.getExchange();
                //获取消息ID
                long deliveryTag = envelope.getDeliveryTag();
                //获取消息信息
                String message = new String(body,"utf-8");
                System.out.println(
                        "routingKey:" + routingKey +
                        ",exchange:" + exchange +
                        ",deliveryTag:" + deliveryTag +
                        ",message:" + message);
            }
        };
        /**
         * 消息消费
         * 参数1：队列名称
         * 参数2：是否自动应答，true为自动应答[mq接收到回复会删除消息]，设置为false则需要手动应答
         * 参数3：消息接收到后回调
         */
        channel.basicConsume("fanout_queue1",true,callback);

        //注意，此处不建议关闭资源，让程序一直处于读取消息
    }
}
```
###### 消费者TWO
``` bash
/**
 * RabbitMQ入门案例
 * 消费者开发-完成广播消息接收
 * @author Steven
 * @description com.itheima.mq.simple
 */
public class FanoutConsumerTwo {
    public static void main(String[] args) throws Exception{
        Connection connection = ConnectionUtils.getConnection();
        //8、创建频道-channel = connection.createChannel()
        Channel channel = connection.createChannel();
        //9、声明队列-channel.queueDeclare(名称，是否持久化，是否独占本连接,是否自动删除,附加参数)
        channel.queueDeclare("fanout_queue2",true,false,false,null);
        //队列绑定交换机-channel.queueBind(队列名, 交换机名, 路由key[广播消息设置为空串])
        channel.queueBind("fanout_queue2", "fanout_exchange", "");
        //创建消费者
        Consumer callback = new DefaultConsumer(channel){
            /**
             * @param consumerTag 消费者标签，在channel.basicConsume时候可以指定
             * @param envelope 消息包的内容，可从中获取消息id，消息routingkey，交换机，消息和重传标志(收到消息失败后是否需要重新发送)
             * @param properties  属性信息(生产者的发送时指定)
             * @param body 消息内容
             * @throws IOException
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                //路由的key
                String routingKey = envelope.getRoutingKey();
                //获取交换机信息
                String exchange = envelope.getExchange();
                //获取消息ID
                long deliveryTag = envelope.getDeliveryTag();
                //获取消息信息
                String message = new String(body,"utf-8");
                System.out.println(
                        "routingKey:" + routingKey +
                        ",exchange:" + exchange +
                        ",deliveryTag:" + deliveryTag +
                        ",message:" + message);
            }
        };
        /**
         * 消息消费
         * 参数1：队列名称
         * 参数2：是否自动应答，true为自动应答[mq接收到回复会删除消息]，设置为false则需要手动应答
         * 参数3：消息接收到后回调
         */
        channel.basicConsume("fanout_queue2",true,callback);

        //注意，此处不建议关闭资源，让程序一直处于读取消息
    }
}
```
##### 测试与查看结果
**注意：**
<font color=#7FFFD4>绑定交换机的前提是得先有这个交换机,所以得先执行一次生产者,如果没有这个交换机就执行消费者绑定交换机的话会报错.执行完两个消费者再执行生产者后,就会看到两个消费者都消费这一条消息了。</font> <br>
启动所有消费者，然后使用生产者发送消息；在每个消费者对应的控制台可以查看到生产者发送的所有消息；到达**广播**的效果。

![upload successful](/images/pasted-12.png)

![upload successful](/images/pasted-13.png)
在执行完测试代码后，到RabbitMQ的管理后台找到Exchanges选项卡，点击 fanout_exchange 的交换机，可以查看到如下的绑定：

![upload successful](/images/pasted-14.png)
##### 小结
交换机需要与队列进行绑定，绑定之后；一个消息可以被多个消费者都收到。<br>
发布订阅模式与work队列模式的区别:
``` bash
1、work队列模式不用定义交换机，而发布/订阅模式需要定义交换机。 
2、发布/订阅模式的生产方是面向交换机发送消息，work队列模式的生产方是面向队列发送消息(底层使用默认交换机)。
3、发布/订阅模式的消费者需要设置队列和交换机的绑定，work队列模式不需要设置，实际上work队列模式会将队列绑 定到默认的交换机。
```

#### Routing路由模式
##### 模式说明
路由模式特点：
``` bash
1.队列与交换机的绑定，不能是任意绑定了，而是要指定一个RoutingKey（路由key）
2.消息的发送方在 向 Exchange发送消息时，也必须指定消息的 RoutingKey。
3.Exchange不再把消息交给每一个绑定的队列，而是根据消息的Routing Key进行判断，只有队列的Routingkey与消息的 Routing key完全一致，才会接收到消息 
```
![upload successful](/images/pasted-15.png)
``` bash
P：生产者，向Exchange发送消息，发送消息时，会指定一个routing key。
X：Exchange（交换机），接收生产者的消息，然后把消息递交给 与routing key完全匹配的队列
C1：消费者，其所在队列指定了需要routing key 为 error 的消息
C2：消费者，其所在队列指定了需要routing key 为 info、error、warning 的消息
```
##### 代码实现
###### 生产者
**注意：**
``` bash
1.交换机的类型为：Direct
2.发送消息的时候根据相关逻辑指定相应的routing key。
```
``` bash
/**
 * RabbitMQ入门案例
 * 生产者开发-完成路由消息发送
 * @author Steven
 * @description com.itheima.mq.routing
 */
public class RoutingProducer {
    public static void main(String[] args) throws Exception{
        Connection connection = ConnectionUtils.getConnection();
        //8、创建频道-channel = connection.createChannel()
        Channel channel = connection.createChannel();

        //声明交换机- channel.exchangeDeclare(交换机名字,交换机类型)
        channel.exchangeDeclare("routing_exchange", BuiltinExchangeType.DIRECT);
        //连续发3条消息
        for (int i = 0; i < 3; i++) {
            String routingKey = "";
            //发送消息的时候根据相关逻辑指定相应的routing key。
            switch (i){
                case 0:  //假设i=0，为error消息
                    routingKey = "log.error";
                    break;
                case 1: //假设i=1，为info消息
                    routingKey = "log.info";
                    break;
                case 2: //假设i=2，为warning消息
                    routingKey = "log.warning";
                    break;
            }
            //10、创建消息-String m = xxx
            String message = "hello,欢迎来到深圳黑马！" + i;
            //11、消息发送-channel.basicPublish(交换机[默认Default Exchage],路由key[简单模式可以传递队列名称],消息其它属性,消息内容)
            channel.basicPublish("routing_exchange",routingKey,null,message.getBytes("utf-8"));
        }
        //12、关闭资源-channel.close();connection.close()
        channel.close();
        connection.close();
    }
}
```
###### 消费者ONE

``` bash
/**
 * RabbitMQ入门案例
 * 消费者开发-完成路由消息接收
 * @author Steven
 * @description com.itheima.mq.routing
 */
public class RoutingConsumerOne {
    public static void main(String[] args) throws Exception{
        Connection connection = ConnectionUtils.getConnection();
        //8、创建频道-channel = connection.createChannel()
        Channel channel = connection.createChannel();
        //9、声明队列-channel.queueDeclare(名称，是否持久化，是否独占本连接,是否自动删除,附加参数)
        channel.queueDeclare("routing_queue1",true,false,false,null);

        //队列绑定交换机-channel.queueBind(队列名, 交换机名, 路由key[广播消息设置为空串])
        channel.queueBind("routing_queue1", "routing_exchange", "log.error");
        //创建消费者
        Consumer callback = new DefaultConsumer(channel){
            /**
             * @param consumerTag 消费者标签，在channel.basicConsume时候可以指定
             * @param envelope 消息包的内容，可从中获取消息id，消息routingkey，交换机，消息和重传标志(收到消息失败后是否需要重新发送)
             * @param properties  属性信息(生产者的发送时指定)
             * @param body 消息内容
             * @throws IOException
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                //路由的key
                String routingKey = envelope.getRoutingKey();
                //获取交换机信息
                String exchange = envelope.getExchange();
                //获取消息ID
                long deliveryTag = envelope.getDeliveryTag();
                //获取消息信息
                String message = new String(body,"utf-8");
                System.out.println(
                        "routingKey:" + routingKey +
                        ",exchange:" + exchange +
                        ",deliveryTag:" + deliveryTag +
                        ",message:" + message);
            }
        };
        /**
         * 消息消费
         * 参数1：队列名称
         * 参数2：是否自动应答，true为自动应答[mq接收到回复会删除消息]，设置为false则需要手动应答
         * 参数3：消息接收到后回调
         */
        channel.basicConsume("routing_queue1",true,callback);

        //注意，此处不建议关闭资源，让程序一直处于读取消息
    }
}
```
###### 消费者TWO
``` bash
/**
 * RabbitMQ入门案例
 * 消费者开发-完成路由消息接收
 * @author Steven
 * @description com.itheima.mq.routing
 */
public class RoutingConsumerTwo {
    public static void main(String[] args) throws Exception{
        Connection connection = ConnectionUtils.getConnection();
        //8、创建频道-channel = connection.createChannel()
        Channel channel = connection.createChannel();
        //9、声明队列-channel.queueDeclare(名称，是否持久化，是否独占本连接,是否自动删除,附加参数)
        channel.queueDeclare("routing_queue2",true,false,false,null);

        //队列绑定交换机-channel.queueBind(队列名, 交换机名, 路由key[广播消息设置为空串])
        channel.queueBind("routing_queue2", "routing_exchange", "log.error");
        //队列绑定交换机-channel.queueBind(队列名, 交换机名, 路由key[广播消息设置为空串])
        channel.queueBind("routing_queue2", "routing_exchange", "log.info");
        //队列绑定交换机-channel.queueBind(队列名, 交换机名, 路由key[广播消息设置为空串])
        channel.queueBind("routing_queue2", "routing_exchange", "log.warning");
        //创建消费者
        Consumer callback = new DefaultConsumer(channel){
            /**
             * @param consumerTag 消费者标签，在channel.basicConsume时候可以指定
             * @param envelope 消息包的内容，可从中获取消息id，消息routingkey，交换机，消息和重传标志(收到消息失败后是否需要重新发送)
             * @param properties  属性信息(生产者的发送时指定)
             * @param body 消息内容
             * @throws IOException
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                //路由的key
                String routingKey = envelope.getRoutingKey();
                //获取交换机信息
                String exchange = envelope.getExchange();
                //获取消息ID
                long deliveryTag = envelope.getDeliveryTag();
                //获取消息信息
                String message = new String(body,"utf-8");
                System.out.println(
                        "routingKey:" + routingKey +
                        ",exchange:" + exchange +
                        ",deliveryTag:" + deliveryTag +
                        ",message:" + message);
            }
        };
        /**
         * 消息消费
         * 参数1：队列名称
         * 参数2：是否自动应答，true为自动应答[mq接收到回复会删除消息]，设置为false则需要手动应答
         * 参数3：消息接收到后回调
         */
        channel.basicConsume("routing_queue2",true,callback);

        //注意，此处不建议关闭资源，让程序一直处于读取消息
    }
}
```
##### 测试与查看结果
启动所有消费者，然后使用生产者发送消息；在消费者对应的控制台可以查看到生产者发送对应routing key对应队列的消息；到达**按照需要接收**的效果。

![upload successful](/images/pasted-16.png)

![upload successful](/images/pasted-17.png)

##### 小结
Routing模式要求队列在绑定交换机时要指定routing key，消息会转发到符合routing key的队列
#### Topics通配符模式
##### 模式说明

![upload successful](/images/pasted-18.png)
Topic类型与Direct相比，都是可以根据RoutingKey把消息路由到不同的队列。只不过Topic类型Exchange可以让队列在绑定Routing key的时候使用通配符！
Routingkey 一般都是有一个或多个单词组成，多个单词之间以“ . ”分割，例如： item.insert<br>
**通配符规则：**
``` bash
#：匹配一个或多个词
*：匹配不多不少恰好1个词 
```
举例：<br>
item.#：能够匹配item.insert.abc 或者 item.insert<br>
item.*：只能匹配item.insert

![upload successful](/images/pasted-19.png)
图解：<br>
红色Queue：绑定的是usa.# ，因此凡是以 usa.开头的routing key 都会被匹配到<br>
黄色Queue：绑定的是#.news ，因此凡是以 .news结尾的 routing key 都会被匹配

##### 代码实现
###### 生产者
**注意**
``` bash
1.交换机的类型为：Topic
2.发送消息的时候根据相关逻辑指定相应的routing key。
```
``` bash
/**
 * RabbitMQ入门案例
 * 生产者开发-完成Topics通配符消息发送
 * @author Steven
 * @description com.itheima.mq.topic
 */
public class TopicProducer {
    public static void main(String[] args) throws Exception{
        Connection connection = ConnectionUtils.getConnection();
        //8、创建频道-channel = connection.createChannel()
        Channel channel = connection.createChannel();

        //声明交换机- channel.exchangeDeclare(交换机名字,交换机类型)
        channel.exchangeDeclare("topic_exchange", BuiltinExchangeType.TOPIC);
        //连续发3条消息
        for (int i = 0; i < 5; i++) {
            String routingKey = "";
            //发送消息的时候根据相关逻辑指定相应的routing key。
            switch (i){
                case 0:  //假设i=0，为error消息
                    routingKey = "log.error";
                    break;
                case 1: //假设i=1，为info消息
                    routingKey = "log.info";
                    break;
                case 2: //假设i=2，为warning消息
                    routingKey = "log.warning";
                    break;
                case 3: //假设i=3，为log.info.add消息
                    routingKey = "log.info.add";
                    break;
                case 4: //假设i=4，为log.info.update消息
                    routingKey = "log.info.update";
                    break;
            }
            //10、创建消息-String m = xxx
            String message = "hello,欢迎来到深圳黑马！" + i;
            //11、消息发送-channel.basicPublish(交换机[默认Default Exchage],路由key[简单模式可以传递队列名称],消息其它属性,消息内容)
            channel.basicPublish("topic_exchange",routingKey,null,message.getBytes("utf-8"));
        }
        //12、关闭资源-channel.close();connection.close()
        channel.close();
        connection.close();
    }
}
```
###### 消费者ONE

``` bash
/**
 * RabbitMQ入门案例
 * 消费者开发-完成Topics通配符消息接收
 * @author Steven
 * @description com.itheima.mq.topic
 */
public class TopicConsumerOne {
    public static void main(String[] args) throws Exception{
        Connection connection = ConnectionUtils.getConnection();
        //8、创建频道-channel = connection.createChannel()
        Channel channel = connection.createChannel();
        //9、声明队列-channel.queueDeclare(名称，是否持久化，是否独占本连接,是否自动删除,附加参数)
        channel.queueDeclare("topic_queue1",true,false,false,null);
        //队列绑定交换机与路由key
        channel.queueBind("topic_queue1", "topic_exchange", "log.*");
        //创建消费者
        Consumer callback = new DefaultConsumer(channel){
            /**
             * @param consumerTag 消费者标签，在channel.basicConsume时候可以指定
             * @param envelope 消息包的内容，可从中获取消息id，消息routingkey，交换机，消息和重传标志(收到消息失败后是否需要重新发送)
             * @param properties  属性信息(生产者的发送时指定)
             * @param body 消息内容
             * @throws IOException
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                //路由的key
                String routingKey = envelope.getRoutingKey();
                //获取交换机信息
                String exchange = envelope.getExchange();
                //获取消息ID
                long deliveryTag = envelope.getDeliveryTag();
                //获取消息信息
                String message = new String(body,"utf-8");
                System.out.println(
                        "routingKey:" + routingKey +
                        ",exchange:" + exchange +
                        ",deliveryTag:" + deliveryTag +
                        ",message:" + message);
            }
        };
        /**
         * 消息消费
         * 参数1：队列名称
         * 参数2：是否自动应答，true为自动应答[mq接收到回复会删除消息]，设置为false则需要手动应答
         * 参数3：消息接收到后回调
         */
        channel.basicConsume("topic_queue1",true,callback);

        //注意，此处不建议关闭资源，让程序一直处于读取消息
    }
}
```
###### 消费者TWO
``` bash
/**
 * RabbitMQ入门案例
 * 消费者开发-完成Topics通配符消息接收
 * @author Steven
 * @description com.itheima.mq.topic
 */
public class TopicConsumerTwo {
    public static void main(String[] args) throws Exception{
        Connection connection = ConnectionUtils.getConnection();
        //8、创建频道-channel = connection.createChannel()
        Channel channel = connection.createChannel();
        //9、声明队列-channel.queueDeclare(名称，是否持久化，是否独占本连接,是否自动删除,附加参数)
        channel.queueDeclare("topic_queue2",true,false,false,null);

        //队列绑定路由key
        channel.queueBind("topic_queue2", "topic_exchange", "log.#");
        //创建消费者
        Consumer callback = new DefaultConsumer(channel){
            /**
             * @param consumerTag 消费者标签，在channel.basicConsume时候可以指定
             * @param envelope 消息包的内容，可从中获取消息id，消息routingkey，交换机，消息和重传标志(收到消息失败后是否需要重新发送)
             * @param properties  属性信息(生产者的发送时指定)
             * @param body 消息内容
             * @throws IOException
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                //路由的key
                String routingKey = envelope.getRoutingKey();
                //获取交换机信息
                String exchange = envelope.getExchange();
                //获取消息ID
                long deliveryTag = envelope.getDeliveryTag();
                //获取消息信息
                String message = new String(body,"utf-8");
                System.out.println(
                        "routingKey:" + routingKey +
                        ",exchange:" + exchange +
                        ",deliveryTag:" + deliveryTag +
                        ",message:" + message);
            }
        };
        /**
         * 消息消费
         * 参数1：队列名称
         * 参数2：是否自动应答，true为自动应答[mq接收到回复会删除消息]，设置为false则需要手动应答
         * 参数3：消息接收到后回调
         */
        channel.basicConsume("topic_queue2",true,callback);

        //注意，此处不建议关闭资源，让程序一直处于读取消息
    }
}
```
##### 测试与查看结果
启动所有消费者，然后使用生产者发送消息；在消费者对应的控制台可以查看到生产者发送对应routing key对应队列的消息；到达**按照需要接收**的效果。

![upload successful](/images/pasted-20.png)

![upload successful](/images/pasted-21.png)

##### 小结
Topic主题模式可以实现 Publish/Subscribe发布订阅模式 和 Routing路由模式 的双重功能；只是Topic在配置routing key 的时候可以使用通配符，显得更加灵活。

#### 模式总结
RabbitMQ工作模式：
``` bash
1、简单模式 HelloWorld
一个生产者、一个消费者，不需要设置交换机（使用默认的交换机）
2、工作队列模式 Work Queue
一个生产者、多个消费者（竞争关系），不需要设置交换机（使用默认的交换机）
3、发布订阅模式 Publish/subscribe
需要设置类型为fanout的交换机，并且交换机和队列进行绑定，当发送消息到交换机后，交换机会将消息发送到绑定的队列
4、路由模式 Routing
需要设置类型为direct的交换机，交换机和队列进行绑定，并且指定routing key，当发送消息到交换机后，交换机会根据routing key将消息发送到对应的队列
5、通配符模式 Topic
需要设置类型为topic的交换机，交换机和队列进行绑定，并且指定通配符方式的routing key，当发送消息到交换机后，交换机会根据routing key将消息发送到对应的队列
```
### Spring Boot整合RabbitMQ
#### 简介
在spring boot项目中只需要引入对应的amqp启动器依赖即可，方便的使用RabbitTemplate发送消息，使用注解接收消息<br>
**生产者工程：**
``` bash
1.application.properties文件配置RabbitMQ相关信息；
2.在生产者工程中编写配置类，用于创建交换机、队列与配置队列绑定交换机
3.注入RabbitTemplate对象，通过RabbitTemplate对象发送消息到交换机
```
**消费者工程：**
``` bash
1.application. properties文件配置RabbitMQ相关信息
2.创建消息处理监听类，用于接收队列中的消息并进行处理
```
#### 搭建生产者工程
##### 创建工程

使用Idea的Spring Initializr创建生产者工程springboot_rabbitmq_producer ，如下

![upload successful](/images/pasted-22.png)
pom.xml
``` bash
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.7.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.itheima</groupId>
    <artifactId>springboot_rabbitmq_producer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot_rabbitmq_producer</name>
    <description>Spring Boot整合RabbitMQ</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```
#### 配置RabbitMQ
##### 配置连接信息
在application.properties中,去编辑和RabbitMQ相关的配置信息
``` bash
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.virtual-host=/szitheima
spring.rabbitmq.username=admin
spring.rabbitmq.password=admin
```
##### 配置交换机与队列
创建RabbitMQ队列与交换机绑定的配置类com.itheima.config.SendConfig,代码如下
``` bash
/**
 * 配置RabbitMQ生产者信息
 * @author Steven
 * @description com.itheima.config
 */
@Configuration
public class RabbitMQConfig {
    //创建交换机
    @Bean(name = "topicExchange")
    public TopicExchange topicExchange(){
        return new TopicExchange("topic_exchange_springboot");
    }

    //创建队列
    @Bean(name = "topicQueueSpringBoot")
    public Queue topicQueue(){
        return QueueBuilder.durable("topic_queue_springboot").build();
    }

    //队列绑定交换机
    @Bean
    public Binding bindingExchangeTopicQueue(@Qualifier("topicQueueSpringBoot") Queue queue,
                                             @Qualifier("topicExchange")Exchange exchange){
        return BindingBuilder.bind(queue).to(exchange).with("log.#").noargs();
    }
} 
```
##### 测试生产者
``` bash
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootRabbitmqProducerApplicationTests {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Test
    public void testSendMessage() {
        //convertAndSend(交换机名称，路由key,消息内容)
        rabbitTemplate.convertAndSend("topic_exchange_springboot","log.info","发送了info消息");
        rabbitTemplate.convertAndSend("topic_exchange_springboot","log.error","发送了error消息");
        rabbitTemplate.convertAndSend("topic_exchange_springboot","log.warning","发送了warning消息");
    }
}
```
#### 搭建消费者工程
##### 创建工程
使用Idea的Spring Initializr创建生产者工程springboot_rabbitmq_consumer ，如下

![upload successful](/images/pasted-23.png)
pom.xml
``` bash
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.7.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.itheima</groupId>
    <artifactId>springboot_rabbitmq_consumer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot_rabbitmq_consumer</name>
    <description>Spring Boot整合RabbitMQ</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```
#### 配置RabbitMQ
##### 配置连接信息
在application.properties中,去编辑和RabbitMQ相关的配置信息
``` bash
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.virtual-host=/szitheima
spring.rabbitmq.username=admin
spring.rabbitmq.password=admin
```
##### 创建消息处理监听类
编写消息监听器com.itheima.listener.MessageListener，代码如下：
``` bash
/**
 * 消息监听器
 * @author Steven
 * @description com.itheima.listener
 */
@Component
public class MessageListener {

    /**
     * 监听某个队列的消息
     * @param msg 接收到的消息
     */
    @RabbitListener(queues = "topic_queue_springboot")
    public void topicListener(String msg){
        System.out.println("接收到消息：" + msg);
    }
}
```
##### 测试消费者
先运行生产者测试程序(交换机和队列才能先被声明和绑定)，然后启动消费者(启动引导类)；在消费者工程springboot-rabbitmq-consumer中控制台查看是否接收到对应消息。

![upload successful](/images/pasted-24.png)