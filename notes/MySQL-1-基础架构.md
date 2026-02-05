[[MySQL]]
## 总体架构

![img](MySQL-1-%E5%9F%BA%E7%A1%80%E6%9E%B6%E6%9E%84.assets/0d2070e8f84c4801adbfa03bda1f98d9.png)

Server层包括连接器、查询缓存、分析器、优化器、执行器等，涵盖MySQL的大多数核心服务功能，以及所有的内置函数（如日期、时间、数学和加密函数等），所有跨存储引擎的功能都在这一层实现，比如存储过程、触发器、视图等。

而存储引擎层负责数据的存储和提取。

- 连接器：负责跟客户端建立连接、获取权限、维持和管理连接。
- 查询缓存：查询缓存的失效非常频繁，只要有对一个表的更新，这个表上所有的查询缓存都会被清空。**不建议使用**

- 分析器：“词法分析”+““语法分析””

- 优化器：在表里面有多个索引的时候，决定使用哪个索引；或者在一个语句有多表关联（join）的时候，决定各个表的连接顺序。
- 执行器：执行语句
  - 预处理阶段：检查表或字段是否存在；将 `select *` 中的 `*` 符号扩展为表上的所有列。
  - 优化阶段：基于查询成本的考虑， 选择查询成本最小的执行计划。
  - 执行阶段：根据执行计划执行 SQL 查询语句，从存储引擎读取记录，返回给客户端。


![查询语句执行流程](MySQL-1-%E5%9F%BA%E7%A1%80%E6%9E%B6%E6%9E%84.assets/mysql%E6%9F%A5%E8%AF%A2%E6%B5%81%E7%A8%8B.png)

## MySQL记录的存储

### MySQL 的数据存放在哪个文件？

MySQL 数据库的文件存放在哪个目录？

![image-20250520122907317](MySQL-1-%E5%9F%BA%E7%A1%80%E6%9E%B6%E6%9E%84.assets/image-20250520122907317.png)



每创建一个 database（数据库）都会在 /var/lib/mysql/ 目录里面创建一个以 database 为名的目录，然后保存表结构和表数据的文件都会存放在这个目录里。

比如，我这里有一个名为 my_test 的 database，该 database 里有一张名为 t_order 数据库表。

![img](MySQL-1-%E5%9F%BA%E7%A1%80%E6%9E%B6%E6%9E%84.assets/database.png)

```shell
[root@xiaolin ~]#ls /var/lib/mysql/my_test
db.opt  
t_order.frm  
t_order.ibd 
```

这三个文件分别代表着：

- db.opt，用来存储当前数据库的默认字符集和字符校验规则。
- t_order.frm，t_order 的**表结构**会保存在这个文件。在 MySQL 中建立一张表都会生成一个.frm 文件，该文件是用来保存每个表的元数据信息的，主要包含表结构定义。
- t_order.ibd，t_order 的**表数据**会保存在这个文件。表数据既可以存在共享表空间文件（文件名：ibdata1）里，也可以存放在独占表空间文件（文件名：表名字.ibd）。(默认是放在独占表空间)

好了，现在我们知道了一张数据库表的数据是保存在「表名字.ibd」的文件里的，这个文件也称为独占表空间文件。

### 表空间文件的结构

**表空间由段（segment）、区（extent）、页（page）、行（row）组成**，InnoDB 存储引擎的逻辑存储结构大致如下图：

![img](MySQL-1-%E5%9F%BA%E7%A1%80%E6%9E%B6%E6%9E%84.assets/%E8%A1%A8%E7%A9%BA%E9%97%B4%E7%BB%93%E6%9E%84.drawio.png)

1. 行：数据行
2. 页：页是 InnoDB 存储引擎磁盘管理的最小单元，默认每个页的大小为 16KB，一次读取按照页为单位。
3. 区：让链表中相邻的页的物理位置也相邻。

> **在表中数据量大的时候，为某个索引分配空间的时候就不再按照页为单位分配了，而是按照区（extent）为单位分配。每个区的大小为 1MB，对于  16KB 的页来说，连续的 64 个页会被划为一个区，这样就使得链表中相邻的页的物理位置也相邻，就能使用顺序 I/O 了**。

4. 段：表空间是由各个段（segment）组成的，段是由多个区（extent）组成的。

   - 索引段：存放 B + 树的非叶子节点的区的集合；
   - 数据段：存放 B + 树的叶子节点的区的集合；
   - 回滚段：存放的是回滚数据的区的集合

   

### InnoDB 行格式

行格式（row_format），就是一条记录的存储结构。

- Compact ：一种紧凑的行格式，让一个数据页中可以存放更多的行记录
- Dynamic 和 Compressed 两个都是紧凑的行格式，都是基于 Compact 改进一点东西。

#### COMPACT 行格式

![image-20250522095530349](MySQL-1-%E5%9F%BA%E7%A1%80%E6%9E%B6%E6%9E%84.assets/image-20250522095530349.png)

![image-20250522095602947](MySQL-1-%E5%9F%BA%E7%A1%80%E6%9E%B6%E6%9E%84.assets/image-20250522095602947.png)

![image-20250520124020186](MySQL-1-%E5%9F%BA%E7%A1%80%E6%9E%B6%E6%9E%84.assets/image-20250520124020186.png)

##### 1. 变长字段长度列表-

```sql
CREATE TABLE `t_user` (
  `id` int(11) NOT NULL,
  `name` VARCHAR(20) DEFAULT NULL,
  `phone` VARCHAR(20) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB DEFAULT CHARACTER SET = ascii ROW_FORMAT = COMPACT;
```



![image-20250520124157182](MySQL-1-%E5%9F%BA%E7%A1%80%E6%9E%B6%E6%9E%84.assets/image-20250520124157182.png)

- name 列的值为 a，真实数据占用的字节数是 1 字节，十六进制 0x01；
- phone 列的值为 123，真实数据占用的字节数是 3 字节，十六进制 0x03；

这些变长字段的真实数据占用的字节数会按照列的顺序**逆序存放**。

![img](MySQL-1-%E5%9F%BA%E7%A1%80%E6%9E%B6%E6%9E%84.assets/%E5%8F%98%E9%95%BF%E5%AD%97%E6%AE%B5%E9%95%BF%E5%BA%A6%E5%88%97%E8%A1%A81.png)

![image-20250520124438270](MySQL-1-%E5%9F%BA%E7%A1%80%E6%9E%B6%E6%9E%84.assets/image-20250520124438270.png)

![img](MySQL-1-%E5%9F%BA%E7%A1%80%E6%9E%B6%E6%9E%84.assets/%E5%8F%98%E9%95%BF%E5%AD%97%E6%AE%B5%E9%95%BF%E5%BA%A6%E5%88%97%E8%A1%A83.png)

最小行、最大行

### InnoDB 页

#### 页头

![image-20250522095308406](MySQL-1-%E5%9F%BA%E7%A1%80%E6%9E%B6%E6%9E%84.assets/image-20250522095308406.png)

#### 页尾

![image-20250522095352804](MySQL-1-%E5%9F%BA%E7%A1%80%E6%9E%B6%E6%9E%84.assets/image-20250522095352804.png)
