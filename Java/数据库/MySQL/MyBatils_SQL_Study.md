

## MyBatis

### 单表通用查询--Example

#### 第一种写法:
<details>
<summary>代码演示:</summary>

```java

Example example = new Example(ODMInterviewerInformationT.class);
Example.Criteria criteria = example.createCriteria();
criteria.andEqualTo("applicantAccount",interviewer.getApplicantAccount());
if (!StringUtils.isEmpty(interviewer.getInterviewerType())){
	criteria.andEqualTo("interviewerType",interviewer.getInterviewerType());
}
if (!CollectionUtils.isEmpty(interviewer.getInterviewerTypeList())){
	criteria.andIn("interviewerType",interviewer.getInterviewerTypeList());
}
if (!StringUtils.isEmpty(interviewer.getInterviewLanguage())){
	criteria.andLike("interviewLanguage","%"+ interviewer.getInterviewLanguage() +"%");
}
criteria.setOrderByClause("resume_id desc,created_time desc limit 0,1");
List<ODMInterviewerInformationT> list = interviewerInformationTDao.selectByExample(example);


```
</details>

#### 第二种写法:
<details>
<summary>代码演示:</summary>

```java

Example example=Example.builder(OdrmResumeOperationHistoryT.class)
            .andWhere(Sqls.custom()
                .andEqualTo("resumeId",resumeId))
            .andWhere(Sqls.custom()
                .orEqualTo("operationType",ResumeOperatingType.RESUME_OPERATING_FREED.getOperatingType())
                .orEqualTo("operationType",ResumeOperatingType.RESUME_SLA_SCHEDULER.getOperatingType()))
            .orderByDesc("opratingTime")
        .build();
List<OdrmResumeOperationHistoryT> historyTS=operationHistoryTDao.selectByExample(example);
if(CollectionUtil.isNullOrEmpty(historyTS)){
        return null;
}

```
</details>
