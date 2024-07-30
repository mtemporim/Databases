[<-- Back to Tunnig](https://github.com/mtemporim/Databases/tree/main/MySQL/Tunning)


### Some queries that i use to tunning

### It show quantity of connections to schema/database group by schema/database
```sql
SELECT
	USER AS User,
	COALESCE(DB, 'NA') AS `Schema`,
	COUNT(id) AS Sessions
FROM
	information_schema.processlist
GROUP BY
	USER,
	DB
ORDER BY
	COUNT(DB) DESC,
	DB
```


### It show SQLs in execution real time 

```sql
SELECT 
    pl.id AS Thread,
    pl.user AS Usuario,
    pl.HOST AS Host,
    pl.DB AS `Schema`,
    it.trx_started,
    NOW() - it.trx_started AS TimeInSeconds,
    pl.state,
    it.trx_state,
    it.trx_id, 
    it.trx_id AS blocking_trx_id,
    it.trx_mysql_thread_id AS blocking_thread,
    it.trx_query AS blocking_query,
    pl.info AS Query
FROM information_schema.processlist AS pl
  INNER JOIN information_schema.innodb_trx AS it ON pl.id = it.trx_mysql_thread_id
```	