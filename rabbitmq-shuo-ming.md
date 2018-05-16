# RabbitMQ

RabbitMQ是一个开源的，实现AMQP协议的，可复用企业消息队列系统。
类似的系统还有ActiveMQ(实现JMS)和Kafka(分布式)，RocketMQ。RabbitMQ支持主流的操作系统，支持多种开发语言，能降低系统间访问的耦合度，便于数据同步。

### RabbitMQ架构图

![](https://img-blog.csdn.net/20170531105156976)
几个概念说明：
Broker：简单来说就是消息队列服务器实体。
　　Exchange：消息交换机，它指定消息按什么规则，路由到哪个队列。
　　Queue：消息队列载体，每个消息都会被投入到一个或多个队列。
　　Binding：绑定，它的作用就是把exchange和queue按照路由规则绑定起来。
　　Routing Key：路由关键字，exchange根据这个关键字进行消息投递。
　　vhost：虚拟主机，一个broker里可以开设多个vhost，用作不同用户的权限分离。
　　producer：消息生产者，就是投递消息的程序。
　　consumer：消息消费者，就是接受消息的程序。
　　channel：消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务。

消息队列的使用过程大概如下：
 （1）客户端连接到消息队列服务器，打开一个channel。
　　（2）客户端声明一个exchange，并设置相关属性。
　　（3）客户端声明一个queue，并设置相关属性。
　　（4）客户端使用routing key，在exchange和queue之间建立好绑定关系。
　　（5）客户端投递消息到exchange

###HelloWorld 示例

Maven项目，添加依赖
```maven
 <dependencies>
    <dependency>
        <groupId>com.rabbitmq</groupId>
        <artifactId>amqp-client</artifactId>
        <version>5.2.0</version>
    </dependency>
       <dependency>
           <groupId>org.slf4j</groupId>
           <artifactId>slf4j-log4j12</artifactId>
           <version>1.7.7</version>
       </dependency>
   </dependencies>
```
```java
public class HelloWorld {
    public static final  String QUEUENANME="hello";
    private static final String HOST="localhost";
    private static final String ADMIN="admin";
    private static final String PWD="admin";
    public static void main(String []args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory=new ConnectionFactory();
        connectionFactory.setHost(HOST);
//        connectionFactory.setPassword(ADMIN);
//        connectionFactory.setPassword(PWD);
//        connectionFactory.setPort(15672);
        Connection connection=connectionFactory.newConnection();
        Channel channel=connection.createChannel();
        channel.queueDeclare(QUEUENANME,false,false,false,
        null);
        String message="HelloWorld";
        channel.basicPublish("",QUEUENANME,null,message.getBytes());
        System.out.println("[" + message + "]");

        // 最后，我们关闭channel和连接，释放资源。
        channel.close();
        connection.close();
    }

```




