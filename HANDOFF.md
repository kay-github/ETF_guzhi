# ETF 估值仪表盘 · 项目经验与交接文档

> 用途：供不同 AI / 后续协作者快速接手本项目。涵盖架构、部署、历史决策、坑点与进度。
> 维护者：Kay（GitHub [@kay-github](https://github.com/kay-github)）。最近一次整理：2026-09-09。

## 1. 项目定位（一句话）

把 A 股 / 港股 / 美股 20 个核心指数的估值（PE/PB 百分位）、回撤、股息率、ROE 聚合到同一视图，用 Value / Pain 双评分帮「逆向 + 估值驱动」型 ETF 投资者判断贵贱。纯静态前端 + CSV 驱动，零后端。

## 2. 速查信息

| 项 | 值 |
|---|---|
| 本地仓库 | `D:\work_code2\etf-dashboard-auto` |
| 远程仓库 | https://github.com/kay-github/ETF_guzhi.git |
| 线上地址 | https://kay-github.github.io/ETF_guzhi/ |
| 默认分支 | `main` |
| Pages 源 | `main` 分支 `/docs` 目录 |
| 数据自动更新 | 工作日 UTC 10:30（≈北京 18:30）GitHub Actions |
| 提交署名 | kay <34707899+kay-github@users.noreply.github.com> |
| 原始上游 | https://github.com/bryanzhang1024/etf-dashboard-auto |

## 3. 技术栈

- 前端：单文件 `docs/index.html`（纯 HTML/CSS/JS，无框架、无构建步骤；字体走 Google Fonts CDN）
- 数据：CSV 驱动（`docs/assets.csv`），前端 fetch 后渲染表格 + 卡片
- 抓数：Python 脚本（`scripts/`）+ AKShare / Yahoo Finance / 蛋卷 / 恒指 PDF
- 计算：`compute_metrics.py`（百分位/评分）+ `build_assets.py`（输出 CSV）
- 自动化：GitHub Actions（`.github/workflows/update.yml`）
- 托管：GitHub Pages（免费，无需自有服务器）

## 4. 目录结构

```
etf-dashboard-auto/
├── docs/
│   ├── index.html      # 唯一前端页面（暗色重设计版）
│   └── assets.csv      # 数据源（被 Actions 每日覆盖）
├── scripts/            # 抓数 + 计算脚本
├── config/indices.yaml # 指数清单与数据源配置（扩展入口）
├── data/raw, data/processed  # 中间产物
├── requirements.txt
├── .github/workflows/update.yml  # 自动更新工作流
├── README.md
└── HANDOFF.md          # 本文件
```

## 5. 数据流水线

`fetch_djeva.py`(蛋卷) → `fetch_cn_csindex.py`(中证/AKShare) → `fetch_hk_hsi.py`(恒指 PDF) → `fetch_us_yf.py`(Yahoo) → `compute_metrics.py`(百分位/评分) → `build_assets.py`(写 `docs/assets.csv`) → Actions 自动 `git commit & push`。

前端按 Value（估值便宜度）+ Pain（回撤痛感，逆向用）排序与配色。

## 6. 部署与权限（重要坑点）

本项目已上线，部署链路为：**GitHub Actions 抓数 → 提交 CSV → Pages 托管 /docs**。无需自有服务器。

GitHub PAT 需要两个独立作用域，缺一不可：

- `workflow`：允许**推送 / 修改** `.github/workflows/*.yml` 文件。缺它则整推被拒，GitHub 报错 "refusing to allow a Personal Access Token to create or update workflow ... without workflow scope"。
- `actions: write`：允许通过 **API 触发** `workflow_dispatch`（手动跑一次）。缺它则 `POST /actions/workflows/update.yml/dispatches` 返回 403 "Resource not accessible by personal access token"。注意：cron 定时触发不受此限制，只有 API 手动触发受限。

推送方式：用内联 token 的 URL 一次性 push，**不要**把 token 写进 `.git/config`（安全）：

```bash
git remote set-url origin https://github.com/kay-github/ETF_guzhi.git   # 无 token
git push "https://34707899+kay-github:<TOKEN>@github.com/kay-github/ETF_guzhi.git" main
```

Pages 首次需在仓库 Settings → Pages → Source 选 `Deploy from a branch` → `main` / `/docs`。

## 7. 本地常用操作

```bash
cd D:\work_code2\etf-dashboard-auto
# 本地预览
python -m http.server 8080 --directory docs

# 触发一次数据刷新（需 actions:write 的 token）
curl -X POST -H "Authorization: Bearer <TOKEN>" \
  -d '{"ref":"main"}' \
  https://api.github.com/repos/kay-github/ETF_guzhi/actions/workflows/update.yml/dispatches

# 改完文件后提交并推送
git add -A && git commit -m "..." && git push "https://34707899+kay-github:<TOKEN>@github.com/kay-github/ETF_guzhi.git" main
```

> 与自动任务并发时的注意：工作流会自行 `git commit & push` 更新 `assets.csv`。若你本地也要 push，先 `git pull --rebase` 再 push，避免非快进冲突。

## 8. 已做的设计决策

- **暗色「终端金融」UI 重设计**（2026-09-09，由 WorkBuddy 执行）：原版仅 Tailwind slate 灰阶反转，被判定为「单调 + 亮」。新方案深墨底 #0A0E14 + 玻璃卡 + 噪点；语义色翠绿/琥珀/玫红/蓝；字体 Fraunces + Manrope + IBM Plex Mono；新增估值水位仪表、KPI 芯片、评分色条、明暗切换。仅改视觉层，不动数据逻辑。
- 原 `index.html` 仍在 git 历史中，可随时回退。
- 提交署名已统一为 kay，远程 remote 设为无 token 地址。

## 9. 进度 / 变更日志

- 2026-09-09：克隆原项目；重做暗色 UI；推送到 `kay-github/ETF_guzhi`；开启 Pages；补推工作流。
- 2026-09-09 晚：用户补充 PAT 的 `workflow` + `actions: write` 权限，手动触发首次数据刷新（run 34346573044）。
- 2026-09-09 晚：更新 README 为个人版、页脚加署名、新增本交接文档。

## 10. 后续待办 / 可扩展

- [ ] 增加「近一月涨跌」列，让表格信息更密。
- [ ] 评分列改用环形或更强视觉焦点。
- [ ] 扩展指数：在 `config/indices.yaml` 加条目即可。
- [ ] 如需自定义域名 + 去掉 Pages 中间页，需 ICP 备案（国内节点）。
