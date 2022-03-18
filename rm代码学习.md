```java


/*
 * Copyright (c) Huawei Technologies Co., Ltd. 2021-2021. All rights reserved.
 */

package com.huawei.it.omp.odrm.service.messageconsumer.impl;

import com.huawei.cs.json.JSONException;
import com.huawei.cs.json.JSONObject;
import com.huawei.it.auth.exception.AuthException;
import com.huawei.it.eip.ump.client.consumer.ConsumeStatus;
import com.huawei.it.eip.ump.client.consumer.Consumer;
import com.huawei.it.eip.ump.client.listener.MessageListener;
import com.huawei.it.eip.ump.common.exception.UmpException;
import com.huawei.it.eip.ump.common.message.Message;
import com.huawei.it.eip.ump.common.util.DateUtil;
import com.huawei.it.jalor5.core.exception.ApplicationException;
import com.huawei.it.jalor5.core.ioc.Jalor;
import com.huawei.it.jalor5.core.util.StringUtil;
import com.huawei.it.jalor5.registry.service.IRegistryQueryService;
import com.huawei.it.omp.odrm.constant.LookupType;
import com.huawei.it.omp.odrm.constant.ResumeProcessPhaseType;
import com.huawei.it.omp.odrm.constant.ResumeProcessStatusType;
import com.huawei.it.omp.odrm.constant.WelinkMessageConstant;
import com.huawei.it.omp.odrm.constant.company.HCMJoinCompanyType;
import com.huawei.it.omp.odrm.dao.master.company.OdrmHcmEnterCompanyTDao;
import com.huawei.it.omp.odrm.dao.master.company.OdrmHcmJoinTiDao;
import com.huawei.it.omp.odrm.dao.master.interview.OdrmResumeRecruitmentLinkTDao;
import com.huawei.it.omp.odrm.dao.master.resume.OdrmPersonnelResumeBasicTDao;
import com.huawei.it.omp.odrm.dao.master.resume.OdrmResumeStatusSlaTDao;
import com.huawei.it.omp.odrm.dao.model.company.OdrmHcmEnterCompanyT;
import com.huawei.it.omp.odrm.dao.model.company.OdrmHcmJoinTi;
import com.huawei.it.omp.odrm.dao.model.interview.OdrmResumeRecruitmentLinkT;
import com.huawei.it.omp.odrm.dao.model.resume.OdrmPersonnelResumeBasicT;
import com.huawei.it.omp.odrm.dao.model.resume.OdrmResumeStatusSlaT;
import com.huawei.it.omp.odrm.entity.vo.common.WelinkMessageContent;
import com.huawei.it.omp.odrm.entity.vo.common.WelinkMessageTitle;
import com.huawei.it.omp.odrm.exception.OdrmWbException;
import com.huawei.it.omp.odrm.service.common.impl.WelinkMessageCommon;
import com.huawei.it.omp.odrm.util.DateUtils;

import lombok.extern.slf4j.Slf4j;

import org.apache.commons.lang3.StringUtils;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.stereotype.Component;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;

import tk.mybatis.mapper.entity.Example;

import javax.annotation.Resource;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.Date;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * ConsumerSubscribeJoinTask
 * 消费由hcm那边审批发送过来的消息
 *
 * @author lwx1036315
 * @since 2021年7月20日
 */
@Slf4j
@Component
public class ConsumerSubscribeJoinTask implements ApplicationRunner {
    @javax.annotation.Resource
    private OdrmHcmJoinTiDao odrmHcmJoinTiDao;

    @javax.annotation.Resource
    private OdrmHcmEnterCompanyTDao hcmEnterCompanyTDao;

    @javax.annotation.Resource
    private OdrmResumeRecruitmentLinkTDao linkTDao;

    @javax.annotation.Resource
    private OdrmResumeStatusSlaTDao slaTDao;

    @javax.annotation.Resource
    private DataSourceTransactionManager dataSourceTransactionManager;

    @javax.annotation.Resource
    private TransactionDefinition transactionDefinition;

    @Resource
    private WelinkMessageCommon welinkMessageCommon;

    @Resource
    private OdrmPersonnelResumeBasicTDao basicTDao;

    private String appId;

    private String currentEvn;

    private String credential;

    private String topic;

    private String joinTag;

    private String umpNamesrvUrls;

    private String subGroup;

    private final Object poolSizeMonitor = new Object();
    private int corePoolSize = 1;
    private int maxPoolSize = 2147483647;
    private int keepAliveSeconds = 60;
    private int queueCapacity = 2147483647;

    private ThreadPoolExecutor threadPoolExecutor;

    /**
     * consumer subscribe join task
     *
     */
    public ConsumerSubscribeJoinTask() {
        super();
    }

    /**
     * run
     *
     * @param args args
     * @throws Exception exception
     */
    @Override
    public void run(ApplicationArguments args) throws Exception {
        try {
            // 初始化参数时，如果发现是dev环境则会退出订阅
            if (init()) {
                String msg1 = "当前MQS的运行环境是" + this.currentEvn + "环境，ConsumerSubscribeJoinTask开始订阅消息服务";
                log.info(msg1);
                subscribe();
                String msg2 = "当前MQS的运行环境是" + this.currentEvn + "环境，ConsumerSubscribeJoinTask开始订阅消息结束";
                log.info(msg2);
                String msg3 = "当前MQS的运行环境是" + this.currentEvn + "环境，ConsumerSubscribeJoinTask开始初始化线程池";
                log.info(msg3);
                initialize();
                String msg4 = "当前MQS的运行环境是" + this.currentEvn + "环境，ConsumerSubscribeJoinTask结束初始化线程池";
                log.info(msg4);
            }else {
                String msg = "当前MQS的运行环境是dev环境，ConsumerSubscribeJoinTask取消初始化订阅消息服务";
                log.warn(msg);
            }
        } catch (IOException | ApplicationException e) {
            String msg = "入司消息订阅服务异常：";
            log.error(msg, e);
            throw e;
        }
    }

    /**
     * init
     * 一开始想用 @PostConstruct 后来发现不行，IRegistryQueryService公服上还没注册拿不到
     *
     * @return the boolean
     * @throws IOException ioexception
     * @throws ApplicationException application exception
     */
    public boolean init() throws IOException, ApplicationException {
        IRegistryQueryService registryQueryService = null;
        if (Jalor.getContext().getBean("registryQueryService") instanceof IRegistryQueryService) {
            registryQueryService = (IRegistryQueryService)Jalor.getContext().getBean("registryQueryService");
        }
        if (registryQueryService != null) {
            this.appId = registryQueryService.findRegistryByPath("App.odrm.IntegrationHCM.appId",
                    true).getValue();
            this.subGroup = registryQueryService.findRegistryByPath("App.odrm.IntegrationHCM.hcmEnv",
                    true).getValue();
            this.topic = registryQueryService.findRegistryByPath("App.odrm.IntegrationHCM.topic_join",
                    true).getValue();
            this.joinTag = registryQueryService.findRegistryByPath("App.odrm.IntegrationHCM.topic_join_tag",
                    true).getValue();
            this.umpNamesrvUrls = registryQueryService.findRegistryByPath("App.odrm.IntegrationHCM.umpNamesrvUrls",
                    true).getValue();
            this.credential = registryQueryService.findRegistryByPath("App.odrm.IntegrationHCM.credential",
                    true).getValue();
        }
        return true;
    }

    /**
     * subscribe
     */
    public void subscribe() {
        String msg = "odrm服务的：ump message start";
        log.info(msg);
        try {
            Consumer consumer = new Consumer();
            consumer.setUmpNamesrvUrls(umpNamesrvUrls); // 设置统一消息平台的服务器地址(dev环境)
            consumer.setEncryptTransport(false); // 设置是否需要加密传输
            consumer.setAppSecret(credential); // 设置客户端密钥
            consumer.setSubGroup(subGroup);
            consumer.setTopic(topic); // 设置Topic Name
            consumer.setTags(joinTag); // 设置订阅消息的标签，可以指定消费某一类型的消息，默认*表示消费所有类型的消息
            consumer.setAppId(appId); // 设置客户端账号
            subscribe(consumer);
            consumer.start(); // 启动消费者，建议在应用程序关闭的时候关闭。
        } catch (UmpException e) {
            log.error(e.getMessage());
            return;
        }
        String msg1 = "odrm服务的：ump message end!";
        log.info(msg1);
    }

    private void subscribe(Consumer consumer) {
        consumer.subscribe(new MessageListener() {
            /**
             * consume
             *
             * @param message message
             * @return ConsumeStatus
             */
            public ConsumeStatus consume(Message message) {
                try {
                    String returnJsonString = new String(message.getBody(), StandardCharsets.UTF_8);
                    log.info("odrm服务的：get ump info: " + returnJsonString);
                    if (StringUtils.isNotBlank(returnJsonString)) {
                        JSONObject object = new JSONObject(returnJsonString);
                        OdrmHcmJoinTi vo = new OdrmHcmJoinTi();
                        Long sourceId = null;
                        String integratedSourceId = object.getString("integratedSourceId");
                        try {
                            sourceId = Long.valueOf(integratedSourceId);
                        } catch (NumberFormatException e) {
                            log.error("系统主要业务id转化异常：", e);
                        }
                        setVOInfo(object, vo, integratedSourceId);
                        if (null == odrmHcmJoinTiDao) {
                            odrmHcmJoinTiDao = Jalor.getContext().getBean(OdrmHcmJoinTiDao.class);
                        }
                        odrmHcmJoinTiDao.insertSelective(vo);
                        if ( (HCMJoinCompanyType.FINISHED_JOIN.getCode().equals(vo.getJoinStatus())
                                || HCMJoinCompanyType.CANCEL_TO_JOIN.getCode().equals(vo.getJoinStatus()))
                                && sourceId != null && sourceId > 0) {
                            handle(sourceId, vo.getJoinStatus());
                        }
                        log.info("odrm服务的：finished ump info consume: ");
                    }
                    return ConsumeStatus.CONSUME_SUCCESS;
                } catch (JSONException e) {
                    log.error("odrm服务的：get ump info error:" + e.getMessage());
                    return ConsumeStatus.RECONSUME_LATER;
                }
            }
        });
    }

    private void setVOInfo(JSONObject object, OdrmHcmJoinTi vo, String integratedSourceId) {
        vo.setIntegratedSourceId(integratedSourceId); // 数据来源
        vo.setIntegratedSystemId(object.getString("integratedSystemId")); // 集成系统业务主键
        vo.setUuid(object.getString("UUID")); // 集成系统业务主键
        vo.setBillNo(object.getString("billNo")); // 单据编号
        // JA010    暂存  JA020   审批中  JA030    待重新提交  JA040  审批通过  JA050 已终止
        vo.setBillStatus(object.getString("billStatus")); // 审批状态
        vo.setEditEmpName(object.getString("editEmpName")); // 操作人姓名
        vo.setEditEmpNumber(object.getString("editEmpNumber")); // 操作人编号
        vo.setEmpName(object.getString("empName")); // 姓名
        vo.setEmpNameEn(object.getString("empNameEN")); // 拼音名
        vo.setEmpNumber(object.getString("empNumber")); // 编号
        // JE010    审批通过 JE020  审批不通过 JE030   单据驳回 JE040  确认入司 JE050  终止入司 JE060  改期入司 JE070  编号生成
        vo.setEventCode(object.getString("eventCode")); // 事件编码
        vo.setEventText(object.getString("eventText")); // 事件内容
        if (!StringUtil.isNullOrEmpty(object.getString("eventTime")) &&
                !"null".equalsIgnoreCase(object.getString("eventTime"))) {
            vo.setEventTime(DateUtil.toDate("yyyy-MM-dd HH:mm:ss",
                    object.getString("eventTime"))); // 事件时间
        }
        if (!StringUtil.isNullOrEmpty(object.getString("expectJoinDate")) &&
                !"null".equalsIgnoreCase(object.getString("expectJoinDate"))) {
            vo.setExpectJoinDate(DateUtil.toDate("yyyy-MM-dd HH:mm:ss",
                    object.getString("expectJoinDate"))); // 预计入司日期
        }
        if (!StringUtil.isNullOrEmpty(object.getString("realityJoinDate")) &&
                !"null".equalsIgnoreCase(object.getString("realityJoinDate"))) {
            vo.setRealityJoinDate(DateUtil.toDate("yyyy-MM-dd HH:mm:ss",
                    object.getString("realityJoinDate"))); // 实际入司日期
        }
        if (!StringUtil.isNullOrEmpty(object.getString("terminationDate")) &&
                !"null".equalsIgnoreCase(object.getString("terminationDate"))) {
            vo.setTerminationDate(DateUtil.toDate("yyyy-MM-dd HH:mm:ss",
                    object.getString("terminationDate"))); // 终止入司日期
        }
        // 1010    待入司 1020 待确认入司 1030   已入司 1040 终止入司
        vo.setJoinStatus(object.getString("joinStatus")); // 人员入司状态
        vo.setPayroll(object.getString("payroll")); // 费用结算主体
        vo.setOrgCode(object.getString("orgCode")); // 部门编码
        vo.setOrgName(object.getString("orgName")); // 部门名称
        vo.setBaseLocation(object.getString("baseLocation")); // 常驻工作地
        vo.setShift(object.getString("shift")); // 班次
        vo.setCreateTime(new Date());
        vo.setCreateBy("系统自动拉取ump消息");
    }

    /**
     * 初始化线程池，使用阻塞队列
     */
    public void initialize() {
        BlockingQueue<Runnable> queue = this.createQueue(this.queueCapacity);
        this.threadPoolExecutor = this.createThreadPoolExecutor(queue);
    }

    /**
     * get core pool size
     *
     * @return the int
     */
    public int getCorePoolSize() {
        synchronized(this.poolSizeMonitor) {
            return this.corePoolSize;
        }
    }

    /**
     * set max pool size
     *
     * @param maxPoolSize max pool size
     */
    public void setMaxPoolSize(int maxPoolSize) {
        synchronized(this.poolSizeMonitor) {
            this.maxPoolSize = maxPoolSize;
            if (this.threadPoolExecutor != null) {
                this.threadPoolExecutor.setMaximumPoolSize(maxPoolSize);
            }
        }
    }

    /**
     * get max pool size
     *
     * @return the int
     */
    public int getMaxPoolSize() {
        synchronized(this.poolSizeMonitor) {
            return this.maxPoolSize;
        }
    }

    /**
     * set keep alive seconds
     *
     * @param keepAliveSeconds keep alive seconds
     */
    public void setKeepAliveSeconds(int keepAliveSeconds) {
        synchronized(this.poolSizeMonitor) {
            this.keepAliveSeconds = keepAliveSeconds;
            if (this.threadPoolExecutor != null) {
                this.threadPoolExecutor.setKeepAliveTime((long)keepAliveSeconds, TimeUnit.SECONDS);
            }
        }
    }

    /**
     * get keep alive seconds
     *
     * @return the int
     */
    public int getKeepAliveSeconds() {
        synchronized(this.poolSizeMonitor) {
            return this.keepAliveSeconds;
        }
    }


    /**
     * create thread pool executor
     *
     * @param queue queue
     * @return the thread pool executor
     */
    protected ThreadPoolExecutor createThreadPoolExecutor(BlockingQueue<Runnable> queue) {
        return new ThreadPoolExecutor(this.getCorePoolSize(), this.getMaxPoolSize(), this.getKeepAliveSeconds(),
                TimeUnit.SECONDS, queue, (key1, executor) -> {
            Long sourceId = ((NamedRunnable)key1).getSourceId();
            String joinStatus = "";
            if (key1 instanceof NamedRunnable) {
                joinStatus = ((NamedRunnable)key1).getJoinStatus();
            }
            log.warn("Threads conduit could not process Odrm-HCM join or leave company (sourceId:" + sourceId +
                    " joinStatus:" + joinStatus + "), maybe threads exceed max threads(" +
                    ConsumerSubscribeJoinTask.this.getMaxPoolSize() + ") and task queue is full!");
        });
    }

    /**
     * create queue
     *
     * @param queueCapacity queue capacity
     * @return the blocking queue < runnable >
     */
    protected BlockingQueue<Runnable> createQueue(int queueCapacity) {
        return queueCapacity > 0 ? new LinkedBlockingQueue(queueCapacity) : new SynchronousQueue();
    }

    /**
     * handle
     *
     * @param sourceId source id
     * @param joinStatus join status
     */
    public void handle(Long sourceId, String joinStatus) {
        this.threadPoolExecutor.execute(new ConsumerSubscribeJoinTask.NamedRunnable(sourceId, joinStatus));
    }

    /**
     * destroy
     *
     * @throws ApplicationException application exception
     */
    public void destroy() throws ApplicationException {
        this.threadPoolExecutor.shutdown();
    }

    /**
     * named runnable
     *
     * @since 2021-11-24
     */
    class NamedRunnable implements Runnable {
        private Long sourceId;
        private String joinStatus;

        /**
         * named runnable
         *
         * @param sourceId source id
         * @param joinStatus join status
         */
        public NamedRunnable(Long sourceId, String joinStatus) {
            this.sourceId = sourceId;
            this.joinStatus = joinStatus;
        }

        /**
         * run
         *
         */
        public void run() {
            if ((HCMJoinCompanyType.FINISHED_JOIN.getCode().equals(joinStatus)
                    || HCMJoinCompanyType.CANCEL_TO_JOIN.getCode().equals(joinStatus))
                    && sourceId != null && sourceId > 0) {
                if (null == hcmEnterCompanyTDao) {
                    hcmEnterCompanyTDao = Jalor.getContext().getBean(OdrmHcmEnterCompanyTDao.class);
                }
                if (null != hcmEnterCompanyTDao) {
                    String msg1 = "odrm的MQ的处理 sourceId{},joinStatus{} 业务数据开始";
                    log.info(msg1, sourceId, joinStatus);
                    updateInfo(hcmEnterCompanyTDao, sourceId, joinStatus);
                    String msg = "odrm的MQ的处理 sourceId{},joinStatus{} 业务数据结束";
                    log.info(msg, sourceId, joinStatus);
                }else {
                    String msg = "odrm的MQ的处理sourceId{},joinStatus{}时未获取到odrmHcmJoinTiDao对象,无法进行近一步的操作";
                    log.info(msg, sourceId, joinStatus);
                }
            }
        }

        /**
         * 入司成功、失败处理的业务逻辑
         *
         * @param hcmEnterCompanyTDao 入司核心业务表的dao
         * @param sourceId 入司核心表的主键id
         * @param joinStatus 入司状态
         */
        private void updateInfo(OdrmHcmEnterCompanyTDao hcmEnterCompanyTDao, Long sourceId, String joinStatus) {
            OdrmHcmEnterCompanyT hcmEnterCompanyT = getHcmEnterCompanyT(hcmEnterCompanyTDao, sourceId);
            if (null == hcmEnterCompanyT) {
                String msg = "odrlam的MQ的处理 sourceId{},joinStatus{} 根据sourceId未查出有效的数据，更新入司状态失败！";
                log.info(msg, sourceId, joinStatus);
                return;
            }

            log.info("ConsumerSubscribeJoinTask-NamedRunnable-updateInfo start transaction！");
            // 开启事务
            TransactionStatus transactionStatus = dataSourceTransactionManager.getTransaction(transactionDefinition);

            // 更新sla表
            OdrmResumeStatusSlaT slaT = setSlaInfo(joinStatus, hcmEnterCompanyT);
            Date updateTime = new Date();
            slaT.setModifiedTime(updateTime);
            String updateMan = HCMJoinCompanyType.SYSTEM_OPERATION_MAN.getCode();
            slaT.setModifiedBy(updateMan);
            // 对应的条件
            Example exampleSla = getSlaExample(hcmEnterCompanyT);
            int slaCount = slaTDao.updateByExampleSelective(slaT, exampleSla);
            String msg8 = "开始更新OdrmResumeStatusSlaT表----结束，更新数据的行数为：";
            log.info(msg8 + slaCount);
            if (slaCount <= 0) {
                slaTransactionRollBack(transactionStatus);
                return;
            }

            // 更新link环节表数据
            OdrmResumeRecruitmentLinkT linkT = setLinkInfo(joinStatus, updateTime, updateMan);
            Example exampleLink = getExampleLink(hcmEnterCompanyT);
            int linkCount = linkTDao.updateByExampleSelective(linkT, exampleLink);
            String msg6 = "开始更新OdrmResumeRecruitmentLinkT表----结束，更新数据的行数为：";
            log.info(msg6 + linkCount);
            if (linkCount <= 0) {
                linkTransactionRollBack(transactionStatus);
                return;
            }

            /**
             * JOIN_COMPANY joinCompanyTemplate
             * odrm_join_company_completed
             * 【OD招聘-SLA到期提醒】${employeeName}【${resumeCode}】已经${joinCompanyMsg}
             * 入司成功或者入司取消提醒
             * ${employeeName}【${resumeCode}】已经${joinCompanyMsg}
             */
            // 根据简历id，查询简历的相关信息
            OdrmPersonnelResumeBasicT resumeBasicT = basicTDao.selectByPrimaryKey(hcmEnterCompanyT.getResumeId());
            WelinkMessageContent content = getAndSetWelinkMessageContent(joinStatus, resumeBasicT);
            // 更新入司主表
            OdrmHcmEnterCompanyT enterCompanyT = getAndSetHcmEnterCompanyT(sourceId, joinStatus, updateTime, updateMan);
            int enterCount = hcmEnterCompanyTDao.updateByPrimaryKeySelective(enterCompanyT);
            String msg4 = "开始更新OdrmHcmEnterCompanyT表----结束，更新数据的行数为：";
            log.info(msg4 + enterCount);
            if (enterCount <= 0) {
                enterCompanyTransactionRollBack(transactionStatus);
                return;
            }

            commitTranction(transactionStatus);

            sendWeLinkMessage(resumeBasicT, content);
        }

        /**
         * get source id
         *
         * @return the long
         */
        public Long getSourceId() {
            return sourceId;
        }

        /**
         * set source id
         *
         * @param sourceId source id
         */
        public void setSourceId(Long sourceId) {
            this.sourceId = sourceId;
        }

        /**
         * get join status
         *
         * @return the string
         */
        public String getJoinStatus() {
            return joinStatus;
        }

        /**
         * set join status
         *
         * @param joinStatus join status
         */
        public void setJoinStatus(String joinStatus) {
            this.joinStatus = joinStatus;
        }


    }

    private void sendWeLinkMessage(OdrmPersonnelResumeBasicT resumeBasicT, WelinkMessageContent content) {
        WelinkMessageTitle title = new WelinkMessageTitle();
        title.setEmployeeName(resumeBasicT.getEmployeeName());
        title.setResumeCode(resumeBasicT.getResumeCode());
        try {
            welinkMessageCommon.sendWelinkMessage(WelinkMessageConstant.JOIN_COMPANY, content,
                    title, resumeBasicT.getOperatorId(), "");
        } catch (ApplicationException | IllegalAccessException | AuthException | OdrmWbException e) {
            log.info("send welink message error", e);
        }
    }

    private WelinkMessageContent getAndSetWelinkMessageContent(String joinStatus,
                OdrmPersonnelResumeBasicT resumeBasicT) {
        WelinkMessageContent content = new WelinkMessageContent();
        content.setEmployeeName(resumeBasicT.getEmployeeName());
        content.setResumeCode(resumeBasicT.getResumeCode());
        if (HCMJoinCompanyType.FINISHED_JOIN.getCode().equals(joinStatus)) {
            content.setJoinCompanyMsg("入司完成");
        } else if (HCMJoinCompanyType.CANCEL_TO_JOIN.getCode().equals(joinStatus)) {
            content.setJoinCompanyMsg("取消入司");
        } else {
            log.error("join type error");
        }
        return content;
    }

    private void commitTranction(TransactionStatus transactionStatus) {
        String msg1 = "ConsumerSubscribeJoinTask-NamedRunnable-updateInfo准备提交事务！";
        log.info(msg1);
        dataSourceTransactionManager.commit(transactionStatus); // 提交事务
        String msg = "ConsumerSubscribeJoinTask-NamedRunnable-updateInfo提交事务成功！数据处理正常！";
        log.info(msg);
    }

    private void enterCompanyTransactionRollBack(TransactionStatus transactionStatus) {
        String msg3 = "更新OdrmHcmEnterCompanyT表数据异常，未更新到数据，为确保三张业务表的数据更新一致性，回滚事务";
        log.info(msg3);
        dataSourceTransactionManager.rollback(transactionStatus); // 回滚事务
        String msg2 = "OdrmHcmEnterCompanyT之前的事务回滚成功，将不执行后续业务，退出方法！";
        log.info(msg2);
    }

    private OdrmHcmEnterCompanyT getAndSetHcmEnterCompanyT(Long sourceId, String joinStatus,
                Date updateTime, String updateMan) {
        String msg5 = "开始更新OdrmHcmEnterCompanyT表----开始";
        log.info(msg5);
        OdrmHcmEnterCompanyT enterCompanyT = new OdrmHcmEnterCompanyT();
        enterCompanyT.setId(sourceId);
        if (HCMJoinCompanyType.FINISHED_JOIN.getCode().equals(joinStatus)) {
            enterCompanyT.setJoinStatus("2");
        } else if (HCMJoinCompanyType.CANCEL_TO_JOIN.getCode().equals(joinStatus)) {
            enterCompanyT.setJoinStatus("3");
        } else {
            log.info("HCMJoinCompanyType is fail");
        }
        enterCompanyT.setModifyTime(updateTime);
        enterCompanyT.setModifyBy(updateMan);
        return enterCompanyT;
    }

    private void linkTransactionRollBack(TransactionStatus transactionStatus) {
        String msg = "更新OdrmResumeRecruitmentLinkT表数据异常，未更新到数据，为确保三张业务表的数据更新一致性，回滚事务";
        log.info(msg);
        dataSourceTransactionManager.rollback(transactionStatus); // 回滚事务
        String msg1 = "OdrmResumeRecruitmentLinkT之前的事务回滚成功，将不执行后续业务，退出方法！";
        log.info(msg1);
    }

    private Example getExampleLink(OdrmHcmEnterCompanyT hcmEnterCompanyT) {
        Example exampleLink = new Example(OdrmResumeRecruitmentLinkT.class);
        Example.Criteria criteriaLink = exampleLink.createCriteria();
        // 更新数据的条件
        criteriaLink.andEqualTo("resumeId", hcmEnterCompanyT.getResumeId());
        criteriaLink.andEqualTo("linkName", ResumeProcessPhaseType.RESUME_JOIN_HUAWEI.getCode());
        criteriaLink.andEqualTo("linkVersion", hcmEnterCompanyT.getResumeVersion());
        criteriaLink.andEqualTo("linkState", "0");
        return exampleLink;
    }

    private OdrmResumeRecruitmentLinkT setLinkInfo(String joinStatus, Date updateTime, String updateMan) {
        String msg7 = "开始更新OdrmResumeRecruitmentLinkT表----开始";
        log.info(msg7);
        OdrmResumeRecruitmentLinkT linkT = new OdrmResumeRecruitmentLinkT();

        if (HCMJoinCompanyType.FINISHED_JOIN.getCode().equals(joinStatus)) {
            linkT.setLinkStatus(ResumeProcessStatusType.ENROLLED_COMPLETE.getCode());
        } else if (HCMJoinCompanyType.CANCEL_TO_JOIN.getCode().equals(joinStatus)) {
            linkT.setLinkStatus(ResumeProcessStatusType.ENROLLED_CANCELS.getCode());
        } else {
            log.info("HCMJoinCompanyType is fail");
        }
        linkT.setModifiedTime(updateTime);
        linkT.setModifiedBy(updateMan);
        return linkT;
    }

    private void slaTransactionRollBack(TransactionStatus transactionStatus) {
        String msg = "更新OdrmResumeStatusSlaT表数据异常，未更新到数据，为确保三张业务表的数据更新一致性，回滚事务";
        log.info(msg);
        dataSourceTransactionManager.rollback(transactionStatus); // 回滚事务
        String msg1 = "OdrmResumeStatusSlaT之前的事务回滚成功，将不执行后续业务，退出方法！";
        log.info(msg1);
    }

    private Example getSlaExample(OdrmHcmEnterCompanyT hcmEnterCompanyT) {
        Example exampleSla = new Example(OdrmResumeStatusSlaT.class);
        Example.Criteria criteriaSla = exampleSla.createCriteria();
        criteriaSla.andEqualTo("resumeId", hcmEnterCompanyT.getResumeId());
        criteriaSla.andEqualTo("resumeStatus", "0");
        criteriaSla.andEqualTo("exceptionsStatus", "0");
        return exampleSla;
    }

    private OdrmResumeStatusSlaT setSlaInfo(String joinStatus, OdrmHcmEnterCompanyT hcmEnterCompanyT) {
        String msg9 = "开始更新OdrmResumeStatusSlaT表----开始";
        log.info(msg9);
        OdrmResumeStatusSlaT slaT = new OdrmResumeStatusSlaT();
        if (HCMJoinCompanyType.FINISHED_JOIN.getCode().equals(joinStatus)) {
            slaT.setSlaControlPoint(DateUtils.getCurrentDate());
            slaT.setRecruitmentStatus(ResumeProcessStatusType.ENROLLED_COMPLETE.getCode());
            slaT.setSlaPhaseType(LookupType.JOIN_HUAWEI_SUCCESS.getItemCode());
        } else if (HCMJoinCompanyType.CANCEL_TO_JOIN.getCode().equals(joinStatus)) {
            slaT.setSlaControlPoint(DateUtils.getCurrentDate());
            slaT.setRecruitmentStatus(ResumeProcessStatusType.ENROLLED_CANCELS.getCode());
            slaT.setSlaPhaseType(LookupType.JOIN_HUAWEI_CANCELS.getItemCode());
        } else {
            log.info("HCMJoinCompanyType is fail");
        }
        slaT.setResumeId(hcmEnterCompanyT.getResumeId());
        return slaT;
    }

    private OdrmHcmEnterCompanyT getHcmEnterCompanyT(OdrmHcmEnterCompanyTDao hcmEnterCompanyTDao, Long sourceId) {
        Example example = new Example(OdrmHcmEnterCompanyT.class);
        Example.Criteria criteria = example.createCriteria();
        criteria.andEqualTo("id", sourceId);
        criteria.andEqualTo("deleteFlag", false);
        criteria.andEqualTo("isEffective", true);
        return hcmEnterCompanyTDao.selectOneByExample(example);
    }

    /**
     * set credential
     *
     * @param credential credential
     */
    public void setCredential(String credential) {
        this.credential = credential;
    }

    /**
     * get credential
     *
     * @return the string
     */
    public String getCredential() {
        return credential;
    }

    /**
     * set ump namesrv urls
     *
     * @param umpNamesrvUrls ump namesrv urls
     */
    public void setUmpNamesrvUrls(String umpNamesrvUrls) {
        this.umpNamesrvUrls = umpNamesrvUrls;
    }

    /**
     * get ump namesrv urls
     *
     * @return the string
     */
    public String getUmpNamesrvUrls() {
        return umpNamesrvUrls;
    }

    /**
     * set app id
     *
     * @param appId app id
     */
    public void setAppId(String appId) {
        this.appId = appId;
    }

    /**
     * get app id
     *
     * @return the string
     */
    public String getAppId() {
        return appId;
    }

    /**
     * set topic
     *
     * @param topic topic
     */
    public void setTopic(String topic) {
        this.topic = topic;
    }

    /**
     * get topic
     *
     * @return the string
     */
    public String getTopic() {
        return topic;
    }


}



```
