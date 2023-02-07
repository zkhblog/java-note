SQL优化从这几个维度思考：  
① explain 分析SQL查询计划（重点关注type、extra、filtered字段）  
② show profile分析，了解SQL执行的线程的状态以及消耗的时间  
③ 索引优化 （覆盖索引、最左前缀原则、隐式转换、order by以及group by的优化、join优化）  
④ 大分页问题优化（延迟关联、记录上一页最大ID）  
⑤ 数据量太大（分库分表、同步到es，用es查询）

```
SELECT
	lb.NAME,
	count( CASE WHEN STATE = 4 THEN 1 ELSE NULL END ) quanlity,
	count( CASE WHEN STATE = 3 THEN 1 ELSE NULL END ) unquanlity --,
	STEP_ID,
	timeunion,
CASE
		
		WHEN STEP_ID = 463 THEN
		timeunion ELSE NULL 
	END,
CASE
		
		WHEN STEP_ID = 236 THEN
		timeunion ELSE NULL 
	END,
CASE
		
		WHEN STEP_ID = 238 THEN
		timeunion ELSE NULL 
	END 
	FROM
		CF_TQM_AbnormalFeedbackReview af WITH ( nolock )
		LEFT JOIN lbOrganization lb WITH ( nolock ) ON lb.ID = af.CFAccWorkShop
		INNER JOIN OS_WFENTRY ow WITH ( nolock ) ON af.InstID = ow.ID
		LEFT JOIN (
		SELECT
			CFAccWorkShop,
			STEP_ID,
			CONVERT ( DECIMAL ( 18, 2 ),( AVG( totalTime )/ 60.0 ), 2 ) timeunion 
		FROM
			(
			SELECT
				af.CFAccWorkShop,
				af.InstID,
				oh.STEP_ID,
				SUM(
				DATEDIFF( MINUTE, oh.START_DATE, oh.FINISH_DATE )) AS totalTime 
			FROM
				CF_TQM_AbnormalFeedbackReview af WITH ( nolock )
				INNER JOIN OS_WFENTRY ow WITH ( nolock ) ON af.InstID = ow.ID 
				AND ow.STATE = 4
				INNER JOIN OS_HISTORYSTEP oh WITH ( nolock ) ON oh.ENTRY_ID = ow.ID 
				AND oh.STEP_ID IN ( 463, 236, 238 ) 
			WHERE
				DATEPART ( YEAR, af.CFBillDate )= 2022 
				AND DATEPART ( MONTH, af.CFBillDate )= 8 
			GROUP BY
				af.CFAccWorkShop,
				oh.STEP_ID,
				af.InstID 
			) AS tmp 
		GROUP BY
			CFAccWorkShop,
			STEP_ID 
		) t ON t.CFAccWorkShop = af.CFAccWorkShop 
	WHERE
		DATEPART ( YEAR, af.CFBillDate )= 2022 
		AND DATEPART ( MONTH, af.CFBillDate )= 8 
	GROUP BY
		lb.NAME,
	timeunion,
	STEP_ID;
```