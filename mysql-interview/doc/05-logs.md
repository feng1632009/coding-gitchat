# MySQL 中日志的面试题总结

1. #### MySQL 有哪些重要的日志文件？

   MySQL 中的重要日志分为以下几个： 

   **① 错误日志**：用来记录 MySQL 服务器运行过程中的错误信息。比如，无法加载 MySQL 数据库的数据文件，或权限不正确等都会被记录在此，还有复制环境下，从服务器进程的信息也会被记录进错误日志。默认情况下，错误日志是开启的，且无法被禁止。默认情况下，错误日志是存储在数据库的数据文件目录中，名称为 hostname.err，其中 hostname 为服务器主机名。在 MySQL 5.5.7 之前，数据库管理员可以删除很长时间之前的错误日志，以节省服务器上的硬盘空间， MySQL 5.5.7 之后，服务器将关闭此项功能，只能使用重命名原来的错误日志文件，手动冲洗日志创建一个新的，命令为：

   ```shell
   mv hostname.err  hostname.err.old mysqladmin flush-logs
   ```

   **② 查询日志**：查询日志在 MySQL 中被称为 general log(通用日志)，查询日志里的内容不要被“查询日志”误导，认为里面只存储 select 语句，其实不然，查询日志里面记录了数据库执行的所有命令，不管语句是否正确，都会被记录，具体原因如下:

   - insert 查询为了避免数据冲突，如果此前插入过数据，当前插入的数据如果跟主键或唯一键的数据重复那肯定会报错；
   - update 时也会查询因为更新的时候很可能会更新某一块数据；
   - delete 查询，只删除符合条件的数据；

   因此都会产生日志，在并发操作非常多的场景下，查询信息会非常多，那么如果都记录下来会导致 IO 非常大，影响 MySQL 性能，因此如果不是在调试环境下，是不建议开启查询日志功能的。

   查询日志的开启有助于帮助我们分析哪些语句执行密集，执行密集的 select 语句对应的数据是否能够被缓存，同时也可以帮助我们分析问题，所以，我们可以根据自己的实际情况来决定是否开启查询日志。

   查询日志模式是关闭的，可以通过以下命令开启查询日志：

   ```mysql
   set global general_log=1;
   set global log_output='table';
   ```

   general_log=1 为开启查询日志，0 为关闭查询日志，这个设置命令即时生效，不用重启 MySQL 服务器。

   **③ 慢日志**：慢查询会导致 CPU、IOPS、内存消耗过高，当数据库遇到性能瓶颈时，大部分时间都是由于慢查询导致的。开启慢查询日志，可以让 MySQL 记录下查询超过指定时间的语句，之后运维人员通过定位分析，能够很好的优化数据库性能。默认情况下，慢查询日志是不开启的，只有手动开启了，慢查询才会被记录到慢查询日志中。使用如下命令记录当前数据库的慢查询语句：

   ```mysql
   set global slow_query_log='ON';
   ```

   使用 set global slow_query_log='ON' 开启慢查询日志，只是对当前数据库有效，如果 MySQL 数据库重启后就会失效。所以如果要永久生效，就要修改配置文件 my.cnf，设置 slow_query_log=1 并重启 MySQL 服务器。

   **④ redo log（重做日志）**：为了最大程度的避免数据写入时，因为 IO 瓶颈造成的性能问题，MySQL 采用了这样一种缓存机制，先将数据写入内存中，再批量把内存中的数据统一刷回磁盘。为了避免将数据刷回磁盘过程中，因为掉电或系统故障带来的数据丢失问题，InnoDB 采用 redo log 来解决此问题。

   **⑤ undo log（回滚日志）**：用于存储日志被修改前的值，从而保证如果修改出现异常，可以使用 undo log 日志来实现回滚操作。 undo log 和 redo log 记录物理日志不一样，它是逻辑日志，可以认为当 delete 一条记录时，undo log 中会记录一条对应的 insert 记录，反之亦然，当 update 一条记录时，它记录一条对应相反的 update 记录，当执行 rollback 时，就可以从 undo log 中的逻辑记录读取到相应的内容并进行回滚。undo log 默认存放在共享表空间中，在 MySQL 5.6 中，undo log 的存放位置还可以通过变量 innodb_undo_directory 来自定义存放目录，默认值为“.”表示 datadir 目录。

   **⑥ bin log（二进制日志）**：是一个二进制文件，主要记录所有数据库表结构变更，比如，CREATE、ALTER TABLE 等，以及表数据修改，比如，INSERT、UPDATE、DELETE 的所有操作，bin log 中记录了对 MySQL 数据库执行更改的所有操作，并且记录了语句发生时间、执行时长、操作数据等其它额外信息，但是它不记录 SELECT、SHOW 等那些不修改数据的 SQL 语句。

   binlog 的作用如下：

   - 恢复（recovery）：某些数据的恢复需要二进制日志。比如，在一个数据库全备文件恢复后，用户可以通过二进制日志进行 point-in-time 的恢复；
   - 复制（replication）：其原理与恢复类似，通过复制和执行二进制日志使一台远程的MySQL数据库（一般称为 slave 或者 standby）与一台 MySQL 数据库（一般称为 master 或者 primary）进行实时同步；
   - 审计（audit）：用户可以通过二进制日志中的信息来进行审计，判断是否有对数据库进行注入攻击。

   除了上面介绍的几个作用外，binlog 对于事务存储引擎的崩溃恢复也有非常重要的作用，在开启 binlog 的情况下，为了保证 binlog 与 redo 的一致性，MySQL 将采用事务的两阶段提交协议。当 MySQL 系统发生崩溃时，事务在存储引擎内部的状态可能为 prepared（准备状态）和 commit（提交状态）两种，对于 prepared 状态的事务，是进行提交操作还是进行回滚操作，这时需要参考 binlog，如果事务在 binlog 中存在，那么将其提交；如果不在 binlog 中存在，那么将其回滚，这样就保证了数据在主库和从库之间的一致性。

   binlog 默认是关闭状态，可以在 MySQL 配置文件（my.cnf）中通过配置参数 log-bin = [base-name] 开启记录 binlog 日志，如果不指定 base-name，则默认二进制日志文件名为主机名，并以自增的数字作为后缀，比如：mysql-bin.000001，所在目录为数据库所在目录（datadir）。

   通过以下命令来查询 binlog 是否开启：

   ```mysql
   show variables like 'log_%';
   ```

   ```
   +----------------------------------------+----------------------------------------+
   | Variable_name                          | Value                                  |
   +----------------------------------------+----------------------------------------+
   | log_bin                                | ON                                     |
   | log_bin_basename                       | /var/lib/mysql/binlog                  |
   | log_bin_index                          | /var/lib/mysql/binlog.index            |
   | log_bin_trust_function_creators        | OFF                                    |
   | log_bin_use_v1_row_events              | OFF                                    |
   | log_error                              | stderr                                 |
   | log_error_services                     | log_filter_internal; log_sink_internal |
   | log_error_suppression_list             |                                        |
   | log_error_verbosity                    | 2                                      |
   | log_output                             | FILE                                   |
   | log_queries_not_using_indexes          | OFF                                    |
   | log_raw                                | OFF                                    |
   | log_slave_updates                      | ON                                     |
   | log_slow_admin_statements              | OFF                                    |
   | log_slow_extra                         | OFF                                    |
   | log_slow_slave_statements              | OFF                                    |
   | log_statements_unsafe_for_binlog       | ON                                     |
   | log_throttle_queries_not_using_indexes | 0                                      |
   | log_timestamps                         | UTC                                    |
   +----------------------------------------+----------------------------------------+
   19 rows in set (0.01 sec)
   ```

   binlog 格式分为: STATEMENT、ROW 和 MIXED 三种：

   - STATEMENT 格式的 binlog 记录的是数据库上执行的原生 SQL 语句。这种格式的优点是简单，简单地记录和执行这些语句，能够让主备保持同步，在主服务器上执行的 SQL 语句，在从服务器上执行同样的语句。另一个好处是二进制日志里的时间更加紧凑，所以相对而言，基于语句的复制模式不会使用太多带宽，同时也节约磁盘空间。并且通过 mysqlbinlog 工具容易读懂其中的内容。缺点就是同一条 SQL 在主库和从库上执行的时间可能稍微或很大不相同，因此在传输的二进制日志中，除了查询语句，还包括了一些元数据信息，如当前的时间戳。即便如此，还存在着一些无法被正确复制的 SQL。比如，使用 INSERT INTO TB1 VALUE(CUURENT_DATE()) 这一条使用函数的语句插入的数据复制到当前从服务器上来就会发生变化，存储过程和触发器在使用基于语句的复制模式时也可能存在问题；另外一个问题就是基于语句的复制必须是串行化的，比如：InnoDB 的 next-key 锁等，并不是所有的存储引擎都支持基于语句的复制；
   - ROW 格式是从 MySQL 5.1 开始支持基于行的复制，也就是基于数据的复制，基于行的更改。这种方式会将实际数据记录在二进制日志中，它有其自身的一些优点和缺点，最大的好处是可以正确地复制每一行数据，一些语句可以被更加有效地复制，另外就是几乎没有基于行的复制模式无法处理的场景，对于所有的 SQL 构造、触发器、存储过程等都能正确执行；它的缺点就是二进制日志可能会很大，而且不直观，所以，你不能使用 mysqlbinlog 来查看二进制日志，也无法通过看二进制日志判断当前执行到那一条 SQL 语句。现在对于 ROW 格式的二进制日志基本是标配了，主要是因为它的优势远远大于缺点，并且由于 ROW 格式记录行数据，所以可以基于这种模式做一些 DBA 工具，比如数据恢复，不同数据库之间数据同步等；
   - MIXED 也是 MySQL 默认使用的二进制日志记录方式，但 MIXED 格式默认采用基于语句的复制，一旦发现基于语句的无法精确的复制时，就会采用基于行的复制。比如用到 UUID()、USER()、CURRENT*USER()、ROW*COUNT() 等无法确定的函数。

2. #### redo log 和 binlog 有什么区别？

   redo log（重做日志）和 binlog（归档日志）都是 MySQL 的重要的日志，它们的区别如下：

   - redo log 是物理日志，记录的是“在某个数据页上做了什么修改”。
   - binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1 ”。
   - redo log 是 InnoDB 引擎特有的；binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用。
   - redo log 是循环写的，空间固定会用完；binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。

   最开始 MySQL 里并没有 InnoDB 引擎，MySQL 自带的引擎是 MyISAM，但是 MyISAM 没有 crash-safe 的能力，binlog 日志只能用于归档。而 InnoDB 是另一个公司以插件形式引入 MySQL 的，既然只依靠 binlog 是没有 crash-safe 能力的，所以 InnoDB 使用另外一套日志系统，也就是 redo log 来实现 crash-safe 能力。

3. #### 什么是 crash-safe？

   crash-safe 是指发生宕机等意外情况下，服务器重启后数据依然不会丢失的情况。

4. #### 什么是脏页和干净页？

   MySQL 为了操作的性能优化，会把数据更新先放入内存中，之后再统一更新到磁盘。当内存数据和磁盘数据内容不一致的时候，我们称这个内存页为脏页；内存数据写到磁盘后，内存的数据和磁盘上的内容就一致了，我们称为“干净页”。

5. #### 什么情况下会引发 MySQL 刷脏页（flush）的操作？

   - 内存写满了，这个时候就会引发 flush 操作，对应到 InnoDB 就是 redo log 写满了；
   - 系统的内存不足了，当需要新的内存页的时候，就会淘汰一些内存页，如果淘汰的是脏页这个时候就会触发 flush 操作；
   - 系统空闲的时候，MySQL 会同步内存中的数据到磁盘也会触发 flush 操作；
   - MySQL 服务关闭的时候也会刷脏页，触发 flush 操作。

6. #### MySQL 刷脏页的速度很慢可能是什么原因？

   在 MySQL 中单独刷一个脏页的速度是很快的，如果发现刷脏页的速度很慢，说明触发了 MySQL 刷脏页的“连坐”机制，MySQL 的“连坐”机制是指当 MySQL 刷脏页的时候如果发现相邻的数据页也是脏页也会一起刷掉，而这个动作可以一直蔓延下去，这就是导致 MySQL 刷脏页慢的原因了。

7. #### 如何控制 MySQL 只刷新当前脏页？

   在 InnoDB 中设置 innodb_flush_neighbors 这个参数的值为 0，来规定 MySQL 只刷当前脏页，MySQL 8 这个值默认是 0。

8. #### MySQL 的 WAL 技术是解决什么问题的？

   > A.防止误删除，找回数据用的
   >
   > B.容灾恢复，为了还原异常数据用的
   >
   > C.事务处理，为了数据库的稳定性
   >
   > D.为了降低 IO 成本
   
   答：D
   
   题目解析：WAL 技术的全称是 Write Ahead Logging（中文：预写式日志），是先写日志，再写磁盘的方式，因为每次更新都写磁盘的话 IO 成本很高，所以才有了 WAL 技术。
   
9. #### 为什么有时候会感觉 MySQL 偶尔卡一下？

   如果偶尔感觉 MySQL 卡一下，可能是 MySQL 正在刷脏页，正在把内存中的更新操作刷到磁盘中。

10. #### redo log 和 binlog 是怎么关联的?

    它们有一个共同的数据字段，叫 XID。崩溃恢复的时候，会按顺序扫描 redo log：

    - 如果碰到既有 prepare、又有 commit 的 redo log，就直接提交；
    - 如果碰到只有 parepare、而没有 commit 的 redo log，就拿着 XID 去 binlog 找对应的事务。

11. #### MySQL 怎么知道 binlog 是完整的?

    - statement 格式的 binlog，完整的标识是最后有 COMMIT 关键字。
    - row 格式的 binlog，完整的标识是最后会有一个 XID event 关键字。

12. #### MySQL 中可不可以只要 binlog，不要 redo log？

    不可以，binlog 没有崩溃恢复的能力。

13. #### MySQL 中可不可以只要 redo log，不要 binlog？

    不可以，原因有以下两个：

    - redo log 是循环写不能保证所有的历史数据，这些历史数据只能在 binlog 中找到；
    - binlog 是高可用的基础，高可用的实现原理就是 binlog 复制。

14. #### 为什么 binlog cache 是每个线程自己维护的，而 redo log buffer 是全局共用的？

    因为 binlog 是不能“被打断的”，一个事务的 binlog 必须连续写，因此要整个事务完成后，再一起写到文件里。而 redo log 并没有这个要求，中间有生成的日志可以写到 redo log buffer 中，redo log buffer 中的内容还能“搭便车”，其他事务提交的时候可以被一起写到磁盘中。

15. #### 事务执行期间，还未提交，如果发生 crash，redo log 丢失，会导致主备不一致呢？

    不会，因为这时候 binlog 也还在 binlog cache 里，没发给备库，crash 以后 redo log 和 binlog 都没有了，从业务角度看这个事务也没有提交，所以数据是一致的。

16. #### 在 MySQL 中用什么机制来优化随机读/写磁盘对 IO 的消耗？

    redo log 是用来节省随机写磁盘的 IO 消耗，而 change buffer 主要是节省随机读磁盘的 IO 消耗。

    redo log 会把 MySQL 的更新操作先记录到内存中，之后再统一更新到磁盘，而 change buffer 也是把关键查询数据先加载到内存中，以便优化 MySQL 的查询。

17. #### 以下说法错误的是？

    > A.redo log 是 InnoDB 引擎特有的，它的固定大小的
    >
    > B.redo log 日志是不全的，只有最新的一些日志，这和它的内存大小有关
    >
    > C.redo log 可以保证数据库异常重启之后，数据不丢失
    >
    > D.binlog 是 MySQL 自带的日志，它能保证数据库异常重启之后，数据不丢失
    
    答：D
    
    题目解析：binlog 是 MySQL 自带的日志，但它并不能保证数据库异常重启之后数据不丢失。
    
18. #### 以下说法正确的是？

    > A.redo log 日志是追加写的，后面的日志并不会覆盖前面的日志
    >
    > B.binlog 日志是追加写的，后面的日志并不会覆盖前面的日志
    >
    > C.redo log 和 binlog 日志都是追加写的，后面的日志并不会覆盖前面的日志
    >
    > D.以上说法都正确
    
    答：B
    
    题目解析：binlog 日志是追加写的，后面的日志并不会覆盖前面的日志，redo log 日志是固定大小的，后面的日志会覆盖前面的日志。
    
19. #### 有没有办法把 MySQL 的数据恢复到过去某个指定的时间节点？怎么恢复？

    可以恢复，只要你备份了这段时间的所有 binlog，同时做了全量数据库的定期备份。比如，一天一备，或者三天一备，这取决于你们的备份策略。这个时候你就可以把之前备份的数据库先还原到测试库，从备份的时间点开始，将备份的 binlog 依次取出来，重放到你要恢复数据的那个时刻，这个时候就完成了数据到指定节点的恢复。

    比如，今天早上 9 点的时候，你想把数据恢复成今天早上 6:00:00 的状态，这个时候你可以先取出今天凌晨（00:01:59）备份的数据库文件，还原到测试库，再从 binlog 文件中依次取出 00:01:59 之后的操作信息，重放到 6:00:00 这个时刻，这就完成了数据库的还原。