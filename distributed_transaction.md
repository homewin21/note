## 2PC
	基于XA协议 由TM调控多个RM 保证了ACID
	1、该模式对代码的嵌入性为低。
	2、该模式仅限于本地存在连接对象且可通过连接对象控制事务的模块。
	3、该模式下的事务提交与回滚是由本地事务方控制，对于数据一致性上有较高的保障。
	4、该模式缺陷在于代理的连接需要随事务发起方一共释放连接，增加了连接占用的时间。

* 同步阻塞问题。执行过程中，所有参与节点都是事务阻塞型的。当参与者占有公共资源时，其他第三方节点访问公共资源不得不处于阻塞状态。

* 单点故障。由于协调者的重要性，一旦协调者发生故障。参与者会一直阻塞下去。尤其在第二阶段，协调者发生故障，那么所有的参与者还都处于锁定事务资源的状态中，而无法继续完成事务操作。（如果是协调者挂掉，可以重新选举一个协调者，但是无法解决因为协调者宕机导致的参与者处于阻塞状态的问题）

* 数据不一致。在二阶段提交的阶段二中，当协调者向参与者发送commit请求之后，发生了局部网络异常或者在发送commit请求过程中协调者发生了故障，这回导致只有一部分参与者接受到了commit请求。而在这部分参与者接到commit请求之后就会执行commit操作。但是其他部分未接到commit请求的机器则无法执行事务提交。于是整个分布式系统便出现了数据部一致性的现象。

* 二阶段无法解决的问题：协调者再发出commit消息之后宕机，而唯一接收到这条消息的参与者同时也宕机了。那么即使协调者通过选举协议产生了新的协调者，这条事务的状态也是不确定的，没人知道事务是否被已经提交。

[参考资料](https://www.cnblogs.com/duanxz/p/4672708.html)

## TCC
	TCC事务机制相对于传统事务机制（X/Open XA Two-Phase-Commit），其特征在于它不依赖资源管理器(RM)对XA的支持，
	而是通过对（由业务系统提供的）业务逻辑的调度来实现分布式事务。
	主要由三步操作，Try: 尝试执行业务、 Confirm:确认执行业务、 Cancel: 取消执行业务。

* 该模式对代码的嵌入性高，要求每个业务需要写三种步骤的操作。
* 该模式对有无本地事务控制都可以支持使用面广。
* 数据一致性控制几乎完全由开发者控制，对业务开发难度要求高。
* 牺牲了ACID中的C和I，但它带来了可观的收益：资源不再需要长时间上锁，极大地提高了吞吐量。最终Confirm阶段结束后，或者Cancel阶段结束后数据都会保持一致

[参考资料](https://blog.csdn.net/z69183787/article/details/86699181)

## TXC

	TXC模式命名来源于淘宝，实现原理是在执行SQL之前，先查询SQL的影响数据，然后保存执行的SQL快走信息和创建锁。
	当需要回滚的时候就采用这些记录数据回滚数据库，有三种模式（AT,MT,RT），对应不同的场景
[参考资料](https://blog.csdn.net/z69183787/article/details/86699181)	

## 第三方依赖实现-LCN
[文档](https://www.codingapi.com/docs)
[官网](https://www.codingapi.com/)
	
