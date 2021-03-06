### **第一部分MySQL架构原理**

**第1节 MySQL体系架构**

![image-20210123000026428](https://tva1.sinaimg.cn/large/008eGmZEly1gmwx0on2r2j310u0ti107.jpg)

**第2节 MySQL运行机制**

![image-20210123000200595](https://tva1.sinaimg.cn/large/008eGmZEly1gmwx2bhm9mj310m0o0tc6.jpg)

**第3节 MySQL存储引擎**

存储引擎在MySQL的体系架构中位于第三层，负责MySQL中的数据的存储和提取，是与文件打交道的 子系统，它是根据MySQL提供的文件访问层抽象接口定制的一种文件访问机制，这种机制就叫作存储引 擎。

使用**show engines**命令，就可以查看当前数据库支持的引擎信息。

**3.1 InnoDB和MyISAM对比**

InnoDB和MyISAM是使用MySQL时最常用的两种引擎类型，我们重点来看下两者区别。

- 事务和外键
   InnoDB支持事务和外键，具有安全性和完整性，适合大量insert或update操作 MyISAM不支持事务和外键，它提供高速存储和检索，适合大量的select查询操作

- 锁机制
   InnoDB支持行级锁，锁定指定记录。基于索引来加锁实现。
   MyISAM支持表级锁，锁定整张表。

-  索引结构

  InnoDB使用聚集索引(聚簇索引)，索引和记录在一起存储，既缓存索引，也缓存记录。 MyISAM使用非聚集索引(非聚簇索引)，索引和记录分开。

- 并发处理能力
     MyISAM使用表锁，会导致写操作并发率低，读之间并不阻塞，读写阻塞。 InnoDB读写阻塞可以与隔离级别有关，可以采用多版本并发控制(MVCC)来支持高并发
     
- 存储文件 

     InnoDB表对应两个文件，一个.frm表结构文件，一个.ibd数据文件。InnoDB表最大支持64TB;

     MyISAM表对应三个文件，一个.frm表结构文件，一个MYD表数据文件，一个.MYI索引文件。从 MySQL5.0开始默认限制是256TB。

- 适用场景

     **MyISAM**

     - 不需要事务支持(不支持) 
     - 并发相对较低(锁定机制问题) 
     - 数据修改相对较少，以读为主 
     - 数据一致性要求不高

     **InnoDB**

     - 需要事务支持(具有较好的事务特性)
     - 行级锁定对高并发有很好的适应能力
     - 数据更新较为频繁的场景
     - 数据一致性要求较高
     - 硬件设备内存较大，可以利用InnoDB较好的缓存能力来提高内存利用率，减少磁盘IO
     
-  总结
    两种引擎该如何选择?
    
    - 是否需要事务?有，InnoDB 
    - 是否存在并发修改?有，InnoDB 
    - 是否追求快速查询，且数据修改少?是，MyISAM 
    - 在绝大多数情况下，推荐使用InnoDB

**3.2 InnoDB存储结构**

从MySQL 5.5版本开始默认使用InnoDB作为引擎，它擅长处理事务，具有自动崩溃恢复的特性，在日

常开发中使用非常广泛。下面是官方的InnoDB引擎架构图，主要分为内存结构和磁盘结构两大部分。

![image-20210123000933217](https://tva1.sinaimg.cn/large/008eGmZEly1gmwxa6amzjj312d0u017h.jpg)

