
###  [Basic Boost.Asio Anatomy](https://www.boost.org/doc/libs/1_81_0/doc/html/boost_asio/overview/basics.html "Basic Boost.Asio Anatomy")

已剪辑自: [https://www.boost.org/doc/libs/1_81_0/doc/html/boost_asio/overview/basics.html](https://www.boost.org/doc/libs/1_81_0/doc/html/boost_asio/overview/basics.html)

Boost.Asio may be used to perform both synchronous and asynchronous operations on I/O objects such as sockets. Before using Boost.Asio it may be useful to get a conceptual picture of the various parts of Boost.Asio, your program, and how they work together.

As an introductory example, let's consider what happens when you perform a connect operation on a socket. We shall start by examining synchronous operations.

Your program will have at least one I/O execution context, such as an boost::asio::io_context object, boost::asio::thread_pool object, or boost::asio::system_context. This I/O execution context represents your program's link to the operating system's I/O services.

boost::asio::io_context io_context;  
 

To perform I/O operations your program will need an I/O object such as a TCP socket:

boost::asio::ip::tcp::socket socket(io_context);  
 

When a synchronous connect operation is performed, the following sequence of events occurs:

1. Your program initiates the connect operation by calling the I/O object:

socket.connect(server_endpoint);  
 

2. The I/O object forwards the request to the I/O execution context.

3. The I/O execution context calls on the operating system to perform the connect operation.

4. The operating system returns the result of the operation to the I/O execution context.

5. The I/O execution context translates any error resulting from the operation into an object of type boost::system::error_code. An error_code may be compared with specific values, or tested as a boolean (where a false result means that no error occurred). The result is then forwarded back up to the I/O object.

6. The I/O object throws an exception of type boost::system::system_error if the operation failed. If the code to initiate the operation had instead been written as:

boost::system::error_code ec;  
socket.connect(server_endpoint, ec);  
 

then the error_code variable ec would be set to the result of the operation, and no exception would be thrown.

// 下面是异步操作时的事件(event)发生顺序

When an asynchronous operation is used, a different sequence of events occurs.

// 第一步是使用async_connect初始化连接，同时按照异步操作完成返回时的需要创建你自己的回调处理函数handler

1. Your program initiates the connect operation by calling the I/O object:

socket.async_connect(server_endpoint, your_completion_handler);  
 

where your_completion_handler is a function or function object with the signature:

void your_completion_handler(const boost::system::error_code& ec);  
 

The exact signature required depends on the asynchronous operation being performed. The reference documentation indicates the appropriate form for each operation.

// I/O object会将请求转发给  I/O execution context

2. The I/O object forwards the request to the I/O execution context.

// I/O execution context 会通知OS(操作系统)开始一次异步的连接

3. The I/O execution context signals to the operating system that it should start an asynchronous connect.

Time passes. (In the synchronous case this wait would have been contained entirely within the duration of the connect operation.)

// 当OS系统层面的操作完成时，结果会被放入一个queue， I/O execution contex可以将其读取

4. The operating system indicates that the connect operation has completed by placing the result on a queue, ready to be picked up by the I/O execution context.

// 如果使用 io_context 作为 I/O execution contex ，要想这个结果被io_context检索到，则需要手动执行 io_context::run() 或类似的方法。 在执行 io_context::run() 的时候如果有未完成的异步操作，则run会被阻塞，所以需要在你的第一个异步操作开始后立即执行io_context::run()

5. When using an io_context as the I/O execution context, your program must make a call to io_context::run() (or to one of the similar io_context member functions) in order for the result to be retrieved. A call to io_context::run() blocks while there are unfinished asynchronous operations, so you would typically call it as soon as you have started your first asynchronous operation.

//在run内部，I/O execution context(通常也是io_context)将操作结果出队，然后转换成error_code，最后传递给你的程序设置好的handler

6. While inside the call to io_context::run(), the I/O execution context dequeues the result of the operation, translates it into an error_code, and then passes it to your completion handler.

This is a simplified picture of how Boost.Asio operates. You will want to delve further into the documentation if your needs are more advanced, such as extending Boost.Asio to perform other types of asynchronous operations.


Reference: 
1. doc https://www.boost.org/doc/libs/1_81_0/doc/html/boost_asio.html
2. book https://theboostcpplibraries.com/boost.asio
3. examples https://alexott.net/en/cpp/BoostAsioNotes.html