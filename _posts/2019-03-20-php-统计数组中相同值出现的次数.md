---
layout: post
title:  "Morbi a Semper"
date:   2014-08-31 14:36:23
---
## PHP 统计数组中出现相同值的次数

### [array_count_values ](http://php.net/manual/zh/function.array-count-values.php)

> > array_count_values — 统计数组中所有的值
> >
> > **说明**
> >
> > array array_count_values ( array `$input` )
> >
> > **array_count_values()** 返回一个数组： 数组的键是 `input`里单元的值； 数组的值是 input 单元的值出现的次数。
> >
> > **参数**
> >
> > ```php
> > input
> > ```
> >
> > > > 需要统计的数组
> >
> >  **返回值**
> >
> > > > 返回一个关联数组，用 `input` 数组中的值作为键名，该值在数组中出现的次数作为值。

### [array_column](http://php.net/manual/zh/function.array-column.php)

> > array_column — 返回数组中指定的一列
> >
> >  **说明** 
> >
> > array array_column ( array `$input` , [mixed](http://php.net/manual/zh/language.pseudo-types.php#language.types.mixed) `$column_key` [, [mixed](http://php.net/manual/zh/language.pseudo-types.php#language.types.mixed) `$index_key` = null ] )
> >
> > **array_column()** 返回`input`数组中键值为`column_key`的列， 如果指定了可选参数`index_key`，那么`input`数组中的这一列的值将作为返回数组中对应值的键
> >
> > **参数**
> >
> > ```p&#39;h&#39;p
> > input
> > ```
> >
> > > > 需要取出数组列的多维数组。 如果提供的是包含一组对象的数组，只有 public 属性会被直接取出。 为了也能取出 private 和 protected 属性，类必须实现 **__get()** 和 **__isset()** 魔术方法。
> >
> > ```
> > column_key
> > ```
> >
> > > > 需要返回值的列，它可以是索引数组的列索引，或者是关联数组的列的键，也可以是属性名。 也可以是**NULL**，此时将返回整个数组（配合`index_key`参数来重置数组键的时候，非常管用）
> >
> > ```php
> > index_key
> > ```
> >
> > > >  作为返回数组的索引/键的列，它可以是该列的整数索引，或者字符串键值。
> >
> > **返回值**
> >
> > > >  从多维数组中返回单列数组。
> > > >


1.  一维数组

   针对一维数组，直接使用`array_count_values`即可满足需求。

   eg：

   ```php
   $fruit = [
   'apple','orange','banana','banana','apple'
   ];
   
   $res = array_count_values($fruit);
   print_r($res);
   ```

   输出：

```php
Array
(
    [apple] => 2
    [orange] => 1
    [banana] => 2
)
```



2. 二维数组（统计某一固定列中 相同值出现的次数）

   二位数组需要先使用`array_column`取出其中的某一列，然后再使用`array_count_values`统计次数。

   eg：统计*男*、*女*人数

   ```php
   $user = [
   	[
   		'name' => '小华',
   		'sex' => '男',
   		'age' => 25
   	],
   	[
   		'name' => '晓华',
   		'sex' => '女',
   		'age' => 18
   	],
   	[
   		'name' => '小王',
   		'sex' => '男',
   		'age' => 20
   	],
   	[
   		'name' => '小莉',
   		'sex' => '女',
   		'age' => 18
   	],
   	[
   		'name' => '小明',
   		'sex' => '男',
   		'age' => 25
   	]
   ];
   
   $res = array_count_values(array_column($user, 'sex'));
   print_r($res);
   ```

   输出：

   ```php
   Array
   (
       [男] => 3
       [女] => 2
   )
   ```
