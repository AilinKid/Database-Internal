# PostgreSQL Notes
#### psql进入pg的一个门户工具
    进入数据库：
    1: ./psql -h 127.0.0.1 -p 5432 student(dbname) postgres(user)
    2: ./psql -d student(dbname)
    两种直接利用psql工具进入数据库的方式，或者直接进入psql，在工具内部使用简化命令\c dbname
    1: ./psql dbname 没有名字默认进入与当前用户同名的postgres，否则报错
    2: \c dbname
    psql的常用命令
    \d 显示数据库中所有表
    \d tablename 显示该表的结构定义
    \d indexname 显示该索引的信息
    \d x？ （？或者*）会显示有关x开头的表，索引，序列等等覆盖该通配的信息
    \d+ 显示比\d 更加详细的信息
    如果只想对特定的对象显示
    可以在\d后面加上相关的名词开头，如
    \dt 只看表
    \di 只看索引
    \ds 只看序列
    \dv 只看视图
    \df 只看函数
    显示执行时间
    \timing on/off
    \dn 显示所有的schema
    \db 显示所有的表空间
    \du 列出用户
    \dg 列出组 （这同上一样，都是列出role）
    \dp tablename 显示出表的权限分配情况
    \encoding 查看客户端编码情况
    \encoding utf8 指定编码集
    \pset 设置输出的显示效果
    border只是边界的的效果 0无边框，1内边框，2全边框
    \x 扩展显示，将每一行的每一列数据都拆分成单行展示
    \i filename（当然可以是相对，绝对路径）在psql工具中，使用外部文件中的sql命令
    ./psql -f filename(当然可以是相对，绝对路径)，非psql中的启动，执行完了之后还是在外部
    \? 查看多有的简化命令
    postgresql避免自动提交
    1：使用begin开头，最后执行commit或者rollback语句。
    2：\set AUTOCOMMIT off 注意这边必须是大写的AUTOCOMMIT，然后手动commit或者rollback。
    运行时显示psql简化命令的完整sql，进入psql时候，使用如下
    ./psql -E dbname 没有名字默认进入与当前用户同名的postgres，否则报错
    如果只是想在psql中有需要的显示简化命令的完整sql
    \set ECHO_HIDDEN on/off
    
    continued in 2017-11-14
    显示当前的search_path
    show search_path
    设置当前的模式搜索路径
    SET search_path TO myschema [,public...];
    以查询子句来建表，不会把约束搞进来
    create table tablename as select...
    以表模板来建表，including all会把所有的约束，索引，存储都弄过来
    create table tablename （like template including XXX）
    临时表 可用TEMPOARY 或者 temp 关键字，该表存在于该会话层面，所属模式为pg_temp_x
    create temp table tablename(...);
    将该表改成事务层面（自己保证表的创建，和数据插入在同一个事务中）
    create temp table tablename（...）on commit delete rows;
    global和local关键字可以用来兼容，实质上没有什么用
    默认值得使用
    字段后面加上 default xxx
    表的修改：
    alter table tablename add column columnname type;
    alter table tablename drop column columnname [cascade];
    alter table tablename add check(...)
    alter table tablename add constraint name check()/unique()/primary key()
    alter table tablename alter column id set not null
    alter table tablename alter column id drop not null
    alter table tablename drop constraint name
    alter table tablename alter column columnname type newtype（需要隐式支持）
    alter table tablename rename cloumn columnname to newcolumnname
    表的继承 父表可以看见子表的更新的数据（除了新的属性）
    create table tablename（新增的属性）inherits（父表名）
    仅看父表自己的数据 加上only
    select * from only father
    
    表分区步骤
    1创建父表，有字段结构，所有的分区表都从他继承，父表一般没有数据
    2子表不添加任何字段，子表为分区，就是普通的heap表
    3给分区增加约束，定义每个分区的键值
    4对于每个分区，在关键字上建立一个索引
    5定义一个规则或者触发器，把主表的数据重定向到合适的分区表
    6确保constraint_exclusion里配置参数在postgresql中是打开的，打开后查询中where字句的过滤条件将与分区的约束条件智能匹配，查询优化
    
    触发器注意：
    对于语句statement的触发器，函数的中必须显式的返回null
    对于before和instead of类型的行级触发器，函数中返回更新后（在new上更新）new行
    对于after类型的行级触发器，函数中返回值被忽略
#### cast：
    类型转换的语法，语法比如：cast(1 as TEXT)，将数字1转化成文本类型。
    举个栗子：select round(1/4,4); 除法取小数点后4位，结果为0.0000，因为是整数除法，直接是在0上补后4位。
    可以采用以下的方法，先将整数转化为实数类型：
    select round(cast(1 as numeric)/cast(4 as numeric),4); 这时结果就是0.2500。
    补充一下：select round(1::numeric/4::numeric,4); 可以非函数式地将数字转换为实数。
     
##### 表达式索引： 索引并非一定要建立在一个表的属性上，还可以建立在一个函数或者从表中一个或多个属性计算出来的标量表达式上。
    创 建：CREATE  INDEX  st_lower_idx  ON  student (lower(en_name));
    查 询：SELECT  *  FROM  student  WHERE  lower(en_name) = ‘jack’;   
##### 部分索引：该索引只包含满足这个谓词的行，不一定非要建立在全表之上
    CREATE INDEX stu_name_idx ON student(name)  where(id>1 AND id<255);

