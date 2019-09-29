# NIO

### 服务端

1. 创建Selector

2. 通过ServerSocketChannel创建channel通道

3. 为channel通道绑定监听端口

4. 设置channel为非阻塞模式

5. 将channel注册到selector上，监听连接事件

6. 循环等待新接入的连接：

   1)         获取可用channel数量

   2)         如果channel数量为0跳过

   3)         获取可用channel的集合

   4)         迭代channel集合：

   &emsp;&emsp;&emsp;a)          SelectionKey实例
   
   &emsp;&emsp;&emsp;b)         移除Set中的当前selectoinKey
   
   &emsp;&emsp;&emsp;c)          根据就绪状态调用对应方法处理业务逻辑：接入事件/可读事件

**接入事件处理器：**

1. 创建socketChannel

2. 将socketChannel设置为非阻塞工作模式

3. 将channel注册到selector上，监听可读事件

4. 回复客户端提示信息

**可读事件处理器：**

1. 要从selectionKey中获取到已经就绪的channel

2. 创建buffer

3. 循环读取客户端请求信息：

&emsp;&emsp;1)         切换buffer为读模式

&emsp;&emsp;2)         读取buffer中的内容

&emsp;&emsp;3)         将channel再次注册到selector上，监听可读事件

&emsp;&emsp;4)         将客户端发送的请求信息，广播给其他客户端：

&emsp;&emsp;&emsp;&emsp;&emsp;a)          获取到所有已接入的客户端channel

&emsp;&emsp;&emsp;&emsp;&emsp;b)         循环向所有channel广播信息

### 客户端

1. 连接服务端

2. 新开线程，负责接收服

3. 1)         获取可用channel数量

   2)         如果channel数量为0跳过

   3)         获取可用channel的集合

   4)         迭代channel集合：

   &emsp;&emsp;&emsp;a)          SelectionKey实例

   &emsp;&emsp;&emsp;b)         移除Set中的当前selectoinKey

   &emsp;&emsp;&emsp;c)          根据就绪状态调用对应方法处理业务逻辑：可读事件

4. 向服务端发送数据
