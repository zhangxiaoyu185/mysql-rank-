# mysql-rank-
mysql 排名排序 模拟rank

## 建表
```
CREATE TABLE `record` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `name` varchar(64) DEFAULT NULL COMMENT '昵称',
  `score` int(11) DEFAULT NULL COMMENT '分数',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=12 DEFAULT CHARSET=utf8;
```

## 表数据
```
INSERT INTO `record` VALUES ('1', '1', '10');
INSERT INTO `record` VALUES ('2', '2', '20');
INSERT INTO `record` VALUES ('3', '3', '50');
INSERT INTO `record` VALUES ('4', '4', '80');
INSERT INTO `record` VALUES ('5', '5', '50');
INSERT INTO `record` VALUES ('6', '6', '30');
INSERT INTO `record` VALUES ('7', '7', '20');
INSERT INTO `record` VALUES ('8', '8', '40');
INSERT INTO `record` VALUES ('9', '9', '10');
INSERT INTO `record` VALUES ('10', '10', '70');
INSERT INTO `record` VALUES ('11', '11', '80');
```

## 排名跳位
> 效果

![](http://media.hzlingdian.com/blog/rank/rank_jump.png)
1. 计算出比当前小的个数来得到rank
```
SELECT t1.id, t1.name, t1.score, COUNT(t2.score) as rank 
FROM record t1, record t2 
WHERE t1.score < t2.score OR (t1.score=t2.score AND t1.id = t2.id) 
GROUP BY t1.id
ORDER BY t1.score DESC, t1.id DESC;
```
2. 增加一个变量（@incRank）用于记录序号，如果当前 age 与上一条记录相同，使用与前一记录相同的排名，否则使用序号（@incRank）
```
SELECT id, name, score,
	@curRank :=
		IF (
			@prevRank = score,
			@curRank,
			@incRank
		) as rank,
	@incRank := @incRank + 1,
	@prevRank := score
FROM record, (SELECT @curRank := 0, @prevRank := NULL, @incRank := 1) init
ORDER BY score DESC, id DESC
```
## 排名不跳位
> 效果

![](http://media.hzlingdian.com/blog/rank/rank_not_jump.png)
1. 去除重复分数后，计算出比当前小的分数来得到rank
```
SELECT t1.id, t1.name, t1.score, count(distinct t2.score) as rank 
FROM record t1, record t2 
WHERE t1.score <= t2.score
GROUP BY t1.id
ORDER BY t1.score DESC, t1.id DESC
```
或
```
SELECT t1.id, t1.name, t1.score, (SELECT COUNT(distinct t2.score) FROM record t2 WHERE t1.score <= t2.score) as rank 
FROM record t1
ORDER BY t1.score DESC, t1.id DESC
```
2. 暂存上一条记录的 score(@prev)，如果当前 score与其相等，使用与前一记录相同的排名，否则排名加一，并更新 @prevRank
```
SELECT id, name, score, @rank := @rank + (@prev <> (@prev := score)) as rank
FROM record, (SELECT @rank := 0, @prev := -1) init
ORDER BY score DESC, id DESC
```
或
```
SELECT id, name, score, 
case 
	when @prev = score then @rank 
	when @prev := score then @rank := @rank + 1
end as rank
FROM record, (SELECT @rank := 0, @prev := -1) init
ORDER BY score DESC, id DESC
```
