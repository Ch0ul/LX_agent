# LX-Agent RTL代码手册vscode终端生成教程

## 说明

本文仅是对LX_agent在vscode中的终端使用说明，若想使用web UI版或需要详细说明，请下载仓库代码后阅读readme文件




## 环境要求

建议使用 Python 3.10 或更高版本。

安装依赖：

```powershell
python -m pip install -r requirements.txt
```

当前依赖：

```text
openai>=1.0.0
python-dotenv>=1.0.0
flask>=3.0.0
```

## 环境变量配置

在项目根目录创建或修改 `.env`。

最小配置：

```env
DEEPSEEK_API_KEY=your_api_key_here
```

推荐配置：

```env
DEEPSEEK_API_KEY=your_api_key_here
DEEPSEEK_BASE_URL=https://api.deepseek.com
DEEPSEEK_MODEL=deepseek-v4-pro
DEEPSEEK_REVIEW_MODEL=deepseek-v4-flash
KNOWLEDGE_IR_SEMANTIC_MODEL=deepseek-v4-flash

RTL_MANUAL_SEMANTIC_WORKERS=4
RTL_MANUAL_PARSER_TIMEOUT=220
RTL_MANUAL_KNOWLEDGE_TIMEOUT=10800
RTL_MANUAL_KNOWLEDGE_WRAPPER_TIMEOUT=10920
```

常用变量：

| 变量 | 默认值 | 说明 |
| --- | ---: | --- |
| `PORT` | `5000` | Flask Web 服务端口。 |
| `DEEPSEEK_API_KEY` | 空 | Web/CLI 默认模型 API key。 |
| `DEEPSEEK_BASE_URL` | `https://api.deepseek.com` | DeepSeek 兼容接口地址。 |
| `DEEPSEEK_MODEL` | `deepseek-v4-pro` | 文档增强默认模型名称。 |
| `DEEPSEEK_REVIEW_MODEL` | `deepseek-v4-flash` | source-review、review 和 compose 后自动审查默认模型名称。 |
| `KNOWLEDGE_IR_SEMANTIC_MODEL` | `deepseek-v4-flash` | Knowledge Semantic Layer 默认模型名称。 |b
| `RTL_MANUAL_PARSER_TIMEOUT` | `220` | Parser Tool 超时时间，单位秒。 |
| `RTL_MANUAL_KNOWLEDGE_TIMEOUT` | `3600` | Knowledge pipeline 内层超时时间，单位秒。 |
| `RTL_MANUAL_KNOWLEDGE_WRAPPER_TIMEOUT` | `knowledge_timeout + 120` | Knowledge Tool 外层 wrapper 超时时间。 |
| `RTL_MANUAL_SEMANTIC_WORKERS` | `4` | Semantic Layer 并发 LLM 请求数。遇到限流可降到 `2` 或 `1`。 |

## 变量配置

只需要修改两个个变量：

``` powershell
$RTL_ROOT = "<RTL工程根目录>"
$TOP_MODULE = "<顶层模块名称>"
```

## 1. 进入智能体目录

``` powershell
cd 项目当前的根目录
```

## 2. 检查TCL文件

``` powershell
Test-Path "$RTL_ROOT\read_rtl_list.tcl"
```

``` powershell
Test-Path "$RTL_ROOT\rtl_top_list.tcl"
```

如果出现false，即不存在，先生成TCL文件。


``` powershell

Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force; & ".\run_rtl_manual.ps1"

```

## 3. 确认 TCL 是否正确

```
Get-Content "$RTL_ROOT\read_rtl_list.tcl" | Select-Object -First 10
```

```
Get-Content "$RTL_ROOT\rtl_top_list.tcl"
```

## 4. Parser

``` powershell
python .\\backend\\skills\\catalog\\rtl-manual-generation\\scripts\\run_parser_tool.py `
--project-root $RTL_ROOT `
--rtl-inputs "." `
--timeout 600
```

成功标志：

Parser Tool execution succeeded


## 5. Build

``` powershell
python -m backend.manual_cli build `
--project-root $RTL_ROOT `
--rtl-inputs "." `
--top-module $TOP_MODULE `
--knowledge-timeout 10800 `
--log-events
```

## 6. Source Review

``` powershell
python -m backend.manual_cli source-review `
--project-root $RTL_ROOT `
--top-module $TOP_MODULE `
--require-llm `
--log-events
```

## 7. AI增强

主手册：

``` powershell
python -m backend.manual_cli enhance `
--project-root $RTL_ROOT `
--top-module $TOP_MODULE `
--main-manual `
--require-llm `
--log-events
```

模块：

``` powershell
python -m backend.manual_cli enhance `
--project-root $RTL_ROOT `
--top-module $TOP_MODULE `
--all-modules `
--module-filter retryable `
--require-llm `
--log-events
```

## 8. 生成最终完整手册

``` powershell
python -m backend.manual_cli compose `
--project-root $RTL_ROOT `
--top-module $TOP_MODULE `
--require-llm `
--log-events
```


## 注意事项

-   PowerShell 多行命令需要保留反引号 \`。
-   粘贴终端时选择"粘贴"，不要选择"粘贴为一行"。
-   RTL路径和TOP_MODULE必须替换为实际工程信息。
