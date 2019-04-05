				
## 1.查询" 01 "课程比" 02 "程成绩高的学生的信息及课程分数
SELECT * FROM student s 
RIGHT JOIN(
	SELECT t1.s_id, class01, class02 from
		(SELECT sc.s_id, sc.score as class01 FROM sc sc WHERE sc.c_id = '01' ) as t1,
		(SELECT sc.s_id, sc.score as class02 FROM sc sc WHERE sc.c_id = '02' ) as t2
	WHERE t1.s_id = t2.s_id 
	AND t1.class01 > t2.class02
) r
ON s.s_id = r.s_id;


SELECT * FROM course;
SELECT * FROM student;
SELECT * FROM sc;
SELECT * FROM teacher;

## 2.查询同时存在" 01 "课程和" 02 "课程的情况
SELECT t1.s_id, t1.score01, t2.score02 FROM
	(SELECT sc.s_id, sc.score as score01 FROM sc WHERE sc.c_id = '01') as t1,
	(SELECT sc.s_id, sc.score as score02 FROM sc WHERE sc.c_id = '02') as t2
WHERE t1.s_id = t2.s_id;


## 3.查询存在" 01 "课程但可能不存在" 02 "课程的情况(不存在时显示为 null )
## 用01的数据左连02的数据就好了
SELECT t1.s_id, t1.score01, t2.score02 FROM
	(SELECT sc.s_id, sc.score as score01 FROM sc WHERE sc.c_id = '01') as t1
LEFT JOIN
	(SELECT sc.s_id, sc.score as score02 FROM sc WHERE sc.c_id = '02') as t2
ON t1.s_id = t2.s_id;


## 4.查询不存在" 01 "课程但存在" 02 "课程的情况
SELECT * FROM sc
WHERE sc.s_id NOT in (
	SELECT s_id from sc
	WHERE c_id = '01'
)
AND c_id = '02';


## 5.查询平均成绩大于等于 60 分的同学的学生编号和学生姓名和平均成绩
## 先查score表,得到平均成绩大于60的数据, 再通过sid关联student表,得到目标数据
SELECT s.s_id, s.s_name, t.avgScore FROM 
student s, 
	(SELECT s_id, AVG(score) as avgScore FROM sc
		GROUP BY s_id
		HAVING AVG(score) > 60
	) t
WHERE s.s_id = t.s_id;


## 6.查询在 SC 表存在成绩的学生信息 
## 思路1 全查出来再去重
SELECT DISTINCT  s.* FROM sc sc, student s
WHERE sc.s_id = s.s_id;

## 思路2 先分组去重完再关联
SELECT s.* FROM student s ,(
	SELECT s_id FROM sc GROUP BY s_id
	) t
WHERE s.s_id = t.s_id;


## 7.查询所有同学的学生编号、学生姓名、选课总数、所有课程的总成绩(没成绩的显示为 null )
SELECT s.s_id, s.s_name, t.count, t.scoreSum FROM student s 
LEFT JOIN
	(SELECT s_id ,COUNT(sc.s_id) AS count, SUM(sc.score) AS scoreSum FROM sc 
		GROUP BY s_id
	) t
ON s.s_id = t.s_id;


# 8. 查有成绩的学生信息
## 先根据成绩表分组,得到s_id , 再跟student表关联
SELECT  s.* FROM student s, (
	SELECT s_id FROM sc GROUP BY s_id
	)t
WHERE t.s_id = s.s_id;


#9. 查询「李」姓老师的数量
SELECT COUNT(t_id) AS count FROM teacher 
WHERE t_name LIKE '李%' 
GROUP BY t_id;


## 10. 查询学过「张三」老师授课的同学的信息
## 先查出张三对应的课程id,通过课程id查出学这门课的学生id,再关联学生表得到学生信息
SELECT s.* FROM student s ,(
	SELECT s_id FROM sc WHERE c_id = (
			SELECT c.c_id FROM teacher t, course c  
			WHERE t.t_name = '张三'
			AND c.t_id = t.t_id
		)
	) t
WHERE s.s_id = t.s_id;


## 11. 查询没有学全所有课程的同学的信息
## 查出学全的学生id , 用学生表去过滤即可
SELECT * FROM student
WHERE s_id  NOT IN (
	SELECT s_id FROM
		(SELECT COUNT(c_id)  as countAll FROM course) a,
		(SELECT s_id, COUNT(s_id) as count FROM sc GROUP BY s_id) b
			WHERE  b.count >= a.countAll
);


## 12. 查询至少有一门课与学号为" 01 "的同学所学相同的同学的信息
## 先查出01同学所学的所有课程id,将其他同学所学的课程id与之过滤,得到符合条件的学生id,关联学生表查询即可
SELECT s.* FROM student s,(
	SELECT s_id FROM sc 
	WHERE c_id in(SELECT c_id FROM sc WHERE s_id = '01') 
	GROUP BY s_id
	)t
WHERE s.s_id = t.s_id;


## 13. 查询和" 01 "号的同学学习的课程   完全相同的其他同学的信息
# 不会


## 14. 查询没学过"张三"老师讲授的任一门课程的学生姓名
# 多表联查, 先查出学过张三的课的s_id, 用学生表去过滤
SELECT * FROM student s
WHERE s.s_id NOT IN(
	SELECT sc.s_id FROM course c, sc sc, 	teacher t
	WHERE t.t_name = '张三'
	AND c.t_id = t.t_id
	AND sc.c_id = c.c_id
);


## 15. 查询两门及其以上不及格课程的同学的学号，姓名及其平均成绩
# 根据s_id进行分组, 查询score < 60的 , 通过having(count(*)) >= 2 判断两门以上不及格的 
SELECT  s.s_id, s.s_name , AVG(sc.score) as avgScore FROM student s, sc  
WHERE 
	s.s_id = sc.s_id
	AND sc.score < 60
GROUP BY sc.s_id
HAVING COUNT(*) >= 2;


## 16. 检索" 01 "课程分数小于 60，按分数降序排列的学生信息
SELECT s.* FROM student s, sc
WHERE sc.c_id = '01'
	AND sc.s_id = s.s_id
	AND sc.score < 60
ORDER BY sc.score DESC;


## 17. 按平均成绩从高到低显示所有学生的所有课程的成绩以及平均成绩
# 左连查询, 将平均成绩补充到sc表上, 再按规则排序即可
SELECT  sc.s_id, sc.c_id, sc.score, t.avgScore FROM sc 
LEFT JOIN(
	SELECT sc.s_id, AVG(score) as avgScore FROM sc
	GROUP BY sc.s_id
) t
ON sc.s_id = t.s_id
ORDER BY t.avgScore DESC;


## 18. 查询各科成绩最高分、最低分和平均分：
# 以如下形式显示：课程 ID，课程 name，最高分，最低分，平均分，及格率，中等率，优良率，优秀率
# 及格为>=60，中等为：70-80，优良为：80-90，优秀为：>=90
# 要求输出课程号和选修人数，查询结果按人数降序排列，若人数相同，按课程号升序排列
# 按各科成绩进行排序，并显示排名， Score 重复时保留名次空缺


## 19. 按各科成绩进行排序，并显示排名， Score 重复时合并名次


## 20. 查询学生的总成绩，并进行排名，总分重复时保留名次空缺


## 21. 查询学生的总成绩，并进行排名，总分重复时不保留名次空缺


## 22. 统计各科成绩各分数段人数：课程编号，课程名称，[100-85]，[85-70]，[70-60]，[60-0] 及所占百分比


## 23. 查询各科成绩前三名的记录
SELECT * FROM sc ORDER BY c_id ,score DESC;


## 24. 查询每门课程被选修的学生数
SELECT c_id, count(c_id) AS count FROM sc GROUP BY c_id;


## 25. 查询出只选修两门课程的学生学号和姓名
SELECT s.s_id,s.s_name FROM student s,(
	SELECT s_id FROM sc
	GROUP BY s_id 
	HAVING COUNT(sc.c_id) = 2
) t
WHERE s.s_id = t.s_id;


## 26. 查询男生、女生人数
SELECT s_sex, COUNT(*) as manNum FROM student s 
GROUP BY s.s_sex;


## 27. 查询名字中含有「风」字的学生信息
SELECT * FROM student 
WHERE student.s_name LIKE '%风%';


## 28. 查询同名同姓学生名单，并统计同名人数
SELECT s.s_name, COUNT(*) as count 
FROM student s  
GROUP BY s_name
HAVING COUNT(*) > 1


## 29. 查询 1990 年出生的学生名单
SELECT * FROM student WHERE YEAR(s_brithday) = 1990;


## 30. 查询每门课程的平均成绩，结果按平均成绩降序排列，平均成绩相同时，按课程编号升序排列
SELECT sc.c_id, c.c_name, AVG(score) AS avgScore 
FROM sc , course c 
WHERE sc.c_id = c.c_id
GROUP BY c_id 
ORDER BY avgScore DESC , c_id ASC;


## 31. 查询平均成绩大于等于 85 的所有学生的学号、姓名和平均成绩
SELECT s.s_id, s.s_name, t.avgScore 
FROM student s,(
	SELECT s_id, AVG(score) AS avgScore 
	FROM sc 
	GROUP BY s_id 
	HAVING avgScore > 85
	) t
WHERE s.s_id = t.s_id;


## 32. 查询课程名称为「数学」，且分数低于 60 的学生姓名和分数
SELECT s.s_name, t.score FROM student s,(
	SELECT sc.s_id, sc.score FROM sc , course c
	WHERE c.c_name = '数学' 
	AND c.c_id = sc.c_id
	AND score < 60 
	)t
WHERE s.s_id = t.s_id;


## 33. 查询所有学生的课程及分数情况（存在学生没成绩，没选课的情况）
SELECT s.s_id, sc.c_id, sc.score FROM student s
LEFT JOIN sc
ON s.s_id = sc.s_id;


## 34. 查询任何一门课程成绩在 70 分以上的姓名、课程名称和分数
SELECT s.s_name, c.c_name, sc.score FROM student s, sc ,course c
WHERE s.s_id = sc.s_id 
AND sc.score > 70
AND c.c_id = sc.c_id;


## 35. 查询不及格的课程
SELECT sc.c_id FROM sc 
WHERE sc.score < 60
GROUP BY sc.c_id;


## 36. 查询课程编号为 01 且课程成绩在 80 分以上的学生的学号和姓名
SELECT s.s_id, s.s_name FROM student s , sc sc
WHERE s.s_id = sc.s_id
AND sc.c_id = '01'
AND sc.score >= 80;


## 37. 求每门课程的学生人数
SELECT sc.c_id, COUNT(*)  AS count FROM sc sc
GROUP BY sc.c_id;


## 38. 成绩不重复，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩
SELECT s.*, sc.c_id, sc.score FROM student s , sc sc, teacher t, course c
WHERE s.s_id = sc.s_id
AND t.t_id = c.t_id
AND sc.c_id = c.c_id
AND t.t_name = '张三'
HAVING MAX(sc.score);


## 39. 成绩有重复的情况下，查询选修「张三」老师所授课程的学生中，成绩最高的学生信息及其成绩



## 40. 查询不同课程成绩相同的学生的学生编号、课程编号、学生成绩



## 41. 查询每门功成绩最好的前两名



## 42. 统计每门课程的学生选修人数（超过 5 人的课程才统计）。
SELECT c_id, COUNT(*)  AS count FROM sc
GROUP BY c_id
HAVING count > 5;


## 43. 检索至少选修两门课程的学生学号
SELECT s_id, COUNT(c_id)  AS count FROM sc
GROUP BY s_id
HAVING count >= 2;


## 44. 查询选修了全部课程的学生信息
SELECT s.* FROM student s, sc 
WHERE s.s_id = sc.s_id
GROUP BY sc.s_id
HAVING COUNT(*) = (SELECT DISTINCT count(*) FROM course);


## 45. 查询各学生的年龄，只按年份来算



## 46. 按照出生日期来算，当前月日 < 出生年月的月日则，年龄减一


## 47. 查询本周过生日的学生
SELECT * FROM student s
WHERE WEEKOFYEAR(s.s_brithday) = WEEKOFYEAR(CURDATE());


## 48. 查询下周过生日的学生
SELECT * FROM student s
WHERE WEEKOFYEAR(s.s_brithday) = WEEKOFYEAR(CURDATE()) + 1;


## 49. 查询本月过生日的学生
SELECT * FROM student s
WHERE MONTH(s.s_brithday) = MONTH(CURDATE());


## 50. 查询下月过生日的学生
SELECT * FROM student s
WHERE MONTH(s.s_brithday) = MONTH(CURDATE()) + 1;
