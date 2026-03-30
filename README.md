# Case Study: LiveData AI Context Engine

**Role:** Product Manager (spec + build)
**Stack:** TypeScript, Next.js, PostgreSQL, OpenAI Embeddings, MCP, Prisma
**Status:** Live — widely adopted across the organization

> This is a case study. No proprietary source code, customer data, or internal content is included. See `NOTICE.md`.

---

## The Problem

LiveData serves 90+ hospitals with a complex product suite spanning surgical scheduling, real-time OR coordination, analytics, and patient flow. The company has deep institutional knowledge — product details, pricing, customer personas, competitive positioning, clinical domain expertise, and operational KPIs — spread across documents, tribal knowledge, and individual heads.

For a lean team, that creates a constant tax: every new employee ramps slowly, every sales rep has to remember which battle card applies, every PM has to re-research domain benchmarks before writing a PRD. The knowledge existed. Getting to it was the problem.

The goal was to make that institutional knowledge instantly accessible to anyone in the org, in natural language, calibrated to their role.

---

## My Role

I designed and built the entire system. This is an internal product in active daily use across customer success, sales, marketing, and product.

---

## What I Built

### Department-Specific Engines
Rather than a single generic chatbot, the system is organized into purpose-built engines for each function:

| Engine | Primary Users | Optimized For |
|--------|--------------|---------------|
| **Sales Engine** | Account executives | Competitive positioning, objection handling, pricing, customer profiles |
| **CS Engine** | Customer success managers | Product deep-dives, implementation guidance, escalation context |
| **PM Engine** | Product managers | Domain benchmarks, feature context, roadmap alignment |
| **Marketing Engine** | Marketing | Brand voice, product messaging, content guidance |

Each engine loads only the context files relevant to its domain — a key cost and quality optimization.

### Smart Context Routing
The system doesn't load all company knowledge for every query. A routing layer determines which context files are needed based on the question, and loads only those. A query about Insights pricing loads product files. A query about VA hospital KPIs loads the domain and company files. This keeps responses fast, focused, and cheap.

### Structured Knowledge Base
The underlying knowledge is maintained as structured markdown files covering:
- Company strategy, priorities, and FY targets
- Full product suite (features, pricing, integrations, architecture)
- Customer segments (VA vs. commercial, personas, pain points, buying patterns)
- Competitive landscape and battlecards
- Clinical domain knowledge (OR metrics, benchmarks, surgical workflow)

### Webapp
A Next.js interface for employees to interact with the engine, with role-appropriate views and query history.

### MCP Server
An MCP server exposes the context engine's capabilities to Claude Code and other AI tooling used internally — turning the knowledge base into a tool other AI systems can call.

---

## Architecture

```
┌──────────────────────────────────────────────┐
│            Next.js Webapp (Internal)          │
│   Department selector → Query → Response      │
└───────────────────┬──────────────────────────┘
                    │
┌───────────────────▼──────────────────────────┐
│              Routing Layer                    │
│  Reads query → selects relevant context files │
│  Loads 1-3 files, never the full corpus       │
└──────────┬─────────────────┬─────────────────┘
           │                 │
┌──────────▼──────┐ ┌────────▼────────────────┐
│  Context Files  │ │   OpenAI Embeddings      │
│  (structured    │ │   PostgreSQL (Prisma)    │
│   markdown)     │ │   Semantic search        │
└─────────────────┘ └─────────────────────────┘
           │
┌──────────▼──────────────────────────────────┐
│              MCP Server                      │
│  Exposes engine as tools for AI assistants   │
└─────────────────────────────────────────────┘
```

---

## Key Design Decisions

**1. Smart context loading over full-corpus RAG**
The most common RAG failure mode is dumping too much context and getting diluted, unfocused answers. This system uses explicit routing rules — based on query type — to load only the relevant files. The result is faster, more precise responses at lower cost.

**2. Department engines instead of one generic assistant**
A sales rep asking about competitive positioning needs different framing than a PM asking the same question. Separate engines with separate system prompts and context priorities meant the tool was useful on day one, not after weeks of prompt tuning.

**3. Structured markdown as the knowledge layer**
Rather than ingesting unstructured documents and hoping embeddings catch everything, the knowledge base is actively maintained as clean, structured files. This makes answers more reliable and makes knowledge updates intentional rather than accidental.

**4. MCP integration for composability**
Building an MCP server meant the context engine could become a tool other internal AI systems call — Claude Code sessions, custom agents, future products — without duplicating the knowledge or the retrieval logic.

---

## Outcomes

- Widely adopted across customer success, sales, marketing, and product teams
- Reduced ramp time for new employees by giving instant access to institutional knowledge
- Sales reps can access competitive positioning and objection handling without searching through decks
- PMs can pull domain benchmarks and product context directly into their workflow
- Foundation that other internal AI tools now build on via MCP
