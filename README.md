财务服务项目
===

## 简介

这是财务服务项目 finance-service

业务说明中的图可通过支持mermaid扩展的md工具观看（如有道云笔记）

---

## 目录

[TOC]

---

## 正文

### todo 上游数据异常，数据修复后，财务数据需要同步修复的点

订单：

1、全流程标签

2、客户对账(包含发票申请和授信对账)

3、合伙人对账


供应商订单：

todo

逆向单：

todo


### 版本说明

最新finance-service-api版本号,请看

/pom.xml

### controller维护入口说明

维护业务|入口|负责人
---|---|---
操作日志-技术维护|OperationLogController|刘骏泳
发票申请和发票开票-技术维护|InvoiceController|刘骏泳
授信对账-技术维护|CreditBillController|刘骏泳
供应商优惠规则-技术维护|SupplierRuleController|刘骏泳
供应商对账-技术维护|SupplierBillController|刘骏泳
合伙人对账-技术维护|PartnerBillController|刘骏泳

### 定时任务

定时任务|入口|负责人
---|---|---
授信对账账单定时生成任务|CreditBillAutoBuildJob|刘骏泳
合伙人对账账单定时生成任务|PartnerBillAutoBuildJob|刘骏泳
发票申请账单定时生成任务|InvoiceAutoApplyJob|刘骏泳
发票开票流水定时生成任务|InvoiceAutoBuildJob|刘骏泳
退供超时确认的任务|ReturnSupplierTimeoutConfirmJob|刘骏泳
承运商对账的月结和周结账单的“生成”任务|CarrierMonthlyAndWeeklyBillGenerateJob|
物流出入库记录信息补全任务|SaleStorageJob|
供应商对账-采购入库明细生成任务|SupplierBillForwardDetailBuildJob|刘骏泳

### api服务入口说明

业务|包|负责人
---|---|---
供应商优惠规则|com.baturu.finance.api.service.supplierRule|刘骏泳
供应商对账 合同|com.baturu.finance.api.service.supplierSettlement|刘骏泳
供应商对账 账单(+采购明细+退供明细)|com.baturu.finance.api.service.supplier.bill|刘骏泳
退货相关金额服务(暂时只有退货免折日志)|com.baturu.finance.api.service.returns.goods|刘骏泳
退款操作接口(未完成，仅对接退款不退货，作为paybuss的中继)|com.baturu.finance.api.service.refund|刘骏泳
发票申请和发票开票|com.baturu.finance.api.service.invoice|刘骏泳
授信对账|com.baturu.finance.api.service.credit|刘骏泳
合伙人对账|com.baturu.finance.api.service.partner|刘骏泳
物流出入库对接(有待调整，应把生成逻辑从物流服务中迁移过来)|com.baturu.finance.api.service.sale|刘骏泳
承运商对账|com.baturu.finance.api.service.carrierBill|
金蝶数据对接(对接暂停中)|com.baturu.finance.api.service.kingdee|
服务商对账(该业务已停用)|com.baturu.finance.api.service.agentBill|

### 服务管理的数据库表说明

1. 财务专属数据源：finance

2. 在main库的财务相关表(todo 迁移到finance库)

    (1) fin开头的表

|表名|说明|
|---|---|
|fin_XXX|fin开头的表(大部分和供应商对账有关)|
|qp_invoice_apply|发票相关表|
|qp_invoice_apply_order|发票相关表|
|qp_invoice_apply_orderdetail|发票相关表|
|qp_invoice_apply_record|发票相关表|
|qp_invoice_apply_record_copy|发票相关表|
|qp_invoice_apply_record_relation|发票相关表|
|qp_invoice_apply_recorddetail|发票相关表|
|qp_invoice_tax|发票相关表|

### 业务汇总说明

```
graph LR
subgraph 业务线
a0[财务业务] --> a1[客户对账线]
a0 --> a2[供应商对账线]
a0 --> a3[承运商对账线]
a0 --> a4[合伙人对账线]
a0 --> a5>投资人对账线]
a0 --> a6>其他]
end
```

```
graph LR
subgraph 功能1
a1[客户对账线] --> b11[交易统计报表]
a1 --> b12[全流程标签清单-交易部分]
a1 --> b13[退款审核-包括取消订单,退货,退款不退货]
a1 --> b14[普通对账单]
a1 --> b15[授信对账单]
a1 --> b16[*授信合同管理]
a1 --> b17[*开票管理]
a1 --> b18[*报税管理]
a1 --> b19[*其他自营支付渠道管理-汽配宝-授信宝-线下汇款]
end
```

```
graph LR
subgraph 功能2
a2[供应商对账线] --> b21[采购入库清单]
a2 --> b22[退供出库清单]
a2 --> b23[其他应收应付管理]
a2 --> b24[供应商对账单]
a2 --> b25[*供应商合同与佣金设置管理]
a2 --> b26[*供应商优惠规则管理]
end

```

```
graph LR
subgraph 功能3
a3[承运商对账线] --> b31[承运商日清单]
a3 --> b32[承运商对账单]

a4[合伙人对账线] --> b41[合伙人对账单]
a4 --> b42[全流程标签清单-合伙人部分]

a5>投资人对账线] --> b51>供应链金融]

a6>其他] --> b61>上传到金蝶系统]
a6 --> b62[销售出入库清单]
end
```

### 客户对账

```
graph TB
A1(每月订单) --数据--> B{判断是否在授信合同账期内}
A2(每月取消订单) --数据--> B
A3(每月退货单) --数据--> B
A4(每月退款不退货退货单) --数据--> B
B --否--> B3{判断是否特殊开票}
B --是--> B2{判断是否保险单}
B2 --否--> D3[生成客户对账授信账单]
B2 --是--> D4[生成中配宝对账账单]
B3 --否--> D1[生成客户对账普通账单]
B3 --是--> D2[生成客户对账特殊账单]
D1 --> F((保存客户对账普通账单))
D2 --> F((保存客户对账普通账单))
D3 --> E((保存客户对账授信账单主单))
D3 --> F
subgraph 中配宝系统
D4 --> H[中配宝客户确认]
end
H --> I[生成客户对账保险授信账单]
I --> E
I --> F
E --> J[财务进行授信对账]
```

### 开票管理

```
graph TB
A5(发票资质信息) --数据--> D[生成发票记录]
A5 --数据--> E1[刷新发票资质信息]
A1(普通对账账单) --数据--> B{判断是否需要审核}
B --否--> I[定时任务]
B --是--> J[等待]
J --> K[等到通知或者调用]
A2[通知或调用] --> K
K --> D
I --> D
D --> E((保存发票记录))
E --> E1
E1 --> E
E --> F[财务人员登录航天发票系统]
subgraph 航天发票系统
F --> G[开票]
end
G --> H((回填发票记录))
H --> 结束
```

### 报税管理

```
graph TB
A1(每月生成或更新的发票记录) --数据--> B1{判断是否已开票}
A2(导入表格) --数据--> B2{是否需要手动报税}
B1 --否--> B2
B1 --是--> B3{是否之前已手动报税}
B2 --否--> C[生成未报税未开票记录]
B2 --是--> D[生成报税未开票记录]
B3 --否--> E[生成报税已开票记录]
B3 --是--> F[生成原报税未开票记录的冲红]
F --> E
C --> G((保存报税记录))
D --> G
E --> G
F --> G
```

### 供应商对账

```
graph TB
A1(销售入库报表记录) --数据--> B1
A2(供应商反馈入库) --数据--> B2
A3(退款不退货冲红) --数据--> B2
A4(物流退供出库) --数据--> B3
A5(供应商合同信息,佣金信息,账期信息) --数据--> E
A6(其他应收应付记录) --数据--> E
subgraph 主流程
B1[定时任务-生成采购入库记录] --> C1((保存采购入库明细记录))
B2[直接生成采购入库记录] --> C1
B3[直接生成退供出库记录] --> C2((保存退供出库明细记录))
C1 --> D1{流程确认是否采购成功}
C2 --> D2{流程确认是否退供成功}
D1 --是--> E[定时任务-根据合同生成供应商对账单]
D2 --是--> E
E --> F((保存供应商对账单))
F --> G[财务审核]
G --> G1[财务刷新账单]
G1 --> G
G --> H[供应商确认]
H --> 结束
end
```


### 合伙人对账

```
graph TB
A1(CRM人员维护汽修厂关系数据) --数据--> B
A2(CRM团队负责人员关系数据) --数据--> B
A3(完成支付的订单的明细) --数据--> B
subgraph 主流程
B[定时任务-生成全流程标签-相当于日清账单] --> C((保存全流程标签))
C --每月数据--> E[定时任务-生成合伙人账单主单]
E --> F((保存合伙人账单))
end

O1>后门1-启动全流程标签生成定时任务] --> B
C --> O2>后门2-重置全流程标签状态]
O2 --> O1
F --> O3>后门3-重新生成合伙人账单明细]
O3 --> F
F --> O4>后门4-逻辑删除合伙人账单]
O4 --> O5>后门5-触发生成合伙人账单的定时任务]
O5 --> E
```
