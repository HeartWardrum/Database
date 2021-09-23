# 存储过程

### 创建存储过程

````T-SQL
--计算两个数的乘积
CREATE PROCEDURE p_multi
	@var1 INT = 5,@var2 INT,@var3 INT OUTPUT
AS
SET @var3 = @var1 * @var2;
````

### 执行存储过程

##### 1.按参数位置传递值(默认值也要写出来)

~~EXEC p_multi 7,@res OUTPUT~~

````T-SQL
DECLARE @res INT
EXEC p_multi 5,7,@res OUTPUT
PRINT @res;
-- 输出 35
````

##### 2.按参数名传递值

````T-SQL
DECLARE @res INT
EXEC p_multi @var2 = 7,@var3 = @res OUTPUT
PRINT @res;
````

### 若将上述两段代码放在一起则要再中间加 'GO'

````T-SQL
CREATE PROC p_multi
@var1 INT = 5,@var2 INT,@var3 INT OUTPUT
AS
SET @var3 = @var1 * @var2
GO
DECLARE @res INT
EXEC p_multi @var2 = 7,@var3 = @res OUTPUT
PRINT @res;
````

### 删除存储过程

````T-SQL
DROP PROC p_multi
````



例2：

````T-SQL
--  删除销售日期在指定年份之前的销售单据明细表中的记录
CREATE PROC p_Delete
@year INT
AS
	DELETE FROM Table_SaleBillDetail
	WHERE SaleBillID IN(
					SELECT SaleBillID FROM Table_SaleBill
					WHERE year(SaleDate) < @year
					);
````

例3：

````T-SQL
--  将指定类别商品的单价降低5%
CREATE PROC p_Update
@class CHAR(10)
AS
	UPDATE Table_Goods SET SaleUnitPrice = SaleUnitPrice * 0.95
	WHERE GoodsClassID IN(
						SELECT GoodsClassID FROM Table_GoodsClass
						WHERE GoodsClassName = @class
						);
````



# GO

* GO 不是T-SQL语句
* GO单独一行
* 当一段代码必须发生在另一段代码之前时  使用 GO



# 用户自定义函数

## 1. 标量函数

* 返回单个数据值

例1：

````T-SQL
--输入长宽高 求体积
CREATE FUNCTION dbo.CubicVolume(@length INT,@width INT,@height INT)
RETURNS INT		--返回类型
AS
BEGIN
	RETURN(@length * @width * @height)
END
GO
--  调用标量函数
SELECT dbo.CubicVolume(3,4,5) AS 立方体体积;
--  删除标量函数
DROP　FUNCTION dbo.CubicVolume;
````

例2：

````T-SQL
--  查询指定商品类别的商品种类数
CREATE FUNCTION dbo.f_GoodsCount(@class varchar(10))
RETURNS INT
AS
BEGIN 
	DECLARE @x INT
	SELECT @x = COUNT(*) FROM Table_GoodsClass a JOIN Table_Goods b
		ON a.GoodsClassID = b.GoodsClassID 
		WHERE GoodsClassName = @class
	RETURN @x
END
GO
--  调用函数  查询”服装“类商品的商品名和种类数
SELECT GoodsName AS 商品名，dbo.f_GoodsCount('服装') AS 种类数
	FROM Table_GoodsClass a JOIN Table_Goods b
	ON a.GoodsClassID = b.GoodsClassID
	WHERE GoodsClassName = '服装';
--  删除标量函数
DROP FUNCTION dbo.f_GoodsCount;
````

## 2. 内联表值函数

* 返回值是一个表，该表的内容是一个查询语句的结果

例：

````T-SQL
--  创建查询指定类别的商品名称和单价的内联表值函数
CREATE FUNCTION f_GoodsInfo(@class CHAR(10))
RETURNS TABLE		-- 返回值类型是表
AS
RETURN(
		SELECT GoodsName,SaleUnitPrice FROM Table_GoodsClass AS a
		JOIN Table_Goods AS b
		ON a.GoodsClassID = b.GoodsClassID
		WHERE GoodsClassName = @class)
GO
--  调用内联表值函数
SELECT * FROM f_GoodsInfo('服装');
````



## 3.多语句表值函数


```T-SQL
/*  定义查询指定类别商品的名称、单价、生产日期和新旧商品的多语句表值函数，其中新旧商品的值为：
生产月数>12个月  ->  旧商品
6个月<生产月数<12个月  ->  一般商品
生产月数<6个月  ->  新商品
*/
CREATE FUNCTION f_GoodsType(@class VARCHAR(20))
RETURNS @f_GoodsType TABLE(
					商品名		VARCHAR(50),
					单价		MONEY,
					生产日期	DATETIME,
					类型		VARCHAR(10)
					)
AS
BEGIN 
INSERT INTO @f_GoodsType
SELECT GoodsName,SaleUnitPrice,ProductionDate,CASE
	WHEN DATEDIFF(MONTH,ProductionDate,'2007/2/10')>12
	THEN '新商品'
	WHEN DATEDIFF(MONTH,ProductionDate,'2007/2/10') BETWEEN 6 AND 12 
	THEN '一般商品'
	WHEN DATEDIFF(MONTH,ProductionDate,'2007/2/10')<6
	THEN '新商品'
END
FROM Table_GoodsClass AS a JOIN Table_Goods AS b
ON a.GoodsClassID = b.GoodsClassID
WHERE GoodsClassName = @class
RETURN 
END
GO
--  调用
SELECT * FROM dbo.f_GoodsType('家用电器');
```

# 权限管理

## 授权语句

GRANT  [^SELECT] ON  [tableName] TO  [userName];

[^SELECT]:或者INSERT  UPDATE  DELETE  REFERENCES  EXECUTE

```T-SQL
--  授权用户User_Name对Table_Name表具有SELECT权限
GRANT SELECT ON Table_Name To User_Name
--  使用GRANT OPTION选项，授予用户Wanida对vEmployee视图中EmployeeID列具有REFERENCES权限
GRANT REFERENCES(EmployeeID) ON vEmployee TO Wanida
WITH GRANT OPTION -- 指示该主体还可以向其他主体授予所指定的权限
```

