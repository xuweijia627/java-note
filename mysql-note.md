
### 1.SQL优化
#### 表结构检查
* 尽量使用TINYINT、SMALLINT、MEDIUM_INT作为整数类型而非INT，如果非负则加上UNSIGNED
* 列字符集: 从MySQL 5.6开始建议所有对象字符集应该使用用utf8mb4，包括MySQL实例字符集，数据库字符集，表字符集，列字符集。避免在关联查询Join时字段字符集不匹配导致索引失效，同时目前只有utf8mb4支持emoji表情存储。
* 不得使用外键与级联，一切外键概念必须在应用层解决。外键影响数据库的插入速度。
* 根据业务含义，尽量将字段都添加上NOT NULL DEFAULT VALUE属性，如果列值存储了大量的NULL，会影响索引的稳定性。


#### select检查
* UDF用户自定义函数: SQL语句的select后面使用了自定义函数UDF，SQL返回多少行，那么UDF函数就会被调用多少次，这是非常影响性能的。
```bash
  #getOrderNo是用户自定义一个函数用户来根据order_sn来获取订单编号
  select id, payment_id, order_sn, getOrderNo(order_sn) from payment_transaction;
```
* text类型检查: 如果select出现text类型的字段，就会消耗大量的网络和IO带宽，由于返回的内容过大超过max_allowed_packet设置会导致程序报错，需要评估谨慎使用。
```bash
#表request_log的中content是text类型。
select user_id, content, status, url, type from request_log where user_id = 32121;
```
* group_concat谨慎使用: gorup_concat是一个字符串聚合函数，会影响SQL的响应时间，如果返回的值过大超过了max_allowed_packet设置会导致程序报错。
* 内联子查询: 在select后面有子查询的情况称为内联子查询，SQL返回多少行，子查询就需要执行过多少次，严重影响SQL性能。
```bash
select id,(select rule_name from member_rule limit 1) as rule_name, member_id, member_type from xxx
```
#### from检查
* 表的链接方式: 在MySQL中不建议使用Left Join，即使ON过滤条件列索引，一些情况也不会走索引，导致大量的数据行被扫描，SQL性能变得很差，同时要清楚ON和Where的区别。
* 子查询: 由于MySQL的基于成本的优化器CBO对子查询的处理能力比较弱，不建议使用子查询，可以改写成Inner Join。
#### where检查
* 索引列被运算: 当一个字段被索引，同时出现where条件后面，是不能进行任何运算，会导致索引失效。
```bash
#device_no列上有索引，由于使用了ltrim函数导致索引失效
select id, name , phone, address, device_no from users where ltrim(device_no) = 'Hfs1212121';
#balance列有索引,由于做了运算导致索引失效
select account_no, balance from accounts where balance + 100 = 10000 and status = 1;
```
* 类型转换: 对于Int类型的字段，传varchar类型的值是可以走索引，MySQL内部自动做了隐式类型转换；相反对于varchar类型字段传入Int值是无法走索引的，应该做到对应的字段类型传对应的值总是对的。
* 禁止使用全模糊，左模糊匹配，如果需要用搜索引擎解决
* 使用 ISNULL()来判断是否为 NULL 值。NULL 与任何值的直接比较都为 NULL。
#### group by检查

#### limit m,n要慎重
对于limit m, n分页查询，越往后面翻页即m越大的情况下SQL的耗时会越来越长，对于这种应该先取出主键id，然后通过主键id跟原表进行Join关联查询。

#### 索引检查(单列索引，多列索引，前缀索引，主键索引，唯一索引。从索引数据结构来看分为：聚集索引，非聚集索引)
* 查看表有哪些索引：show index from table_name;
```bash
mysql> show index from member_base_info;
+------------------+------------+----------------------------+--------------+-------------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table            | Non_unique | Key_name                   | Seq_in_index | Column_name       | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+------------------+------------+----------------------------+--------------+-------------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| member_info |          0 | PRIMARY                    |            1 | id                | A         |      131088 | NULL     | NULL   |      | BTREE      |         |               |
| member_info |          0 | uk_member_id               |            1 | member_id         | A         |      131824 | NULL     | NULL   |      | BTREE      |         |               |
| member_info |          1 | idx_create_time            |            1 | create_time       | A         |        6770 | NULL     | NULL   |      | BTREE      |         |               |
+------------------+------------+----------------------------+--------------+-------------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
#Table：表名
#Non_unique ：是否为unique index，0-是，1-否。
#Key_name：索引名称
#Seq_in_index：索引中的顺序号，单列索引-都是1；复合索引-根据索引列的顺序从1开始递增。
#Column_name：索引的列名
#Collation：排序顺序，如果没有指定asc/desc，默认都是升序ASC。
#Cardinality：索引基数-索引列唯一值的个数。索引基数指的是被索引的列唯一值的个数，唯一值越多接近表的count(*)说明索引的选择率越高，
#通过索引扫描的行数就越少，性能就越高，例如主键id的选择率是100%，在MySQL中尽量所有的update都使用主键id去更新，因为id是聚集索引存储着整行数据，不需要回表，性能是最高的。

#sub_part：前缀索引的长度；例如index (member_name(10)，长度就是10。
#Packed：索引的组织方式，默认是NULL。
#Null：YES:索引列包含Null值；'':索引不包含Null值。
#Index_type：默认是BTREE，其他的值FULLTEXT，HASH，RTREE。
#Comment：在索引列中没有被描述的信息，例如索引被禁用。
#Index_comment：创建索引时的备注。
```
* 前缀索引：对于变长字符串类型varchar(m)，为了减少key_len，可以考虑创建前缀索引，但是前缀索引不能消除group by， order by带来排序开销。
如果字段的实际最大值比m小很多，建议缩小字段长度。
```bash
alter table member_info add index idx_member_name_part(member_name(10));
```
* 复合索引顺序：有很多人喜欢在创建复合索引的时候，总以为前导列一定是唯一值多的列，例如索引index idx_create_time_status(create_time, status)，这个索引往往是无法命中，因为扫描的IO次数太多，总体的cost的比全表扫描还大，CBO最终的选择是走full table scan。
```bash
#MySQL遵循的是索引最左匹配原则，对于复合索引，从左到右依次扫描索引列，到遇到第一个范围查询（>=, >,<, <=, between ….. and ….）就停止扫描，索引正确的索引顺序应该是index idx_status_create_time(status, create_time)。
select account_no, balance from accounts where status = 1 and create_time between '2020-09-01 00:00:00' and '2020-09-30 23:59:59';
```
* 时间列索引：对于默认字段created_at(create_time)、updated_at(update_time)这种默认就应该创建索引，这一般来说是默认的规则。
* 超过三个表禁止 join。需要 join 的字段，数据类型保持绝对一致；多表关联查询时，保证被关联的字段需要有索引。

### 存储引擎
存储引擎存在的意义：数据本身在磁盘上面，那么怎么从磁盘上查询这些数据呢？这便是储存引擎决定的，储存引擎负责实现读写接口，上层只调用读写接口，不需要关心底层怎么实现的。
#### InnoDB
InnoDB整体架构分为两大块：In-Memory Structures(内存结构)，On-Disk Structures(磁盘结构)
* On-Disk Structures：表空间(存放表数据的一个空间，每一个表会对应一个文件或者多个表对应一个文件)
* In-Memory Structures：Buffer Pool，Change Buffer
* 查询数据的过程：先从磁盘查到数据，再加载到Buffer Pool，再返回给调用者。
* 修改数据的过程：先改Buffer Pool中的数据，再写redo log，最后修改磁盘。如果缓存池中没有这一条要修改的数据，那么会先去磁盘查出来放入缓存池再修改。所以一切修改操作都在缓存池中进行。
* InnoDB以Page为单位将数据从磁盘中读取出来。Page默认大小为16K
