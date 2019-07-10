---
layout:     post
title:      "多个Pandas 的 Dataframe 的join 和merge"
subtitle:   " \"不忘初心\""
date:       2019/7/9
author:     "Mason"
header-img: "img/in-post/pandas.jpg"
catalog: true
tags:
    - python
    - Pandas
---



## 前言

这次参加群里的实战，利用pandas 进行股票数据清洗与分析，中间遇到太多场景

- 两个Dataframe的合并，类似数据库两个表join，有一列主键key作为关联的唯一性
- 利用Pandas给折线图产生一个特定区间的按月横坐标
- 按照Dataframe中日期列分组统计

根据这些遇到的场景，利用Google在对每一个进行深入的学习并归纳：

### 本次主要整理Dataframe的合并 
<br>

### Join And Merge 

1. 准备数据,准备数据我特意让Dataframe不一样多，这样能更接近一些特殊场景

 ```py
 import pandas as pd

 raw_data1 = {
        "user_id": ["1", "2", "3", "4", "5"],
        "first_name": ["warren", "clyde", "Allen", "Alice", "Ayoung"],
        "last_name": ["ke", "gao", "Ali", "li", "wang"],
    }

 raw_data2 = {
        "user_id": ["4", "5", "6", "7", "8", "9"],
        "first_name": ["Billy", "Brian", "Bran", "Bryce", "Betty", "mason"],
        "last_name": ["liu", "Black", "li", "Brice", "wei", "wei"],
        "age": [25, 26, 27, 28, 30, 31],
    }

 raw_data3 = {
        "user_id": ["1", "2", "3", "4", "5", "7", "8", "9", "10", "11"],
        "score": [51, 15, 15, 61, 16, 14, 15, 1, 61, 16],
    }

 df_one = pd.DataFrame(raw_data1, columns=["user_id", "first_name", "last_name"])
 df_two = pd.DataFrame(raw_data2, columns=["user_id", "first_name", "last_name", "age"])
 df_third = pd.DataFrame(raw_data3, columns=["user_id", "score"])

 ```

3个DataFrame的数据结构如下：

df_one View：

 ||user_id| first_name| last_name|
 | :-----| ----: | :----: |:-----|
|0    |   1   |  warren   |    ke|
|1    |   2   |   clyde   |   gao|
|2    |   3   |   Allen   |   Ali|
|3    |   4   |   Alice   |    li|
|4    |   5   |  Ayoung   |  wang|

df_two View：

| | user_id| first_name |last_name | age|
| :-----| ----: | :----: |:-----|:-----|
|0    |   4 |    Billy |      liu|  25|
|1    |   5 |    Brian |    Black|  26|
|2    |   6 |     Bran |       li|  27|
|3    |   7 |    Bryce |    Brice|  28|
|4    |   8 |    Betty |      wei|  30|
|5    |   9 |    mason |      wei|  31|

df_third View：

| | user_id | score|
| :-----| ----: | :----: |
|0     | 1   |  51|
|1     | 2   |  15|
|2     | 3   |  15|
|3     | 4   |  61|
|4     | 5   |  16|
|5     | 7   |  14|
|6     | 8   |  15|
|7     | 9   |   1|
|8     |10   |  61|
|9     |11   |  16|

> join之后取最多列，数据row方向（按照行堆砌）合并

```py
 >>> df_new_row = pd.concat([df_one, df_two], sort=True)
 >>> print(df_new_row)
```

df_one有5行，df_two有6行，join之前结果为11行,结果table如下：

|  | age | first_name |user_id|last_name|
| :-----| ----: | :----: |:-----|:-----|
|0   |NaN   |  warren | 1   |     ke
|1   |NaN   |   clyde | 2   |    gao
|2   |NaN   |  Allen  | 3   |    Ali
|3   |NaN   |   Alice | 4   |    li
|4   |NaN   |  Ayoung | 5   |   wang
|0  |25.0   |  Billy  | 4   |    liu
|1  |26.0   |  Brian  | 5   |  Black
|2  |27.0   |  Bran   | 6   |     li
|3  |28.0   |  Bryce  | 7   |  Brice
|4  |30.0   |  Betty  | 8   |    wei
|5  |31.0   |  mason  | 9   |    wei

> join之后取最多列，数据col方向取最多行合并,行数取最多df数据行

```py
 df_new_col = pd.concat([df_one, df_two], axis=1)
 print(df_new_col)
```

df_one有5行，df_two有6行，join之前结果为6行，同列名也是重复的,结果table如下：

|  | user_id |first_name| last_name |user_id |first_name| last_name|  age|
| :-----| ----: | :----: |:-----|:-----|:-----|:-----|:-----|
0  |  1   |  warren   |     ke  |4   |   Billy   |    liu   |25|
1  |  2   |   clyde   |    gao  |5   |   Brian   |  Black   |26|
2  |  3   |   Allen   |    Ali  |6   |    Bran   |     li   |27|
3  |  4   |   Alice   |     li  |7   |   Bryce   |  Brice   |28|
4  |  5   |  Ayoung   |   wang  |8   |   Betty   |    wei   |30|
5  |NaN   |     NaN    |    NaN   |9   |   mason    |   wei   |31|


> 根据主键user_id进行关联merg

```py
 df_merge = pd.merge(df_new, df_third, on="user_id")
 print(df_merge)
```

|  |  age |first_name| last_name |user_id | score|
| :-----| ----: | :----: |:-----|:-----|:-----|
| 0|   NaN |     warren|        ke|       1|     51|
| 1|   NaN |      clyde|       gao|       2|     15|
| 2|   NaN |      Allen|       Ali|       3|     15|
| 3|   NaN |      Alice|        li|       4|     61|
| 4|  25.0|      Billy|       liu|       4|     61|
| 5|   NaN |     Ayoung|      wang|       5|     16|
| 6|  26.0|      Brian|     Black|       5|     16|
| 7|  28.0|      Bryce|     Brice|       7|     14|
| 8|  30.0|      Betty|       wei|       8|     15|
| 9|  31.0|      mason|       wei|       9|      1|

> 根据主键user_id进行关联然后 outer join

```py
df_merge_outer = pd.merge(df_one, df_two, on="user_id", how="outer")
print(df_merge_outer)
```

注意列名的变化，个人感觉这个场景使用比较少,结果视图如下：

|  |user_id |first_name_x| last_name_x |first_name_y| last_name_y |  age|
| :-----| ----: | :----: |:-----|:-----|:-----|:-----|
|0     | 1    |   warren     |    ke    |       NaN    |      NaN |   NaN|
|1     | 2    |    clyde     |   gao    |       NaN    |      NaN |   NaN|
|2     | 3    |    Allen     |   Ali    |       NaN    |      NaN |   NaN|
|3     | 4    |    Alice     |    li    |    Billy    |     liu | 25.0|
|4     | 5    |   Ayoung     |  wang    |    Brian    |   Black | 26.0|
|5     | 6    |       NaN     |    NaN    |     Bran    |      li | 27.0|
|6     | 7    |       NaN     |    NaN    |    Bryce    |   Brice | 28.0|
|7     | 8    |       NaN     |    NaN    |    Betty    |     wei | 30.0|
|8     | 9    |       NaN     |    NaN    |    mason    |     wei | 31.0|

> 根据主键user_id进行关联然后 inner  join

```py
 df_merge_inner = pd.merge(df_one, df_two, on="user_id", how="inner")
 print(df_merge_inner)
```

这种场景使用最广，多表关联

|  |user_id |first_name_x| last_name_x |first_name_y| last_name_y |  age|
| :-----| ----: | :----: |:-----|:-----|:-----|:-----|
|0     |  4    |   Alice    |      li    |    Billy    |     liu   |25|
|1     |  5    |  Ayoung    |    wang    |    Brian    |   Black   |26|

> 根据主键user_id进行关联然后 left/right join

```py
 df_merge_left = pd.merge(df_one, df_two, on="user_id", how="left"，suffixes=("_left", "_right"))
 print(df_merge_left)
```

left/right 两者基本写法一致，我就偷个懒。除了join之外请注意重复列的设置别名

|  |user_id |first_name_left| last_name_left |first_name_right| last_name_right |  age|
| :-----| ----: | :----: |:-----|:-----|:-----|:-----|
|0   |   1    |   warren     |     ke    |     NaN   |     NaN|   NaN|
|1   |   2    |    clyde     |    gao    |     NaN   |     NaN|   NaN|
|2   |   3    |    Allen     |    Ali    |     NaN   |     NaN|   NaN|
|3   |   4    |    Alice     |     li    |   Billy   |     liu|  25.0|
|4   |   5    |   Ayoung     |   wang    |   Brian   |   Black|  26.0|

数据合并就整理这么多
