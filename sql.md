# 统计数据重复数量
```sql
select count(*),CategoryId from T_CategoryProperty group by CategoryId order by count(*) desc
```
>使用场景一：统计属性最多的类目