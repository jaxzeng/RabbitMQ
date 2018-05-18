```java
import com.qihoo.finance.msf.mq.AccessBuilder;
import com.qihoo.finance.msf.mq.base.Message;
import com.qihoo.finance.msf.mq.base.enums.ExchangeType;
import com.qihoo.finance.msf.mq.consumer.Consumer;
import com.qihoo.finance.msf.mq.producer.MessageProducer;
import com.qihoo.finance.msf.mq.producer.Producer;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.MessageProperties;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;

import java.io.IOException;
import java.util.Random;

public class Finalize {

    public static void main(String[] args) throws InterruptedException {
        Finalize finalize = new Finalize();
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory("127.0.0.1", 5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");
        connectionFactory.setChannelCacheSize(50);
        connectionFactory.setPublisherConfirms(true);

        connectionFactory.setPublisherReturns(true);
        connectionFactory.getRabbitConnectionFactory().setAutomaticRecoveryEnabled(true);

        while (true){
            Thread.sleep(5);

            MessageProducer messageProducer = new Producer(connectionFactory, ExchangeType.TOPIC, "topic.exchange.apv", "queue.apv.two", "queue.apv.two.key", "queue.apv.two.*", null);
            messageProducer.produce(new Message("dii"));
            try {
                messageProducer.destroy();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    public User build(CachingConnectionFactory connectionFactory){
        Channel channel = connectionFactory.createConnection().createChannel(false);
        return new User(){
            @Override
            public void send() {
                try {
                    channel.basicPublish("topic.exchange.apv","queue.apv.two.key",true, false, MessageProperties.BASIC, "xx".getBytes());
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            @Override
            protected void finalize() throws Throwable {
                super.finalize();
                channel.close();
                System.err.println("gc user soon");
            }
        };
    }
    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.err.println("gc Finalize soon");
    }
}


public class ReciveTest {

    public static  void main(String args[]){

    ReciveTest finalize = new ReciveTest();
    CachingConnectionFactory connectionFactory = new CachingConnectionFactory("127.0.0.1", 5672);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");
        connectionFactory.setChannelCacheSize(50);
        connectionFactory.setPublisherConfirms(true);

        connectionFactory.setPublisherReturns(true);
        connectionFactory.getRabbitConnectionFactory().setAutomaticRecoveryEnabled(true);

            MessageConsumer messageConsumer=new Consumer(connectionFactory, ExchangeType.TOPIC, "topic.exchange.apv", "queue.apv.two", "queue.apv.two.key", 1, new StringHandler());
              messageConsumer.consume();
    }
}

```