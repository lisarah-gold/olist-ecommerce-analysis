# 数据字典（Data Dictionary）

## 1. 文档目的

本文件用于说明 Olist 巴西电商公开数据集中各张核心表的业务含义、数据粒度、关键字段、逻辑连接关系，以及这些字段在本项目中的使用方式。

本项目使用的是历史公开数据。数据字典中的“候选唯一标识”“逻辑连接字段”和“数据质量状态”需要结合 `notebooks/01_data_understanding.ipynb` 的实际检查结果确认。

---

## 2. 数据集范围

* 数据来源：Brazilian E-Commerce Public Dataset by Olist
* 原始数据位置：`data/raw/`
* 数据表数量：9 张 CSV 表
* 项目分析主题：

  * 月度经营趋势
  * 商品品类表现
  * 客户重复购买
  * 区域差异
  * 配送表现
  * 配送与客户评分的关系

---

## 3. 核心表总览

| 原始 CSV 文件                               | 项目表名                           | 一行代表什么            | 候选唯一标识                                        | 主要逻辑连接字段                             | 本项目主要用途             |
| --------------------------------------- | ------------------------------ | ----------------- | --------------------------------------------- | ------------------------------------ | ------------------- |
| `olist_customers_dataset.csv`           | `customers`                    | 一条客户与订单关联记录       | `customer_id`，待 Notebook 验证                   | `customer_id`、`customer_unique_id`   | 客户地区、客户重复购买、州级分析    |
| `olist_orders_dataset.csv`              | `orders`                       | 一笔订单              | `order_id`，待 Notebook 验证                      | `order_id`、`customer_id`             | 订单状态、订单时间、配送时间、延迟判断 |
| `olist_order_items_dataset.csv`         | `order_items`                  | 一笔订单中的一件商品明细      | `order_id + order_item_id`，待 Notebook 验证      | `order_id`、`product_id`、`seller_id`  | 商品金额、商品品类、卖家、运费     |
| `olist_order_payments_dataset.csv`      | `order_payments`               | 一笔订单中的一条支付记录      | `order_id + payment_sequential`，待 Notebook 验证 | `order_id`                           | 支付方式、支付金额、分期次数      |
| `olist_order_reviews_dataset.csv`       | `order_reviews`                | 一条订单评价记录          | `review_id`，待 Notebook 验证                     | `order_id`                           | 评分、低评分率、配送与评分对比     |
| `olist_products_dataset.csv`            | `products`                     | 一个商品              | `product_id`，待 Notebook 验证                    | `product_id`、`product_category_name` | 商品品类、商品属性           |
| `product_category_name_translation.csv` | `product_category_translation` | 一条商品品类葡萄牙语与英语映射记录 | `product_category_name`，待 Notebook 验证         | `product_category_name`              | 图表中的英文品类名称          |
| `olist_sellers_dataset.csv`             | `sellers`                      | 一个卖家              | `seller_id`，待 Notebook 验证                     | `seller_id`、`seller_zip_code_prefix` | 卖家维度和可选区域分析         |
| `olist_geolocation_dataset.csv`         | `geolocation`                  | 一条邮编前缀对应的地理位置记录   | 待 Notebook 验证                                 | `geolocation_zip_code_prefix`        | 可选地图分析、客户和卖家区域补充    |

---

## 4. 表关系与分析粒度说明

### 4.1 核心逻辑关系

```text
customers.customer_id
        ↓
orders.customer_id

orders.order_id
        ↓
order_items.order_id
        ↓
products.product_id

order_items.seller_id
        ↓
sellers.seller_id

orders.order_id
        ↓
order_payments.order_id

orders.order_id
        ↓
order_reviews.order_id

products.product_category_name
        ↓
product_category_translation.product_category_name
```

### 4.2 重要粒度规则

| 表                              | 数据粒度      | 分析时需要注意什么                                                |
| ------------------------------ | --------- | -------------------------------------------------------- |
| `orders`                       | 订单粒度      | 一笔订单应只计算一次                                               |
| `order_items`                  | 订单商品明细粒度  | 一笔订单可能有多件商品，因此同一个 `order_id` 可能重复出现                      |
| `order_payments`               | 支付记录粒度    | 一笔订单可能存在多条支付记录，直接连接可能导致订单数重复                             |
| `order_reviews`                | 评价记录粒度    | 需要检查一个订单是否可能对应多条评价                                       |
| `customers`                    | 客户—订单关联粒度 | `customer_id` 用于连接订单；识别真实客户重复购买时要使用 `customer_unique_id` |
| `products`                     | 商品粒度      | 一个 `product_id` 对应一个商品                                   |
| `sellers`                      | 卖家粒度      | 一个 `seller_id` 对应一个卖家                                    |
| `geolocation`                  | 地理位置记录粒度  | 邮编前缀可能不唯一，需要验证后再用于连接                                     |
| `product_category_translation` | 品类映射粒度    | 用于把葡萄牙语品类名转换为英文品类名                                       |

---

# 5. 各表字段说明

## 5.1 customers：客户与订单关联表

### 表说明

* 原始文件：`olist_customers_dataset.csv`
* 一行代表什么：一条客户与订单关联记录。
* 主要作用：将订单连接到客户地区；使用 `customer_unique_id` 识别同一个真实客户是否有重复购买。
* 关键提醒：`customer_id` 与 `customer_unique_id` 不相同。

| 字段                         | 字段含义            | 原始读取类型        | 是否缺失          | 是否用于连接 | 本项目用途         | 备注                        |
| -------------------------- | --------------- | ------------- | ------------- | -----: | ------------- | ------------------------- |
| `customer_id`              | 客户记录 ID，用于连接订单表 | 待 Notebook 验证 | 待 Notebook 验证 |      是 | 连接 `orders` 表 | 候选唯一标识                    |
| `customer_unique_id`       | 同一真实客户的唯一标识     | 待 Notebook 验证 | 待 Notebook 验证 |      是 | 重复购买分析        | 复购分析必须使用该字段               |
| `customer_zip_code_prefix` | 客户邮编前缀          | 待 Notebook 验证 | 待 Notebook 验证 |     可选 | 区域补充、地图分析     | 可连接 `geolocation`，但唯一性待验证 |
| `customer_city`            | 客户所在城市          | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 区域分析补充        | 城市名称可能存在拼写或格式差异           |
| `customer_state`           | 客户所在州           | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 州级订单、配送和评分分析  | Q4 核心字段                   |

---

## 5.2 orders：订单表

### 表说明

* 原始文件：`olist_orders_dataset.csv`
* 一行代表什么：一笔订单。
* 主要作用：订单状态筛选、月度趋势、配送时长、延迟订单判断。
* 关键提醒：后续“已完成订单”通常使用 `order_status = 'delivered'` 作为筛选条件。

| 字段                              | 字段含义      | 原始读取类型        | 是否缺失          | 是否用于连接 | 本项目用途            | 备注               |
| ------------------------------- | --------- | ------------- | ------------- | -----: | ---------------- | ---------------- |
| `order_id`                      | 订单 ID     | 待 Notebook 验证 | 待 Notebook 验证 |      是 | 连接订单商品、支付和评价表    | 候选唯一标识           |
| `customer_id`                   | 下单客户记录 ID | 待 Notebook 验证 | 待 Notebook 验证 |      是 | 连接 `customers` 表 | 不能直接用于复购判断       |
| `order_status`                  | 订单状态      | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 筛选已完成订单          | 需检查有哪些状态         |
| `order_purchase_timestamp`      | 下单时间      | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 月度趋势、配送时长        | 后续需要转换为 datetime |
| `order_approved_at`             | 订单审批时间    | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选订单流程分析         | 非核心指标字段          |
| `order_delivered_carrier_date`  | 订单交给承运商时间 | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选配送流程分析         | 非核心指标字段          |
| `order_delivered_customer_date` | 实际送达客户时间  | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 配送时长、延迟判断        | 配送分析关键字段         |
| `order_estimated_delivery_date` | 预计送达时间    | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 延迟订单判断           | 与实际送达时间比较        |

---

## 5.3 order_items：订单商品明细表

### 表说明

* 原始文件：`olist_order_items_dataset.csv`
* 一行代表什么：一笔订单中的一件商品明细。
* 主要作用：计算商品金额代理指标、连接商品品类、连接卖家、分析运费。
* 关键提醒：一个订单可以有多件商品，因此 `order_id` 在该表中通常不是唯一的。

| 字段                    | 字段含义       | 原始读取类型        | 是否缺失          | 是否用于连接 | 本项目用途           | 备注                      |
| --------------------- | ---------- | ------------- | ------------- | -----: | --------------- | ----------------------- |
| `order_id`            | 订单 ID      | 待 Notebook 验证 | 待 Notebook 验证 |      是 | 连接 `orders` 表   | 一个订单可能多行                |
| `order_item_id`       | 同一订单中的商品序号 | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 区分同一订单中的商品明细    | 与 `order_id` 共同构成候选组合标识 |
| `product_id`          | 商品 ID      | 待 Notebook 验证 | 待 Notebook 验证 |      是 | 连接 `products` 表 | 用于品类分析                  |
| `seller_id`           | 卖家 ID      | 待 Notebook 验证 | 待 Notebook 验证 |      是 | 连接 `sellers` 表  | 可用于卖家分析                 |
| `shipping_limit_date` | 卖家最晚发货时间限制 | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选配送流程分析        | 非核心指标字段                 |
| `price`               | 商品价格       | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 商品金额代理指标        | 不等于真实企业营收或利润            |
| `freight_value`       | 运费金额       | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 运费与订单分析         | 可选分析字段                  |

---

## 5.4 order_payments：订单支付表

### 表说明

* 原始文件：`olist_order_payments_dataset.csv`
* 一行代表什么：一笔订单中的一条支付记录。
* 主要作用：支付方式、支付金额、分期次数分析和金额核验。
* 关键提醒：一个订单可能存在多条支付记录，连接时可能导致订单重复。

| 字段                     | 字段含义         | 原始读取类型        | 是否缺失          | 是否用于连接 | 本项目用途         | 备注                     |
| ---------------------- | ------------ | ------------- | ------------- | -----: | ------------- | ---------------------- |
| `order_id`             | 订单 ID        | 待 Notebook 验证 | 待 Notebook 验证 |      是 | 连接 `orders` 表 | 一个订单可能多条支付记录           |
| `payment_sequential`   | 同一订单中支付记录的顺序 | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 区分多条支付记录      | 可与 `order_id` 构成候选组合标识 |
| `payment_type`         | 支付方式         | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选支付方式分析      | 需要检查类别值                |
| `payment_installments` | 分期付款次数       | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选分期分析        | 数值字段                   |
| `payment_value`        | 支付金额         | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 支付金额核验        | 与商品价格和运费的关系后续再验证       |

---

## 5.5 order_reviews：订单评价表

### 表说明

* 原始文件：`olist_order_reviews_dataset.csv`
* 一行代表什么：一条订单评价记录。
* 主要作用：平均评分、低评分率、配送与评分关系分析。
* 关键提醒：没有评价不代表客户满意；评分分析的分母只能使用有效评价订单。

| 字段                        | 字段含义      | 原始读取类型        | 是否缺失          | 是否用于连接 | 本项目用途         | 备注              |
| ------------------------- | --------- | ------------- | ------------- | -----: | ------------- | --------------- |
| `review_id`               | 评价 ID     | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 评价记录识别        | 候选唯一标识          |
| `order_id`                | 被评价的订单 ID | 待 Notebook 验证 | 待 Notebook 验证 |      是 | 连接 `orders` 表 | 需检查一个订单是否可能多条评价 |
| `review_score`            | 客户评分      | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 平均评分、低评分率     | 低评分暂定义为评分小于等于 2 |
| `review_comment_title`    | 评价标题      | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 非核心字段         | 可能存在缺失          |
| `review_comment_message`  | 评价正文      | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选文本分析        | 本项目核心版暂不做文本分析   |
| `review_creation_date`    | 评价创建时间    | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选时间分析        | 非核心字段           |
| `review_answer_timestamp` | 商家或平台回复时间 | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选服务响应分析      | 非核心字段           |

---

## 5.6 products：商品表

### 表说明

* 原始文件：`olist_products_dataset.csv`
* 一行代表什么：一个商品。
* 主要作用：为订单商品明细补充商品品类和商品属性。
* 关键提醒：商品类别可能存在缺失，需要在后续品类分析中明确处理规则。

| 字段                           | 字段含义             | 原始读取类型        | 是否缺失          | 是否用于连接 | 本项目用途              | 备注               |
| ---------------------------- | ---------------- | ------------- | ------------- | -----: | ------------------ | ---------------- |
| `product_id`                 | 商品 ID            | 待 Notebook 验证 | 待 Notebook 验证 |      是 | 连接 `order_items` 表 | 候选唯一标识           |
| `product_category_name`      | 商品品类名称，原始语言为葡萄牙语 | 待 Notebook 验证 | 待 Notebook 验证 |      是 | 商品品类分析             | 可连接翻译表           |
| `product_name_lenght`        | 商品名称长度           | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选商品属性分析           | 原始字段拼写为 `lenght` |
| `product_description_lenght` | 商品描述长度           | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选商品属性分析           | 原始字段拼写为 `lenght` |
| `product_photos_qty`         | 商品图片数量           | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选商品属性分析           | 非核心字段            |
| `product_weight_g`           | 商品重量，单位为克        | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选商品属性分析           | 非核心字段            |
| `product_length_cm`          | 商品长度，单位为厘米       | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选商品属性分析           | 非核心字段            |
| `product_height_cm`          | 商品高度，单位为厘米       | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选商品属性分析           | 非核心字段            |
| `product_width_cm`           | 商品宽度，单位为厘米       | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选商品属性分析           | 非核心字段            |

---

## 5.7 product_category_translation：商品品类翻译表

### 表说明

* 原始文件：`product_category_name_translation.csv`
* 一行代表什么：一条商品品类名称翻译映射。
* 主要作用：将葡萄牙语商品品类名称转换为英文，提高图表、Dashboard 和报告的可读性。

| 字段                              | 字段含义       | 原始读取类型        | 是否缺失          | 是否用于连接 | 本项目用途           | 备注           |
| ------------------------------- | ---------- | ------------- | ------------- | -----: | --------------- | ------------ |
| `product_category_name`         | 葡萄牙语商品品类名称 | 待 Notebook 验证 | 待 Notebook 验证 |      是 | 连接 `products` 表 | 候选唯一标识       |
| `product_category_name_english` | 英文商品品类名称   | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 图表和报告展示         | 后续优先使用英文名称展示 |

---

## 5.8 sellers：卖家表

### 表说明

* 原始文件：`olist_sellers_dataset.csv`
* 一行代表什么：一个卖家。
* 主要作用：为订单商品补充卖家区域信息；核心版中不是重点，但可用于进阶分析。

| 字段                       | 字段含义   | 原始读取类型        | 是否缺失          | 是否用于连接 | 本项目用途              | 备注                       |
| ------------------------ | ------ | ------------- | ------------- | -----: | ------------------ | ------------------------ |
| `seller_id`              | 卖家 ID  | 待 Notebook 验证 | 待 Notebook 验证 |      是 | 连接 `order_items` 表 | 候选唯一标识                   |
| `seller_zip_code_prefix` | 卖家邮编前缀 | 待 Notebook 验证 | 待 Notebook 验证 |     可选 | 卖家区域补充             | 可连接 `geolocation`，唯一性待验证 |
| `seller_city`            | 卖家所在城市 | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选卖家区域分析           | 非核心字段                    |
| `seller_state`           | 卖家所在州  | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选卖家区域分析           | 非核心字段                    |

---

## 5.9 geolocation：地理位置表

### 表说明

* 原始文件：`olist_geolocation_dataset.csv`
* 一行代表什么：一条邮编前缀与经纬度、城市、州之间的地理位置记录。
* 主要作用：补充客户或卖家的区域信息，支持进阶地图分析。
* 关键提醒：需要先检查 `geolocation_zip_code_prefix` 是否唯一；若不唯一，不能直接把该字段当成严格主键。

| 字段                            | 字段含义 | 原始读取类型        | 是否缺失          | 是否用于连接 | 本项目用途        | 备注       |
| ----------------------------- | ---- | ------------- | ------------- | -----: | ------------ | -------- |
| `geolocation_zip_code_prefix` | 邮编前缀 | 待 Notebook 验证 | 待 Notebook 验证 |      是 | 连接客户或卖家的邮编前缀 | 唯一性必须验证  |
| `geolocation_lat`             | 纬度   | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选地图分析       | 数值字段     |
| `geolocation_lng`             | 经度   | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 可选地图分析       | 数值字段     |
| `geolocation_city`            | 城市名称 | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 地理位置补充       | 可能存在格式差异 |
| `geolocation_state`           | 州名称  | 待 Notebook 验证 | 待 Notebook 验证 |      否 | 地理位置补充       | 非核心字段    |

---

## 6. Phase 1 验证后需要更新的内容

完成 `notebooks/01_data_understanding.ipynb` 后，需要将本文件中的“待 Notebook 验证”替换为真实结果，包括：

1. 每个关键字段的 pandas 数据类型；
2. 每个关键字段是否存在缺失值；
3. 候选唯一标识是否真实唯一；
4. 组合字段是否能够唯一识别记录；
5. 一个订单是否存在多件商品；
6. 一个订单是否存在多条支付记录；
7. 一个订单是否可能存在多条评价；
8. `geolocation_zip_code_prefix` 是否唯一；
9. 每张表的行数、列数和关键字段检查结果。

详细的数据质量问题、清洗决定和可能影响，应记录在：

```text
docs/data-quality-log.md
```
