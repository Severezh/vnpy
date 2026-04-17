# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

VeighNa（原 vn.py）是一个 Python 量化交易框架（v4.3.0），提供事件驱动核心引擎，通过 `vnpy_*` 插件包扩展网关（券商连接）、应用（策略引擎）、数据库和数据源。支持 Python 3.10–3.13（仅 CPython）。

## 常用命令

```bash
# 安装开发环境
pip install ruff mypy uv types-tqdm
uv pip install ta-lib==0.6.4 --index=https://pypi.vnpy.com --system
uv pip install -e ".[alpha,dev]" --system

# 代码检查
ruff check .

# 类型检查
mypy vnpy

# 构建
uv build

# 测试
pytest tests/test_alpha101.py

# 运行交易客户端（需要先配置网关和应用）
python examples/veighna_trader/run.py
```

## 代码规范

- Lint 规则：`B`（bugbear）、`E`（pycodestyle）、`F`（pyflakes）、`UP`（pyupgrade）、`W`（warnings），**E501（行长度）被忽略**
- mypy 开启严格模式：`disallow_untyped_defs`、`warn_return_any`、`strict_optional` 等全部启用
- 所有用户可见字符串使用 `_()` 进行国际化包装（gettext）
- 枚举值也需国际化，如 `Direction.LONG = _("多")`

## 核心架构

### 事件驱动引擎 (`vnpy/event/engine.py`)
`EventEngine` 基于 `Queue` + 工作线程 + 定时器线程（1秒心跳），是整个系统的消息总线。事件类型包括 `eTick`、`eOrder`、`eTrade`、`ePosition`、`eAccount`、`eContract` 等。

### MainEngine 中枢 (`vnpy/trader/engine.py`)
所有组件通过 `MainEngine` 连接：添加网关（`add_gateway`）、添加应用（`add_app`）、连接券商（`connect`）、订阅行情（`subscribe`）、发送订单（`send_order`）。内嵌 `OmsEngine`（订单管理）、`LogEngine`、`EmailEngine`。

### 插件体系
- **网关**：继承 `BaseGateway`（`vnpy/trader/gateway.py`），由 `vnpy_ctp`、`vnpy_ib` 等独立包提供
- **应用**：继承 `BaseApp`（`vnpy/trader/app.py`），由 `vnpy_ctastrategy` 等独立包提供
- **数据库/数据源**：继承 `BaseDatabase`/`BaseDatafeed`，通过 `vt_setting.json` 中的名称动态加载

### 数据对象 (`vnpy/trader/object.py`)
全部为 frozen `@dataclass`，带 `gateway_name` 字段和自动计算的 `vt_symbol`（格式：`symbol.EXCHANGE`）。核心对象：`TickData`、`BarData`、`OrderData`、`TradeData`、`PositionData`、`ContractData`。

### 开平转换器 (`vnpy/trader/converter.py`)
处理中国期货市场的开/平/平今/平昨偏移量逻辑，自动跟踪多空持仓并拆分订单。

### RPC 多进程 (`vnpy/rpc/`)
基于 ZeroMQ 的 REQ/REP + PUB/SUB 模式，允许跨进程暴露 `MainEngine` API。`RpcClient.__getattr__` 使用 `@lru_cache` 惰性生成代理方法。

### Alpha 研究管道 (`vnpy/alpha/`)
独立的 ML 因子研究流水线：`AlphaLab`（Parquet 数据湖）→ `AlphaDataset`（Polars 特征工程，支持 101 Formulaic Alphas 语法）→ `AlphaModel`（LightGBM/PyTorch）→ `AlphaStrategy`（组合回测）。可选依赖：`polars`、`lightgbm`、`torch`。

### GUI (`vnpy/trader/ui/`, `vnpy/chart/`)
PySide6（Qt6）+ qdarkstyle 深色主题。`ChartWidget` 基于 pyqtgraph 渲染实时 K 线图，通过 `BarManager` 实现高效数据窗口管理。

### 国际化构建
`.po` 文件在构建时通过 `vnpy/trader/locale/build_hook.py` 编译为 `.mo` 文件（Babel）。
