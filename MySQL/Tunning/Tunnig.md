-- Some queries that i use to tunning

-- It show quantity of connections to database 
SELECT 
  `DB`,
  -- USER,
  COUNT(ID) AS QTD_Conections
--  `USER`, 
--  `HOST`,  
--  db,
 -- COMMAND, 
 -- TIME, 
 -- STATE, 
 -- INFO   
 -- COUNT(USER) 
FROM INFORMATION_SCHEMA.PROCESSLIST 
GROUP BY `DB` -- , USER   -- , `USER`, `HOST` -- , `ID` , COMMAND, TIME, STATE, INFO 
ORDER BY QTD_Conections DESC;


-- It show SQLs in execution real time 
SELECT 
  pl.USER AS User,
  pl.DB AS Database, 
  SUBSTR(pl.HOST,1,(INSTR(pl.HOST, ':')-1) ) AS `Host`, 
  pl.COMMAND AS Command, 
  DATE_FORMAT (tran.trx_started, ' %h:%m:%s') AS StartTime,
  pl.time AS ExecutionTime,
  pl.ID AS TREAD_ID,
  tran.trx_query AS QUERY    
FROM INFORMATION_SCHEMA.INNODB_TRX tran
	JOIN INFORMATION_SCHEMA.PROCESSLIST pl ON tran.trx_mysql_thread_id = pl.ID;
	

