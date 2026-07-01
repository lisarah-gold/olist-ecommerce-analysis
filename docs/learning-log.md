# Learning Log

## 2026-06-27 — Phase 0
### 我已完成的内容

* 创建了项目的初始文件夹结构。
* 添加了 `project-charter.md`、`README.md`、`AGENTS.md`、`requirements.txt` 和 `.gitignore`。
* 创建了 `docs` 文件夹。
* 在 `docs` 文件夹中创建了 `phase-checklist.md` 和 `learning-log.md`。
* 将项目章程更新为 v3.0 版本。
* 在 `requirements.txt` 中写入了本项目需要的基础 Python 依赖。

### 我学到的内容

* 虚拟环境（`.venv`）可以让不同 Python 项目的依赖相互隔离，避免项目之间的包版本冲突。
* `.gitignore` 用于防止本地文件、敏感文件、大型文件或不需要上传的文件被 Git 跟踪并上传到 GitHub。
* `requirements.txt` 用于记录运行该项目所需要安装的 Python 包。
* `README.md` 用于向其他人说明项目的目的、数据来源、技术栈和当前进度。
* 阶段验收清单用于确认每项任务是否真实完成，而不是仅凭印象判断。
* 学习日志用于记录我做了什么、理解了什么、遇到了什么问题，以及下一步计划做什么。

### 当前问题或待确认事项

* 需要确认项目根目录中是否真实存在 `.venv` 虚拟环境。
* 需要确认 `.venv` 中的 Python 是否可以正常运行。
* 需要确认 `requirements.txt` 中列出的依赖是否全部安装成功。
* 需要确认 `pandas`、`numpy`、`matplotlib`、`seaborn`、`jupyter`、`sqlalchemy`、`pymysql`、`dotenv` 和 `openpyxl` 是否可以正常导入。
* 需要检查 `.gitignore` 是否包含 `.ipynb_checkpoints/`。
* 需要确认本地项目是否已经连接 GitHub 远程仓库。
* 需要确认首次 Git commit 是否真实创建并成功 push 到 GitHub。
* 需要确认 Git 没有跟踪 `.venv`、`.env`、原始数据或其他敏感文件。

### 后续验证依据

* 终端中 Python 环境测试的输出结果。
* 终端中 Python 包导入测试的输出结果。
* `pip check` 的检查结果。
* `git remote -v` 的输出结果。
* `git log` 的提交记录。
* GitHub 仓库页面中显示的首次提交记录。
* `git status` 和 Git 跟踪文件检查结果。

### 下一步

* 让 Codex 在真实的本地项目目录中验收 Phase 0。
* 根据 Codex 的终端检查结果更新 `phase-checklist.md`。
* 只有在所有必需项目都验证通过后，才正式进入 Phase 1：下载 Olist 数据并理解数据表结构。

## 2026-07-01 — Phase 1：Data Understanding

### 我完成的内容
- 已将 9 张 Olist 原始 CSV 放入 `data/raw/`。
- 已运行 `notebooks/01_data_understanding.ipynb`。
- 已读取全部 9 张表，并检查表规模、字段、缺失值、候选唯一标识、日期范围、订单状态和主要表关系。
- 已确认订单表、订单商品表、支付表和评价表的分析粒度不同。
- 已识别一单多商品、一单多支付记录，以及评价记录可能重复等风险。

### 我理解到的关键知识
- `orders` 是订单粒度；`order_items` 是订单商品明细粒度。
- 订单连接 `order_items` 或 `order_payments` 后，不能直接统计普通订单行数，否则会重复计算。
- 客户复购必须使用 `customer_unique_id`，不能使用 `customer_id`。
- 配送天数和延迟分析只能基于已送达且实际送达时间完整的订单。
- 没有评价不代表客户满意；评分分析只能使用有效评价记录。

## 2026-07-01 — Phase 1 收尾更新

### 本次完成的收尾内容

* 已创建并完成 `docs/data-quality-log.md`，记录了原始数据中的关键质量问题、可能造成的计算风险，以及后续 SQL 和 Python 分析中的处理原则。
* 已使用 `01_data_understanding.ipynb` 的真实检查结果更新 `docs/data-dictionary.md`。
* 数据字典现已覆盖 9 张原始 CSV 对应的核心表，说明了每张表的一行代表什么、关键字段、字段类型、缺失情况、唯一性、连接关系和后续用途。
* 已完成表关系说明。关系说明同时记录在数据字典中，并已通过概念关系图辅助理解。
* 已确认 Phase 1 不进行数据清洗、不删除缺失值、不计算业务指标，也不输出业务结论。

### 本阶段最重要的理解

* `orders`、`order_items`、`order_payments` 和 `order_reviews` 的数据粒度不同，不能在多表连接后直接统计普通行数。
* 订单数量通常应以订单粒度计算；商品金额和品类分析可以保留订单商品明细粒度。
* 客户复购必须使用 `customer_unique_id`，而不是用于连接订单的 `customer_id`。
* 配送天数和延迟分析只能使用已送达且相关时间字段完整的订单。
* 评价表中同一订单可能出现多条评价记录，因此后续评分分析前必须先确定订单级的评价处理规则。
* 数据质量检查不是为了马上删除“异常数据”，而是为了避免后续 SQL 和 Python 分析出现重复计算或错误解释。

### Phase 1 当前状态

Phase 1 的数据理解、数据字典、数据质量日志、Notebook 检查和表关系说明均已完成。

在进入 Phase 2 前，还需要：

1. 更新 `docs/phase-checklist.md`；
2. 完成并推送 Phase 1 的 Git 提交；
3. 确认 Git 没有跟踪 `data/raw/`、`.venv/`、`.env` 或 Notebook 自动检查点文件。

### 下一阶段

Phase 2 将进入 MySQL 建库、创建核心表、导入 CSV，并用 SQL 再次验证行数、连接关系和关键数据质量规则。
