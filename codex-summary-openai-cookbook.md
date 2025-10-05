# OpenAI Cookbook Agent Patterns – Working Notes

## Agents SDK Fundamentals
- `AGENTS.md` highlights repo conventions: runnable examples under `examples/`, long-form guidance in `articles/`, and emphasizes tracing, guardrails, and registry metadata for discoverability.
- Agents are defined with the SDK’s `Agent` class plus `Runner.run`; handoffs and tool orchestration are core primitives (`examples/agents_sdk/dispute_agent.ipynb`).
- Function tools come from decorating Python callables with `@function_tool`, auto-generating JSON schemas and enabling re-use across agents.
- Guardrails can wrap inputs/outputs to enforce domain limits (see `supply_chain_guardrails.py` in Databricks example) and should default to fail-closed behaviour.

## Tool Calling & Function Interfaces
- Chat Completions API supports declarative `tools` specs and `tool_choice` control (`examples/How_to_call_functions_with_chat_models.ipynb`).
- Patterns include: forced tool usage, inhibiting tools, multi-tool parallel calls, and iterative execution when arguments are incomplete.
- Knowledge retrieval workflow couples function-calling with embeddings/file stores and structured conversation state (`examples/How_to_call_functions_for_knowledge_retrieval.ipynb`).
- Node SDK tutorial (`examples/How_to_build_an_agent_with_the_node_sdk.mdx`) demonstrates browser-based agents, iterative tool planning loops, and managing function responses without backend services.

## Context & Memory Management
- Session objects (`examples/agents_sdk/session_memory.ipynb`) enable configurable short-term memory: trimming (`TrimmingSession`) to keep last *N* turns, or summarizing older turns via LLM-driven compression with quality prompts and evaluation.
- Guidance on choosing `max_turns`, crafting summary prompts, and evaluating memory strategies (LLM-as-judge, transcript replay, token pressure monitoring).

## Multi-Agent Orchestration
- Parallel execution via `asyncio.gather` vs “agents as tools” trade-offs (`examples/agents_sdk/parallel_agents.ipynb`): latency vs planner convenience.
- Dispute workflow (`examples/agents_sdk/dispute_agent.ipynb`) shows triage → specialized agents with handoffs, Stripe integration, and conditional tool usage.
- Portfolio collaboration demo (`examples/agents_sdk/multi-agent-portfolio-collaboration/`) layers orchestrator (PM) + specialist agents + memo editor, using MCP servers, code interpreter, retry logic, and guardrails.
- `Orchestrating_agents.ipynb` introduces routines vs handoffs, function-based agent transfers, and the Swarm library ethos.
- `Structured_outputs_multi_agent.ipynb` demonstrates schema-enforced outputs with `strict: true`, ensuring deterministic tool argument formats across agent tiers.

## Tooling, MCP, and External Systems
- MCP tool guide notebook documents hosted MCP usage, caching `mcp_list_tools`, restricting tool lists, and mixing MCP with other tools.
- Databricks supply-chain example (`examples/mcp/building-a-supply-chain-copilot-with-agent-sdk-and-databricks-mcp/`) integrates OAuth, vector search, UC functions, guardrails, FastAPI streaming, and React UI.
- Multi-agent portfolio MCP server (`mcp/yahoo_finance_server.py`) shows FastMCP design: logging, CSV/JSON exports, async timeouts, deduplicated file naming.
- Deep Research agents (`examples/deep_research_api/introduction_to_deep_research_api_agents.ipynb` & MCP README) describe triage/clarifier/instruction builder pipelines feeding `o3`/`o4` research models with search/fetch tools and streamed progress.

## Evaluation, Tracing, and Analytics
- Langfuse evaluation notebook (`examples/agents_sdk/evaluate_agents.ipynb`) connects Agents SDK spans to OpenTelemetry, enabling online metrics (latency, token cost, user feedback) plus offline dataset runs with LLM-as-judge scores.
- Tracing is deeply integrated (Agents SDK, Deep Research, Databricks, voice assistant) enabling debugging, observability dashboards, and logs for guardrails/tool retries.

## Voice & Multimodal Agents
- Voice assistant cookbook (`examples/agents_sdk/app_assistant_voice_agents.ipynb`) combines search, knowledge base, and account agents via triage + handoffs, then upgrades to voice (text-to-speech personalities, realtime streaming, audio tracing) and tone optimization.

## Safety & Secure Execution
- Object-oriented agentic approach (`examples/object_oriented_agentic_approach/`) defines reusable agent/tool classes, Docker-isolated code execution, file access proxies, and reasoning-effort controls for `o3`/`o4` models.
- Tool retry prompts enforce resilience: retry transient tool failures up to three times, log errors, and continue with partial data when appropriate.

## Additional Takeaways
- Prefer parallel tool calls when dependencies allow to control latency (fundamental/macro/quant prompts explicitly require batching requests).
- MCP and hosted tools benefit from caching (`previous_response_id`) to avoid repeated tool-list imports.
- Always reference generated files (CSV/PNG) by exact name and ensure outputs exist before citing (enforced in multi-agent memo workflow).
- Use disclaimer macros and consistent PM pushback sections to keep outputs aligned with organizational policy (e.g., `DISCLAIMER` constant in portfolio demo).
- Combine guardrails, tool schema validation, and evaluation suites to mitigate hallucinations and domain drift.
