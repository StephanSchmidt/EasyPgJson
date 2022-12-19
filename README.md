A small pre-parser for Postgres SQL for Go.

It turns

```sql
WITH tasks AS $(
 SELECT
   t.id id,
   t.title title,
   u.name name,
   s.status status
   FROM tasks t, users u, status s
   WHERE t.user_id=u.id AND t.status_id=s.id
   AND t.user_id=$1
)
$$(
  'tasks',  (SELECT * from tasks)
)
```

into 

```sql
WITH tasks AS ( 
  select json_agg(row_to_json(t)) from (                          
    SELECT
        t.id id,
        t.title title,
        u.name name,
        s.status status
        FROM tasks t, users u, status s
        WHERE t.user_id=u.id AND t.status_id=s.id
        AND t.user_id=$1
    ) t )
select json_build_object(
         'tasks',  (SELECT * from tasks)
)
```

and makes creating SQL that returns JSON more readable.

The API is

```go
	sql := easypgjson.ToSql(....)
```