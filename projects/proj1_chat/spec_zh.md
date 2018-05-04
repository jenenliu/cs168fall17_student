# 聊天程序

在这次作业中, 你将需要写一个简单的程序通过网络连接用户：一个聊天服务器.  该程序类似 [Slack](https://www.slack.com) 或者 [IRC](https://tools.ietf.org/html/rfc1459), 你完成后的聊天服务器将能支持用户在不同群组中聊天. 用户可以创建并且加入群组，一旦一个用户加入某个群组中，那么他发送的消息通过群组的其他用户将会收到此消息。

此次作业(包括后面的所有作业)将用Python 2实现. 这次作业将会让你熟悉socket api编程

#### 一些要点

- 项目的基础代码可以在 [这里找到](https://github.com/jenenliu/cs168fall17_student/tree/master/projects/proj1_chat). 你可以自己下载或者用git clone下来.
- 你最后完成的时候应该只有两个文件: 一个是 `client.py`, 另外一个是 `server.py`. 无需修改 `utils.py`. 

#### 参考资源

- 往期做过次项目的一些演示视频在[这里可以看到](https://youtu.be/4btZs--wlpI).
- 如果你有任何疑问，可以先参考 [FAQ 部分](#faq).  如果问题还未解决，找你哥.
- [Python Socket 编程指南](https://docs.python.org/2/howto/sockets.html) 是一个很好的Python2 socket编程教程，你应该读一下.
- 我们准备了两个文件给你测试你的代码:
  - `client_split_messages.py` 能帮助并确保你的服务器能正常缓存消息, 详细描述在文档后面.
  - `simple_test.py` 通过模拟创建两个用户在一个群组里聊天来测试你的程序.  `simple_test.py` 只是通过客户端聊天的输出来测试你的代码是否正常工作，并且只能测试到一小部分.

##  什么是 sockets?

socket表示的是两个程序通过网络连接时它们所连接的点.  每个socket都会和某个特定的端口绑定在一起.  Socket 是操作系统提供的一层抽象: 程序创建了socket, 从这些socket里面读取信息, 并且向这些socket里面写入信息.  当一个程序向socket里面写入信息时, 操作系统将会通过该socket绑定的端口发送信息出去; 同样的, 当操作系统从某个端口接收到信息时, 这些信息将会被绑定该端口的socket所读取.

在Python里, 你可以通过[socket 标准库](https://docs.python.org/2/library/socket.html) 来创建socket并且连接到一个远程端口, 代码如下:

    import socket
    # socket构造函数会接收几个参数; 在这里使用默认参数即可.
    client_socket = socket.socket()
    client_socket.connect(("1.2.3.4", 5678))
    client_socket.sendall("Hello World")
    
这个例子中我们创建了一个socket并且连接到端口5678而IP地址是1.2.3.4的机器. 然后，它发送了一个"Hello World"消息到服务器 1.2.3.4:5678.

上面的例子创建了一个socket客户端并且只连接到一个远程机器.  而当你创建一个服务器时, 你将能同时接收多个客户端连接, 并且你创建客户端的socket时并不需要知道他们的地址.  所以，socket服务器的代码将会不太一样:

    server_socket = socket.socket()
    server_socket.bind(("1.2.3.4", 5678))
    server_socket.listen(5)
    
当你创建socket后, 相比于连接到某个特定的远程机器, 上面的代码 *绑定* 了socket在某个特定的IP地址和端口上, 此时便相当于告诉操作系统把这个socket绑定在对应的IP地址和端口上.  最后,  `listen` 会监听所有连接到此IP地址和端口的socket.  当有新的客户端连接到此socket时, python的socket标准库将会创建一个新的socket和此客户端保持连接和通信, 所以服务器的socket才能继续监听端口并等待其他新的客户端连接上来:

    (new_socket, address) = server_socket.accept()
    
这个函数将会一个堵塞直到有个新的客户端连接上来 (通过使用 `connect()` 函数, 就像上面的例子所展示的), 并且返回一个新创建的socket, `new_socket`, 这个socket可以从来和客户端发送并且接收该客户端的消息.  比如，下面的代码

    message = new_socket.recv(1024)
    
将会堵塞知道有新的消息从客户端那里接收过来, 并且将会返回最多1024字节的信息.

你将需要阅读一下Python socket教程来理解一下这些api的含义.  __特别需要注意的是，比如举个例子, 使用 `send` 和 `recv`! `send` 不需要发送所有传给它的数据.__  [Python Socket 编程教程](https://docs.python.org/2/howto/sockets.html) 是个很好的socket学习资源.

## 第一部分 

作业的第一部分将会帮助你开始使用socket编程api并且介绍一下最基本的客户端-服务端交互.  

在这一部分的作业中, 你需要写一个简单的客户端和服务器. 客户端将会从标准输入读取然后发送消息到服务器然后断开连接.  服务器需要输出所接收到的消息到标准输出.  如果有多个客户端连接到服务器, 服务器应当能排队处理 (比如, 它应当输出收到的当前客户端发过来的所有消息然后关闭客户端连接，然后接着处理下一个连接).

服务器程序应该接受一个命令行参数, 用来表示服务器使用的端口:

    $ python basic_server.py 12345
    
客户端应当接收两个参数: 一个是要连接的服务器的主机名字 (或者IP地址) 以及服务器的端口:

    $ python basic_client.py localhost 12345
    
你的服务器应当能被所有和你主机有网络连接的机器连接上 (具体看 FAQ).

### 阻塞型socket

在这部分的作业中, 使用阻塞型socket是没问题的.  "阻塞" 表示的是一个socket可能会停住一段时间, 直至整个函数处理完成.  socket调用完会被阻塞的情况包括:

- `send`: 如果socket所使用的缓冲区满了没法继续写入  (这是发送函数)
- `recv`: 如果socket所使用的缓冲区是空的没有东西可以读取 (比如, 如果客户端突然停止发送了)  (这是接收函数)
- `accept`: 如果此时没有客户端连接上来  (这是接收客户端连接的函数)

### 一些使用例子

你写好的客户端和服务器应当和下面的演示例子一样.  假设有两个客户端分别连上服务器:

    $ python basic_client.py localhost 12345
    I am a student in CS168. This class is awesome!
    $ python basic_client.py localhost 12345
    Why is Shenker so bad at drawing?
    
如果服务器在客户端连接上来之前已经在12345端口启动了, 它应当有如下输出:

    $ python basic_server.py 12345
    I am a student in CS168. This class is awesome!
    Why is Shenker so bad at drawing?
 

## 第二部分

在接下来的作业中, 你将需要在你上面所写的代码基础上写一个支持多用户多群组聊天的服务器. 这里的 [演示视频](https://youtu.be/4btZs--wlpI)可以
告诉你你的程序应当长什么样子

### 服务器功能

服务器程序应该接受一个命令行参数，改参数表示服务器绑定在哪个端口上.

#### 消息

和你在第一部分的服务器不同, 这部分作业的服务器需要支持多个用户同时在线并且可以同时互发消息.  每个客户端应当有个对应的名字 (这样每个客户端才能知道他们在给谁发送消息) 并且需要知道该客户端在哪个群组里面. 当客户端第一次连上来时，不会有对应的名字和群组.  服务器接收到的第一个客户端的消息将作为客户端的名字.

服务器从客户端接收的消息分为两种类型.  第一种类型的消息为命令控制消息; 命令控制消息始终以"/"开头.  有几种不同的控制消息需要服务器处理:

- `/join <channel>` 命令应当添加发送该命令的客户端到此群组中. 客户端每次只能在一个群组中, 所以如果一个客户端已经加入了一个群组, 这个命令将会把客户端从该群组中移除. 当一个客户端加入某个群组时, 应当有一条消息广播给群里的每个人告诉他们此客户端加入群组.  同样的，当一个客户端离开群组时, 一条此客户端退出群组的消息应当广播给群组里的每个人.
- `/create <channel>` 命令应当创建一个群组, 并且添加此客户端到该群组.  就像 `/join` 命令一样, 此时客户端应当从他已加入的群组中移除.
- `/list` 命令应当发送一条消息给客户端当前已有的所有群组, 每个群组用换行符隔开.

第二种类型的消息就是普通的群里聊天消息.  所有不是以 `/` 开头的消息都当做普通消息.  每次发送出去的消息应当广播给群组里的其他所有人, 消息开头应该是发消息的客户端的名字，用中括号括起来 (参考下面例子).  消息 _不应当_ 从一个群组发送到另外一个群组. 如果客户端不在某个群组里面，客户端发送消息时，服务器应该回复一个错误提醒消息.

当一个客户端断开连接时, 一个客户端下线的消息应当广播给群里所有的人.

##### Delineating Messages

Sockets provide a data stream functionality, but they don't delineate different messages.  When a given `recv` call returns some data, the socket won't tell you whether the data returned is a single message, or multiple messages, or part of one message.  As a result, you'll need a way to determine when a message ends.  For this assignment, use fixed length messages that have 200 characters for all messages (including messages from the server to the client). If a message is shorter than 200 characters, you should pad the message with spaces (and the receiver should strip any spaces off of the end of the message). You can assume that no messages are longer than 200 characters.

Be sure that your code correctly handles the case where less than one message is available in the socket's buffer (so a `recv` call returns fewer than 200 characters of data) and the case where more than one message is available in the socket's buffer.  You should handle partial messages by buffering: if a `recv` call returns only part of a message, your code should hold on to the part of the message until the remainder of the mesage is received, and then handle the complete message.  For example, if a client receives 150 characters from the server, it should hold those 150 characters until 50 more characters are received.  The client should only write the message to stdout once all 200 characters have been received.  To help you check for your server's handling of these cases, we've provided a special client (`client_split_messages.py`) that splits messages into many smaller messages before sending them to the server.  This client only tests some of the scenarios your server should handle!  You'll likely want to modify this client to test for additional cases.

#### Error Handling

Your server should handle cases where the client sends an invalid message by returning an appropriate error message to the client.  For example, if a client uses the `/join` command but doesn't provide the name of a channel to join, the server should send back an error message.  The provided `utils.py` includes format strings for all of the errors you should handle.  You can use these format strings using Python's string formatting operations.  For example, `utils.py` defines the following error message:

    CLIENT_SERVER_DISCONNECTED = "Server at {0}:{1} has disconnected"
    
You can use the `.format` function to replace the brackets with strings as follows:

    error_message = CLIENT_SERVER_DISCONNECTED.format("localhost", 12345)

__You are required to use the error messages defined as constants in `utils.py`.  If you do not use the constants defined in `utils.py` , you will not get credit for error handling.__

When commands lead to an error, the command should not cause any changes at the server.  For example, if a client is currently in the `cs168_tas` channel and then tries to join a channel that doesn't exist, the client should __not__ be removed from the `cs168_tas` channel.

We will only test for errors that have associated error messages in `utils.py` in our testing.  You're welcome to check for additional errors if you'd like, but you will not be graded on them.

### Client Functionality

Each client connects to a particular chat server, and is associated with a name.  Your client should be started as follows:

    $ python client.py Scott 127.0.0.1 55555
    
This command should connect to the server at the given address and port number, and then send a message with the name Scott.

After being started, the client should listen for messages from the server and from stdin.  Messages from the server should be printed to the command line (after being stripped of any spaces at the end) and messages from stdin should be sent to the server (after being padded with spaces, as needed).  The client code should print [Me] before each message sent by that particular client so that the client can differentiate its own messages from messages sent by others.  When the client gets a message from the server, it should write over the `[Me]` with the message from the server.  For an example of this, take a look at the demo video linked above.

Here's an example of a client's interaction with a server that was started locally on port 55555:

	python client.py Panda localhost 55555
	[Me] Hello world!
	Not currently in any channel. Must join a channel before sending messages.
	[Me] /list
	[Me] /create 168_tas
	[Me] /list
	168_tas
	[Me] Hello world!
	Alice has joined
	[Alice] Hi everyone! Does anyone know what we're doing on the first day of lecture?
	
After seeing Alice's message, Panda stopped his client. After Panda created the 168_tas channel, a second client started:

	python client.py Alice 127.0.0.1 55555
	[Me] /list
	168_tas
	[Me] /join 168_tas
	[Me] Hi everyone! Does anyone know what we're doing on the first day of lecture?
	Panda has left

### Details

We've provided a `utils.py` file that has error messages that you should use.  These are intended to make your life easier, and also to enable testing.  Be sure you use these messages; otherwise, your code will fail the tests!

### Non-blocking sockets

You'll need to use non-blocking sockets for this part of the assignment, because both your client and server need to receive data from multiple sources, in an unknown order.  Consider what would happen if your client used blocking sockets, as in part 0, with a call like:

    message_from_server = client_socket.recv(200)
    
Now suppose that the server doesn't send any messages for a while, but while the client is blocked waiting on the `recv` call to return, the user types some data into stdin.  The client should read the data from stdin and send it to the server -- but the client is stuck blocked waiting on data from the server socket!  To address this problem, you can use non-blocking sockets.

To use non-blocking sockets, you'll need to use the `select` call in the `select` library.  For more about how to use `select` and a very relevant example, take a look at [this page](http://www.bogotobogo.com/python/python_network_programming_tcp_server_client_chat_server_chat_client_select.php).  While you are required to use non-blocking sockets for reading data and accepting connections, it's fine to use blocking sockets for sending messages (since the messages you're sending are short and you don't need to handle sending a large number of messages in quick succession, `send` and `sendall` should not block for long periods of time).

## Submission Details

You will be submitting your project on [okpy](http://okpy.org). When you visit the webpage, sign in using your preferred email from bCourses (Unless you have changed it, this should default to your Berkeley email). You should already be automatically registered as a student in the course. If this is not the case or you encounter any issues, please fill out this [form](https://goo.gl/forms/JdFQkp933jHH3kJm1).

You can then upload your project files into the "Project 1" assignment by selecting the assignment and then selecting to create a new submission. You will not be receiving any feedback from the autograder until the project is over, but you can submit as many times as you want. By default, your most recent submission will be graded. If you don't want this behavior, you can select to have a previous one graded instead.

## FAQ

##### What python libraries can I use?

Our solution code imports select, socket, and sys.  You should not import any other python libraries without asking first on Piazza.

##### How do I figure out the IP address to use to connect to my chat server?

If you started a server locally, your client can access the server using "localhost" or 127.0.0.1.  These are special names / addresses that are always used to refer to the local machine.  localhost and 127.0.0.1 cannot be used if you're trying to connect to your server from a different machine, because if a different machine uses those addresses, they refer to the local interface on that machine.  If you'd like to access your chat server from another machine, you'll need to determine your machine's externally reachable IP address.  There are some websites that will do this (e.g., Google "What's my IP address"), or you can also do this by running the `ifconfig` command.  `ifconfig` will list all the local interfaces; e.g.:

    $ ifconfig
    lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> mtu 16384
	options=3<RXCSUM,TXCSUM>
		inet6 ::1 prefixlen 128 
		inet 127.0.0.1 netmask 0xff000000 
		inet6 fe80::1%lo0 prefixlen 64 scopeid 0x1 
		nd6 options=1<PERFORMNUD>
	en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
		ether 78:31:c1:c0:a7:44 
		inet6 fe80::7a31:c1ff:fec0:a744%en0 prefixlen 64 scopeid 0x4 
		inet 172.19.131.124 netmask 0xffffff00 broadcast 172.19.131.255
		nd6 options=1<PERFORMNUD>
		media: autoselect
		status: active
	en1: flags=963<UP,BROADCAST,SMART,RUNNING,PROMISC,SIMPLEX> mtu 1500
		options=60<TSO4,TSO6>
		ether 72:00:02:3f:79:50 
		media: autoselect <full-duplex>
		status: inactive
		
The first entry above is the loopback address, which can be used to access the machine locally.  You'll notice the IP address of this interface (listed after `inet`) is always `127.0.0.1`.  Look for an interface listed with an IP address that's different than the loopback one -- in this example, the externally reachable IP address is `127.19.131.124` (listed under `en0`).

If you're behind a [NAT](http://en.wikipedia.org/wiki/Network_address_translation), you won't be able to reach your server from different machines.  We'll learn more about NATs later this semester.  In the meantime, if you'd like to play around with using your server from remote machines, try running it while connected to the wifi in Soda hall, which doesn't use a NAT, and assigns users unique IP addresses.

##### What's a good port number to pass in?

Many low port numbers are reserved; try using a port number greater than 10000.

##### When I start the server, I get an error that says `Address already in use`

This error means that another process is currently using the port.  Sometimes this happens transiently -- for example, if your server exited with an error, and not all of the sockets it was using have been cleaned up by the operating system yet.  When this happens, try using a different port.

##### What maximum number of connections should I use in the `listen` call?

5 is fine for this assignment.

##### Using a fixed-length message seems clunky and wasteful.  What are some alternatives?

For this assignment, you must use fixed-length messages.  If you're curious about what's used in practice, there are two common approaches.  The first is to use a delimiter between messages (e.g., some unique sequence of characters that's unlikely to be seen in any messages, such as `:==`) that can be used to determine when a message has ended.  The second (and more common) option is for the first thing in any message to be the size of the message.

##### How can I write over the "[Me]" on stdout when I get a message from the server?

You can start writing at the beginning of the current line of stdout by using "\r" at the beginning of the message you output.  One thing to be careful about is messages from the server that are shorter than the length of the "[Me]" string. For example, suppose the client sends the `/list` command to the server, and one of the channels is named "A".   The output should look like:

    [Me] /list
    A

But if you're not careful, you'll end up with output that looks like:

    [Me] /list
    AMe]
    
Use `utils.CLIENT_WIPE_ME` to fix this issue (note that if you use this constant, you'll need a "\r" at the beginning of the next string printed to avoid extra whitespace).

##### Do I ever need to close the server socket?

No. It's fine if your server code is inside of a `while True:` loop, like the server in Part 0.

##### Should the client quit if the server disconnects?

Yes.  The client should quit and print the appropriate error message from `utils.py`.

##### How will our code be tested?

We'll do end-to-end tests using your client and server together, and we'll also do tests where we use our own client to interact with your server (and vice versa).  As a result, you should make sure that your client and server communicate as described in this document.

 ##### I'm running Windows and select isn't working.
  
Unfortunately, stdin does not count as a socket in Windows due to how file descriptors work differently. Since the tests will be run on a Linux machine, this will cause issues if you write the code to use the Windows version of file descriptors. Your options are to download a Linux virtual machine, or use the instructional servers through SSH, or use the lab machines to test your code. Side note: If you are using Windows 10, you can use the Linux subsystem for Windows and run your code from there and it may work.

### Acknowledgments

This assignment was inspired by the [Introduction to Socket Programming Assignment](http://www.cs.princeton.edu/courses/archive/spr15/cos461/assignments/0-sockets.html) in Princeton's Computer Networking course.
