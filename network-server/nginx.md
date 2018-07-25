# Nginx

## 什么是Nginx？

Nginx是一个高性能的HTTP和反向代理服务器，也是一个IMAP/POP3/SMTP服务器

Nginx是一款轻量级的Web服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器 目前使用的最多的web服务器或者代理服务器，像淘宝、新浪、网易、迅雷等都在使用

## 为什么要用Nginx？

优点：

- 跨平台、配置简单
- 非阻塞、高并发连接：处理2-3万并发连接数，官方监测能支持5万并发
- 内存消耗小：开启10个nginx才占150M内存 成本低廉：开源
- 内置的健康检查功能：如果有一个服务器宕机，会做一个健康检查，再发送的请求就不会发送到宕机的服务器了。重新将请求提交到其他的节点上。
- 节省宽带：支持GZIP压缩，可以添加浏览器本地缓存
- 稳定性高：宕机的概率非常小
- master/worker结构：一个master进程，生成一个或者多个worker进程
- 接收用户请求是异步的：浏览器将请求发送到nginx服务器，它先将用户请求全部接收下来，再一次性发送给后端web服务器，极大减轻了web服务器的压力
- 一边接收web服务器的返回数据，一边发送给浏览器客户端
- 网络依赖性比较低，只要ping通就可以负载均衡
- 可以有多台nginx服务器
- 事件驱动：通信机制采用epoll模型

## 为什么Nginx性能这么高？

得益于它的事件处理机制： 异步非阻塞事件处理机制：运用了epoll模型，提供了一个队列，排队解决

## Nginx是如何实现高并发的

service nginx start之后，然后输入#ps -ef|grep nginx，会发现Nginx有一个master进程和若干个worker进程，这些worker进程是平等的，都是被master fork过来的。在master里面，先建立需要listen的socket（listenfd），然后再fork出多个worker进程。当用户进入nginx服务的时候，每个worker的listenfd变的可读，并且这些worker会抢一个叫accept_mutex的东西，accept_mutex是互斥的，一个worker得到了，其他的worker就歇菜了。而抢到这个accept_mutex的worker就开始“读取请求--解析请求--处理请求”，数据彻底返回客户端之后（目标网页出现在电脑屏幕上），这个事件就算彻底结束。

nginx用这个方法是底下的worker进程抢注用户的要求，同时搭配“异步非阻塞”的方式，实现高并发量。

## 为什么不使用多线程？

因为线程创建和上下文的切换非常消耗资源，线程占用内存大，上下文切换占用cpu也很高，采用epoll模型避免了这个缺点

## Nginx是如何处理一个请求的呢？

首先，nginx在启动时，会解析配置文件，得到需要监听的端口与ip地址，然后在nginx的master进程里面

先初始化好这个监控的socket(创建socket，设置addrreuse等选项，绑定到指定的ip地址端口，再listen)

然后再fork(一个现有进程可以调用fork函数创建一个新进程。由fork创建的新进程被称为子进程 )出多个子进程出来

然后子进程会竞争accept新的连接。此时，客户端就可以向nginx发起连接了。当客户端与nginx进行三次握手，与nginx建立好一个连接后

此时，某一个子进程会accept成功，得到这个建立好的连接的socket，然后创建nginx对连接的封装，即ngx_connection_t结构体

接着，设置读写事件处理函数并添加读写事件来与客户端进行数据的交换。最后，nginx或客户端来主动关掉连接，到此，一个连接就寿终正寝了

## 正向代理

一个位于客户端和原始服务器(origin server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)

然后代理向原始服务器转交请求并将获得的内容返回给客户端。客户端才能使用正向代理

正向代理总结就一句话：代理端代理的是客户端

## 反向代理

反向代理（Reverse Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求，发给内部网络上的服务器

并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器

反向代理总结就一句话：代理端代理的是服务端

## 动态资源、静态资源分离

动态资源、静态资源分离是让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后

我们就可以根据静态资源的特点将其做缓存操作，这就是网站静态化处理的核心思路

动态资源、静态资源分离简单的概括是：动态文件与静态文件的分离

## 为什么要做动、静分离？

在我们的软件开发中，有些请求是需要后台处理的（如：.jsp,.do等等），有些请求是不需要经过后台处理的（如：css、html、jpg、js等等文件）

这些不需要经过后台处理的文件称为静态文件，否则动态文件。因此我们后台处理忽略静态文件。这会有人又说那我后台忽略静态文件不就完了吗

当然这是可以的，但是这样后台的请求次数就明显增多了。在我们对资源的响应速度有要求的时候，我们应该使用这种动静分离的策略去解决

动、静分离将网站静态资源（HTML，JavaScript，CSS，img等文件）与后台应用分开部署，提高用户访问静态代码的速度，降低对后台应用访问

这里我们将静态资源放到nginx中，动态资源转发到tomcat服务器中

## 负载均衡

负载均衡即是代理服务器将接收的请求均衡的分发到各服务器中

负载均衡主要解决网络拥塞问题，提高服务器响应速度，服务就近提供，达到更好的访问质量，减少后台服务器大并发压力

---
以上来源：[Nginx面试题](https://github.com/Homiss/Java-interview-questions/blob/master/Nginx/Nginx%E9%9D%A2%E8%AF%95%E9%A2%98.md)