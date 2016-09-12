简单来说，一台服务器在单位时间里能处理的请求越多，服务器的能力越高，也就是服务器并发处理能力越强。

# 为什么需要高并发？

相比十年前，互联网已得到了广泛的普及与应用。现在我们甚至难以想象离开互联网的世界。

从最初基于 NCSA 进化到基于Apache web 提供的可进行简单文本操作的html，再到目前的全世界20亿用户永久在线媒介。互联网一直在高速进化，即时信息与娱乐服务变得更加轻盈精巧，在线业务的安全可靠性也发生了显著变化。因此，当下的网站服务比之前变得更复杂，web引擎需要在工程上更具健壮性和可扩展性。

**并发一直是web体系结构最大的挑战之一**。自web services应用以来，并发能力持续增长。流行站点服务10w甚至100w的访问量非常常见。

十年前，影响并发的主要因素是客户端缓慢－－用户使用ADSL或者拨号连接网络。现在，并发增长主要是由移动端和新的应用架构引起，这些应用采用长链接以便能让用户及时更新消息，微博和朋友圈等等。
浏览器行为的变化是另一个导致并发增长的原因，现代浏览器访问网站时一般会打开4～6个并发以提高页面加载速度。

## 慢客户端

设想一个基于Apache的简单网站服务器，该服务器生成相对简短的100K的返回体–由文本或图片组成的简单页面。服务器耗费了不到一秒的时间生成一个页面，但是在80kbps(10KB/s)的网络环境下却耗费了10秒传输到客户端。网站服务器相对较快地推送10KB的内容，却需要在释放连接前耗费10秒钟将内容发送到客户端。

现在设想有1000个客户端同时连接上，请求类似的内容。假设仅分配1M内存给每个客户端，那么就是说，1000个客户端，100KB的请求内容，服务端却要消耗1G的内存。

事实上，一个典型的基于Apache的网站服务器需要为每个client分配超过1M的内存，而移动端的有效速度往往也在几十kbps。针对发送内容到客户端低效这个问题，虽然提高操作系统内核的socket缓冲区的大小，能一定程度上解决缓解问题，但是并不是一个通用的解决方案，并且有无法预期的副作用。



# Web 服务器并发处理

**单进程**：此种架构方式中，web服务器一次处理一个请求，结束后读取并处理下一个请求。在某请求处理过程中，其它所有的请求将被忽略，因此，在并发请求较多的场景中将会出现严重的性能问题。

**多进程/多线程**：此种架构方式中，web服务器生成多个进程或线程并行处理多个用户请求，进程或线程可以按需或事先生成。有的web服务器应用程序为每个用户请求生成一个单独的进程或线程来进行响应，不过，一旦并发请求数量达到成千上万时，多个同时运行的进程或线程将会消耗大量的系统资源。

**I/O多路复用**：为了能够支持更多的并发用户请求，越来越多的web服务器正在采用多种复用的架构：即**同步监控所有的连接请求的活动状态，当一个连接的状态发生改变时(如数据准备完毕或发生某错误)，将为其执行一系列特定操作**；在操作完成后，此连接将重新变回暂时的稳定态并返回至打开的连接列表中，直到下一次的状态改变。由于其多路复用的特性，进程或线程不会被空闲的连接所占用，因而可以提供高效的工作模式。

**基于事件的模型**：一个进程处理多个请求，并且通过epoll机制来通知用户请求完成。

## nginx

nginx采用了模块化、事件驱动、异步、单线程及非阻塞的架构，并大量采用了多路复用及事件通知机制。在nginx中，连接请求由为数不多的几个仅包含一个线程的进程worker以高效的回环(run-loop)机制进行处理，而每个worker可以并行处理数千个的并发连接及请求。
 
Nginx会按需同时运行多个进程：一个主进程(master)和几个工作进程(worker)，配置了缓存时还会有缓存加载器进程(cache loader)和缓存管理器进程(cache manager)等。所有进程均是仅含有一个线程，并主要通过“共享内存”的机制实现进程间通信。主进程以root用户身份运行，而worker、cache loader和cache manager均应以非特权用户身份运行。
 
主进程主要完成如下工作：

1. 读取并验正配置信息；
2. 创建、绑定及关闭套接字；
3. 启动、终止及维护worker进程的个数；
4. 无须中止服务而重新配置工作特性；
5. 控制非中断式程序升级，启用新的二进制程序并在需要时回滚至老版本；
6. 重新打开日志文件；
7. 编译嵌入式perl脚本；
 
worker进程主要完成的任务包括：

1. 接收、传入并处理来自客户端的连接；
2. 提供反向代理及过滤功能；
3. nginx任何能完成的其它任务；

如果负载以CPU密集型应用为主，如SSL或压缩应用，则worker数应与CPU数相同；如果负载以IO密集型为主，如响应大量内容给客户端，则worker数应该为CPU个数的1.5或2倍
 

# 更多阅读

[如何提高服务器并发处理能力](http://blog.arganzheng.me/posts/how-to-improve-server-concurrency.html)  
[Nginx介绍(为什么高并发很重要)](http://andremouche.github.io/nginx/nginx-introduction-I.html)  
[Nginx介绍(Nginx架构综述)](http://andremouche.github.io/nginx/nginx-introduction-II.html)  
[Web服务器处理并发连接请求的工作模型](http://www.360doc.com/content/15/0907/11/7256015_497436359.shtml)  

