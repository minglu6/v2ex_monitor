# v2ex_monitor

一个用于监控 V2EX 指定节点新帖子与提醒通知的 Python 项目，支持命令行运行、周期性巡检，以及以 MCP 服务方式接入支持 MCP 的 Agent。

## 功能特性

- 监控多个 V2EX 节点的新帖子
- 拉取当前账号的提醒通知
- 基于主题 ID / 提醒 ID（或内容哈希）去重
- 生成 Markdown 格式监控报告
- 提供统一入口脚本，便于作为 skill 使用
- 提供 MCP 服务入口，便于 Agent 调用

## 项目结构

```text
.
├─ README.md
├─ .gitignore
└─ skills/
   ├─ requirements.txt
   ├─ run_skill.py
   ├─ skill.md
   ├─ v2ex-monitor.md
   ├─ v2ex_hourly_report.md
   ├─ v2ex_mcp.py
   ├─ v2ex_monitor.py
   ├─ v2ex_monitor_config.example.json
   └─ v2ex_monitor_data/
```

## 环境要求

- Python 3.9+
- 一个可用的 V2EX API Key

建议先安装依赖：

```bash
pip install -r skills/requirements.txt
```

## 快速开始

### 1. 配置 API Key 与监控节点

可直接复制示例配置文件：

```bash
copy skills\v2ex_monitor_config.example.json skills\v2ex_monitor_config.json
```

或者使用命令配置：

```bash
python skills/run_skill.py config --nodes python,linux,programmer --apikey <你的_api_key>
```

### 2. 执行一次监控

```bash
python skills/run_skill.py run
```

### 3. 查看最近一次报告

```bash
python skills/run_skill.py report
```

## 直接使用核心脚本

如果你不想通过统一入口脚本，也可以直接运行核心监控程序：

```bash
python skills/v2ex_monitor.py config --nodes python,linux,programmer --apikey <你的_api_key>
python skills/v2ex_monitor.py run
python skills/v2ex_monitor.py daemon --interval 1
```

## MCP 用法

项目提供了 MCP 服务入口：

```bash
python skills/v2ex_mcp.py --stdio
```

适合在支持 MCP 的 Agent 环境中使用。相关能力包括：

- 获取节点主题列表
- 获取主题详情
- 获取主题回复
- 获取提醒通知
- 获取个人信息
- 执行监控与配置

## 在 OpenClaw 中使用

如果你是将这个项目作为 OpenClaw 的自定义 skill 使用，推荐直接保留当前仓库结构，并让 OpenClaw 调用 `skills/` 目录中的入口脚本或说明文件。

### 方式一：作为普通 Skill 使用

OpenClaw 可优先读取以下文件：

- `skills/skill.md`：Skill 总说明
- `skills/run_skill.py`：统一命令入口

典型调用方式：

```bash
python skills/run_skill.py config --nodes python,linux,programmer --apikey <你的_api_key>
python skills/run_skill.py run
python skills/run_skill.py report
```

适合以下场景：

- 让 Agent 帮你完成首次配置
- 手动触发一次监控
- 读取最近一次 Markdown 报告

### 方式二：作为 MCP Skill 使用

如果你的 OpenClaw 环境支持 MCP，可直接启动：

```bash
python skills/v2ex_mcp.py --stdio
```

启动后，OpenClaw / Agent 可通过 MCP 调用 V2EX 相关能力，例如：

- `v2ex_get_node_topics`
- `v2ex_get_topic`
- `v2ex_get_topic_replies`
- `v2ex_get_notifications`
- `v2ex_get_my_info`
- `v2ex_get_node_info`
- `v2ex_monitor_topics`
- `v2ex_config`

### OpenClaw 使用建议

- 初次使用时，先配置 `skills/v2ex_monitor_config.json`
- 不要把真实 API Key 写入仓库已跟踪文件
- 建议把 `skills/` 整体作为 skill 资源目录
- 若需要让 Agent 稳定调用，优先使用 `python skills/run_skill.py run`
- 若需要结构化工具调用，再使用 MCP 模式

### 可直接给 OpenClaw 的提示词模板

如果你希望在 OpenClaw 中尽量减少手动操作，可以直接给它下面这段提示词。通常你只需要把其中的 `YOUR_V2EX_API_KEY` 改成你自己的 API Key，OpenClaw 就可以按步骤自动安装依赖、写入本地配置并开始使用。

```text
请把当前仓库作为一个可运行的 V2EX 监控 skill 来配置并使用：

1. 先安装依赖：
   pip install -r skills/requirements.txt

2. 使用下面参数完成配置：
   APIKEY=YOUR_V2EX_API_KEY
   NODES=python,linux,programmer

3. 执行命令：
   python skills/run_skill.py config --nodes python,linux,programmer --apikey YOUR_V2EX_API_KEY

4. 然后运行一次监控：
   python skills/run_skill.py run

5. 最后输出报告内容：
   python skills/run_skill.py report

要求：
- 如果缺少依赖就自动安装
- 不要把 API Key 提交到 git
- 优先复用仓库内现有脚本，不要重新实现功能
```

如果你想让 OpenClaw 通过 MCP 方式接入，也可以使用下面这个简化提示词：

```text
请将当前仓库作为 OpenClaw 的 MCP skill 使用。

- 安装依赖：pip install -r skills/requirements.txt
- 使用 APIKEY=YOUR_V2EX_API_KEY 完成本地配置
- 启动 MCP 服务：python skills/v2ex_mcp.py --stdio
- 不要修改仓库源码；若只缺配置文件则自动从 example 生成
- 不要把真实 API Key 写回 git 跟踪文件
```

## 输出文件

- `skills/v2ex_hourly_report.md`：监控报告
- `skills/v2ex_monitor_config.json`：本地配置文件（已被 `.gitignore` 忽略）
- `skills/v2ex_monitor_data/`：本地运行数据目录（已被 `.gitignore` 忽略）

## 去重逻辑

- 帖子：按主题 ID 去重
- 提醒：优先按提醒 ID 去重；若无 ID，则按提醒内容计算哈希去重

因此在重复执行监控时，不会反复把同一条帖子或提醒统计为新增。

## 注意事项

- 请不要把真实 API Key 提交到仓库
- 本项目已默认忽略本地配置、虚拟环境与运行期数据
- 若在 Windows 下使用 Git，可能会看到 LF/CRLF 换行提示，这通常不影响使用

## 仓库地址

GitHub: https://github.com/minglu6/v2ex_monitor