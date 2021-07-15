## DDL
> 数据定义语句

### 修改字段

- 修改字段类型

alter table 表名 modify column 字段名 [修改后的字段类型];

- 修改字段(包括字段名,字段类型,字段备注等)

ALTER  TABLE table_name   CHANG COLUMN  column_name_old   column_name_new VACHAR(30) DEFAULT NULL  COMMENT  '流程节点顺序(从1开始)';
> 注意 : change/first|after 字段名 这些关键字都是属于MySQL在标准SQL上的扩展，在其他的数据库上不一定适用

### 删除字段

ALTER  TABLE  table_name   DROP COLUMN;

### 增加字段

ALTER TABLE  table_name   ADD COLUMN  show_flag  tinyint(4)  DEFAULT NULL COMMENT '是否应该出现在offer沟通栏(0 出现 1 不出现)' AFTER delete_flag;
