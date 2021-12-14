### Optional常用用法

```Java
// 判空赋默认值,Optional类比较简便
  String fullName = matchResume.getFullName() == null ? "" : matchResume.getFullName();
  String fullName = Optional.ofNullable(matchResume.getFullName()).orElse(""); // matchResume对象不能为null
 
// Student student = new Student();
  Student student = null;
  String name = Optional.ofNullable(student).map(Student::getName).orElse("测试"); // 当student或name属性为null时,name为"测试"

```
