# Oracle 实用sql

## 当前库所用表名和表注释查询

```sql
select t.table_name tableName, f.comments comments
  from user_tables t
 inner join user_tab_comments f
    on t.table_name = f.table_name
```

