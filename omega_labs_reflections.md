# Year in Review: What I Built at Omega Labs (Aug 2024 – Feb 2026)

> A behind-the-scenes look at the systems, decisions, and lessons from 18 months building production AI infrastructure at Omega Labs — spanning decentralized AI subnets, multi-agent frameworks, productivity tooling, and autonomous social agents.

---

## The Context

Omega Labs operates at the intersection of decentralized AI and applied LLM engineering. Most of my work lived across two Bittensor subnets (SN21 and SN24), a productivity platform called Omega Focus, and a growing set of internal AI agents and automation tools. The scale was real, the stakes were real, and the pace was relentless.

---

## 1. Bittensor Subnets — Decentralized AI at Scale

This was the most technically unique work I've done. Most AI engineers never touch decentralized infrastructure — I spent over a year building and maintaining two of Bittensor's production subnets.

### SN24 — The World's Largest AGI Multimodal Dataset

SN24 is a decentralized dataset creation subnet with **1M+ hours of footage** scraped, validated, and embedded. Miners scrape YouTube; validators score submissions using **ImageBind embeddings**; rewards flow via the Bittensor protocol.

**What I worked on:**
- Video similarity analysis and clustering using DBSCAN and K-means with t-SNE/UMAP visualizations
- A production Celery-based video scoring system with ACID transactions and PostgreSQL task management
- A real-time Next.js 15 dual-facing dashboard — miner leaderboards, submission tracking, payout insights
- A FastAPI backend serving live analytics, mining metrics, and Bittensor network integration

**The hard part:** Scoring at this scale requires efficient batched processing. We needed validators to be fast, consistent, and exploit-resistant. Building a reliable queue system that could handle spikes without dropping tasks was genuinely difficult.

### SN21 — Any-to-Any Multimodal Model Training

SN21 focused on training multimodal models using a **Llama 3 backbone with ImageBind embeddings and LoRA adapters**, starting with voice-to-voice generation.

**What I worked on:**
- FastAPI backend with Google SSO, automated wallet creation, encrypted seed storage, and alpha_balance tracking
- React + Vite + TypeScript + Material-UI dashboard for real-time network performance monitoring
- End-to-end integration between the Bittensor protocol, model training pipelines, and the frontend

**Lesson:** Decentralized systems introduce coordination problems that centralized systems don't have. You can't just restart a service — every change has network-level consequences.

---

## 2. Multi-Agent Systems — From Prototypes to Production

I built and shipped several multi-agent systems during this period. Here's what I learned building each one.

### Agent Task Validation Pipeline

A 5-stage multi-agent validation system: **Orchestrator → Sub-Validators → Synthesizer → Reward Engine → Pattern DB**. Used soft-majority voting to detect exploits and issue ΩTAO token rewards.

Built with async Python, Claude, Gemini, and OpenAI models in parallel. The key insight here was that **no single model is reliable enough for exploit detection** — ensemble voting across models with different priors catches more edge cases.

### Content Agent (3-Agent Creative Pipeline)

A three-agent system — Creative Director, Art Agent, Writing Agent — for Omega Labs content creation using Claude Sonnet 4.5 and Gemini. Produces images, blog drafts, and tweets with Discord integration for review.

This taught me that **creative pipelines need structured handoffs**. When the Art Agent and Writing Agent work in parallel without a clear contract, outputs diverge. The Creative Director agent's role as a synchronization point was more important than I initially expected.

### CatGPT — Autonomous Twitter AI Agent (@AskCatGPT)

This was one of the more complex agentic systems I've built. Key components:

- **Graph-based workflow** for autonomous tweet generation and reply
- **Multi-model routing** — dynamically selects GPT, Claude, Gemini, or Grok based on task type
- **Reinforcement learning feedback loop** — tweet performance metrics feed back into prompt optimization
- **Discord approval system** — human-in-the-loop for borderline content before posting
- **ArXiv paper analysis** — monitors AI research accounts, extracts paper content, generates contextual replies
- **Multi-layer memory system** — tracks conversation history, account context, and performance data
- **Admin dashboard** — real-time monitoring and prompt management

The RL feedback loop was the most interesting part. Tweet engagement (likes, replies, retweets) becomes a reward signal that gradually shifts generation toward higher-performing styles. It works — but it also drifts toward sensationalism if you're not careful. The Discord approval gate was as much about content quality as it was about safety.

### Digital Twin Discord Bot

Discord bot creating digital twin replicas of users via the Agno framework, with personality configs, multi-agent orchestration, and context from GitHub/Notion. Supports multiple persona bots via JSON config files.

---

## 3. Omega Focus — AI-Powered Productivity Platform

Omega Focus is a cross-platform productivity tracker with screen recording, AI conversations, and multimodal analysis. I worked across the full stack here.

**What I built:**
- **Electron companion app** (TypeScript/React 18/Tailwind) with native screen capture binaries for macOS/Windows
- **Rust-based CLI screen recorder** capturing 1080p at 30 FPS with <30% CPU usage
- **T3 Stack web version** (Next.js, tRPC, Tailwind, Prisma/Drizzle) with real-time WebSocket chat
- **Video pre-processing pipeline** using FFmpeg for fast concatenation before multimodal model ingestion (Gemini)
- **Streamlit dashboards** for productivity metrics — XP rewards, streak tracking, app usage analytics

**Lesson:** Screen recordings as a data source for AI are underexplored. The bottleneck isn't the model — it's pre-processing. Getting FFmpeg to handle diverse screen recording formats reliably in production took significantly more work than expected.

---

## 4. Infrastructure & Backend Systems

### What I Ran in Production

- **GCP** for screen buckets and compute — used heavily for the Omega Focus media pipeline
- **Docker + Kubernetes** for AI microservices deployment
- **PostgreSQL** as the primary datastore across most services — with proper indexing, async ops, and optimized SQL
- **Celery** for async task queues (video scoring pipeline)
- **FastAPI** as the backend framework of choice across SN21, SN24, and Granny API

### Database Migration Tooling

Built a Python tool for MySQL → PostgreSQL/Supabase migration with batch processing, schema preservation, row count validation, and automatic sequence reset. Saved significant time when we needed to move off legacy databases.

---

## 5. Research-Adjacent Work

I contributed to or supported several research-adjacent projects:

- **entropix** — entropy-based context-aware sampling to simulate o1-style reasoning via inference-time compute (JAX, PyTorch, MLX)
- **Search-o1** — enhancing reasoning models with agentic web search and a "Reason-in-Documents" module
- **VoiceBench** — a benchmark comparing 30+ voice assistant models across reasoning, QA, instruction-following, and safety

None of these were solely my work, but they sharpened how I think about **evaluation**. Most AI teams underinvest in evals. The teams that benchmark rigorously ship better systems.

---

## 6. Open Source Contributions

- **agenticSeek** — 100% local Manus AI alternative; voice-enabled, autonomous web browsing and task planning using Ollama/LM-Studio
- **agno** — Lightweight production agent library with ~3μs instantiation time, native multimodal, multi-agent Teams, and Agentic RAG
- **DSPy Matchmaking App** — 59 ⭐ on GitHub; full-stack internship finder using DSPy + Weaviate hybrid search

---

## Key Takeaways

**On multi-agent systems:** Agents fail at handoffs. The interface between agents — what data gets passed, in what format, with what assumptions — is where most bugs live. Invest in structured outputs and explicit contracts between agents early.

**On decentralized AI:** Bittensor adds a coordination layer that most ML engineers never deal with. Incentive design matters as much as model quality. A technically correct validator can still be gamed if the reward function has blind spots.

**On production LLM systems:** Token costs compound quickly at scale. Prompt engineering is real engineering — a 20% reduction in token consumption across all production calls is a meaningful infrastructure improvement.

**On evaluation:** Ship fast, but benchmark everything. The teams I saw move fastest were the ones with the tightest feedback loops — not the ones who moved without measuring.
