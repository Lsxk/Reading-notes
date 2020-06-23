### innodb引擎

###### Read commited和Repeatable Read的区别

1. Repeatable Read为默认隔离级别
2. 均使用了**非锁定的一致性读**

> 一致性的非锁定读（consistent nonlocking read）是指InnoDB存储引擎通过行多版本控制（multi versioning）的方式来读取当前执行时间数据库中行的数据。如果读取的行正在执行DELETE或UPDATE操作，这时读取操作不会因此去等待行上锁的释放。相反地，InnoDB存储引擎会去读取行的一个快照数据。

3. Read Commited读最新的历史版本，而Repeatable Read读事务开始时的历史版本。如下图，二者结果不同：

   (普通select均时非锁定读，但MVCC使用的快照版本不同)
   ![avatar](pic/read_comp.PNG)

   Read Commited读的结果为1,1,0；Repeatable Read读的结果为1,1,1。

4. Repeatable Read使用Next-Key Lock（Record Lock + gap Lock）算法来解决了**幻读**的问题，因此，已经满足了ACID中隔离性要求，不需要SERIALIZABLE级别。

   参考链接https://www.jianshu.com/p/04a2a6580bff
   
###### 优化器选择不使用索引的情况

1. 辅助索引查询数据较多时（通常为range，join，超过20%），选选择直接查聚集索引，避免查书签离散读(查主键索引)，所以直接查主键索引；
2. 存在联合索引的情况下，在排序读的情况下，直接查联合索引，因为已经排序过了。

###### 怎么理解联合索引的最左匹配原则
> 举个例子，例如(a,b,c)联合索引，```select * from t where a=1 and b > 2 and c = 3```,其中a使用到了联合索引，查出a为1的所有记录，并且这些记录中，b是**有序的**，那么，b可以再次利用索引，二分法找出大于2的所有记录，这些记录中，c是**无序的**，因此，c不能使用二分查找了，不能再利用到索引了。

###### Mysql CheckPoint

将内存中的脏页刷新到磁盘的机制

主要了两种：

1. Sharp checkPoint：全量脏页刷到磁盘，发生在mysql关闭时。

2. Fuzzy checkPoint：部分脏页，有以下几种

   > master线程定时异步
   >
   > LRU移除时，如果是脏页，先刷回磁盘
   >
   > 脏页太多时
   >
   > 重做日志占满时
   
  
### 操作系统

###### 用户态和内核态切换，发生了什么？
> X86体系结构中包括了一个特殊的段类型：任务状态段（TSS），用它来存放硬件上下文。TSS反映了CPU上的当前进程的特权级。
1. 读取tr寄存器，访问TSS段
2. 从TSS段中的esp0、ss0获取进程内核栈的栈顶指针
3. 使用ss0和esp0指向的内核栈将当前进程的cs(代码段寄存器,代码段段值),eip(代码段即将被执行的下一条指令的偏移)也就是说cs+eip可确定CPU要执行的下条指令,eflags(程序运行状态),ss(堆栈段的段值),esp(栈顶)信息保存起来，这个过程也完成了由用户栈到内核栈的切换过程。
4. 把内核代码选择符写入CS寄存器，内核栈指针写入ESP寄存器，把内核入口点的线性地址写入EIP寄存器。
5. 执行完成后，恢复现场（之前压入内核栈中的寄存器值恢复）

###### 操作系统如何给进程分配内存？
首先来看下虚拟地址空间的布局
![avatar](pic/virtual_adress.png)

0. 如果申请的空间小，则使用brk，否则使用mmap在内存映射区域分配
1. brk系统调用将数据段_edata指针向上推
2. 返回_edata指针地址
3. 只建立页表而没有真实物理内存的映射，因此页表里的权限是R，发生Page Fault，在Page Fault回调中，linux会去申请一页内存，此时把页表权限设置为R+W。
4. 应用程序使用时通过CR3寄存器值，发生缺页异常（参考：https://lrita.github.io/2019/03/07/linux-page-fault/）。


###### 页面置换算法

背景：在非连续内存分配的基础上，把一部分内容放到外存   ----> 虚拟存储

虚拟页式存储，页表项中相关的字段有：

1. 驻留位：表示该页是否在内存
2. 修改位：表示在内存中的该页是否被修改过（如果被修改过，则在置换出该页时，需要将该页写回到外存）
3. 访问位：表示该页面是否被访问过（用于页面置换算法）
4. 保护位：只读、只写等。
5. 锁定位：页面不允许被置换。



缺页：虚拟页式存储中发生的。

CPU找页表中，改页表项的驻留位为0或未找到改页表项，则发出缺页异常，由缺页服务例程处理--->在外存中找到该页（**如何找？**），并将该页**换入内存**（如果没有空闲页面，则使用页面置换算法），并修改页表项。

1. 如何找？

   https://blog.csdn.net/m0_37962600/article/details/81448553

   do_pg_fault.c  --> 最终ireturn，重新执行出错的指令

   虚拟页和磁盘扇区的对应关系？swap_entry.c。页面项**prevent**位表示当前映射关系时 逻辑地址--->页号或者 逻辑地址--->磁盘扇区号。

2. 页面置换算法

   选择被置换的物理页面。分为**局部置换算法**（选择的置换页面仅限于当前进程占用的页），和全局置换算法。

   局部置换：1.最优算法，2.先进先出.3.LRU算法，4.时钟算法，5.最不常用算法

   全局置换：1.工作集算法，2.缺页率算法

   LRU算法的实现：1.用栈（严格不是栈），每次访问的页面都移动到栈顶。2.双端链表

   

2.进程

 程序在数据集上**动态执行**的过程。   --->内核维护执行需要的状态 PCB   
 
 
 
 ###### exec()程序加载
 elh文件头--可执行文件的格式
 
 
#### 计算机网络
 
   ###### 三次握手流程

   流程如下两图，主要涉及到状态的变化

   client: 发出syn后变成SYN_Send，收到ack后，变为ESTABLISHED

   server：收到syn后变成SYN_RECV，发出syn+ack。收到ack后变成ESTABLISHED

   ![avatar](pic/3次握手.gif)
   ![avatar](pic/3次握手2gif.gif)
   
###### 四次挥手流程
**为啥要四次挥手？三次行不行？**

因为TCP是一个全双工协议，必须单独拆除每一条信道。4次挥手的目的是终止数据传输，并回收资源，此时两个端点两个方向的序列号已经没有了任何关系，必须等待两方向都没有数据传输时才能拆除虚链路，不像初始化时那么简单，发现SYN标志就初始化一个序列号并确认SYN的序列号。因此必须单独分别在一个方向上终止该方向的数据传输。

如果是三次挥手，会怎么样？三次的话，被动关闭端在收到FIN消息之后，需要同时回复ACK和Server端的FIN消息。如果Server端在该连接上面并没有Pending的消息要处理，那么是可以的，如果Server端还需要等待一段时间才可以关闭另外一个方向的连接，那么这样的三次挥手就不能满足条件。
![avatar](pic/4次挥手.gif)
![avatar](pic/4次挥手2.gif)

**为什么最后client要Time-Wait要等待2MSL？**

1. server发送了最后一个fin给client后，server变为last-ack，期待client返回最后一个ack，然后closed，但不幸的是，client发送的ack，server没有收到。则这个时候server会重传fin，要确保client在closed之前没有收到这个重传的fin，所有要等待2msl。也就是**client发ack的时间 + server重传的fin时间**
2. 确保网络中所有报文段失效。
