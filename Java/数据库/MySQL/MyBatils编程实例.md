#### 1.返回结果
```java
   /**
     * 判断简历是否处于释放状态
     * @param resumeIds 简历主键ID集合
     * @return 条件匹配数据行数
     */
    List<Map<String , Object>> judgmentFreedStatus(@Param("resumeIds") List<Long> resumeIds);
```
```xml
<select id="judgmentFreedStatus" resultType="java.util.Map">
    SELECT
        basic.id resumeId,
        basic.resume_code resumeCode,
        count(1) resumeNum
    FROM
        odrm.odrm_personnel_resume_basic_t basic,
        odrm.odrm_resume_status_sla_t sla
    WHERE
            basic.id = sla.resume_id
        AND sla.resume_status = 0
        AND basic.id in
      <foreach collection="resumeIds" item="resumeId" open="(" separator="," close=")">
          #{resumeId}
      </foreach>
      group by basic.id;
  </select>
```
```java
List<Map<String , Object>> usageList = resumeSlaMapper.judgmentFreedStatus(resumeIds);
Integer resumeNum = (Integer)usageResume.get("resumeNum");
(Long)usageResume.get("resumeId");
(String)usageResume.get("resumeCode")
```

> 当不想创建VO封装查询结果时,可以用Map<String,Object>容器去封装数据!

2.
```java
 <if test="queryParam.interviewLanguages != null and queryParam.interviewLanguages.length > 0">
      and ( 1 = 1
      <foreach collection="queryParam.interviewLanguages" item="interviewLanguage">
      and interview_language like CONCAT('%',#{interviewLanguage},'%')
      </foreach>
      )
    </if>

<if test="queryParam.resumeIdsByLink != null and queryParam.resumeIdsByLink.size > 0">
        AND basic.id  in
<foreach collection="queryParam.resumeIdsByLink" item="resumeId" separator="," open="(" close=")">
        #{resumeId}
        </foreach>
        </if>

        <if test="queryParam.education != null and '' != queryParam.education">
        and basic.education = #{queryParam.education}
        </if>
```
