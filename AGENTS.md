# ARIS：网页版 Codex 项目安装

## 项目与技能位置

- 本项目已经位于独立的云任务环境中；复用当前 checkout，除非用户明确要求，否则不要创建 Git worktree。
- Codex 技能来源是 `skills/skills-codex/`；可发现的项目安装位于 `.agents/skills/<技能名>/SKILL.md`，共 87 个技能。使用 Codex 版本的指令。
- `.agents/skills/` 是真实文件副本，无需原安装容器、全局安装、绝对路径链接或 `~/.aris/repo`。复制或提交时必须保留隐藏目录 `.agents/`。
- 共享协议在 `.agents/skills/shared-references/`。技能自己的 `scripts/`、`templates/`、`references/`、`agents/` 等目录随安装保留。
- 部分资源由主技能单独维护：海报脚本/模板及许可证、实验队列脚本、图形渲染器等已补入对应的安装目录，未覆盖 Codex `SKILL.md`。根目录 `templates/` 和 `skills/` 也应随仓库保留。
- 原始源码与安装副本不会自动同步。修改或更新技能时先比较两边并保留本地改动，再同步源目录及主技能补充资源。不要直接运行会产生绝对路径链接的默认安装器覆盖此安装。

## 调用方法与辅助脚本

先查看当前会话的技能目录，再使用其列出的技能名。部分 Codex 宿主会显示 `aris:` 前缀。也可以直接要求“读取 `.agents/skills/research-wiki/SKILL.md` 并执行 init”。仓库中的 `/research-wiki` 等写法是工作流示例，不保证网页版提供同名斜杠菜单。

执行 shell 辅助脚本前，从项目根目录开始；从子目录进入时可以使用：

```bash
cd "$(git rev-parse --show-toplevel)"
export ARIS_REPO="$PWD"
python3 tools/reviewer_route.py --project . --host codex --tier regular
```

`ARIS_REPO` 每次从当前 checkout 推导，不应保存容器绝对路径。公共辅助脚本保留在根目录 `tools/`，遵循 `.agents/skills/shared-references/integration-contract.md` 的查找链：`$ARIS_REPO/tools/` → 当前项目 `tools/` → 宿主支持的其他安装位置。本安装不依赖最后一层，也不使用安装器的绝对路径清单。

代表性入口（先用 `--help` 查看参数）：

```bash
python3 tools/research_wiki.py --help
python3 tools/reviewer_route.py --project . --host codex --tier deep
python3 .agents/skills/render-html/scripts/render_html.py --help
python3 .agents/skills/citation-crosscheck/scripts/cx_verify.py --selftest
python3 .agents/skills/paper-poster-html/scripts/run_gates.py --help
```

单技能资源优先取 `.agents/skills/<技能名>/` 下的对应目录；例如 HTML 模板在 `render-html/scripts/templates/`，论文模板在 `paper-write/templates/`。`cx_verify.py` 位于 `citation-crosscheck/scripts/`，本来就没有根目录 `tools/` shim。

## 默认 Codex reviewer

保留宿主默认路线，不安装 Claude、Gemini、Grok 等外部审稿服务，也不创建外部 reviewer 配置。普通审稿使用 `gpt-6-astra` / `xhigh`，深度审计使用 `ultra`；具体路由和允许的能力回退见 `shared-references/reviewer-routing.md`。

在用户任务和适用技能授权委派时才创建 reviewer。先发现当前宿主实际提供的工具：

- 首次调用：`spawn_agent`；本宿主对应 `collaboration.spawn_agent`。必须显式使用 `fork_turns: "none"`、所需 `model` 和 `reasoning_effort`，只给原始任务与证据路径。
- 同一 reviewer 续聊：技能中的 `send_input` 对应本宿主 `collaboration.followup_task`，保留同一 agent id；`send_message` 本身不保证唤醒空闲代理。
- 等待/检查完成：使用宿主的 `wait_agent`、`list_agents` 和返回的最终消息；创建成功不等于审稿完成。
- 普通副本、历史继承或主代理自评不能替代要求的新上下文审稿。子代理工具/轮数/递归限制在当前宿主并非独立硬隔离，应按协议记录这一限制。
- Codex 主代理与 Codex reviewer 属于同模型家族：记录 `review_independence: same-family` 和 `acceptance_status: provisional`，不得称为跨家族接受。缺少所需工具/模型时报告 `REVIEW_UNAVAILABLE`，不要伪造通过。

模型可选项与工具可用性需要在新会话重新确认。普通档的创建和续聊已做最小探测；深度档 `ultra` 仅检查工具声明及路由，未执行实际深度审稿。

## 依赖与执行范围

公共工具以 Python 3 和 Bash 为主；开发测试依赖 `pytest`、`httpx` 及 `tools/requirements-codex-policy-audit.txt`。需要在新机器重建测试环境时，可用仓库已忽略的目录：

```bash
python3 -m venv .aris/dev-venv
.aris/dev-venv/bin/python -m pip install pytest httpx -r tools/requirements-codex-policy-audit.txt
.aris/dev-venv/bin/python -m pytest tests/ -q
```

这只是新环境的重建方法；当前安装不依赖该目录存在。其他工作流依赖按需检查，见 `docs/CODEX_WEB_INSTALL.md`。不要把“技能已安装”当作网络检索、浏览器渲染、LaTeX 编译、模型推理或 GPU 实验已经验证。安装验证阶段只检查安装和能力；完整研究流水线及 GPU 实验应留给用户后续明确指定的任务。
