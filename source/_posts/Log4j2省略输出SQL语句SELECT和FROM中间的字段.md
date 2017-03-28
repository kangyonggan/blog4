---
title: Log4j2省略输出SQL语句SELECT和FROM中间的字段
date: 2017-03-28 14:30:48
categories: Java后台
tags:
- Java
---

在项目中，起初我们是不打印SQL语句的，后来为了查问题更方便，就打印了SQL。  
但是也带来了一些问题，比如查询的字段太多，造成满屏SQL，给查日志造成了一定的影响。  
所以我就在想，能不能把`SELECT`和`FROM`中间的字段省略输出呢？

<!-- more -->

## 线上日志中的SQL
```
DEBUG : ==>  Preparing: SELECT serial_no as serialNo,bnk_no as bnkNo ,mer_org_serial_no as merOrgSerialNo,bnk_org_serial_no as bnkOrgSerialNo, mer_date as merDate,mer_time as merTime, mer_org_date as merOrgDate,mer_org_time as merOrgTime ,bnk_org_date as bnkOrgDate,bnk_org_time as bnkOrgTime, last_try_date as lastTryDate,last_try_time as lastTryTime ,last_snd_date as lastSndDate,last_snd_time as lastSndTime, last_qry_serial_no as lastQrySerialNo,last_qry_date as lastQryDate,last_qry_time as lastQryTime, input_date as inputDate,input_time as inputTime,mer_tran_co as merTranCo,bnk_tran_co as bnkTranCo, mer_org_tran_co as merOrgTranCo,bnk_org_tran_co as bankOrgTranCo ,tran_tp as tranTy,syn_flg as synFlg,bat_flg as batFlg, acount as acount,retry_flg as retryFlg,retry_max_time as retryMaxTime ,retry_interval as retryInterval, retry_counter as retryCounter,resnd_flg as resndFlg ,resnd_max_time as resndMaxTime,resnd_interval as resndInterval,resnd_counter as resndCounter, qry_flg as qryFlg,qry_tran_co as qryTranCo,qry_max_time as qryMaxTime,qry_interval as qryInterval, qry_counter as qryCounter,priority as priority,model as model ,product_id as productId,product_tp as productTp,cur_co as currencyCode, amount as amount,fee_amt as feeAmt,tran_purpose as tranPurpose ,tran_remark as tranRemark,ref_app_no as refAppNo, ref_app_kind as refAppKind,rcvr_bnk_no as rcvrBnkNo ,rcvr_acct_no as rcvrAcctNo,rcvr_acct_nm as rcvrAcctNm,rcvr_id_tp as rcvrIdTp,rcvr_id_no as rcvrIdNo, rcvr_province as rcvrProvince,rcvr_city as rcvrCity,rcvr_area_co as rcvrAreaCo,rcvr_area_nm as rcvrAreaNm, rcvr_mer_id as rcvrMerId,rcvr_mer_cert_id as rcvrMerCertId,rcvr_post_id as rcvrPostId ,rcvr_contract_no as rcvrContractNo,rcvr_contract_dt as rcvrContractDt, rcvr_proto_no as rcvrProtoNo,rcvr_mer_user_tp as rcvrMerUserTp,rcvr_mer_user_id as rcvrMerUserId, rcvr_bnk_user_tp as rcvrBnkUserTp,rcvr_bnk_user_id as rcvrBnkUserId,rcvr_resv1 as rcvrResv1, rcvr_resv2 as rcvrResv2,sndr_bnk_no as sndrBankNo,sndr_name as sndrName, sndr_acct_no as sndrAcctNo,sndr_acct_nm as sndrAcctName, sndr_id_tp as sndrIdType,sndr_id_no as sndrIdNo,sndr_province as sndrProvince,sndr_city as sndrCity, sndr_area_co as sndrAreaCode,sndr_area_nm as sndrAreaName,sndr_mer_id as sndrMerId ,sndr_mer_cert_id as sndrMerCertId,sndr_post_id as sndrPostId, sndr_contract_no as sndrContractNo,sndr_contract_dt as sndrContractDate,sndr_proto_no as sndrProtoNo, sndr_mer_user_tp as sndrMerUserType,sndr_mer_user_id as sndrMerUserId ,sndr_bnk_user_tp as sndrBnkUserType,sndr_bnk_user_id as sndrBankUserId, sndr_resv1 as sndrResv1,sndr_resv2 as sndrResv2,lock_st as lockSt,tran_st as tranSt,business_type as businessType, rvrs_st as rvrsSt,product_nm as productName, APP_VERSION as appVersion,APP_SOURCE as appSource, RCVR_BNK_BRANCH_NAME as rcvrBnkBranchName,SNDR_BNK_BRANCH_NAME as sndrBnkBranchName, created_at as insertTimestamp, updated_at as updateTimestamp FROM be_command T WHERE T.REF_APP_NO = ?
```

一大坨SQL占满了整个屏幕，并且日志问题也会变得很大，其实这句SQL有用信息很少:

```
DEBUG : ==>  Preparing: SELECT xxx FROM be_command T WHERE T.REF_APP_NO = ?
```

所以我就想能不能像上面这样输出呢？看下面的demo

## 使用Log4j2的Replace功能
### pom.xml
```
<log4j2.api.version>2.5</log4j2.api.version>

...

<!--Log4j2-->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>${log4j2.api.version}</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>${log4j2.api.version}</version>
</dependency>
```

### log4j2.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN" monitorInterval="300">
    <properties>
        <property name="LOG_HOME">/Users/kyg/logs/book</property>
    </properties>

    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%replace{%msg}{Preparing: SELECT [\w ,]+ FROM be_command}{Preparing: SELECT xxx FROM be_command}%n"/>
        </Console>
    </Appenders>

    <Loggers>
        <Root level="debug" additivity="true">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```

### Test.java
```
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

/**
 * @author kangyonggan
 * @since 2017/3/28
 */
public class Test {

    private static Logger log = LogManager.getLogger(Test.class);

    public static void main(String[] args) {
        log.info("DEBUG : ==>  Preparing: SELECT serial_no as serialNo,bnk_no as bnkNo ,mer_org_serial_no as merOrgSerialNo,bnk_org_serial_no as bnkOrgSerialNo, mer_date as merDate,mer_time as merTime, mer_org_date as merOrgDate,mer_org_time as merOrgTime ,bnk_org_date as bnkOrgDate,bnk_org_time as bnkOrgTime, last_try_date as lastTryDate,last_try_time as lastTryTime ,last_snd_date as lastSndDate,last_snd_time as lastSndTime, last_qry_serial_no as lastQrySerialNo,last_qry_date as lastQryDate,last_qry_time as lastQryTime, input_date as inputDate,input_time as inputTime,mer_tran_co as merTranCo,bnk_tran_co as bnkTranCo, mer_org_tran_co as merOrgTranCo,bnk_org_tran_co as bankOrgTranCo ,tran_tp as tranTy,syn_flg as synFlg,bat_flg as batFlg, acount as acount,retry_flg as retryFlg,retry_max_time as retryMaxTime ,retry_interval as retryInterval, retry_counter as retryCounter,resnd_flg as resndFlg ,resnd_max_time as resndMaxTime,resnd_interval as resndInterval,resnd_counter as resndCounter, qry_flg as qryFlg,qry_tran_co as qryTranCo,qry_max_time as qryMaxTime,qry_interval as qryInterval, qry_counter as qryCounter,priority as priority,model as model ,product_id as productId,product_tp as productTp,cur_co as currencyCode, amount as amount,fee_amt as feeAmt,tran_purpose as tranPurpose ,tran_remark as tranRemark,ref_app_no as refAppNo, ref_app_kind as refAppKind,rcvr_bnk_no as rcvrBnkNo ,rcvr_acct_no as rcvrAcctNo,rcvr_acct_nm as rcvrAcctNm,rcvr_id_tp as rcvrIdTp,rcvr_id_no as rcvrIdNo, rcvr_province as rcvrProvince,rcvr_city as rcvrCity,rcvr_area_co as rcvrAreaCo,rcvr_area_nm as rcvrAreaNm, rcvr_mer_id as rcvrMerId,rcvr_mer_cert_id as rcvrMerCertId,rcvr_post_id as rcvrPostId ,rcvr_contract_no as rcvrContractNo,rcvr_contract_dt as rcvrContractDt, rcvr_proto_no as rcvrProtoNo,rcvr_mer_user_tp as rcvrMerUserTp,rcvr_mer_user_id as rcvrMerUserId, rcvr_bnk_user_tp as rcvrBnkUserTp,rcvr_bnk_user_id as rcvrBnkUserId,rcvr_resv1 as rcvrResv1, rcvr_resv2 as rcvrResv2,sndr_bnk_no as sndrBankNo,sndr_name as sndrName, sndr_acct_no as sndrAcctNo,sndr_acct_nm as sndrAcctName, sndr_id_tp as sndrIdType,sndr_id_no as sndrIdNo,sndr_province as sndrProvince,sndr_city as sndrCity, sndr_area_co as sndrAreaCode,sndr_area_nm as sndrAreaName,sndr_mer_id as sndrMerId ,sndr_mer_cert_id as sndrMerCertId,sndr_post_id as sndrPostId, sndr_contract_no as sndrContractNo,sndr_contract_dt as sndrContractDate,sndr_proto_no as sndrProtoNo, sndr_mer_user_tp as sndrMerUserType,sndr_mer_user_id as sndrMerUserId ,sndr_bnk_user_tp as sndrBnkUserType,sndr_bnk_user_id as sndrBankUserId, sndr_resv1 as sndrResv1,sndr_resv2 as sndrResv2,lock_st as lockSt,tran_st as tranSt,business_type as businessType, rvrs_st as rvrsSt,product_nm as productName, APP_VERSION as appVersion,APP_SOURCE as appSource, RCVR_BNK_BRANCH_NAME as rcvrBnkBranchName,SNDR_BNK_BRANCH_NAME as sndrBnkBranchName, created_at as insertTimestamp, updated_at as updateTimestamp FROM be_command T WHERE T.REF_APP_NO = ?");
    }

}
```

### 输出
```
DEBUG : ==>  Preparing: SELECT xxx FROM be_command T WHERE T.REF_APP_NO = ?

Process finished with exit code 0
```




