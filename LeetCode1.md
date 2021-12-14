
```java

/*
 * Copyright (c) Huawei Technologies Co., Ltd. 2021-2021. All rights reserved.
 */

package com.huawei.it.omp.odrm.service.resume.impl;

import com.huawei.it.auth.util.StringUtils;
import com.huawei.it.jalor5.core.annotation.JalorOperation;
import com.huawei.it.jalor5.core.annotation.JalorResource;
import com.huawei.it.jalor5.core.exception.ApplicationException;
import com.huawei.it.jalor5.core.util.CollectionUtil;
import com.huawei.it.jalor5.core.util.StringUtil;
import com.huawei.it.jalor5.excel.exporter.service.impl.LocalExcelExportAssistant;
import com.huawei.it.jalor5.lookup.LookupItemVO;
import com.huawei.it.jalor5.lookup.service.ILookupItemQueryService;
import com.huawei.it.omp.odrm.common.core.ODRMJsonResult;
import com.huawei.it.omp.odrm.constant.LookupType;
import com.huawei.it.omp.odrm.constant.ResumeConstant;
import com.huawei.it.omp.odrm.constant.ResumeOperatingType;
import com.huawei.it.omp.odrm.constant.ResumeProcessPhaseType;
import com.huawei.it.omp.odrm.constant.ResumeProcessStatusType;
import com.huawei.it.omp.odrm.constant.UserRoleConstant;
import com.huawei.it.omp.odrm.dao.master.interview.OdrmInterviewerSchedulingTDao;
import com.huawei.it.omp.odrm.dao.master.interview.OdrmResumeMachineTestTDao;
import com.huawei.it.omp.odrm.dao.master.interview.OdrmResumeRecruitmentLinkTDao;
import com.huawei.it.omp.odrm.dao.master.interviewer.OdrmInterviewerInformationTDao;
import com.huawei.it.omp.odrm.dao.master.resume.OdrmPersonnelEducationHistoryTDao;
import com.huawei.it.omp.odrm.dao.master.resume.OdrmPersonnelProjectExperienceTDao;
import com.huawei.it.omp.odrm.dao.master.resume.OdrmPersonnelResumeBasicTDao;
import com.huawei.it.omp.odrm.dao.master.resume.OdrmPersonnelWorkExperienceTDao;
import com.huawei.it.omp.odrm.dao.master.resume.OdrmResumeLanguageAbilityTDao;
import com.huawei.it.omp.odrm.dao.master.resume.OdrmResumeOperationHistoryTDao;
import com.huawei.it.omp.odrm.dao.model.interview.OdrmInterviewerSchedulingT;
import com.huawei.it.omp.odrm.dao.model.interview.OdrmResumeMachineTestT;
import com.huawei.it.omp.odrm.dao.model.interviewer.OdrmInterviewerInformationT;
import com.huawei.it.omp.odrm.dao.model.resume.OdrmLsmsResumeT;
import com.huawei.it.omp.odrm.dao.model.resume.OdrmPersonnelResumeBasicT;
import com.huawei.it.omp.odrm.dao.model.resume.OdrmResumeOperationHistoryT;
import com.huawei.it.omp.odrm.entity.excel.interview.ProfessionalInterviewBaseVO;
import com.huawei.it.omp.odrm.entity.vo.interview.InterviewResultVO;
import com.huawei.it.omp.odrm.entity.vo.interview.ProfessionalInterviewVO;
import com.huawei.it.omp.odrm.entity.vo.resume.AttachmentInfoVO;
import com.huawei.it.omp.odrm.entity.vo.resume.LwmsResumeQueryVO;
import com.huawei.it.omp.odrm.entity.vo.resume.PersonnelEducationHistVO;
import com.huawei.it.omp.odrm.entity.vo.resume.PersonnelLanguageAbilityVO;
import com.huawei.it.omp.odrm.entity.vo.resume.PersonnelProjectExperienceVO;
import com.huawei.it.omp.odrm.entity.vo.resume.PersonnelResumeBasicVO;
import com.huawei.it.omp.odrm.entity.vo.resume.PersonnelResumeDetailVO;
import com.huawei.it.omp.odrm.entity.vo.resume.PersonnelWorkExperienceVO;
import com.huawei.it.omp.odrm.entity.vo.resume.ResumeOperationHistoryVO;
import com.huawei.it.omp.odrm.entity.vo.resume.ResumeQueryParamVO;
import com.huawei.it.omp.odrm.entity.vo.resume.ResumeResultVO;
import com.huawei.it.omp.odrm.entity.vo.resume.ResumeSlaVO;
import com.huawei.it.omp.odrm.entity.vo.resume.StandardResumeDataSetVO;
import com.huawei.it.omp.odrm.entity.vo.resume.datachange.PushDataVO;
import com.huawei.it.omp.odrm.entity.vo.resume.datachange.ResumePhaseVO;
import com.huawei.it.omp.odrm.entity.vo.sys.EnumVO;
import com.huawei.it.omp.odrm.entity.vo.sys.OrganizationVO;
import com.huawei.it.omp.odrm.entity.vo.sys.SupplierVO;
import com.huawei.it.omp.odrm.entity.vo.sys.SysUserVO;
import com.huawei.it.omp.odrm.exception.ODRMException;
import com.huawei.it.omp.odrm.service.common.impl.ResumeSlaCommon;
import com.huawei.it.omp.odrm.service.interview.impl.InterviewResultService;
import com.huawei.it.omp.odrm.service.resume.ResumeDataChangeService;
import com.huawei.it.omp.odrm.service.resume.ResumeQueryService;
import com.huawei.it.omp.odrm.service.sys.IUserUtiltools;
import com.huawei.it.omp.odrm.util.OdrmUtils;
import lombok.extern.slf4j.Slf4j;
import org.apache.http.HttpStatus;
import org.springframework.beans.BeanUtils;
import org.springframework.util.CollectionUtils;
import tk.mybatis.mapper.entity.Example;
import tk.mybatis.mapper.util.Sqls;

import javax.annotation.Resource;
import javax.inject.Inject;
import javax.inject.Named;
import java.lang.reflect.InvocationTargetException;
import java.math.BigDecimal;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;
import java.util.Objects;
import java.util.stream.Collectors;

/**
 * 简历数据变更
 *
 * @author lWX1023987
 * @since 2021-01-30
 */
@Named
@Slf4j
@JalorResource(code = "omp.odrm.res.DataService", desc = "简历数据变更")
public class ResumeDataChangeServiceImpl implements ResumeDataChangeService {
    @Resource
    private OdrmPersonnelResumeBasicTDao resumeBasicTDao;

    @Resource
    private OdrmResumeRecruitmentLinkTDao linkTDao;

    @JalorOperation(code = "omp.odrm.res.DataService.queryBy", desc = "简历数据变更-进页面前拉去数据")
    @Override
    public ODRMJsonResult<PushDataVO> queryBy(Long resumeId) {
        PushDataVO result = resumeBasicTDao.selectDataChange(resumeId);
        result.setRecruitmentLinkByShow(ResumeProcessPhaseType.getPhaseValueByCode(result.getRecruitmentLink()));
        linkTDao.selectMaxLinkName(resumeId);
        List<ResumePhaseVO> listData = getListData(result);
        result.setChangeType(listData);

        return ODRMJsonResult.successReturn();
    }

    private List<ResumePhaseVO> getListData(PushDataVO result) {
        String recruitmentLink = result.getRecruitmentLink();
        String recruitmentStatus = result.getRecruitmentStatus();
        Long resumeId = result.getResumeId();
        List<ResumePhaseVO> allTypes = setFullList();
        int maxSortValue = getMaxSortValue(allTypes, recruitmentLink, recruitmentStatus, resumeId);
        removeChildRen(allTypes, maxSortValue);
        return allTypes;
    }

    private int getMaxSortValue(List<ResumePhaseVO> allTypes,String recruitmentLink, String recruitmentStatus, Long resumeId) {
        int maxSortValue = 10;
        if (ResumeProcessPhaseType.RESUME_JOIN_HUAWEI.getCode().equalsIgnoreCase(recruitmentLink)) {
            maxSortValue = 100;
        } else if (ResumeProcessPhaseType.RESUME_INTERVIEW.getCode().equals(recruitmentLink) && ) {
        }



        int number = matchingData(allTypes, recruitmentLink);
        if (number != -1) {
            maxSortValue = number;
        }
        return maxSortValue;
    }


    private int matchingData(List<ResumePhaseVO> listData, String recruitmentLink) {
        for (ResumePhaseVO phaseVO : listData) {
            if (phaseVO.getValue().equals(recruitmentLink)) {
                return phaseVO.getSortValue();
            } else {
                List<ResumePhaseVO> children = phaseVO.getChildren();
                if (!CollectionUtil.isNullOrEmpty(children)) {
                    int data = matchingData(children, recruitmentLink);
                    if (data != -1) {
                        return data;
                    }
                }
            }
        }
        return -1;
    }

    private void removeChildRen(List<ResumePhaseVO> allTypes, int maxSortValue) {
        List<String> removeList = new ArrayList<>();
        for (ResumePhaseVO phaseVO : allTypes) {
            if (phaseVO.getSortValue().intValue() > maxSortValue) {
                removeList.add(phaseVO.getValue());
            } else {
                List<ResumePhaseVO> children = phaseVO.getChildren();
                if (!CollectionUtil.isNullOrEmpty(children)) {
                    removeChildRen(children, maxSortValue);
                }
            }
        }
        allTypes.removeIf(vo -> removeList.contains(vo.getValue()));
    }

    private List<ResumePhaseVO> setFullList() {
        List<ResumePhaseVO> allTypes = new ArrayList<>();
        // 【第一层】
        ResumePhaseVO phaseVO1 = new ResumePhaseVO("简历信息", ResumeProcessPhaseType.RESUME_BASIC.getCode(), 10);
        ResumePhaseVO phaseVO2 = new ResumePhaseVO("面试结果", ResumeProcessPhaseType.RESUME_INTERVIEW.getCode(), 20);
        ResumePhaseVO phaseVO3 = new ResumePhaseVO("租用结果", ResumeProcessPhaseType.RESUME_HIRES.getCode(), 30);
        ResumePhaseVO phaseVO4 = new ResumePhaseVO("offer结果", ResumeProcessPhaseType.RESUME_OFFER.getCode(), 40);

        // 【第二层】
        // 面试环节
        ResumePhaseVO phaseVO21 = new ResumePhaseVO("机考结果", ResumeProcessPhaseType.RESUME_MACHINE_TESTING.getCode(), 21);
        ResumePhaseVO phaseVO22 = new ResumePhaseVO("综合测评结果", ResumeProcessPhaseType.RESUME_COMPREHENSIVE_ASSESSMENT.getCode(), 22);
        ResumePhaseVO phaseVO23 = new ResumePhaseVO("资格面试结果", ResumeProcessPhaseType.RESUME_QUALIFICATION_INTERVIEW.getCode(), 23);
        ResumePhaseVO phaseVO24 = new ResumePhaseVO("专业面试1结果", ResumeProcessPhaseType.RESUME_PROFESSIONAL_INTERVIEW_FIR.getCode(), 24);
        ResumePhaseVO phaseVO25 = new ResumePhaseVO("专业面试2结果", ResumeProcessPhaseType.RESUME_PROFESSIONAL_INTERVIEW_SEC.getCode(), 25);
        ResumePhaseVO phaseVO26 = new ResumePhaseVO("追加面试结果", ResumeProcessPhaseType.RESUME_PROFESSIONAL_INTERVIEW_THR.getCode(), 26);
        ResumePhaseVO phaseVO27 = new ResumePhaseVO("最终确认结果", ResumeProcessPhaseType.RESUME_PROFESSIONAL_INTERVIEW_CONFIRMATION.getCode(), 27);
        ResumePhaseVO phaseVO28 = new ResumePhaseVO("业务主管面试结果", ResumeProcessPhaseType.RESUME_FINAL_INTERVIEW.getCode(), 28);
        List<ResumePhaseVO> phaseVO2All = new ArrayList<>();
        phaseVO2All.add(phaseVO21);
        phaseVO2All.add(phaseVO22);
        phaseVO2All.add(phaseVO23);
        phaseVO2All.add(phaseVO24);
        phaseVO2All.add(phaseVO25);
        phaseVO2All.add(phaseVO26);
        phaseVO2All.add(phaseVO27);
        phaseVO2All.add(phaseVO28);
        phaseVO2.setChildren(phaseVO2All);
        // offer环节
        ResumePhaseVO phaseVO41 = new ResumePhaseVO("offer下发", ResumeProcessPhaseType.RESUME_OFFER_DELIVER.getCode(), 41);
        ResumePhaseVO phaseVO42 = new ResumePhaseVO("offer沟通", ResumeProcessPhaseType.RESUME_OFFER_COMMUNICATION.getCode(), 42);
        List<ResumePhaseVO> phaseVO4All = new ArrayList<>();
        phaseVO4All.add(phaseVO41);
        phaseVO4All.add(phaseVO42);
        phaseVO4.setChildren(phaseVO4All);

        allTypes.add(phaseVO1);
        allTypes.add(phaseVO2);
        allTypes.add(phaseVO3);
        allTypes.add(phaseVO4);
        return allTypes;
    }
}



```
