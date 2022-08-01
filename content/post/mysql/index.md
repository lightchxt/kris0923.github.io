---
title: "一文了解MySql"
date: 2021-09-08T22:36:54+08:00
draft: false
categories: ["MySql"]
tags: ["MySql"]
---

## MySlq 逻辑架构


{{ $image := .Resources.GetMatch "mysql.png" }}

<img src="/post/mysql/mysql.png" width="{{ $image.Width }}" height="{{ $image.Height }}">

##  为什么不要使用长事务：
1. 长事务意味着系统里面存在着很老的事务视图，在事务提交之前这些回滚记录都必须保留，导致占用大量的存储空间
2. 占用锁资源，可能拖垮整库
## 事务的启动方式：
1. 显式启动事务语句， begin 或 start transaction。配套的提交语句是 commit，回滚语句是 rollback
2. set autocommit=0，这个命令会将这个线程的自动提交关掉。意味着如果只执行一个 select 语句，这个事务就启动了，而且并不会自动提交。这个事务持续存在直到你主动执行 commit 或 rollback 语句，或者断开连接
在set autocommit=0 的情况下，如果是长连接，就有可能导致意外的长事务，所以建议使用set autocommit=1

## 使用自增ID做主键有什么好处：
1. 二级索引叶子节点占用空间小，int占4字节，bigint 占8字节
2. 自增ID符合B+树索引递增插入的模式，每次插入一条新记录都是追加操作，不会涉及挪动其他数据
3. 用做KV存储的时候才适合用业务ID做主键
4. MySql 5.6 之后支持了索引下推。减少回表次数

## 全局锁和表锁：
## 数据库备份：
对于支持事务的引擎可以使用mysql-dump --single-transaction来保证视图的一致性，其他的需要使用FTWRL加全局锁来保证
表锁：
- 表锁
- 元数据锁
  
 lock tables ... read/write, unlock tables ... 释放锁
MDL (metadata lock) 不需要显示使用，在访问一个表的时候会自动加上，保证读写的正确性

## 行锁：
### 死锁和死锁检测：
### 出现死锁时有两种策略：
1. 直接进入等待，直到超时，超时时间通过innodb_wait_timeout 来设置
2. 发起死锁检测，发现死锁后主动回滚死锁链条中某一个事务，使其他事务能够继续执行，通过设置innodb_deadlock_decect

#### 死锁检测要消耗大量的CPU资源，所以需要解决热点行更新的性能问题：
1. 临时关掉死锁检测（不可取）
2. 控制并发度，比如控制同一行数据最多只有10个线程在同时更新，做在数据库服务端的中间件
3. 业务上做优化，将一行数据拆分成多行，但业务代码需要考虑详细，如某行数据为0等
### 乐观锁和悲观锁
乐观锁与悲观锁是从程序员的角度进行划分：
- 乐观锁：认为对同一数据的操作并不总发生，实用时间戳或者版本号机制来保证数据操作的排他性
- 悲观锁：通过数据库本身的锁机制来保证数据的排他性

## 索引选择：（优化器）
1. 扫描行数
2. 是否回表
3. 是否排序
4. 是否使用临时表
analyize table t 重新统计索引信息

## 为字符串类型的字段建立索引
1. 使用前缀索引，只使用前n个字符
2. 倒序存储，正着存前n个字符的区分度比较低，如身份证号
3. hash字段，可能有冲突，需要精确判断字段是否相同
### 缺点：
- 前缀索引不能使用覆盖索引
- 倒序存储和哈希都不支持范围查询

## MySql 为什么有时会发生的抖动
当出现以下情况时，系统正在刷脏页：
1. redo log满了，正在flush redo log
2. 内存不足，需要淘汰一些内存页，如果是脏页，要回写磁盘，保证每个数据页只有两种状态：
  - 在内存里，内存里就是正确的结果
  - 不在内存中，文件中是正确的结果，这样的话效率最高
3. MySql认为系统空闲，刷脏页
4. MySql正常关闭
   
> innodb_io_capacity 控制刷脏页的策略

## Order by 执行过程：
1. 全字段排序（数据单行长度小于max_length_for_sort_data）：从索引中找出满足条件的主键ID，按主键ID取出整行，将查询的字段放入sort_buffer中，然后多sort_buffer中的字段做快速排序（如果数据大于sort_buffer_size，还需要使用外部排序），取前n条记录返回客户端；
2. rowid排序：从索引中找出满足条件的主键ID，按主键ID取出整行，将排序字段和ID放入sort_buffer中，对sort_buffer 中的数据进行排序，取前n行，按主键ID从原表中取出查询字段返回给客户端
两种方式相比：全字段排序需要更多的内存，rowid排序需要再次惠表查询数据

## MySql不使用索引的场景：
1. 对索引字段做函数操作
2. 隐式类型转换
3. 字符编码转换
本质是都是：破坏了索引值的有序性：

## MySql查询慢的原因
1. 查询长时间不返回：
   - 等MDL锁
   - 等flush
   - 等行锁
2. 查询慢：
   - 没有索引，扫描行数多
   - 事务内一致性读，多次执行undo log

## 什么情况会出现幻读：
1. 在可重复读的隔离级别下，仅“当前读”才会出现幻读
2. 幻读仅指新插入的行
### Gap Lock：
为了解决幻读的问题，但也可能引起死锁（锁住同一个间隙，然后向同一个间隙insert数据）

在读已提交的隔离级别下，没有间隙锁，但是需要把binlog的格式设置为row（statement格式可能日志顺序错乱）

## MySql加锁规则：（两个原则，两个优化，一个BUG）
默认在RR隔离级别下
- 原则1:加锁的基本单位是next-key lock，是前开后闭区间
- 原则2:查找过程中访问到的对象才会加锁
- 优化1:索引上的等值查询，给唯一索引加锁的时候，next-key lock 退化为行锁
- 优化2:索引上的等值查询，向右查询且最后一个值不满足等值条件的时候，next-key lock退化为间隙锁
- 一个BUG：唯一索引上的范围查询会访问到不满足条件的第一个值为止

间隙锁和间隙锁之间并不冲突，限制的是不能insert

gh-ost MySql Online DDL 工具
pt-query-digist

## MySql 如何保证数据不丢：
1. Bin log的写入机制：事务执行的过程中写把日志写到binlog_cache,事务提交的时候先把binlog_cache写入到binlog文件中
trx running -> binlog_cache -> trx commit->binlog_file -> fsync disk
sync_binlog 控制什么时候fsync到磁盘
1. sync_binlog=0 的时候，表示每次提交事务都只 write，不 fsync；
2. sync_binlog=1 的时候，表示每次提交事务都会执行 fsync；
3. sync_binlog=N(N>1) 的时候，表示每次提交事务都 write，但累积 N 个事务后才 fsync。
2. redo log写入机制：
### 流程：
redolog_buffer -> write disk -> fsync(真正持久化)

所以redo log 有三种状态：
- innodb_flush_log_at_trx_commit控制了redolog的写入策略：
- innodb_flush_log_at_trx_commit = 0， 只写到redolog buffer
- innodb_flush_log_at_trx_commit = 1，每次事务提交都持久化到磁盘
- innodb_flush_log_at_trx_commit = 2，每次事务提交都写到文件系统的page cache中
## 性能优化：
- 组提交 binlog_group_commit_sync_delay（延迟多少微秒后才调用 fsync）， binlog_group_commit_sync_no_delay_count（累积多少次以后才调用 fsync） 减少 binlog 的写盘次数
- 将 sync_binlog 设置为大于 1 的值（比较常见是 100~1000）。这样做的风险是，主机掉电时会丢 binlog 日志。
- 将 innodb_flush_log_at_trx_commit 设置为 2。这样做的风险是，主机掉电的时候会丢数据。

## 主备一致：
 不应该设置binlog_format 为statement，可能会引起主备不一致（加了limit的时候，如果使用的索引不一样，排序可能不同），至少是mixed

每个库的server id 必须不同，生成的binlog中的server id不同，保证不会循环复制
### 主备延迟的原因：
- 备库的机器性能比主库差
- 备库查询压力大
- 主库执行大事务：
- 大表的DDL
### 可靠性优先的主从切换策略：
- 判断从库的seconds_behind_master小于某个值，比如5s，否则一直重复这一步
- 主库改readonly=true
- 判断从库的seconds_behind_master，直到等于0
- 从库修改readonly=false（可读写状态）
- 业务请求切换到从库（新主库）
### 并行复制：
coordinator 在分发Relay log的时候，需要满足以下这两个基本要求：
- 不能造成覆盖更新，这就要求更新同一行的两个事务必须分发到同一个worker中
- 同一个事务不能被拆开，必须放到同一个worker中
### 分发策略：
- 按表分发，适用于多表的负载均衡，对于热点数据表，可能退化成为单线程复制
- 按行分发，要求binlog的格式必须是row, 并行度高，但是操作大事务的时候，耗费内存和CPU
  
Maria 模拟主库的并行，将相同commitID的事务分发给不同的worker，但是也容易被大事务拖后腿

## GTID （主从同步）
全称是 Global Transaction Identifier，也就是全局事务 ID，是一个事务在提交的时候生成的，是这个事务的唯一标识。它由两部分组成，格式是：
  GTID=server_uuid:gno

server_uuid 是一个实例第一次启动时自动生成的，是一个全局唯一的值

gno 是一个整数，初始值是 1，每次提交事务的时候分配给这个事务，并加 1

## 读写分离的弊端
两种架构：客户端直连和带Proxy
### 主从延迟：
- 强制走主库
- sleep（不靠谱）
- 判断主备无延迟：
  - a. show salve status 判断second_behind_master是否等于0

  - b. Master_Log_File和Relay_master_Log_Pos、Read_Master_Log_Pos 和Exec_Master_Log_Pos 这两组值完全相同（读到主库的最新位点和备库执行的最新位点）

  - c. 对比GTID集合，Auto_Position=1, Retrieved_Gtid_set 和 Executed_Gtid_Set两个集合相同
  
b，c比a要精确很多，second_behind_master单位是秒

但是b, c 也是完全准确的，可能有一部分binlog正在从主库发送到从库

配合上semi-sync（半同步复制）+位点判断可以解决e的问题，但仅限于一主一从，多从的情况下，一个从库发送ack主库就提交事务了

业务高峰期，主库位点更新很快，所以位点判断一直不相等，可能出现从库迟迟无法响应的情况

#### 等主库位点的方案

主库事务提交后立即执行 show master status，得到当前的file 和position，从库执行select master_pos_wait(File, Position,1),如果大于0，则已包含刚才提交的事务

#### GTID方案
select wait_for_executed_gtid_set(gtid_set, 1) = 0 表示从库中已经包含传入的gtid_set

5.7 之后的版本会把gtid返回给客户端

## MySql健康检查
SHOW VARIABLES LIKE 'innodb_thread_concurrency';  并发查询数

等行锁和间隙锁的线程是不算在并发查询数里面的
select 1；
update 系统表数据
内部统计，判断单次I/O 是否大于某个时间

## 误删数据找回
### 误删行 delete
- flashback工具，需要保证binlig_format=row和binlog_row_image=FULL
- 提前预防：设置sql_safe_updates=on, delete 和update的语句中必须有where条件且包含索引字段
### 误删库/表
- 全量备份+增量日志 mysqlbinlog
- 搭建延迟复制备库，指定一个备库和主库有N秒的延迟，减少需要恢复的数据
### 预防：
- 帐号分离，不给Drop、truncate的权限
- 日常使用只读帐号
- 删表之前，先做更名操作，观察一段时间再删
### rm数据
- 高可用（HA）集群只要不是把整个集群删除，HA会自动选出一个新主库

## kill query/ kill connection 
1. MySql kill 和 linux的kill命令类似，并不是直接终止，而是给进程发送一个终止的信号；
2. 把被kill的session的状态改成 THD::KILL_QUERY
3. 给被kill的session 
### kill 不能生效的情况：
1. 线程没有执行到判断线程状态的逻辑（一直在循环判断是否可以执行）
2. 终止逻辑耗时较长
3. 超大事务执行被kill，回滚操作耗时较长
4. 大查询回滚，生成了比较大的临时文件，如果文件系统压力大，则删除临时文件可能需要等待I/O资源
5. DDL命令执行到最后阶段，如果被kill，也需要删除过程中的临时文件

## 读取数据
客户端使用 --quick 参数会使用mysq_use_result方法
mysql_use_result: 读一行处理一行，如果处理的很慢，客户端过很久才去读下一行，会出现sending to client
mysql_store_result: 将查询的结果保存到本地内存
查询结果是分段发送给客户端的，所以不会打爆内存
Innodb Buffer Pull的LRU 算法是用链表实现的，并且分成young区和old区（5：3）；每次淘汰都是在old去，在old区存在超过1s的数据才能进入young区，保证了在全表扫描的时候，正常业务的查询命中率不会降低

## Join
- 如果可以使用被驱动表的索引，join 语句是有其优势的；
- 不能使用被驱动表的索引，只能使用 Block Nested-Loop Join 算法，这样的语句就尽量不要使用；
- join_buffer_size 设定了join buffer的大小，越大被驱动表扫描的次数越少，扫描次数 N + k*M
- 在使用 join 的时候，应该让小表做驱动表
- 通过where条件过滤之后的数据量小的为小表


## Muity-Range Read 优化：
思路：因为大多数数据都是按照主键id自增的顺序插入到得到的，所以按照主键id递增的顺序查询是比较接近顺序读，能够提升读性能
实现：将根据索引定位到的记录放到read_rnd_buffer中，将read_rnd_buffer 中的id递增排序，用排序后的数组到主键索引中查找记录
BKA  (Batched Key Access)，对 BNL算法的优化
- 在被驱动表上加索引
- 创建临时表
不支持hash join

## 临时表的应用
- 分库分表系统的跨库查询
- 在使用分区key的查询中可以直接定位到数据放在了哪一个表
- 未使用到分区key的查询，只能到所有分区中区查找数据，然后统一操作
- 在proxy中实现，需要的工作量大，对中间层的开发能力比较高，对proxy的压力比较大
- 把各个分库拿到的数据汇总到一个Mysql实例的一个表中，然后在这个汇总的实力上操作（临时表）
### 内部临时表
#### 使用场景：
- UNION：需要对结果去重，需要使用临时表，Union ALL 就不需要
- Group By：无序的数据需要对结果统计计数
- 如果执行过程可以一边读数据，一边直接得到结果，就不需要额外的内存
- join_buffer 是无序数组，sort_buffer是有序数组，临时表是二维表结构
- 如果需要二维表的特性，就会考虑使用临时表（union 唯一索引约束，group by 需要额外的字段计数）
- SQL_BIG_RESULT 告诉Mysq 直接使用排序算法得到group by的结果

> 除了内存临时表，普通的内存表不建议使用，一是不支持行锁，性能可能并没有预想的好，二是在集群架构上数据持久化可能会有问题
主键自增

innoDB引擎的索引值存在内存里，MyISAM存储在数据文件中，8.0版本之后记录到了redolog中，保证重启也不会丢失
## 主键自增不连续的原因：
- 插入数据失败
- 事务回滚
- 批量插入，每次申请的自增id的数量都是上一次的两倍，最后未用完的就浪费了

## 表复制
### mysqldump
```
mysqldump -h$host -P$port -u$user --add-locks=0 --no-create-info --single-transaction  --set-gtid-purged=OFF db1 t --where="a>900" --result-file=/client_tmp/t.sql
```
### 导出csv文件
```
select * from db1.t where a>900 into outfile '/server_tmp/t.csv';
```
生成的文件在服务器上，不在客户端机器上
受 secure_file_priv 的限制
目标文件不存在
## 物理拷贝
5.6之前是不允许的，5.6之后引入了可传输表空间

## Flush Privileges
1. grant/revoke语句会同时修改数据表和内存，正常情况下是没有必要跟flush privileges 的；
2. 不规范的操作，直接使用DML操作系统权限表才需要跟flush privileges；

## 分区表
1. 对于引擎层来说，分区表是多个表；对于Server层来说分区表是一个表
2. 不建议使用分区表的原因：
MySql在第一次打开分区表的时候需要访问所有的分区
在Server层认为是同一张表，因此公用同一个MDl锁，执行DDL时，影响更大

## 自增ID
1. 如果没有指定自增ID，MySql会创建一个不可见的长度为6个字节的row_id; 当row_id达到最大值(2^48 -1)后，再申请row_id就是0，并且会覆盖原来的数据
2. redolog 和 binlog中的Xid（Server层维护）是事务中执行的第一个Sql的query_id, global_query_id 是一个内存变量，重启后会清零，但同时也会生成新的binlog文件，所以同一个binlog文件中的xid是唯一的
3. Innodb trx_id  事务ID是引擎层维护的，最大值2^48-1, 只读事务不分配trx_id，理论上，只要MySql运行的足够久，trx_id重新从0开始分配就会出现幻读