
### 1.SQL优化
#### select检查
* 尽量使用TINYINT、SMALLINT、MEDIUM_INT作为整数类型而非INT，如果非负则加上UNSIGNED
* SQL语句的select后面使用了自定义函数UDF，SQL返回多少行，那么UDF函数就会被调用多少次，这是非常影响性能的。
```bash
  #getOrderNo是用户自定义一个函数用户来根据order_sn来获取订单编号
  select id, payment_id, order_sn, getOrderNo(order_sn) from payment_transaction;
```
* 如果select出现text类型的字段，就会消耗大量的网络和IO带宽，由于返回的内容过大超过max_allowed_packet设置会导致程序报错，需要评估谨慎使用。
```bash
#表request_log的中content是text类型。
select user_id, content, status, url, type from request_log where user_id = 32121;
```
* gorup_concat是一个字符串聚合函数，会影响SQL的响应时间，如果返回的值过大超过了max_allowed_packet设置会导致程序报错。
* 在select后面有子查询的情况称为内联子查询，SQL返回多少行，子查询就需要执行过多少次，严重影响SQL性能。
```bash
select id,(select rule_name from member_rule limit 1) as rule_name, member_id, member_type from xxx
```
#### from检查
* 在MySQL中不建议使用Left Join，即使ON过滤条件列索引，一些情况也不会走索引，导致大量的数据行被扫描，SQL性能变得很差，同时要清楚ON和Where的区别。
* 由于MySQL的基于成本的优化器CBO对子查询的处理能力比较弱，不建议使用子查询，可以改写成Inner Join。
* 当一个字段被索引，同时出现where条件后面，是不能进行任何运算，会导致索引失效。
```bash
#device_no列上有索引，由于使用了ltrim函数导致索引失效
select id, name , phone, address, device_no from users where ltrim(device_no) = 'Hfs1212121';
#balance列有索引,由于做了运算导致索引失效
select account_no, balance from accounts where balance + 100 = 10000 and status = 1;
```
* 对于Int类型的字段，传varchar类型的值是可以走索引，MySQL内部自动做了隐式类型转换；相反对于varchar类型字段传入Int值是无法走索引的，应该做到对应的字段类型传对应的值总是对的。
* 从MySQL 5.6开始建议所有对象字符集应该使用用utf8mb4，包括MySQL实例字符集，数据库字符集，表字符集，列字符集。避免在关联查询Join时字段字符集不匹配导致索引失效，同时目前只有utf8mb4支持emoji表情存储。
