## 说明

## 目录

## 基础知识

### ROWID与ROWNUM的区别

> - ROWID表示的是每行数据存储的物理地址，是在每条数据插入时生成的
>
> - ROWNUM表示每次查询出的结果集中每条数据的行号，从1开始，常用来做分页，这里有个坑，如执行如下查询语句，是不会有结果的
>
>   ```sql
>   SELECT * FROM STU WHERE ROWNUM>1 AND ROWNUM<5
>   ```
>
>   原因：oracle查询产生的结果集第一条数据ROWNUM=1，oracle判断该条数据不符合条件，则将该数据从结果集移除，则下一条数据的ROWNUM又是1，则不满足查询条件，最终导致一条数据也查询不到
>
>   解决办法：嵌套一层，里层将数据的ROWNUM一起查出，外层再进行条件判断即可
>
>   方式1：
>
>   ```sql
>   select t.* from
>   (SELECT s.*,ROWNUM rn FROM STU s) t
>   where t.rn>1 and t.rn<5
>   ```
>
>   方式2：
>
>   ```sql
>   select t.* from
>   (SELECT s.*,ROWNUM rn FROM STU s where ROWNUM<5) t
>   where t.rn>1
>   ```
>
>   这样可以减少查询的结果，加快查询速度，因为内层查询限制了行号
>
> - 

