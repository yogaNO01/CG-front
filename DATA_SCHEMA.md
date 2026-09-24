# 商城数据字段与格式规范

适用范围：当前首页、商品列表、商品详情、企业店铺、企业档案和留言问价页面。

约定：

- ID 使用 UUID 字符串；若项目统一使用雪花 ID，可将所有 `uuid` 换为 `bigint` 字符串。
- 金额不使用浮点数。接口传字符串小数，如 `"1280.00"`；数据库使用 `DECIMAL(14,2)`。
- 时间使用 ISO 8601，例：`"2026-09-24T10:30:00+08:00"`；日期使用 `YYYY-MM-DD`。
- 枚举值用英文 code，前端根据字典显示中文。
- 前台展示的脱敏信息和后台原始信息应分字段、按权限返回。

## 1. 公司 / 店铺 `company`

### 1.1 主表字段

| 字段 | 类型（数据库） | 必填 | 校验 / 枚举 | 示例 | 页面用途 |
| --- | --- | --- | --- | --- | --- |
| `companyId` | `CHAR(36)` | 是 | UUID，唯一 | `"f42f..."` | 商品、询价、资质关联主键 |
| `shopId` | `CHAR(36)` | 是 | UUID，唯一 | `"be3d..."` | 店铺主页主键 |
| `companyName` | `VARCHAR(200)` | 是 | 2–200 字 | `"华东智能装备有限公司"` | 店铺/企业标题 |
| `shortName` | `VARCHAR(80)` | 否 | ≤80 字 | `"华智装备"` | 店铺简称 |
| `logoUrl` | `VARCHAR(500)` | 否 | 图片 URL | `"https://.../logo.png"` | 企业 Logo |
| `logoText` | `VARCHAR(8)` | 否 | 1–8 字 | `"HZ"` | 无 Logo 时占位 |
| `themeColor` | `VARCHAR(20)` | 否 | `blue`/`green`/`orange`/`teal` | `"blue"` | 前端展示主题 |
| `introduction` | `TEXT` | 是 | 20–5000 字 | `"专注于工业自动化设备..."` | 企业简介 |
| `mainCategoryIds` | `JSON` | 是 | 类目 ID 数组，1–10 项 | `["cat-machinery"]` | 主营领域、筛选 |
| `mainProductKeywords` | `JSON` | 否 | 字符串数组 | `["工控平板","触控终端"]` | 主营描述、搜索 |
| `companyType` | `VARCHAR(32)` | 是 | 见公司枚举 | `"limited_liability_company"` | 工商信息 |
| `businessStatus` | `VARCHAR(20)` | 是 | 见公司枚举 | `"active"` | 工商信息、前台状态 |
| `legalRepresentative` | `VARCHAR(80)` | 是 | 2–80 字 | `"张三"` | 工商信息 |
| `registeredCapital` | `DECIMAL(14,2)` | 否 | ≥0 | `1000000.00` | 工商信息 |
| `registeredCapitalCurrency` | `CHAR(3)` | 否 | ISO 4217 | `"CNY"` | 工商信息 |
| `establishedAt` | `DATE` | 否 | 不晚于当天 | `"2021-06-18"` | 工商信息 |
| `businessTermStart` | `DATE` | 否 | ≤结束日期 | `"2021-06-18"` | 工商信息 |
| `businessTermEnd` | `DATE` | 否 | 可为 `null`（长期） | `null` | 工商信息 |
| `businessScope` | `TEXT` | 否 | ≤5000 字 | `"智能装备销售..."` | 工商信息 |
| `registrationAuthority` | `VARCHAR(200)` | 否 | ≤200 字 | `"苏州市市场监督管理局"` | 工商信息 |
| `unifiedSocialCreditCode` | `CHAR(18)` | 是 | 中国统一社会信用代码格式 | `"9132XXXXXXXXXXXXXX"` | 工商核验，前台脱敏 |
| `organizationCode` | `VARCHAR(20)` | 否 | ≤20 字 | `"12345678-9"` | 工商信息 |
| `taxpayerId` | `VARCHAR(30)` | 否 | ≤30 字 | `"9132XXXXXXXXXXXXXX"` | 工商信息 |
| `registrationNumber` | `VARCHAR(50)` | 否 | ≤50 字 | `"3205XXXXXXXXXXXX"` | 工商信息 |
| `registeredAddress` | `VARCHAR(500)` | 否 | ≤500 字 | `"江苏省苏州市..."` | 工商信息 |
| `websiteUrl` | `VARCHAR(500)` | 否 | 合法 URL | `"https://example.com"` | 企业网址 |
| `status` | `VARCHAR(20)` | 是 | `draft`/`pending_review`/`online`/`offline`/`rejected`/`frozen` | `"online"` | 店铺上下架 |
| `createdAt` | `DATETIME` | 是 | 系统生成 | `"2026-09-24T..."` | 审计 |
| `updatedAt` | `DATETIME` | 是 | 系统生成 | `"2026-09-24T..."` | 审计 |

### 1.2 联系、地址、认证：建议拆表

#### `company_contact`

| 字段 | 类型 | 必填 | 示例 / 说明 |
| --- | --- | --- | --- |
| `contactId` | `CHAR(36)` | 是 | 联系人主键 |
| `companyId` | `CHAR(36)` | 是 | 所属公司 |
| `name` | `VARCHAR(80)` | 是 | `"李经理"` |
| `mobile` | `VARCHAR(32)` | 是 | 原始号码加密存储；前端按权限脱敏 |
| `telephone` | `VARCHAR(32)` | 否 | 固话 |
| `email` | `VARCHAR(120)` | 否 | 联系邮箱 |
| `position` | `VARCHAR(80)` | 否 | `"销售负责人"` |
| `isPrimary` | `BOOLEAN` | 是 | 是否主联系人 |
| `visibleToBuyer` | `BOOLEAN` | 是 | 是否允许前台展示 |

#### `company_shipping_address`

| 字段 | 类型 | 必填 | 示例 / 说明 |
| --- | --- | --- | --- |
| `addressId` | `CHAR(36)` | 是 | 地址主键 |
| `companyId` | `CHAR(36)` | 是 | 所属公司 |
| `provinceCode` / `cityCode` / `districtCode` | `VARCHAR(20)` | 是 | 使用统一行政区编码，支持地区筛选 |
| `addressDetail` | `VARCHAR(500)` | 是 | `"吴中区工业园区 1 号"` |
| `isDefaultShippingAddress` | `BOOLEAN` | 是 | 商品未单独配置发货地时继承 |

#### `company_verification`

| 字段 | 类型 | 必填 | 校验 / 说明 |
| --- | --- | --- | --- |
| `verificationId` | `CHAR(36)` | 是 | 认证记录主键 |
| `companyId` | `CHAR(36)` | 是 | 所属公司 |
| `verificationType` | `VARCHAR(30)` | 是 | `subject`（主体）、`factory`（验厂）、`business_license`（执照）、`supply`（供货） |
| `verificationStatus` | `VARCHAR(20)` | 是 | `pending`/`approved`/`rejected`/`expired` |
| `verifiedAt` / `expiresAt` | `DATETIME` | 否 | 核验与过期时间 |
| `documentUrls` | `JSON` | 否 | 认证凭证 URL 数组，仅有权限的用户可见 |
| `remark` | `TEXT` | 否 | 审核备注，仅后台 |

### 1.3 公司接口返回样例

```json
{
  "companyId": "f42fbca9-9628-4b42-9d1c-bdbe0b610001",
  "shopId": "be3d2ea0-9ef1-4317-b15c-a50d899b0001",
  "companyName": "华东智能装备有限公司",
  "shortName": "华智装备",
  "logoUrl": "https://cdn.example.com/company/hz-logo.png",
  "logoText": "HZ",
  "themeColor": "blue",
  "introduction": "专注于机械设备领域，为工业采购和工程项目提供稳定货源与按需定制支持。",
  "mainCategoryIds": ["cat-machinery", "cat-industrial-control"],
  "mainProductKeywords": ["工控平板", "数据采集终端"],
  "businessInfo": {
    "companyType": "limited_liability_company",
    "businessStatus": "active",
    "legalRepresentative": "张三",
    "registeredCapital": "1000000.00",
    "registeredCapitalCurrency": "CNY",
    "establishedAt": "2021-06-18",
    "businessTermStart": "2021-06-18",
    "businessTermEnd": null,
    "unifiedSocialCreditCodeMasked": "9132************",
    "registeredAddress": "江苏省苏州市吴中区工业园区",
    "businessScope": "智能装备、工业控制设备销售及配套服务。"
  },
  "shippingAddress": { "provinceCode": "320000", "cityCode": "320500", "districtCode": "320506", "displayName": "江苏·苏州" },
  "verification": { "subjectVerified": true, "factoryAudited": true, "verifiedAt": "2026-01-12T09:00:00+08:00" },
  "serviceCapabilities": ["source_factory", "customization", "batch_quote"],
  "statistics": { "operatingYears": 5, "responseRate": "98.00", "onSaleProductCount": 12 },
  "status": "online"
}
```

## 2. 商品 SPU `product`

一个 SPU 表示一个商品模型；有规格/价格/库存差异时，必须再建 SKU 表。当前页面商品详情、商品列表、店铺商品均从 SPU 读取，价格和库存优先取默认 SKU。

| 字段 | 类型（数据库） | 必填 | 校验 / 枚举 | 示例 | 页面用途 |
| --- | --- | --- | --- | --- | --- |
| `productId` | `CHAR(36)` | 是 | UUID，唯一 | `"b941..."` | 详情路由与主键 |
| `companyId` | `CHAR(36)` | 是 | 有效企业 ID | `"f42f..."` | 店铺关联、同店推荐 |
| `spuCode` | `VARCHAR(64)` | 是 | 企业内唯一 | `"HZ-IPC-001"` | 商品管理、对接 |
| `productName` | `VARCHAR(300)` | 是 | 2–300 字 | `"工业级触控一体机..."` | 所有商品页 |
| `shortDescription` | `VARCHAR(500)` | 否 | ≤500 字 | `"组态兼容，稳定高效"` | 列表标签、详情副标题 |
| `description` | `LONGTEXT` / JSON | 否 | 富文本/结构化内容 | `"<p>...</p>"` | 商品说明 |
| `categoryId` | `CHAR(36)` | 是 | 必须末级启用类目 | `"cat-industrial-control"` | 类目导航、筛选 |
| `brand` | `VARCHAR(100)` | 否 | ≤100 字 | `"华智"` | 搜索、选型 |
| `model` | `VARCHAR(150)` | 否 | ≤150 字 | `"HZ-IPC-15"` | 搜索、询价 |
| `manufacturer` | `VARCHAR(200)` | 否 | ≤200 字 | `"华东智能装备有限公司"` | 商品追溯 |
| `originPlace` | `VARCHAR(200)` | 否 | 行政区展示文本或地区 ID | `"江苏苏州"` | 商品参数 |
| `mainImageUrl` | `VARCHAR(500)` | 是 | 合法图片 URL | `"https://.../main.jpg"` | 首图 |
| `imageUrls` | `JSON` | 是 | 1–20 张图片 URL | `["https://.../1.jpg"]` | 详情图集 |
| `videoUrls` | `JSON` | 否 | 视频 URL 数组 | `[]` | 视频标识/详情 |
| `marketingBadge` | `VARCHAR(30)` | 否 | 如 `source_supply` | `"source_supply"` | 列表角标 |
| `sellingPoints` | `JSON` | 否 | 1–10 个短句 | `["支持定制","现货速发"]` | 列表/详情卖点 |
| `supplyMethod` | `JSON` | 是 | `spot`/`custom`/`preorder` | `["spot","custom"]` | 供货方式 |
| `applicationScenarios` | `JSON` | 否 | 字符串或场景 ID 数组 | `["production_line","project"]` | 产品参数 |
| `serviceGuarantees` | `JSON` | 否 | 服务 code 数组 | `["platform_verified","fast_shipping"]` | 详情服务保障 |
| `afterSalesService` | `TEXT` | 否 | ≤2000 字 | `"提供售后技术支持"` | 产品参数 |
| `technicalParameters` | `JSON` | 否 | 类目参数 key-value 数组 | 见下方样例 | 产品参数 |
| `featureTags` | `JSON` | 否 | 特性 code 数组 | `["selection_support"]` | 商品说明 |
| `customizationEnabled` | `BOOLEAN` | 是 | 0/1 | `true` | 定制入口 |
| `inquiryEnabled` | `BOOLEAN` | 是 | 0/1 | `true` | 立即询价 |
| `minOrderQuantity` | `DECIMAL(12,3)` | 是 | >0 | `1.000` | 起订量 |
| `unit` | `VARCHAR(20)` | 是 | 件/台/套/米等 | `"台"` | 数量显示 |
| `shipWithinHours` | `INT` | 否 | ≥0 | `48` | 发货时效 |
| `shippingAddressId` | `CHAR(36)` | 否 | 公司地址 ID | `"a109..."` | 商品独立发货地 |
| `searchKeywords` | `JSON` | 否 | ≤20 项 | `["工控屏","PLC终端"]` | 全文搜索 |
| `sortOrder` | `INT` | 是 | 默认 0 | `100` | 综合排序 |
| `salesCount` / `viewCount` | `INT` / `BIGINT` | 是 | ≥0，系统维护 | `38` / `1250` | 人气排序 |
| `saleStatus` | `VARCHAR(20)` | 是 | `draft`/`pending_review`/`on_sale`/`off_sale`/`rejected` | `"on_sale"` | 前台可见性 |
| `auditStatus` | `VARCHAR(20)` | 是 | `pending`/`approved`/`rejected` | `"approved"` | 审核流 |
| `createdAt` / `updatedAt` / `publishedAt` | `DATETIME` | 是 | 系统生成 | `"2026-09-24T..."` | 审计与排序 |

### 2.1 技术参数格式

不要把不同行业参数限制为固定字段。使用类目参数模板 + 商品填写值：

```json
[
  { "parameterId": "screen_size", "name": "屏幕尺寸", "value": "15.6", "unit": "英寸", "group": "基础参数" },
  { "parameterId": "protection_level", "name": "防护等级", "value": "IP65", "unit": "", "group": "基础参数" },
  { "parameterId": "interface", "name": "接口", "value": ["USB", "RS232", "LAN"], "unit": "", "group": "接口参数" }
]
```

### 2.2 商品接口返回样例

```json
{
  "productId": "b941f635-34cc-4aa8-85e0-a08fc0960001",
  "companyId": "f42fbca9-9628-4b42-9d1c-bdbe0b610001",
  "spuCode": "HZ-IPC-001",
  "productName": "工业级触控一体机 嵌入式工控平板",
  "shortDescription": "组态兼容，稳定高效；支持工程采购与按需定制。",
  "categoryId": "cat-industrial-control",
  "brand": "华智",
  "model": "HZ-IPC-15",
  "mainImageUrl": "https://cdn.example.com/product/hz-ipc-001/main.jpg",
  "imageUrls": ["https://cdn.example.com/product/hz-ipc-001/main.jpg", "https://cdn.example.com/product/hz-ipc-001/detail-1.jpg"],
  "marketingBadge": "source_supply",
  "sellingPoints": ["组态兼容", "稳定高效", "支持定制"],
  "supplyMethod": ["spot", "custom"],
  "applicationScenarios": ["production_line", "workshop", "engineering_project"],
  "serviceGuarantees": ["platform_verified", "customization", "fast_shipping"],
  "afterSalesService": "提供售后技术支持与选型建议。",
  "technicalParameters": [{ "parameterId": "screen_size", "name": "屏幕尺寸", "value": "15.6", "unit": "英寸", "group": "基础参数" }],
  "customizationEnabled": true,
  "inquiryEnabled": true,
  "minOrderQuantity": "1.000",
  "unit": "台",
  "shipWithinHours": 48,
  "saleStatus": "on_sale",
  "defaultSku": { "skuId": "db7f...", "displayPrice": "128000.00", "priceCurrency": "CNY", "priceFrom": true, "stockStatus": "in_stock" }
}
```

## 3. 商品 SKU、规格和阶梯价格

### `product_sku`

| 字段 | 类型 | 必填 | 示例 / 说明 |
| --- | --- | --- | --- |
| `skuId` / `productId` | `CHAR(36)` | 是 | SKU 主键、所属 SPU |
| `skuCode` | `VARCHAR(64)` | 是 | `"HZ-IPC-15-STD"`，同 SPU 内唯一 |
| `specValues` | `JSON` | 是 | `[{"specName":"配置","specValue":"标准配置"}]` |
| `imageUrl` | `VARCHAR(500)` | 否 | SKU 专属图 |
| `referencePrice` | `DECIMAL(14,2)` | 否 | `128000.00`；询价商品可为 `null` |
| `priceCurrency` | `CHAR(3)` | 是 | `"CNY"` |
| `priceUnit` | `VARCHAR(20)` | 是 | `"台"`、`"W"` |
| `priceFrom` | `BOOLEAN` | 是 | 是否显示“起” |
| `stockQuantity` | `DECIMAL(14,3)` | 是 | `120.000` |
| `stockStatus` | `VARCHAR(20)` | 是 | `in_stock`/`low_stock`/`out_of_stock`/`preorder` |
| `weightKg` | `DECIMAL(10,3)` | 否 | `18.500` |
| `lengthMm` / `widthMm` / `heightMm` | `DECIMAL(10,2)` | 否 | 包装或运输尺寸 |
| `status` | `VARCHAR(20)` | 是 | `active`/`inactive` |

### `sku_tier_price`

| 字段 | 类型 | 必填 | 示例 / 说明 |
| --- | --- | --- | --- |
| `tierPriceId` / `skuId` | `CHAR(36)` | 是 | 阶梯价主键、SKU 关联 |
| `minQuantity` | `DECIMAL(12,3)` | 是 | `10.000` |
| `maxQuantity` | `DECIMAL(12,3)` | 否 | `99.000`；空表示无上限 |
| `unitPrice` | `DECIMAL(14,2)` | 是 | `118000.00` |
| `currency` | `CHAR(3)` | 是 | `"CNY"` |

## 4. 类目与参数模板

### `category`

| 字段 | 类型 | 必填 | 说明 |
| --- | --- | --- | --- |
| `categoryId` | `CHAR(36)` | 是 | 类目主键 |
| `parentId` | `CHAR(36)` | 否 | 一级类目为 `null` |
| `categoryName` | `VARCHAR(100)` | 是 | 例如“机械设备”“手动工具”“套筒” |
| `level` | `TINYINT` | 是 | 当前页面为 1–3 级 |
| `path` | `VARCHAR(500)` | 是 | 如 `/machinery/tools/socket`，提升查询效率 |
| `icon` | `VARCHAR(100)` | 否 | 一级类目图标 |
| `sortOrder` | `INT` | 是 | 导航排序 |
| `status` | `VARCHAR(20)` | 是 | `active`/`disabled` |

### `category_parameter_template`

字段：`templateId`、`categoryId`、`parameterId`、`parameterName`、`inputType`（`text`/`number`/`select`/`multi_select`）、`unit`、`options`（JSON）、`required`、`sortOrder`。该表决定商品发布时要填的“技术参数”。

## 5. 询价单 `inquiry`

| 字段 | 类型（数据库） | 必填 | 校验 / 说明 |
| --- | --- | --- | --- |
| `inquiryId` | `CHAR(36)` | 是 | 询价单主键 |
| `companyId` | `CHAR(36)` | 是 | 被询价公司 |
| `productId` | `CHAR(36)` | 否 | 企业咨询可为空；商品询价必须有值 |
| `skuId` | `CHAR(36)` | 否 | 已选择规格时传入 |
| `productNameSnapshot` | `VARCHAR(300)` | 是 | 保存提交时商品名称，防止后续改名影响历史 |
| `requirement` | `VARCHAR(300)` | 是 | 当前页面限制 300 字 |
| `expectedSpec` | `VARCHAR(500)` | 否 | 型号、规格、按图定制说明 |
| `quantity` | `DECIMAL(12,3)` | 否 | 采购数量 |
| `unit` | `VARCHAR(20)` | 否 | 与数量配套 |
| `expectedDeliveryAt` | `DATE` | 否 | 希望交期 |
| `contactName` | `VARCHAR(80)` | 是 | 页面当前必填 |
| `contactPhone` | `VARCHAR(32)` | 是 | 页面当前必填；加密存储 |
| `contactEmail` | `VARCHAR(120)` | 否 | 可选补充 |
| `status` | `VARCHAR(20)` | 是 | `submitted`/`assigned`/`contacted`/`quoted`/`closed`/`invalid` |
| `handlerId` | `CHAR(36)` | 否 | 商家跟进人员 |
| `handledAt` | `DATETIME` | 否 | 首次处理时间 |
| `createdAt` / `updatedAt` | `DATETIME` | 是 | 系统生成 |

### 创建询价请求样例

```json
{
  "companyId": "f42fbca9-9628-4b42-9d1c-bdbe0b610001",
  "productId": "b941f635-34cc-4aa8-85e0-a08fc0960001",
  "skuId": "db7f7fab-d18e-410c-9e64-7f75ebdd0001",
  "requirement": "采购 20 台，需提供 15.6 英寸标准配置的含税报价和交付周期。",
  "expectedSpec": "标准配置",
  "quantity": "20.000",
  "unit": "台",
  "expectedDeliveryAt": "2026-10-15",
  "contactName": "王先生",
  "contactPhone": "13800138000"
}
```

## 6. 前端枚举字典

```json
{
  "companyStatus": ["draft", "pending_review", "online", "offline", "rejected", "frozen"],
  "businessStatus": ["active", "cancelled", "revoked", "moved_out"],
  "companyType": ["limited_liability_company", "sole_proprietorship", "partnership", "individual_business"],
  "supplyMethod": ["spot", "custom", "preorder"],
  "serviceGuarantee": ["platform_verified", "factory_audited", "customization", "fast_shipping", "after_sales", "platform_guarantee"],
  "productSaleStatus": ["draft", "pending_review", "on_sale", "off_sale", "rejected"],
  "inquiryStatus": ["submitted", "assigned", "contacted", "quoted", "closed", "invalid"]
}
```

## 7. 页面字段替换映射

| 当前前端字段 / 固定内容 | 应替换为 |
| --- | --- |
| `supplier.name`、`supplier.shortName`、`supplier.mark`、`supplier.tone` | `company.companyName`、`shortName`、`logoUrl/logoText`、`themeColor` |
| `supplier.category` | `company.mainCategoryIds` + 类目字典 |
| 固定“经营 5 年”“98% 响应率”“已验厂” | `company.statistics`、`company.verification` |
| 固定工商表格 | `company.businessInfo` |
| `product.name`、`price`、`tag`、`badge`、`image` | `product.productName`、`defaultSku`、`sellingPoints`、`marketingBadge`、`mainImageUrl` |
| 固定产品参数、规格、48 小时发货 | `technicalParameters`、`product_sku.specValues`、`shipWithinHours` |
| 只提交“需求、联系人、电话”的弹窗 | `POST /inquiries`，字段见第 5 节 |

## 8. 最小可上线字段集

如果第一期只支持展示、列表、详情和询价，最少应实现：

- 公司：`companyId`、`companyName`、`shortName`、`logoUrl/logoText`、`mainCategoryIds`、`introduction`、地区、认证状态、`status`。
- 商品：`productId`、`companyId`、`productName`、`categoryId`、`mainImageUrl`、`imageUrls`、卖点、`minOrderQuantity`、`unit`、`saleStatus`。
- SKU：`skuId`、`productId`、规格、`referencePrice`、`priceUnit`、`priceFrom`、库存状态。
- 询价：`companyId`、`productId`、需求、联系人、电话、状态、创建时间。

工商全量信息、资质文件、技术参数模板、阶梯价和库存数量可以第二期补齐；但 ID、状态字段、SKU 结构和商品/公司关联必须在第一期就确定。
