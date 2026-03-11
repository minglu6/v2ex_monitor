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