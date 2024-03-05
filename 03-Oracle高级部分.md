# Oracle高级部分

# 一、PLSQL编程

&emsp;&emsp;是过程语言(Procedural Language)与结构化查询语言(SQL)结合而成的编程语言.通过增加变量、控制语句，使我们可以写一些逻辑更加复杂的数据库操作.

语法结构

```sql
declare
	--声明变量  变量名称 v_ 开头，规范
begin
   --执行具体的语句
   --异常处理
end;
```

注意：

1. 赋值通过':='完成
2. begin和end之间必须有一行可执行的代码
3. end之后必须跟上';'
4. 如果没有需要声明的变量declare可以省略掉

```sql
declare
    v_hello varchar(20);
begin
    v_hello := 'Hello Oracle';
    dbms_output.put_line(v_hello);
end;

begin
   dbms_output.put_line('hello');
end;
```

> dbms_output不输出的问题。执行如下命令即可
>
> set serveroutput on;

## 1. dbms_output用法

&emsp;&emsp;dbms_output包主要用于调试pl/sql程序，或者在sql*plus命令中显示信息(displaying message)和报表，譬如我们可以写一个简单的匿名pl/sql程序块，而该块出于某种目的使用dbms_output包来显示一些信息。

1. enable：在serveroutput on的情况下，用来使dbms_output生效(默认即打开)
2. disable：在serveroutput on的情况下，用来使dbms_output失效
3. put：将内容写到内存，等到put_line时一起输出
4. put_line：不用多说了，输出字符
5. new_line：作为一行的结束，可以理解为写入buffer时的换行符
6. get_line(value, index)：获取缓冲区的单行信息
7. get_lines(array, index)：以数组形式来获取缓冲区的多行信息

```sql
begin
   dbms_output.put('a1');
   dbms_output.put('b2');
   dbms_output.new_line(); -- 输出缓存中的信息，新起一行
   dbms_output.put_line('aaaaa'); -- 会输出缓存中的信息和当前的信息，不会换行
end;
```

## 2.赋值操作

### 2.1 :=

```java
-- 定义两个变量 v_a,v_b 计算和是多少
declare
    v_a number(3); --- 声明变量
    v_b number(3) :=20 ; -- 声明变量同时赋值
    v_num number(3);
    v_f constant varchar(20) :='我是常量';
begin
   -- v_f := 'aaa'; -- 常量不能够被修改
    v_a := 30;
    v_num := v_a + v_b;
    dbms_output.put_line(v_a||'+'|| v_b ||'='||v_num); -- || 字符串拼接我们通过 || 来实现
end;
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1462/1679296827057/d150d5bf82cb4f4e88723a7c7dcb19f7.png)

### 2.2 into

&emsp;&emsp;into我们在执行SQL操作的时候，需要把查询的字段信息赋值给变量。那么这时我们就可以通过into 关键字来实现。如果有多个字段要赋值。我们只需要在into的左右两侧建立好对应关系即可。

```sql
declare
   v_name varchar2(30);
   v_sex varchar2(3);
   v_dept varchar2(10);
begin
   select name,sex,department into v_name,v_sex,v_dept from student where id = 901;
   dbms_output.put_line(v_name||'-'||v_sex||'-'||v_dept);
end;


-- 定义两个变量 v_a,v_b 计算和是多少
declare
    v_a number(3) :=&请输入a; --- 声明变量
    v_b number(3) :=&请输入b; -- 声明变量同时赋值
    v_num number(3);
begin
    v_num := v_a + v_b;
    dbms_output.put_line(v_a||'+'|| v_b ||'='||v_num); -- || 字符串拼接我们通过 || 来实现
end;
```

### 2.3 属性类型

1. %type:变量和字段类型的绑定
2. %rowtype：表结构中的一条记录的绑定

```sql
-- 变量的类型如果和字段的类型不一致怎么办?
-- 属性类型
declare
   v_name student.name%type;
   v_sex student.sex%type;
   v_dept student.department%type;
begin
   select name,sex,department into v_name,v_sex,v_dept from student where id = 901;
   dbms_output.put_line(v_name||'-'||v_sex||'-'||v_dept);
end;
-- 表结构中有很多个字段。我们对于的就需要声明多少个变量，很繁琐。
declare
   v_row student%rowtype;
begin
   select * into v_row from student where id = 901;
   dbms_output.put_line(v_row.id||'-'||v_row.name||'-'||v_row.sex);
end;
```

## 3.控制语句

### 3.1 分支语句

#### 3.1.1 if语句

&emsp;&emsp;if语句的作用是控制程序的执行顺序。范围控制

```sql
declare
   v_age number(3) := &请输入年龄;
begin 
   dbms_output.put_line('v_age='||v_age);
   if v_age = 18 then
      dbms_output.put_line('成年小伙');
   end if;
   dbms_output.put_line('-------');
   if v_age = 18 then
      dbms_output.put_line('成年小伙');
      else
      dbms_output.put_line('未知...');
   end if;
   dbms_output.put_line('-------');
   if v_age = 18 then
      dbms_output.put_line('成年小伙');
      elsif v_age < 18 then
      dbms_output.put_line('小孩子');
      elsif v_age > 18 then
      dbms_output.put_line('成年人');
      else
      dbms_output.put_line('未知...');
   end if;
end;
```

#### 3.1.2 case语句

&emsp;&emsp;case语句是一个非常强大的关键字。既可以实现类似于Java中的switch语句的作用。也可以像if语句一样来实现范围的处理。

```sql
-- case 语句
declare
   v_age number(3) := &输入年龄;
begin
   case
     when v_age < 18 then
      dbms_output.put_line('小朋友');
      when v_age > 18 then 
      dbms_output.put_line('成年人');
      else
      dbms_output.put_line('刚好成年');
   end case;
end;
-- case 语句可以实现类似于Java中的switch语句。在 case 和when之间声明变量就可以
-- 如果是在when 和 then 之间指定条件那么和if语句是类似的
declare
   v_age number(3) := &输入年龄;
begin
   case v_age
     when 18 then
      dbms_output.put_line('18');
      when 19 then 
      dbms_output.put_line('19');
      else
      dbms_output.put_line('未知');
   end case;
end;
```

### 3.2 循环语句

#### 3.2.1 无限循环

&emsp;&emsp;loop循环可以通过exit来指定条件跳出循环。如果不指定那么就是无限循环

```sql
-- 输出1~10
declare
  v_i number(3) := 1;
begin
  loop
     dbms_output.put_line(v_i);
     exit when v_i >= 10; -- 退出循环
     v_i := v_i + 1;
  end loop;
end;
```

#### 3.2.2 有条件循环

通过while来指定循环的条件

```sql
declare
   v_i number(3) := 1;
begin
   while v_i <= 10 loop
      dbms_output.put_line(v_i);
      -- 修改变量
      v_i := v_i + 1;
   end loop;
end;
```

#### 3.2.3 for循环

```sql
--for循环
begin
   for i in 1..10 loop
    dbms_output.put_line(i);
   end loop;
end;
select * from student;
begin
  for cur_row in (select id,name,sex,department  from student) loop
     dbms_output.put_line(cur_row.id||'-'|| cur_row.name ||'-' || cur_row.sex || '-' || cur_row.department);
  end loop;
end;
```

### 3.2.4 goto

顺序控制用于按顺序执行语句,goto关键字会跳转到我们指定的位置开始自上而下执行。

```sql
-- goto
declare
   v1 number(3) := &请输入v1的值;
begin
   if v1 > 10 then
        goto c1;
   elsif v1 = 10 then 
      goto c2; 
   else
      dbms_output.put_line('其他');
   end if;
        dbms_output.put_line('666');
   <<c1>>
   dbms_output.put_line('大于10');
   <<c2>>
   dbms_output.put_line('等于10');
   
   dbms_output.put_line('----1----');
   dbms_output.put_line('----2----');
end;
```

## 4.动态SQL语句

&emsp;&emsp;动态 SQL 是指在PL/SQL程序执行时生成的SQL 语句。

语法结构为：

```sql
EXECUTE IMMEDIATE dynamic_sql_string
      [INTO  define_variable_list]
      [USING bind_argument_list];
```

案例

```sql
-- 可以根据名字或者性别来查询学生的信息
declare
    v_name student.name%type := '&请输入姓名';
    v_sex student.sex%type :='&请输入性别';
    v_sql varchar2(200);
    v_row student%rowtype;
begin
    v_sql := 'select * from student where 1=1 ';
    if v_name is not null then
        v_sql := v_sql || ' and name like ''%'||v_name||'%''' ;
    end if;
  
    if v_sex is not null then
       v_sql := v_sql || ' and sex = '''|| v_sex||'''' ;
    end if;
    execute immediate v_sql into v_row ;
  
  
    dbms_output.put_line(v_row.name||'---'||v_row.sex||'---'||v_row.department);
end;
```

如果查询的结果不存在或者返回的记录过多那么都会爆出异常信息

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1462/1679296827057/7e13f1cb5e224c62b49407c63ada8b43.png)

## 5.异常语句

在运行程序时出现的错误叫做异常
发生异常后，语句将停止执行，控制权转移到PL/SQL 块的异常处理部分
异常有两种类型

* 预定义异常 - 当 PL/SQL 程序违反 Oracle 规则或超越系统限制时隐式引发
* 用户定义异常 - 用户可以在 PL/SQL 块的声明部分定义异常，自定义的异常通过 RAISE 语句显式引发

处理系统预定义异常：

```sql
-- 异常的应用
-- 系统预定义异常： 
-- too_many_rows 多行数据
-- no_data_found 找不到
-- others 其他异常
declare
   v_name student.name%type;
   
begin
   select name into v_name from student where id = 900 ;
   dbms_output.put_line(v_name);
   
   -- 异常语句块
   exception
       when too_many_rows then
        dbms_output.put_line('返回太多行');
       when no_data_found then
        dbms_output.put_line('找不到数据');
       when others then
        dbms_output.put_line('其他错误');
end;

```

自定义异常：

步骤：

1. 需要显示的声明自定义的异常
2. 在业务逻辑代码中通过raise关键字抛出自定义异常
3. 我们需要在异步部分来声明自定义异常满足条件的处理方案

```sql
-- 自定义异常
declare
   myException exception; -- 声明异常
   v_name varchar2(30) := '张三1';
begin
   if v_name not in ('张三','李四','王五') then
      -- 满足条件就抛出异常
      raise myException;
    else
      dbms_output.put_line('---------------------');
   end if;
    dbms_output.put_line('---------66666------------');
   
   exception
      when myException then
        dbms_output.put_line('---------触发了自定义异常------------');
      when others then
       dbms_output.put_line('---------其他异常------------');
end;
```

# 二、游标

游标的作用：处理多行数据，类似与java中的集合

## 1.隐式游标

&emsp;&emsp;一般是配合显示游标去使用的，不需要显示声明，打开，关闭，系统自定维护,名称为：sql

常用属性：

* sql%found:语句影响了一行或者多行时为true
* %NOTFOUND:语句没有任何影响的时候为true
* %ROWCOUNT:语句影响的行数
* %ISOPEN:游标是否打开，始终为false

案例：

```sql
begin
           update t_student set age=20 ;
  
           if sql%found then
               dbms_output.put_line('修改成功，共修改了   ' ||  sql%rowcount  || '   条记录');
             else
                 dbms_output.put_line('没有这个学生');
             end if;
   
            -- commit ;-- 提交应该要放在隐式游标后面
        end ; 
```

## 2.显示游标

&emsp;&emsp;显式游标在PL/SQL块的声明部分定义查询，该查询可以返回多行,处理多行数据

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1462/1679296827057/6ea8b3d7129c40298ff4afbbe285475a.png)

实现步骤：

1. 声明一个游标
2. 打开游标
3. 循环提取数据
4. 关闭游标

案例：

a) 无参数 ：查询所有学生信息，并显示出学生姓名，性别，年龄

```sql
-- 步骤：1.声明一个游标  2.打开游标  3.循环提取数据 4.关闭游标
-- 查询所有的学生信息。并且显示学生的姓名，年龄和性别
declare
   v_row t_student%rowtype;
   -- 1.游标的声明
   cursor mycursor is select * from t_student ;
begin
    -- 2.打开游标
    open mycursor;
  
    -- 3.循环提取数据
    loop
        fetch mycursor into v_row;
        -- 找到出口
        exit when mycursor%notfound;
        dbms_output.put_line(v_row.name||'-'||v_row.gender||'-'||v_row.age);
    end loop;
    -- 4.关闭游标
    close mycursor;
end;
```

b) 有参数：

```sql
declare
          v_sex varchar2(4) :='&请输入性别' ;
           v_row t_student%rowtype ;
           cursor mycursor(p_sex varchar2) is select * from t_student where sex=p_sex ; -- 注：参数的类型不要指定长度大小
        begin
           open mycursor(v_sex) ;-- 2、打开游标
           loop
               fetch mycursor into v_row;
                  exit when mycursor%notfound;
               dbms_output.put_line(v_row.stuname || ',' || v_row.sex || ',' || v_row.age);
   
           end loop; 
           close mycursor;--  4、 关闭游标
        end ;   
```

c )  循环游标. 简化 游标 for：不需要打开游标 也不需要关闭游标

```sql
declare
          v_sex varchar2(4) :='&请输入性别' ;
           cursor mycursor(p_sex varchar2) is select * from t_student where sex=p_sex ; -- 注：参数的类型不要指定长度大小
        begin
   
          for v_row   in  mycursor(v_sex)    loop
               dbms_output.put_line(v_row.stuname || ',' || v_row.sex || ',' || v_row.age);  
           end loop; 
   
        end ;   
```

d) 使用显式游标更新行：

允许使用游标删除或更新活动集中的行，声明游标时必须使用 select ... for update 语句。

```sql
declare
          v_sex varchar2(4) :='&请输入性别' ;
           v_row t_student%rowtype ;
           cursor mycursor(p_sex varchar2) is select * from t_student where sex=p_sex  for update; -- 注：参数的类型不要指定长度大小
        begin
           open mycursor(v_sex) ;-- 2、打开游标
           loop
               fetch mycursor into v_row;
                  exit when mycursor%notfound;
              -- dbms_output.put_line(v_row.stuname || ',' || v_row.sex || ',' || v_row.age);
   
               update t_student set age = age +10 where current of mycursor;
   
           end loop; 
           --commit ;
           close mycursor;--  4、 关闭游标
        end ;   
  
```

## 3.REF游标

&emsp;&emsp;处理运行时动态执行的 SQL 查询,特点：

优点：

1. 动态SQL语句
2. 在存储过程中可以当参数

缺点：

1. 不能使用循环游标for
2. 不能使用游标更新行

使用步骤：

1. 定义一个ref的类型
2. 声明游标
3. 打开游标
4. 提取数据
5. 关闭游标

案例讲解

```sql
declare
         v_sex varchar2(4) ;
       --type mytype is ref cursor return t_student%rowtype; -- 强类型的 ref 游标类型
         type mytype is ref cursor  ;   --  1）弱类型的 ref 游标类型
         mycursor mytype;   --  2) 声明游标
         v_sql varchar2(100) ;
         v_row t_student%rowtype ;
     begin
         v_sql :=' select * from t_student ' ;
  
        -- open mycursor for select * from t_student; 
         open mycursor for v_sql ;
         loop
             fetch mycursor into v_row ;
             exit when mycursor%notfound ;
             dbms_output.put_line(v_row.stuname || ',' || v_row.sex || ',' || v_row.age);
         end loop;
         close mycursor ;  
   
     end ;
```

可以使用sys_refcursor类型

```sql
declare
         v_stuname t_student.stuname%type :='&请输入名字' ;
           v_sex varchar2(3) :='&请输入性别' ;
  
         mycursor  sys_refcursor ;   --  2) 声明游标
         v_sql varchar2(100) ;
         v_row t_student%rowtype ;
     begin
   
          v_sql :='select * from t_student  where  1=1 ';
   
             if  v_stuname is not null then 
                 v_sql :=v_sql  || '  and stuname like  ''%'  || v_stuname || '%'' ' ;
              end if;  
  
            if  v_sex is not null then
                   v_sql :=v_sql || '  and sex = '''  || v_sex || ''' ' ;
            end if;
  
            dbms_output.put_line('v_sql= ' || v_sql );
   
  
        -- open mycursor for select * from t_student; 
         open mycursor for v_sql ;
         loop
             fetch mycursor into v_row ;
             exit when mycursor%notfound ;
             dbms_output.put_line(v_row.stuname || ',' || v_row.sex || ',' || v_row.age);
         end loop;
         close mycursor ;  
   
     end ;
```

游标的小结：

* 游标用于处理查询结果集中的数据
* 游标类型有：隐式游标、显式游标和 REF游标
* 隐式游标由 PL/SQL 自动定义、打开和关闭
* 显式游标用于处理返回多行的查询
* 显式游标可以删除和更新活动集中的行
* 要处理结果集中所有记录时，可使用循环游标

# 三、子程序

&emsp;&emsp;什么是子程序：命名的 PL/SQL 块，编译并存储在数据库中

## 1.存储过程

### 1.1 语法结构

```sql
CREATE [OR REPLACE] PROCEDURE 
   <procedure name> [(<parameter list>)]
IS|AS 
   <local variable declaration>
BEGIN
   <executable statements>
[EXCEPTION
   <exception handlers>]
END;
```

### 1.2 案例讲解

无参数案例：写一个存储过程 ，往学生表中模拟 10 00 条数据（插入1000  条数据 ）

```sql
create or replace procedure protest01
     is
        -- 声明变量  
     begin
         for  i  in 1..100 loop
             insert into t_student(id,stuname,sex,age) values(seq_t_student.nextval  , '小李' || i  , '男' , i );
         end loop;
         commit ;
     end ;
```

调用存储过程：

```sql
declare
        begin
           -- protest01();
            protest01;   --  当没有参数里，括号可省略不写
        end;
```

有参数的案例：

```sql
create or replace procedure protest02(
	p_name varchar2,
	p_sex  varchar2, 
	p_age number
	)
     is
        -- 声明变量  
     begin
   
         dbms_output.put_line(p_name || ',' || p_sex || ',' || p_age );
   
     end ;
```

调用处理

```sql
declare
           v_name varchar2(10) :='&请输入名字' ;
           v_sex varchar2(4) :='&请输入性别';
           v_age number(3) :='&请输入年龄'; 
        begin
            protest02(v_name,  v_sex , v_age);   --  当没有参数里，括号可省略不写
        end;
```

参数的三种类型

* in 输入
* out 输出
* in out 输入输出

案例讲解

```create or replace procedure protest03(
	p_name in varchar2, 
	p_sex in out  varchar2, 
	p_age in out number)
     is
        -- 声明变量  
     begin
   
         dbms_output.put_line(p_name || ',' || p_sex || ',' || p_age );
         p_sex :='我是P_sex';  
     end ;

调用过程

​```sql
     declare
           v_name varchar2(10) :='&请输入名字' ;
           v_sex varchar2(50) :='&请输入性别';
           v_age number(3) :='&请输入年龄'; 
        begin
            protest03(v_name,  v_sex , v_age);   --  当没有参数里，括号可省略不写
             dbms_output.put_line(v_sex);
        end;
  
```

案例： 请根据性别或名字查询相关记录，并把结果 返回来 打印了出来  提示用 sys_refcursor

```sql
create or replace procedure protest04(
		p_name varchar2,
		p_sex varchar2, 
		myresult out sys_refcursor)
          is
             v_sql  varchar2(100) ;
          begin
             v_sql :='select * from t_student where 1=1 ';
             if p_name is not null then
                v_sql :=v_sql  ||  '  and  stuname like  ''%'  || p_name  ||   '%'' ' ;
             end if;
             if p_sex is not null then
                v_sql := v_sql || '  and  sex = ''' || p_sex || '''  ';
             end if;
             dbms_output.put_line('v_sql='  || v_sql);
   
             open myresult for v_sql ;
  
          end;
```

调用

```sql
-- 执行测试
          declare
             v_name varchar2(20) :='&请输入名字';
             v_sex varchar2(4) :='&请输入性别' ;
             mycursor sys_refcursor ;
             v_row t_student%rowtype;
          begin
             --  protest04(v_name, v_sex , mycursor);  
	-- 1) 位置传递
                --2 ）名称传递
                 --protest04( p_sex =>v_sex , p_name => v_name , myresult => mycursor);
                 --3) 混合使用  : 先用位置传递，如果后面有用了名称传递，后面就不能用位置传递
                     protest04(v_name , myresult=>mycursor ，p_sex => v_sex,);
               loop
                   fetch mycursor into  v_row ;
                    exit when mycursor%notfound ;
                    dbms_output.put_line(v_row.stuname || ',' || v_row.sex || ',' || v_row.age );
               end loop;
   
               close mycursor ;   
          end;
```

## 2.存储函数

&emsp;&emsp;类似于java中方法，有返回值可以返回值的命名的 PL/SQL 子程序。

### 2.1 语法结构

```sql
CREATE [OR REPLACE] FUNCTION 
  <function name> [(param1,param2)]
RETURN <datatype>  IS|AS 
  [local declarations]
BEGIN
  Executable Statements;
  RETURN result;
EXCEPTION
  Exception handlers;
END;
```

### 2.2 案例讲解

无参案例：写一个函数 ，获取学生名称

```sql
create or replace function funtest02
   return varchar2
   is
      v_name varchar2(20) ;
   begin
         select stuname into v_name from t_student where id=201;
         return v_name;
   end ;
```

如何调用呢

* a)用PL/SQL块调用函数
* b)用select语句调用: 在函数中不能有增加删除修改的语句，只能是查询的语句

有参案例：

```sql
create or replace function  funtest03(p_name in varchar2, p_sex out varchar2, p_age in out number)
     return varchar2
     is
        -- 声明变量  
     begin
   
         dbms_output.put_line(p_name || ',' || p_sex || ',' || p_age );
         p_sex :='我是函数的p_sex' ;
            return '成功';
     end ; 
```

调用

```sql
   -- 调用 
     declare
           v_name varchar2(10) :='&请输入名字' ;
           v_sex varchar2(20) :='&请输入性别';
           v_age number(3) :='&请输入年龄'; 
           v_result varchar2(30) ;
        begin
            v_result := funtest03(v_name, v_sex , p_age=>v_age);   
              dbms_output.put_line('v_result=' || v_result || ', v_sex=' || v_sex );
        end;
```

## 3.程序包

程序包：作用就是管理我们的存储过程和方法

```sql
-- 程序包：作用就是管理我们的存储过程和方法
-- 规范和主体两部分组成
-- 创建一个规范
create or replace package pak01
is
   procedure myprocdure01(p_name varchar2);
   function myfun01 return number;
end pak01;


-- 创建规范对应的主体：主体中的方法如果在规范中声明了。那么外包可以访问。如果没有那么就只能被内部调用
create or replace package body pak01
is

    -- myprodure01 存储过程
    procedure myprocdure01(p_name varchar2)
    as
    begin
        dbms_output.put_line('p_name = '||p_name);
    end;
    -- myfun01 方法
    function myfun01 return number
    is
    begin
         dbms_output.put_line('方法执行了.... ');
         return 666;
    end;
end pak01;


create or replace package body pak01
is

    -- myprodure01 存储过程
    procedure myprocdure01(p_name varchar2)
    as
    begin
        dbms_output.put_line('p_name = '||p_name);
    end;
    -- myfun01 方法
    function myfun01 return number
    is
    begin
         dbms_output.put_line('方法执行了.... ');
         return 666;
    end;
  
    function myfun02 return number
    is
  
    begin
       dbms_output.put_line('方法执行了..222.. ');
       return 999;
    end;
end pak01;

-- 调用package中的过程和方法 package.
begin
   pak01.myprocdure01('李四');
end;

select pak01.myfun02 from dual;
```

# 四、触发器

## 1.触发器的基本讲解

&emsp;&emsp;当特定事件出现时自动执行的存储过程

语法结构

```sql
CREATE [OR REPLACE] TRIGGER trigger_name
AFTER | BEFORE | INSTEAD OF
[INSERT] [[OR] UPDATE [OF column_list]] 
[[OR] DELETE]
ON table_or_view_name
[REFERENCING {OLD [AS] old / NEW [AS] new}]
[FOR EACH ROW]
[WHEN (condition)]
declare
begin
end;
```

案例：对学生表进行增加删除修改后打印一句  操作成功

```sql
create or replace trigger trigger01
after insert or update or delete on t_student
declare
   
begin
   dbms_output.put_line('操作成功');
end ;
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1462/1679296827057/5c4f229453584703904b28b59256fce0.png)

## 2.触发器的类型

### 2.1 语句级触发器

&emsp;关注的是执行了这条语句

案例：创建一个对学生表的增删改的审计触发器

准备表

```sql
CREATE TABLE t_audit_table
(
  stablename varchar2(30),
  nins number,--记录添加次数
  nupd number,--记录修改次数
  ndel number,--记录删除次数
  startdate date,
  enddate date
)
```

实现：

```sql
create or replace trigger trigger02
    after insert or delete or update on t_student
    declare
       v_count number(3);
    begin
        -- 先判断t_student在这个日志表中是否有这条记录，如果没有，要先插入数据
        select count(*) into v_count from t_audit_table where stablename='t_student';
        if v_count<=0 then
             insert into t_audit_table(stablename,nins,nupd,ndel) values('t_student', 0,0 ,0);
        end if;
  
        if inserting then
            update t_audit_table set nins=nins+1 where stablename='t_student';
        end if;
        if updating then
             update t_audit_table set nupd=nupd+1 where stablename='t_student';
        end if;
        if deleting then
            update t_audit_table set ndel=ndel+1 where stablename='t_student';
        end if;
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1462/1679296827057/5ca7d187910441b4a2a2bacb16f50b7f.png)

### 2.2 行级触发器

&emsp;&emsp;和影响的行数：影响了多少行数据。那么这个触发器就会触发多少次

```sql
create or replace trigger trigger02
    after insert or delete or update on t_student
    FOR EACH ROW
    declare
       v_count number(3);
    begin
        -- 先判断t_student在这个日志表中是否有这条记录，如果没有，要先插入数据
        select count(*) into v_count from t_audit_table where stablename='t_student';
        if v_count<=0 then
             insert into t_audit_table(stablename,nins,nupd,ndel) values('t_student', 0,0 ,0);
        end if;
  
        if inserting then
            update t_audit_table set nins=nins+1 where stablename='t_student';
        end if;
        if updating then
             update t_audit_table set nupd=nupd+1 where stablename='t_student';
        end if;
        if deleting then
            update t_audit_table set ndel=ndel+1 where stablename='t_student';
        end if;
  
```

### 2.3 限制行级触发器

&emsp;&emsp;对部分数据做特定的处理，比如：不能删除管理员

```sql
create or replace trigger trigger03
   before  delete on t_student
    for each row
    when(old.stuname='小李6')  
  declare
  begin
         dbms_output.put_line('班长不能被删除');
   
        RAISE_APPLICATION_ERROR(-20001, '班长不能被删除');
  end;
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1462/1679296827057/8bbcee36b1fb48b48b6cb7b7961c9e25.png)

# 五、视图和索引

# 六、数组

IS TABLE OF ：指定是一个集合的表的数组类型,简单的来说就是一个可以存储一列多行的数据类型。

INDEX BY BINARY_INTEGER：指索引组织类型

RECORD 类型来指定元素类型

```sql
DECLARE
   TYPE TimeRecTyp IS RECORD (
      hour   SMALLINT := 0, --赋默认值0
      minute SMALLINT := 0,
      second SMALLINT := 0);
   TYPE TimeTabTyp IS TABLE OF TimeRecTyp
      INDEX BY BINARY_INTEGER; -- BY后面为索引类型，可以是char、number、varchar2
```

# 七、常见内置函数

## 1.字符函数

| 函数                     | 说明                                                         |
| :----------------------- | ------------------------------------------------------------ |
| ASCII(X)                 | 返回字符X的ASCII码                                           |
| CONCAT(X,Y)              | 连接字符串X和Y                                               |
| INSTR(X,STR[,START][,N)  | 从X中查找str，可以指定从start开始，也可以指定从n开始         |
| LENGTH(X)                | 返回X的长度                                                  |
| LOWER(X),UPPER(X)        | X转换成小/大写                                               |
| LTRIM(X[,TRIM_STR])      | 把X的左边截去trim_str字符串，缺省截去空格                    |
| RTRIM(X[,TRIM_STR])      | 把X的右边截去trim_str字符串，缺省截去空格                    |
| TRIM([TRIM_STR FROM]X)   | 把X的两边截去trim_str字符串，缺省截去空格                    |
| REPLACE(X,old,new)       | 在X中查找old，并替换成new                                    |
| SUBSTR(X,start[,length]) | 返回X的字串，从start处开始，截取length个字符，缺省length，默认到结尾 |

## 2.日期函数

| 函数                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| ADD_MONTHS(d,n)     | 在某一个日期 d 上，加上指定的月数 n，返回计算后的新日期。（d 表示日期，n 表示要加的月数） |
| LAST_DAY(d)         | 返回指定日期当月的最后一天                                   |
| ROUND(d[,fmt])      | 返回一个以 fmt 为格式的四舍五入日期值                        |
| EXTRACT(fmt FROM d) | 提取日期中的特定部分，fmt 为：YEAR、MONTH、DAY、HOUR、MINUTE、SECOND，**注意时区** |
|                     |                                                              |
|                     |                                                              |
|                     |                                                              |
|                     |                                                              |
|                     |                                                              |
|                     |                                                              |
|                     |                                                              |

补充：

**ROUND(d[,fmt])**：

1.  如果 fmt 为“YEAR”则舍入到某年的 1 月 1 日，即前半年舍去，后半年作为下一年。
2.  如果 fmt 为“MONTH”则舍入到某月的 1 日，即前月舍去，后半月作为下一月。

3.  默认为“DDD”，即月中的某一天，最靠近的天，前半天舍去，后半天作为第二天。

4.  如果 fmt 为“DAY”则舍入到最近的周的周日，即上半周舍去，下半周作为下一周周日。

![image-20240228110114803](C:\Users\codef\AppData\Roaming\Typora\typora-user-images\image-20240228110114803.png)

![image-20240228110126263](C:\Users\codef\AppData\Roaming\Typora\typora-user-images\image-20240228110126263.png)

## 3.转换函数

| 函数                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| TO_CHAR(d\|n[,fmt]) | 把日期和数字转换为制定格式的字符串。Fmt是格式化字符串,例如：SELECT TO_CHAR(SYSDATE,'YYYY"年"MM"月"DD"日" HH24:MI:SS')"date" FROM dual; |
| TO_DATE(X,[,fmt])   | 把一个字符串以fmt格式转换成一个日期类型                      |
| TO_NUMBER(X,[,fmt]) | 把一个字符串以fmt格式转换为一个数字                          |
|                     |                                                              |
|                     |                                                              |

## 4.其他函数

```sql
NVL(X,VALUE) --如果X为空，返回value，否则返回X
NVL(x,value1,value2) --如果x非空，返回value1，否则返回value2
decode(expression,value,result1,result2) --如果expression=value，则输出result1，否则输出result2
decode(expression,value1,result1,value2,result2,value3,result3......,default) --如果expression=value1，则输出result1，expression=value2，输出reslut2，expression=value3，输出result3，若expression不等于所列出的所有value，则输出为default
```

