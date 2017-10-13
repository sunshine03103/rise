# 基本概念
zookeeper是一个分布式，开放源码的分布式应用程序协调服务。
分布式应该程序基于它实现 《同步服务》,《配置维护》,《命名服务》等。
可靠，可扩展，分布式，可配置的协调机制，来统一系统的状态。

# 角色
## 领导者 leader
负责进行投票的发起和决议，更新系统状态
## 跟随者 follower
用于接收客户请求并向客户端返回结果，在选主过程中参与机票
## 观察者 Observer
可以接收客户端连接，将写请求转发给leader节点，但Observer不参与投票过程，只同步leader状态。Observer的目的是为了扩展系统，提高读取速度。
## 客户端 client
请求发起方

# 设计目的
1.最终一致性。2.可靠性。3.实时性。4.等待无关。5.原子性。6.顺序性。

# zookeeper工作原理
当服务启动或者在领导者崩溃后， zab就进入恢复模式（选主），当领导者被选举出来且大多数server完成和leader的状态同步后，恢复模式就结束了。
每个server在工作过程中有三种状态：
looking不知道leader是谁，正在搜寻。
leading当前server即为选举出来的leader.
following.leader已经选举出来，当前server与之同步。

## 选主流程
当leader崩溃或者leader失去大多数follower，进入恢复模式，恢复模式需要重新选举一个新的leader.让所有的server都恢复到一个正确的状态。
选举算法有两种：basic paxos 和 fast paxos（默认）算法实现的
## 同步流程
选完leader以后，zk就进入状态同步过程
1.leader等待server连接
2.follower连接leader，将最大的zxid发送给leader
3.leader跟据follower的zxid确定同步点
4.完成同步后通知follower已经成为uptodate的状态
5.follower收到uptodate的消息后，又可以重新接受client的请求进行服务了

# 工作流程
## leader工作流程
三个功能
### 恢复数据
### 维持与leader的心跳，接收leader的请求并判断leader的请求消息类型
### leader的消息类型主要
有ping消息（leader的心跳信息），
request消息(follower发送的提议信息，包括写请求及同步请求)，
ack消息(follower对提议的回复，超过半数的follower通过，则commint该提议)，
revalidate消息（用来延长session的有效时间），
跟据不同的消息类型，进行不同的处理。

## follower的工作流程
4个功能
### 向leader发送请求（ping消息，request消息，ack消息，revalidate消息）
### 接收leader消息并进行处理
### 接收client的请求，如果为写请求，发送给leader进行投票
### 返回client结果

follower消息循环处理如下几种来自leader的消息
### ping消息：心跳消息
### proposal消息:leader发起的提案，要求follower投票
### commint消息:服务器端最新一次提案的信息
### uptodate消息：表明同步完成
### revalidate消息:根据leader的revalidate结果，关闭等待的revalidate的session还是允许其接受消息
### sync消息：返回sync结果到客户端，这个消息最初由客户端发起，用来强制得到最新的更新。












