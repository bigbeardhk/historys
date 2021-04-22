```java 
package com.hk.java_study.java1_8;


import javafx.scene.input.DataFormat;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.time.format.DateTimeFormatter;
import java.util.*;
import java.util.concurrent.CopyOnWriteArraySet;
import java.util.stream.Collectors;
import java.util.stream.LongStream;

/**
 * @Author bigbeardhk
 * @Date 2021/3/22 22:00
 * @Email bigbeardhk@163.com
 */
public class SteamStudy {

    public static void main(String[] args) throws ParseException {

        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
// 初始化
        List<Student> students = new ArrayList<Student>() {
            {
                add(new Student(20160001L, "孔明", 20, 1, "土木工程", "武汉大学",format.parse("2021-05-12 12:02:01")));
                add(new Student(20160002L, "伯约", 21, 2, "信息安全", "武汉大学",format.parse("2020-05-10 12:02:01")));
                add(new Student(20160001L, "玄德", 22, 3, "经济管理", "武汉大学",format.parse("2021-04-02 12:02:01")));
                add(new Student(20160004L, "云长", 21, 2, "信息安全", "武汉大学",format.parse("2022-03-25 10:02:01")));
                add(new Student(20161001L, "翼德", 21, 2, "机械与自动化", "华中科技大学",format.parse("2022-03-26 12:02:01")));
                add(new Student(20161002L, "元直", 23, 4, "土木工程", "华中科技大学",format.parse("2021-05-12 12:00:01")));
                add(new Student(20161003L, "奉孝", 23, 4, "计算机科学", "华中科技大学",format.parse("2023-05-12 13:02:01")));
                add(new Student(20162001L, "仲谋", 22, 3, "土木工程", "浙江大学",format.parse("1995-05-12 12:02:01")));
                add(new Student(20162002L, "鲁肃", 23, 4, "计算机科学", "浙江大学",format.parse("2021-05-12 13:02:01")));
                add(new Student(20163001L, "丁奉", 24, 5, "土木工程", "南京大学",format.parse("2021-05-12 13:03:01")));
            }
        };

//        Map<Long, List<Student>> collect = students.stream()
//                .filter(Objects::nonNull)//筛选出不为null的对象
//                .filter(o -> o.getId() != null)//筛选出其id值不为null
//                .collect(Collectors.groupingBy(Student::getId));//按id进行分组
//        collect.forEach((k,v) ->v.forEach(System.out::println));


        //按Study类的studyDate时间属性由最大时间起始排序，第一条值为最大时间
        List<Student> collect1 = students.stream().sorted(Comparator.comparing(Student::getStudyDate).reversed()).collect(Collectors.toList());
        Student student = collect1.get(0);
        System.out.println(student);
        collect1.forEach(v -> System.out.println(v.toString()));


//        .filter(o ->)
//                .mapToLong(Student::getId)

//                .distinct()
//                .toArray();
//        for (int i = 0; i < longs.length; i++) {
//            System.out.println(longs[i]);
//        }


//        String date=null;
//        Date parse = new SimpleDateFormat("yyyy/MM/dd").parse(date);
//        System.out.println(parse);
//
//        Student student =null;
//        System.out.println(Optional.ofNullable(student).map(Student::getId).orElse(null));
//
//        Student student1 =new Student( 2L);
//        Long id = Optional.ofNullable(student1).map(Student::getId).orElse(null);
//
//
//        String string = null;
//        System.out.println(Optional.ofNullable(string).orElse("hh"));
//
//        String string1 = "不错";
//        System.out.println(Optional.ofNullable(string1).orElse("hh"));


    }

}

class Student {

    /** 学号 */
    private Long id;

    private String name;

    private Integer age;

    /** 年级 */
    private Integer grade;

    /** 专业 */
    private String major;

    /** 学校 */
    private String school;

    /** 入学日期 */
    private Date studyDate;
    // 省略getter和setter


    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Integer getGrade() {
        return grade;
    }

    public void setGrade(Integer grade) {
        this.grade = grade;
    }

    public String getMajor() {
        return major;
    }

    public void setMajor(String major) {
        this.major = major;
    }

    public String getSchool() {
        return school;
    }

    public void setSchool(String school) {
        this.school = school;
    }

    public Date getStudyDate() {
        return studyDate;
    }

    public void setStudyDate(Date studyDate) {
        this.studyDate = studyDate;
    }

    public Student() {
    }

    public Student(Long id) {
        this.id = id;
    }

    public Student(Long id, String name, Integer age, Integer grade, String major, String school,Date studyDate) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.grade = grade;
        this.major = major;
        this.school = school;
        this.studyDate = studyDate;
    }


    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", grade=" + grade +
                ", major='" + major + '\'' +
                ", school='" + school + '\'' +
                ", studyDate=" + studyDate +
                '}';
    }
}
```
