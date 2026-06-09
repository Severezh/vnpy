# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⚠️ 代码修改范围（重要）

本目录（`vnpy/` 核心框架包）属于框架代码。**修改前必须先与用户确认**——默认只在 `vnpy-trader/` 用户工作区改代码。

## 项目概述

VeighNa（原 vn.py）是一个 Python 量化交易框架（v4.3.0），提供事件驱动核心引擎，通过 `vnpy_*` 插件包扩展网关（券商连接）、应用（策略引擎）、数据库和数据源。支持 Python 3.10–3.13（仅 CPython）。

## 常用命令

```bash
# 安装开发环境
pip install ruff mypy uv types-tqdm
uv pip install ta-lib==0.6.4 --index=https://pypi.vnpy.com --system
uv pip install -e ".[alpha,dev]" --system

ruff check .                 # 代码检查
mypy vnpy                    # 类型检查
uv build                     # 构建
pytest tests/test_alpha101.py    # 测试（唯一的 pytest 套件）
python examples/veighna_trader/run.py   # 运行交易客户端（需先配置网关和应用）
```

## 代码规范

- Ruff 规则：B, E, F, UP, W；**忽略 E501**（行长度）
- mypy 严格模式全开（`disallow_untyped_defs`、`warn_return_any`、`strict_optional` 等）
- 用户可见字符串用 `_()` 国际化（gettext），枚举值也需国际化，如 `Direction.LONG = _("多")`

## 核心架构

详见 `../vnpy-trader/.claude/knowledge/10-vnpy-monorepo-architecture.md`，涵盖：
事件驱动引擎（`vnpy/event/engine.py`）、MainEngine 中枢（`vnpy/trader/engine.py`）、插件体系（网关/应用/数据库）、
数据对象（`vnpy/trader/object.py`）、开平转换器（`vnpy/trader/converter.py`）、RPC 多进程（`vnpy/rpc/`）、
Alpha 研究管道（`vnpy/alpha/`）、GUI（`vnpy/trader/ui/`、`vnpy/chart/`）、国际化构建。
