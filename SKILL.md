---
name: a-share-market-watch
description: A-share/A股 market monitoring, 盘中监测, 盘中汇报, 盯盘, 重新预测, 明日预测, 看走势, 看板块, 板块趋势, 短期趋势, 中期趋势, 长期趋势, 5/10/20/60/120/250日趋势, 预测依据, 真实依据, 上涨依据, 下跌依据, 为什么上涨, 为什么下跌, 选股, 股票推荐, 候选股排序, 买入前检查, 能不能买, 可买性检查, 主力资金, 机构资金, 散户资金, 大单流入流出, 资金流, 龙虎榜, 融资融券, 最新新闻, 利好利空, 公告政策, 舆情情绪, and conditional trade-assist predictions for Chinese stocks. Use for themes such as 创新药/CXO/医药, 半导体/芯片/存储/封测, 机器人/人形机器人, AI算力/液冷/数据中心, 游戏/传媒, 贵金属/黄金, 电力/煤炭, 商业航天/军工, 传统消费/白酒/家电, 大金融/银行/券商/保险. Trigger when the user says 现在汇报, 重新分析, 给出预测, 看主力, 看新闻, 哪些值得关注, 哪些可以买, 明天买什么, or asks to rank A股 watchlists while real orders remain manual.
---

# A-Share Market Watch

## Purpose

Use this skill to produce disciplined A-share monitoring reports and conditional predictions. The workflow must combine real-time price action, sector/theme strength, short/medium/long-term sector trends, public fund-flow proxies, latest news or policy, sentiment spread, and buyability checks before ranking stocks. Every directional prediction must include real evidence and the reasoning chain behind why the stock or sector may rise, fall, or stay uncertain.

Do not place, stage, or automate real orders. Assume the user executes trades manually outside Codex.

## Chinese Trigger Coverage

Trigger this skill for Chinese requests like:

- `现在汇报`, `盘中汇报`, `重新预测`, `重新分析`, `明天预测`, `给出预测结果`
- `盯盘`, `监测股票走势`, `看走势`, `看分时`, `看盘中走向`, `十分钟查一次`
- `推荐股票`, `股票推荐`, `哪些值得关注`, `哪些可以买`, `明天买什么`, `候选股排序`
- `看主力`, `主力资金`, `机构资金`, `散户资金`, `大单`, `资金流入流出`, `龙虎榜`, `融资融券`
- `看新闻`, `最新消息`, `市场情绪`, `利好利空`, `公告`, `政策`
- `能不能买`, `可买性`, `创业板权限`, `科创板权限`, `涨停买不进`, `停牌`, `ST`

## Data Rules

- Avoid Eastmoney/Dongfangcaifu endpoints by default unless the user explicitly requests them or confirms working access.
- Prefer verified non-Eastmoney sources:
  - Real-time quotes: Sina batch quote, Tencent batch quote.
  - Trend/history: Tencent daily history, sector/index daily series when available, and recent highs/lows or moving-average context.
  - Sector/theme strength: TongHuaShun industry summary and concept index data.
  - Fund-flow proxies: Tencent tick flow, TongHuaShun big-deal flow, exchange margin data, Sina LHB when relevant.
  - News: company announcements, exchange disclosures, regulator policy, 财联社, 证券时报, 上海证券报, 中国证券报, 科创板日报.
- For latest news, browse current sources and include links. If news cannot be verified quickly, say it is unverified and lower its weight.
- Treat 主力、机构、散户、资金流 as public proxy signals, not confirmed account-level institutional trading.
- If the market is closed or the quote timestamp is stale, state the timestamp clearly and avoid intraday claims.

## Standard Workflow

1. Load local context if available:
   - Prefer `configs/a-share-theme-monitor.json` in the current workspace for themes, watchlists, output fields, and buyability rules.
   - Use user constraints when provided: manual execution, unavailable board permissions, preferred themes, and data-source restrictions.
2. Refresh real-time market state:
   - Indexes: 上证指数, 深证成指, 创业板指, 科创50, 沪深300.
   - Focus themes: 创新药/CXO, 半导体/芯片/存储, 机器人/人形机器人, 游戏传媒, 贵金属/黄金, 电力煤炭, 商业航天/军工, AI算力/液冷, 传统消费, 大金融.
3. Measure sector/theme strength:
   - Rank TongHuaShun industries by 涨跌幅, 总成交额, 净流入, 上涨/下跌家数, 领涨股.
   - Compare user themes against the whole market; do not promote a stock if its theme is weak unless it is explicitly a defensive name.
4. Assess multi-horizon trend before predicting:
   - For every focus sector/theme, check at least recent, medium, and long-term direction when data is available: 5/10/20-day for short-term momentum, 60-day for medium trend, and 120/250-day for long-term trend.
   - Compare current price or sector index against moving averages, recent 20/60/120-day highs and lows, and relative strength versus major indexes.
   - Label the sector as trend-up, rebound-in-downtrend, range-bound, weakening, or trend-down. Do not treat a one-day spike as a durable trend unless it also improves the medium-term structure.
   - If a sector is long-term down but intraday strong, describe it as 情绪反弹 or 趋势修复待确认 rather than a confirmed main trend.
5. Refresh candidate quotes:
   - For every candidate collect price, change percent, high/low, amount, timestamp, intraday position, and minimum cash for 100 shares.
6. Check buyability:
   - Flag 创业板 `300/301`, 科创板 `688/689`, 北交所 `4/8/920` as permission-needed unless the user has confirmed access.
   - Flag ST/*ST, suspension/no quote, near limit-up, very low amount/liquidity, and excessive chase risk.
7. Check fund-flow proxies:
   - Use Tencent tick buy/sell net for a small focus list.
   - Use TongHuaShun big-deal net when available.
   - If tick flow and big-deal flow conflict, call it 分歧 instead of forcing a bullish/bearish label.
8. Check latest news, macro/policy, and sentiment:
   - Identify direct company catalysts, sector news, policy support/headwind, and market liquidity context.
   - Weight direct announcements and official/premium financial news higher than social chatter.
   - Separately identify sentiment source stocks, popularity-board leaders, social-media narratives, and meme/low-price spillover paths; do not confuse sentiment spread with fundamental sector catalysts.
9. Rank and predict:
   - Prefer names where theme strength, multi-horizon trend, price action, flow proxy, news/sentiment, and buyability align.
   - For every upward or downward prediction, print the evidence chain: observed fact, source or timestamp, why it supports the direction, and what contrary evidence would invalidate it.
   - Separate facts from inference. Use labels such as `事实`, `推断`, `反向证据`, and `证据强度`.
   - If evidence is mixed or weak, say `证据不足，暂不预测方向` instead of forcing a bullish or bearish call.
   - Use conditional predictions only: base case, upside trigger, invalidation trigger.
   - Avoid certainty language and avoid "must buy" phrasing.

## Output Contract

Every stock line must include:

- `code` and `name`
- `板块/主题`
- `当前价` and `涨跌幅`
- `板块趋势`: short/medium/long-term sector trend and whether the move is trend continuation, range rebound, or sentiment spike
- `资金流代理`: tick flow, big-deal flow, or unavailable with reason
- `新闻/消息判断`
- `可买性`: normal, permission-needed, blocked, or caution
- `推荐动作`: active focus, wait for pullback, monitor only, avoid chasing, permission blocked
- `预测`: conditional base case
- `真实依据`: concrete evidence for the predicted direction, including data source/timestamp and why it matters
- `反向证据`: what conflicts with the prediction or would make it wrong
- `证据强度`: strong, medium, weak, or insufficient
- `触发条件`: what confirms upside
- `失效条件`: what invalidates the idea

Keep reports compact. Put the strongest actionable names first, then monitor-only names, then blocked or avoid-chasing names.

## Units

Always print units next to numeric money and flow fields. Do not output bare values such as `净流入 6.71`.

- Stock price: `元`.
- Percent change, sector change, and amplitude: `%`.
- Sector `总成交额` and `净流入` from TongHuaShun industry summary: `亿元`.
- Stock quote `成交额`: convert yuan to `亿元` when large enough; otherwise use `万元`.
- Tencent tick buy/sell/net amount: convert yuan to `亿元` or `万元` and label it as `逐笔买盘`, `逐笔卖盘`, or `逐笔净买/净卖`.
- TongHuaShun big-deal amount: label as `万元` unless the source is explicitly converted; use `大单净买/净卖`.
- Exchange margin balance and margin buy amount: label as `元`, `万元`, or `亿元` according to conversion.
- Minimum cash for 100 shares: `元/100股`.

## Recommendation Labels

- `active focus`: theme is strong, stock action is constructive, flow/news do not contradict, and buyability is acceptable.
- `wait for pullback`: stock is strong but already extended, near high, or showing big-deal selling.
- `monitor only`: useful signal but not ready for action because theme, flow, or setup is incomplete.
- `avoid chasing`: excessive intraday rise, weak board confirmation, or risk/reward deteriorated.
- `permission blocked`: needs 创业板/科创板/北交所 access or otherwise cannot likely be bought.

## Practical Defaults

- 10-minute refresh is reasonable during active monitoring; refresh faster only for a very small focus list.
- In weak index regimes, rank defensive strength higher: 创新药, 贵金属, 传统消费, 银行/保险 can outrank high-beta technology.
- In strong risk-on regimes, allow 半导体、AI算力、液冷、机器人 to move up only if sector breadth and flows confirm.
- Do not upgrade a stock only because its sector is strong intraday. If the sector's 60/120/250-day trend is still weak, mark the idea as rebound/repair unless it breaks above key medium-term levels with breadth and volume confirmation.
- Do not write `看涨`, `看跌`, `上涨概率大`, or `下跌风险大` without real evidence in the same line. Directional language must be backed by quote data, trend data, flow proxy, verified news, sentiment evidence, or a clearly stated combination.
- If fund-flow proxies are slow, run them on the top 5-10 candidates only and state the limitation.

## Safety

- Do not interact with brokerage order screens or submit orders.
- Do not ask for passwords, OTPs, or account credentials.
- If the user asks whether to buy, answer with a readiness verdict and conditions, not a command.
