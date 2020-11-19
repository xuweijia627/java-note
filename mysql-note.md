
### 1.SQL优化
* 尽量使用TINYINT、SMALLINT、MEDIUM_INT作为整数类型而非INT，如果非负则加上UNSIGNED
* SQL语句的select后面使用了自定义函数UDF，SQL返回多少行，那么UDF函数就会被调用多少次，这是非常影响性能的。
```bash
  #getOrderNo是用户自定义一个函数用户来根据order_sn来获取订单编号
  select id, payment_id, order_sn, getOrderNo(order_sn) from payment_transaction where status = 1 and create_time between '2020-10-01 10:00:00' and '2020-10-02 10:00:00';
```
* 如果select出现text类型的字段，就会消耗大量的网络和IO带宽，由于返回的内容过大超过max_allowed_packet设置会导致程序报错，需要评估谨慎使用。
```bash
#表request_log的中content是text类型。
select user_id, content, status, url, type from request_log where user_id = 32121;
```
