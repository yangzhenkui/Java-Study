# MySQL数据习题练习

一、数据库字段说明

1、学生表 Student(SId,Sname,Sage,Ssex)
SId ：学生编号
Sname：学生姓名
Sage ：出生年月
Ssex：学生性别

2、课程表 Course(CId,Cname,TId)
CId ：课程编号
Cname ：课程名称
TId ：教师编号

3、教师表 Teacher(TId,Tname)
TId ：教师编号
Tname ：教师姓名

4、成绩表 SC(SId,CId,score)
SId ：学生编号
CId ：课程编号
score： 分数

二、插入表及数据

```mysql
# 学生表 Student：
create table Student(
SId varchar(10) ,
Sname varchar(10),
Sage datetime,
Ssex varchar(10));

insert into Student values('01' , '赵雷' , '1990-01-01' , '男');
insert into Student values('02' , '钱电' , '1990-12-21' , '男');
insert into Student values('03' , '孙风' , '1990-05-20' , '男');
insert into Student values('04' , '李云' , '1990-08-06' , '男');
insert into Student values('05' , '周梅' , '1991-12-01' , '女');
insert into Student values('06' , '吴兰' , '1992-03-01' , '女');
insert into Student values('07' , '郑竹' , '1989-07-01' , '女');
insert into Student values('09' , '张三' , '2017-12-20' , '女');
insert into Student values('10' , '李四' , '2017-12-25' , '女');
insert into Student values('11' , '李四' , '2017-12-30' , '女');
insert into Student values('12' , '赵六' , '2017-01-01' , '女');
insert into Student values('13' , '孙七' , '2018-01-01' , '女');
```

```mysql
# 课程表 Course
create table Course(
CId varchar(10),
Cname nvarchar(10),
TId varchar(10)); 

insert into Course values('01' , '语文' , '02'); 
insert into Course values('02' , '数学' , '01'); 
insert into Course values('03' , '英语' , '03'); 
```

```mysql
# 教师表 Teacher
create table Teacher(
TId varchar(10),
Tname varchar(10)); 

insert into Teacher values('01' , '张三');
insert into Teacher values('02' , '李四'); 
insert into Teacher values('03' , '王五'); 
```

```mysql
# 成绩表 SC

create table SC(
SId varchar(10),
CId varchar(10),
score decimal(18,1)); 

insert into SC values('01' , '01' , 80); 
insert into SC values('01' , '02' , 90); 
insert into SC values('01' , '03' , 99); 
insert into SC values('02' , '01' , 70); 
insert into SC values('02' , '02' , 60); 
insert into SC values('02' , '03' , 80); 
insert into SC values('03' , '01' , 80); 
insert into SC values('03' , '02' , 80); 
insert into SC values('03' , '03' , 80); 
insert into SC values('04' , '01' , 50); 
insert into SC values('04' , '02' , 30); 
insert into SC values('04' , '03' , 20); 
insert into SC values('05' , '01' , 76); 
insert into SC values('05' , '02' , 87); 
insert into SC values('06' , '01' , 31); 
insert into SC values('06' , '03' , 34); 
insert into SC values('07' , '02' , 89); 
insert into SC values('07' , '03' , 98); 
```

三、题目与答案

1.查询" 01 "课程比" 02 "课程成绩高的学生的信息及课程分数

```mysql
select *
from Student a  INNER JOIN SC b
on a.SId=b.SId
inner join SC c
on a.sid=c.sid AND b.CId=01 and c.CId=02
where b.score > c.score
-- 需要比较分数，所以得有两列的分数来进行相应的比较，所以需要用到两个表
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201128191249291-766805825.png)

 1.1查询同时存在" 01 "课程和" 02 "课程的情况

```mysql
SELECT *
FROM
(select * from  SC where cid='01') a inner join 
(select * from  SC where cid='02') b on a.sid=b.sid
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201128173745970-255587698.png)

1.2 查询存在" 01 "课程但可能不存在" 02 "课程的情况(不存在时显示为 null ) 

```mysql
select *
FROM SC a left join SC b
on a.SId=b.SId and b.cid='02'
WHERE a.cid='01'
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201128174630772-1699092862.png)

1.3 查询不存在"01 "课程但存在" 02 "课程的情况

```mysql
select *
FROM (SELECT * FROM SC WHERE sid not in(SELECT sid FROM SC WHERE cid='01')) a inner join SC b
on a.SId=b.SId and b.cid='02'
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201128175222524-1409238001.png)

2.查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩（涉及到两张表）

```mysql
SELECT  a.sid,b.sname,a.avg_score
FROM 
(SELECT sid,avg(score) as avg_score
FROM SC
GROUP BY sid 
HAVING avg_score>=60) a left join Student b
on a.sid=b.sid ,

-- 思路：先根据sid进行分组从sc表中筛选出符合平均分大于60的学生，
-- 并单独的作为一个表与student通过sid字段进行连接即可
select a.sid, a.sname,b.sc_avg from student a join 
(select  sid, avg(score) sc_avg from sc group by sid having sc_avg>=60) b 
on a.sid = b.sid;
+------+--------+----------+
| sid  | sname  | sc_avg   |
+------+--------+----------+
| 01   | 赵雷   | 89.66667 |
| 02   | 钱电   | 70.00000 |
| 03   | 孙风   | 80.00000 |
| 05   | 周梅   | 81.50000 |
| 07   | 郑竹   | 93.50000 |
+------+--------+----------+
```



 3.查询在 SC 表存在成绩的学生信息

```mysql
select * from
Student
where sid in(SELECT sid from SC)

-- 学生信息需要查询student表，成绩查询sc表
select * from student where sid in(select sid from sc);
+------+--------+---------------------+------+
| SId  | Sname  | Sage                | Ssex |
+------+--------+---------------------+------+
| 01   | 赵雷   | 1990-01-01 00:00:00 | 男   |
| 02   | 钱电   | 1990-12-21 00:00:00 | 男   |
| 03   | 孙风   | 1990-05-20 00:00:00 | 男   |
| 04   | 李云   | 1990-08-06 00:00:00 | 男   |
| 05   | 周梅   | 1991-12-01 00:00:00 | 女   |
| 06   | 吴兰   | 1992-03-01 00:00:00 | 女   |
| 07   | 郑竹   | 1989-07-01 00:00:00 | 女   |
+------+--------+---------------------+------+
7 rows in set (0.00 sec)
```



4. 查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩(没成绩的显示为 null )

```mysql
select 
a.sid,
a.sname,
count(b.cid) as 'scount', sum(b.score) as 'sum'
from Student a left join SC b on a.sid=b.sid
GROUP BY a.sid 

-- 使用聚集函数sum，count计算总分，先根据sc表根据sid进行分组筛选出所需的字段
-- 定义为一张新的表与student进行关联查询
select a.sid, a.sname, b.cnt, b.total_score 
from student a left join 
(select sid, sum(score) total_score, count(cid)cnt from sc group by sid) b 
on a.sid =b.sid;
+------+--------+------+-------------+
| sid  | sname  | cnt  | total_score |
+------+--------+------+-------------+
| 01   | 赵雷   |    3 |       269.0 |
| 02   | 钱电   |    3 |       210.0 |
| 03   | 孙风   |    3 |       240.0 |
| 04   | 李云   |    3 |       100.0 |
| 05   | 周梅   |    2 |       163.0 |
| 06   | 吴兰   |    2 |        65.0 |
| 07   | 郑竹   |    2 |       187.0 |
| 09   | 张三   | NULL |        NULL |
| 10   | 李四   | NULL |        NULL |
| 11   | 李四   | NULL |        NULL |
| 12   | 赵六   | NULL |        NULL |
| 13   | 孙七   | NULL |        NULL |
+------+--------+------+-------------+
12 rows in set (0.00 sec)
```



4.1 查有成绩的学生信息

```mysql
select * from
Student
where sid in(SELECT sid from SC)

select  * from student where sid in (select sid from sc);
+------+--------+---------------------+------+
| SId  | Sname  | Sage                | Ssex |
+------+--------+---------------------+------+
| 01   | 赵雷   | 1990-01-01 00:00:00 | 男   |
| 02   | 钱电   | 1990-12-21 00:00:00 | 男   |
| 03   | 孙风   | 1990-05-20 00:00:00 | 男   |
| 04   | 李云   | 1990-08-06 00:00:00 | 男   |
| 05   | 周梅   | 1991-12-01 00:00:00 | 女   |
| 06   | 吴兰   | 1992-03-01 00:00:00 | 女   |
| 07   | 郑竹   | 1989-07-01 00:00:00 | 女   |
+------+--------+---------------------+------+
7 rows in set (0.00 sec)
```



5. 查询「李」姓老师的数量

```mysql
SELECT count(1) FROM Teacher
where tname LIKE '李%'

select count(*) from teacher where tname like "李%";
+----------+
| count(*) |
+----------+
|        1 |
+----------+
1 row in set (0.00 sec)
```



6. 查询学过「张三」老师授课的同学的信息

```mysql
SELECT a.*
FROM Student a left join SC b on a.sid=b.sid
left join Course c on b.cid=c.cid
left join Teacher d on c.tid=d.tid
WHERE d.tname='张三'

-- 涉及到student、teacher、course三张表
-- 通过中间表sc讲teacher与course连接起来，并筛选出张三老师的课程
-- 最后通过student筛选字段进行连接查询
select * from student where sid in
(select b.sid from course a join teacher t on a.tid = t.tid join sc b on a.cid = b.cid and t.tname = "张三");
+------+--------+---------------------+------+
| SId  | Sname  | Sage                | Ssex |
+------+--------+---------------------+------+
| 01   | 赵雷   | 1990-01-01 00:00:00 | 男   |
| 02   | 钱电   | 1990-12-21 00:00:00 | 男   |
| 03   | 孙风   | 1990-05-20 00:00:00 | 男   |
| 04   | 李云   | 1990-08-06 00:00:00 | 男   |
| 05   | 周梅   | 1991-12-01 00:00:00 | 女   |
| 07   | 郑竹   | 1989-07-01 00:00:00 | 女   |
+------+--------+---------------------+------+
6 rows in set (0.00 sec)
```



7. 查询没有学全所有课程的同学的信息

```mysql
SELECT *
FROM Student a left join 
SC b on a.sid=b.sid
GROUP BY a.sid
HAVING count(b.cid)<(SELECT COUNT(*) FROM Course)

-- 3 应该使用select * from course
select * from student a left join (select sid, count(cid) cnt from sc group by sid having cnt<3) b on a.sid = b.sid;
+------+--------+---------------------+------+------+------+
| SId  | Sname  | Sage                | Ssex | sid  | cnt  |
+------+--------+---------------------+------+------+------+
| 05   | 周梅   | 1991-12-01 00:00:00 | 女   | 05   |    2 |
| 06   | 吴兰   | 1992-03-01 00:00:00 | 女   | 06   |    2 |
| 07   | 郑竹   | 1989-07-01 00:00:00 | 女   | 07   |    2 |
| 01   | 赵雷   | 1990-01-01 00:00:00 | 男   | NULL | NULL |
| 02   | 钱电   | 1990-12-21 00:00:00 | 男   | NULL | NULL |
| 03   | 孙风   | 1990-05-20 00:00:00 | 男   | NULL | NULL |
| 04   | 李云   | 1990-08-06 00:00:00 | 男   | NULL | NULL |
| 09   | 张三   | 2017-12-20 00:00:00 | 女   | NULL | NULL |
| 10   | 李四   | 2017-12-25 00:00:00 | 女   | NULL | NULL |
| 11   | 李四   | 2017-12-30 00:00:00 | 女   | NULL | NULL |
| 12   | 赵六   | 2017-01-01 00:00:00 | 女   | NULL | NULL |
| 13   | 孙七   | 2018-01-01 00:00:00 | 女   | NULL | NULL |
+------+--------+---------------------+------+------+------+
12 rows in set (0.00 sec)
 
```

8. 查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息

```mysql
SELECT DISTINCT b.*
from SC a inner join Student b on a.sid=b.sid
WHERE a.cid in(SELECT cid from SC where sid='01')

-- 一般来说看到至少两个字  主要用any关键字
select distinct  a.* from student a join sc b on a.sid = b.sid where b.cid=any(select cid from sc where sid = '01');
+------+--------+---------------------+------+
| SId  | Sname  | Sage                | Ssex |
+------+--------+---------------------+------+
| 01   | 赵雷   | 1990-01-01 00:00:00 | 男   |
| 02   | 钱电   | 1990-12-21 00:00:00 | 男   |
| 03   | 孙风   | 1990-05-20 00:00:00 | 男   |
| 04   | 李云   | 1990-08-06 00:00:00 | 男   |
| 05   | 周梅   | 1991-12-01 00:00:00 | 女   |
| 06   | 吴兰   | 1992-03-01 00:00:00 | 女   |
| 07   | 郑竹   | 1989-07-01 00:00:00 | 女   |
+------+--------+---------------------+------+
7 rows in set (0.00 sec)
```

9. 查询和" 01 "号的同学学习的课程 完全相同的其他同学的信息

```mysql
SELECT b.*
FROM
(select * FROM SC where sid not in (SELECT sid FROM SC where cid not in (SELECT cid FROM SC WHERE sid='01') )and sid!='01' )a
left join Student b on a.sid=b.sid 
GROUP BY a.sid
HAVING count(cid)=(SELECT count(cid) FROM SC WHERE sid='01')

-- 待定
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201128182833622-1643034667.png)

10. 查询没学过"张三"老师讲授的任一门课程的学生姓名

```mysql
SELECT sid,sname FROM Student WHERE sid not in 
(SELECT  DISTINCT a.sid
FROM Student a left join SC b on a.sid=b.sid
left join Course c on b.cid=c.cid
left join Teacher d on c.tid=d.tid
WHERE d.tname='张三')
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201124161409608-80785702.png)

11. 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩

```mysql
select a.sid,
b.sname,
AVG(score) as avg_score
from SC a 
left join Student b
on a.sid=b.sid  INNER JOIN (select sid
FROM SC where score<60
GROUP BY sid
having COUNT(1)>1) c on
a.sid=c.sid
group by a.sid
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201128183042260-957465118.png)

12. 检索" 01 "课程分数小于 60，按分数降序排列的学生信息

```mysql
SELECT a.* ,b.score
FROM Student  a left join SC b on a.sid=b.sid 
WHERE b.cid='01' and b.score<60 
ORDER BY b.score desc 
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201124162744515-1871348127.png)

13. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩

```mysql
SELECT a.*,avg_score
FROM SC a left join 
(SELECT sid,avg(score) as avg_score 
FROM SC
GROUP BY sid ) b on a.sid=b.sid
ORDER BY avg_score desc
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201124163647320-871893400.png)

14.查询各科成绩最高分、最低分和平均分： 以如下形式显示：课程 ID，课程 name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率 及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90 要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列

```mysql
SELECT a.*,b.Cname FROM
(SELECT CId,
MAX(score) as 最高分,
MIN(score) as 最低分,
AVG(score) as 平均分,
COUNT(1)  as 选修人数,
SUM(case when score>=60 then 1 else 0 end) / COUNT(1) as 及格率,
SUM(case when score>=70 and score< 80 then 1 else 0 end) / COUNT(1) as 中等率,
SUM(case when score>=80 and score< 90 then 1 else 0 end) / COUNT(1) as 优良率,
SUM(case when score>=90 then 1 else 0 end) / COUNT(1) as 优秀率
FROM SC
GROUP BY CId) a left join Course b on a.CId=b.CId
ORDER BY 选修人数 DESC,CId ASC
```

15. 按各科成绩进行排序，并显示排名， Score 重复时保留名次空缺

```mysql
SELECT 
sid,cid,score,@rank:=@rank+1 as rk
FROM SC,(SELECT @rank:=0) as t
ORDER BY score desc 
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201128184048380-2034173453.png)

![img](http://cdn.nefu-yzk.top/img/1800906-20201128184212329-2094802992.png)

15.1 按各科成绩进行排序，并显示排名， Score 重复时合并名次

```mysql
select 
*,
case when (@sco=score) then @rank else @rank:=@rank+1 end as rn,
@sco:=score  -- 保存上一次的分数
 from SC ,(select @rank:=0,@sco:=null) as t order by score desc
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201128184822369-971976249.png)

​										 ![img](http://cdn.nefu-yzk.top/img/1800906-20201128184840542-1616418295.png)

16. 查询学生的总成绩，并进行排名，总分重复时保留名次空缺

```mysql
select 
s.*,
case when @sco=scos then '' else @rank:=@rank+1 end as rn ,
@sco:=scos
from 
(select 
sid,sum(score) as scos 
from SC group by sid order by scos desc) s,
(select @rank:=0,@sco:=null) as t
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201128185013816-208852587.png)

16.1 查询学生的总成绩，并进行排名，总分重复时不保留名次空缺

```mysql
select 
a.*,
@RANK:=if(@sco=scos,@rank,@rank+1) as rank,
@sco:=scos
FROM (SELECT sid,sum(score) as scos
FROM SC
GROUP BY sid
ORDER BY scos desc) a,
(SELECT @sco:=null,@rank:=0) b
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201128190801515-2009351282.png)

17. 统计各科成绩各分数段人数：课程编号，课程名称，[100-85]，[85-70]，[70-60]，[60-0] 及所占百分比

```mysql
select *,
sum(case when 0<=score and score <=60 then 1 else 0 end )/count(1) as '[0,60]',
sum(case when 60<score and score <=70 then 1 else 0 end )/count(1) as '[60,70]',
sum(case when 70<score and score <=85 then 1 else 0 end )/count(1) as '[70,85]',
sum(case when 85<score and score <=100 then 1 else 0 end )/count(1) as '[85,100]'
from SC 
group by cid
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201128185254760-82747274.png)

18. 查询各科成绩前三名的记录

```mysql
select a.*
from SC a
where (select count(1) from SC b where a.cid=b.cid and b.score>a.score)<3
ORDER BY cid DESC,score DESC 
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201128190140067-391630936.png)

19. 查询每门课程被选修的学生数

```mysql
SELECT cid,count(1) as cons
from SC
GROUP BY cid
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201124164023789-1566582160.png)

20. 查询出只选修两门课程的学生学号和姓名

```mysql
SELECT a.sid,b.sname FROM
(select SID,COUNT(1)  AS 选课数量
from SC
GROUP BY SID 
HAVING COUNT(1) =2 
) a left join Student b on a.sid=b.sid
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201124164149230-1698648615.png)

21. 查询男生、女生人数

```mysql
SELECT ssex,count(1) as 人数
from Student
GROUP BY ssex
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120153022024-1045430265.png)

22. 查询名字中含有「风」字的学生信息

```mysql
SELECT *
FROM Student
WHERE sname like '%风%'
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120153145050-1344330318.png)

23. 查询同名同性学生名单，并统计同名人数

解题思路：按照姓名分组，姓名形同的情况下按照性别分组统计人数，如果统计人数大于等于1，那说明这个人就是同名同性的

```mysql
SELECT sname,ssex,count(sname)
FROM Student
GROUP BY sname,ssex
HAVING count(sname)>1
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201216141756135-1818021209.png)

24. 查询 1990 年出生的学生名单

```mysql
SELECT * 
FROM Student
WHERE year(sage)='1990'
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120153338089-565558631.png)

25. 查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列

```mysql
SELECT cid,avg(score) as avg_score
FROM SC
GROUP BY cid
ORDER BY avg_score DESC,cid ASC
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120153614757-1454562254.png)

26. 查询平均成绩大于等于 85 的所有学生的学号、姓名和平均成绩

```mysql
SELECT 
a.sid,
b.sname,
a.avg_score
FROM
(SELECT sid,avg(score) as avg_score
FROM SC
GROUP BY sid
HAVING avg(score)>=85 ) a left join Student b
on a.sid=b.sid 
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120154001377-494675484.png)

27. 查询课程名称为「数学」，且分数低于 60 的学生姓名和分数

```mysql
SELECT b.sname, a.score 
FROM SC a
left join Student b on a.sid=b.sid
WHERE a.cid IN
(SELECT cid FROM Course WHERE cname='数学') and a.score<60
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120154532462-1754282635.png)

28. 查询所有学生的课程及分数情况（存在学生没成绩，没选课的情况）

```mysql
SELECT a.*,b.*
FROM Student a
left join SC b
on a.sid=b.sid
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120154725686-489235893.png)

29. 查询任何一门课程成绩在 70 分以上的姓名、课程名称和分数

```mysql
SELECT b.sname,c.cname,a.score
FROM SC a LEFT JOIN Student b 
on a.sid=b.sid LEFT JOIN Course c on a.cid=c.cid
WHERE a.score>70
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120155343868-324665944.png)

30. 查询不及格的课程

```mysql
SELECT distinct cid 
FROM SC
WHERE score<60
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120155701676-143267089.png)

31. 查询课程编号为 01 且课程成绩在 80 分以上的学生的学号和姓名

```mysql
SELECT a.sid,b.sname
FROM
(SELECT sid
from SC
where cid='01' and score>=80) a left JOIN Student b on a.sid=b.sid
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120155945268-872206724.png)

32. 求每门课程的学生人数

```mysql
select cid,
count(1) as con
from SC
group by cid
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120160050831-537226481.png)

33. 成绩不重复，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩

解题思路：4张表连接起来，按成绩排序降序，取第一条记录，就是成绩最高的

```mysql
SELECT a.*,b.score
FROM Student a  INNER JOIN SC  b on a.sid=b.sid
INNER JOIN Course c on b.cid=c.cid 
INNER JOIN Teacher d on c.tid=d.tid
WHERE d.Tname='张三'
ORDER BY b.score desc 
LIMIT 1
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201216143317191-1503341876.png)

34. 成绩有重复的情况下，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩

35. 查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩

36. 查询每门功成绩最好的前两名

37. 统计每门课程的学生选修人数（超过 5 人的课程才统计）。

```mysql
SELECT cid,count(1) as cons 
FROM SC
GROUP BY cid
HAVING count(1)>5
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120102803109-1897968177.png)

38. 检索至少选修两门课程的学生学号

```mysql
SELECT sid,count(1) as cons 
FROM SC
GROUP BY sid
HAVING count(1)>=2
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120102914809-1578055575.png)

39. 查询选修了全部课程的学生信息

```mysql
SELECT * from Student where sid IN
(select sid from  SC  GROUP BY sid HAVING COUNT(*)=
(SELECT count(cid) FROM Course) )
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120103038545-1792716630.png)

40. 查询各学生的年龄，只按年份来算

```mysql
select *, year(NOW())-year(sage) as 年龄
from Student
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120102147685-911335918.png)

41. 按照出生日期来算，当前月日 < 出生年月的月日则，年龄减一

```mysql
select *, TIMESTAMPDIFF(YEAR,sage,NOW()) as 年龄
from Student
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120102248636-351215702.png)

42. 查询本周过生日的学生

```mysql
select *,week(Sage),week(now()) 
from Student 
where week(Sage)=week(now());
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120102350416-524825131.png)

43. 查询下周过生日的学生

```mysql
select *,week(Sage),week(now()) 
from Student 
where week(Sage)=week(now())+1;
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120102431428-1752019084.png)

44. 查询本月过生日的学生

```mysql
select *,month(Sage),month(now()) 
from Student 
where month(Sage)=month(now());
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120102539472-1909632162.png)

45. 查询下月过生日的学生

```mysql
select *,month(Sage),month(now()) 
from Student 
where month(Sage)=month(now())+1;
```

![img](http://cdn.nefu-yzk.top/img/1800906-20201120102654965-515006861.png)

 