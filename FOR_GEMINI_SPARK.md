# 致 Gemini Spark —— 数据源与协作情况说明

> 本文件用于告知 Gemini Spark 当前的数据链路、仓库地址、抓取方式与推送规格变更。

---

## 一、这个数据源是什么

A 股每日复盘数据链路，聚焦两条主线：
- **半导体材料设备**（指数 931743.CSI，代表基金 159516.SZ）
- **光模块 / CPO / 通信设备**（指数 931160.CSI）

数据由 **Wind 金融终端**产出，每个交易日收盘后自动生成，推送到 GitHub 公开仓库，供 NotebookLM / Gemini 免鉴权抓取。

## 二、仓库地址（公开，免鉴权）

```
https://github.com/wanyanjunxi-web/AStock-Daily-Review
```

**★ 每天盯这一个链接即可（每天覆盖为最新）：**
```
https://raw.githubusercontent.com/wanyanjunxi-web/AStock-Daily-Review/main/latest.md
```

逐日归档（跨日对比用）：
```
https://raw.githubusercontent.com/wanyanjunxi-web/AStock-Daily-Review/main/<YYYY-MM-DD>/report.md
https://raw.githubusercontent.com/wanyanjunxi-web/AStock-Daily-Review/main/<YYYY-MM-DD>/data.json
```

> 这是**公开仓库**，抓取无需任何授权/Token/Cookie。若用私有仓库，NotebookLM 的无状态爬虫因无凭据会返回 404——之前踩过的坑，已修正。

## 三、当前已归档的数据

| 交易日 | 报告 | 原始数据 |
|--------|------|---------|
| 2026-09-22（一）| ✅ | ✅ |
| 2026-09-23（二）| ✅ | ✅ |
| 2026-09-24（三）| ✅ | ✅ |

**9/25 状态**：Wind 数据源尚未入库 9/25 收盘数据，暂缺，入库后补。

## 四、推送规格（重要变更）

每次推送**只含两类内容**：

1. **纯数据** → `<日期>/data.json`（Wind 原始字段，无删改）
2. **分析文字** → `latest.md` + `<日期>/report.md`（Markdown 日报）

**不再推送**：HTML 看板、图片等非文字内容。请一律以 Markdown 文本为准。

## 五、数据字段说明

**report.md 结构：**
- 一、指数速览：收盘 / 涨跌 / 成交额 / 近6日走势
- 二、异常提示（自动检测）：资金反转 / 龙头背离 / 连续流出
- 三、个股资金明细：主力净流入 / 涨跌 / 成交额
- 四、跟踪基金：159516.SZ 净值
- 五、事件流：最新资讯

**data.json 字段：**
- `date` / `generated_at`
- `indexes[]`: code, name, open, close, high, low, turnover_yi, chg_pct, recent_closes
- `stocks[]`: code, name, sector, tag, flow_yi（主力净流入，亿元）, chg_pct, amount_yi
- `funds[]`: code, name, nav, chg_pct
- `alerts[]`: type, target, detail

## 六、口径与注意事项

- **数据来源**：Wind 金融终端，可回溯
- **主力资金单位**：统一归一化为「亿元」（Wind 原始单位不固定，已换算）
- **异常提示是规则产出**：资金反转/龙头背离/连续流出由工具台规则计算，属“判断”，非 Wind 原始字段；原始数值本身来自 Wind
- **涨跌颜色**：A 股惯例，红涨绿跌
- **免责**：本数据仅供研究参考，不构成个人投资建议

## 七、覆盖标的

- 指数：931743.CSI 半导体材料设备、931160.CSI 通信设备、000001.SH、399006.SZ、000688.SH
- 个股：中际旭创 300308、新易盛 300502、天孚通信 300394、光迅科技 002281、北方华创 002371、中微公司 688012、拓荆科技 688072、华海清科 688120
- 基金：国泰半导体材料设备 ETF 159516.SZ

---

## 建议给 Spark 的提示词

> 数据源：`https://raw.githubusercontent.com/wanyanjunxi-web/AStock-Daily-Review/main/latest.md`（每日覆盖）。
> 请自动读取最新一期，做以下分析：
> 1. 对比最近数个交易日，指出主力资金最坚决撤离/流入的标的，附具体数值证据；
> 2. 半导体设备 vs 光模块，哪个更强/弱，用板块跌幅与龙头资金流支撑；
> 3. 哪些异常信号（资金反转/龙头背离/连续流出）重复出现，说明什么；
> 4. 给出下一步需要监控的信号清单。
