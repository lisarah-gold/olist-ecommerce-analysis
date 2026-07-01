# 阶段验收清单

## Phase 0：项目初始化与规则建立

### 一、项目文件与目录结构

* [x] 已创建项目根目录。
* [x] 已保存最新版 v3.0 `project-charter.md`。
* [x] 已创建 `README.md`。
* [x] 已创建 `.gitignore`。
* [x] 已创建 `requirements.txt`。
* [x] 已在项目根目录保存 `AGENTS.md`。
* [x] 已在正确位置创建 `docs/phase-checklist.md`。
* [x] 已在正确位置创建 `docs/learning-log.md`。

### 二、Python 环境

* [x] 已创建 Python 虚拟环境 `.venv`。
* [x] 已成功安装项目依赖。
* [x] 已通过 Python 环境测试。
* [x] `requirements.txt` 中所需的全部 Python 包均可正常导入。
* [x] 已运行 `pip check`，且没有重要依赖冲突。

### 三、Git 与 GitHub

* [x] 已初始化本地 Git 仓库。
* [x] 已连接 GitHub 仓库。
* [x] 已确认 Git remote 配置正确。
* [x] 已创建首次 Git commit。
* [x] 已将首次 commit 成功 push 到 GitHub。

### 四、`.gitignore` 与安全检查

* [x] `.gitignore` 已包含 `.venv/`。
* [x] `.gitignore` 已包含 `.env`。
* [x] `.gitignore` 已包含 `data/raw/`。
* [x] `.gitignore` 已包含 `data/processed/`。
* [x] `.gitignore` 已包含 `__pycache__/`。
* [x] `.gitignore` 已包含 `.ipynb_checkpoints/`。
* [x] `.gitignore` 已包含 `.DS_Store`。
* [x] Git 没有跟踪 `.venv`。
* [x] Git 没有跟踪 `.env`。
* [x] Git 没有跟踪原始数据文件。
* [x] Git 没有跟踪数据库密码、API Key、Token 或其他敏感信息。

### 五、Phase 0 范围确认

* [x] 尚未下载原始数据。
* [x] 尚未创建数据库。
* [x] 尚未导入 CSV 文件。
* [x] 尚未开始 SQL 数据分析。
* [x] 尚未创建 Python Notebook 分析文件。
* [x] 尚未写入未经真实数据验证的业务结论。

## Phase 0 验收结果

* [x] 已由 Codex 完成真实文件与终端环境检查。
* [x] Phase 0 验收通过。
* [y] 可以进入 Phase 1。


# Phase 1：下载数据、理解表结构与建立数据字典

## 一、原始数据
- [x] 已将 9 张原始 CSV 文件放入 `data/raw/`。
- [x] 已确认原始数据未被 Git 跟踪。
- [x] 已确认 CSV 文件名称完整，没有缺失或多余文件。

## 二、Notebook 数据理解检查
- [x] 已运行 `notebooks/01_data_understanding.ipynb`。
- [x] 已检查每张表的行数、列数和字段名。
- [x] 已检查关键字段缺失值。
- [x] 已检查候选唯一标识和组合标识。
- [x] 已检查订单日期范围和订单状态分布。
- [x] 已检查主要表之间的关联覆盖情况。
- [x] 已确认一笔订单可能对应多件商品。
- [x] 已确认一笔订单可能对应多条支付记录。
- [x] 已确认评价表需要进一步处理重复订单评价风险。
- [x] 已确认 geolocation 邮编前缀不是唯一键。

## 三、文档产出
- [ ] 已用真实结果更新 `docs/data-dictionary.md`。
- [ ] 已创建并完成 `docs/data-quality-log.md`。
- [ ] 已更新 `docs/learning-log.md`。
- [ ] 已完成表关系说明。
- [ ] 已完成 Phase 1 Git 提交。

## Phase 1 验收结果
- [ ] Phase 1 验收通过。
- [ ] 可以进入 Phase 2。