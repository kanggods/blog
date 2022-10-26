---
title: SpringBoot集成Xxl-job做定时任务
date: 2022-10-26 18:00:05
tags: [SpringBoot,定时任务]
categories: SpringBoot集成框架
---
### 1、XXL-JOB实现定时任务条件

项目地址github.com/xuxueli/xxl-job下载下来

http://42.194.208.171:9000/images/XXLjob.png

![](http://42.194.208.171:9000/images/XXLjob.png)

要实现定时任务需要两个组件：

1：任务调度器xxl-job-admin

2:   任务执行器

### 2、前置条件运行任务调度器xxl-job-admin

#### 1、导入sql文件

https://github.com/xuxueli/xxl-job/tree/master/doc/db

###### mysql生成表：

在db目录下下载sql文件。直接导入到数据库

###### postgeSQL生成表：

http://42.194.208.171:9000/images/xxl_job_Postge.sql

直接把mysql的表复制到pgsql上，但是主键需要加上序列，有些属性要给上默认值。

当懒狗直接运行我改好的，直接下载运行sql文件

#### 2、修改xxl-job-admin配置文件

主要运行端口，邮箱，数据库配置等等，右手就行

```properties
### web
server.port=8080
server.servlet.context-path=/xxl-job-admin

### actuator
management.server.servlet.context-path=/actuator
management.health.mail.enabled=false

### resources
spring.mvc.servlet.load-on-startup=0
spring.mvc.static-path-pattern=/static/**
spring.resources.static-locations=classpath:/static/

### freemarker
spring.freemarker.templateLoaderPath=classpath:/templates/
spring.freemarker.suffix=.ftl
spring.freemarker.charset=UTF-8
spring.freemarker.request-context-attribute=request
spring.freemarker.settings.number_format=0.##########

### mybatis
mybatis.mapper-locations=classpath:/mybatis-mapper/*Mapper.xml
#mybatis.type-aliases-package=com.xxl.job.admin.core.model

### xxl-job, datasource
spring.datasource.url=jdbc:postgresql://127.0.0.1:5432/xxl-job?&schema=device&useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true&stringtype=unspecified&serverTimezone=GMT%2B8
spring.datasource.username=postgres
spring.datasource.password=postgres123
spring.datasource.driver-class-name=org.postgresql.Driver

### datasource-pool
spring.datasource.type=com.zaxxer.hikari.HikariDataSource
spring.datasource.hikari.minimum-idle=10
spring.datasource.hikari.maximum-pool-size=30
spring.datasource.hikari.auto-commit=true
spring.datasource.hikari.idle-timeout=30000
spring.datasource.hikari.pool-name=HikariCP
spring.datasource.hikari.max-lifetime=900000
spring.datasource.hikari.connection-timeout=10000
spring.datasource.hikari.connection-test-query=SELECT 1
spring.datasource.hikari.validation-timeout=1000

### xxl-job, email
spring.mail.host=smtp.qq.com
spring.mail.port=25
spring.mail.username=xxx@qq.com
spring.mail.from=xxx@qq.com
spring.mail.password=xxx
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
spring.mail.properties.mail.smtp.socketFactory.class=javax.net.ssl.SSLSocketFactory

### xxl-job, access token
xxl.job.accessToken=default_token

### xxl-job, i18n (default is zh_CN, and you can choose "zh_CN", "zh_TC" and "en")
xxl.job.i18n=zh_CN

## xxl-job, triggerpool max size
xxl.job.triggerpool.fast.max=200
xxl.job.triggerpool.slow.max=100

### xxl-job, log retention days
xxl.job.logretentiondays=30

xxl.job.executor.logpath=/data/applogs/xxl-job/jobhandler
### xxl-job executor log-retention-days
xxl.job.executor.logretentiondays=30

```

#### 3、运行xxl-job-admin

搞完了直接运行，如果PGSQL运行时xml文件报错请用我的。

http://42.194.208.171:9000/images/mybatis-mapper/mybatis-mapper.zip

运行直接网页localhost:8080/xxl-job-admin访问输入账号密码：

admin  123456

任务管理：你所有的定时任务。

调度日志：定时任务运行情况

执行器管理：相当于你要执行这个任务的服务，后面服务要进行配置，与Appname一样。

用户管理：懂得都懂

#### 4、因为xxl-job有权限用http访问不到所以新建几个接口开放权限

@PermissionLimit(limit = false)跳过权限验证。增删查改的逻辑按需求修改。

任务详情JobInfoController加接口

```Java
/**
 * 新建方法
 */
/*------------------自定义方法----------------------  */
@RequestMapping("/addJob")
@ResponseBody
@PermissionLimit(limit = false)
public ReturnT<String> addJobInfo(@RequestBody XxlJobInfo jobInfo) {
   xxlJobService.add(jobInfo);
   int id = jobInfo.getId();
   return new ReturnT(id);
}

@RequestMapping("/updateJob")
@ResponseBody
@PermissionLimit(limit = false)
public ReturnT<String> updateJobCron(@RequestBody XxlJobInfo jobInfo) {
   return xxlJobService.update(jobInfo);
}

@RequestMapping("/removeJob")
@ResponseBody
@PermissionLimit(limit = false)
public ReturnT<String> removeJob(@RequestBody XxlJobInfo jobInfo) {
   return xxlJobService.remove(jobInfo.getId());
}

@RequestMapping("/pauseJob")
@ResponseBody
@PermissionLimit(limit = false)
public ReturnT<String> pauseJob(@RequestBody XxlJobInfo jobInfo) {
   return xxlJobService.stop(jobInfo.getId());
}

@RequestMapping("/startJob")
@ResponseBody
@PermissionLimit(limit = false)
public ReturnT<String> startJob(@RequestBody XxlJobInfo jobInfo) {
   return xxlJobService.start(jobInfo.getId());
}

@RequestMapping("/addAndStart")
@ResponseBody
@PermissionLimit(limit = false)
public ReturnT<String> addAndStart(@RequestBody XxlJobInfo jobInfo) {
   ReturnT<String> result = xxlJobService.add(jobInfo);
   int id = Integer.valueOf(result.getContent());
   xxlJobService.start(id);
   return result;
}
```

工作组JobGroup加接口

```Java
	@RequestMapping("/getGroupId")
	@ResponseBody
	@PermissionLimit(limit = false)
	public ReturnT<String> getGroupId(@RequestBody XxlJobGroup jobGroup) {
		XxlJobGroup group = xxlJobGroupDao.findByName(jobGroup.getAppname());
		return new ReturnT<String>(String.valueOf(group.getId()));
	}
```



### 3、编写自己的任务执行器

#### 1、自己的项目导入xxl核心依赖

```xml
<dependency>
    <groupId>com.xuxueli</groupId>
    <artifactId>xxl-job-core</artifactId>
    <version>2.3.1</version>
</dependency>
```

#### 2、编写配置类

```java
package com.suntang.ssc.ipms.integral.config;

import com.xxl.job.core.executor.impl.XxlJobSpringExecutor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * xxl-job config
 *
 * @author xuxueli 2017-04-28
 */
@Configuration
public class XxlJobConfig {
    private Logger logger = LoggerFactory.getLogger(XxlJobConfig.class);

    @Value("${xxl.job.admin.addresses}")
    private String adminAddresses;

    @Value("${xxl.job.accessToken}")
    private String accessToken;

    @Value("${xxl.job.executor.appname}")
    private String appname;

    @Value("${xxl.job.executor.address}")
    private String address;

    @Value("${xxl.job.executor.ip}")
    private String ip;

    @Value("${xxl.job.executor.integral.port}")
    private int port;

    @Value("${xxl.job.executor.logpath}")
    private String logPath;

    @Value("${xxl.job.executor.logretentiondays}")
    private int logRetentionDays;


    @Bean
    public XxlJobSpringExecutor xxlJobExecutor() {
        logger.info(">>>>>>>>>>> xxl-job config init.");
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
        xxlJobSpringExecutor.setAppname(appname);
        xxlJobSpringExecutor.setAddress(address);
        xxlJobSpringExecutor.setIp(ip);
        xxlJobSpringExecutor.setPort(port);
        xxlJobSpringExecutor.setAccessToken(accessToken);
        xxlJobSpringExecutor.setLogPath(logPath);
        xxlJobSpringExecutor.setLogRetentionDays(logRetentionDays);

        return xxlJobSpringExecutor;
    }

    /**
     * 针对多网卡、容器内部署等情况，可借助 "spring-cloud-commons" 提供的 "InetUtils" 组件灵活定制注册IP；
     *
     *      1、引入依赖：
     *          <dependency>
     *             <groupId>org.springframework.cloud</groupId>
     *             <artifactId>spring-cloud-commons</artifactId>
     *             <version>${version}</version>
     *         </dependency>
     *
     *      2、配置文件，或者容器启动变量
     *          spring.cloud.inetutils.preferred-networks: 'xxx.xxx.xxx.'
     *
     *      3、获取IP
     *          String ip_ = inetUtils.findFirstNonLoopbackHostInfo().getIpAddress();
     */
    }
```

```properties
xxl.job.admin.addresses=http://192.168.8.62:8080/xxl-job-admin
xxl.job.executor.appname=xxl-job-executor-ipms
xxl.job.executor.ip=

xxl.job.executor.notice.port=9999
xxl.job.executor.integral.port=9998
xxl.job.executor.address=
xxl.job.executor.logpath=/data/applogs/xxl-job/jobhandler
### xxl-job executor log-retention-days
xxl.job.executor.logretentiondays=30
xxl.job.accessToken=default_token
```

配置文件，需要的自己改

#### 3、添加调用xxl-job-Admin的工具类

对任务进行增删改查和页面上请求一样

任务详情类

```java
package com.suntang.common.job;

import java.util.Date;

/**
 * xxl-job info
 *
 * @author xuxueli  2016-1-12 18:25:49
 */
public class XxlJobInfo {

   private int id;             // 主键ID

   private int jobGroup;     // 执行器主键ID
   private String jobDesc;
   private Date addTime;
   private Date updateTime;
   private String author;    // 负责人
   private String alarmEmail; // 报警邮件
   private String scheduleType;         // 调度类型
   private String scheduleConf;         // 调度配置，值含义取决于调度类型
   private String misfireStrategy;          // 调度过期策略
   private String executorRouteStrategy;  // 执行器路由策略
   private String executorHandler;           // 执行器，任务Handler名称
   private String executorParam;         // 执行器，任务参数
   private String executorBlockStrategy;  // 阻塞处理策略
   private int executorTimeout;          // 任务执行超时时间，单位秒
   private int executorFailRetryCount;       // 失败重试次数
   private String glueType;      // GLUE类型  #com.xxl.job.core.glue.GlueTypeEnum
   private String glueSource;    // GLUE源代码
   private String glueRemark;    // GLUE备注
   private Date glueUpdatetime;   // GLUE更新时间
   private String childJobId;    // 子任务ID，多个逗号分隔
   private int triggerStatus;    // 调度状态：0-停止，1-运行
   private long triggerLastTime;  // 上次调度时间
   private long triggerNextTime;  // 下次调度时间


   public int getId() {
      return id;
   }

   public void setId(int id) {
      this.id = id;
   }

   public int getJobGroup() {
      return jobGroup;
   }

   public void setJobGroup(int jobGroup) {
      this.jobGroup = jobGroup;
   }

   public String getJobDesc() {
      return jobDesc;
   }

   public void setJobDesc(String jobDesc) {
      this.jobDesc = jobDesc;
   }

   public Date getAddTime() {
      return addTime;
   }

   public void setAddTime(Date addTime) {
      this.addTime = addTime;
   }

   public Date getUpdateTime() {
      return updateTime;
   }

   public void setUpdateTime(Date updateTime) {
      this.updateTime = updateTime;
   }

   public String getAuthor() {
      return author;
   }

   public void setAuthor(String author) {
      this.author = author;
   }

   public String getAlarmEmail() {
      return alarmEmail;
   }

   public void setAlarmEmail(String alarmEmail) {
      this.alarmEmail = alarmEmail;
   }

   public String getScheduleType() {
      return scheduleType;
   }

   public void setScheduleType(String scheduleType) {
      this.scheduleType = scheduleType;
   }

   public String getScheduleConf() {
      return scheduleConf;
   }

   public void setScheduleConf(String scheduleConf) {
      this.scheduleConf = scheduleConf;
   }

   public String getMisfireStrategy() {
      return misfireStrategy;
   }

   public void setMisfireStrategy(String misfireStrategy) {
      this.misfireStrategy = misfireStrategy;
   }

   public String getExecutorRouteStrategy() {
      return executorRouteStrategy;
   }

   public void setExecutorRouteStrategy(String executorRouteStrategy) {
      this.executorRouteStrategy = executorRouteStrategy;
   }

   public String getExecutorHandler() {
      return executorHandler;
   }

   public void setExecutorHandler(String executorHandler) {
      this.executorHandler = executorHandler;
   }

   public String getExecutorParam() {
      return executorParam;
   }

   public void setExecutorParam(String executorParam) {
      this.executorParam = executorParam;
   }

   public String getExecutorBlockStrategy() {
      return executorBlockStrategy;
   }

   public void setExecutorBlockStrategy(String executorBlockStrategy) {
      this.executorBlockStrategy = executorBlockStrategy;
   }

   public int getExecutorTimeout() {
      return executorTimeout;
   }

   public void setExecutorTimeout(int executorTimeout) {
      this.executorTimeout = executorTimeout;
   }

   public int getExecutorFailRetryCount() {
      return executorFailRetryCount;
   }

   public void setExecutorFailRetryCount(int executorFailRetryCount) {
      this.executorFailRetryCount = executorFailRetryCount;
   }

   public String getGlueType() {
      return glueType;
   }

   public void setGlueType(String glueType) {
      this.glueType = glueType;
   }

   public String getGlueSource() {
      return glueSource;
   }

   public void setGlueSource(String glueSource) {
      this.glueSource = glueSource;
   }

   public String getGlueRemark() {
      return glueRemark;
   }

   public void setGlueRemark(String glueRemark) {
      this.glueRemark = glueRemark;
   }

   public Date getGlueUpdatetime() {
      return glueUpdatetime;
   }

   public void setGlueUpdatetime(Date glueUpdatetime) {
      this.glueUpdatetime = glueUpdatetime;
   }

   public String getChildJobId() {
      return childJobId;
   }

   public void setChildJobId(String childJobId) {
      this.childJobId = childJobId;
   }

   public int getTriggerStatus() {
      return triggerStatus;
   }

   public void setTriggerStatus(int triggerStatus) {
      this.triggerStatus = triggerStatus;
   }

   public long getTriggerLastTime() {
      return triggerLastTime;
   }

   public void setTriggerLastTime(long triggerLastTime) {
      this.triggerLastTime = triggerLastTime;
   }

   public long getTriggerNextTime() {
      return triggerNextTime;
   }

   public void setTriggerNextTime(long triggerNextTime) {
      this.triggerNextTime = triggerNextTime;
   }
}
```

```java
package com.suntang.ssc.ipms.integral.util;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.suntang.common.job.XxlJobInfo;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

import java.util.HashMap;
import java.util.Map;

@Slf4j
@Component
public class XxlJobUtil {

    @Value("${xxl.job.admin.addresses}")
    private String adminAddresses;

    @Value("${xxl.job.executor.appname}")
    private String appname;

    private RestTemplate restTemplate = new RestTemplate();
    private static final String ADD_URL = "/jobinfo/addJob";
    private static final String UPDATE_URL = "/jobinfo/updateJob";
    private static final String REMOVE_URL = "/jobinfo/removeJob";
    private static final String PAUSE_URL = "/jobinfo/pauseJob";
    private static final String START_URL = "/jobinfo/startJob";
    private static final String ADD_START_URL = "/jobinfo/addAndStart";
    private static final String GET_GROUP_ID = "/jobgroup/getGroupId";


    public String add(XxlJobInfo jobInfo){
        // 查询对应groupId:
        Map<String,Object> param = new HashMap<>();
        param.put("appname", appname);
        String json = JSON.toJSONString(param);
        String result = doPost(adminAddresses + GET_GROUP_ID, json);
        JSONObject jsonObject = JSON.parseObject(result);
        String groupId = jsonObject.getString("content");
        jobInfo.setJobGroup(Integer.parseInt(groupId));
        String json2 = JSON.toJSONString(jobInfo);
        return doPost(adminAddresses + ADD_URL, json2);
    }

    public String update(int id, String cron){
        Map<String,Object> param = new HashMap<>();
        param.put("id", id);
        param.put("jobCron", cron);
        String json = JSON.toJSONString(param);
        return doPost(adminAddresses + UPDATE_URL, json);
    }

    public String remove(int id){
        Map<String,Object> param = new HashMap<>();
        param.put("id", id);
        String json = JSON.toJSONString(param);
        return doPost(adminAddresses + REMOVE_URL, json);
    }

    public String pause(int id){
        Map<String,Object> param = new HashMap<>();
        param.put("id", id);
        String json = JSON.toJSONString(param);
        return doPost(adminAddresses + PAUSE_URL, json);
    }

    public String start(int id){
        Map<String,Object> param = new HashMap<>();
        param.put("id", id);
        String json = JSON.toJSONString(param);
        return doPost(adminAddresses + START_URL, json);
    }

    public String addAndStart(XxlJobInfo jobInfo){
        Map<String,Object> param = new HashMap<>();
        param.put("appname", appname);
        String json = JSON.toJSONString(param);
        String result = doPost(adminAddresses + GET_GROUP_ID, json);

        JSONObject jsonObject = JSON.parseObject(result);
        String groupId = jsonObject.getString("content");
        jobInfo.setJobGroup(Integer.parseInt(groupId));
        String json2 = JSON.toJSONString(jobInfo);

        return doPost(adminAddresses + ADD_START_URL, json2);
    }

    public String doPost(String url, String json){
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity<String> entity = new HttpEntity<>(json ,headers);
        log.info(entity.toString());
        ResponseEntity<String> stringResponseEntity = restTemplate.postForEntity(url, entity, String.class);
        return stringResponseEntity.getBody().toString();
    }

}
```

#### 4、使用

写一个执行器clockInJobHandler

```java
@Slf4j
@Component
public class HandleUserTaskJob {

    @Autowired
    private UserTaskService userTaskService;

    @XxlJob("clockInJobHandler")
    public void userTaskJobHandle(){
       // 定时任务业务逻辑
    }
}
```



修改XxlJobInfo的数据实现定时任务,添加任务后，在admin网页上可以看到新建的任务信息。

```Java
@Test
public void test(){
    XxlJobInfo xxlJobInfo = new XxlJobInfo();
    xxlJobInfo.setScheduleConf("0/5 * * * * ?");
    xxlJobInfo.setJobGroup(1);
    xxlJobInfo.setJobDesc("我来试试");
    xxlJobInfo.setAddTime(new Date());
    xxlJobInfo.setUpdateTime(new Date());
    xxlJobInfo.setAuthor("JCccc");
    xxlJobInfo.setAlarmEmail("864477182@com");
    xxlJobInfo.setScheduleType("CRON");
    xxlJobInfo.setScheduleConf("0/5 * * * * ?");
    xxlJobInfo.setMisfireStrategy("DO_NOTHING");
    xxlJobInfo.setExecutorRouteStrategy("FIRST");
    xxlJobInfo.setExecutorHandler("clockInJobHandler");
    xxlJobInfo.setExecutorParam("att");
    xxlJobInfo.setExecutorBlockStrategy("SERIAL_EXECUTION");
    xxlJobInfo.setExecutorTimeout(0);
    xxlJobInfo.setExecutorFailRetryCount(1);
    xxlJobInfo.setGlueType("BEAN");
    xxlJobInfo.setGlueSource("");
    xxlJobInfo.setGlueRemark("GLUE代码初始化");
    xxlJobInfo.setGlueUpdatetime(new Date());
    xxlJobUtil.add(xxlJobInfo);
}
```

