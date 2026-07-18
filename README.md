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
```

That's it. Open any Claude Code session and type:

```
/stock_reco AAPL
```

Claude Code will substitute `$ARGUMENTS` with `AAPL` and run the full pipeline.

**Alternatively, install per-project** (only active when Claude Code is open in that directory):

```bash
mkdir -p /path/to/your/project/.claude/commands
cp skills/stock_reco/skill.md /path/to/your/project/.claude/commands/stock_reco.md
```

---

### Claude Chat (claude.ai)

Slash commands are a Team/Enterprise plan feature and are not available on Pro. The workaround is to use a **Project** with custom **Instructions** — this sets a persistent system prompt that teaches Claude to recognize the `/stock_reco` trigger across all chats in that project.

**Steps:**

1. Go to [claude.ai](https://claude.ai) and open or create a **Project** (e.g. "Trading").
2. On the right-hand panel, click **+** next to **Instructions**.
3. Paste the full contents of [`skills/stock_reco/chat_instructions.md`](skills/stock_reco/chat_instructions.md).
4. Save.

Now, in any new chat within that project, type:

```
/stock_reco SPCX
```

Claude will recognize the trigger and work through all six pipeline stages automatically, using today's date. No additional prompt is needed.

**Note:** Instructions are applied to every chat in the project, so this works best if the project is dedicated to trading analysis.

---

## Repository structure

```
trading-skills/
└── skills/
    └── stock_reco/
        ├── skill.md               # Claude Code command (uses $ARGUMENTS)
        └── chat_instructions.md   # Claude Chat project instructions
```

---

## Disclaimer

This repository derives ideas from the TradingAgents paper and framework. All pipeline design credit belongs to the original authors. This is not affiliated with TauricResearch. Output from this skill should not be used as the basis for real trading decisions.
