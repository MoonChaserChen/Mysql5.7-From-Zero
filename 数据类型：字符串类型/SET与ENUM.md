## 集合类型与枚举类型
> 列值只能从固定集合中选取的时候，可以使用SET与ENUM数据类型。若列值只能选一个，则使用ENUM，若列值可选零个或多个不重复的值，则使用SET。

### SET

1. SET最多可含有64个集合成员。
    ```
    mysql> CREATE TABLE myset (col SET('1','2','3','4','5','6','7','8','9','10','11','12','13','14','15','16','17','18','19','20','21','22','23','24','25','26','27','28','29','30','31','32','33','34','35','36','37','38','39','40','41','42','43','44','45','46','47','48','49','50','51','52','53','54','55','56','57','58','59','60','61','62','63','64', '65'));
    ERROR 1097 (HY000): Too many strings for column col and SET
    ```
2. SET每个元素用逗号分隔，因此元素不能包含逗号
3. 创建表时会自动去除SET中元素尾部空格
4. 数值类型为SET的列，实际保存值时会用数值保存，因此在存储时会更加节省空间
    ```
    mysql> create table myset (col set('one', 'two', 'three', 'four'));
    mysql> insert into myset values('one'), ('two'), ('three'), ('one,two,three'), ('four');
    mysql> select col, col+0 from myset;
    +---------------+-------+
    | col           | col+0 |
    +---------------+-------+
    | one           |     1 |
    | two           |     2 |
    | three         |     4 |
    | one,two,three |     7 |
    | four          |     8 |
    +---------------+-------+
    ```
    
    |  列值  | 数值 | 二进制 |
    | ---- | :----: | :----: |
    | one   | 1  | 0001 |
    | two   | 2  | 0010 |
    | three   | 4  | 0100 |
    | one,two,three   | 7  | 0111 |
    | four   | 8  | 1000 |
    
    上面二进制的表示方式也说明了为什么SET最多可含有64个集合成员
5. 同时排序的时候会根据其数值大小排序
    ```
    mysql> select col from myset order by col desc;
    +---------------+
    | col           |
    +---------------+
    | four          |
    | one,two,three |
    | three         |
    | two           |
    | one           |
    +---------------+
    ```
6. 在查询及插入时也可以直接使用其对应的数值。
    1. 等于查询：
        ```
        mysql> insert into myset value(5);
        mysql> select * from myset where col = 5;
        +-----------+
        | col       |
        +-----------+
        | one,three |
        +-----------+
        mysql> select * from myset where col = 'one,three';
        +-----------+
        | col       |
        +-----------+
        | one,three |
        +-----------+
        ```
    2. 查询包含元素'one'的：
        ```
        mysql> select * from myset;
        +---------------+
        | col           |
        +---------------+
        | one           |
        | two           |
        | three         |
        | one,two,three |
        | four          |
        | one,three     |
        +---------------+
        
        mysql> select * from myset where col like '%one%';
        mysql> select * from myset where find_in_set('one', col);
        mysql> select * from myset where col & 1;
        +---------------+
        | col           |
        +---------------+
        | one           |
        | one,two,three |
        | one,three     |
        +---------------+
        ```
    3. 同理，查询包含元素'one' 和 元素 'three'的：
        ```
        mysql> select * from myset where col & 1 and col & 4;
        +---------------+
        | col           |
        +---------------+
        | one,two,three |
        | one,three     |
        +---------------+
        ```
### ENUM
1. ENUM最多65535个元素
2. 如果ENUM定义为允许NULL，则默认值为NULL，若定义为不允许NULL，则默认值为第一个元素
3. ENUM与SET一样，内部存储也是采用数值来存储，因此要注意ENUM元素若本身也是数值则容易产生混淆
    ```
    mysql> create table t1(c1 enum('5', '4', '3', '2', '1'));
    mysql> insert into t1 values('5'), ('4'), ('3'), ('2'), ('1');
    mysql> select c1, c1+0 from t1;
    +------+------+
    | c1   | c1+0 |
    +------+------+
    | 5    |    1 |
    | 4    |    2 |
    | 3    |    3 |
    | 2    |    4 |
    | 1    |    5 |
    +------+------+
    mysql> select * from t1 where c1 = 1;
    +------+
    | c1   |
    +------+
    | 5    |
    +------+
    mysql> select * from t1 where c1 = '1';
    +------+
    | c1   |
    +------+
    | 1    |
    +------+
    ```
    对于排序来说也是同样：
    ```
    mysql> select * from t1 order by c1 asc;
    +------+
    | c1   |
    +------+
    | 5    |
    | 4    |
    | 3    |
    | 2    |
    | 1    |
    +------+
    mysql> select * from t1 order by cast(c1 as char);
    +------+
    | c1   |
    +------+
    | 1    |
    | 2    |
    | 3    |
    | 4    |
    | 5    |
    +------+
    mysql> select * from t1 order by concat(c1);
    +------+
    | c1   |
    +------+
    | 1    |
    | 2    |
    | 3    |
    | 4    |
    | 5    |
    +------+
    ```
    这里排序的时候是按ENUM的数值来进行排序的，也就是ENUM中元素的声明顺序，因此声明ENUM顺序的时候最好按alphabetic顺序

### SET & ENUM
| 类型 | 占用空间 |
| ---- | ---- |
| ENUM | 1 or 2bytes，取决于元素个数，最多65535 |
| SET  | 1, 2, 3, 4, or 8 bytes，取决于成员个数，最多64 |