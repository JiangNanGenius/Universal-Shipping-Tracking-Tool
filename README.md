# 📦 快递查询工具 · Universal Shipping Tracking Tool

> 查询国内外主要快递公司的物流信息，支持中通、圆通、韵达、顺丰、EMS 等数百家快递公司（含大量别名/别称映射）。  
> Query shipment tracking across hundreds of domestic & international carriers (with extensive alias mapping).

- **Author / 作者**: [JiangNanGenius](https://github.com/JiangNanGenius)  
- **Repo / 仓库**: https://github.com/JiangNanGenius/Universal-Shipping-Tracking-Tool  
- **License**: MIT  
- **Version**: 1.0.2  
- **Requires / 依赖**: `requests`, `pydantic`  
- **OpenWebUI**: `required_open_webui_version: 0.4.0`

---

## ✨ Features | 功能亮点

- **Carrier alias resolution**：输入“顺丰/SF/shunfeng”等均可识别；同样覆盖 FedEx/UPS/DHL/USPS/日本邮政/澳邮等。  
- **Formatted markdown result**：自动输出带表格、图标与分节的易读 Markdown。  
- **Raw JSON**：设置 `raw=True` 可返回 API 原始 JSON。  
- **Route & prediction**：支持展示 `routeInfo`、`predictedRoute`（企业版接口返回时）。  
- **Reference side panel (OpenWebUI)**：可将结果推送到引用侧栏，便于查看与留存。  
- **Graceful fallback**：未配置密钥时给出格式化参考示例（可开启 `show_reference` 查看）。

---

## 📦 What this is | 工具简介

该工具封装了 **快递100企业版轮询查询接口**（`https://poll.kuaidi100.com/poll/query.do`），并对公司名/别名做了**大规模映射**与模糊匹配。主入口：

```python
await Tools().delivery_tracking(
    company: str,
    number: str,
    phone: Optional[str] = None,
    from_addr: str = "",
    to_addr: str = "",
    resultv2: int = 1,
    order: str = "desc",
    page: int = 1,
    limit: int = 100,
    raw: bool = False,
    show_reference: Optional[bool] = None,
    __event_emitter__: Any = None,
)
```

> **注意 / Note**  
> - 顺丰等个别公司 **必须** 提供收/寄件人手机号后四位：`phone="1234"`。  
> - 该接口为 **企业版**，需在 `Valves` 中配置 `customer_code` 与 `api_key`。

---

## 🔧 Installation | 安装

```bash
pip install requests pydantic
```

将 `Tools` 类文件放入项目并导入使用；或作为 Open WebUI 的自定义 **Tool** 运行。

---

## ⚙️ Configuration (Valves) | 配置项（阀门）

在 `Tools.Valves` 中设置：

| 字段 | 类型 | 说明 |
|---|---|---|
| `customer_code` | `str` | 快递100 企业版 **授权码** |
| `api_key` | `str` | 快递100 企业版 **密钥** |
| `api_url` | `str` | 查询地址（默认 `https://poll.kuaidi100.com/poll/query.do`） |
| `show_reference` | `bool` | 是否在未配置密钥时返回 **参考输出**（并推送到引用侧栏） |
| `timeout_seconds` | `int` | HTTP 超时时间（默认 12 秒） |

> **安全建议**：不要将密钥直接写入仓库。可使用环境变量或 Open WebUI 的安全存储。

---

## 🚀 Quick Start | 快速开始

### A) 作为 Python 库调用

```python
import asyncio
from your_module import Tools

async def main():
    tool = Tools()
    # 设置 Valves（建议从环境变量读取）
    tool.valves.customer_code = "YOUR_CUSTOMER"
    tool.valves.api_key = "YOUR_API_KEY"

    # 示例：查询顺丰
    md = await tool.delivery_tracking(
        company="顺丰",          # 支持别名：SF/shunfeng/顺丰速运 等
        number="SF1234567890123",
        phone="1234",            # 顺丰等可能必填：手机号后四位
        show_reference=True      # 若无密钥，展示参考输出
    )
    print(md)

asyncio.run(main())
```

### B) 在 Open WebUI 中用作 Tool

1. 将本工具注册为 Open WebUI 的 **Tool**。  
2. 在 Tool 的 **Valves** 面板设置 `customer_code`、`api_key`。  
3. 在对话中直接调用该 Tool，并传入 `company` 与 `number`（必要时传 `phone`）。  
4. 若开启 `show_reference` 且未配置密钥，会返回**格式示例**并推送到引用侧栏。

---

## 🧭 Parameters | 入参说明

| 参数 | 类型 | 说明 |
|---|---|---|
| `company` | `str` | 快递公司名或别名（如“顺丰/中通/ZTO/FedEx/USPS/日本邮政”） |
| `number` | `str` | 运单号 |
| `phone` | `str?` | 手机号后四位（个别承运商必要，如顺丰） |
| `from_addr` / `to_addr` | `str` | 发件地/目的地（可选） |
| `resultv2` | `int` | 行政区解析（`1`=开启，默认） |
| `order` | `str` | 轨迹排序（`asc`/`desc`，默认 `desc`） |
| `page` / `limit` | `int` | 分页（企业版支持） |
| `raw` | `bool` | 为 `True` 时返回 **原始 JSON** |
| `show_reference` | `bool?` | 覆盖 `valves.show_reference` |
| `__event_emitter__` | `Any` | Open WebUI 引用侧栏推送句柄（可选） |

---

## 📤 Output | 返回格式

### Markdown（默认）

- `## 基本信息`：公司、单号、当前状态、签收状态  
- `## 路由信息`：`routeInfo.from/cur/to`（如返回）  
- `## 预计路线`：`predictedRoute`（如返回）  
- `## 物流轨迹`：最近 10 条（可在源码中调整）

### Raw JSON（`raw=True`）

直接返回快递100原始响应，例如关键字段：

```json
{
  "status": "200",
  "message": "ok",
  "state": "5",
  "ischeck": "0",
  "nu": "SF1234567890123",
  "com": "shunfeng",
  "data": [ ... ],            // 轨迹
  "routeInfo": { ... },       // 路由（可选）
  "predictedRoute": [ ... ]   // 预计路线（可选）
}
```

---

## 🧩 Status & Errors | 状态与错误

**State 对照：**

| state | 含义 |
|---|---|
| `0` 在途中 | `1` 已揽收 | `2` 疑难件 | `3` 已签收 |
| `4` 退签 | `5` 派件中 | `6` 退回 | `7` 转投 |
| `8` 清关中 | `10` 待清关 | `11` 清关中 | `12` 已清关 |
| `13` 清关异常 | `14` 拒签 |

**常见错误码（`status != 200`）：**

| code | 含义 |
|---|---|
| `400` 找不到公司或参数不完整 |
| `408` 公司参数异常（如手机号校验失败） |
| `500` 查询无结果（未发货或信息未同步） |
| `501` 服务器内部错误 |
| `502` 服务器繁忙 |
| `503` 验签失败（检查密钥/签名） |
| `601` 授权过期（检查企业版授权） |

---

## 🔎 Company Matching | 公司匹配

- 内置**超大规模映射表**（数百家，含常见别名、缩写与中英文名）。  
- 若无法精确匹配，会基于输入提供 **智能候选**（例如“sf”→“顺丰/顺丰快运/顺丰冷链”等）。  
- 你也可直接传入快递100的**公司编码**（如 `shunfeng`、`zhongtong`）。

---

## 🧪 Examples | 示例

**顺丰（含手机号后四位）：**
```python
await tool.delivery_tracking("顺丰", "SF1234567890123", phone="1234")
```

**FedEx（国际）：**
```python
await tool.delivery_tracking("FedEx", "123456789012")
```

**返回原始 JSON：**
```python
data = await tool.delivery_tracking("中通", "7510******", raw=True)
```

**展示参考输出（无密钥调试）：**
```python
tool.valves.show_reference = True
md = await tool.delivery_tracking("顺丰", "SF123...", phone="1234", show_reference=True)
```

---

## 🧰 Development Notes | 开发者说明

- **签名计算**：`sign = MD5( param_json + api_key + customer_code ).upper()`  
- **轮询接口**：`api_url` 默认指向快递100企业版轮询查询地址。  
- **Reference（Citations）**：若传入 `__event_emitter__`，工具会把**最终 Markdown**推送到引用侧栏，元数据包含公司、单号、状态等，便于检索与保存。  
- **输出格式函数**：`format_tracking`、`format_route_info`、`format_predicted` 可按需修改。  
- **超时**：默认 12 秒，可在 `Valves.timeout_seconds` 调整。

---

## 🔐 Security | 安全

- 切勿将 `customer_code` 与 `api_key` 提交到公共仓库。  
- 建议使用环境变量、密钥管理或 OpenWebUI 的安全参数存储。  
- 生产环境中请开启 HTTPS 与最小权限控制。

---

## ⚠️ Limits & Notes | 限制与说明

- 本工具依赖 **快递100企业版**；若无授权密钥，仅提供**格式示例**用于演示输出。  
- 某些承运商（如顺丰）**必须**提供 `phone`（后四位）才能获取完整轨迹。  
- 轨迹同步存在延迟；若返回“查询无结果”，可能是包裹尚未揽收或数据未同步。

---

## 🤝 Contributing | 参与贡献

欢迎 PR / Issue：新增承运商别名、改进输出格式、优化错误提示等。  
Welcome contributions—PRs for carrier aliases, formatting tweaks, error handling, docs, etc.

---

## 📝 License | 许可证

**MIT** — 详见仓库内 `LICENSE`。  
MIT — see `LICENSE` file for details.

---

# (EN) Universal Shipping Tracking Tool

> Query tracking info from hundreds of carriers worldwide (domestic CN carriers included), with **robust alias mapping** and **clean Markdown output**.

### Why this tool?

- **Carrier aliases**: “SF”, “顺丰”, “shunfeng” → all recognized. Same for ZTO, YTO, Yunda, EMS, FedEx, UPS, DHL, USPS, Japan Post, AUS Post, etc.  
- **Readable Markdown**: sections, icons, tables.  
- **Raw JSON mode**: set `raw=True` to get the original API payload.  
- **Route & ETA**: shows `routeInfo` / `predictedRoute` when available (enterprise API).  
- **OpenWebUI citations**: push rendered Markdown to the reference side panel.  
- **Graceful fallback**: without API keys, returns a reference example (opt‑in via `show_reference`).

### Requirements

```
requests
pydantic
```

### Configure (Valves)

- `customer_code` – Kuaidi100 enterprise **customer**  
- `api_key` – Kuaidi100 enterprise **API key**  
- `api_url` – defaults to `https://poll.kuaidi100.com/poll/query.do`  
- `show_reference` – show example output when no keys are configured  
- `timeout_seconds` – HTTP timeout (default 12s)

> **Note**: Some carriers (e.g., **SF Express**) require **last 4 digits of phone** via `phone="1234"`.

### Usage (Python)

```python
tool = Tools()
tool.valves.customer_code = "YOUR_CUSTOMER"
tool.valves.api_key = "YOUR_API_KEY"

md = await tool.delivery_tracking(
    company="SF", number="SF1234567890123", phone="1234"
)
print(md)
```

**Raw JSON**:

```python
data = await tool.delivery_tracking("ZTO", "7510******", raw=True)
```

**Reference example (no keys)**:

```python
tool.valves.show_reference = True
md = await tool.delivery_tracking("SF", "SF123...", phone="1234", show_reference=True)
```

### Output

- **Markdown**: Basic info, route, predicted route, latest 10 trace items.  
- **JSON**: Kuaidi100 native fields (`status`, `message`, `state`, `ischeck`, `nu`, `com`, `data`, `routeInfo`, `predictedRoute`).

**State mapping**:  
`0 in transit, 1 picked up, 2 exception, 3 delivered, 4 returned, 5 out for delivery, 6 returned to sender, 7 forwarded, 8 customs, 10 pending customs, 11 in customs, 12 released, 13 customs exception, 14 refused`.

**Errors** (non-`200`):  
`400` company/params invalid, `408` validation error (e.g., phone), `500` no result yet, `501` server error, `502` busy, `503` signature failed, `601` authorization expired.

### Dev Notes

- **Signature**: `MD5( param_json + api_key + customer_code ).upper()`  
- **OpenWebUI citations**: pass `__event_emitter__` to push a rendered Markdown document with useful metadata.  
- **Formatting helpers**: `format_tracking`, `format_route_info`, `format_predicted`.

### Security

- Don’t commit credentials; use env vars or OpenWebUI secrets.  
- Prefer HTTPS and least-privilege.

### Limitations

- Requires **Kuaidi100 Enterprise** credentials; otherwise you’ll see a demo output.  
- Some carriers **require `phone`** (last four digits) to fetch full tracking details.  
- Tracking updates may lag.

### License

**MIT**.
