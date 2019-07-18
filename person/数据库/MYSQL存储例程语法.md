# 储存例程语法

## 基础语法


```sql
DROP PROCEDURE IF EXISTS my_procedure;
CREATE PROCEDURE my_procedure (params1 INT)
BEGIN
    ...
END;
```

```sql
call my_procedure(UNIX_TIMESTAMP('2019-07-09 00:00:00'));
```

## 变量定义

- 本地变量声明

```sql
DECLARE var_name [, var_name] ... type [DEFAULT value]
```

- Demo
  
```sql
CREATE PROCEDURE sp1 (x VARCHAR(5))
BEGIN
  DECLARE xname VARCHAR(5) DEFAULT 'bob';
  DECLARE newname VARCHAR(5);
  DECLARE xid INT;

  SELECT xname, id INTO newname, xid
    FROM table1 WHERE xname = xname;
  SELECT newname;
END;
```

!> `DECLARE`声明的变量是不区分大小写的  
`DECLARE`作用域是在`BEGIN ... END`内  
可以在嵌套的`BEGIN ... END`内使用改变量  
可以在内嵌快中重新`DECLARE`进行覆盖
`DECLARE`声明的变量必须在出现在游标（`cursor`）和句柄（`handler`）的`DECLARE`声明之前

- 用户定义变量（会话可用）

> 用户定义变量在同一个mysql会话中都是可用的

- 语法

```sql
SET @var_name = expr [, @var_name = expr] ...
```

- Demo

```sql
SET @t1=1, @t2=2, @t3:=4;
SELECT @t1, @t2, @t3, @t4 := @t1+@t2+@t3;

SET @a = @a + 1;

SELECT @a, @a:=@a+1, ...;

SET @a='test';
SELECT @a,(@a:=20) FROM tbl_name;

SELECT (@aa:=id) AS a, (@aa+3) AS b FROM tbl_name HAVING b=5;
```

!> `SET`变量赋值可以使用`=`或`:=`  
可以将一个用户自定义变量赋值给另一个用户自定义变量，在这种场景下必须用`:=`，因为`=`被当作比较运算符

## 表达式

expr:
- `expr OR expr`
- `expr || expr`
- `expr XOR expr`
- `expr AND expr`
- `expr && expr`
- `NOT expr`
- `! expr`
- `boolean_primary IS [NOT] {TRUE | FALSE | UNKNOWN}`
- `boolean_primary`

boolean_primary:
- `boolean_primary IS [NOT] NULL`
- `boolean_primary <=> predicate`
- `boolean_primary comparison_operator predicate`
- `boolean_primary comparison_operator {ALL | ANY} (subquery)`
- `predicate`

comparison_operator: `=`、`>=`、`>`、`<=`、`<`、`<>`、`!=`

predicate:
- `bit_expr [NOT] IN (subquery)`
- `bit_expr [NOT] IN (expr [, expr] ...)`
- `bit_expr [NOT] BETWEEN bit_expr AND predicate`
- `bit_expr SOUNDS LIKE bit_expr`
- `bit_expr [NOT] LIKE simple_expr [ESCAPE simple_expr]`
- `bit_expr [NOT] REGEXP bit_expr`
- `bit_expr`

bit_expr:
- `bit_expr | bit_expr`
- `bit_expr & bit_expr`
- `bit_expr << bit_expr`
- `bit_expr >> bit_expr`
- `bit_expr + bit_expr`
- `bit_expr - bit_expr`
- `bit_expr * bit_expr`
- `bit_expr / bit_expr`
- `bit_expr DIV bit_expr`
- `bit_expr MOD bit_expr`
- `bit_expr % bit_expr`
- `bit_expr ^ bit_expr`
- `bit_expr + interval_expr`
- `bit_expr - interval_expr`
- `simple_expr`

simple_expr:
- `literal`
- `identifier`
- `function_call`
- `simple_expr COLLATE collation_name`
- `param_marker`
- `variable`
- `simple_expr || simple_expr`
- `+ simple_expr`
- `- simple_expr`
- `~ simple_expr`
- `! simple_expr`
- `BINARY simple_expr`
- `(expr [, expr] ...)`
- `ROW (expr, expr [, expr] ...)`
- `(subquery)`
- `EXISTS (subquery)`
- `{identifier expr}`
- `match_expr`
- `case_expr`
- `interval_expr`



## 流程控制语句

### CASE 语法

> 语法

```sql
CASE case_value
    WHEN when_value THEN statement_list
    [WHEN when_value THEN statement_list] ...
    [ELSE statement_list]
END CASE
```

或

```sql
CASE
    WHEN search_condition THEN statement_list
    [WHEN search_condition THEN statement_list] ...
    [ELSE statement_list]
END CASE
```

> Demo

```sql
DELIMITER |

CREATE PROCEDURE p()
  BEGIN
    DECLARE v INT DEFAULT 1;

    CASE v
      WHEN 2 THEN SELECT v;
      WHEN 3 THEN SELECT 0;
      ELSE
        BEGIN
        END;
    END CASE;
  END;
  |
```

### IF 语法

```SQL
IF search_condition THEN statement_list
    [ELSEIF search_condition THEN statement_list] ...
    [ELSE statement_list]
END IF
```

> Demo

DELIMITER //

```sql
CREATE FUNCTION SimpleCompare(n INT, m INT)
  RETURNS VARCHAR(20)

  BEGIN
    DECLARE s VARCHAR(20);

    IF n > m THEN SET s = '>';
    ELSEIF n = m THEN SET s = '=';
    ELSE SET s = '<';
    END IF;

    SET s = CONCAT(n, ' ', s, ' ', m);

    RETURN s;
  END //

DELIMITER ;
```

```sql
DELIMITER //

CREATE FUNCTION VerboseCompare (n INT, m INT)
  RETURNS VARCHAR(50)

  BEGIN
    DECLARE s VARCHAR(50);

    IF n = m THEN SET s = 'equals';
    ELSE
      IF n > m THEN SET s = 'greater';
      ELSE SET s = 'less';
      END IF;

      SET s = CONCAT('is ', s, ' than');
    END IF;

    SET s = CONCAT(n, ' ', s, ' ', m, '.');

    RETURN s;
  END //

DELIMITER ;
```

### ITERATE 语法

```sql
ITERATE label
```

!> `ITERATE` 能够出现在 `LOOP`, `REPEAT`, 和 `WHILE` 语句中. `ITERATE` 意味着再次开始这个循环。

### LEAVE 语法

```sql
LEAVE label
```

!> 这个语句通常用于退出给定label（标签）的流程控制语句。如果这个语句放在存储例程的最外层，则回退出当前储存例程。

### LOOP 语法

```sql
[begin_label:] LOOP
    statement_list
END LOOP [end_label]
```

> Demo

```sql
CREATE PROCEDURE doiterate(p1 INT)
BEGIN
  label1: LOOP
    SET p1 = p1 + 1;
    IF p1 < 10 THEN
      ITERATE label1;
    END IF;
    LEAVE label1;
  END LOOP label1;
  SET @x = p1;
END;
```

### REPEAT 语法

```sql
[begin_label:] REPEAT
    statement_list
UNTIL search_condition
END REPEAT [end_label]
```

> Demo

```sql
delimiter //

CREATE PROCEDURE dorepeat(p1 INT)
BEGIN
  SET @x = 0;
  REPEAT
    SET @x = @x + 1;
  UNTIL @x > p1 END REPEAT;
END //

DELIMITER ;
```

### RETURN 语法

```sql
RETURN expr
```

!> `RETURN`语句用于**储存函数**执行结束之后返回*expr*值给它的调用者，存储函数必须包含至少一个`RETURN`语句。

!> 这个语句不支持储存过程，触发器，或者事件。`LEAVE`语句能够用于退出这个存储例程。

### WHILE 语法

```sql
[begin_label:] WHILE search_condition DO
    statement_list
END WHILE [end_label]
```

> demo

```sql
CREATE PROCEDURE dowhile()
BEGIN
  DECLARE v1 INT DEFAULT 5;

  WHILE v1 > 0 DO
    ...
    SET v1 = v1 - 1;
  END WHILE;
END;
```


## 游标

> 游标是可嵌套的
> 游标有以下属性
- 敏感：服务器可能会也可能不会复制其结果表
- 只读：支持读操作
- 非跳跃式读：只能沿一个方向读，不可以跳过任何一条记录

!> `DECLARE cur2 CURSOR ...`游标声明语句必须在`DECLARE CONTINUE HANDLER ...`句柄声明语句之前  
而且在`DECLARE b, c INT;`变量和`DECLARE done INT DEFAULT FALSE ...`条件声明语句之后。

- Demo

```sql
CREATE PROCEDURE curdemo()
BEGIN
  DECLARE done INT DEFAULT FALSE;
  DECLARE a CHAR(16);
  DECLARE b, c INT;
  DECLARE cur1 CURSOR FOR SELECT id,data FROM test.t1;
  DECLARE cur2 CURSOR FOR SELECT i FROM test.t2;
  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

  OPEN cur1;
  OPEN cur2;

  read_loop: LOOP
    FETCH cur1 INTO a, b;
    FETCH cur2 INTO c;
    IF done THEN
      LEAVE read_loop;
    END IF;
    IF b < c THEN
      INSERT INTO test.t3 VALUES (a,b);
    ELSE
      INSERT INTO test.t3 VALUES (a,c);
    END IF;
  END LOOP;

  CLOSE cur1;
  CLOSE cur2;
END;
```

### 语法

- 声明游标：`DECLARE cursor_name CURSOR FOR select_statement`
- 打开游标`OPEN cursor_name`
- 读取数据`FETCH [[NEXT] FROM] cursor_name INTO var_name [, var_name] ...`，读取数据之前一定要记得打开游标
- 关闭游标：`CLOSE cursor_name`，1. 如果游标未开启将会有错误发生 ，2.如果未关闭游标将在`BEGIN ... END`结束之后自动关闭

### 条件声明

句柄`HANDLER`能够定义几种类型的条件：**警告**，**异常**，**特定的条件（如特定的错误码）**  
为了声明条件使用语法`DECLARE ... CONDITION`，为了声明句柄`HENDLER`使用语法`DECLARE ... HANDLER`

- 语法

```sql
DECLARE condition_name CONDITION FOR condition_value
```

*condition_value：MYSQL错误码 或者 SQLSTATE [VALUE] sqlstate_value*
- MYSQL错误码：一个指示MYSQL错误的整数，不要使用**0**，因为**0**是一个成功码而不是错误码
- SQLSTATE [VALUE] sqlstate_value： 一个5个字符的`SQLSTATE`值，不要使用`SQLSTATE`值为`'00'`开始的标识符，因为它是成功标识符，而不是失败标识符

- DEMO

```sql
DECLARE CONTINUE HANDLER FOR 1051
  BEGIN
    -- body of handler
  END;
```

```sql
DECLARE no_such_table CONDITION FOR 1051;
DECLARE CONTINUE HANDLER FOR no_such_table
  BEGIN
    -- body of handler
  END;
```  

```sql
DECLARE no_such_table CONDITION FOR SQLSTATE '42S02';
DECLARE CONTINUE HANDLER FOR no_such_table
  BEGIN
    -- body of handler
  END;
```

## HANDLER(句柄)声明

- 语法

```sql
DECLARE handler_action HANDLER
    FOR condition_value [, condition_value] ...
    statement

handler_action:
    CONTINUE
  | EXIT
  | UNDO

condition_value:
    mysql_error_code
  | SQLSTATE [VALUE] sqlstate_value
  | condition_name
  | SQLWARNING
  | NOT FOUND
  | SQLEXCEPTION
```

`DECLARE ... HANDLER`语句用于定义一个句柄处理一个或多个条件。一旦这些条件中的其中一个发生，*statement*语句将被执行，*statement*可以为简单的语句如`SET var_name = value`，或者`BEGIN and END`

!> **HANDLER（句柄）声明语句**必须在**变量声明语句**和**条件声明语句**之后。

*handler_action*指示当HANDLER(句柄)事件被触发时，将执行什么动作：
- `CONTINUE`：继续执行当前的存储程序
- `EXIT`：退出当前`HANDLER声明语句`包裹的`BEGIN ... END`结构体
- `UNDO`： 不支持.

*condition_value*为`DECLARE ... HANDLER`表明激活HANDLER的**条件码**或者**条件类**，一般有以下几种形式：
- mysql_error_code（MYSQL错误码）：一个表明MYSQL错误的整数，如1051 ：unknown table

```sql
DECLARE CONTINUE HANDLER FOR 1051
  BEGIN
    -- body of handler
  END;
```

!> 不要使用0，因为0是表示成功，而不是错误条件，更多错误码请参考：[MYSQL服务器错误码和信息](/person/数据库/MYSQL错误码和信息)

- SQLSTATE [VALUE] sqlstate_value: 一个长度为5个字符的字符串用来表明SQLSTATE的值，例如'42S01'表示“unknown table”:

```sql
DECLARE CONTINUE HANDLER FOR SQLSTATE '42S02'
  BEGIN
    -- body of handler
  END;
```

- condition_name: 一个之前通过`DECLARE ... CONDITION`声明好的条件变量，一个条件名能够跟**MYSQL错误码**或者**SQLSTATE值**进行关联，更多参考[Section 13.6.7.1, “DECLARE ... CONDITION]()

- **SQLWARNING**: 以'01'开头的SQLSTATE值类的简写

```sql
DECLARE CONTINUE HANDLER FOR SQLWARNING
  BEGIN
    -- body of handler
  END;
```

- **NOT FOUND**:以'02'开头的SQLSTATE值类的简写，这与游标的上下文相关：游标到达数据集的结尾。如果没有更多数据可用，一个**SQLSTATE值'02000'**无数据条件事件将被产生，为了检测这样的条件，你可以设置`NOT FOUND`的HANDLER(句柄)条件

```sql
DECLARE CONTINUE HANDLER FOR NOT FOUND
  BEGIN
    -- body of handler
  END;
```

!> 对于其他例子，可以查看`游标相关`章节。`NOT FOUND`也能在SELECT ... INTO变量列表的语句中，当没有任何数据返回时。

- SQLEXCEPTION: 不是以'00', '01', or '02'开头的SQLSTATE值类的简写

```sql
DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
  BEGIN
    -- body of handler
  END;
```


## 代码注释

```sql
SELECT 1+1;     # This comment continues to the end of line

SELECT 1+1;     -- This comment continues to the end of line

SELECT 1 /* this is an in-line comment */ + 1;

SELECT 1+
/*
this is a
multiple-line comment
*/
1;
```

