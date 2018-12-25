[TOC]

## ThinkPHP 3.X 实现多字段 OR 查询

### user 数据表结构

| 字段名 | 类型 | 说明                 |
| ------ | ---- | -------------------- |
| id     | Int  | 表主键               |
| sex    | Int  | 性别: 1=> 男 , 2=>女 |
| age    | Int  | 年龄                 |

### 查询年龄大于20，或者性别为女的数据

```php
$cond = [
	'age' => array('gt', 20),
	'sex' => 2,
	'_logic' => 'OR'
];

$where = [
	'_complex' => $cond
];

$data = M('user')->where($where)->select();
```

