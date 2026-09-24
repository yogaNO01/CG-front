# 页面数据字段清单

本文按当前已实现的首页、商品列表页、商品详情页、企业店铺页、企业档案页及“留言问价”弹窗汇总。`src/mock/home.js` 与 `src/App.vue` 中的数据目前均为前端 mock 或模板常量；下面将页面已经展示的字段和建议接口必须提供的字段分开，便于后续建表与接口拆分。

## 一、公司（供应商 / 店铺）

### 1. 页面已展示的公司字段

| 字段名 | 建议字段 key | 类型 | 来源 / 页面用途 |
| --- | --- | --- | --- |
| 企业名称 | `companyName` | string | 店铺标题、企业档案、商品右侧商家卡片 |
| 企业简称 | `shortName` | string | 店铺头部、商家卡片 |
| 所属行业 / 主营类目 | `industryCategoryId` | string / ID | 企业标签、企业简介 |
| 店铺标识文字 | `logoText` | string | 当前为 `mark`，用于无 Logo 时的头像占位 |
| 店铺视觉主题 | `themeColor` | enum | 当前为 `tone`，仅前端展示可选 |
| 店铺 Logo | `logoUrl` | URL | 当前未有真实字段，页面应替代文字占位 |
| 企业简介 | `introduction` | text | 企业档案与店铺首页简介 |
| 主营商品 | `mainProducts` | relation / array | 当前由商品名称拼接；建议改为商品关联或关键词 |
| 经营年限 | `operatingYears` | integer | 店铺头部、商家卡片 |
| 响应率 | `responseRate` | decimal | 店铺首页、商家卡片 |
| 在售商品数 | `onSaleProductCount` | integer | 店铺首页；可由商品表统计，不建议手工维护 |
| 发货地 | `shippingLocation` | region ID + text | 店铺头部、商品详情 |
| 认证供应商状态 | `isPlatformVerified` | boolean | 多处“认证供应商”标识 |
| 验厂状态 | `isFactoryAudited` | boolean | 多处“已验厂企业”标识 |
| 供货能力标签 | `supplyCapabilities` | string[] | 当前“源头供货、支持定制” |
| 注册资本 | `registeredCapital` | decimal + currency | 工商信息 |
| 注册地址 | `registeredAddress` | string | 工商信息 |
| 企业网址 / 店铺链接 | `websiteUrl` / `shopUrl` | URL | 工商信息 |
| 营业执照核验状态 | `businessLicenseVerified` | enum | 工商信息 |
| 统一社会信用代码 | `unifiedSocialCreditCode` | string | 工商信息，前台宜脱敏展示 |
| 组织机构代码 | `organizationCode` | string | 工商信息 |
| 纳税人识别号 | `taxpayerId` | string | 工商信息 |
| 工商注册号 | `registrationNumber` | string | 工商信息 |
| 法定代表人 | `legalRepresentative` | string | 工商信息 |
| 经营状态 | `businessStatus` | enum | 工商信息，如存续、注销 |
| 成立日期 | `establishedAt` | date | 工商信息 |
| 营业期限 | `businessTerm` | string / date range | 工商信息 |
| 审核 / 年检日期 | `annualInspectionAt` | date | 工商信息 |
| 企业类型 | `companyType` | enum / string | 工商信息 |
| 经营范围 | `businessScope` | text | 工商信息 |
| 登记机关 | `registrationAuthority` | string | 工商信息 |

### 2. 公司建档应补齐的必要字段

以下字段尚未在页面显式展示，但缺少它们无法稳定地管理企业、联系采购方或控制前台展示：

| 字段名 | 建议字段 key | 类型 | 说明 |
| --- | --- | --- | --- |
| 公司 ID | `companyId` | UUID / bigint | 所有商品、询价、资质的关联主键 |
| 店铺 ID / 店铺状态 | `shopId`, `shopStatus` | ID, enum | 区分企业主体和平台店铺；支持上架、冻结、审核中 |
| 联系人 | `contactName` | string | 询价分配与后台联系 |
| 联系电话 | `contactPhone` | string | 建议加密存储，前台按权限展示 |
| 联系邮箱 | `contactEmail` | string | 运营与采购通知 |
| 所在地区 | `provinceCode`, `cityCode`, `districtCode` | region IDs | 用于地区筛选；不能只保存“江苏苏州”文本 |
| 详细发货地址 | `shippingAddress` | string | 计算发货和展示地址 |
| 营业执照文件 | `businessLicenseUrl` | URL | 认证审核凭证 |
| 认证状态与时间 | `verificationStatus`, `verifiedAt` | enum, datetime | 审核流与前台认证标识来源 |
| 审核备注 | `verificationRemark` | text | 仅后台使用 |
| 创建 / 更新 / 发布状态 | `createdAt`, `updatedAt`, `publishedAt`, `status` | datetime, enum | 数据审计与上下架 |

## 二、商品

### 1. 页面已展示的商品字段

| 字段名 | 建议字段 key | 类型 | 来源 / 页面用途 |
| --- | --- | --- | --- |
| 商品名称 | `productName` | string | 首页、列表、详情、企业主营商品 |
| 商品主图 | `mainImageUrl` | URL | 首页、列表、详情首图 |
| 商品图集 | `imageUrls` | URL[] | 详情页缩略图；当前除主图外为固定图片 |
| 参考价格 | `referencePrice` | decimal | 当前 `price` 为带货币与“起”的文本，建议拆分存储 |
| 价格单位 | `priceUnit` | enum / string | 如 元/件、元/W |
| 起售标识 | `priceFrom` | boolean | 当前价格中的“起” |
| 最小起订量 | `minOrderQuantity` | decimal | 详情页“件起订”，当前数量未落库 |
| 计量单位 | `unit` | string | 如 件、台、套 |
| 商品卖点 / 短描述 | `sellingPoints` | string[] / text | 当前 `tag`，列表和详情副标题 |
| 营销角标 | `marketingBadge` | string | 当前 `badge`，如源头直供、热销优品 |
| 展示场景 | `displayScene` | enum | 当前 `scene`，首页卡片样式用途 |
| 展示图标 | `displayIcon` | string | 当前 `icon`，首页卡片样式用途 |
| 商品类目 | `categoryId` | ID | 列表、导航、企业所属领域；当前商品未实际关联类目 |
| 产品类型 | `productType` | string | 详情参数，当前固定为“工业自动化设备” |
| 供货方式 | `supplyMethod` | enum[] | 详情参数，如现货、定制 |
| 适用场景 | `applicationScenarios` | string[] | 详情参数 |
| 服务保障 | `serviceGuarantees` | string[] | 平台认证、支持定制、快速发货等 |
| 质量 / 售后服务 | `afterSalesService` | text | 详情参数 |
| 交付周期 | `deliveryLeadTime` | string / integer | 详情参数 |
| 发货地 | `shippingLocation` | region ID + text | 详情采购区；可默认继承公司，但商品可覆盖 |
| 发货时效 | `shipWithinHours` | integer | 当前为固定 48 小时 |
| 商品详情 | `description` | rich text / JSON | 当前固定商品说明 |
| 商品特性 / 选型支持 | `featureTags` | string[] | 当前“提供选型支持、专属采购对接”等 |
| 商品规格选项 | `specifications` | JSON / SKU relation | 当前固定为“标准配置、工程加强款、按图定制” |
| 供应商 ID | `companyId` | foreign key | 商品详情商家信息、同店推荐、企业在售商品 |

### 2. 商品建档及交易应补齐的必要字段

| 字段名 | 建议字段 key | 类型 | 说明 |
| --- | --- | --- | --- |
| 商品 ID / SPU 编码 | `productId`, `spuCode` | UUID / string | 路由与商品管理不能再以商品名称作为唯一标识 |
| SKU 编码与规格组合 | `skuId`, `skuCode`, `specValueIds` | ID / string / array | 不同规格、价格和库存的最小销售单元 |
| 品牌 / 型号 | `brand`, `model` | string | 首页搜索占位和询价需求已明确涉及 |
| 生产厂家 / 产地 | `manufacturer`, `originPlace` | string / region ID | B2B 商品的基础追溯信息 |
| 库存及库存状态 | `stockQuantity`, `stockStatus` | integer, enum | 现货速发、交期判断 |
| SKU 价格 / 阶梯价 | `skuPrice`, `tierPrices` | decimal / JSON | 批量报价场景的基础数据 |
| 是否支持询价 / 定制 | `inquiryEnabled`, `customizationEnabled` | boolean | 控制“立即询价”和“按图定制” |
| 是否上架 / 审核状态 | `saleStatus`, `auditStatus` | enum | 前台列表筛选和发布流 |
| 重量、尺寸、包装 | `weight`, `dimensions`, `packageInfo` | decimal / JSON / text | 物流与工程选型 |
| 技术参数 | `technicalParameters` | JSON | 不同行业商品需要动态参数模板，不能只用固定六项 |
| 合规 / 检测资料 | `certificates`, `attachments` | URL[] / JSON | 质量保障、资质展示 |
| 搜索关键词 | `searchKeywords` | string[] | 商品名称、品牌、型号搜索 |
| 排序、销量、浏览量 | `sortOrder`, `salesCount`, `viewCount` | integer | 综合排序、人气排序的真实数据来源 |
| 创建 / 更新 / 发布时间 | `createdAt`, `updatedAt`, `publishedAt` | datetime | 审计、排序、运营 |

## 三、类目与询价：页面已依赖的关联数据

### 类目

类目导航目前是三级结构：一级类目（如“机械设备”）→ 二级分组（如“手动工具”）→ 三级类目（如“套筒”）。建议独立类目表字段：`categoryId`、`parentId`、`categoryName`、`level`、`sortOrder`、`icon`、`status`。商品至少关联末级 `categoryId`，公司关联主营一级/多级类目。

### 询价单（留言问价）

弹窗已经收集或隐含以下字段：`inquiryId`、`companyId`、`productId`（企业咨询时可为空）、`productNameSnapshot`、`requirement`（最多 300 字）、`contactName`、`contactPhone`、`quantity`、`expectedSpec`、`expectedDeliveryAt`、`status`、`createdAt`、`handledAt`、`handlerId`。其中页面当前真正输入的只有采购需求、联系人、联系电话；数量、型号规格、交付时间仅写在提示文案中，建议改为独立可填字段。

## 四、当前硬编码位置与替换优先级

| 优先级 | 位置 | 当前问题 | 应替换为 |
| --- | --- | --- | --- |
| P0 | `src/App.vue` 的企业档案与商品详情 | 注册资本、信用代码、法人、产品参数、服务、发货时效等直接写死 | 公司详情接口、商品详情接口 |
| P0 | `src/App.vue` 路由 | 用企业名、商品名定位详情，重名会跳错数据 | `companyId`、`productId` |
| P1 | `src/mock/home.js` 的 `supplierShowcases` | 公司与商品嵌套，无法复用、分页或独立管理 | 公司表、商品表，以 `companyId` 关联 |
| P1 | `src/mock/home.js` 的 `products` / `catalogProducts` | 价格为展示文本，商品类别和 SKU 缺失 | 商品列表接口 + SKU / 价格结构 |
| P2 | `src/App.vue` 的询价弹窗 | 表单数据未持久化 | 询价创建接口与询价单表 |

> 结论：若先满足现有页面，核心需要 3 个实体（公司、商品、类目）和 1 个业务实体（询价单）。商品、企业详情中的静态展示字段应优先 API 化；价格、规格、库存应按 SKU 设计，避免后期改表。
