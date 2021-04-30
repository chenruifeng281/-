1.多表left join是会生成一张临时表，并返回给用户
2.where条件是针对最后生成的这张临时表进行过滤，过滤掉不符合where条件的记录，是真正的不符合就过滤掉。
3.on条件是对left join的右表进行条件过滤，但依然返回左表的所有行，右表中没有的补为NULL
4.on条件中如果有对左表的限制条件，无论条件真假，依然返回左表的所有行,但是会影响右表的匹配值。也就是说on中左表的限制条件只影响右表的匹配内容，不影响返回行数。
结论：
    1.where条件中对左表限制，不能放到on后面
    2.where条件中对右表限制，放到on后面，会有数据行数差异，比原来行数要多
测试：
创建两张表：
CREATE TABLE a
(
id INT,
name VARCHAR(20)
);
insert  into a(`id`,`name`) values (1,'a11');
insert  into a(`id`,`name`) values (2,'a22');
insert  into a(`id`,`name`) values (3,'a33');
insert  into a(`id`,`name`) values (4,'a44');
 
CREATE TABLE b
(
id INT,
local VARCHAR(20)
);
insert  into b(`id`,`local`) values (1,'beijing');
insert  into b(`id`,`local`) values (2,'shanghai');
insert  into b(`id`,`local`) values (5,'chongqing');
insert  into b(`id`,`local`) values (6,'tianjin');


测试01：返回左表所有行，右表符合on条件的原样匹配，不满足条件的补NULL
SELECT a.id,a.name,b.local FROM a LEFT JOIN b ON a.id=b.id;
![MG)5}12$8AM4 {LJ$O}5~C](https://user-images.githubusercontent.com/83159371/115987431-b2c1b600-a5e7-11eb-868d-3b807bdb2920.png)

测试02：on后面增加对右表的限制条件：b.local='beijing'
结论02：左表记录全部返回，右表筛选条件生效
SELECT a.id,a.name,b.local FROM a LEFT JOIN b ON a.id=b.id and b.local='beijing';
![image](https://user-images.githubusercontent.com/83159371/115987519-f4eaf780-a5e7-11eb-9194-48c549975d54.png)


测试03：只在where后面增加对右表的限制条件：b.local='beijing'
结论03：针对右表，相同条件，在where后面是对最后的临时表进行记录筛选，行数可能会减少；在on后面是作为匹配条件进行筛选，筛选的是右表的内容。
![image](https://user-images.githubusercontent.com/83159371/115987568-2794f000-a5e8-11eb-8a80-60d4e712c05e.png)


测试04：a.name='a11' 或者 a.name='a33'
结论04：on中对左表的限制条件，不影响返回的行数，只影响右表的匹配内容
SELECT a.id,a.name,b.local FROM a LEFT JOIN b ON a.id=b.id and a.name='a11'; 
![image](https://user-images.githubusercontent.com/83159371/115987610-5b701580-a5e8-11eb-9de6-1cde803f640e.png)

SELECT a.id,a.name,b.local FROM a LEFT JOIN b ON a.id=b.id and a.name='a33';
![image](https://user-images.githubusercontent.com/83159371/115987635-78a4e400-a5e8-11eb-863b-79fed6cfeabd.png)


测试05：where a.name='a33' 或者 where a.name='a22'
结论05：where条件是在最后临时表的基础上进行筛选，显示只符合最后where条件的行
SELECT a.id,a.name,b.local FROM a LEFT JOIN b ON a.id=b.id where a.name='a33'; 
![image](https://user-images.githubusercontent.com/83159371/115987666-a0944780-a5e8-11eb-8c43-99f23954a3cd.png)

SELECT a.id,a.name,b.local FROM a LEFT JOIN b ON a.id=b.id where a.name='a22';
![image](https://user-images.githubusercontent.com/83159371/115987699-c0c40680-a5e8-11eb-9f01-7e9526d7d41a.png)
