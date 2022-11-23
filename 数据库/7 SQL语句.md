SQL优化从这几个维度思考：  
① explain 分析SQL查询计划（重点关注type、extra、filtered字段）  
② show profile分析，了解SQL执行的线程的状态以及消耗的时间  
③ 索引优化 （覆盖索引、最左前缀原则、隐式转换、order by以及group by的优化、join优化）  
④ 大分页问题优化（延迟关联、记录上一页最大ID）  
⑤ 数据量太大（分库分表、同步到es，用es查询）