# DUID(Distributed Universal Unique Identifier)

# 要求
* 全局唯一
* 高性能
* 高可用
* 接入方便
* 趋势递增（部分业务要求）

# 1. uuid
    123e4567-e89b-12d3-a456-426655440000
    128-bit

# 2. 数据库自增ID
mysql瓶颈

# 3. 基于数据库集群模式
多主

# 4. 基于数据库的号段模式

号段模式可以理解为从数据库批量的获取自增ID，每次从数据库取出一个号段范围，例如 (1,1000] 代表1000个ID，具体的业务服务将本号段，生成1~1000的自增ID并加载到内存。表结构如下：

```sql
CREATE TABLE id_generator (
  id int(10) NOT NULL,
  max_id bigint(20) NOT NULL COMMENT '当前最大id',
  step int(20) NOT NULL COMMENT '号段的布长',
  biz_type	int(20) NOT NULL COMMENT '业务类型',
  version int(20) NOT NULL COMMENT '版本号',
  PRIMARY KEY (`id`)
)
```
# 5. 基于Redis模式
原理就是利用redis的 incr命令实现ID的原子性自增
用redis实现需要注意一点，要考虑到redis持久化的问题
* RDB会定时打一个快照进行持久化，假如连续自增但redis没及时持久化，而这会Redis挂掉了，重启Redis后会出现ID重复的情况。
* AOF会对每条写命令进行持久化，即使Redis挂掉了也不会出现ID重复的情况，但由于incr命令的特殊性，会导致Redis重启恢复的数据时间过长。

# 6. 雪花算法（Snowflake）模式
![img.png](img.png)

国内大厂都有基于Snowflake的方案
