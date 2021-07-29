

> 习题资料:<br>
> 1.[Mysql经典练习题50题](https://blog.csdn.net/original_recipe/article/details/91958663) <br>






---
#### 1.排行表
1.[分数排名](https://leetcode-cn.com/problems/rank-scores/description/) <br>
2.[在MySQL中实现Rank高级排名函数](https://blog.csdn.net/qq686867/article/details/79355760?utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control) <br>
解法一:<br>
```sql
SELECT
    S1.score 'Score',
    COUNT( DISTINCT S2.score ) 'Rank'
FROM
    Scores S1
    INNER JOIN Scores S2
    ON S1.score <= S2.score
GROUP BY
    S1.id, S1.score
ORDER BY
    S1.score DESC;
```
> 先观察这条sql的执行结果!
```sql
select *
FROM Scores S1
         INNER JOIN Scores S2
                    ON S1.score <= S2.score
ORDER BY S1.id, S1.score
```
按S1.id, S1.score进行分组,这样可以保证相同分数也不会分到一组,因为id肯定是不一样的(其实只按id分组效果也一样!)
