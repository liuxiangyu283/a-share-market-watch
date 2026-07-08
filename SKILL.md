---
name: a-share-market-watch
description: A-share/A股 market monitoring, 盘中监测, 盘中汇报, 盯盘, 重新预测, 明日预测, 看走势, 看板块, 板块趋势, 短期趋势, 中期趋势, 长期趋势, 5/10/20/60/120/250日趋势, 预测依据, 真实依据, 上涨依据, 下跌依据, 为什么上涨, 为什么下跌, 买入时机, 卖出时机, 买点, 卖点, 退出条件, 止盈止损, 选股, 股票推荐, 候选股排序, 买入前检查, 能不能买, 可买性检查, 明日开盘可以买什么, 一周内涨跌预测, 预测上涨/下跌概率, 预测涨跌幅, TradingAgents辅助分析, 主力资金, 机构资金, 散户资金, 大单流入流出, 资金流, 龙虎榜, 融资融券, 最新新闻, 利好利空, 公告政策, 舆情情绪, prediction-vs-actual review, and conditional trade-assist predictions for Chinese stocks. Use for fixed focus themes such as 创新药/CXO/医药, 半导体/芯片/存储/封测, 机器人/人形机器人, AI算力/液冷/数据中心, 游戏/传媒, 贵金属/黄金, 电力/煤炭, 商业航天/军工, 传统消费/白酒/家电, 大金融/银行/券商/保险, 养殖/农业育种. Trigger when the user says 现在汇报, 重新分析, 给出预测, 看主力, 看新闻, 哪些值得关注, 哪些可以买, 明天买什么, 复盘准确率, or asks to rank A股 watchlists while real orders remain manual.
---

# A-Share Market Watch

## Purpose

Use this skill to produce disciplined A-share monitoring reports and conditional predictions. The workflow must combine real-time price action, sector/theme strength, short/medium/long-term sector trends, public fund-flow proxies, latest news or policy, sentiment spread, and buyability checks before ranking stocks. Every directional prediction must include real evidence and the reasoning chain behind why the stock or sector may rise, fall, or stay uncertain.

Do not place, stage, or automate real orders. Assume the user executes trades manually outside Codex.

## Current User Defaults

Apply these defaults unless the user explicitly changes scope in the current turn:

- Use the fixed focus themes and watchlists from `configs/a-share-theme-monitor.json`; do not drift into a broad whole-market scan.
- Exclude 创业板 `300/301`, 科创板 `688/689`, and 北交所 `4/8/920` candidates from buy lists when the user says not to consider them or has not re-enabled them.
- Do not write Excel workbooks unless the user explicitly asks for Excel in the current turn. Markdown/JSON prediction baselines are acceptable when the user wants next-day accuracy review.
- Always include `TradingAgents` as an auxiliary field for every candidate. Use local logs such as `analysis_output/tradingagents` when available; if unavailable, print `agent_missing` or `agent_error` instead of omitting the field.
- Keep the answer actionable. When the user asks "哪些可以买" or "明天买什么", name the best buyable candidates first, then state the entry setup and invalidation; do not answer as a generic research report.
- If a stock has continuation probability but poor entry quality, say so directly: e.g. `有继续涨可能，但不追高/等换手`.
- Always separate `板块方向评级` from `买入动作`. A strong theme can remain `强主线/看继续走强` while the individual stock action is `不追高/等换手`.

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
7. Add TradingAgents overlay:
   - Try to load or run the local TradingAgents analysis for the focused names. If full live runs are too slow, load the latest local logs and mark coverage.
   - Include `agent_available`, `agent_missing`, or `agent_error` for every stock line.
   - When available, include rating, trade date, entry, stop, and target/trigger from TradingAgents.
   - Treat TradingAgents as an overlay, not the sole decision source: live quote, board strength, trend, flow, news, and buyability still decide ranking.
   - Penalize missing/error coverage in scoring, but do not discard an otherwise strong setup solely because the agent log is missing.
8. Check fund-flow proxies:
   - Use Tencent tick buy/sell net for a small focus list.
   - Use TongHuaShun big-deal net when available.
   - If tick flow and big-deal flow conflict, call it 分歧 instead of forcing a bullish/bearish label.
9. Check latest news, macro/policy, and sentiment:
   - Identify direct company catalysts, sector news, policy support/headwind, and market liquidity context.
   - Weight direct announcements and official/premium financial news higher than social chatter.
   - Separately identify sentiment source stocks, popularity-board leaders, social-media narratives, and meme/low-price spillover paths; do not confuse sentiment spread with fundamental sector catalysts.
10. Rank and predict:
   - Prefer names where theme strength, multi-horizon trend, price action, flow proxy, news/sentiment, and buyability align.
   - For every upward or downward prediction, print the evidence chain: observed fact, source or timestamp, why it supports the direction, and what contrary evidence would invalidate it.
   - Separate facts from inference. Use labels such as `事实`, `推断`, `反向证据`, and `证据强度`.
   - If evidence is mixed or weak, say `证据不足，暂不预测方向` instead of forcing a bullish or bearish call.
   - Give conditional timing only: recommended buy timing should be an entry setup such as pullback support, breakout confirmation, board confirmation, or sentiment continuation. Recommended sell/exit timing should include profit-taking, trend break, invalidation, or sentiment fade conditions.
   - Use conditional predictions only: base case, upside trigger, invalidation trigger.
   - Avoid certainty language and avoid "must buy" phrasing.

## Latest Scoring Gates

Use these risk gates when converting the evidence stack into buyability and probabilities:

- Weak theme breadth gate: if a theme has negative average change with breadth below 50%, or breadth below 34%, mark `theme_weak`, cap next-session upside probability at `57%`, and force `monitor only` unless the user explicitly asks for speculative observation.
- Intraday fade gate: if a stock is up at least `4%` but closes in the lower 35% of its intraday range, or pulls back at least `2.5%` from the intraday high, mark `intraday_fade_risk`, lower the entry-action quality, and avoid chase labels. Do not downgrade the whole theme direction only because an individual leader is extended.
- Weakening trend gate: if trend label is `weakening` and the stock is flat/down, cap upside probability at `52%`.
- Chase-risk gate: if current-day gain is at least `7%`, or price is near the day high with gain at least `4%`, use `avoid chasing` unless there is a confirmed limit-up/leader continuation setup.
- TradingAgents adjustment: `agent_missing` or `agent_error` should lower confidence, but it must not by itself remove a stock from a confirmed strong-mainline buy pool. `Overweight/Buy` can add confidence; `Hold/Neutral` lowers confidence slightly; `Underweight/Sell` materially lowers confidence.
- Probability range: keep routine short-horizon upside probability inside a realistic band unless evidence is exceptional. Do not inflate percentages just because the user asks for direct recommendations.
- Separate probability from action: a stock can have high continuation probability but still be `avoid chasing` if entry quality is poor.

## Direction-vs-Entry Decision Rules

Use a two-layer decision model in every prediction:

1. `板块方向评级`: judge whether the theme itself is `强主线`, `偏强可参与`, `中性轮动`, `弱修复`, or `回避`.
2. `买入动作`: judge how to participate: `可买`, `回踩低吸`, `换手确认`, `只观察`, or `不参与`.

Do not let an entry-risk label erase a strong theme call. For example, AI算力/液冷 can be `强主线` even when 浪潮信息 is `不追高/等换手`.

Upgrade a theme to `强主线` when most of these conditions align:

- Theme ranks top 1-2 by average change or relative strength among the fixed focus themes.
- Theme breadth is at least 65%, or there are at least three liquid leaders rising together.
- Total成交额 is meaningfully above other focus themes.
- At least one leader has a direct catalyst such as earnings, order-book/news catalyst, policy/news confirmation, or clear sentiment leadership.
- The move is confirmed by multiple constituents rather than one isolated涨停.

For a `强主线` theme:

- Put its leading buyable names ahead of defensive/steady sectors in the offensive ranking.
- Use `主线可买池` or equivalent wording when the user asks what can be bought.
- If a leader is extended, change the action to `换手确认` or `回踩低吸`; do not reduce the sector call to `观察`.
- Keep directional probability for leading names at or above the low-60% area unless there is intraday collapse, major negative news, broad-market break, or confirmed资金流 reversal.

## Cross-Sector Calibration

Before final ranking, compare all focus sectors on the same scale:

- Theme strength: average change, breadth, total amount, leader count, and whether leaders are liquid.
- Trend quality: 5/20-day momentum plus 60/120/250-day structure when available.
- Catalyst quality: verified company/sector news, policy, earnings, or sentiment source stock.
- Entry quality: whether the best candidates are still buyable or only chase-risk leaders.
- Risk state: weak breadth, intraday fade, trend weakening, permission block, or stale data.

Use separate buckets in the final answer:

- `进攻主线`: strongest short-term theme candidates. These should not be hidden behind defensive sectors.
- `稳健低吸`: lower-volatility names with better entry but lower upside.
- `观察/回避`: weak breadth, poor trend, missing catalyst, or bad entry quality.

If another sector is judged better than the user-mentioned sector, explicitly say why using the same fields. If confidence is only medium or low, mark it. Do not imply other sector calls are automatically correct just because one theme call was fixed.

## Calibrated Probability Model

Use the point-score model only as a raw signal. Treat its output as `raw_score`, not a fully reliable statistical probability.

When data is available, improve probability estimates with a calibrated ensemble:

1. Build raw direction signals:
   - Cross-sectional relative strength versus the fixed focus universe and major indexes.
   - Short-horizon momentum: 1/3/5/10/20-day return, breakout, and distance from recent highs.
   - Medium trend: 20/60/120/250-day moving average structure.
   - Theme breadth: rising count, average change, leader count, and成交额 concentration.
   - Volume/liquidity: amount, turnover expansion, and volume ratio versus recent average.
   - Price quality: intraday position, limit-up/limit-down state, gap, and fade risk.
   - Catalyst: verified earnings, order/news/policy catalyst, or sentiment leadership.
   - TradingAgents overlay: rating, entry, stop, and target when available.
2. Convert raw signals to a probability only after calibration:
   - Start with logistic-style mapping from raw score to probability instead of treating score points as exact probability.
   - If historical prediction outcomes are available, calibrate with Platt scaling or isotonic-style bucket calibration.
   - Track Brier score and log loss for every prediction batch. Lower Brier/log loss means better probability quality.
   - Report whether a probability is `uncalibrated`, `bucket-calibrated`, or `backtest-calibrated`.
3. Evaluate hit quality by buckets, not only one-day hit rate:
   - `55-60%`, `60-65%`, `65-70%`, `70%+`.
   - For each bucket, compare predicted probability with realized direction frequency.
   - If a bucket is overconfident or underconfident, adjust future probabilities.

Use a rule-based fallback only when historical samples are insufficient. In that fallback, label probabilities as `uncalibrated-rule-score`.

## Expected Move Model

Do not estimate predicted upside/downside only from fixed rule offsets. Use volatility and price structure:

- Calculate ATR or true-range based volatility from recent daily data when available.
- Convert ATR to percent: `ATR_pct = ATR / close * 100`.
- Estimate upside/downside as scenario ranges:
  - `强主线`: upside often `0.8-1.3 * ATR_pct`; downside depends on chase/fade risk, often `1.0-1.5 * ATR_pct`.
  - `稳健低吸`: upside often `0.5-0.9 * ATR_pct`; downside often `0.7-1.1 * ATR_pct`.
  - `弱板块/回避`: upside should be capped unless a reversal trigger appears; downside should use at least `1.0 * ATR_pct`.
- Anchor ranges to nearby support/resistance, previous high/low, limit-up/limit-down, moving averages, and entry trigger.
- For A-shares, account for涨跌停、T+1、开板换手、一字板买不到、创业板/科创板/北交所权限, and liquidity.

When printing `预测上涨幅度/预测下跌幅度`, state whether it comes from `ATR/volatility`, `support-resistance`, or `rule fallback`.

## Output Contract

Every stock line must include:

- `code` and `name`
- `板块/主题`
- `板块方向评级`: strong mainline, constructive, neutral rotation, weak repair, or avoid
- `当前价` and `涨跌幅`
- `板块趋势`: short/medium/long-term sector trend and whether the move is trend continuation, range rebound, or sentiment spike
- `明日上涨概率` and `明日下跌概率`
- `未来3-5个交易日上涨概率` and `未来3-5个交易日下跌概率`
- `预测上涨幅度` and `预测下跌幅度`
- `TradingAgents`: agent_available/agent_missing/agent_error plus rating, entry, stop, and target/trigger when available
- `资金流代理`: tick flow, big-deal flow, or unavailable with reason
- `新闻/消息判断`
- `可买性`: normal, permission-needed, blocked, or caution
- `推荐动作`: mainline buy pool, active focus, wait for pullback, turnover confirmation, monitor only, avoid chasing, permission blocked
- `预测`: conditional base case
- `建议买入时机`: conditional entry setup, not an order instruction
- `建议卖出/退出时机`: conditional sell, trim, stop-loss, or invalidation setup
- `真实依据`: concrete evidence for the predicted direction, including data source/timestamp and why it matters
- `反向证据`: what conflicts with the prediction or would make it wrong
- `证据强度`: strong, medium, weak, or insufficient
- `触发条件`: what confirms upside
- `失效条件`: what invalidates the idea
- `复盘标签`: tags such as theme_top3, theme_weak, trend_up, trend_risk, intraday_fade_risk, chase_risk, agent_available, agent_missing

Keep reports compact but complete. Put buyable or actionable names first, then high-probability-but-no-chase names, then monitor-only names, then blocked candidates. When the user says the prediction will be compared tomorrow, save or print a stable baseline containing trade date, target compare date, source timestamp, probabilities, predicted amplitude, action, trigger, invalidation, and tags.

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

- `mainline buy pool`: theme is a confirmed strong short-term mainline; leading buyable names should appear before steady defensive sectors in the offensive ranking.
- `active focus`: theme is strong, stock action is constructive, flow/news do not contradict, and buyability is acceptable.
- `wait for pullback`: stock is strong but already extended, near high, or showing big-deal selling.
- `turnover confirmation`: stock or theme is strong but current price is too extended; only consider after sufficient换手, opened limit-up with support, or a controlled pullback.
- `monitor only`: useful signal but not ready for action because theme, flow, or setup is incomplete.
- `avoid chasing`: excessive intraday rise, weak board confirmation, or risk/reward deteriorated.
- `permission blocked`: needs 创业板/科创板/北交所 access or otherwise cannot likely be bought.

Use clear Chinese action labels in final answers when helpful:

- `可积极关注`
- `主线可买池`
- `等回踩低吸`
- `不追高/等换手`
- `观察/暂不买`
- `权限限制/不纳入`

## Practical Defaults

- 10-minute refresh is reasonable during active monitoring; refresh faster only for a very small focus list.
- In weak index regimes, rank defensive strength higher: 创新药, 贵金属, 传统消费, 银行/保险 can outrank high-beta technology.
- In strong risk-on regimes, allow 半导体、AI算力、液冷、机器人 to move up only if sector breadth and flows confirm.
- Do not upgrade a stock only because its sector is strong intraday. If the sector's 60/120/250-day trend is still weak, mark the idea as rebound/repair unless it breaks above key medium-term levels with breadth and volume confirmation.
- Do not write `看涨`, `看跌`, `上涨概率大`, or `下跌风险大` without real evidence in the same line. Directional language must be backed by quote data, trend data, flow proxy, verified news, sentiment evidence, or a clearly stated combination.
- Express buy and sell timing as conditional scenarios, not commands. Use phrasing like `若...则考虑`, `等...确认`, `跌破...退出观察`, and keep real execution manual.
- If fund-flow proxies are slow, run them on the top 5-10 candidates only and state the limitation.
- If Chinese text appears as `????` or mojibake in generated files, regenerate with UTF-8 output before using the file as the baseline.
- For next-day accuracy review, compare each prediction against actual next-session close direction from the prediction timestamp or previous close. Separately score direction hit, trigger-hit, invalidation-hit, and action-label effectiveness.

## Safety

- Do not interact with brokerage order screens or submit orders.
- Do not ask for passwords, OTPs, or account credentials.
- If the user asks whether to buy, answer with a readiness verdict and conditions, not a command.
