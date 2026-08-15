# Trading Skills — Claude Slash Commands for Stock Analysis

Custom Claude skills for financial analysis, derived from the [TradingAgents](https://github.com/TauricResearch/TradingAgents) framework by [TauricResearch](https://github.com/TauricResearch).

---

## Skills

### `/stock_reco <TICKER>`

Runs a full multi-agent financial analysis pipeline for any stock ticker and delivers a final trade recommendation. The pipeline mirrors the architecture described in the TradingAgents paper — six sequential stages, each handled by a specialized agent persona:

| Stage | Agents |
|-------|--------|
| 1 | Fundamentals Analyst, Market Analyst, News Analyst, Sentiment Analyst |
| 2 | Bull Researcher vs. Bear Researcher (2-round debate) |
| 3 | Research Manager (Buy / Overweight / Hold / Underweight / Sell) |
| 4 | Trader (BUY / SELL / HOLD proposal) |
| 5 | Aggressive, Conservative, and Neutral Risk Analysts (2-round debate) |
| 6 | Portfolio Manager (final decision) |

Output ends with:

```
FINAL TRANSACTION PROPOSAL: [BUY / OVERWEIGHT / HOLD / UNDERWEIGHT / SELL] — one sentence rationale
```

**Example:** `/stock_reco SPCX`

---

### `/sell_put <TICKER>`

Cash-secured put selling advisor. For a stock you're bullish on or willing to own, this pipeline recommends the specific strike price and expiration date for selling a put to collect premium income. Fine if the option expires worthless; willing to be assigned at the strike.

| Stage | Agent |
|-------|-------|
| 1 | Price & Trend Analyst — current price, trend direction, key support levels |
| 2 | Options Environment Analyst — IV rank, catalysts, put skew, liquidity |
| 3 | Strike Selector — optimal strike anchored to support + delta target |
| 4 | Expiration Selector — optimal DTE in the theta sweet spot |
| 5 | Trade Risk Analyst — full metrics table (premium, breakeven, return on capital) |
| 6 | Options Advisor — final recommendation with risk notes and exit rule |

Output ends with:

```
╔══════════════════════════════════════════════╗
║         SELL PUT RECOMMENDATION              ║
╠══════════════════════════════════════════════╣
║ Ticker     : [TICKER]                        ║
║ Action     : SELL PUT                        ║
║ Strike     : $XXX                            ║
║ Expiration : YYYY-MM-DD (~XX DTE)            ║
║ Est. Premium: $X.XX/share ($XXX/contract)    ║
║ Breakeven  : $XXX.XX                         ║
║ Cost basis if assigned: $XXX.XX              ║
║ Return if OTM: X.X% (~XX% annualized)        ║
╚══════════════════════════════════════════════╝
```

**Example:** `/sell_put SPMO`

---

### `/sell_call <TICKER> <COST_BOUGHT>`

Covered call selling advisor. You provide your average cost basis — the pipeline assesses the stock's long-term quality to set a **Retention Mode**, then recommends the optimal strike and expiration.

| Mode | When | Strike approach |
|------|------|----------------|
| **RETAIN** | Long-term quality holding | Far OTM (10–15%+), low delta — maximize probability of keeping the stock |
| **NEUTRAL** | Decent but not compelling | Moderate OTM (7–12%), balanced premium |
| **ACCEPT** | Not a long-term hold; exit welcome | Closer to ATM (3–8%) — maximize premium, assignment is fine |

| Stage | Agent |
|-------|-------|
| 1 | Long-Term Quality Analyst — retention mode verdict (RETAIN / NEUTRAL / ACCEPT) |
| 2 | Price & Trend Analyst — current price vs. cost basis, key resistance levels |
| 3 | Options Environment Analyst — IV rank, earnings timing, ex-div risk, call skew |
| 4 | Strike Selector — mode-dependent strike; enforces strike ≥ cost_bought |
| 5 | Expiration Selector — DTE sweet spot; earnings clearance for RETAIN mode |
| 6 | Covered Call Advisor — full P&L table + final recommendation with exit rule |

Output ends with:

```
╔═══════════════════════════════════════════════════╗
║          SELL CALL RECOMMENDATION                 ║
╠═══════════════════════════════════════════════════╣
║ Ticker         : [TICKER]                         ║
║ Retention Mode : RETAIN / NEUTRAL / ACCEPT        ║
║ Action         : SELL CALL                        ║
║ Strike         : $XXX                             ║
║ Expiration     : YYYY-MM-DD (~XX DTE)             ║
║ Est. Premium   : $X.XX/share ($XXX/contract)      ║
║ Cost Basis     : $XXX.XX → Net: $XXX.XX           ║
║ P&L if called away  : +$XXX/contract              ║
║ Return if OTM  : X.X% (~XX% annualized)           ║
╚═══════════════════════════════════════════════════╝
```

**Example:** `/sell_call SPMO 87.50`

---

### `/buy_leap <TICKER>`

LEAP call buying advisor. Use after `/stock_reco` gives a strong BUY signal to answer: *should I buy shares or a LEAP call?* Assesses long-term conviction, chooses the right approach, and delivers a full cost/leverage comparison.

| Conviction | Approach | Strike target |
|-----------|----------|--------------|
| **HIGH** | Stock Replacement | Deep ITM, delta 0.75–0.85 — behaves like owning shares at ~30–50% of the capital |
| **MODERATE** | Speculative | ATM/OTM, delta 0.35–0.50 — higher leverage, higher risk |
| **LOW** | No trade | Pipeline stops and advises against buying a LEAP |

| Stage | Agent |
|-------|-------|
| 1 | Long-Term Conviction Analyst — HIGH / MODERATE / LOW verdict (stops if LOW) |
| 2 | LEAP Approach Selector — stock replacement vs. speculative, with tradeoff |
| 3 | Price & Upside Analyst — current price, bear/base/bull 12-month targets |
| 4 | Strike Selector — delta, intrinsic/extrinsic value, breakeven % move required |
| 5 | Expiration Selector — January LEAP dates, roll trigger date |
| 6 | Risk & Cost Analyst — 3-table comparison: cost, scenario P&L, key metrics |
| 7 | LEAP Advisor — final recommendation with sizing and management rules |

Output ends with:

```
╔════════════════════════════════════════════════════╗
║           BUY LEAP RECOMMENDATION                 ║
╠════════════════════════════════════════════════════╣
║ Ticker      : [TICKER]                            ║
║ Approach    : Stock Replacement / Speculative     ║
║ Action      : BUY CALL (LEAP)                     ║
║ Strike      : $XXX                                ║
║ Expiration  : YYYY-MM-DD (~XX months)             ║
║ Est. Premium: $XX.XX/share ($X,XXX/contract)      ║
║ Breakeven   : $XXX.XX at expiry (+XX%)            ║
║ vs. 100 shares: saves $X,XXX upfront              ║
║ Leverage    : ~X.Xx                               ║
║ Max loss    : $X,XXX (premium paid)               ║
║ Roll by     : YYYY-MM-DD                          ║
╚════════════════════════════════════════════════════╝
```

**Example:** `/buy_leap AAPL` or `/bl AAPL`

---

## Credit

This skill is a distillation of the **TradingAgents** multi-agent framework, created by **Yijia Xiao, Edward Sun, Di Luo, and Wei Wang** at TauricResearch. The agent roles, pipeline structure, debate mechanics, and rating scale all originate from their work.

> **Paper:**
>
> Yijia Xiao, Edward Sun, Di Luo, Wei Wang. *TradingAgents: Multi-Agents LLM Financial Trading Framework.* arXiv:2412.20138 [q-fin.TR], 2025.
> https://arxiv.org/abs/2412.20138

```bibtex
@misc{xiao2025tradingagentsmultiagentsllmfinancial,
      title={TradingAgents: Multi-Agents LLM Financial Trading Framework},
      author={Yijia Xiao and Edward Sun and Di Luo and Wei Wang},
      year={2025},
      eprint={2412.20138},
      archivePrefix={arXiv},
      primaryClass={q-fin.TR},
      url={https://arxiv.org/abs/2412.20138},
}
```

The original framework runs a full agentic pipeline with live data (Yahoo Finance, StockTwits, Reddit, FRED, Polymarket) and supports multiple LLM providers. This skill adapts the same pipeline into a single-prompt format that runs entirely within Claude's context window using its built-in knowledge.

> **Disclaimer:** This skill is for research and educational purposes only. It is not financial, investment, or trading advice.

---

## Setup

### Claude Code

Claude Code supports custom slash commands as Markdown files. The command file uses `$ARGUMENTS` as a placeholder for whatever you type after the command name.

**Install globally** (available in every project):

```bash
mkdir -p ~/.claude/commands
cp skills/stock_reco/skill.md ~/.claude/commands/stock_reco.md
cp skills/sell_put/skill.md ~/.claude/commands/sell_put.md
cp skills/sell_call/skill.md ~/.claude/commands/sell_call.md
cp skills/buy_leap/skill.md ~/.claude/commands/buy_leap.md
```

**Optional short aliases** (`/sr`, `/sp`, `/sc`, `/bl`):

```bash
cp ~/.claude/commands/stock_reco.md ~/.claude/commands/sr.md
cp ~/.claude/commands/sell_put.md ~/.claude/commands/sp.md
cp ~/.claude/commands/sell_call.md ~/.claude/commands/sc.md
cp ~/.claude/commands/buy_leap.md ~/.claude/commands/bl.md
```

Open any Claude Code session and use either the full name or the alias:

```
/stock_reco AAPL       or  /sr AAPL
/sell_put SPMO         or  /sp SPMO
/sell_call SPMO 87.50  or  /sc SPMO 87.50
/buy_leap AAPL         or  /bl AAPL
```

Claude Code will substitute `$ARGUMENTS` with the ticker and run the full pipeline.

**Alternatively, install per-project** (only active when Claude Code is open in that directory):

```bash
mkdir -p /path/to/your/project/.claude/commands
cp skills/stock_reco/skill.md /path/to/your/project/.claude/commands/stock_reco.md
cp skills/sell_put/skill.md /path/to/your/project/.claude/commands/sell_put.md
cp skills/sell_call/skill.md /path/to/your/project/.claude/commands/sell_call.md
cp skills/buy_leap/skill.md /path/to/your/project/.claude/commands/buy_leap.md
```

---

### Claude Chat (claude.ai)

Slash commands are a Team/Enterprise plan feature and are not available on Pro. The workaround is to use a **Project** with custom **Instructions** — this sets a persistent system prompt that teaches Claude to recognize the `/stock_reco` trigger across all chats in that project.

**Steps:**

1. Go to [claude.ai](https://claude.ai) and open or create a **Project** (e.g. "Trading").
2. On the right-hand panel, click **+** next to **Instructions**.
3. Paste the full contents of all instruction files, one after the other:
   - [`skills/stock_reco/chat_instructions.md`](skills/stock_reco/chat_instructions.md)
   - [`skills/sell_put/chat_instructions.md`](skills/sell_put/chat_instructions.md)
   - [`skills/sell_call/chat_instructions.md`](skills/sell_call/chat_instructions.md)
   - [`skills/buy_leap/chat_instructions.md`](skills/buy_leap/chat_instructions.md)
4. Save.

Or use the one-liner to copy all four to clipboard:

```bash
cat skills/stock_reco/chat_instructions.md \
    skills/sell_put/chat_instructions.md \
    skills/sell_call/chat_instructions.md \
    skills/buy_leap/chat_instructions.md | pbcopy
```

Now, in any new chat within that project, type any of:

```
/stock_reco SPCX    or  /sr SPCX
/sell_put SPMO      or  /sp SPMO
/sell_call SPMO 87.50  or  /sc SPMO 87.50
/buy_leap AAPL      or  /bl AAPL
```

Claude will recognize the trigger and work through all pipeline stages automatically, using today's date. No additional prompt is needed.

**Note:** Instructions are applied to every chat in the project, so this works best if the project is dedicated to trading analysis.

---

## Repository structure

```
trading-skills/
└── skills/
    ├── stock_reco/
    │   ├── skill.md               # Claude Code command (uses $ARGUMENTS)
    │   └── chat_instructions.md   # Claude Chat project instructions
    ├── sell_put/
    │   ├── skill.md               # Claude Code command (uses $ARGUMENTS)
    │   └── chat_instructions.md   # Claude Chat project instructions
    ├── sell_call/
    │   ├── skill.md               # Claude Code command (uses $ARGUMENTS: TICKER COST_BOUGHT)
    │   └── chat_instructions.md   # Claude Chat project instructions
    └── buy_leap/
        ├── skill.md               # Claude Code command (uses $ARGUMENTS)
        └── chat_instructions.md   # Claude Chat project instructions
```

---

## Disclaimer

This repository derives ideas from the TradingAgents paper and framework. All pipeline design credit belongs to the original authors. This is not affiliated with TauricResearch. Output from this skill should not be used as the basis for real trading decisions.
