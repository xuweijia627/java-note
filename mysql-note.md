
### 1.SQL优化
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
