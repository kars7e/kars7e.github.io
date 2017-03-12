---
date: 2010-05-16T18:39:00Z
tags: []
title: NOT IN operator on Oracle 10g Express Edition
tumblr_url: http://blog.bigfun.eu/post/604142323/not-in-operator-on-oracle-10g-express-edition
url: /2010/05/16/not-in-operator-on-oracle-10g-express-edition/
---

Today i had quite annoying problem with query on Oracle 10g. What annoyed me even more is that query was so short:


select department_name from departments where departments.department_id not in (employees.department_id from employees group by department_id );


And made so many problems….
This statement was supposed to return all departments which do not have any employees. Sounds easy, hmm? So it was for me, but… it returned nothing, and i knew there are at least 8 such departments.

After throwing away theory about data errors, i started googling. I found some info about NOT IN operator. Shortly, if it gets set of values which NULL inside, it will fail to return anything. i checked this and… yep, there was 1 (literally: ONE) employee that had no department (department_id was NULL). Argh.
Anyway, the fixed version of query:


select department_name from departments where departments.department_id not in (select NVL(employees.department_id,0) from employees group by department_id );


Remember: always ensure that you are not getting NULL values in NOT IN subqueries!
