> 部分源自公众号《架构师之路》

### 负向条件查询不能使用索引

```
SELECT * FROM `e_orders` WHERE `status` != 0 AND `stauts` != 1;
```
```NOT IN```和```NOT EXISTS```一样，可以优化为```IN```查询语句：
```
SELECT * FROM `e_orders` WHERE `status` IN (2,3);
```

### 模糊查询前置```%```不能使用索引

```
SELECT * FROM `e_orders` WHERE `product_name` LIKE '%全新';
SELECT * FROM `e_orders` WHERE `product_name` LIKE '%全新%';
```
非前置```%```则可以：
```
SELECT * FROM `e_orders` WHERE `product_name` LIKE '全新%';
```

### 数据区分度不大的字段不宜使用索引

```
SELECT * FROM `user` WHERE `sex` = 1;
```
原因：性别只有男、女，每次过滤掉的数据很少，不宜使用索引。经验上，能过滤80%数据时就可以使用索引。对于订单状态，如果状态值很少，不宜使用索引，如果状态值很多，能够过滤大量数据，则应该建立索引。

### 在属性上进行计算不能命中索引

```
SELECT * FROM `e_orders` WHERE YEAR(`date`) >= '2018';
```
即使date上建立了索引，也会全表扫描，可优化为值计算：
```
SELECT * FROM `e_orders` WHERE `date` >= '2017';
```
或者
```
SELECT * FROM `e_orders` WHERE `date` >= '2018-01-01';
```
或者
```
SELECT * FROM `e_orders` WHERE `date` >= $year;
```

### 单条查询，使用Hash索引性能更好

```
SELECT * FROM `user` WHERE id = 1000;
SELECT * FROM `user` WHERE `username`= 'admin';
```
原因：B-Tree索引的时间复杂度是O(log(n))，而Hash索引的时间复杂度是O(1)

### 不允许为```NULL```的列

单列索引不存```NULL```值，复合索引不存全为```NULL```的值，如果列允许为```NULL```，可能会得到“不符合预期”的结果集
```
SELECT * FROM `user` WHERE `username` != 'admin';
```
如果username允许为```NULL```，索引不存储```NULL```值，结果集中不会包含这些记录。所以，请使用```NOT NULL```约束以或默认值。

### 复合索引前缀原则

用户user建立(username, password)的复合索引
```
SELECT * FROM `user` WHERE `username` = 'admin' AND `password` = 'e10adc3949ba59abbe56e057f20f883e';
SELECT * FROM `user` WHERE `password` = 'e10adc3949ba59abbe56e057f20f883e' AND `username` = 'admin';
```
都能够命中索引
```
SELECT * FROM `user` WHERE `username` = 'admin';
```
也能命中索引，满足复合索引最左前缀
```
SELECT * FROM `user` WHERE `password` = 'e10adc3949ba59abbe56e057f20f883e';
```
不能命中索引，不满足复合索引前缀原则

### 使用```enum```而不是字符串

```enum```保存的是```tinyint```，别在枚举中搞一些“中国”“北京”“技术部”这样的字符串，字符串空间又大，效率又低。

### 如果明确知道只有一条结果返回```LIMIT 1```能够提高效率

```
SELECT * FROM `user` WHERE `username` = 'admin';
```
可以优化为：
```
SELECT * FROM `user` WHERE `username` = 'admin' LIMIT 1;
```
原因：你知道只有一条结果，但数据库并不知道，明确告诉它，让它主动停止游标移动

### 禁止数据库业务逻辑操作

```
SELECT * FROM `e_order` WHERE `date` <= CURDATE();
```
优化为
```
$date = date('Y-m-d');
$res  = mysql_query('SELECT * FROM `e_orders` WHERE date <= $date');
```
原因：释放了数据库的CPU，多次调用，传入的SQL相同，才可以利用查询缓存

### 隐式数据类型转换会造成全表扫描

```
SELECT * FROM `user` WHERE `phone` = 18817771888;
```
应该优化为：
```
SELECT * FROM `user` WHERE `phone` = '18817771888';
```
由于数据库```phone```本身存储的为```char```类型，而条件为整型，会造成全表扫描，将```phone```每条数据转为```float```然后进行比较

### 禁用```SELECT * ```操作，明确查询列

```SELECT * ```会增加磁盘IO，网络数据传输量，CPU负载

### 保证```SQL```大小写一致

可以有效的使用缓存，所以使用```ORM```可以保证生成的```SQL```一致

### 使用```EXPLAIN/DESC```查看```SQL```执行计划

```
DESC SELECT * FROM `user` WHERE `username` = 'admin';
```

### 数据量比较少的表不适宜建索引

索引本质上是空间换时间，因为数据量少，先查询索引，然后查询表数据，因为索引并不能过滤多少数据，不如直接查询表
