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

   Client：&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;Server：

![](https://github.com/xubinlee/Notes/blob/master/assets/socket-client.png?raw=true)           <img src="https://github.com/xubinlee/Notes/blob/master/assets/socket-server.png?raw=true" style="zoom:80%;" />   

**服务端并发处理能力：**

&emsp;&emsp;通过循环处理多个Socket请求时，当一个请求的处理比较耗时时，后面的请求将被阻塞，所以一般都是用多线程的方式来处理Socket，每一个Socket请求创建一个线程来处理：

![](https://github.com/xubinlee/Notes/blob/master/assets/socket-thread.png?raw=true)   

线程池优点：

1. 线程复用，创建线程耗时，回收线程慢

2. 防止短时间内高并发，指定线程池大小，超过数量将等待，防止短时间创建大量线程导致资源耗尽，服务挂掉