SELECT PERIOD_DIFF(201710, 201703);
返回两个时期之间的差异

SELECT CONCAT_WS("-", "SQL", "Tutorial", "is", "fun!") AS ConcatenatedString;
将几个表达式连接在一起，并在它们之间添加“ - ”分隔符。将跳过具有NULL值的表达式

SELECT LEFT("SQL Tutorial", 3) AS ExtractString;
从字符串中提取3个字符（从左侧开始）