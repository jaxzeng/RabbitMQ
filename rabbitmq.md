#### RabbitMQ 五种队列模型
 1.Simple
 ![](https://images2015.cnblogs.com/blog/1139438/201704/1139438-20170404145647160-674299616.jpg)
 
 2.Work. 工作模式，一个消息只能被一个消费者获取
 ![](https://images2015.cnblogs.com/blog/1139438/201704/1139438-20170404145730378-520808425.jpg)
 
 3.Publish/Subscribe. 订阅模式，消息被路由投递给多个队列，一个消息被多个消费者获取。ExchangeType为fanout。
 
 ![](https://images2015.cnblogs.com/blog/1139438/201704/1139438-20170404145953410-2120188475.jpg)
 
 4.Routing. 路由模式，一个消息被多个消费者获取。并且消息的目的queue可被生产者指定。ExchangeType为direct。
 ![](https://images2015.cnblogs.com/blog/1139438/201704/1139438-20170404150031472-865479935.jpg)
 
 5.Topic. 通配符模式，一个消息被多个消费者获取。消息的目的queue可用BindingKey以通配符(#：一个或多个词，*：一个词)的方式指定。ExchangeType为topic。
 
 ![](https://images2015.cnblogs.com/blog/1139438/201704/1139438-20170404150059332-855520242.jpg)
 
 6.~~PRC. 远程调用~~
 ![](https://images2015.cnblogs.com/blog/1139438/201704/1139438-20170404150159082-1237472789.jpg)
 
 ##### demo
 2.work 模式
 ```java
  发送者
    public class Send {

    private static  final  String QUEUENAME="queuedemo";
    public static void main(String args[]) throws IOException, TimeoutException, InterruptedException {
        Connection connection=new ConnectionUtil().getConnection();

        Channel channel=connection.createChannel();

        //定义队列
        channel.queueDeclare(QUEUENAME,false,false,false,null);

        for(int i=0;i<10;i++){
            String message=i+"messageß";
            channel.basicPublish("",QUEUENAME,null,message.getBytes());
            System.out.println(message);
         Thread.sleep(50);
        }
        channel.close();
        connection.close();
    }
    
    接收者
    
    public class Recive {

    private static  final  String QUEUENAME="queuedemo";
    public static  void  main(String args[]) throws IOException, TimeoutException, InterruptedException {
        Connection connection=new ConnectionUtil().getConnection();

        Channel channel=connection.createChannel();
        channel.queueDeclare(QUEUENAME,false,false,false,null);
        channel.basicQos(1);

        QueueingConsumer consumer = new QueueingConsumer(channel);
        channel.basicConsume(QUEUENAME, false, consumer);
        while (true) {
            QueueingConsumer.Delivery delivery = consumer.nextDelivery();
            String message = new String(delivery.getBody());
            System.out.println(" [x] Received '" + message + "'");
            // 模拟handling
            Thread.sleep(200);
            // ACK
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        }
    }
 ```java
 Publish/Subscribe 模式只需将队列与 exchange 绑定就ok
 public static final String QUEUE_NAME = "queuedemo";
 public static final String QUEUE_NAME1 = "queuedemo1";

 public final static String EXCHANGEDECLARE = "fanoutdemo";
 channel.queueBind(QUEUE_NAME, EXCHANGEDECLARE, "");
 channel.queueBind(QUEUE_NAME1, EXCHANGEDECLARE, "");

routing 模式
发送方
channel.basicPublish(EXCHANGEDECLARE,"insert",null,message.getBytes());

队列
channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "insert");
可获取消息 而
channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "update");
绑定 无法获取

 topic 模式 需要定义
 关于绑定键有两种特殊的情况：*（星号）可以代替一个任意标识符 ；#（井号）可以代替零个或多个标识符
 // 声明exchange
        channel.exchangeDeclare(EXCHANGE_NAME, "topic");
        // 发送消息, 指定RoutingKey
        channel.basicPublish(EXCHANGE_NAME, "item.delete", null, message.getBytes());
        
        接收方
        // 绑定队列到交换机
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "item.update");
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "item.delete");
   
   
   // 绑定队列到交换机. 通配符!
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "item.#"); 
        可获取
 ```
 
 
 