# 网页版 Codex 项目级安装记录

检查日期：2026-10-26。技能源版本：`18bceb3c71c638bef58096274e1c9a6d323dbc0a`。

## 安装结果

- 安装位置：`.agents/skills/`，87 个 Codex 技能，32 个共享参考文件。
- 采用真实文件复制，无符号链接。源包的 164 个有效文件逐字节一致；另补齐 36 个主技能资源文件，总计 200 个安装文件（不计 Python 缓存）。
- 补充资源来自同一仓库的 `skills/experiment-queue/`、`skills/figure-spec/`、`skills/paper-poster-html/`、`skills/paper-write/` 和 `skills/proof-orchestrator/`，包括脚本、模板、说明与许可证。Codex 技能指令未替换为其他平台版本。
- 根目录 `tools/`、`templates/`、`skills/` 保持原样。原本没有根目录 `AGENTS.md`，现已创建，记录技能位置、调用方法、动态路径解析及 reviewer 规则。
- 未安装 reviewer overlay，未创建外部服务配置，未修改全局 Codex/Claude 配置，也未生成依赖容器路径的安装清单。

## 已执行的验证

| 检查 | 结果 |
| --- | --- |
| 87 个 `SKILL.md` 与 Codex 源包一致 | 通过 |
| 安装包文件及符号链接检查 | 200 个文件；0 个符号链接 |
| Codex CLI 0.163.0 技能目录发现 | 临时配置下可发现全部 87 个技能；部分名称有 `aris:` 前缀 |
| 换路径可移植性 | 复制到含空格的临时目录后，共享辅助脚本、海报入口、离线 HTML 实际渲染均通过 |
| 默认 reviewer 路由解析 | 普通档 `codex / native / gpt-6-astra / xhigh`；深度档 `ultra` |
| 原生子代理生命周期 | 新上下文 `gpt-6-astra / xhigh` 创建及同一代理续聊均成功；仅返回探测文本 |
| 引用核对脚本自测 | `cx_verify.py --selftest`：121/121 通过 |
| 同步最新上游后的完整测试集 | 1156 项通过、43 项按平台/凭据/expect 条件跳过，另有 48 个子测试通过；0 失败 |
| 技能库存检查 | 通过 |

Git 空白检查发现 24 个安装文件沿用上游的行尾空白或文件末尾空行；已确认与源文件逐字节一致，为保留源包未做格式改写。

原生子代理探测证明本次宿主接受所请求的模型/普通档参数并支持续聊；返回消息没有独立服务端模型身份凭证。`ultra` 已在工具声明和路由中确认，未实际调用。未进行研究审稿、完整流水线或 GPU 实验。

目录发现检查复用了仓库的 `tools/probe_codex_invocation.py` 调用方法，并在临时 Codex 配置中建立项目信任；没有更改真实用户配置。直接使用受管默认配置目录时 CLI 报只读错误，因此这项检查不代表网页版当前聊天已经热加载技能。新建/重开该项目的 Codex 会话，并以它实际列出的技能目录为准。

公共辅助脚本查找链可在仓库根目录解析。文本扫描中看似缺失的 `tools/cx_verify.py` 是文档明确说明“不提供 shim”的路径，其实际文件随技能安装；`tools/threat_patterns.py` 是上游项目的来源署名，不是 ARIS 缺失依赖。

## 当前能力与暂时受限的工作流

| 能力 / 工作流 | 当前状态与限制 |
| --- | --- |
| 本地 Wiki、离线 HTML、已有材料的写作/分析 | 所需文件可定位；离线 HTML 已做实际渲染。每个正式任务仍需自己的输入和验证。 |
| 默认 Codex 审稿 | 原生创建与续聊可用；同家族结果只能标为 `same-family / provisional`，不提供跨家族接受。当前宿主未提供独立的子代理工具白名单、递归禁用或轮数硬上限。 |
| 在线文献检索、在线引用核对、研究流水线的联网阶段 | 当前未暴露通用 WebSearch、Zotero、Obsidian 工具。此前检查的启动网络白名单未包含 `arxiv.org`、`export.arxiv.org`、`api.semanticscholar.org`、`api.openalex.org`、`api.exa.ai`、`alphaxiv.org`；本次没有改网络策略或发起这些请求。启用所选来源前需要检查当前策略并按需添加域名。 |
| 新增 Keenable / Firecrawl | 已随最新源码安装，辅助脚本位于根目录 `tools/`，使用 Python 标准库。Keenable 支持无 key 模式，Firecrawl 无 key 模式有每日限额；`api.keenable.ai` 和 `api.firecrawl.dev` 的实际访问未验证，本次不发起检索或配置凭据。 |
| OpenAlex / Exa / DeepXiv | 默认 Python 缺少 OpenAlex 的 `requests`；未配置 Exa key；`deepxiv` CLI 缺失。其他来源的可选 key 不应当作所有检索的强制要求。 |
| HTML 海报、需要默认 Playwright 浏览器的渲染 | Playwright 包可导入，但默认捆绑的 Chromium headless shell 缺失，海报脚本直接使用默认 `chromium.launch()`，当前不能完成默认渲染步骤。系统 Chromium 已验证可用，显式 `executable_path` 的本地页面渲染通过；这不代表原海报脚本已修复。 |
| LaTeX 论文/幻灯片 | `latexmk`、`pdflatex`、`xelatex`、`pdfinfo`、`pdftoppm`、`pandoc` 均存在；未编译具体稿件，模板相关 TeX 包和完整产出未验证。 |
| Lean 形式化 | 缺少 `lean` 和 `lake`，`lean-formalize` 暂不能执行正式检查。 |
| GPU / 云实验 | 未发现 `nvidia-smi`，缺少 Modal、Vast、启智相关 CLI/配置；本次未验证或运行任何 GPU 工作负载。需要这些后端的实验阶段尚不可运行。 |
| 外部模型、飞书、Overleaf 等集成 | 未配置或验证。Claude/Gemini 等外部 reviewer 按要求暂不启用；专用 LLM/MiniMax 工作流仍需各自依赖与授权。 |

Python 测试环境已在本云实例验证，但它不是仓库的可移植文件。新机器可以按 `AGENTS.md` 的命令在忽略目录 `.aris/dev-venv` 重建。平台提供的浏览器、TeX 等系统工具也不会随 Git 文件自动安装。

## 携带与更新

将 `.agents/skills/`、`AGENTS.md` 和本说明随整个仓库保存；保留根目录 `tools/`、`templates/` 和 `skills/`。不依赖原机器路径；也不要只复制 `SKILL.md` 而漏掉其脚本、模板或共享参考。

安装文件应随 Git 提交保存到后续任务使用的分支。另一个 checkout 必须包含此安装提交，或完整复制包含新增文件的工作区。云环境发布快照与 Git 提交是不同操作。

更新时先比较安装副本与源包，保存本地修改，再同步 `skills/skills-codex/` 和上述补充资源；不要用默认绝对链接安装器覆盖真实文件安装。搬移验证不等于所有研究工作流已经端到端运行成功。
