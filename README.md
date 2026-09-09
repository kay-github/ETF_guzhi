# ETF 估值与性价比仪表盘（Kay 个人版）

面向**逆向 + 估值驱动**型 ETF 投资者。把 A 股、港股、美股核心指数的估值区间、回撤、股息率等关键指标聚合到同一视图，帮你快速判断「现在贵还是便宜」。

> 在线地址：https://kay-github.github.io/ETF_guzhi/
> 维护者：Kay（GitHub [@kay-github](https://github.com/kay-github)）
> 数据刷新：每个交易日北京时间 18:30 左右由 GitHub Actions 自动更新

## 这个仪表盘能帮你

- 一眼看到 20 个核心指数的 Value（PE/PB 百分位）与 Pain（近十年回撤）得分，找出真正被低估的板块。
- 展示当前股息率、ROE、估值区间标签，辅助在「便宜但有陷阱」和「便宜有支撑」之间做决策。
- 保留 ETF 代理列表（如 SPY、QQQ、XLV），点开即知道可交易的对标产品。

## 关于这版 UI

原项目为亮色表格风格。本版在 [bryanzhang1024/etf-dashboard-auto](https://github.com/bryanzhang1024/etf-dashboard-auto) 基础上，由 WorkBuddy 重做为**暗色「终端金融」风格**：

- 深墨底（#0A0E14）+ 玻璃质感卡片 + 极淡噪点纹理，久看不刺眼；
- 语义配色：翠绿 = 低估/机会、琥珀 = 警戒、玫红 = 泡沫/风险、蓝 = 回撤；
- 字体：Fraunces（标题）+ Manrope（界面）+ IBM Plex Mono（数字）；
- 新增估值水位仪表、KPI 芯片、评分色条、明暗切换。

数据逻辑与抓取脚本完全沿用原项目，仅重做视觉层。

## 如何使用

### 在线直接查看

1. 打开上方在线地址，默认展示最新一次数据快照。
2. 如遇缓存，用 `Shift + F5` 强刷。
3. 移动端自动切换为评分环卡片视图。

### 本地预览或二次开发

```bash
pip install -r requirements.txt
python scripts/fetch_djeva.py
python scripts/fetch_cn_csindex.py
python scripts/fetch_hk_hsi.py
python scripts/fetch_us_yf.py
python scripts/compute_metrics.py
python scripts/build_assets.py
python -m http.server 8000 --directory docs   # 本地预览
```

运行后结果写入 `docs/assets.csv`，本地访问 `http://localhost:8000` 即可复现线上页面。

> 抓数脚本依赖外网；CI / 内网环境需保证出口可访问中证指数、恒指官网及 Yahoo Finance。

## 数据来源与更新频率

- **中证系**：AKShare 请求中证官网估值与行情接口，持有 15 年历史；接口 403 时可在 `config/indices.yaml` 将 `pe_source/pb_source/dp_source` 暂设 `none`，模型自动降权。
- **恒生系列**：定期抓取恒指官网 Factsheet PDF 中的 PE/PB/股息率，再用 Yahoo Finance 行情补齐时间序列。
- **美股及海外指数**：Yahoo Finance 指数行情 + ETF 代理股息率（SPY、QQQ、XLV 等）计算分位。
- **缺失兜底**：估值缺失不中断任务，记录日志到 `data/raw/*` 并按降权策略保证评分可用。

## 自动更新如何运作（维护者参考）

- 工作流：`.github/workflows/update.yml` 的 `Update ETF dashboard data` 在工作日 UTC 10:30 自动触发，也可手动 `workflow_dispatch`。
- 步骤：Checkout → 装依赖 → 抓估值 `fetch_djeva.py` → 抓行情 → 计算指标 → 生成 `docs/assets.csv` → 自动提交。
- 部署：GitHub Pages 指向 `main` 分支 `/docs` 目录，对外提供 `docs/index.html` 静态页。

## 常见问题

- **日志里「缺少估值数据」？** 先确认 `fetch_djeva.py` 是否成功，再检查网络与 `config/indices.yaml` 的 `djeva_code`。
- **想新增 / 替换指数？** 在 `config/indices.yaml` 补充条目并指定行情/估值源，重跑脚本或等自动任务即可上线。
- **在线与本地结果不同？** 在线依赖 Pages 缓存，强刷或等工作流跑完即同步。
