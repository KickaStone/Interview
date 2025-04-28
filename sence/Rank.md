# Rank Design

需求描述：
假设有一款游戏需要设计一个排行榜功能：
1. 需要关联用户ID，游戏分数和游戏完成时间
2. 比较方法是游戏分数降序，相同分数按照游戏完成时间升序（比谁分高和更快）

## 数据库设计

如果仅仅用数据库实现
```sql
CREATE TABLE leaderboard (
    user_id     INTEGER     NOT NULL PRIMARY KEY,
    SCORE       INTEGER     NOT NULL,
    duration    INTEGER     NOT NULL,
    update_at   DATETIME    NOT NULL DEFAULT CURRENT_TIMESTAMP
)
```

插入更新逻辑删除逻辑不说了，主要是查询逻辑

基本的查询逻辑如下

```sql
SELECT user_id, score, duration FROM leaderboard
ORDER BY score DESC, duration ASC LIMIT 100;
```

### 性能优化

1. 复合索引

​	为了加快排序，可对主要查询子段使用覆盖索引

```sql
CREATE INDEX idx_leader_score_duration 
ON leaderboard(score DESC, duration ASC);
```

>覆盖索引：覆盖索引不是一种索引，而是一种基于索引查询的方式。
>
>即将查询sql中的字段添加到联合索引里面，只要保证查询语句里面的字段都在索引文件中，就无需进行回表查询（InnoDB）

2. 分库分表（本场景不太适用）

### 缓存优化

使用redis实现排行榜

主要用到的是zset结构



zset底层结构是skip list和zip list

基于一篇论文？

什么时候使用压缩列表（元素个数小于128 && 总大小小于64B），什么时候使用跳表

skiplist 添加多级索引，可以快速查找元素

[查找元素的动画](https://www.bilibili.com/video/BV1uCYLeiEzw)

查找元素基本就是从顶层索引向下查找，每次定位一个小区间，然后再向下查找

```c
char* search(skipList *list, int searchKey){
  skipNode *x = list->header;
  for(int i = list->level-1; i >=0 ;i--){
    while(x->level[i].forward && x->level[i].forward->key < searchKey){
      x = x->level[i].forward;
    }
  } // 多层索引查找
  x = x->level[0].forward; // 最后的x应该是刚好小于searchKey，还需要向前移动一步
  if(x->key == searchKey){
    return x->value;
  }else{
    return NULL;
  }
}
```



**基本使用方法**

- 添加 member 命令格式：`zadd key score member [score member ...]`
- 增加 member 的 score 命令格式：`zincrby key increment member`
- 获取 member 排名命令格式：`zrank/zrevrank key member`
- 返回指定排名范围内的 member 命令格式：`zrange/zrevrange key start end [withscores]`
- 获取全部排名`start=0, end = -1`



上面的redis命令已经可以基本实现排行榜的功能，但有些细节需要设计



#### score分数设计-多维排序

score是double类型的，double类型最重要特点就是精度有限，在设计score的计算规则的时候需要尤其注意！

score 52位精度，前20位存得分， 后面做个时间戳，取unix时间算个值，注意和对应时间精度做匹配。

时间部分

- 时间短排名高：使用一个截止时间，用户时间求差值
- 最新的排名高：使用提交时间即可

设计时注意数据大小排序的意义



#### **关联用户数据缓存**

上面zset给的只是一个排序，和一个member值而已，









## 参考

[凉了呀，面试官叫我设计一个排行榜](https://www.cnblogs.com/thisiswhy/p/14470861.html)
