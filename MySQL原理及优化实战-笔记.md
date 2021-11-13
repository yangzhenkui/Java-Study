

本笔记基于 B 站上"千锋教育2021新版MySQL数据库高级教程" 视频教程整理，主要是课程的所提供的课件以及自己听课时的一些补充内容。

视频地址为：[千锋教育2021新版MySQL数据库高级教程](https://www.bilibili.com/video/BV1h64y1y77i?spm_id_from=333.999.0.0)

课堂内容个人讲得是十分不错的，受益很深，面试过很多大厂，之前困惑的问题都在本视频中得到了解决。简而言之，我认为课堂的内容对于大厂面试以及日后的工作都是十分有益的。

要是有什么需要也可以联系我，微信:B612_2021

# 一、索引的概述

## 1.为什么要使用索引

在海量数据中进行查询某条记录的场景是经常发生的，那么如何提升查询性能，就跟要查询的数据字段是否有索引有关系。如果字段加了索引，那么查询的性能就非常快！——就是为了快！

- 索引为什么快？  基于特定的数据结构
- 索引到底是什么？   索引是一种数据结构
- 在使用索引的是要注意什么样的事项？

### **批量生成数据的程序**

```java
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.InputStream;
import java.io.OutputStream;
import java.io.RandomAccessFile;
import java.net.HttpURLConnection;
import java.net.URL;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.Timestamp;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

public class UserUtil {
   private static void createUser(int count) throws Exception {
      List<User> users = new ArrayList<>(count);
      //生成数据，并添加进list中
      for (int i = 0; i < count; i++) {
         User user = new User();
         user.setId(0L + i);
         user.setName("user" + i);
         users.add(user);
      }
      System.out.println("create user");
	  //插入数据库
      Connection conn = getConn();
      String sql = "insert into t_user(name, id) values(?, ?)";
      PreparedStatement pstmt = conn.prepareStatement(sql);
      for (int i = 0; i < users.size(); i++) {
         User user = users.get(i);
         pstmt.setString(1, user.getName());
         pstmt.setLong(2, user.getId());
         pstmt.addBatch();
      }
      pstmt.executeBatch();
      pstmt.close();
      conn.close();
   }

   private static Connection getConn() throws Exception {
      //注意配置数据库链接的相关参数
      String url = "jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai";
      String username = "root";  //注意改成自己的用户名
      String password = "root";  //注意改成自己的mysql密码
      //mysql版本是8.xx以上的为com.mysql.cj.jdbc.Driver，5.xxx的为com.mysql.jdbc.Driver
      String driver = "com.mysql.cj.jdbc.Driver";
      Class.forName(driver);
      return DriverManager.getConnection(url, username, password);
   }
   public static void main(String[] args) throws Exception {
      createUser(500000);   //表示生成500000个user
   }
}
```





## 2.索引是什么

查字典的方式？“数"->shu -- 通过目录来查，能够快速的定位到目标数据所在的页码。 主要是减少IO的次数

没有使用索引的时候，数据的查询需要进行多次IO读写，这样的性能较差——==全表扫描==的过程。

![image-20210517091842385](http://cdn.nefu-yzk.top/img/image-20210517091842385.png)



为数据库的某个字段创建索引，相当是为这个字段的内容创建了一个目录(name+location)。通过这个目录可以快速的实现数据的定位，也就是通过索引能够快速的找到某条数据所在磁盘的位置。

为某个字段创建索引相当于创建了一个目录，目录的内容为字段值+这条记录在磁盘中的内存地址。

![image-20210517092445135](http://cdn.nefu-yzk.top/img/image-20210517092445135.png)

现在的疑问？

- 索引存放位置    inoodb 聚簇索引  ibd文件    myisam 非聚簇索引  myi和myd文件
- 索引的分类及如何创建  索引分为主键索引、普通索引、唯一索引、联合索引、全文索引
- 索引使用了哪种数据结构：各种数据结构的查询性能进行分析     B+树



## 3.索引存放的位置

对于mac系统在/usr/local/mysql文件夹中，对于win系统c:/programdata/mysql(隐藏文件夹)

- InnoDB存储引擎的表：将索引和数据存放在同一个文件里。（==为什么？有什么优势？==）*.ibd    索引+数据

- MyISAM存储引擎的表：索引和数据分开两个文件来存储。索引：*.MYI ; .myi(i->index)     数据：MYD *.myd(d->data)

  聚簇索引和非聚簇索引   聚簇：数据+索引在一个文件.ibd    非聚簇：索引在.myi文件中，数据在.myd文件中



## 4.索引的分类

- 主键索引：主键自带索引效果，也就意味着通过主键来查询表中的记录，性能是非常好的。
- 普通索引：为普通列创建的索引。  
- ==索引分类：主键索引、普通索引、唯一索引、联合索引、全文索引==

创建索引的命令：

```sql
# 格式
create index 索引名称 on 表名(列名)
# 例子
create index idx_name on employees(name)
```

- 唯一索引：就像是唯一列，列中的数据是唯一的。比普通索引的性能要好。

```sql
# 格式
create unique index 索引名称 on 表名(列名)
# 例子
create unique index idx_unique_name on employees(name)
```

- 联合索引（组合索引）：一次性为表中的多个字段一起创建索引，最左前缀法则（如何命中联合索引中的索引列）。注意：一个联合索引建议不要超过5个列

```sql
# 格式
create index 索引名称 on 表(列1,列2,列3)
# 例子
create index idx_name_age_position on employees(name,age,position)
```

- 全文索引

进行查询的时候，==数据源可能来自于不同的字段或者不同的表==。比如去百度中查询数据，千锋教育，来自于网页的标题或者网页的内容。==MyISAM存储引擎支持全文索引==。在实际生产环境中，并不会使用MySQL提供的MyISAM存储引擎的全文索引功能来是实现全文查找。而是会使用第三方的搜索引擎中间件比如ElasticSearch（多）、Solr。==InnoDB在1.2x版本后支持全文索引==

# 二、索引使用的数据结构

使用索引查找数据性能很快，避免了全表扫描的多次磁盘IO读写。但是我们发现，使用索引实际上也需要在索引中查找数据，而且数据量是一样的，那么凭什么索引就能快呢？这就跟索引使用了哪种数据结构支持快速查找。

什么叫数据结构：存放数据的结构。比如：数组、链表、栈、堆、队列等等这些概念。

## 1.线性表：

线性的维护数据的顺序。

对于线性表来说，有两种数据结构来支撑：

- 线性顺序表：相邻两个数据的逻辑关系和物理位置是相同的。

- 线性链式表：相邻两个数据的逻辑关系和物理存放位置没有关系。数据是有先后的逻辑关系，但是数据的物理存储位置并不连续。

  - 单向链表：能够通过当前结点找到下一个节点的位置，以此来维护链表的逻辑关系

    结点结构： 数据内容+下一个数据的指针

  - 双向链表：能够通过当前结点找到上一个或下一个节点的位置，双向都可找。

    结点结构：  上一个数据的指针+数据内容+下一个数据的指针

顺序表和链式表的区别：

	- 数组：进行数据的查询性能（可以通过数组的索引/下标） ：时间复杂度（比较次数）/空间复杂度（算法需要使用多少个变量空间）

​              数组的查询性能非常好： 时间复杂度是O(1)。

​              数组的增删性能是非常差的：

    - 链表：查询的性能是非常差的： 时间复杂度是O(n)。

​                  增删性能是非常好的：



## 2.栈、队列、串、广义表

- 栈：先进后出，有顺序栈、链式栈
- 队列：先进先出，有顺序队列、链式队列
- 串：String 定长串、StringBuffer/Stringbuilder动态串
- 广义表：更加灵活的多维数组，可以在不同的元素中创建不同的维度的数组。



## 3.树

查找树的查找性能是明显比线性表的性能要好，那么接下来我们就要学习这么几种树：

### 1）多叉树

非二叉树

### 2）二叉树

 一个结点最多只能有2个子结点，可以是0、1、2子结点。

### 3）二叉查找树

二叉查找树的查找性能是ok的，查询性能跟树的高度有关，树的高度又跟你插入数据的顺序有关系。特点：二叉树的根结点的数值是比所有左子树的结点的数值大，比右子树的几点的数值小。这样的规律同样满足于他的所有子树。

![](http://cdn.nefu-yzk.top/img/image-20210517111331876.png)

### 4）平衡二叉树（理想概念的树）

我们知道二叉查找树不能非常智能的维护树的高度，因此二叉查找树在某些情况下查询性能是不ok的，此时平衡二叉树就出现了。

- 特点： 平衡二叉树中的树及其所有子树都应满足：左子树和右子树的深度差不能超过1

如果平衡二叉树不满足这个特点，那么平衡二叉树要进行自己旋转，如何自己旋转：

   左旋、右旋、双向（先左后右、先右后左）

​    [平衡二叉树的介绍](https://blog.csdn.net/isunbin/article/details/81707606)

### 5）红黑树（平衡二叉树的一种体现）

平衡二叉树为了维护树的平衡，在一旦不满足平衡的情况就要进行自旋，但是自旋会造成一定的系统开销。因此红黑树在自旋造成的系统开销和减少查询次数之间做了权衡。==因此红黑树有时候并不是一颗平衡二叉树==。红黑树不是严格意义上的平衡二叉树

![](http://cdn.nefu-yzk.top/img/image-20210517143114665.png)

红黑树已经是在查询性能上得到了优化，但索引依然没有使用红黑树作为数据结构来存储数据，因为红黑树在每一层上存放的数据内容是有限的，导致数据量一大，树的深度就变得非常大，于是查询性能非常差。因此索引没有使用红黑树。因为数据量大时，树的高度会非常高，查询次数为变多。O(logN)

不选择红黑树作为索引存储结构是因为红黑树每一层存储的数据有限，不适合用来存储海量数据。

### 6）B树

B树允许一个结点存放多个数据，每一层存储的数据量变大了。这样可以使更小的树的深度来存放更多的数据。但是，B树的一个结点中到底能存放多少个数据，决定了树的深度。

![image-20210517144044898](http://cdn.nefu-yzk.top/img/image-20210517144044898.png)

通过数值计算，B树的一个结点(16KB)最多只能存放15个数据，因此B树依然不能满足海量数据的查询性能优化。    ==每个节点都存储了数据==

![image-20210517144848683](http://cdn.nefu-yzk.top/img/image-20210517144848683.png)

### 7）B+树

B+树叶子节点存储数据，非叶子节点存储值，这样一来，每个节点存储的主键值就很多了

非叶子节点只存储键，不存储数据，只有叶子节点存储数据

![image-20210517145117901](http://cdn.nefu-yzk.top/img/image-20210517145117901.png)

- B+树的特点：
  - 非叶子结点冗余了叶子结点中的键。  非叶子节点只存储索引键和指针域，叶子节点存储数据和索引键
  - 叶子结点是从小到大、从左到右排列的
  - 叶子结点之间提供了指针，提高了区间访问的性能
  - 只有叶子结点存放数据，非叶子结点是不存放数据的，只存放键
  
  ==非叶子节点存储键和指针域，大小为8B+6B=14B，一个节点的默认大小为16KB，所以一个节点可以存储16KB/14B=1170个索引键，提高了海量存储的性能==

![image-20210517151054705](http://cdn.nefu-yzk.top/img/image-20210517151054705.png)





### 8）哈希表

使用哈希表来存取数据的性能是最快的，O(1)，但是不支持范围查找（区间访问）

# 三、InnoDB和MyISAM的区别

InnoDB和MyISAM都是数据库表的存储引擎。那么在互联网公司，或者追求查询性能的场景下，都会使用InnoDB(ibd)作为表的存储引擎。   innoDB->聚簇索引

为什么？InnoDB使用聚簇索引，减少IO次数

## 1.InnoDB引擎——聚集索引

把索引和数据存放在一个文件中，通过找到索引后就能直接在索引树上的叶子结点中获得完整的数据。

可以实现行锁/表锁(表锁粒度有些大，一般不用)

![image-20210517152159059](http://cdn.nefu-yzk.top/img/image-20210517152159059.png)



## 2.MyISAM存储引擎——非聚集索引

把索引和数据存放在两个文件中，查找到索引后还要去另一个文件中找数据，性能会慢一些。  索引在myi文件中  数据在myd文件中

除此之外，MyISAM天然支持表锁，而且支持全文索引。  追求效率则选择InnoDB引擎

![image-20211102142901077](http://cdn.nefu-yzk.top/img/image-20211102142901077.png)

# 四、索引常见的面试题

## 1.问题一：为什么非主键索引的叶子节点存放的数据是主键值

![image-20210517155634440](http://cdn.nefu-yzk.top/img/image-20210517155634440.png)

如果普通索引中不存放主键，而存放完整数据，那么就会造成：

> 一是这样做会造成数据冗余，空间浪费，因为需要一份数据在多个地方存储下来
>
> 二是这样会造成维护麻烦，当对数据进行修改时，需要在多个地方进行修改
>
> 总而言之，相比于提升的查询性能而言，普通索引的叶子节点如果存储数据的话，显然是得不偿失的。

- 数据冗余，虽然提升了查询性能，但是需要更多的空间来存放冗余的数据
- 维护麻烦：一个地方修改数据，需要在多棵索引树上修改。



## 2.问题二：为什么InnoDB表必须创建主键

创建InnoDB表不使用主键能创建成功吗？如果能创建功能，能不能为这张表的普通列创建索引？

如果没有主键，MySQL优化器会给一个虚拟的主键，于是普通索引会使用这个虚拟主键——也会造成性能开销。为了性能考虑，和设计初衷，那么创建表的时候就应该创建主键。



## 3.问题三：为什么使用主键时推荐使用整型的自增主键

### 1）为什么要使用整型：

主键-主键索引树-树里的叶子结点和非叶子结点的键存放的是主键的值，而且这颗树是一个二叉查找树。==数据的存放是有大小顺序的==。

- 整型： 大小顺序是很好比较的
- 字符串：字符串的自然顺序的比较是要进行一次编码成为数值后再进行比较的。（字符串的自然顺序，A Z）

   uuid随机字符串

### 2）为什么要自增：

如果不用自增：  （10  1  6  200  18  29）使用不规律的整数来作为主键，那么主键索引树会使用更多的自旋次数来保证树索引树的叶子节点中的数据是从小到大-从左到右排列，因此性能必然比使用了自增主键的性能要差！



# 五、联合索引和最左前缀法则

## 1.联合索引的特点

在使用一个索引来实现表中多个字段的索引效果。



## 2.联合索引是如何存储的

![](http://cdn.nefu-yzk.top/img/image-20210517161552146.png)

使用联合索引，则两个索引的列值作为索引键的值

## 3.最左前缀法则

==最左==前缀法则是表示一条sql语句在联合索引中有没有走索引（命中索引/不会全表扫描）

```sql
# 创建联合索引
create index idx_a_b_c on table1(a,b,c);
# sql语句有没有命中索引 
select * from table1 where a = 10;   # 命中
select * from table1 where a = 10 and b=20;  # 命中
select * from table1 where a = 10 and b=20 and c=30;  # 命中
select * from table1 where b = 10;  # 未命中
select * from table1 where b = 10 and c=30;  # 未命中
select * from table1 where a = 10 and c=30;  # 只会走a的索引 c未命中索引
select * from table1 where c = 30;  # 未命中
select * from table1 where a = 10 and c = 30 and b = 20;   # (abc全走)=>  mysql有一个内部优化器 会做一次内部优化。
```



# 六、SQL优化

SQL优化的目的是为了SQL语句能够具备优秀的查询性能，实现这样的目的有很多的途径：

- 工程优化如何实现：数据库标准、表的结构标准、字段的标准、创建索引
- SQL语句的优化：当前SQL语句有没有命中索引。

## 1.工程优化如何实现

### **一、基础规范**

- 表存储引擎必须使用 InnoDB

- 表字符集默认使用 utf8，必要时候使用 utf8mb4
  - 通用，无乱码风险，汉字 3 字节，英文 1 字节
  - utf8mb4 是 utf8 的超集，有存储 4 字节例如表情符号时，使用它

- 禁止使用存储过程，视图，触发器，Event(能让mysql闲着，绝对让他闲着)
  - 对数据库性能影响较大，互联网业务，能让站点层和服务层干的事情，不要交到数据库层
  - 调试，排错，迁移都比较困难，扩展性较差

- 禁止在数据库中存储大文件，例如照片，可以将大文件存储在对象存储系统(OSS)，数据库中存储路径

- 禁止在线上环境做数据库压力测试

- 测试，开发，线上数据库环境必须隔离


### **二、命名规范**

- 库名，表名，列名必须用小写，采用下划线分隔t_book tb_book
  - abc，Abc，ABC 都是给自己埋坑
- 库名，表名，列名必须见名知义，长度不要超过 32 字符
  - tmp，wushan 谁 TM 知道这些库是干嘛的
- 库备份必须以 bak 为前缀，以日期为后缀

- 从库必须以 - s 为后缀

- 备库必须以 - ss 为后缀


### **三、表设计规范**

- 单实例表个数必须控制在 2000 个以内

- 单表分表个数必须控制在 1024 个以内

- 表必须有主键，推荐使用 UNSIGNED 整数为主键
  - 潜在坑：删除无主键的表，如果是 row 模式的主从架构，从库会挂住
- 禁止使用外键，如果要保证完整性，应由应用程式实现
  - 解读：外键使得表之间相互耦合，影响 update/delete 等 SQL 性能，有可能造成死锁，高并发情况下容易成为数据库瓶颈
- 建议将大字段，访问频度低的字段拆分到单独的表中存储，==分离冷热数据==(如商品详情)
  - 解读：具体参加《如何实施数据库垂直拆分》

### **四、列设计规范**

- 根据业务区分使用 tinyint/int/bigint，分别会占用 1/4/8 字节

- 根据业务区分使用 char/varchar

  - 字段长度固定，或者长度近似的业务场景，适合使用 char，能够减少碎片，查询性能高
  - 字段长度相差较大，或者更新较少的业务场景，适合使用 varchar，能够减少空间
- 根据业务区分使用 datetime/timestamp
  - 解读：前者占用 5 个字节，后者占用 4 个字节，存储年使用 YEAR，存储日期使用 DATE，存储时间使用 datetime
- 必须把字段定义为 NOT NULL 并设默认值

  - NULL 的列使用索引，索引统计，值都更加复杂，MySQL 更难优化
  - NULL 需要更多的存储空间
  - NULL 只能采用 IS NULL 或者 IS NOT NULL，而在 =/ != /in /not in 时有大坑
- 使用 INT UNSIGNED 存储 IPv4，不要用 char(15)

- 使用 varchar(20) 存储手机号，不要使用整数

  - 牵扯到国家代号，可能出现 +/-/() 等字符，例如 + 86
  - 手机号不会用来做数学运算
  - varchar 可以模糊查询，例如 like ‘138%’
- 使用 TINYINT 来代替 ENUM
  - 解读：ENUM 增加新值要进行 DDL 操作

### **五、索引规范**

- 唯一索引使用 uniq_[字段名] 来命名

- 非唯一索引使用 idx_[字段名] 来命名

- 单张表索引数量建议控制在 5 个以内

  - 互联网高并发业务，太多索引会影响写性能(即是对数据修改时，也需要维护索引树)
  - 生成执行计划时，如果索引太多，会降低性能，并可能导致 MySQL 选择不到最优索引
  - 异常复杂的查询需求，可以选择 ES 等更为适合的方式存储
- 组合索引字段数不建议超过 5 个
  - 解读：如果 5 个字段还不能极大缩小 row 范围，八成是设计有问题
- 不建议在频繁更新的字段上建立索引

- 非必要不要进行 JOIN 查询，如果要进行 JOIN 查询，被 JOIN 的字段必须类型相同，并建立索引
  - 解读：踩过因为 JOIN 字段类型不一致，而导致全表扫描的坑么？
- 理解组合索引最左前缀原则，避免重复建设索引，如果建立了 (a,b,c)，相当于建立了 (a), (a,b), (a,b,c)


### **六、SQL 规范**

- 禁止使用 select *，只获取必要字段(尽量走覆盖索引)
  - select * 会增加 cpu/io / 内存 / 带宽的消耗
  - 指定字段能有效利用索引覆盖
  - 指定字段查询，在表结构变更时，能保证对应用程序无影响
- insert 必须指定字段，禁止使用 insert into T values()
  - 解读：指定字段插入，在表结构变更时，能保证对应用程序无影响
- 隐式类型转换会使索引失效，导致全表扫描

- 禁止在 where 条件列使用函数或者表达式
  - 解读：导致不能命中索引，全表扫描
- 禁止负向查询以及 % 开头的模糊查询
  - 解读：导致不能命中索引，全表扫描
- 禁止大表 JOIN 和子查询

- 同一个字段上的 OR 必须改写问 IN，IN 的值必须少于 50 个

- 应用程序必须捕获 SQL 异常
  - 解读：方便定位线上问题

**说明**：本军规适用于并发量大，数据量大的典型互联网业务，可直接带走参考，不谢。
军规练习：为什么下列 SQL 不能命中 phone 索引？
select uid from user where phone=13811223344

## 2.Explain执行计划——SQL优化神器

得知道当前系统里有哪些SQL是慢SQL，查询性能超过1s的sql，然后再通过Explain工具可以对当前SQL语句的性能进行判断——为什么慢，怎么解决。

要想知道哪些SQL是慢SQL，有两种方式，一种是开启本地MySQL的慢查询日志；另一种是阿里云提供的RDS（第三方部署的MySQL服务器），提供了查询慢SQL的功能。

```sql
explain SELECT * from employees where name like "user100%"
```

通过在SQL语句前面加上explain关键字，执行后并不会真正的执行sql语句本身，而是通过explain工具来分析当前这条SQL语句的性能细节：比如是什么样的查询类型、可能用到的索引及实际用到的索引，和一些额外的信息。

![image-20211102152544113](http://cdn.nefu-yzk.top/img/image-20211102152544113.png)

## 3.MySQL的内部优化器

在SQL查询开始之前，MySQL内部优化器会进行一次自我优化，让这一次的查询性能尽可能的好。

当前执行的SQL

```sql
explain select * from tb_book where id=1;
show warnings;  # 主要是显示内部优化器的语句
```

内部优化器优化后的效果：

```sql
/* select#1 */ select '1' AS `id`,'千锋Java厉害' AS `name` from `db_mysql_pro`.`tb_book` where true
# 上面条语句相当于走主键索引，并不会进行全表扫描，即是直接从索引树上获取对应的值
```

## 4.select_type列

5个值：dreived、primary、subquery、simple、union

关闭 MySQL 对衍生表的合并优化：

```sql
set session optimizer_switch='derived_merge=off'; 
```



执行了这样的计划：

```sql
EXPLAIN select (select 1 from tb_author where id=1) from (select * from tb_book where id=1) der;
# select * from tb_book where id=1 会产生一张临时表
```

![image-20210518093837712](http://cdn.nefu-yzk.top/img/image-20210518093837712.png)

- derived：

第一条执行的sql是from后面的子查询，该子查询只要在from后面，就会生成一张衍生表，因此他的查询类型：derived

```sql
select * from tb_book where id=1  # 在from之后的查询
```

- subquery：

在select之后 from之前的子查询

```sql
select 1 from tb_author where id=1  #  在select之后  from之前的子查询
```

- primary：

最外部的select

- simple：

不包含子查询的简单的查询

- union：

使用union进行的联合查询的类型

![image-20211102153848794](http://cdn.nefu-yzk.top/img/image-20211102153848794.png)

## 5.table列

当前查询正在查哪张表



## 6.type列

type值：null、system、const、eq_ref、ref、range、index、all

type列可以直观的判断出当前的sql语句的性能。type里的取值和性能的优劣顺序如下：

```sql
null > system > const > eq_ref > ref > range > index > all
```

对于SQL优化来说，要==尽量保证type列的值是属于range及以上级别。==

- null

性能最好的，一般在使用了==聚合函数(count、min、max、sum、avg)==操作索引列，结果直接从索引树获取即可，因此是性能最好。

- system

很少见。直接和一条记录进行匹配。

- const

使用主键索引或唯一索引和常量进行比较，这种性能非常好

- eq_ref

在进行==多表连接查询==时。如果查询条件是**使用了主键进行比较**，那么当前查询类型是eq_ref

```sql
EXPLAIN select * from tb_book_author left JOIN tb_book on tb_book_author.book_id = tb_book.id
```

- ref

  - 简单查询：EXPLAIN select * from tb_book where name='book1'

  ​      如果查询条件是普通列索引，那么类型ref

  - 复杂查询：EXPLAIN select book_id from tb_book left join tb_book_author on tb_book.id = tb_book_author.book_id

  ​     如果查询条件是普通列索引，那么类型ref

- range:

 使用索引进行范围查找

```sql
explain select * from tb_book where id>1
```

- index

查询没有进行条件判断。但是所有的数据都可以直接从索引树上获取(book表中的所有列都有索引)

```sql
explain select * from tb_book
```

- all

没有走索引，进行了全表扫描

```sql
explain select * from tb_author
```

## 7.id列

在多个select中，id越大越先执行，如果id相同。上面的先执行。



## 8.possible_keys列

这一次的查询可能会用到的索引(并不一定会用该索引)。也就是说mysql内部优化器会进行判断，如果这一次查询走索引的性能比全表扫描的性能要差，那么内部优化器就让此次查询进行全表扫描——这样的判断依据我们可以通过trace工具来查看

```sql
EXPLAIN select * from employees where name like 'custome%'
```

这条sql走索引查询的行数是500多万，那么总的数据行数也就500多万，因此直接进行全表扫描性能更快

![image-20210518103742876](http://cdn.nefu-yzk.top/img/image-20210518103742876.png)



## 9.key列

实际该sql语句使用的索引



## 10.rows列

该sql语句可能要查询的数据条数



## 11.key_len列

键的长度，通过这一列可以让我们知道当前命中了联合索引中的哪几列。

```sql
EXPLAIN select * from employees where name = 'customer10011' # 74
EXPLAIN select * from employees where name = 'customer10011' and age=30 # 74 4 = 78
EXPLAIN select * from employees where name = 'customer10011' and age=30 and position='dev' # 74 4 62 = 140
EXPLAIN select * from employees where name = 'customer10011' and position='dev' # 74
```

name长度是74，也就是当看到key_len是74，表示使用了联合索引中的name列

![image-20210518104705482](http://cdn.nefu-yzk.top/img/image-20210518104705482.png)

计算规则：

```ABAP
- 字符串
1. char(n): n字节长度
2. varchar(n): 2字节存储字符串长度,如果是utf-8,则长度3n + 2

- 数值类型
1. tinyint: 1字节
2. smallint: 2字节
3. int: 4字节
4. bigint: 8字节

- 时间类型
1. date: 3字节
2. timestamp: 4字节
3. datetime: 8字节

如果字段允许为NULL,需要1字节记录是否为NULL
索引最大长度是768字节,当字符串过长时, mysql会做一个类似左前缀索引的处理,将前半部分的字符提取出来做索引
```



## 12.extra列

extra列提供了额外的信息，是能够帮助我们判断当前sql的是否使用了覆盖索引、文件排序、使用了索引进行查询条件等等的信息。

- Using index:使用了覆盖索引

  所谓的覆盖索引，指的是当前查询的所有数据字段都是索引列，这就意味着可以直接从索引列中获取数据，而不需要进行查表。

  ==使用覆盖索引进行性能优化==这种手段是之后sql优化经常要用到的。

```sql
EXPLAIN select book_id,author_id from tb_book_author where book_id = 1 -- 覆盖索引，从索引列获取数据
EXPLAIN select * from tb_book_author where book_id = 1 -- 没有使用覆盖索引 *中包括了没有建立索引的列
```

- using where

  ==使用了普通索引列做查询条件==

```sql
EXPLAIN select * from tb_author where name > 'a'
```

- using index condition

查询结果没有使用覆盖索引，建议可以使用覆盖索引(查询的数据列全部是索引列)来优化

```sql
EXPLAIN select * from tb_book_author where book_id > 1
EXPLAIN select book_id,author_id from tb_book_author where book_id > 1  #  则是using index；using where
```

- Using temporary

在非索引列上进行去重操作就需要使用一张临时表来实现，性能是非常差的。当前name列没有索引

```sql
EXPLAIN select DISTINCT name from tb_author
```

- Using filesort

使用文件排序： 会使用磁盘+内存的方式进行文件排序，会涉及到两个概念：单路排序、双路排序

```sql
EXPLAIN select * from tb_author order by name
```

- Select tables optimized away

直接在索引列上进行聚合函数的操作，没有进行任何的表的操作

```sql
EXPLAIN select min(id) from tb_book
```



## 13、小试牛刀

```sql
EXPLAIN SELECT * FROM employees WHERE name= 'customer30' AND age > 28 AND position ='dev';
#  对于上面条语句   name索引命中   age也是命中的  但是position是未命中的

EXPLAIN SELECT * FROM employees WHERE left(name,3) = 'customer30';
# 不能在索引列上做计算、函数、类型转换，否则索引失效

EXPLAIN select * from employees where date(hire_time) ='2022-04-22';
#转换成范围查找
# 优化之后   range
EXPLAIN select * from employees where hire_time >='2022-04-22 00:00:00' and hire_time <='2022-04-22 23:59:59';

# 尽量使用覆盖索引
EXPLAIN SELECT * FROM employees WHERE name= 'customer30' AND age = 30 AND position ='dev';

#使用不等于(! =或者<>)会导致全表扫描
EXPLAIN SELECT * FROM employees WHERE name != 'customer30'   # 会导致全表扫描，可以在业务层面进行处理，或者使用搜索引擎

# 使用is null、is not null会导致全表扫描
EXPLAIN SELECT * FROM employees WHERE name is null   # 可以在业务层面进行处理，不让其到数据库层面上来

#使用like以通配符开头('%abc...')会导致全表扫描  模糊查询可以在中间件去做
EXPLAIN SELECT * FROM employees WHERE name like '%mer'

#如何解决'%customer%'的查找
#       - 使用覆盖索引
#       - 使用搜索引擎中间件 ES/solr

# 字符串不加单引号会导致全表扫描
EXPLAIN SELECT * FROM employees WHERE name = 1000;  # 全表扫描
EXPLAIN SELECT * FROM employees WHERE name = '1000'; # 非全表扫描

# 少用or或in，MySQL内部优化器可能不走索引
EXPLAIN SELECT * FROM employees WHERE name = 'customer30' or name = 'customer40'

EXPLAIN SELECT * FROM employees WHERE name in(100000条数据)
-- 解决方案：100000进行拆分 一次插1000条，要进行100次的查询——多线程-进行结果的整合countDownLatch

# 范围查询优化,范围过大可能会造成全表扫描
explain select * from employees where age >=1 and age <=2000; -- 2.6s

# 优化为：0.001 *20 = 0.02s
explain select * from employees where age >=1 and age <=1000;
explain select * from employees where age >=1001 and age <=2000;

```



# 七、Trace工具

在执行计划中我们发现有的sql会走索引，有的sql即使明确使用了索引也不会走索引。这是因为mysql的内部优化器任务走索引的性能比不走索引全表扫描的性能要差，因此mysql内部优化器选择了使用全表扫描。依据来自于trace工具的结论。

```sql
set session optimizer_trace="enabled=on", end_markers_in_json=on; -- 开启trace
select * from employees where name > 'a' order by position; -- 执行查询
SELECT * FROM information_schema.OPTIMIZER_TRACE;            -- 获得trace的分析结果
```



```json
{
  "steps": [
    {
      "join_preparation": { -- 阶段1:进入到准备阶段
        "select#": 1,
        "steps": [
          {
            "expanded_query": "/* select#1 */ select `employees`.`id` AS `id`,`employees`.`name` AS `name`,`employees`.`age` AS `age`,`employees`.`position` AS `position`,`employees`.`hire_time` AS `hire_time` from `employees` where (`employees`.`name` > 'a') order by `employees`.`position`"
          }
        ] /* steps */
      } /* join_preparation */
    },
    {
      "join_optimization": { -- 阶段2: 进入到优化阶段
        "select#": 1,
        "steps": [
          {
            "condition_processing": { -- 条件处理
              "condition": "WHERE",
              "original_condition": "(`employees`.`name` > 'a')",
              "steps": [
                {
                  "transformation": "equality_propagation",
                  "resulting_condition": "(`employees`.`name` > 'a')"
                },
                {
                  "transformation": "constant_propagation",
                  "resulting_condition": "(`employees`.`name` > 'a')"
                },
                {
                  "transformation": "trivial_condition_removal",
                  "resulting_condition": "(`employees`.`name` > 'a')"
                }
              ] /* steps */
            } /* condition_processing */
          },
          {
            "substitute_generated_columns": {
            } /* substitute_generated_columns */
          },
          {
            "table_dependencies": [ -- 表依赖详情
              {
                "table": "`employees`",
                "row_may_be_null": false,
                "map_bit": 0,
                "depends_on_map_bits": [
                ] /* depends_on_map_bits */
              }
            ] /* table_dependencies */
          },
          {
            "ref_optimizer_key_uses": [
            ] /* ref_optimizer_key_uses */
          },
          {
            "rows_estimation": [
              {
                "table": "`employees`",
                "range_analysis": {
                  "table_scan": {
                    "rows": 5598397,
                    "cost": 576657
                  } /* table_scan */,   -- 表扫描
                  "potential_range_indexes": [ -- 可能使用到的索引
                    {
                      "index": "PRIMARY", -- 主键索引
                      "usable": false,
                      "cause": "not_applicable"
                    },
                    {
                      "index": "idx_name_age_position", -- 联合索引
                      "usable": true,
                      "key_parts": [
                        "name",
                        "age",
                        "position",
                        "id"
                      ] /* key_parts */
                    },
                    {
                      "index": "idx_hire_time",
                      "usable": false,
                      "cause": "not_applicable"
                    }
                  ] /* potential_range_indexes */,
                  "setup_range_conditions": [
                  ] /* setup_range_conditions */,
                  "group_index_range": {
                    "chosen": false,
                    "cause": "not_group_by_or_distinct"
                  } /* group_index_range */,
                  "skip_scan_range": {
                    "potential_skip_scan_indexes": [
                      {
                        "index": "idx_name_age_position",
                        "usable": false,
                        "cause": "query_references_nonkey_column"
                      }
                    ] /* potential_skip_scan_indexes */
                  } /* skip_scan_range */,
                  "analyzing_range_alternatives": { -- 分析各个索引使用的成本
                    "range_scan_alternatives": [
                      {
                        "index": "idx_name_age_position",
                        "ranges": [
                          "a < name"
                        ] /* ranges */,
                        "index_dives_for_eq_ranges": true,
                        "rowid_ordered": false,
                        "using_mrr": true,
                        "index_only": false, -- 是否使用了覆盖索引
                        "rows": 2799198, -- 要扫描的行数
                        "cost": 2.08e6, -- 要花费的时间
                        "chosen": false, -- 是否选择使用这个索引
                        "cause": "cost" -- 不选择的原因：开销比较大
                      }
                    ] /* range_scan_alternatives */,
                    "analyzing_roworder_intersect": {
                      "usable": false,
                      "cause": "too_few_roworder_scans"
                    } /* analyzing_roworder_intersect */
                  } /* analyzing_range_alternatives */
                } /* range_analysis */
              }
            ] /* rows_estimation */
          },
          {
            "considered_execution_plans": [   -- 可以被考虑的执行计划
              {
                "plan_prefix": [
                ] /* plan_prefix */,
                "table": "`employees`",
                "best_access_path": { -- 最优访问路径
                  "considered_access_paths": [ -- 最后选择的访问路径
                    {
                      "rows_to_scan": 5598397, -- 全表扫描的行数
                      "access_type": "scan", -- 全表扫描
                      "resulting_rows": 5.6e6, -- 结果的行数
                      "cost": 576655, -- 花费的时间
                      "chosen": true, -- 选择这种方式
                      "use_tmp_table": true
                    }
                  ] /* considered_access_paths */
                } /* best_access_path */,
                "condition_filtering_pct": 100,
                "rows_for_plan": 5.6e6,
                "cost_for_plan": 576655,
                "sort_cost": 5.6e6,
                "new_cost_for_plan": 6.18e6,
                "chosen": true
              }
            ] /* considered_execution_plans */
          },
          {
            "attaching_conditions_to_tables": {
              "original_condition": "(`employees`.`name` > 'a')",
              "attached_conditions_computation": [
              ] /* attached_conditions_computation */,
              "attached_conditions_summary": [
                {
                  "table": "`employees`",
                  "attached": "(`employees`.`name` > 'a')"
                }
              ] /* attached_conditions_summary */
            } /* attaching_conditions_to_tables */
          },
          {
            "optimizing_distinct_group_by_order_by": {
              "simplifying_order_by": {
                "original_clause": "`employees`.`position`",
                "items": [
                  {
                    "item": "`employees`.`position`"
                  }
                ] /* items */,
                "resulting_clause_is_simple": true,
                "resulting_clause": "`employees`.`position`"
              } /* simplifying_order_by */
            } /* optimizing_distinct_group_by_order_by */
          },
          {
            "reconsidering_access_paths_for_index_ordering": {
              "clause": "ORDER BY",
              "steps": [
              ] /* steps */,
              "index_order_summary": {
                "table": "`employees`",
                "index_provides_order": false,
                "order_direction": "undefined",
                "index": "unknown",
                "plan_changed": false
              } /* index_order_summary */
            } /* reconsidering_access_paths_for_index_ordering */
          },
          {
            "finalizing_table_conditions": [
              {
                "table": "`employees`",
                "original_table_condition": "(`employees`.`name` > 'a')",
                "final_table_condition   ": "(`employees`.`name` > 'a')"
              }
            ] /* finalizing_table_conditions */
          },
          {
            "refine_plan": [
              {
                "table": "`employees`"
              }
            ] /* refine_plan */
          },
          {
            "considering_tmp_tables": [
              {
                "adding_sort_to_table": "employees"
              } /* filesort */
            ] /* considering_tmp_tables */
          }
        ] /* steps */
      } /* join_optimization */
    },
    {
      "join_execution": {
        "select#": 1,
        "steps": [
          {
            "sorting_table": "employees",
            "filesort_information": [
              {
                "direction": "asc",
                "expression": "`employees`.`position`"
              }
            ] /* filesort_information */,
            "filesort_priority_queue_optimization": {
              "usable": false,
              "cause": "not applicable (no LIMIT)"
            } /* filesort_priority_queue_optimization */,
            "filesort_execution": [
            ] /* filesort_execution */,
            "filesort_summary": {
              "memory_available": 262144,
              "key_size": 40,
              "row_size": 190,
              "max_rows_per_buffer": 1379,
              "num_rows_estimate": 5598397,
              "num_rows_found": 5913852,
              "num_initial_chunks_spilled_to_disk": 1954,
              "peak_memory_used": 262144,
              "sort_algorithm": "std::stable_sort",
              "sort_mode": "<fixed_sort_key, packed_additional_fields>"
            } /* filesort_summary */
          }
        ] /* steps */
      } /* join_execution */
    }
  ] /* steps */
}
```



# 八、SQL优化实战

## 1.order by优化

如果排序会造成文件排序(在磁盘中完成排序，这样性能会比较差)，说明sql没有命中索引，可以==使用最左前缀法则，让排序遵循最左前缀法则，避免文件排序==

在排序应用场景中，很容易出现文件排序(在磁盘中完成排序，这样性能会比较差)，sql没有命中索引的问题，文件排序会对性能造成影响，因此需要优化

```sql
# 文件排序即是指将数据单独拿到一个文件中，进行排序的过程，这样显然会比直接在索引树上排序性能更低
# using filesort
Explain select * from employees where name='customer' order by position;
# 没有使用文件排序
Explain select * from employees where name='customer' order by age, position;    # , 表示先按照age来排，age一样是再安装position来排
# 不满足最左前缀法则，使用了文件排序
Explain select * from employees where name='customer' order by position, age;
# 满足最左前缀法则，使用索引排序
Explain select * from employees where name='customer' and age=20 order by position, age;  # age 直接忽略了
show WARNINGS;
/* select#1 */ select `db_mysql_pro`.`employees`.`id` AS `id`,`db_mysql_pro`.`employees`.`name` AS `name`,`db_mysql_pro`.`employees`.`age` AS `age`,`db_mysql_pro`.`employees`.`position` AS `position`,`db_mysql_pro`.`employees`.`hire_time` AS `hire_time` from `db_mysql_pro`.`employees` where ((`db_mysql_pro`.`employees`.`age` = 20) and (`db_mysql_pro`.`employees`.`name` = 'customer')) order by `db_mysql_pro`.`employees`.`position`
# 排序方向不同，没有使用索引排序
Explain select * from employees where name='customer' and age=20 order by age, position desc;
# 使用范围查询，使用了文件排序
Explain select * from employees where name in ('customer','aa') order by age, position;
# 使用范围查询，使用了文件排序
Explain select * from employees where name > 'a' order by name;
```

优化手段：

- 如果排序的字段创建了联合索引，那么尽量在业务不冲突的情况下，遵循最左前缀法则来写排序语句。
- 如果文件排序没办法避免，那么尽量想办法使用覆盖索引。all->index



## 2.group by优化

group by 的原理是先排序后分组，因此对于group by 的优化参考order by



## 3.文件排序的原理

在执行文件排序的时候，会把查询的数据的大小与系统变量：max_length_for_sort_data的大小进行比较（默认是1024字节）,如果比系统变量小，那么执行单路排序，反之则执行双路排序

即查询到的数据量小的时候走单路排序，数据量大时走双路排序

-  单路排序 

​       把所有的数据扔到sort_buffer内存缓冲区中，进行排序，然后结束

- 双路排序   将排序字段和主键字段在内存缓冲区进行排序，然后做一次==回表查询==

  取数据的排序字段和主键字段，在内存缓冲区中排序完成后，将主键字段做一次回表查询，获取完整数据。



![image-20210518144357614](http://cdn.nefu-yzk.top/img/image-20210518144357614.png)



回表查询之后再根据id进行排序

## 4.分页优化

对于这样的优化查询，mysql会把全部的10010数据拿到，并舍弃掉前面的10000条

```sql
-- 一次行获取10010，再舍弃掉前10000条
Explain select * from employees limit 10000,10
```

 如果在主键连续的情况下，可以使用主键来做条件，但是这种情况是很少见的

```sql
Explain select * from employees where id>100000 limit 10
```

对于主键不连续情况下的例子：

```sql
Explain select * from employees order by name limit 1000000,10  

-- 通过先进行覆盖索引的查找，然后在使用join做连接查询获取所有数据。这样比全表扫描要快  使用覆盖索引可以提高效率   join：小表驱动大表
explain select * from employees a inner join (select id from employees order by name limit 1000000,10) b on a.id = b.id;  
select * from t_user limit 5000000, 10;  # 1.512s
select * from t_user a join (select id from t_user limit 5000000, 10) b on a.id = b.id; # 1.212s
```



## 5.join优化

```sql
Explain select * from t1 inner join t2 on t1.a = t2.a
#说明：
#  - t1  有1万行记录（大表）
#  - t2  只有100行记录（小表）
# 小表全表扫描，大表ref  NLJ算法
#在join查询中，如果关联字段建立了索引(前提)，mysql就会使用nlj算法，去找小表（数据量比较小的表）作为驱动表，先从驱动表中读一行数据，然后拿这一行数据去被驱动表（数据量比较大的表）中做查询。这样的大表和小表是由mysql内部优化器来决定的，跟sql语句中表的书写顺序无关。——NLJ算法    NLJ：nested loop join 嵌套循环join

Explain select * from t1 inner join t2 on t1.b = t2.b

# 如果没有索引，会创建一个join buffer 内存缓冲区，把小表数据存进来（为什么不存大表，因为缓冲区大小限制，及存数据消耗性能的考虑），用内存缓冲区中100行记录去和大表中的1万行记录进行比较，比较的过程依然是在内存中进行的。索引join buffer起到了提高join效率的效果。——BNLJ算法

# 结论： 如果使用join查询，那么join的两个表的关联字段一定要创建索引，而且字段的长度 类型一定是要一致的（在建表时就要做好），否则索引会失效，会使用BNLJ算法，全表扫描的效果。
#  使用join查询，关联字段一定要建立索引

#nested loop join:嵌套循环join
#block nested loop join ： 块嵌套循环join

```

在join中会涉及到大表（数据量大）和小表（数据量小）的概念。MySQL内部优化器会根据关联字段是否创建了索引来使用不同的算法：

- NLJ(嵌套循环算法)：如果关联字段使用了索引，mysql会对小表做全表扫描，用小表的数据去和大表的数据去做索引字段的关联查询（type：ref）  无内存缓冲区
- BNLJ（块嵌套循环算法）：如果关联字段没有使用索引，mysql会提供一个join buffer缓冲区，先把小表放到缓冲区中，然后全表扫描大表，把大表的数据和缓冲区中的小表数据在内存中进行匹配。 小表中的数据放入内存缓冲区中，然后每次获取一条记录与大表中的记录进行比较(主要是会有缓冲区)

结论：使用join查询时，一定要建立关联字段的索引，且两张表的关联字段在设计之初就要做到字段类型、长度是一致的，否则索引失效。

![image-20211102201309096](http://cdn.nefu-yzk.top/img/image-20211102201309096.png)

## 6.in和exists优化

在sql中如果A表是大表，B表是小表，那么使用in会更加合适。反之应该使用exists。

in(放小表)             exists(放大表)

- in: B的数据量<A的数据量

```sql
select * from A where id in (select id from B) 
# 相当于：
for(select id from B){ //B的数据量少，所以循环次数少。

   select * from A where A.id = B.id

}
```

- exists:  B的数据量>A的数据量 (10: id 1. 2. 3. 4)

```sql
select * from A where exists (select 1 from B where B.id = A.id)   # 1 主要是不关心返回的是什么类型的值，主要是看有无记录 true / false
# 等价于
for(select * from A){
   select * from B where B.id = A.id
}
```



## 7.count优化

对于count的优化应该是架构层面的优化，因为count的统计是在一个产品会经常出现，而且每个用户访问，所以对于访问频率过高的数据建议维护在缓存中。 null不记录在内

![image-20210518155019048](http://cdn.nefu-yzk.top/img/image-20210518155019048.png)



## 8.复杂SQL语句的优化

### 场景一：

```xml
 <select id="getProjectByVersionProject" resultType="com.qf.entity.dto.ProjectVersionDto">
        select dpp.guid projectGuid,
        dpp.package_number packageNumber,
        dpp.name projectName,
        dpp.app_version_no versionNo,
        dpp.creation_time createTime
        FROM qf_device_package_project dpp left JOIN (
     <!--查询的时候，尽量保证索引查询-->
        SELECT manager_id, PROJECT_GUID FROM csm_manager_custom_project
        UNION
     <!--join的关联的字段需要建立索引-->
        SELECT manager_id, PROJECT_GUID FROM csm_manager_region_project_rela
        ) dp on dp.PROJECT_GUID = dpp.guid
        WHERE dpp.state = 0 and dp.manager_id = #{query.managerId}
        <if test="query.projectName !=null and query.projectName !=''">
            and dpp.name like CONCAT('%',#{query.projectName},'%')
        </if>
        <if test="query.versionNo != null and query.versionNo !=''">
            and dpp.app_version_no = #{query.versionNo}
        </if>
        <if test="pagination.beginIndex !=null">
            <!--分页优化可以使用分次查询或者先进行覆盖索引查询，然后进行嵌套-->
            limit #{pagination.beginIndex},#{pagination.length}
        </if>
    </select>
```



### 场景二：

```xml
<select id="selectAfterSaleManagementsByIds" resultType="com.qf.entity.AfterSaleManagement">
        SELECT
        uasm.ID, DEVICE_KEY, PHOTO_URL, STATE, COMPANY_NAME,NAME, PHONE,uc.CONTENT as PROBLEM_TYPE_GUID,
        VERSION_NO,
        PROBLEM, SHIPPING_METHOD,EXPRESS_NAME,EXPRESS_NUMBER, REPAIR_PERSONNEL, REPAIR_LOG, FOLLOW_UP_FEEDBACK,
        REMARKS,REFUSAL,PRODUCT_MODEL,MEASUREMENT_PROBLEM,BIT_NUMBER,RECEIPT_DATE,
        uasm.CREATE_TIME,uasm.MODIFY_TIME
        FROM
        qf_after_sale_management uasm LEFT JOIN
        qf_constant uc
        on uasm.PROBLEM_TYPE_GUID = uc.GUID
        WHERE uasm.ID IN
    	<!--in中放小表的查询数据， join关联的字段建立索引-->
        <foreach item="afterSaleManagementId" index="index" collection="afterSaleManagementIds" open="("
                 separator="," close=")">
            #{afterSaleManagementId}
        </foreach>
    </select>
```



### 场景三

```xml
<select id="queryPageList" resultType="com.qf.entity.DeviceApprove">
        select da.id,
        da.device_key,
        da.create_time,
        da.approve_time,
        da.extend_json,
        da.reject_reason,
        da.approve_status,
        da.manager_id,
        ed.device_type,
        ed.imei,
        ed.device_status,
        ed.device_model,
        ed.device_info
        from qf_device_approve da
        left join qf_external_device ed on da.device_key = ed.device_key
        where 1 = 1
        <if test="searchModel.id != null">
            da.id = #{searchModel.id,jdbcType=BIGINT}
        </if>
        <if test="searchModel.deviceKey != null">
            and da.device_key = #{searchModel.deviceKey,jdbcType=VARCHAR}
        </if>
        <if test="searchModel.deviceStatus != null">
            and ed.device_status = #{searchModel.deviceStatus,jdbcType=TINYINT}
        </if>
        <if test="searchModel.deviceType != null">
            and ed.device_type = #{searchModel.deviceType,jdbcType=TIMESTAMP}
        </if>
        <if test="searchModel.approveStatus != null">
            and da.approve_status = #{searchModel.approveStatus,jdbcType=TINYINT}
        </if>
        <if test="searchModel.managerId != null">
            and da.manager_id = #{searchModel.managerId,jdbcType=BIGINT}
        </if>
        <if test="searchModel.startTime != null">
            and da.approve_time &gt;= #{searchModel.startTime,jdbcType=TIMESTAMP}
        </if>
        <if test="searchModel.endTime != null">
            and da.approve_time &lt;= #{searchModel.endTime,jdbcType=TIMESTAMP}
        </if>
        and da.approve_status <![CDATA[ <> ]]> 0
        and da.is_delete=0
        and ed.is_delete = 0
    	<!--order by和group by在不影响业务的情况下尽量符合最左前缀法则-->
        order by da.create_time desc
        limit #{pagination.beginIndex},#{pagination.length}
    </select>
```



### 场景四

```xml
<select id="listDevicePackageByTypeAndPlatform" resultType="com.qf.entity.DevicePackage">
        SELECT <include refid="Base_Column_List" />
        apk.STATE apkState,
        system.STATE systemState,
        al.STATE algorithmState,
        apk.VERSION_NO apkVersion,
        system.VERSION_NO systemVersion,
        al.VERSION_NO algorithmVersion,
        apk.DISABLE_REASON apkDisableReason,
        system.DISABLE_REASON systemDisableReason,
        al.DISABLE_REASON algorithmDisableReason,
        dp.APK_GUID,
        dp.ALGORITHM_GUID,
        dp.SYSTEM_GUID,
        dp.VERSION_SETTING
        FROM qf_device_package dp
        LEFT JOIN qf_apk apk on dp.APK_GUID = apk.GUID
        LEFT JOIN qf_system system on dp.SYSTEM_GUID = system.GUID
        LEFT JOIN qf_algorithm al on dp.ALGORITHM_GUID = al.GUID
        WHERE dp.TYPE = #{type} AND dp.PLATFORM = #{platform}
    </select>
```



### 场景五

```xml
<select id="listDevicePackage" resultType="com.qf.entity.DevicePackage">
        SELECT <include refid="Base_Column_List" />
        apk.STATE apkState,
        system.STATE systemState,
        al.STATE algorithmState,
        apk.VERSION_NO apkVersion,
        system.VERSION_NO systemVersion,
        al.VERSION_NO algorithmVersion,
        apk.DISABLE_REASON apkDisableReason,
        system.DISABLE_REASON systemDisableReason,
        al.DISABLE_REASON algorithmDisableReason,
        dp.APK_GUID,
        dp.ALGORITHM_GUID,
        dp.SYSTEM_GUID,
        dp.VERSION_SETTING
        FROM qf_device_package dp
        LEFT JOIN qf_apk apk on dp.APK_GUID = apk.GUID
        LEFT JOIN qf_system system on dp.SYSTEM_GUID = system.GUID
        LEFT JOIN qf_algorithm al on dp.ALGORITHM_GUID = al.GUID
        <where>
            1=1
            AND dp.STATE = 0
            <if test="devicePackage.id != null and devicePackage.id != 0 ">
                and dp.ID like '%${devicePackage.id}%'
            </if>
            <if test="devicePackage.versionSetting != null ">
                and dp.VERSION_SETTING = #{devicePackage.versionSetting}
            </if>
            <if test="devicePackage.description != null and devicePackage.description != '' ">
                and dp.DESCRIPTION like '%${devicePackage.description}%'
            </if>
            <if test="devicePackage.platform != null and devicePackage.platform != 0 ">
                and dp.PLATFORM = #{devicePackage.platform}
            </if>
            <if test="devicePackage.type != null and devicePackage.type != 0">
                and dp.type = #{devicePackage.type}
            </if>
            <if test="devicePackage.hardwareVersion != null and devicePackage.hardwareVersion != '' ">
                and dp.HARDWARE_VERSION = #{devicePackage.hardwareVersion}
            </if>
            <if test="devicePackage.stateReview != null and devicePackage.stateReview != 0 ">
                and dp.STATE_REVIEW = #{devicePackage.stateReview}
            </if>
            <if test="devicePackage.projectGuid != null and devicePackage.projectGuid != ''  ">
                and dp.PROJECT_GUID = #{devicePackage.projectGuid}
            </if>
            <if test="devicePackage.guid != null and devicePackage.guid != '' ">
                and dp.GUID = #{devicePackage.guid}
            </if>
            <if test="devicePackage.versionNo != null and devicePackage.versionNo != '' ">
                and dp.VERSION_NO like "%${devicePackage.versionNo}%"
            </if>
            <if test="devicePackage.apkVersion != null and devicePackage.apkVersion != '' ">
                and apk.VERSION_NO like "%${devicePackage.apkVersion}%"
            </if>
            <if test="devicePackage.algorithmVersion != null and devicePackage.algorithmVersion != '' ">
                and al.VERSION_NO like "%${devicePackage.algorithmVersion}%"
            </if>
            <if test="devicePackage.systemVersion != null and devicePackage.systemVersion != '' ">
                and system.VERSION_NO like "%${devicePackage.systemVersion}%"
            </if>
            and (dp.OTHER_INFO is null or  dp.OTHER_INFO != '测试使用')
        </where>
       	<!-- order by主要优化手段就是避免逆向排序，对于需要逆向排序的字段可以在业务层进行解决  另外一个优化手段就是符合最左前缀法则--> 
        ORDER BY dp.CREATION_TIME DESC ,dp.VERSION_NO DESC
    	<!--limit 主要就是可以先采用覆盖索引进行优化-->
        limit #{pagination.beginIndex},#{pagination.length}
    </select>
```



### 场景六

```xml
<select id="queryByPackageName" parameterType="java.lang.String" resultType="com.qf.entity.DevicePackage">
        SELECT <include refid="Base_Column_List" />
        apk.STATE apkState,
        system.STATE systemState,
        al.STATE algorithmState,
        apk.VERSION_NO apkVersion,
        system.VERSION_NO systemVersion,
        al.VERSION_NO algorithmVersion,
        apk.DISABLE_REASON apkDisableReason,
        system.DISABLE_REASON systemDisableReason,
        al.DISABLE_REASON algorithmDisableReason,
        dp.APK_GUID,
        dp.ALGORITHM_GUID,
        dp.SYSTEM_GUID,
        dp.VERSION_SETTING
        FROM qf_device_package dp
        LEFT JOIN qf_apk apk on dp.APK_GUID = apk.GUID
        LEFT JOIN qf_system system on dp.SYSTEM_GUID = system.GUID
        LEFT JOIN qf_algorithm al on dp.ALGORITHM_GUID = al.GUID
        <where>
            1=1
            AND dp.STATE = 0
            and dp.DESCRIPTION like '%${description}%'
            and (dp.OTHER_INFO is null or  dp.OTHER_INFO != '内部测试使用')
        </where>
        ORDER BY dp.VERSION_NO DESC
    </select>

```



### 场景七

```xml
 <select id="getSystemAllByQuery" resultType="com.qf.entity.firmware.System">
        SELECT
        GUID,
        us.NAME,
        HARDWARE_VERSION,
        VERSION_NO,
        DESCRIPTION,
        MANAGER_ID,
        us.CREATE_TIME,
        STATE,
        FEATURE_NEW,
        CONTENT_FIX,
        OTHER_INFO,
        ma.NAME managerName,
        DISABLE_REASON disableReason,
        (select count(1) FROM qf_device_package WHERE SYSTEM_GUID = us.GUID AND STATE = 0) packageNum
        FROM qf_system us
        LEFT JOIN qf_manager ma ON ma.ID = us.MANAGER_ID
        <where>
            <if test="systemSearchModel.guid!=null">
                us.GUID = #{systemSearchModel.guid}
            </if>
            <if test="systemSearchModel.hardwareVersion!=null">
                us.HARDWARE_VERSION = #{systemSearchModel.hardwareVersion}
            </if>
            <if test="systemSearchModel.name !=null and systemSearchModel.name != ''">
                AND (us.VERSION_NO like "%${systemSearchModel.name}%" OR us.NAME like "%${systemSearchModel.name}%")
            </if>
            <if test="systemSearchModel.state != null">
                AND us.STATE = #{systemSearchModel.state}
            </if>
            <if test="systemSearchModel.endDate != null">
                AND us.CREATE_TIME &lt; #{systemSearchModel.endDate}
            </if>
            <if test="systemSearchModel.startDate != null">
                AND us.CREATE_TIME &gt; #{systemSearchModel.startDate}
            </if>
        </where>
        ORDER BY us.CREATE_TIME DESC
    </select>
```



### 场景八

```xml
<select id="selectDeviceByUserGuidAndDeviceKey" resultType="com.qf.entity.GroundDevice">
        select ud.NAME, ud.DEVICE_KEY, ud.APP_ID, ud.CLIENT_ID, ud.USER_GUID, ud.CID,
            (
                select qf_application.NAME from qf_ground_service.qf_application
                where
                    <if test="userGuid != null">
                        USER_GUID = #{userGuid} AND
                    </if>
                    qf_application.APP_ID = ud.APP_ID
            ) APP_NAME,
            ds.VERSION_NO
        from qf_device ud
        left join qf_device_service.qf_device ds on ds.DEVICE_KEY = ud.DEVICE_KEY
    	<!--join的关联字段主要是需要建立索引-->
        <where>
            <if test="userGuid != null">
                ud.APP_ID IN (
                select APP_ID from qf_ground_service.qf_application
                where USER_GUID = #{userGuid}
                ) AND
            </if>
            ud.DEVICE_KEY = #{deviceKey}
            <if test="appId != null and appId != '' ">and ud.APP_ID = #{appId}</if>
        </where>
    </select>
```

# 九、锁的定义和分类



## 1.锁的定义

锁是用来解决多个任务（线程、进程）在并发访问同一共享资源时带来的数据安全问题。虽然使用锁解决了数据安全问题，但是会带来性能的影响，频繁使用锁的程序的性能是必然很差的。

锁会带来性能的影响。需要在性能和数据安全之间做一个平衡

对于数据管理软件MySQL来说，必然会到任务的并发访问。那么MySQL是怎么样在数据安全和性能上做权衡的呢？——MVCC设计思想。



## 2.锁的分类

### 1）从性能上划分：乐观锁(读多写少)和悲观锁(写多读少)

- 悲观锁：悲观的认为当前的并发是非常严重的，所以在任何时候操作都是互斥。保证了线程的安全，但牺牲了并发性。——总有刁民要害朕。  ==适用于写多读少==
- 乐观锁：乐观的认为当前的并发并不严重，因此对于读的情况，大家都可以进行，但是对于写的情况，再进行上锁。以CAS自旋锁，在某种情况下性能是ok的，但是频繁自旋会消耗很大的资源。——天网恢恢疏而不漏   ==适用于读多写少==

> 悲观锁是指一个进程拿到共享资源时，首先进行加锁，避免其他进程进行写读         读写均上锁
>
> 乐观锁是指一个进程拿到共享资源时，先不加锁，当需要进行修改时再进行加锁      读时不上锁，写时不上锁

### 2）从数据的操作细粒度上划分：表锁和行锁

- 表锁：对整张表上锁  (几乎不用)
- 行锁：对表中的某一行上锁。  ==for update进行上锁==

### 3）从数据库的操作类型上划分：读锁和写锁

这两种锁都是属于悲观锁      ==读锁和写锁都是悲观锁==

- 读锁（共享锁）：对于同一行数据进行”读“来说，是可以同时进行但是写不行。
- 写锁（排他锁）：在上了写锁之后，及释放写锁之前，在整个过程中是不能进行任何的其他并发操作（其他任务的读和写是都不能进行的）。 即上了写锁之后都不能进行读了

> 小红->相当于共享资源  小明、小军、小王->并发的线程   读锁->约会    写锁->结婚
>
> 有小红、小明、小军、小王四个人，在没有确定关系之前，小明、小军、小王三个人都可以与小红进行约会，但是一旦小王与小红结婚之后，则小军和小明不可以与小红进行约会了。

## 3.表锁

对整张表进行上锁。MyISAM存储引擎是天然支持表锁的，也就是说在MyISAM的存储引擎的表中如果出现并发的情况，将会出现表锁的效果。MyISAM不支持事务。InnoDB支持事务

在InnoDB中上一下表锁:

```sql
# 对一张表上读锁/写锁格式：
lock table 表名 read/write;
# 例子
lock table tb_book read;
# 查看当前会话对所有表的上锁情况
show open tables;
# 释放当前会话的所有锁
unlock tables;

# 对一张表进行插入的时候，相当于对其上写锁
```

读锁： 其他任务可以进行读，但是不能进行写

写锁：其他任务不能进行读和写。

## 4.行锁

MyISAM只支持表锁，但不支持行锁，InnoDB可以支持行锁。

在并发事务里，每个事务的增删改的操作相当于是上了行锁。

上行锁的方式：

```sql
# 对id是8的这行数据上了行锁。
update tb_book set name='qfjava2101' where id=8; 
# 对id是5的这行数据上了行锁。
select * from tb_book where id=5 for update; 

# begin; 开启事务
# commit; 提交事务
# rollback; 回滚事务
```

![image-20211102210707623](http://cdn.nefu-yzk.top/img/image-20211102210707623.png)

## 5.死锁

所谓的死锁，就是开启的锁没有办法关闭，导致资源的访问因为无法获得锁而处于阻塞状态。 死锁演示至少需要两把锁

演示：事务A和事物B相互持有对方需要的锁而不释放，造成死锁的情况。

第一张图是对id=4的进行加锁，期待id=5的锁  第二张图是对id=5进行加锁，期待id=4的锁， 相互期待造成了死锁

![image-20210519104203183](http://cdn.nefu-yzk.top/img/image-20210519104203183.png)

## 6.间隙锁

行锁只能对某一行上锁，如果相对某一个范围上锁，就可以使用间隙锁。间隙锁给的条件where id>13 and id<19，会对13 和19 所处的间隙进行上锁。

![image-20210519105034758](http://cdn.nefu-yzk.top/img/image-20210519105034758.png)
# 十、MVCC设计思想

MySQL为了权衡数据安全和性能，使用了MVCC 多版本并发控制的设计。



## 1.事务的特性

- ACID：A 原子性、C 一致性、I 隔离性、D 持久性  
- 原子性：一个事务是一个最小的操作单位（原子），多条sql语句在一个事务中要么同时成功，要么同时失败。  经典的例子：转账  增加money和减少money肯定是同时进行的
- 一致性：事务提交之前和回滚之后的数据是一致的。
- 持久性：事务一旦提交，对数据的影响是持久的。           即是持久到磁盘中去
- 隔离性：==多个事务==在并发访问下，提供了一套隔离机制，不同的隔离级别(4种隔离级别)会有不同的并发效果。



## 2.事务的隔离级别

- read uncommitted（读未提交）： 在一个事务中读取到另一个事务还没有提交(未commit)的数据——脏读。

- Read committed（读已提交）: 已经解决了脏读问题，在一个事务中只会读取另一个事务已提交的数据，这种情况会出现不可重复读的问题。就是：在事务中重复读数据，数据的内容是不一样的。    即开启两个事务，则在另外一个事务提交前后两次读取的数据是不一样的，出现了不可重复读现象。

![image-20211108140654109](http://cdn.nefu-yzk.top/img/image-20211108140654109.png)

![image-20211108140753970](http://cdn.nefu-yzk.top/img/image-20211108140753970.png)

- repeatable read（可重复读、默认的隔离级别）：在一个事务中每次读取的数据都是一致的，不会出现脏读和不可重复读的问题。会出现虚读（幻读）的问题。

```sql
set session transaction isolation level read uncommitted;  #  设置隔离级别 set session transaction isolation level + 隔离级别
show variables like '%tx_isolation%';  # 查看隔离级别的
```

什么是幻读：即是一个事务对表的增删改对其他事务时不可见的。  即使事务1提交了事务2也无法看到事务1提交的东西

![image-20210519100046820](http://cdn.nefu-yzk.top/img/image-20210519100046820.png)

解决方案：

通过上行锁来解决虚读问题：  增删改之前可以先进行上锁

![](http://cdn.nefu-yzk.top/img/image-20210519100825916.png)

- Serializable:串行化的隔离界别直接不允许事务的并发发生，不存在任何的并发性。相当于锁表，性能非常差，一般都不考虑



脏读(读未提交)、不可重复读(读已提交)、虚读（幻读）(可重复读)



## 3.MVCC思想解读

MySQL在读和写的操作中，对读的性能做了并发性的保障，让所有的读都是快照读，对于写的时候(比较事务ID，不一样时需要进行数据更新)，进行版本控制，如果真实数据的版本比快照版本要新，那么写之前就要进行版本（快照）更新，这样就可以既能够提高读的并发性，又能够保证写的数据安全。

快照进行读，写之前先判断事务ID是否一样，不一样时需要进行更新，然后再进行写

![image-20210519095148255](http://cdn.nefu-yzk.top/img/image-20210519095148255.png)





# 十一、MySQL常用命令

## day01学习笔记

```mysql
# 不报错的进行删除表的操作
drop table if exists t_user;
# where后面条件不能出现聚合函数sum、min、max、count、avg
# DQL：select  Q：query
# DDL：create、drop、alter  D：definition
# DML：insert、update、delete  M：Manipulation
# DCL：grant、revoke...  C：control
# TCL:commit、rollback  T：trancation

# 显示数据库
show databases;
# 显示正在使用那个数据库
select database();
# 显示数据库的版本号
select version();
# ctrl + C终止一条命令的执行
# 查看表的结构
desc table_name;

# 快速的复制表
create table emp_bak as select * from tmp;
# 只要其中的一部分
create table emp_bak as select id, name from tmp;

# 注意：在所有的数据库当中，字符串统一使用单引号括起来，单引号是标准，双引号在oracle数据库中用不了。但是在mysql中可以使用。

# 再次强调：数据库中的字符串都是采用单引号括起来。这是标准的。双引号不标准!!!

# 条件查询：=、<> | !=、<、>、>=、<=、between...and...、is null、is not null、and、or、in、not in、like
# like用去模糊查询  _下划线表示匹配一个字符  % 表示匹配多个字符
# 找出名字以T结尾的？
select ename from emp where ename like '%T';
			
# 找出名字以K开始的？
select ename from emp where ename like 'K%';

# 找出第二个字每是A的？
select ename from emp where ename like '_A%';
		
# 找出第三个字母是R的？
select ename from emp where ename like '__R%';

# 找出名字中有“_”的？ 
select name from t_student where name like '%_%';  # 这样不行。

select name from t_student where name like '%\_%'; # \转义字符。

# 注意：使用\转义字符，表示不以通配符的形式进行查询，而是单独的匹配_这个字符

#  注意判断是否为空的时候尽量使用is null || is not null, 使用!=或者=有坑

# 注意：在数据库当中null不能使用等号进行衡量。需要使用is null, 因为数据库中的null代表什么也没有，它不是一个值，所以不能使用等号衡量。


# 数据处理函数，又称为单行处理函数
# 即一个输入对应一个输出，与之对应的是多行处理函数(聚合函数：多个输入，对应一个输出)
# 单行处理函数有：lower、upper、substr、concat、length、trim、str_to_date、date_format、case..when..then..when..then..else..end、format、round、rand()、ifnull
# lower：转小写
# upper：转大写
# substr: 截取字符串 下标从1开始，substr( 被截取的字符串, 起始下标,截取的长度)
# concat: 拼接字符串
# length: 取列字段值的长度
# trim: 去除列的左右两边的空白
# str_to_date: 字符串转日期   默认的为%Y-%m-%d    如'1999-10-01'就不用转了，但如01-10-1999就需要转为str_to_date('01-10-1999', '%d-%m-Y')进行存储
# date_format: 日期转为对应的字符串
# case..when..then..when..then..else..end: 
case job when 'MANAGER' then sal*1.1 when 'SALESMAN' then sal*1.5 else sal end
# 上述的语句即是case的分支执行情况  case 字段 when 条件1 then 执行1 when 条件2 then 执行2 else 其他 end
# format：设置千分位
select format(sal, '$999.999') from emp;
# round：设置字段的保留位数
# rand(): 随机生成0-1的数字
select round(rand()*9 + 1, 0) from emp; # 表示生成1-10的数字

# 在所有数据库当中，只要有NULL参与的数学运算，最终结果就是NULL。为了避免这个现象，需要使用ifnull函数。
# ifnull: ifnull(数据, 被当做哪个值), 如果“数据”为NULL的时候，把这个数据结构当做哪个值。

# 多行处理函数(聚合函数、分组函数)：min、max、count、avg、sum
# min：求最小值
# max：求最大值
# avg：求平均值
# count：求记录数
# sum：对某一列求和

# 分组函数在使用的时候必须先进行分组，然后才能用。如果你没有对数据进行分组，整张表默认为一组。

# 第一点：分组函数自动忽略NULL，你不需要提前对NULL进行处理。
# 第二点：分组函数中count(*)和count(具体字段)有什么区别？
  -- count(具体字段)：表示统计该字段下所有不为NULL的元素的总数。
  -- count(*)：统计表当中的总行数。（只要有一行数据count则++） 因为每一行记录不可能都为NULL，一行数据中有一列不为NULL，则这行数据就是有效的。
# 第三点：分组函数不能够直接使用在where子句中。
# 第四点：所有的分组函数可以组合起来一起用。

select
	...
from
	...
where
	...
group by
	...
order by
	...
		
# 以上关键字的顺序不能颠倒，需要记忆。
# 执行顺序是什么？  from、where、group by、select、order by
# 从某张表中查询数据，先经过where条件筛选出有价值的数据。对这些有价值的数据进行分组。分组之后可以使用having继续筛选。select查询出来。最后排序输出！
		
# 为什么分组函数不能直接使用在where后面？
   -- select ename,sal from emp where sal > min(sal);//报错。
   -- 因为分组函数在使用的时候必须先分组之后才能使用。
   -- where执行的时候，还没有分组。所以where后面不能出现分组函数。
   -- select sum(sal) from emp; 
   -- 这个没有分组，为啥sum()函数可以用呢？
        -- 因为select在group by之后执行。
        
# 使用having可以对分完组之后的数据进一步过滤。having不能单独使用，having不能代替where，having必须和group by联合使用。

# 优化策略：where和having，优先选择where，where实在完成不了了，再选择having。
```

## day02学习笔记

```mysql
# distinct：查询结果去除重复记录
# 注：只是查询结果去重，原表数据不会修改
# distinct只能出现在所有字段的最前方。
# distinct出现在job,deptno两个字段之前，表示两个字段联合起来去重。是且的关系，即job与deptno都重复
select distinct job,deptno from emp;

# 连接查询，跨表查询被称为连接查询
# 根据表的连接方式：内连接(等值连接、非等值连接、自连接)、外连接(左连接、右连接)
# 当两张表进行连接查询，没有任何条件限制的时候，最终查询结果条数，是两张表条数的乘积，这种现象被称为：笛卡尔积现象。
# 怎么避免笛卡尔积现象？连接时加条件，满足这个条件的记录被筛选出来！
# 但是加条件并没有使匹配次数减少，只是使查询结果看起来更加的符合我们的需求

# 注意：通过笛卡尔积现象得出，表的连接次数越多效率越低，尽量避免表的连接次数。

# 等值连接
# SQL92语法：
select e.ename,d.dname from emp e, dept d where e.deptno = d.deptno;
	
# SQL92的缺点：结构不清晰，表的连接条件，和后期进一步筛选的条件，都放到了where后面。

# SQL99语法：
select e.ename,d.dname from emp e join dept d on e.deptno = d.deptno;
# SQL99优点：表连接的条件是独立的，连接之后，如果还需要进一步筛选，再往后继续添加where

# 非等值连接
# 案例：找出每个员工的薪资等级，要求显示员工名、薪资、薪资等级？
select 
	e.ename, e.sal, s.grade
from
	emp e
join
	salgrade s
on
	e.sal between s.losal and s.hisal; 
# 条件不是一个等量关系，称为非等值连接。

# 自连接
# 案例：查询员工的上级领导，要求显示员工名和对应的领导名？
select 
	a.ename as '员工名', b.ename as '领导名'
from
	emp a
join
	emp b
on
	a.mgr = b.empno; 
# 员工的领导编号 = 领导的员工编号
# 以上就是内连接中的：自连接，技巧：一张表看做两张表。

# 外连接：存在主次关系 有一张表是主表，然后顺带查询出副表的记录
# right代表什么：表示将join关键字右边的这张表看成主表，主要是为了将这张表的数据全部查询出来，捎带着关联查询左边的表。在外连接当中，两张表连接，产生了主次关系。

# 注：外连接的查询结果条数>=内连接的查询结果条数



# 子查询
# select语句中嵌套select语句，被嵌套的select语句称为子查询。

select 
	e.ename,e.deptno,(select d.dname from dept d where e.deptno = d.deptno) as dname 
from 
	emp e;
# 上面的语句是对的

select 
	e.ename,e.deptno,(select dname from dept) as dname
from
	emp e;
# 错误：ERROR 1242 (21000): Subquery returns more than 1 row
# 注意：对于select后面的子查询来说，这个子查询只能一次返回1条结果。多于1条，就报错了！

# union合并查询结果集，效果相比之下更高，减少了表的连接，只是对查询结果做一个并的操作
select ename,job from emp where job = 'MANAGER' or job = 'SALESMAN';
select ename,job from emp where job in('MANAGER','SALESMAN');

select ename,job from emp where job = 'MANAGER'
union
select ename,job from emp where job = 'SALESMAN';


# union的效率要高一些。对于表连接来说，每连接一次新表，则匹配的次数满足笛卡尔积，成倍的翻。但是union可以减少匹配的次数。在减少匹配次数的情况下，还可以完成两个结果集的拼接。


# limit
# limit startIndex, length
# startIndex是起始下标，length是长度。起始下标从0开始
# mysql中limit在order by之后执行

# 分页公式 limit (pageNo-1)*pageSize , pageSize
# 其中pageNo是页码，pageSize是每页的记录条数


# 关于DQL(数据查询语言)语句的大总结：
select 
	...
from
	...
where
	...
group by
	...
having
	...
order by
	...
limit
	...
# 执行顺序：from、where、group by、having、select、order by、limit

# DDL(数据定义语言)包括：create drop alter
# 表名：建议以t_ 或者 tbl_开始，可读性强。见名知意。
# 字段名：见名知意。
# 表名和字段名都属于标识符。

# mysql中数据类型
# varchar：可变长度的字符串
# char：固定长度的字符串
# 注：varchar 动态分配空间，速度慢，但节省空间
#     char 速度快，但使用不当会造成空间的浪费
#     两者的最大长度都是255个字符，超过时使用clob(4GB)进行存储
# int: 数字中的整型  与java中的int一样
# bigint： 数字中的长整型，与java中的long一样
# float：与上同理
# double：与上一样
# date：短日期类型，%Y-%m-%d形式  只包括年月日信息
# datetime：长日期类型，%Y-%m-%d %h:%i:%s 包括年月日时分秒信息
# clob:Character Large OBject,超过255个字符的都要采用CLOB字符大对象来存储.最多可以存储4GB的字符串，
# blob：Binary Large OBject ，专门用来存储图片、声音、视频等流媒体数据。往BLOB类型的字段上插入数据的时候，例如插入一个图片、视频等。使用IO流进行操作

# drop table if exists t_student;  # 不报错的进行删除表
# 注意：数据库中的有一条命名规范：所有的标识符都是全部小写，单词和单词之间使用下划线进行衔接。

# mysql的日期格式：
%Y		年
%m 		月
%d 		日
%h		时
%i		分
%s		秒
# now() 函数获取当前的时间


# 修改update（DML）语法格式：
update table_name set clounm1=value1, clounm2=value2, clounm3=value3... where update_condition;

# 删除数据 delete （DML）语法格式:
delete from table_name where delete_condition;
# 注意：没有条件，整张表的数据会全部删除！
# 只是删除表中的数据，表依然存在

# 一次可以插入多条记录：insert into t_user(字段名1,字段名2) values(),(),(),();

# 快速的创建表
create table emp2 as select * from emp;
# 只取表中的部分列
create table emp3 as select id,name from emp;

# delete语句删除数据的原理？（delete属于DML语句！！！）
# 表中的数据被删除了，但是这个数据在硬盘上的真实存储空间不会被释放！！！
# 这种删除缺点是：删除效率比较低。
# 这种删除优点是：支持回滚(rollback)，后悔了可以再恢复数据！！！
delete from dept_bak; 

# truncate语句删除数据的原理？
# 这种删除效率比较高，表被一次截断，物理删除。
# 这种删除缺点：不支持回滚。
# 这种删除优点：快速。
truncate table dept_bak; 

# 大表非常大时，如有上亿条记录？
# 删除的时候，选取delete可能需要1h才能删除，选取truncate可能就1s不到

# 但是使用truncate由于不能回滚，所以必须仔细询问客户是否真的要删除。
# truncate是删除表中的数据，但是表还在
# drop table table_name是删除表，未删除表的数据

# 表的约束：not null、unique、primary key、foreign key、check
# not null: 非空约束not null约束的字段不能为NULL。
# unique: 唯一性约束unique约束的字段不能重复，但是可以为NULL。
# primary key:
# foreign key:

# 使用source 命令来执行SQL脚本文件
mysql> source D:\course\03-MySQL\document\vip.sql
```

