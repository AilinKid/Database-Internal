# PostgreSQL Notes
#### cast：
    类型转换的语法，语法比如：cast(1 as TEXT)，将数字1转化成文本类型。
    举个栗子：select round(1/4,4); 除法取小数点后4位，结果为0.0000，因为是整数除法，直接是在0上补后4位。
    可以采用以下的方法，先将整数转化为实数类型：
    select round(cast(1 as numeric)/cast(4 as numeric),4); 这时结果就是0.2500。
    补充一下：select round(1::numeric/4::numeric,4); 可以非函数式地将数字转换为实数。
     
##### 表达式索引： 索引并非一定要建立在一个表的属性上，还可以建立在一个函数或者从表中一个或多个属性计算出来的标量表达式上。
    创 建：CREATE  INDEX  st_lower_idx  ON  student (lower(en_name));
    查 询：SELECT  *  FROM  student  WHERE  lower(en_name) = ‘jack’;
