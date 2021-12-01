

> [MySQL索引原理以及查询优化](https://www.cnblogs.com/bypp/p/7755307.html)


## 时间范围查询
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm",timezone = "GMT+8")
    private Date applicantStar;

    @JsonFormat(pattern = "yyyy-MM-dd HH:mm",timezone = "GMT+8")
    private Date applicantEnd;
 
 
<!-- 例外日期范围查询 -->
<if test=" queryParam.applicantStar != null">
  AND DATE_FORMAT(appeal.created_time, '%Y-%m-%d %H:%i') <![CDATA[>=]]> DATE_FORMAT(#{queryParam.applicantStar}, '%Y-%m-%d %H:%i')
</if>
<if test="queryParam.applicantEnd!= null">
  AND DATE_FORMAT(appeal.created_time, '%Y-%m-%d %H:%i') <![CDATA[<=]]> DATE_FORMAT(#{queryParam.applicantEnd}, '%Y-%m-%d %H:%i')
</if>


<sql id="queryAllAppealCondiction">
    <if test="queryParam.processCode != null and '' != queryParam.processCode">
      and instanceT.process_code = #{queryParam.processCode}
    </if>
    <if test="queryParam.appealTitle != null and '' != queryParam.appealTitle">
      and appeal.appeal_title like CONCAT('%',#{queryParam.appealTitle},'%')
    </if>
    <if test="queryParam.appealReason != null and '' != queryParam.appealReason">
      and appeal.appeal_reason like CONCAT('%',#{queryParam.appealReason},'%')
    </if>
    <!-- 申请人的ID -->
    <if test="queryParam.appealerId != null">
      and appeal.appealer_id = #{queryParam.appealerId}
    </if>
    <!-- 申请人的姓名 -->
    <if test="queryParam.applicant != null and '' != queryParam.applicant">
      and appeal.created_by like CONCAT('%',#{queryParam.applicant},'%')
    </if>
    <!-- 当前处理人的ID -->
    <if test="queryParam.reviewerId != null">
      and appeal.reviewer_id = #{queryParam.reviewerId}
    </if>
    <if test="queryParam.appealOperator != null and '' != queryParam.appealOperator">
      and instanceT.processor_account like CONCAT('%',#{queryParam.appealOperator},'%')
    </if>
    <!-- 处理状态 -->
    <if test="queryParam.processStatus != null">
      and instanceT.process_status = #{queryParam.processStatus}
    </if>
    <!-- 例外日期范围查询 -->
    <if test=" queryParam.applicantStar != null">
      AND DATE_FORMAT(appeal.created_time, '%Y-%m-%d %H:%i') <![CDATA[>=]]> DATE_FORMAT(#{queryParam.applicantStar}, '%Y-%m-%d %H:%i')
    </if>
    <if test="queryParam.applicantEnd!= null">
      AND DATE_FORMAT(appeal.created_time, '%Y-%m-%d %H:%i') <![CDATA[<=]]> DATE_FORMAT(#{queryParam.applicantEnd}, '%Y-%m-%d %H:%i')
    </if>
    <!-- 查询当前BU下的数据 -->
    <if test="queryParam.buIds != null and queryParam.buIds.size > 0">
      AND appeal.bu_id  in
      <foreach collection="queryParam.buIds" item="buId" separator="," open="(" close=")">
        #{buId}
      </foreach>
    </if>
    <!-- 匹配excel导出勾选流程表id的数据 -->
    <if test="queryParam.instanceIdsOfExcelExport != null and queryParam.instanceIdsOfExcelExport.size > 0">
      AND instanceT.id in
      <foreach collection="queryParam.instanceIdsOfExcelExport" item="instanceId" separator="," open="(" close=")">
        #{instanceId}
      </foreach>
    </if>
  </sql>
