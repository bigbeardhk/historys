## DDL
> 资料:</p>
> ~[MySQL常用关键字与函数](https://blog.csdn.net/weixin_44478196/article/details/106750347?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-0.control&spm=1001.2101.3001.4242)</p>
> ~[MySQL数据库中插入数据的几种方式](https://blog.csdn.net/weixin_44478196/article/details/115679245)</p>
> 数据定义语句

### 修改字段

- 修改字段类型

alter table 表名 modify column 字段名 [修改后的字段类型];</p>
ALTER TABLE odrm_process_instance_t MODIFY COLUMN cc_account VARCHAR ( 500 ) DEFAULT NULL COMMENT '抄送人账号';

- 修改字段(包括字段名,字段类型,字段备注等)

ALTER  TABLE table_name CHANGE COLUMN  column_name_old   column_name_new VARCHAR(30) DEFAULT NULL  COMMENT  '流程节点顺序(从1开始)';
> 注意 : change/first|after 字段名 这些关键字都是属于MySQL在标准SQL上的扩展，在其他的数据库上不一定适用

### 删除字段

ALTER  TABLE  table_name   DROP COLUMN;

### 增加字段

ALTER TABLE  table_name   ADD COLUMN  show_flag  tinyint(4)  DEFAULT NULL COMMENT '是否应该出现在offer沟通栏(0 出现 1 不出现)' AFTER delete_flag;


```sql

drop table if exists odrm_resume_exceptions_t;
CREATE TABLE `odrm_resume_exceptions_t` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `resume_id` bigint(20) NOT NULL COMMENT '简历主键ID',
  `exceptions_type` tinyint(4) DEFAULT NULL COMMENT '例外事项类型(0 一般 1 严重)',
  `exceptions_description` varchar(2000) DEFAULT NULL COMMENT '加入例外原因',
  `exceptions_removal_reason` varchar(2000) DEFAULT NULL COMMENT '解除例外原因',
  `exceptions_status` tinyint(4) DEFAULT NULL COMMENT '例外事项状态(0 存在例外事项 1 例外事项解除)',
  `exceptions_time` datetime DEFAULT NULL COMMENT '加入例外时间',
  `removal_time` datetime DEFAULT NULL COMMENT '解除例外时间',
  `status` tinyint(4) DEFAULT NULL COMMENT '数据表状态 (0 无效 1 有效)',
  `created_time` datetime DEFAULT NULL COMMENT '创建时间',
  `created_by` varchar(50) DEFAULT NULL COMMENT '创建人账号',
  `created_id` bigint(20) DEFAULT NULL COMMENT '创建人ID',
  `modified_time` datetime DEFAULT NULL COMMENT '修改时间',
  `modified_by` varchar(50) DEFAULT NULL COMMENT '修改人账号',
  `modified_id` bigint(20) DEFAULT NULL COMMENT '修改人ID',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='简历例外事项表';

update odrm_interviewer_information_t 
set interview_language = CONCAT(',',interview_language,',')
where 
(interview_language != null or  interview_language <> '')

-- ==================刷新排班表时间窗字段===================
-- 第一步执行
update odrm_interviewer_scheduling_t
set time_window = 2
where
isnull(time_window)
and 
DATE_FORMAT(interview_start_time,'%H:%i:%s') between '18:00:00' and '23:59:59'
;

-- 第二步执行
update odrm_interviewer_scheduling_t
set time_window = 1
where
isnull(time_window)
and 
DATE_FORMAT(interview_start_time,'%H:%i:%s') between '12:00:00' and '17:59:59'
;

-- 第三步执行
update odrm_interviewer_scheduling_t
set time_window = 0
where
isnull(time_window)
and not isnull(interview_start_time)
;



```


- https://www.cnblogs.com/klvchen/p/10137117.html
- https://www.cnblogs.com/ggjucheng/archive/2012/11/11/2765237.html
- https://www.cnblogs.com/ggjucheng/archive/2012/11/07/2758058.html

