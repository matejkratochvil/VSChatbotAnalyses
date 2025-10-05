# Přehled know-how pro vývoj chatbotů a agentů

## Účel dokumentu
- Shrnuje klíčové postupy, architektury a doporučení z repozitářů v `resources/repos_cookbooks` a z odkazů v `resources/links-docs`.
- Slouží jako znalostní základna pro návrh chatbotu podle zadání ve `VSChatBot.md`.

## Struktura a poznatky z repozitářů

### OpenAI Cookbook (`resources/repos_cookbooks/openai-cookbook`)
- **Struktura projektu:**
  - `articles/` (např. `techniques_to_improve_reliability.md`, `gpt-oss/` pro lokální LLM).
  - `examples/agents_sdk/` (hlasový asistent, víceagentní portfolio, paměť, evaluace).
  - `examples/vector_databases/` (integrace s Azure AI Search, Pinecone, PostgreSQL aj.).
  - `examples/mcp/` a `examples/codex/` (MCP servery, agenti s nástroji).
  - `AGENTS.md`, `registry.yaml` popisují standardy a katalog obsahu.
- **Know-how:**
  - Agents SDK podporuje `Agent` + `Runner` s handoffy a orchestrací nástrojů (`examples/agents_sdk/dispute_agent.ipynb`).
  - Paměť se řeší trimmingem/sumarizací (`examples/agents_sdk/session_memory.ipynb`) s metrikami pro úsporu tokenů.
  - Víceagentní spolupráce, striktně definované výstupy a MCP servery (`examples/agents_sdk/multi-agent-portfolio-collaboration`, `examples/mcp/...`).
  - Evaluace a observabilita skrz Langfuse a OpenTelemetry (`examples/agents_sdk/evaluate_agents.ipynb`).
  - RAG pattern s funkcemi a embeddings (`examples/How_to_call_functions_for_knowledge_retrieval.ipynb`).

### Azure vzorová řešení (`resources/repos_cookbooks/azure`)
- **`agent-openai-python-banking-assistant/`:**
  - Struktura `src/`, `docs/`, `infra/`; popisuje vertikální multi-agentní architekturu (supervizor + specialisté), MCP nástroje nad REST API a OCR (Azure Document Intelligence).
  - Učí, jak Semantic Kernel Agent Framework konfiguruje nástroje, Azure Container Apps pro hostování a jak nasadit přes `azd`.
- **`Azure-Language-OpenAI-Conversational-Agent-Accelerator/`:**
  - Strom `docs/`, `src/`, `infra/`; orchestrátor kombinuje CLU, CQA, LLM s RAG a PII filtrováním; flexibilní volba orchestrací (intent router, funkce, Prompt Flow).
  - Důraz na determinismus, guardrails, škálovatelnost a analytiku (App Insights, více modalit).
- **`chat-with-your-data-solution-accelerator/`:**
  - `docs/`, `src/backend`, `src/frontend`; end-to-end RAG na Azure AI Search, ingestace dokumentů/webu, Admin UI, integrace řeč ↔ text.
  - Popisuje různé chunking strategie, volbu orchestrátoru (Semantic Kernel, LangChain, Prompt Flow) a nákladové scénáře.
- **`rag-postgres-openai-python/`:**
  - `src/backend`, `src/frontend`, `docs/`; ukazuje hybridní vyhledávání na PostgreSQL s pgvector, generování SQL přes funkce, autentizaci Managed Identity, logování do Log Analytics.
- **`BotBuilder-Samples/`:**
  - Katalog `samples/<jazyk>/` pro Bot Framework V4; démonstruje uvítací scénáře, vícestavové dialogy, adaptivní karty, stavovou správu, proaktivní zprávy, vícejazyčnou podporu a telemetry.
- **`openai/Agent_Based_Samples/`:**
  - Podsložky (`customer_assist/`, `market_research_analyst/`, `release_manager/` …) s `src/solution_accelerators`, `data/`, `TRANSPARENCY_FAQ.md`.
  - Sdílí architektury na Semantic Kernel Process Framework, multi-agent orchestrace, multimodální vstupy, bezpečnost (Content Safety), evaluace (Azure AI Evaluation SDK).
- **Obecné postřehy:**
  - Standardizace nasazení přes Azure Developer CLI (`azd`), infrastrukturní šablony (Bicep) a integrace s App Insights/GA.
  - Zajištění bezpečnosti: PII redakce, guardrails, Azure Content Safety, role-based access.
  - Doporučení na optimalizaci nákladů: nastavení kapacit modelů, možnost lokálních testů, využití menších modelů pro pomocné úlohy.

### Claude Cookbooks (`resources/repos_cookbooks/claude-cookbooks`)
- **Struktura:**
  - `patterns/agents/` (notebooky pro orchestrátor–worker, evaluator–optimizer, prompt chaining).
  - `tool_use/` (paměťové vzory, paralelní nástroje, Pydantic schémata, zákaznický agent, vizuální nástroje).
  - `skills/` (klasifikace, RAG, contextual embeddings, sumarizace, text-to-SQL).
  - `observability/`, `extended_thinking/`, `tool_evaluation/` (telemetrie, testování, dlouhé uvažování).
- **Know-how:**
  - Multi-agentní schémata s průběžným hodnocením kvality (`patterns/agents/evaluator_optimizer.ipynb`).
  - Paměťový cookbook ukazuje shrnování kontextu, externí úložiště vzpomínek a nástroje pro dlouhodobé profily uživatelů.
  - RAG průvodci zdůrazňují contextual embeddings a re-ranking pro přesnost dotazů.
  - Rozšířené sledování (observability) s event logy a metrikami pro ladění bezpečnosti.

### Claude Quickstarts (`resources/repos_cookbooks/claude-quickstarts`)
- `agents/` (minimální agent s <300 řádky; adresář `tools/` pro MCP i lokální nástroje; `agent_demo.ipynb` pro průchod).
- `customer-support-agent/` (Next.js + Claude + Amazon Bedrock; realtime debug UI, detekce nálady, vizualizace zdrojů znalostní báze).
- `financial-data-analyst/`, `computer-use-demo/` poskytují specializované scénáře (např. ovládání počítače s vizí).
- Vzory ukazují, jak rychle přepínat modely, konfigurovat knowledge bases a nasazovat do AWS prostředí.

### OpenAI CS Agents Demo (`resources/repos_cookbooks/openai-cs-agents-demo`)
- `python-backend/` (uvicorn + Agents SDK orchestrace se stráží relevance/jailbreak; modulární guardrails).
- `ui/` (Next.js vizualizace, sledování handoffů mezi specialisty; highlight triage → specialista).
- Důraz na demonstraci toků a přepínání agentů, snadno adaptovatelné pro jiné domény.

## Poznatky z externích odkazů (`resources/links-docs`)
- **Anthropic (Claude) dokumentace:**
  - `tool-use/overview`, `web-fetch-tool` – popisují deklarativní definici nástrojů, přístup k MCP serverům, prioritizaci autorizovaných zdrojů a bezpečnostní režimy.
  - `prompt-caching` vysvětluje, jak snižovat náklady opakovaným využitím systémových promptů a jak sledovat limity cache.
  - `use-case-guides/overview`, `embeddings`, `reduce-hallucinations` – doporučení pro guardrails, embedding pipeliny a minimalizaci halucinací (evaly, kontrola zdrojů).
- **Microsoft Learn (Azure AI Foundry, chat web app, open-source sample `openai-chat-your-own-data`):**
  - Architektury pro nasazení chat aplikací, využití Prompt Flow, orchestrátorů, App Service + Functions, Azure AI Search, integrace GA4/Application Insights.
  - Doporučení pro CI/CD, správy prostředí, AAD autentizaci a governance.
- **Google Dialogflow CX dokumentace:**
  - Tvorba agentů pomocí playbooku, datových handlerů, konverzační historie, rozšíření datových úložišť – ukazuje, jak strukturovat stavy, znovupoužití scénářů a fallback logiku.
- **Hugging Face Learn (Agents, smolagents, RAG s Neo4j, structured generation):**
  - Ukázky lehkých agentních frameworků s definicí nástrojů, pipeline pro znalostní grafy, generování strukturovaných odpovědí a evaluaci přes testovací datasety.
- **Azure Samples `openai-chat-your-own-data`:**
  - Rychlé nasazení chatbota s Azure AI Search, volba mezi "On Your Data" vs. custom orchestrací, konfigurace chunkingu a UI.
- **Google Agent Team tutorial:**
  - Přístup k týmové spolupráci agentů (triáž, plánovač, vykonavatel), definice rolí a koordinace přes sdílenou paměť.

## Klíčové vzory a doporučení
- **Orchestrace a role agentů:** Supervizor + specialisté (triáž → oborový expert) z Azure a OpenAI demo repo; možnost využít Semantic Kernel Process Framework nebo OpenAI Agents SDK.
- **Paměť:** Kombinace krátkodobé paměti (trimming, sumarizace) a dlouhodobých profilů v databázi; Claude memory cookbook a OpenAI session memory poskytují implementační šablony a metriky.
- **RAG a přístup k datům:**
  - Azure AI Search, PostgreSQL (pgvector), Bedrock Knowledge Bases; možnost contextual embeddings a hybridního vyhledávání.
  - Doporučené pipeline: ingestace → normalizace → chunking → indexace → vyhodnocení kvality.
- **Scénáře a determinismus:** Kombinace CLU/CQA (Azure accelerator) pro prioritní intent handling a fallback do LLM s RAG; BotBuilder ukazuje, jak udržet dialogové stavy a přepínat scénáře.
- **Nástroje a MCP:** Exponování firemních API jako MCP serverů (banking assistant), bezpečné volání externích nástrojů s autorizací.
- **Observabilita a evaluace:** App Insights/GA4+Langfuse/OpenTelemetry; sledování latence, nákladů, kvality odpovědí. Claude/Anthropic materiály doporučují evaluace na reálných transkriptech a guardrail metriky.
- **Bezpečnost a compliance:**
  - PII redakce (Azure AI Language), guardrails (relevance/jailbreak), moderace (Azure Content Safety, Claude reduce hallucinations).
  - Role-based přístup, audit logy, bezpečné ukládání klíčů (Key Vault, env vars).
- **Vícejazyčnost:** BotBuilder vzorky a Azure orchestrátory podporují překladové middleware; u LLM preferovat modely s kvalitní češtinou/slovenštinou (GPT-4o, Claude 3.5, Mistral CS varianty).
- **Latence a náklady:**
  - Prompt caching (Claude), partial caching v Agents SDK, paralelní volání nástrojů (OpenAI agenti, Claude parallel tools).
  - Volba menších modelů pro klasifikaci a pre-processing, streaming odpovědí pro lepší UX.
- **UI a integrace:** Next.js/React frontendy (Azure, OpenAI demo, Claude quickstart) s vizualizací průběhu; BotBuilder pro Teams/OmniChannel; voice rozšíření v OpenAI voice agent příkladech.
- **Testing & QA:** Automatizované evaly (Azure Evaluation SDK, Claude tool evaluation, LLM-as-judge bench), dataset replays a ruční audit kritických scénářů.

## Připravenost na projekt VSChatBot
- Výše uvedené zdroje pokrývají dlouhodobou paměť, dynamické scénáře, RAG nad doménovými daty, orchestraci více agentů, evaluaci i monitorování.
- Datové toky z `vs-weby.txt` lze mapovat na RAG pipeline (scraping → normalizace → indexace v Azure AI Search/PostgreSQL/Neo4j) podle vzorů z Azure a Hugging Face.
- Znalosti o guardrails, PII filtraci, GA4/App Insights a cost managementu podpoří požadavky na bezpečnost, analytiku a nákladovou efektivitu.
