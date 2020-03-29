[TOC]

## 业务逻辑



## 数据库设计

#### database：projectKDB

| 表名                | 含义              | 备注                                                         |
| ------------------- | ----------------- | ------------------------------------------------------------ |
| activity_tbl        | 活动表            |                                                              |
| activity_detail_tbl | 活动详情表        |                                                              |
| activity_image_tbl  | 活动图片          |                                                              |
| template_tbl        | 活动模板          |                                                              |
| templateField_tbl   | 活动模板属性      |                                                              |
| tag_activity_tbl    | 活动标签表        |                                                              |
| subClasses_tbl      | 套餐表            | 套餐即用户可买的一个单位，一个活动包含多个套餐，一个套餐仅属于一个活动。<br />套餐ID在代码中有多种表示方式：subId，subclassId, packageId 指的都是一个东西 |
| sub_classes_task    | 套餐定时发布/取消 |                                                              |
| subClasses_lang_tbl | 套餐多语言信息    |                                                              |
| price_tbl           | 价格表            |                                                              |
| price_custom_tbl    | 特殊售卖价格      |                                                              |
| exchange_rate       | 货币汇率表        | 其他货币对 HKD 的汇率，每日更新一次                          |
| sku_tbl             | sku 表            | 在商品中心系统里，一个套餐底下可以有多个 sku，每个 sku 连接着一个 价格单位(priceId)。sku 起的作用为：连接 活动 ID、套餐 ID、价格 ID 三者的枢纽，一个 skuID 表示actId+packageID+priceId 的唯一表示。 |
| arrangement_tbl     | 套餐可卖/预定日期 | 预定日期是指用户可用预定当前活动、套餐哪天的票等。其包含几个重要的时间：套餐的开始时间、结束时间、截止预定时间 |
| scheduleRules_tbl   | 套餐可卖日期规则  |                                                              |
| otherInfo_tbl       | 其他信息          |                                                              |
|                     |                   |                                                              |
|                     |                   |                                                              |
|                     |                   |                                                              |
|                     |                   |                                                              |
|                     |                   |                                                              |
|                     |                   |                                                              |
|                     |                   |                                                              |

