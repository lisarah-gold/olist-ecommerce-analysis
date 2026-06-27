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
