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
