# CourseSelection: Distributed course selection system for JUFEer.

## 分布式选课系统——基于江西财经大学的实践

### 前言
> 
> 每到一学期末，选课成了各大高校学子心头的痛，学校设置的选课已经俨然不然叫做选课，而应该叫做是**抢课！！**
> 
> 基于这种状况，我设计了一种分布式选课系统架构，用来帮助各位提高选课成功率。这套系统的单机版本已经成功应用于江西财经大学内，分布式版本还尚待检验，其他学校也可以参考借鉴。

### 背景知识
1. Python, Requests, Tornado, Gevent
2. Celery, Redis, RabbitMQ (消息队列)
3. BeautifulSoup (网页解析)
4. Tesseract (验证码识别)
5. Fabric (自动部署)

### 江西财经大学选课平台简介
网址： <http://xk.jxufe.cn>

选课平台会在特定的时间段内，针对不同年级的学生开放，并且分为校内通道和校外通道，校外的公网可以连接校内 VPN 后使用校内通道进入。按照以往的经验，服务器是依次开放，每台服务器限制最大人数为150人（去年刚升级的，以前只能容纳100人），一台服务器满了最大限额人数之后再开放后面一台。

---

### 思路
由于服务器限定了最大容纳人数，并且服务器是依次开放的，所以需要在最短的时间内排队挤进第一台服务器，只有这样才可能成功地选到所有自己想要的课；一旦登录成功，获取到 cookies 之后，再根据预先设定好的课程信息进行相应的处理。这样就完成了整个选课过程。

到了服务器开放之前的一小段时间，网络异常的慢（因为大家都在刷新，大量请求拥向服务器），当采用分布式的架构之后，网络阻塞将不会有太大影响，多个地点的 SlaveNode 同时进行请求，降低了单一节点阻塞后无法快速登录成功的概率。


### 架构介绍
分布式架构采用 Celery 框架作为消息队列，一台 MasterNode 承担任务调度的作用，多台 SlaveNode 运行相同的程序，接收 MasterNode 分发的任务，这些 SlaveNode 支持横向扩展，可以在校内部署几台，并且在公网上部署几台，公网的几台通过VPN连接到校内网（校外通道要比校内通道晚开放几分钟）。

在服务器开放之前，MasterNode 分发登录任务，SlaveNode 开启多线程，每个线程处理一个选课通道的服务器，获取服务器状态。如果服务器已开放，进行登录操作，登录成功返回选课服务器 IP 以及 Cookies，结束登录过程。

MasterNode 获取到 IP 以及 Cookies 之后，查询task queue，确认是否现在有SlaveNode在进行下一步的选课过程或者整个过程是否已结束，如果没有，则发放选课任务，参数包括 IP，Cookies，所有的课程信息等。

SlaveNode 获取到下发的选课任务后，读取参数，并进行选课逻辑处理。成功后返回消息，结束选课过程。

Master 确认是否选课成功，并通过邮件或 PushBullet 推送消息。

整个过程结束。

这个过程还有很多地方需要优化。

### 实战效果
最近一次开放时间将在2015-12-21，这套分布式系统暂时还没投入使用，单机版已经测试成功，等服务器正式开放之后再Update

---

### 使用说明
#### 环境搭建：
**MasterNode:** 

安装RabbitMQ, Redis, Python的相关依赖已经写入requirements_master.txt

安装方法：`pip install -r < requirements_master.txt`

**SlaveNode:**

安装Tesseract，并且拷贝code.traineddata到`/usr/local/share/tesseract`目录下，Python相关依赖已经写入requirements_slave.txt

安装方法：`pip install -r < requirements_slave.txt`

#### 运行方法：
**TODO**

---

### 后期展望
1. 使用更高性能的网络库，比如Tornado, PyCurl等
2. 用更快速的网页解析库比如PyQuery等，替代BeautifulSoup
3. 继续优化任务调度算法
4. 将代码写得更加Pythonic


## License
[MIT-Liscense](http://mit-license.org/) &copy; 2015 xNathan.
