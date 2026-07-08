# A-Share Market Watch

Codex skill for China A-share monitoring, watchlist ranking, and conditional trade-assist reports.

It is designed for Chinese-language requests such as A股盯盘, 盘中汇报, 重新预测, 股票推荐, 主力资金, 资金流, 新闻利好利空, 能不能买, 创业板权限, 科创板权限, 半导体, 存储, 机器人, 创新药, AI算力, 液冷, 传统消费, and 大金融.

## What It Does

- Refreshes real-time quotes for A-share candidates.
- Checks sector and theme strength before ranking stocks.
- Checks recent and long-term sector trends before predicting: 5/10/20-day short-term momentum, 60-day medium trend, and 120/250-day long-term context where data is available.
- Uses public fund-flow proxies such as tick flow, big-deal flow, margin data, and LHB when available.
- Reads latest public news, policy, and company announcement context.
- Separates sentiment-driven moves from fundamental/news catalysts, including popularity-board, social-media, and low-price spillover chains.
- Labels buyability issues such as ST, suspension, near limit-up, liquidity, ChiNext, STAR Market, or BSE permissions.
- Produces sector-labeled predictions with trigger and invalidation conditions.
- Includes next-session up/down probability, 3-5 trading-day up/down probability, and predicted upside/downside range.
- Uses TradingAgents as an auxiliary overlay when local logs or runs are available, and still prints `agent_missing` or `agent_error` when coverage is unavailable.
- Applies risk gates for weak theme breadth, intraday fade, weakening trend, chase risk, and TradingAgents coverage before assigning action labels.
- Separates sector direction from entry action so a strong theme can remain a `strong mainline` while an extended leader is labeled `turnover confirmation` or `wait for pullback`.
- Calibrates offensive mainlines against defensive/value sectors before ranking, so stable low-volatility names do not hide a stronger short-term theme.
- Requires every bullish or bearish prediction to include a concrete evidence chain: observed facts, source/timestamp, reasoning, contrary evidence, and evidence strength.
- Adds conditional buy timing and sell/exit timing, while leaving all real execution to the user.
- Creates Markdown/JSON prediction baselines for next-day accuracy reviews when useful, but does not write Excel unless explicitly requested.
- Requires money and flow units such as 元, 万元, 亿元, and 元/100股.

## What It Does Not Do

- It does not place, stage, submit, or automate brokerage orders.
- It does not ask for brokerage passwords, OTPs, account credentials, or private order screens.
- It does not treat public "main force", "institution", or "retail" flow labels as verified account-level trading.
- It does not update Excel workbooks by default.
- It is not investment advice. Use it as a research and monitoring workflow only.

## Data Source Defaults

The skill avoids Eastmoney/Dongfangcaifu endpoints by default because those endpoints are often blocked, proxied, or unstable in some environments.

Preferred public sources include:

- Sina and Tencent for real-time quotes.
- Tencent daily data for trend and liquidity checks.
- TongHuaShun for sector, concept, and big-deal flow proxies.
- Exchange disclosures, company announcements, regulator policy, and public financial news for catalysts.

If a source is unavailable, the report should state the limitation instead of silently relying on stale data.

## Install

Copy this repository folder into your Codex skills directory:

```text
~/.codex/skills/a-share-market-watch
```

On Windows this is usually:

```text
C:\Users\<you>\.codex\skills\a-share-market-watch
```

Restart Codex or refresh the skill list after installation.

## Usage

Example prompt:

```text
Use $a-share-market-watch to refresh my A-share watchlist and give sector-labeled predictions.
```

Chinese examples:

```text
现在汇报，结合实时数据、资金流、新闻和可买性，重新预测哪些A股值得关注。
```

```text
明天买什么？重点看机器人、半导体、存储、创新药、AI算力、液冷、传统消费和大金融。
```

```text
重新分析预测，排除创业板和科创板，列出明日涨跌概率、未来3-5日概率、预测涨跌幅，并保存明天复盘对照底稿。
```

## Validation

This skill was validated with Codex's skill validator:

```text
python ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py ~/.codex/skills/a-share-market-watch
```

On Windows with Chinese text, set UTF-8 mode first:

```powershell
$env:PYTHONUTF8='1'
```
