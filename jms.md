# JMS

1. 点到点（P2P）模型

生产消息—>Add Queue—>消费消息—>Remove Queue

2. 发布/订阅（Pub/Sub）模型

生产消息—>发送到Topic—>多个消费者监听消息

*好处：消息发送到目的地后，消息发送者就可以继续做自己的事情，而不用等待消息被处理完成。

*一、*  *步骤：*

ConnectionFactory---->Connection--->Session--->Message

Destination + Session------------------------------------>Producer

Destination + Session------------------------------------>MessageConsumer

1. 首先需要得到ConnectionFactoy和Destination，这里创建一个一对一的Queue作为Destination。
      ConnectionFactory factory = new ActiveMQConnectionFactory("vm://localhost");
      Queue queue = new ActiveMQQueue("testQueue");

2. 然后又ConnectionFactory创建一个Connection, 再启动这个Connection: 
      Connection connection = factory.createConnection();
      connection.start();

3. 接下来需要由Connection创建一个Session:
      Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE)

4. 下面就可以创建Message了,这里创建一个TextMessage。
      Message message = session.createTextMessage("Hello JMS!");

5. 要想把刚才创建的消息发送出去，需要由Session和Destination创建一个消息生产者：
      MessageProducer producer = session.createProducer(queue);

6. 下面就可以发送刚才创建的消息了：
      producer.send(message);

7. 消息发送完成之后，我们需要创建一个消息消费者来接收这个消息：
      MessageConsumer comsumer = session.createConsumer(queue);
      Message recvMessage = comsumer.receive();

8. 消息消费者接收到这个消息之后，就可以得到它的内容：
      System.out.println(((TextMessage)recvMessage).getText());

二、  消息的消费者接收消息可以采用两种方式：

1. consumer.receive() 或 consumer.receive(int timeout)；

2. 注册一个MessageListener。

三、  如果有多个消费者同时监听一个Queue的话，无法确定一个消息最终会被哪一个消费者消费

四、  如果有多个消费者同时监听一个Topic的话，每一个消息都会被所有消费者消费