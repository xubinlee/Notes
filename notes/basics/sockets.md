# Sockets

![](https://github.com/xubinlee/Notes/blob/master/assets/sockets.png?raw=true)

**如何告知对方已发送完命令：**

1. 通过Socket关闭

   当Socket关闭，服务端就会收到响应的关闭信息。

   缺点：Socket关闭后，不能再接收消息，也不能再发送消息，需要重新创建Socket连接

2. 通过Socket关闭输出流方式

   socket.shutdownOutput()关闭输出流，可以继续读取输入流，但是不能再次发送消息；outputStream.close()关闭输入流，和直接关闭Socket一个性质；

3. 通过约定符号

   msg=“end”;while ((line= read.readLine)!=null && “end”.equals(line)){}

   优点：不需要关闭流

   缺点：需要额外的约定结束标志，太简单的容易出现在要发送的消息中，误被结束；太复杂的不好处理，还占带宽。

4. 通过指定长度

   ```java
   import java.io.OutputStream;
   import java.net.Socket;
   
   public class SocketClient {
     public static void main(String args[]) throws Exception {
       // 要连接的服务端IP地址和端口
       String host = "127.0.0.1";
       int port = 8888;
       // 与服务端建立连接
       Socket socket = new Socket(host, port);
       // 建立连接后获得输出流
       OutputStream outputStream = socket.getOutputStream();
       String message = "the first message!";
       //首先需要计算得知消息的长度
       byte[] sendBytes = message.getBytes("UTF-8");
       //然后将消息的长度优先发送出去
       outputStream.write(sendBytes.length >>8);
       outputStream.write(sendBytes.length);
       //然后将消息再次发送出去
       outputStream.write(sendBytes);
       outputStream.flush();
       //==========此处重复发送一次，实际项目中为多个命名，此处只为展示用法
       message = "第二条消息";
       sendBytes = message.getBytes("UTF-8");
       outputStream.write(sendBytes.length >>8);
       outputStream.write(sendBytes.length);
       outputStream.write(sendBytes);
       outputStream.flush();
       //==========此处重复发送一次，实际项目中为多个命名，此处只为展示用法
       message = "the third message!";
       sendBytes = message.getBytes("UTF-8");
       outputStream.write(sendBytes.length >>8);
       outputStream.write(sendBytes.length);
       outputStream.write(sendBytes);    
       outputStream.close();
   socket.close();
   
     }
   }
   ```
   
   ```java
   import java.io.InputStream;
   import java.net.ServerSocket;
   import java.net.Socket;
   
   public class SocketServer {
       public static void main(String[] args) throws Exception {
           // 监听指定的端口
           int port = 8888;
           ServerSocket server = new ServerSocket(port);
           
           // server将一直等待连接的到来
           System.out.println("server将一直等待连接的到来");
           Socket socket = server.accept();
           // 建立好连接后，从socket中获取输入流，并建立缓冲区进行读取
           InputStream inputStream = socket.getInputStream();
           byte[] bytes;
           // 因为可以复用Socket且能判断长度，所以可以一个Socket用到底
           while (true) {
               // 首先读取两个字节表示的长度
               int first = inputStream.read();
               //如果读取的值为-1 说明到了流的末尾，Socket已经被关闭了，此时将不能再去读取
               if (first == -1) {
                   break;
               }
               int second = inputStream.read();
               int length = (first << 8) + second;
               // 然后构造一个指定长的byte数组
               bytes = new byte[length];
               // 然后读取指定长度的消息即可
               inputStream.read(bytes);
               System.out.println("get message from client: " + new String(bytes, "UTF-8"));
           }
           inputStream.close();
           socket.close();
           server.close();
       }
   }
   ```

**服务端并发处理能力：**

&emsp;&emsp;通过循环处理多个Socket请求时，当一个请求的处理比较耗时时，后面的请求将被阻塞，所以一般都是用多线程的方式来处理Socket，每一个Socket请求创建一个线程来处理：

```java
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class SocketServer {
  public static void main(String args[]) throws Exception {
    // 监听指定的端口
    int port = 8888;
    ServerSocket server = new ServerSocket(port);
    // server将一直等待连接的到来
    System.out.println("server将一直等待连接的到来");

    //如果使用多线程，那就需要线程池，防止并发过高时创建过多线程耗尽资源
    ExecutorService threadPool = Executors.newFixedThreadPool(100);
    
    while (true) {
      Socket socket = server.accept();
      
      Runnable runnable=()->{
        try {
          // 建立好连接后，从socket中获取输入流，并建立缓冲区进行读取
          InputStream inputStream = socket.getInputStream();
          byte[] bytes = new byte[1024];
          int len;
          StringBuilder sb = new StringBuilder();
          while ((len = inputStream.read(bytes)) != -1) {
            // 注意指定编码格式，发送方和接收方一定要统一，建议使用UTF-8
            sb.append(new String(bytes, 0, len, "UTF-8"));
          }
          System.out.println("get message from client: " + sb);
          inputStream.close();
          socket.close();
        } catch (Exception e) {
          e.printStackTrace();
        }
      };
      threadPool.submit(runnable);
    }
  }
}
```

线程池优点：

1. 线程复用，创建线程耗时，回收线程慢

2. 防止短时间内高并发，指定线程池大小，超过数量将等待，防止短时间创建大量线程导致资源耗尽，服务挂掉