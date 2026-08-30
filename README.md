# ✈️ 机票捕手 · Deal Tracker

> 北京/天津出发的特价机票监控 · 免费部署在 GitHub Pages · 数据每日自动更新

**🛫 打开就用：https://wsj414855138-code.github.io/deal-tracker/**

对标 [Going](https://going.com) 的低价机票发现引擎（自用版）：盯住北京（PEK/PKX）、天津（TSN）出发的机票，目前覆盖澳大利亚 9 城，订阅后可到全球任意目的地。

## 功能

- **低价雷达**：同出发地+目的地聚合为一条路线（「天津 → 珀斯 ¥2573 起 · 停留 3/6/13 天 · 中转 1 次 · 酷航」），支持按目的地/月份/「只看跌破心理价位」筛选
- **订阅**：存在你自己的浏览器里（localStorage），「订阅命中」按你设的心理价位实时计算命中与标红
- **配额透明**：页面常驻本月 SerpApi 用量与剩余次数（免费档 250 次/月，自设硬上限 230）
- **完整报告**：单文件 HTML（[report/latest.html](https://wsj414855138-code.github.io/deal-tracker/report/latest.html)），手机可看
- **自动更新**：GitHub Actions 每日 20:47（北京时间）快扫、周六 03:37 深扫

## 工作原理（GitHub 全家桶，零成本）

```
GitHub Actions 定时扫描（SerpApi Google Flights 真实票价，CNY）
  → 数据 JSON 提交到 pages-site 分支
  → GitHub Pages 自动重建
  → 打开网页即是最新数据
```

- 仓库即数据库：配额账本在 `site/api/budget.json` 中随扫描接力（Actions 环境每次都是干净的）
- 订阅只存在你的浏览器本地，不经过任何服务器
- 站点为纯静态文件，无后端、无登录、无追踪

## 数据说明（诚实条款）

- 所有价格均为**参考价**（Google Flights 探索模式聚合价），含税状态未知，购买前请在航司/OTA 复核
- 空结果不代表无票（标记为未知，不臆断）；金额只比较 CNY
- 密钥只存在于 GitHub Secrets 与本地 `.env`，仓库内无任何密钥

## 目录

- `site/` — GitHub Pages 静态站（数据 JSON 由扫描工作流提交刷新）
- `src/flight_monitor/` — 核心库：扫描/低价判定/配额账本/存储/报告/FastAPI
- `web/` — 本地 API 版前端（配合同目录 FastAPI 使用，见下）
- `miniprogram/` — 微信小程序版（本地 API 版）
- `scripts/` — 定时入口、静态发布、密钥扫描、远期探针等
- `docs/` — 调研、方案、交接与执行日志

## 本地开发（API 完整版）

```bash
python3 -m venv .venv
.venv/bin/pip install tomli pytest httpx fastapi uvicorn
export PYTHONPATH=src
export SERPAPI_API_KEY=你的key   # 或写入 .env（已 gitignore）

python -m pytest -q                                        # 98 个测试
python scripts/run_scheduled.py                            # 真实扫描（消耗配额）
./scripts/start_local.sh                                   # 本地站 http://127.0.0.1:8765/
```

本地版比 Pages 版多出：服务端订阅管理、`POST /api/scan` 手动扫描（支持 `SCAN_ADMIN_TOKEN` 保护）、`GET /api/deals?sub=` 订阅筛选等 API。

## 路线图

- [x] M1-M4：后端 API / 网页 / 小程序 / Server酱通知（本地版）
- [x] GitHub Pages 自用版上线（espanol 模式：Actions 产数、仓库存数、Pages 展示）
- [ ] 定时扫描工作流推上 GitHub（待 `gh auth refresh -s workflow` 授权后推送）
- [ ] 产品框架讨论 → 多端形态（正式网页版 / 邮箱 / H5 / 小程序）与订阅制
