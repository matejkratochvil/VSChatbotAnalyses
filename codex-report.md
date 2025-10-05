# Chatbot Solution Options for "Chci na vejšku"

## Context
Students need a bilingual (CZ/SK) assistant that helps choose suitable universities, works with an internal database (events, admissions, matching attributes), supports long-term memory, dynamic guidance scenarios, analytics, optional web search, Azure alignment, and cost-sensitive operations. Below are four candidate architectures, from lean MVP to full multi-agent ecosystem.

---

## Option A – Guided FAQ Copilot (MVP)
**Concept**: Single agent using OpenAI Responses API (e.g., `gpt-4o-mini`) with prompt-engineered flows and direct SQL/REST connectors.

**Key Components**
- Prompt templates encode greeting + scenario prompts; table-driven branching for the three counselor scenarios.
- Direct DB tool (function call) for program metadata, deadlines, campus info.
- Session-level short-term memory (`previous_response_id`) to reuse context within one visit; persistent memory skipped initially.
- Inline telemetry: conversation events pushed to GA4 via lightweight middleware.

**Pros**
- Fastest path to pilot, minimal engineering.
- Leverages cookbook patterns for function calling and simple routine execution (`examples/How_to_build_an_agent_with_the_node_sdk.mdx`, `How_to_call_functions_with_chat_models.ipynb`).
- Easy to host on Azure Container Apps / Functions.

**Cons / Risks**
- No durable memory across sessions; personalization limited to manual parameters.
- Scenario expansion or multi-step planning handled purely in prompts → fragile.
- No guardrails beyond basic validation.

**Use When**
Need proof-of-value within weeks; acceptable for limited beta with manual supervision.

---

## Option B – Retrieval-Centric Counselor (Enhanced)
**Concept**: Single orchestrator agent with durable memory, RAG over internal content, tool retries, and lightweight web search fallback.

**Key Components**
- Responses API agent + OpenAI Agents SDK `Runner` to gain session management and guardrails.
- Vector store (e.g., Azure Cognitive Search or OpenAI file search) populated with program brochures, counselor notes; scenario prompts stored as Markdown and retrieved per path.
- Customer profile memory: user_id keyed summary (`SummarizingSession` pattern from `session_memory.ipynb`) persisted to CosmosDB/Redis.
- Toolchain: DB lookup, scenario content fetch, file search, optional `web_search_preview` gated by confidence; `tool_retry_prompt.md` logic for resilience.
- Analytics: centralized logging + Langfuse integration for trace inspection (`evaluate_agents.ipynb`).

**Pros**
- Meets long-term memory requirement via summary snapshots plus DB writes.
- Scenario authoring becomes content-driven (Markdown + embeddings) rather than hardcoded prompts.
- Guardrails + retries reduce hallucinations and dead links.

**Cons / Risks**
- Slightly higher latency (RAG + summarization calls); needs caching strategy.
- Still single-agent; complex multi-step flows must be encoded carefully.

**Use When**
Ready for public release with robust content coverage but limited engineering budget.

---

## Option C – Specialist Duo with Planner (Production Baseline)
**Concept**: Portfolio-style architecture—Orchestrator (Planner) calls two specialist agents: Guidance Advisor (scenario reasoning) and Data Specialist (fact retrieval), plus Memo synthesizer.

**Key Components**
- Head agent uses Agents SDK `handoffs`/`agents-as-tools` pattern (see `multi-agent-portfolio-collaboration` prompts) to delegate.
    - **Guidance Advisor**: maintains scenario flow, leverages `run_all_specialists_parallel`-like batching to gather preferences, calls memory service for user history.
    - **Data Specialist**: handles DB queries, RAG, optional web search; runs guardrails ensuring official sources prioritized (`supply_chain_guardrails.py` inspiration).
- Memo/editor agent produces structured answer (summary + recommended programs + next steps) so downstream analytics can parse sections.
- Long-term memory: combination of turn summaries + structured profile tables; scenario completions appended for personalization.
- MCP integration for internal services (e.g., program recommendations microservice) using streamable HTTP servers (guidance from `mcp_tool_guide.ipynb`).
- Observability: Langfuse traces, GA4 events, Azure App Insights.

**Pros**
- Clear separation of reasoning vs retrieval reduces prompt bloat, improves accuracy.
- Parallel tool calls keep latency tolerable while enriching answers (per `parallel_agents.ipynb`).
- Structure facilitates analytics and human review.

**Cons / Risks**
- Higher implementation effort (multiple agents, memo formatting, MCP servers).
- Requires governance of prompts/content to keep sections consistent.

**Use When**
Scaling to thousands of users with expectation of reusable components, robust analytics, and extensible scenarios.

---

## Option D – Advanced Multi-Agent Ecosystem (Best-in-Class)
**Concept**: Full agent mesh with domain-specialist modules, MCP microservices, deep research augmentation, voice channel, and auto-evals.

**Key Components**
- **Planner Hub**: reasoning model (e.g., `o4-mini`) orchestrating specialists as tools, with structured outputs enforcement (`Structured_outputs_multi_agent.ipynb`) for deterministic payloads.
- **Specialists**
    1. **Scenario Coach** – multi-turn planner using `SummarizingSession` + scenario knowledge graph; can launch clarification agent if info missing (pattern from Deep Research triage pipeline).
    2. **Academic Data Agent** – uses MCP servers for DB, vector search, calendar service; caches `mcp_list_tools` and enforces `allowed_tools` filters.
    3. **Career Insights Agent** – integrates external labor-market APIs via MCP; can trigger Deep Research agent for long-form reports when user opts in.
    4. **Compliance Guardrail** – output check using domain schema (similar to supply-chain guardrail) to filter sensitive guidance.
    5. **Voice Frontline Agent** – leverages voice orchestration from `app_assistant_voice_agents.ipynb` with persona controls, streaming UI, fallback to text.
- **Memory Tier**: hybrid—vector store per user (interests, quiz results), summary ledger, plus event sourcing for analytics; supports re-engagement campaigns.
- **Evaluation Fabric**: automated Langfuse datasets, LLM-as-judge scoring (toxicity, helpfulness), cost/latency dashboards.
- **Security**: object-oriented agents for code execution tasks (e.g., generating personalized study plans) run in Docker sandbox (`Secure_code_interpreter_tool_for_LLM_agents.ipynb`).
- **Deployment**: Azure Kubernetes with auto-scaling; caching (prompt + tool results) and budget guardrails to enforce monthly spend caps.

**Pros**
- Delivers premium experience: contextual memory, research add-ons, multi-channel interface, continuous quality monitoring.
- Extensible via MCP microservices; supports future features (voice calls, counselor handoff, analytics deep-dives).
- Resilient operations with guardrails, retries, sandboxed tooling, and comprehensive telemetry.

**Cons / Risks**
- Highest complexity and cost; needs dedicated engineering + MLOps team.
- Requires rigorous governance (data privacy, access control, scenario approval workflows).

**Use When**
Long-term flagship offering with 2026+ roadmap, targeting distinctive advisor-like experience and cross-team collaboration.

---

## Comparison Snapshot
| Dimension | Option A | Option B | Option C | Option D |
|-----------|----------|----------|----------|----------|
| Build time | 4–6 weeks | 8–12 weeks | 12–20 weeks | 6+ months phased |
| Long-term memory | No | Summaries + DB | Summaries + structured profile | Hybrid (vector + ledger) |
| Scenario agility | Prompt rules | RAG-driven | Specialist planner | Multi-agent graph |
| Web search | Manual fallback | Optional with guardrails | Prioritized official + fallback | Deep research + MCP integrations |
| Analytics | Basic GA4 events | GA4 + traces | GA4 + Langfuse + structured outputs | Full observability + eval loops |
| Risk mitigation | Minimal | Guardrails + retries | Guardrails + memo QA | Guardrails, compliance agent, sandboxed tools |
| OpEx per 1k monthly users | Low | Moderate (RAG + summaries) | Higher (multi-agent) | Highest (reasoning models, research, voice) |

---

## Recommendation
- **Near-term**: deliver Option B to satisfy core requirements (memory, scenarios, analytics) while keeping scope manageable; reuse Option A assets for quick wins.
- **Scale-up**: evolve toward Option C by modularizing agents and MCP services; this unlocks richer insights and structured outputs vital for product analytics.
- **Long-term vision**: adopt Option D capabilities selectively (e.g., Deep Research agent for specialized counseling, voice channel) aligned with budget and user demand.
