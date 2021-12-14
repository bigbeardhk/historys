### Optional常用用法

```Java
// 判空赋默认值,Optional类比较简便
  String fullName = matchResume.getFullName() == null ? "" : matchResume.getFullName();
  String fullName = Optional.ofNullable(matchResume.getFullName()).orElse(""); // matchResume对象不能为null
 
// Student student = new Student();
  Student student = null;
  String name = Optional.ofNullable(student).map(Student::getName).orElse("测试"); // 当student或name属性为null时,name为"测试"

// t1或s1或s1.getName()都为空时,fullName依然为orElse中的默认值!
        Teacher t1 = new Teacher(111L, "sdfsdf");
        Student s1 = new Student(5555L, "的说法都是");
        t1.setStudent(s1);
        String fullName = Optional.ofNullable(t1)
                .map(Teacher::getStudent)
                .map(Student::getName)
                .orElse("测试");

```
