# 数据质量日志（Data Quality Log）

## 1. 文档目的

本文件记录 Phase 1 数据理解阶段在 `notebooks/01_data_understanding.ipynb` 中发现的数据质量问题、数据粒度风险、表关联限制，以及后续 SQL 和 Python 分析时需要遵守的处理规则。

本文件中的“初步处理方式”是后续分析的规则，不代表当前已经删除、修改或覆盖了原始数据。

---

## 2. 检查范围与数据来源

* 数据集：Brazilian E-Commerce Public Dataset by Olist
* 原始数据位置：`data/raw/`
* 检查 Notebook：`notebooks/01_data_understanding.ipynb`
* 检查日期：2026-07-01
* 原始 CSV 数量：9 张
* CSV 文件完整性：已确认 9 张预期 CSV 均存在，无缺失文件、无额外 CSV 文件。

### 2.1 各表规模概览

| 表名                     | 原始 CSV 文件                               |        行数 | 列数 |
| ---------------------- | --------------------------------------- | --------: | -: |
| `customers`            | `olist_customers_dataset.csv`           |    99,441 |  5 |
| `geolocation`          | `olist_geolocation_dataset.csv`         | 1,000,163 |  5 |
| `order_items`          | `olist_order_items_dataset.csv`         |   112,650 |  7 |
| `order_payments`       | `olist_order_payments_dataset.csv`      |   103,886 |  5 |
| `order_reviews`        | `olist_order_reviews_dataset.csv`       |    99,224 |  7 |
| `orders`               | `olist_orders_dataset.csv`              |    99,441 |  8 |
| `products`             | `olist_products_dataset.csv`            |    32,951 |  9 |
| `sellers`              | `olist_sellers_dataset.csv`             |     3,095 |  4 |
| `category_translation` | `product_category_name_translation.csv` |        71 |  2 |

---

## 3. 数据质量问题与处理决策

|    编号 | 发现日期       | 问题                                                    | 证据与真实检查结果                                                                                                                                                              | 涉及表/字段                                                                                                          | 初步处理方式                                                                                          | 是否影响核心指标    |
| ----: | ---------- | ----------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- | ----------- |
| DQ-01 | 2026-06-28 | 一个订单可能有多个商品明细，直接连接后会放大订单行数。                           | `order_items.order_id` 有 13,984 条重复记录；但 `order_id + order_item_id` 组合无缺失且无重复，说明一笔订单可包含多件商品。                                                                            | `order_items.order_id`、`order_items.order_item_id`                                                              | 在订单趋势、订单数、配送和评分分析中，以订单粒度计算，并使用 `COUNT(DISTINCT order_id)` 或先汇总到订单级。商品金额、商品品类和卖家分析可保留订单商品明细粒度。   | 是           |
| DQ-02 | 2026-06-28 | 一个订单可能有多条支付记录，直接连接后会放大订单行数或金额。                        | `order_payments.order_id` 有 4,446 条重复记录；但 `order_id + payment_sequential` 组合无缺失且无重复。                                                                                   | `order_payments.order_id`、`payment_sequential`                                                                  | 核心版不直接将支付表连接到订单级分析表。若后续做支付分析，先按 `order_id` 汇总支付记录，再与订单表连接。                                      | 是           |
| DQ-03 | 2026-06-28 | 评价表中 `review_id` 和 `order_id` 均存在重复，不能默认“一笔订单只有一条评价”。 | `review_id` 有 814 条重复记录；`order_id` 有 551 条重复记录。                                                                                                                        | `order_reviews.review_id`、`order_reviews.order_id`                                                              | 当前不删除任何评价记录。进入 SQL 评分分析前，先检查重复评价的具体模式；构建订单级评分表时必须明确并记录去重或汇总规则，避免同一订单被重复计入低评分率或平均评分。             | 是           |
| DQ-04 | 2026-06-28 | 部分订单没有实际送达时间，无法计算配送时长或是否延迟。                           | `orders.order_delivered_customer_date` 缺失 2,965 条，占订单表 2.98%。                                                                                                          | `orders.order_delivered_customer_date`                                                                          | 配送时长、延迟率和配送与评分关系分析，仅保留 `order_status = 'delivered'`、实际送达时间非空、预计送达时间非空的订单。缺失送达时间的订单不进入配送相关指标分母。  | 是           |
| DQ-05 | 2026-06-29 | 部分订单缺少审批时间或交给承运商时间。                                   | `order_approved_at` 缺失 160 条，占 0.16%；`order_delivered_carrier_date` 缺失 1,783 条，占 1.79%。                                                                                | `orders.order_approved_at`、`orders.order_delivered_carrier_date`                                                | 核心版配送分析不使用这两个字段计算主要指标，因此不删除订单。若后续做订单审批或发货流程分析，则只使用相应时间字段完整的订单，并单独记录样本范围。                        | 否，核心版暂不影响   |
| DQ-06 | 2026-06-29 | 商品品类字段存在缺失，部分商品无法归入具体品类。                              | `products.product_category_name` 缺失 610 条，占商品表 1.85%。与该字段相关的商品名称长度、描述长度和图片数量字段也各缺失 610 条。                                                                              | `products.product_category_name` 及相关商品属性字段                                                                      | 核心品类分析中，不删除整张商品表或订单商品明细。缺失品类统一标记为 `Unknown` 或 `Missing category`，并在最终图表或报告中说明该分类包含缺失原始值。        | 是           |
| DQ-07 | 2026-06-29 | 部分葡萄牙语商品品类无法映射为英文品类名称。                                | 商品表中有 32,341 条非空品类记录，其中 13 条非空品类记录未能在 `category_translation` 中找到匹配；匹配覆盖率为 99.96%。                                                                                      | `products.product_category_name`、`category_translation.product_category_name`                                   | 图表和报告优先使用英文品类名称；对无法翻译的非空品类，保留原葡萄牙语名称或标记为 `Untranslated`，不因为没有英文翻译而删除数据。                         | 是           |
| DQ-08 | 2026-06-29 | 评价标题和评价正文缺失比例较高，不适合直接进行评论文本分析。                        | `review_comment_title` 缺失 87,656 条，占 88.34%；`review_comment_message` 缺失 58,247 条，占 58.70%。                                                                             | `order_reviews.review_comment_title`、`review_comment_message`                                                   | 核心版只使用完整的 `review_score` 进行平均评分和低评分率分析，不开展评价文本、情感分析或关键词分析。没有文本评论不代表客户满意或不满意。                    | 否，核心版不做文本分析 |
| DQ-09 | 2026-06-29 | 地理位置表的邮编前缀不唯一，不能将其当作严格一对一主键使用。                        | `geolocation.geolocation_zip_code_prefix` 有 981,148 条重复记录。                                                                                                             | `geolocation.geolocation_zip_code_prefix`                                                                       | 核心版州级分析直接使用 `customers.customer_state`，不依赖地理位置表。若后续进行地图或经纬度分析，需先按邮编前缀聚合地理位置表，并记录聚合方法。           | 否，核心版暂不影响   |
| DQ-10 | 2026-07-01 | 客户和卖家的部分邮编前缀无法在地理位置表中匹配。                              | 客户邮编前缀有 278 条未匹配，匹配率为 99.72%；卖家邮编前缀有 7 条未匹配，匹配率为 99.77%。                                                                                                               | `customers.customer_zip_code_prefix`、`sellers.seller_zip_code_prefix`、`geolocation.geolocation_zip_code_prefix` | 核心版不使用邮编前缀直接计算关键 KPI。若进行地图分析，无法匹配的记录不进入经纬度地图，并在分析说明中写明覆盖范围。                                     | 否，核心版暂不影响   |
| DQ-11 | 2026-07-01 | 订单并非全部已经完成，不能将所有订单混合用于经营、配送和评分指标。                     | 订单状态分布：`delivered` 96,478 条；`shipped` 1,107 条；`canceled` 625 条；`unavailable` 609 条；`invoiced` 314 条；`processing` 301 条；`created` 5 条；`approved` 2 条。                   | `orders.order_status`                                                                                           | 月度经营趋势、商品金额代理指标、复购、配送和评分核心分析默认使用 `order_status = 'delivered'`。如需要分析订单状态结构，应单独按状态统计，不与已完成订单指标混合。 | 是           |
| DQ-12 | 2026-07-01 | 原始日期字段目前以字符串读取，后续不能直接进行月份提取和时间差计算。                    | `order_purchase_timestamp`、`order_approved_at`、`order_delivered_carrier_date`、`order_delivered_customer_date`、`order_estimated_delivery_date` 均可成功转换为日期时间格式，转换失败数均为 0。 | `orders` 中 5 个时间字段                                                                                              | 在后续 SQL 与 Python 清洗阶段，将日期字段转换为正确的日期时间类型。计算配送天数时使用实际送达时间减去购买时间；判断延迟时比较实际送达时间与预计送达时间。             | 是           |

---

## 4. 已验证的关键数据完整性结果

以下结果不是数据问题，但会作为后续分析中可放心使用的基础。

| 检查项目                                           | 真实检查结果      | 结论                          |
| ---------------------------------------------- | ----------- | --------------------------- |
| `orders.order_id`                              | 无缺失、无重复     | 可作为订单表唯一标识。                 |
| `customers.customer_id`                        | 无缺失、无重复     | 可作为客户—订单关联记录的唯一标识，并用于连接订单表。 |
| `products.product_id`                          | 无缺失、无重复     | 可作为商品表唯一标识。                 |
| `sellers.seller_id`                            | 无缺失、无重复     | 可作为卖家表唯一标识。                 |
| `category_translation.product_category_name`   | 无缺失、无重复     | 翻译表中的葡萄牙语品类名一对一映射规则成立。      |
| `orders.customer_id → customers.customer_id`   | 匹配率 100.00% | 所有订单均可找到对应客户记录。             |
| `order_items.order_id → orders.order_id`       | 匹配率 100.00% | 所有订单商品明细均可找到对应订单。           |
| `order_payments.order_id → orders.order_id`    | 匹配率 100.00% | 所有支付记录均可找到对应订单。             |
| `order_reviews.order_id → orders.order_id`     | 匹配率 100.00% | 所有评价记录均可找到对应订单。             |
| `order_items.product_id → products.product_id` | 匹配率 100.00% | 所有订单商品明细均可找到对应商品。           |
| `order_items.seller_id → sellers.seller_id`    | 匹配率 100.00% | 所有订单商品明细均可找到对应卖家。           |

---

## 5. 日期范围验证结果

| 时间字段                            | 最早日期                | 最晚日期                | 转换失败数 |
| ------------------------------- | ------------------- | ------------------- | ----: |
| `order_purchase_timestamp`      | 2016-09-04 21:15:19 | 2018-10-17 17:30:18 |     0 |
| `order_approved_at`             | 2016-09-15 12:16:38 | 2018-09-03 17:40:06 |     0 |
| `order_delivered_carrier_date`  | 2016-10-08 10:34:01 | 2018-09-11 19:48:28 |     0 |
| `order_delivered_customer_date` | 2016-10-11 13:46:32 | 2018-10-17 13:22:46 |     0 |
| `order_estimated_delivery_date` | 2016-09-30 00:00:00 | 2018-11-12 00:00:00 |     0 |

---

## 6. Phase 1 数据处理原则

在进入 Phase 2 和后续分析前，遵守以下原则：

1. 原始 CSV 文件保持不变，不直接覆盖、不删除原始行。
2. 所有清洗、筛选、去重、聚合和排除规则必须在 SQL、Notebook 或指标口径文档中记录。
3. 订单数、配送指标和评分指标优先使用订单粒度，避免被订单商品明细或支付记录重复放大。
4. 商品金额和商品品类分析可以使用订单商品明细粒度，但必须明确说明统计粒度。
5. 客户复购分析必须使用 `customer_unique_id`，不能用 `customer_id` 代替。
6. 评分分析不把“没有评论文本”或“没有评价记录”解释为客户满意。
7. 核心版不使用地理位置表计算严格的一对一地理匹配结果。
8. 所有项目结论仅代表 2016—2018 年 Olist 历史公开数据样本，不代表 Olist 当前经营情况。

---

## 7. 后续阶段待执行事项

| 后续阶段                  | 需要执行的工作                                            |
| --------------------- | -------------------------------------------------- |
| Phase 2：MySQL 导入与验证   | 在导入后再次检查行数、字段类型、空值和关键 ID，确认数据库中的结果与 CSV 检查结果一致。    |
| Phase 2：指标口径定义        | 明确已完成订单数、商品金额代理指标、复购率、配送天数、延迟率和低评分率的分子、分母、粒度和筛选条件。 |
| Phase 3：SQL 核心分析      | 对 `order_reviews` 的重复订单评价进行具体模式检查后，再确定订单级评分分析规则。   |
| Phase 4：Python 清洗与可视化 | 生成日期衍生字段、订单级分析表、延迟标记和客户购买次数，并保留清洗前后样本量记录。          |
| 进阶分析                  | 如做地图分析，先定义地理位置表按邮编前缀的聚合方法，并说明未匹配邮编不会进入地图结果。        |
