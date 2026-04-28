# StockPilot-CN — AI-Powered A-Share Market Analysis Agent

[中文文档](README.md) | English

> **Positioning**: A multi-agent intelligent analysis system for the Chinese A-share market
> **Version**: v1.0 | 2026-04-28

---

## 1. Project Overview

### 1.1 Background & Pain Points

The A-share market serves over 200 million investors, yet faces three core challenges:

| Pain Point | Current State | StockPilot-CN Solution |
|-----------|---------------|----------------------|
| Information Overload | Thousands of daily announcements, research reports, and news — retail investors can't filter effectively | Multi-source data aggregation + LLM summarization |
| High Analysis Barrier | Financial, technical, and capital flow analysis require professional expertise | Natural language interaction lowers the barrier |
| Subjective Decision-Making | Retail investors lack systematic analysis frameworks | Agent-orchestrated multi-dimensional structured analysis |

### 1.2 Project Goals

Build a **multi-agent collaborative** A-share intelligent analysis system where users complete full-cycle analysis — from individual stock diagnosis to investment decision support — through natural language conversation.

Core Capabilities:

- **Deep Stock Analysis**: Fundamentals + Technicals + Capital Flow + Sentiment — four dimensions integrated
- **Industry Comparison**: Horizontal peer comparison to identify relative advantages
- **Research Report Summarization**: Auto-fetch and distill broker research reports
- **Risk Alerts**: Real-time monitoring of position anomalies, announcement risks, capital irregularities
- **Portfolio Diagnostics**: Holdings health assessment, correlation analysis, position sizing suggestions

### 1.3 Target Users

- Individual investors (A-share retail)
- Independent analysts / financial media
- Small private equity fund researchers

---

## 2. System Architecture

### 2.1 Architecture Diagram

```
┌─────────────────────────────────────────────────────┐
│                   User Interface                     │
│           Web UI / CLI / API Gateway                 │
└───────────────────────┬─────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────┐
│                 Orchestrator Agent                    │
│      (Intent Understanding → Task Decomposition      │
│        → Dispatch → Aggregation)                     │
│             Powered by Xiaomi MiMo-V2-Pro            │
└───┬──────┬──────┬──────┬──────┬──────┬──────────────┘
    │      │      │      │      │      │
    ▼      ▼      ▼      ▼      ▼      ▼
┌──────┐┌──────┐┌──────┐┌──────┐┌──────┐┌──────┐
│ Fund. ││ Tech. ││Capital││Sentmt.││Rsch. ││ Risk │
│ Agent ││ Agent ││ Agent ││ Agent ││Agent ││Agent │
└──┬───┘└──┬───┘└──┬───┘└──┬───┘└──┬───┘└──┬───┘
   │       │       │       │       │       │
   └───────┴───────┴───────┴───────┴───────┘
                       │
┌──────────────────────▼─────────────────────────────┐
│                    Data Layer                        │
│  A-share API │ Financial Data │ News Crawler │ DB    │
│  Qichacha    │ Announcements  │ Macro Data           │
└─────────────────────────────────────────────────────┘
```

### 2.2 Agent Responsibilities

| Agent | Core Responsibility | Input | Output |
|-------|-------------------|-------|--------|
| **Orchestrator** | Understand intent, decompose tasks, dispatch agents, synthesize conclusions | Natural language query | Structured analysis report |
| **Fundamental Agent** | Financial analysis: revenue/profit/cashflow/ROE/leverage | Stock code | Financial health score + key metric interpretation |
| **Technical Agent** | Technical analysis: K-line patterns, MA systems, MACD/RSI/KDJ, support/resistance | Stock code + time range | Technical score + trend assessment |
| **Capital Agent** | Capital flow: institutional/retail flow, northbound capital, margin trading, dragon-tiger list | Stock code | Capital flow score + anomaly alerts |
| **Sentiment Agent** | Sentiment analysis: news sentiment, social media heat, policy tracking | Stock code / industry | Sentiment score + event list |
| **Research Agent** | Report aggregation: fetch broker reports, extract key views, summarize target prices | Stock code | Report digest + institutional consensus |
| **Risk Agent** | Risk assessment: financial warnings, delisting risk, pledge risk, compliance risk | Stock code / portfolio | Risk level + risk item checklist |

### 2.3 Agent Collaboration Workflow

Example: User asks "Should I buy BYD (002594) now?"

```
User Query
  │
  ▼
Orchestrator parses intent → Identified as "comprehensive stock analysis"
  │
  ├─→ Fundamental Agent: Pull BYD's 3-year financials, compute metrics
  ├─→ Technical Agent: Fetch 60-day K-line, analyze patterns
  ├─→ Capital Agent: Query 10-day capital flow, northbound holding changes
  ├─→ Sentiment Agent: Search 7-day news, analyze sentiment
  ├─→ Research Agent: Aggregate 30-day analyst reports and target prices
  └─→ Risk Agent: Check pledge ratio, goodwill, insider selling alerts
  │
  ▼
Orchestrator synthesizes six-dimensional analysis → Generates structured report
  │
  ▼
Output: Composite score + detailed analysis per dimension + Action recommendation
```

---

## 3. Core Module Design

### 3.1 Fundamental Analysis Agent

**Data Sources**:

- East Money / Tonghuashun API: Financial statements (balance sheet, income statement, cash flow)
- Qichacha MCP (already integrated): Shareholder structure, related companies, operational risks
- Company announcement data

**Analysis Dimensions**:

```
Profitability: ROE, ROA, gross margin, net margin trends (3-year + TTM)
Growth: Revenue growth, profit growth, PEG
Solvency: Debt-to-asset ratio, current ratio, quick ratio
Cash Flow Quality: Operating CF / net income ratio, free cash flow
Valuation: PE/PB/PS percentile (vs. own history & industry peers)
```

**Output Format**:

```json
{
  "stock_code": "002594",
  "stock_name": "BYD",
  "fundamental_score": 82,
  "metrics": {
    "roe": {"value": "18.5%", "trend": "rising", "rank_in_industry": "Top 20%"},
    "pe_ttm": {"value": 25.3, "percentile_5y": "35%"}
  },
  "highlights": ["4 consecutive quarters of >30% revenue growth", "Operating cash flow significantly improved"],
  "concerns": ["Accounts receivable turnover days increased 15 days YoY"],
  "summary": "Overall healthy fundamentals with outstanding profitability and growth..."
}
```

### 3.2 Technical Analysis Agent

**Capabilities**:

| Analysis Type | Specific Indicators |
|--------------|-------------------|
| Trend Detection | MA5/10/20/60/120/250 alignment, trend strength |
| Momentum | MACD (golden/death cross), RSI (overbought/oversold), KDJ |
| Pattern Recognition | Head & shoulders, double top/bottom, triangle, box breakout |
| Volume-Price | Volume-price coordination, volume expansion breakout detection |
| Support/Resistance | Key level identification (round numbers, prior highs/lows, gaps) |

**Feature**: Natural language explanations of technical indicators — accessible to investors who don't understand K-line charts.

### 3.3 Capital Flow Agent

**Data Sources**:

- Institutional capital flow data (East Money)
- Northbound capital (Stock Connect)
- Margin trading data
- Dragon-tiger list data
- Block trade data

**Core Logic**:

```
IF 5-day institutional net inflow sustained
   AND northbound holding ratio rising
   AND margin buy ratio < 30% (not leverage-driven)
THEN capital flow signal = bullish
```

### 3.4 Sentiment Analysis Agent

**Workflow**:

1. Keyword search: Company name, stock code, executive names, key products
2. Article scraping and cleaning
3. MiMo model sentiment analysis (positive/neutral/negative + confidence)
4. Event extraction (M&A, insider selling, earnings guidance, litigation)
5. Impact assessment (potential price impact of events)

### 3.5 Research Report Agent

**Capabilities**:

- Auto-fetch reports for a given stock over 30-90 days
- MiMo model extracts key views: buy/hold/sell, target price, core thesis
- Aggregate institutional consensus and divergence
- Identify "research coverage spikes" (multiple brokers covering simultaneously often signals inflection points)

### 3.6 Risk Assessment Agent

**Risk Checklist**:

```
□ Financial Risk: Consecutive losses, negative cash flow, goodwill > 50% of net assets
□ Governance Risk: High-pledge major shareholders, frequent related-party transactions, insider selling
□ Compliance Risk: CSRC penalties, exchange inquiry letters, litigation/arbitration
□ Trading Risk: ST/*ST status, abnormal price movements, suspension risk
□ Delisting Risk: Triggering registration-based delisting criteria (revenue < 100M, price < 1 CNY)
□ Liquidity Risk: Daily average trading volume < 50M CNY, low turnover rate
```

### 3.7 Orchestrator Agent

**Core Responsibilities**:

1. **Intent Recognition**: Parse natural language, classify analysis type (stock/industry/portfolio/comparison)
2. **Task Decomposition**: Break complex analysis into parallelizable subtasks
3. **Agent Dispatch**: Orchestrate execution order based on dependencies
4. **Result Synthesis**: Merge agent outputs, resolve contradictions, generate unified conclusion
5. **Context Management**: Maintain conversation context for multi-turn interactions

**Scheduling Strategy**:

```
For individual stock analysis:
  Step 1 (parallel): Fundamental + Technical + Capital + Risk
  Step 2 (depends on Step 1): Sentiment (filter news based on fundamental results)
  Step 3 (parallel): Research
  Step 4 (aggregation): Orchestrator merges all results → generates report
```

---

## 4. Tech Stack

### 4.1 Model Layer

| Component | Selection | Notes |
|-----------|----------|-------|
| Agent Framework | Claude Code + Custom Orchestration | Leverage Claude Code's Agent capabilities + custom dispatch layer |
| Primary Reasoning Model | **GLM-5.1** | Agent reasoning, intent understanding, report generation |
| Long-Text Analysis Model | **Xiaomi MiMo-V2-Pro** | 256K context, deep analysis of research reports and annual reports |
| High-Frequency Lightweight Model | **Qwen 3.6 Plus** | Sentiment analysis, entity extraction, fast summarization |
| Multimodal Model | **Kimi K2.6** | Chart understanding, K-line pattern visual recognition |
| Auxiliary Model | MiMo-V2-Flash | Low-latency supplementary tasks |
| Embedding | Qwen Embedding | Research report retrieval, semantic similarity |

### 4.2 Data Layer

| Data Type | Source | Integration |
|----------|--------|------------|
| A-share Quotes | East Money / AKShare | Python API |
| Financial Data | Tonghuashun iFinD / Tushare | REST API |
| Company Info | Qichacha MCP (integrated) | MCP Tool |
| Research Reports | East Money Research Center | Scraper + API |
| News & Sentiment | cls.cn / Sina Finance | Scraper |
| Macro Economics | NBS / Wind | API |

### 4.3 Engineering Layer

| Component | Selection |
|----------|----------|
| Language | Python 3.12+ |
| Agent Framework | LangGraph / CrewAI (alternative) |
| Vector DB | ChromaDB (local) / Milvus (production) |
| Task Queue | Celery + Redis |
| Database | PostgreSQL + TimescaleDB (time-series) |
| Frontend | Next.js + TailwindCSS |
| Deployment | Docker Compose (dev) / K8s (production) |

---

## 5. MiMo Model Usage Plan

### 5.1 Token Consumption Estimate

| Use Case | Model | Daily Calls | Tokens/Call (est.) | Daily Total |
|---------|-------|------------|-------------------|------------|
| Orchestrator understanding & synthesis | GLM-5.1 | 500 | ~4K input + ~2K output | ~3M Tokens |
| Fundamental analysis interpretation | GLM-5.1 | 300 | ~8K input + ~1.5K output | ~2.85M Tokens |
| Technical analysis interpretation | Qwen 3.6 Plus | 300 | ~2K input + ~1K output | ~0.9M Tokens |
| Capital flow analysis | Qwen 3.6 Plus | 300 | ~2K input + ~1K output | ~0.9M Tokens |
| Sentiment analysis | Qwen 3.6 Plus | 1000 | ~1K input + ~0.3K output | ~1.3M Tokens |
| Research report summarization | MiMo-V2-Pro | 200 | ~15K input + ~2K output | ~3.4M Tokens |
| Risk assessment | MiMo-V2-Flash | 300 | ~3K input + ~0.5K output | ~1.05M Tokens |
| Chart/K-line pattern recognition | Kimi K2.6 | 200 | ~2K input + ~1K output | ~0.6M Tokens |
| **Total** | | **~3,100/day** | | **~14M Tokens/day** |

### 5.2 Multi-Model Routing Strategy

A **model routing** strategy automatically selects the optimal model based on task characteristics:

| Task Characteristic | Routing Target | Reason |
|-------------------|---------------|--------|
| Complex reasoning, multi-step decisions | GLM-5.1 | Strong comprehensive reasoning, ideal for Orchestrator and deep analysis |
| Long text (>10K Tokens) | MiMo-V2-Pro | 256K context window, suited for full report/annual report analysis |
| High concurrency, low latency (sentiment/summary) | Qwen 3.6 Plus | Fast response, cost-efficient, ideal for batch processing |
| Image/chart understanding | Kimi K2.6 | Multimodal capability, suited for K-line charts and financial chart interpretation |
| Lightweight supplementary tasks | MiMo-V2-Flash | Low-cost fallback |

### 5.3 Why Multi-Model Composition

1. **Leverage Each Model's Strengths**: Different models excel at different tasks — combining them yields better results than any single model
2. **Cost Optimization**: Lightweight models for simple tasks, powerful models for complex ones — avoid overkill
3. **Redundancy**: Automatic failover to backup models when a single model API experiences issues
4. **Full Chinese Language Coverage**: GLM, MiMo, Qwen, and Kimi are all deeply optimized for Chinese — naturally aligned with A-share analysis scenarios

---

## 6. Development Roadmap

### Phase 1: Infrastructure (Weeks 1-2)

- [ ] Data layer setup: A-share market API integration, financial data pipeline
- [ ] Agent framework selection and base architecture
- [ ] MiMo API integration and prompt template design
- [ ] Milestone: Successfully fetch single stock quotes + financial data via API

### Phase 2: Core Agent Development (Weeks 3-5)

- [ ] Fundamental Agent: Financial data fetching + MiMo analysis
- [ ] Technical Agent: K-line data + indicator computation + MiMo interpretation
- [ ] Capital Agent: Capital flow data integration + analysis logic
- [ ] Risk Agent: Risk checklist implementation
- [ ] Milestone: Individual agents produce analysis output, manually verified accuracy > 85%

### Phase 3: Agent Collaboration & Orchestration (Weeks 6-7)

- [ ] Orchestrator Agent: Intent recognition + task decomposition
- [ ] Inter-agent communication protocol design
- [ ] Sentiment Agent: News crawling + MiMo sentiment analysis
- [ ] Research Agent: Report fetching + summary generation
- [ ] Milestone: User natural language query triggers automatic multi-agent collaboration with complete report output

### Phase 4: UX Optimization (Weeks 8-9)

- [ ] Web UI development (Next.js)
- [ ] Multi-turn conversation and context memory
- [ ] Analysis report visualization (charts, scorecards)
- [ ] Portfolio management features
- [ ] Milestone: 5 test users complete end-to-end experience feedback

### Phase 5: Polish & Launch (Weeks 10-12)

- [ ] Performance optimization (caching, concurrency, streaming output)
- [ ] Accuracy calibration (backtesting against historical data)
- [ ] Documentation and demo recording
- [ ] Open source preparation (repo cleanup, README, deployment docs)
- [ ] Milestone: GitHub open source + live demo online

---

## 7. Expected Outcomes

### 7.1 Deliverables

| Deliverable | Description | Timeline |
|------------|------------|----------|
| GitHub Open Source Repo | Complete code + deployment docs + README | Phase 5 |
| Live Demo | Interactive web application | Phase 5 |
| Technical Blog Posts | 3-5 articles sharing development insights | Ongoing |
| Video Demo | 5-minute feature demonstration video | Phase 5 |

### 7.2 Key Metrics

| Metric | Target |
|--------|--------|
| Analysis accuracy (vs. professional analysts) | > 80% |
| Full analysis response time | < 30 seconds |
| Stock coverage | Full A-share universe (5,000+) |
| DAU (3 months post-launch) | 500+ |

---

## 8. Toolchain & MCP Integrations

### 8.1 Already Deployed MCP Tools

| MCP Tool | Usage |
|---------|-------|
| **Qichacha MCP (qcc)** | Company registration info, shareholder structure, risk data |
| **Exa MCP** | Financial news search, policy document retrieval |
| **Tavily MCP** | Supplementary search and web scraping |
| **MinerU MCP** | Research report PDF to Markdown conversion |
| **PaddleOCR MCP** | Chart and screenshot text recognition |

### 8.2 AI Tools

| Tool | Usage |
|------|-------|
| **Claude Code** | Core development tool (coding, debugging, Agent development) |
| **Xiaomi MiMo API** | Agent inference model |
| **Cursor** | Auxiliary coding |
| **GitHub Copilot** | Auxiliary coding |

### 8.3 Primary Models

| Model | Usage |
|-------|-------|
| **GLM-5.1** | Agent reasoning, report generation (primary model) |
| **Xiaomi MiMo-V2-Pro** | Long-text research report analysis (256K context) |
| **Qwen 3.6 Plus** | High-frequency lightweight tasks (sentiment analysis, entity extraction) |
| **Kimi K2.6** | Chart/K-line pattern visual recognition |
| **MiMo-V2-Flash** | Low-latency supplementary tasks |
| **Claude Sonnet 4** | Development assistance (Claude Code underlying model) |

---

## 9. Innovation Highlights

1. **Multi-Agent Security Analysis Collaboration**: First open-source A-share analysis system based on multi-agent architecture, with each agent specializing in one analysis dimension
2. **Full Natural Language Interaction**: From data queries to analysis reports — all through natural language, zero learning curve
3. **Multi-Model Intelligent Routing**: Automatically selects the optimal model based on task characteristics (GLM for reasoning, MiMo for long text, Qwen for high concurrency, Kimi for multimodal), leveraging each model's strengths
4. **MCP Toolchain Reuse**: Qichacha, MinerU, and other deployed MCPs directly embedded into agent toolchain, reducing redundant development
5. **Open Source + Self-Hostable**: MIT license, users can self-deploy, data never uploaded to third parties

---

## 10. Risk Assessment & Disclaimer

### 10.1 Project Risks

| Risk | Mitigation |
|------|-----------|
| Data API changes/rate limiting | Multi-source backup + local caching |
| Model hallucination (fabricated data) | Structured prompts + data validation layer + confidence scoring |
| Compliance risk (investment advice) | Clear disclaimer: "For reference only, not investment advice" |

### 10.2 Disclaimer

All analysis results are generated by AI models and are **for learning and research purposes only — they do not constitute investment advice**. Investment involves risk; decisions should be made with caution. The system bears no responsibility for investment losses resulting from the use of analysis results.

---

*This document will be continuously updated as the project progresses. See the GitHub repository for the latest version.*
