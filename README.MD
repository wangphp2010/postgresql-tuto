PostgreSql
=========

PostgreSQL 是一个自由的对象-关系数据库服务器(数据库管理系统)，是从伯克利写的 POSTGRES 软件包发展而来的。


git remote add origin https://github.com/wangphp2010/postgresql-tuto.git

git push -u origin master





### 数据库排名

https://db-engines.com/en/ranking

## 安装
```bash
$ sudo apt-get install postgresql
$ psql --version
```

## 基础

```bash
# 一定要先切换到数据库管理员帐号 postgres
$ sudo su postgres #切换到postgres用户 安装数据库时会自动建立这个用户切换到postgres用户 
$ psql --version
$ psql -l # 一览表
$ createdb blog
$ psql -l
$ psql blog
> help
> \h
> \?
> \l
> \q
> \x # 切换 Expanded display is on/off. 如果数据太多时,把横向显示改为纵向
$ psql blog # 进入blog数据库
> select now();
> select version(); #一定要加分号 
> \q
$ dropdb blog
$ psql -l
 

```

### 问题  

1 要加分号 不加分号psql会认为语句还没有写完,需要继续写知道写到分号;
2 $ dropdb blog 删除数据库时,删除失败,说因为有其他进程正在使用数据库 用 
    SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname='blog' AND pid<>pg_backend_pid();
    来结束语句


## 操作数据表

* create table
* drop table
* psql使用

```bash
$ sudo su postgres # 一定要先切换到数据库管理员帐号 postgres
$ createdb blog
$ psql -l
$ psql blog
> create table posts(title varchar(255), content text ) ;
> \dt # 表一览
> \d posts # 表结构
> alter table psots rename to blogposts;
> \dt
> \q

$ nano db.sql
...
create table posts(title varchar(255), content text ) ;
...
$ psql blog
> \i db.sql 载入并执行sql文件语句
> \dt


```


 ## 常用字段类型

* 数值型
    + integer
    + real 浮点
    + serial 序列型(如id等)
* 文字型
    + char 定义长度后,如果实际不够则会补空格
    + varchar 定义长度后,如果实际不够 不会空格
    + text
* 布尔         
    + boolean
* 日期型
    + date 年月日
    + time 时分秒
    + timestamp 年月日 + 时分秒
* 特色类型
    + Array
    + ip地址.网络地址行(inet)
    + json
    + xml

varchar(n) 和 char(n) 分别是 character varying(n) 和 character(n)的别名
参考

https://www.postgresql.org/docs/10/datatype.html


## 添加表约束 

db.sql

```sql
create table posts(
    id serial primary key  ,
    title varchar(255) not null ,
    content text check(length(content) > 3 ),
    is_draft boolean default TRUE,
    is_del boolean default FALSE ,
    created_date timestamp default 'now' 

);

-- 说明
/*
not null 
unique
check
default
primary key 主键, 自带 不为空, 不重复, 自增


设置自增
CREATE SEQUENCE users_id_seq
START WITH 1 从1开始
INCREMENT BY 1 每次增加1
NO MINVALUE 无最小
NO MAXVALUE  无最大
CACHE 1; 

 cache 说法 https://blog.csdn.net/publishwy/article/details/10498973 
 
alter table posts alter column id set default nextval('users_id_seq');

https://www.postgresql.org/docs/8.1/sql-createsequence.html

https://blog.csdn.net/timo1160139211/article/details/78191470



*/

```




## INSERT语句


* 语法 insert into tablename ( field , ... ) values (value, ... );

```bash

$ psql blog
> \dt
> \d posts

```
```sql

> insert into posts(title,content) values('','');
> insert into posts(title,content) values(NULL,'');
> insert into posts(title,content) values('title1','content1');
> select * from posts;
> insert into posts(title,content) values('title2','content2');
> insert into posts(title,content) values('title3','content3');
> select * from posts;

```


## SELECT语句

user.sql

```sql
create table users(
    id serial primary key,
    player varchar(255) not null ,
    score  real ,
    team varchar(255)

);

# 不要用双引号
insert into users (player , score,team) values
('Curry' , 28.3 , 'warrior' ),
('Haden' , 30.2 , 'rocket' ),
('Adu' ,   25.9 , 'warrior' ),
('Ajean' , 12.0 , 'warrior' ),
('shenggui' , 31.2 , 'chevalier' ),
('whitebian' , 26.3 , 'warrior' ),
('afei' , 19.8 , 'hot fire' );

```

```bash
> select * from users ;
> select player , score  from users ;

```

## WHERE语句 


```sql
> select * from users ;
> select * from users where score > 20 and score <30 ;
> select * from users  where team = 'warrior' ;
> select * from users where team != 'warrior';
> select * from users where  player like 'a%' ;
> select * from users where  player like 'a_' ;  --  以a开头 但是只有两个字符的

```
## 数据抽出选项

* order by
* limit 
* offset


```sql
> select * from users order by score asc ;
> select * from users order by score desc ;
> select * from users order by team , score limit 3 offset 1  ;

```

## 统计抽出数据

* distinct 去重
* sum
* max/min
* group by /having


```sql
> select distinct team from  users; 
> select sum(score) from  users; 
> select max(score) from  users; 
> select min(score) from  users; 
> select  *  from  users where score  = (select max(score) from users )  ;
> select  team , max(score)  from users  group by team having max(score) >= 25  order by max(score) ; --不能用 order by score 


```


## 常用函数

* length
* concat
* alias = as 
* substring
* random

```sql
> select player , length(player) from users ;
> select player , concat(player,'/',team) from users ;
> select player , concat(player,'/',team) as "info of player" from users ;
> select substring(team , 1,1) as "firstLetterOfTeam" from users ;
> select concat('MyTeam:' ,substring(team , 1,1)) as "firstLetterOfTeam" from users ;
> select random() ;
> select * from users order by random()  ; -- 随机排序
> select * from users order by random() limit 1  ;
> select player , length(player) from users where length(player) > 3 ;


```
参考

https://www.postgresql.org/docs/10/functions.html


## 更新与删除

* update [table] set [field = newvalue, ... ] where ... 
* delete from [table] where 


```sql
> update users set score = 29.1 where player ="adu";
> update users set score = score + 1 where team = 'warrior' ; 
> update users set score = score + 1 where team IN ( 'warrior' , 'chevalier' );

> delete from users where score > 30 limit 1 ;

```

## 变更表结构

* alter talbe [table] ... 
* create index .. .
* drop index ... 

```sql

> \d users 
> alter table users add fullname varchar(255) ;
>\d users
> alter table users drop fullname ;
>\d users
> alter table users rename player to nba_player ; 
>\d users
> alter table users alter   nba_player type varchar(100) ; 
>\d users
> create index nba_player_index on users(nba_player) ;
> \d users 
> drop index nba_player_index ;
> \d users 



```


## 操作多个表


twitters.sql
```sql
create table twitters(
    id serial primary key ,
    user_id integer ,
    content varchar(255) not null 

);

insert into twitters(user_id , content ) values
( 1 , '今天又是大声, 可来大的正好' ) , 
( 2 , '今晚又得了60份哈哈' ) , 
( 3 , '获胜咱不怕却谁谁尴尬' ) , 
( 4 , '明年我也可能抓回西部' ) , 
( 5 , '我都20+了怎么球队就是不应呢' ) , 
( 1 , '明年听说有条大雨要来了谁啊' ) ;

```


```sql

> select u.player , t.content from users as u , twitters as t where u.id =t.user_id ; -- 同select * from A cross join B


> select u.player , t.content from users as u , twitters as t where u.id =t.user_id and u.id = 1  ; 
```

### join 用法

https://blog.csdn.net/abcbetty_soft/article/details/54137279



## 使用视图　

视图就是一个select语句,把业务系统中常用的select语句简化成一个类似于表的对象,便于简单读取和开发.
视图是一个虚拟表,即视图所对应的数据不进行实际存储.

在项目中,为了提高数据查询速度,可在表中加入索引index,同时对于经常查询的语句,可以提前建立视图view,方便编码和管理

* create view [viewname] select ... 
* drop view [viewname]

```sql
> select u.player , t.content from users as u , twitters as t where u.id=t.user_id and u.id = 1 ;
> create view curry_twitters as select u.player , t.content from users as u , twitters as t where u.id=t.user_id and u.id = 1 ;
> \dv
> \d  curry_twitters
> CREATE OR REPLACE VIEW curry_twitters as select u.player , t.content from users as u , twitters as t where u.id=t.user_id and u.id = 2 ; -- 修改视图语句
> select * from curry_twitters ;
> drop view curry_twitters
> 


```

## 使用事务(transaction )

+ begin
+ commit 
+ rollback # 回到更新前

```sql
> select * from users;
> begin;
> update users set score = 50 where player = 'Curry' ;
> update users set score = 60 where player = 'Haden' ;
> commit ; 
> select * from users;
> begin;
> update users set score = 0 where player = 'Curry' ;
> update users set score = 0 where player = 'Haden' ;
> rollback ;
> select * from users;

```





