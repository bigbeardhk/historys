# Stream集合处理流


### 分组(Collectors.groupingBy())

> **概述**:</p>
> 1.默认groupingBy代码里会生成一个HashMap(hashMap是无序的,put的顺序与get的顺序不一致)</p>
> 2.groupingBy()的三个参数:</p>
> 第一个参数：分组按照什么分类 </p>
> 第二个参数：分组最后用什么容器保存返回 </p>
> 第三个参数：按照第一个参数分类后，对应的分类的结果如何收集</p>
> 后面默认的二个参数为: 第二个参数默认是HashMap::new， 第三个参数收集器其实默认是Collectors.toList</p>
> 3.需要记录插入顺序时,第二个参数可以使用LinkedHashMap::new</p>

<details>
<summary>代码演示:</summary>

```Java
    //按id分组,分组后的List集合也是按时间排序过的
    Map<Long, List<Student>> collect = students.stream()
        .filter(Objects::nonNull) //筛选出不为null的对象
        .filter(v -> v.getId() != null) //筛选出其id值不为null
        .sorted(Comparator.comparing(Student::getAge).thenComparing(Student::getGrade).reversed()) //进行数据排序(倒序)
        .collect(Collectors.groupingBy(Student::getId)); //按id进行分组
    //分组计数
    Map<Long, Long> map = students.stream()
        .collect(Collectors.groupingBy((Student::getId), Collectors.counting()));
```

</details>
    
### 排序
> // 取出最近的一条做SLA规则 </p>
> slaResume = matchOldResumes.stream().max(Comparator.comparingLong(ResumeSlaStatus ::getResumeId)).get();
    
