# Návrh řešení AI chatbota "Chci na vejšku"

## Analýza požadavků a datových zdrojů

### Dostupná data
Z analýzy zdrojů (vs-weby.txt) jsou k dispozici:
- **Katalogy VŠ**: kampomaturite.cz, vysokeskoly.cz - komplexní přehled VŠ, fakult, studijních programů
- **Přijímací řízení**: scio.cz - informace o NSZ, podmínky přijetí
- **Oficiální registry**: MŠMT registr - akreditované programy, oficiální data
- **Metadata**: Dny otevřených dveří, kontakty, pracovní uplatnění absolventů

### Klíčové funkční požadavky
1. Dlouhodobá paměť (preference, historie, quiz výsledky)
2. Dynamické scénáře (3 kariérové scénáře + rozšíření)
3. Integrace s DB (priorita dat z DB)
4. Vícejazyčnost (CZ/SK)
5. Web search (volitelné, nízká priorita)
6. Analytika (GA4, konverze, zpětná vazba)
7. Optimalizace nákladů (cachování, efektivní využití tokenů)

---

## ŘEŠENÍ 1: MVP - Jednoduché řešení s promptovým řízením

### Popis architektury
**Přístup**: Monolitický chatbot s velkým system promptem obsahující všechny scénáře a instrukce.

**Technologie**:
- **LLM**: Claude 3.5 Sonnet nebo GPT-4o
- **Backend**: Azure Functions (serverless)
- **Databáze**:
  - Azure SQL Database (strukturovaná data VŠ)
  - Azure Cosmos DB (ConversationState + Memory)
- **Frontend**: Jednoduchý chat widget (React)
- **Hosting**: Azure App Service

**Architektura**:
```
User (Web)
  ↓
Chat Widget (React)
  ↓
Azure Functions (API)
  ↓ (1) Načte kontext z DB
  ↓ (2) Sestaví prompt s historií
  ↓
Claude API (s prompt caching)
  ↓
Response → Uložení do Cosmos DB
  ↓
User
```

### Implementace funkčních požadavků

| Požadavek | Implementace MVP |
|-----------|------------------|
| **Dlouhodobá paměť** | Cosmos DB - dokumenty per user (id, preferences, conversationHistory[], quizResults, visitedPrograms[]) |
| **Scénáře** | Hardcoded v system promptu jako structured instructions. Chatbot trackuje aktuální krok scénáře v conversation state. |
| **DB integrace** | Function volá SQL Database přes direct queries (např. "najdi VŠ s oborem informatika v Brně") |
| **Web search** | Neimplementováno v MVP |
| **Multi-language** | Detection z prvního user message, system prompt má sekce CZ/SK |
| **Analytika** | Custom events do GA4 přes gtag (conversation_started, conversation_completed, feedback_submitted) |
| **Optimalizace** | Prompt caching: system prompt + DB schema + 3 scénáře (stabilní části) |

### Odhad tokenů a nákladů

**Předpoklady:**
- Průměrná konverzace: 10-15 minut = 20 zpráv (10 user + 10 assistant)
- Uživatel se vrací 3x měsíčně (rozhodování je proces)
- Cache hit rate: 80% (system prompt se mění jednou za měsíc)

**Token breakdown per konverzace:**

| Komponenta | Input tokens | Output tokens | Poznámka |
|------------|--------------|---------------|----------|
| System prompt + scénáře | 3,000 | - | Cachováno (5min TTL) |
| DB schema | 500 | - | Cachováno |
| User messages (10x) | 1,500 | - | Průměrně 150 tokenů/msg |
| Conversation history | 2,000 | - | Roste během konverzace |
| DB query results | 1,000 | - | Průměrně 2-3 queries |
| Assistant responses | - | 3,000 | 300 tokenů/response |
| **CELKEM** | **8,000** | **3,000** | |

**S prompt cachingem (80% hit rate):**
- Cache write (20% času): 3,500 tokens @ $3.75/MTok = $0.013
- Cache read (80% času): 3,500 tokens @ $0.30/MTok = $0.001
- Regular input: 4,500 tokens @ $3/MTok = $0.014
- Output: 3,000 tokens @ $15/MTok = $0.045
- **Náklady per konverzace: ~$0.06**

**Měsíční náklady na 1 uživatele:**
- 3 konverzace/měsíc × $0.06 = **$0.18/měsíc**
- Long-term memory storage: Cosmos DB ~$0.01/měsíc
- **Celkem: ~$0.19/uživatel/měsíc**

**Náklady při 1,000 uživatelích:**
- Variabilní: 1,000 × $0.19 = **$190/měsíc**
- Fixní:
  - Azure Functions (Consumption): $10/měsíc
  - Azure SQL Database (Basic): $5/měsíc
  - Cosmos DB (400 RU/s): $24/měsíc
  - App Service (Basic B1): $13/měsíc
- **Fixní celkem: ~$52/měsíc**
- **TOTAL: ~$242/měsíc při 1,000 uživatelích**

### Nacenění vývoje

| Fáze | Člověkodny | Náklady (@ $500/den) |
|------|------------|----------------------|
| Analýza a design | 3 | $1,500 |
| Setup infrastruktury (Azure) | 2 | $1,000 |
| DB design a migrace dat | 5 | $2,500 |
| Backend API (Functions) | 8 | $4,000 |
| Prompt engineering (3 scénáře) | 5 | $2,500 |
| Frontend (chat widget) | 5 | $2,500 |
| Integrace GA4 | 2 | $1,000 |
| Testing a QA | 5 | $2,500 |
| Deployment a dokumentace | 2 | $1,000 |
| **CELKEM** | **37 dní** | **$18,500** |

### Výhody MVP
✅ Rychlý vývoj (6-8 týdnů)
✅ Nízké náklady
✅ Jednoduché údržba
✅ Dostačující pro 320 uživatelů (rok 1)

### Nevýhody MVP
❌ Obtížné škálování scénářů (všechno v jednom promptu)
❌ Bez web search
❌ Kontext může růst příliš (řešení: summarizace každých 10 zpráv)
❌ Nedostatečná modularita pro složitější rozhodovací stromy

---

## ŘEŠENÍ 2: INTERMEDIATE - RAG s vektorovou DB

### Popis architektury
**Přístup**: Chatbot s RAG (Retrieval-Augmented Generation) pro efektivnější práci se znalostní bází VŠ.

**Technologie**:
- **LLM**: Claude 3.5 Sonnet (primární) + Claude 3 Haiku (pro embeddingy a jednoduché queries)
- **Backend**: Azure Container Apps (lepší pro RAG pipeline)
- **Databáze**:
  - Azure SQL Database (strukturovaná data)
  - Azure AI Search (vector DB pro RAG)
  - Azure Cosmos DB (conversation state)
- **Framework**: LangChain nebo Semantic Kernel
- **Embedding model**: text-embedding-3-large (OpenAI) nebo custom
- **Hosting**: Azure

**Architektura**:
```
User
  ↓
Chat Widget
  ↓
API Gateway
  ↓
┌──────────────────────────────┐
│ Orchestration Layer          │
│ (LangChain/Semantic Kernel)  │
├──────────────────────────────┤
│  1. Query Understanding      │
│     (Claude Haiku)           │
│  2. Memory Retrieval         │
│  3. RAG Retrieval            │
│     (Azure AI Search)        │
│  4. DB Query (if needed)     │
│  5. Response Generation      │
│     (Claude Sonnet + cache)  │
└──────────────────────────────┘
  ↓
Response + Memory Update
```

**Knowledge Base struktura**:
- **Chunked documents**: Každá VŠ/fakulta/obor = samostatný dokument
- **Metadata**: typ (VŠ/fakulta/obor), město, forma studia, jazyk, NSZ
- **Vector embeddings**: 1536 dimensions (OpenAI ada-003)

### Implementace funkčních požadavků

| Požadavek | Implementace RAG |
|-----------|------------------|
| **Dlouhodobá paměť** | Stejné jako MVP + embedding user preferences pro semantic search |
| **Scénáře** | Strukturované jako separate agents/chains v LangChain. Lze je dynamically načítat z DB/config file. |
| **DB integrace** | Hybrid: RAG pro exploratorní queries + SQL pro specifické data (datum DOD apod.) |
| **Web search** | **Ano** - Tavily API nebo Bing Search jako fallback když RAG nemá data |
| **Multi-language** | Embeddings podporují CZ/SK, response generation podle detekovaného jazyka |
| **Analytika** | Rozšířená: tracking RAG relevance scores, query types, scenario completion rates |
| **Optimalizace** | Multi-level caching: (1) System prompt, (2) RAG context window, (3) User memory summary |

### Odhad tokenů a nákladů

**Token breakdown per konverzace:**

| Komponenta | Input tokens | Output tokens | Poznámka |
|------------|--------------|---------------|----------|
| System prompt | 2,000 | - | Cachováno |
| Scenario definitions (dynamic) | 1,500 | - | Cachováno |
| User messages | 1,500 | - | |
| Conversation history (summarized) | 1,000 | - | Auto-summarize každých 8 zpráv |
| RAG retrieved context (avg 3 chunks) | 2,500 | - | Top-k=3, 800 tokens/chunk |
| DB query results | 800 | - | Targeted queries |
| User memory summary | 300 | - | Cachováno po load |
| Assistant responses | - | 3,000 | |
| **CELKEM** | **9,600** | **3,000** | |

**Dodatečné náklady:**
- **Embedding API calls**: 10 queries/konverzace × 200 tokens = 2,000 tokens @ $0.13/MTok = $0.0003
- **Web search** (30% konverzací): 0.3 × $0.02 = $0.006

**S prompt cachingem (85% hit rate díky lepší struktuře):**
- Cache write: 3,800 tokens @ $3.75/MTok = $0.014
- Cache read: 3,800 tokens @ $0.30/MTok = $0.001
- Regular input: 5,800 tokens @ $3/MTok = $0.017
- Output: 3,000 tokens @ $15/MTok = $0.045
- Embeddings + search: $0.006
- **Náklady per konverzace: ~$0.07**

**Měsíční náklady na 1 uživatele:**
- 3 konverzace/měsíc × $0.07 = **$0.21/měsíc**
- Storage (Cosmos + AI Search): $0.02/měsíc
- **Celkem: ~$0.23/uživatel/měsíc**

**Náklady při 1,000 uživatelích:**
- Variabilní: 1,000 × $0.23 = **$230/měsíc**
- Fixní:
  - Azure Container Apps: $30/měsíc
  - Azure SQL Database (Standard S1): $15/měsíc
  - Azure AI Search (Basic): $75/měsíc
  - Cosmos DB (1000 RU/s): $60/měsíc
- **Fixní celkem: ~$180/měsíc**
- **TOTAL: ~$410/měsíc při 1,000 uživatelích**

### Nacenění vývoje

| Fáze | Člověkodny | Náklady (@ $500/den) |
|------|------------|----------------------|
| Analýza a design (RAG architecture) | 4 | $2,000 |
| Setup infrastruktury (Azure) | 3 | $1,500 |
| Knowledge base creation & chunking | 8 | $4,000 |
| Vector DB setup (Azure AI Search) | 3 | $1,500 |
| RAG pipeline (LangChain/SK) | 10 | $5,000 |
| Prompt engineering (scénáře jako chains) | 6 | $3,000 |
| Backend API | 8 | $4,000 |
| Frontend | 5 | $2,500 |
| Web search integration | 3 | $1,500 |
| Testing RAG relevance & QA | 7 | $3,500 |
| Deployment a dokumentace | 3 | $1,500 |
| **CELKEM** | **60 dní** | **$30,000** |

### Výhody RAG
✅ Efektivnější retrieval velkých znalostních bází
✅ Web search pro aktuální data
✅ Lepší škálovatelnost scénářů
✅ Modularita (lze přidat nové zdroje)
✅ Relevance scoring pro kvalitu odpovědí

### Nevýhody RAG
❌ Vyšší složitost
❌ Vyšší fixní náklady (AI Search)
❌ Nutnost údržby embeddings při změnách dat
❌ Latence (embedding + search + LLM)

---

## ŘEŠENÍ 3: PROFESSIONAL - Multi-agent systém s MCP

### Popis architektury
**Přístup**: Specializované agenty pro různé úkoly, orchestrace přes master agent, MCP servery pro external tools.

**Technologie**:
- **LLM**:
  - Claude 3.5 Sonnet (master agent, career counselor)
  - Claude 3 Haiku (specialist agents, routing)
- **Framework**: Google ADK nebo custom multi-agent s LangGraph
- **Backend**: Azure Kubernetes Service (microservices)
- **Databáze**:
  - Azure SQL Database (strukturovaná data)
  - Azure AI Search (vector DB)
  - Azure Cosmos DB (state & memory)
  - Redis Cache (session state)
- **MCP Servers**:
  - MCP Web Fetch (web search)
  - MCP Database (SQL queries)
  - MCP Analytics (GA4 events)
- **Monitoring**: Azure Application Insights
- **Hosting**: Azure

**Agent architektura**:
```
┌─────────────────────────────────────────┐
│        Master Agent (Orchestrator)       │
│         Claude 3.5 Sonnet                │
│  - Route queries to specialist agents    │
│  - Maintain conversation context         │
│  - Synthesize final responses            │
└─────────────────────────────────────────┘
            ↓  ↓  ↓  ↓
    ┌───────┴──┴──┴──┴────────┐
    │                          │
┌───▼────┐  ┌────▼────┐  ┌────▼─────┐  ┌────▼────┐
│ Career │  │ Search  │  │ Program  │  │ Memory  │
│ Agent  │  │ Agent   │  │ Compare  │  │ Agent   │
│(Sonnet)│  │(Haiku)  │  │ Agent    │  │(Haiku)  │
│        │  │         │  │(Haiku)   │  │         │
│3 scén. │  │RAG+Web  │  │Multi-VŠ  │  │Load/Save│
└────────┘  └─────────┘  └──────────┘  └─────────┘
     │           │              │            │
     └───────────┴──────────────┴────────────┘
                      ↓
            ┌──────────────────┐
            │   MCP Servers    │
            ├──────────────────┤
            │ - web-fetch      │
            │ - database       │
            │ - analytics      │
            └──────────────────┘
```

**Agenti:**
1. **Master Agent (Orchestrator)**: Řídí conversation flow, deleguje na specialty agents
2. **Career Counselor Agent**: Provádí 3 kariérové scénáře (interviewing studenta)
3. **Search Agent**: RAG + web search pro informace o VŠ
4. **Program Comparison Agent**: Porovnává více VŠ/oborů podle kritérií
5. **Memory Agent**: Spravuje long-term memory (load preferences, save insights)
6. **Analytics Agent**: Trackuje events do GA4

### Implementace funkčních požadavků

| Požadavek | Implementace Multi-Agent |
|-----------|--------------------------|
| **Dlouhodobá paměť** | Dedikovaný Memory Agent + embeddings user preferences + conversation summaries. Hierarchical memory (working, short-term, long-term). |
| **Scénáře** | Career Counselor Agent má 3 scénáře jako state machines. Master agent deleguje na něj když user chce guidance. Scénáře lze přidat jako nové sub-agents. |
| **DB integrace** | MCP Database Server - structured queries přes SQL, agents volají MCP tool "query_database" |
| **Web search** | MCP Web Fetch Server - Search Agent používá web_search tool jako fallback |
| **Multi-language** | Detection na úrovni Master Agenta, všichni agenti dostanou language context |
| **Analytika** | MCP Analytics Server - každý agent může emitovat events (career_scenario_started, program_compared, etc.) |
| **Optimalizace** | Multi-level: (1) Per-agent prompt caching, (2) Master agent cachuje agent descriptions, (3) Haiku pro simple tasks = 10x cheaper |

### Odhad tokenů a nákladů

**Token breakdown per konverzace:**

| Komponenta | Input tokens | Output tokens | Model | Poznámka |
|------------|--------------|---------------|-------|----------|
| **Master Agent** | | | Sonnet | |
| - System prompt + agent registry | 2,500 | - | | Cachováno |
| - Conversation context | 1,500 | - | | |
| - Agent responses (synthesis) | 1,000 | - | | |
| - Final responses (5x) | - | 1,500 | | |
| **Career Agent** (3 calls/conv) | | | Sonnet | |
| - Scenario definitions | 1,500 | - | | Cachováno per scenario |
| - User inputs | 600 | - | | |
| - Responses | - | 1,200 | | |
| **Search Agent** (2 calls) | | | Haiku | |
| - RAG context | 1,600 | - | | |
| - Query synthesis | - | 400 | | |
| **Program Compare** (1 call) | | | Haiku | |
| - Comparison data | 800 | - | | |
| - Table generation | - | 300 | | |
| **Memory Agent** (2 calls) | | | Haiku | |
| - Load/save operations | 400 | 200 | | |
| **CELKEM** | **~10,000** | **~3,600** | | Mixed models |

**Cost calculation (weighted by model):**
- **Sonnet usage**: 7,100 input + 2,700 output
  - Input: 7,100 @ $3/MTok (with 85% cache) = $0.015
  - Output: 2,700 @ $15/MTok = $0.041
- **Haiku usage**: 2,900 input + 900 output
  - Input: 2,900 @ $0.25/MTok = $0.0007
  - Output: 900 @ $1.25/MTok = $0.001
- **Embeddings**: 2,500 tokens @ $0.13/MTok = $0.0003
- **MCP Web Fetch**: 0.2 × $0.02 = $0.004
- **MCP calls overhead**: $0.001
- **Náklady per konverzace: ~$0.062**

**Měsíční náklady na 1 uživatele:**
- 3 konverzace/měsíc × $0.062 = **$0.186/měsíc**
- Storage: $0.03/měsíc (více dat kvůli agent state)
- **Celkem: ~$0.22/uživatel/měsíc**

**Náklady při 1,000 uživatelích:**
- Variabilní: 1,000 × $0.22 = **$220/měsíc**
- Fixní:
  - Azure Kubernetes Service (2 nodes): $140/měsíc
  - Azure SQL Database (Standard S2): $30/měsíc
  - Azure AI Search (Standard S1): $250/měsíc
  - Cosmos DB (2500 RU/s): $150/měsíc
  - Redis Cache (Basic C1): $15/měsíc
  - Application Insights: $20/měsíc
- **Fixní celkem: ~$605/měsíc**
- **TOTAL: ~$825/měsíc při 1,000 uživatelích**

### Nacenění vývoje

| Fáze | Člověkodny | Náklady (@ $500/den) |
|------|------------|----------------------|
| Analýza a design (multi-agent architecture) | 6 | $3,000 |
| Infrastructure setup (AKS, microservices) | 5 | $2,500 |
| MCP servers implementation (3 servery) | 12 | $6,000 |
| Knowledge base & RAG pipeline | 8 | $4,000 |
| Master Agent + orchestration | 10 | $5,000 |
| Career Counselor Agent (3 scénáře) | 8 | $4,000 |
| Specialist Agents (Search, Compare, Memory) | 12 | $6,000 |
| Inter-agent communication & state mgmt | 8 | $4,000 |
| Backend APIs & integration | 8 | $4,000 |
| Frontend (advanced chat UI) | 6 | $3,000 |
| Analytics & monitoring (GA4 + App Insights) | 4 | $2,000 |
| Testing (unit, integration, agent behavior) | 10 | $5,000 |
| Security & compliance (GDPR) | 4 | $2,000 |
| Deployment pipelines (CI/CD) | 4 | $2,000 |
| Dokumentace & training | 4 | $2,000 |
| **CELKEM** | **109 dní** | **$54,500** |

### Výhody Multi-Agent
✅ **Nejvyšší škálovatelnost** - lze přidat nové agenty bez úpravy existujících
✅ **Modularita** - každý agent lze vyladit/testovat samostatně
✅ **Cost efficiency** - Haiku pro jednoduché tasky = výrazné úspory
✅ **Professional monitoring** - Application Insights, detailed logging
✅ **Nejlepší user experience** - specializace = kvalitnější odpovědi
✅ **Extensibility** - MCP servery lze snadno přidat (např. email notifications)

### Nevýhody Multi-Agent
❌ **Nejvyšší složitost** - více bodů selhání
❌ **Vyšší latence** - více API calls, orchestration overhead
❌ **Vysoké fixní náklady** - $605/měsíc base
❌ **Náročnější údržba** - více komponent k monitoringu
❌ **Overkill pro malé objemy** - není ekonomické pod 500 uživateli

---

## Srovnání řešení - Souhrnná tabulka

| Kritérium | MVP | RAG Intermediate | Multi-Agent Professional |
|-----------|-----|------------------|--------------------------|
| **Složitost vývoje** | Nízká | Střední | Vysoká |
| **Doba vývoje** | 6-8 týdnů | 10-12 týdnů | 16-20 týdnů |
| **Náklady vývoje** | $18,500 | $30,000 | $54,500 |
| **Fixní náklady/měsíc** | $52 | $180 | $605 |
| **Var. náklady/user/měsíc** | $0.19 | $0.23 | $0.22 |
| **Total @ 320 users** | $113/měsíc | $254/měsíc | $675/měsíc |
| **Total @ 1,000 users** | $242/měsíc | $410/měsíc | $825/měsíc |
| **Total @ 2,000 users** | $432/měsíc | $640/měsíc | $1,045/měsíc |
| **Web search** | ❌ | ✅ | ✅ |
| **Škálovatelnost scénářů** | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Kvalita odpovědí** | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Latence (průměrná)** | 2-3s | 3-5s | 4-7s |
| **Vhodné pro roky** | Rok 1 | Rok 1-2 | Rok 2-3+ |

---

## Evaluace frameworků a platforem

### 1. Azure AI Foundry (Azure AI Studio)

**Popis**: Komplexní platforma od Microsoftu pro vývoj AI aplikací s agentic capabilities.

**Klíčové features:**
- Agent Service pro orchestraci multi-agent systemů
- Integrovaný Azure AI Search (RAG)
- Model catalog (OpenAI, Claude přes partnery)
- Prompt Flow pro vizuální orchestraci
- Monitoring & tracing built-in

**Fit pro naše požadavky:**
| Požadavek | Hodnocení | Poznámka |
|-----------|-----------|----------|
| Long-term memory | ✅ Výborné | Cosmos DB integrace, conversation history |
| Scénáře | ✅ Výborné | Prompt Flow - vizuální scenario builder |
| DB integrace | ✅ Výborné | Native Azure SQL connector |
| Web search | ✅ Ano | Bing Search API integrace |
| CZ/SK podpora | ✅ Ano | GPT-4o a Claude mají dobrou podporu |
| GA4 analytics | ⚠️ Custom | Vyžaduje custom integration |
| Azure hosting | ✅ Native | Vše v Azure ekosystému |

**Náklady:**
- Žádné platformové poplatky (pay-per-use modelu)
- Model consumption podle použití
- Azure services (Cosmos, SQL, Search) podle běžného ceníku

**Development complexity:** ⭐⭐⭐ (Střední - vyžaduje znalost Azure)

**Low-code vs Code:** Hybrid - Prompt Flow je low-code, ale lze extended code-first

**Doporučení pro architekturu:**
- **MVP**: ❌ Overkill
- **RAG**: ✅ Ideální - Prompt Flow + AI Search
- **Multi-agent**: ✅ Perfektní - Agent Service je navržen právě pro toto

**Vendor lock-in:** ⚠️ Vysoký - těžká migrace z Azure ekosystému

---

### 2. Microsoft Copilot Studio

**Popis**: Low-code platforma pro vytváření konverzačních AI botů, nástupce Power Virtual Agents.

**Klíčové features:**
- Drag-and-drop bot builder
- Integrated topics (scénáře)
- Power Automate flows
- Dataverse integrace
- Multi-language support
- Analytics dashboard built-in

**Fit pro naše požadavky:**
| Požadavek | Hodnocení | Poznámka |
|-----------|-----------|----------|
| Long-term memory | ⚠️ Omezené | Dataverse vars, ne complex memory |
| Scénáře | ✅ Výborné | Topics = přesně pro scénáře |
| DB integrace | ⚠️ Omezené | Jen přes Power Automate flows |
| Web search | ✅ Ano | Bing connector |
| CZ/SK podpora | ✅ Ano | Multi-language support |
| GA4 analytics | ❌ Ne | Vlastní analytics, bez GA4 |
| Azure hosting | ✅ Ano | Microsoft cloud |

**Náklady:**
- $200/měsíc za tenant + $20/měsíc per user (limitované sessions)
- Nebo consumption-based: $0.01 per conversation

**Development complexity:** ⭐ (Velmi nízká - low-code)

**Low-code vs Code:** Pure low-code (lze extend přes Power Automate)

**Doporučení pro architekturu:**
- **MVP**: ✅ Rychlé prototypování
- **RAG**: ❌ Omezené RAG capabilities
- **Multi-agent**: ❌ Není navrženo pro multi-agent

**Vendor lock-in:** ⚠️⚠️ Velmi vysoký - Microsoft ekosystém only

**Limitace:**
- Omezená flexibilita pro komplexní memory
- Nepodporuje embeddings/RAG nativně
- Obtížná integrace s GA4
- **Nevhodné pro naše požadavky** - příliš omezené pro long-term memory a RAG

---

### 3. Google Dialogflow CX + Playbooks

**Popis**: Pokročilá konverzační AI platforma s podporou playbooks pro structured scenarios.

**Klíčové features:**
- Playbooks (scenario definitions)
- Flows & Pages (conversation modeling)
- Data stores (RAG integrace)
- Multi-language NLU
- Vertex AI integration
- Rich analytics

**Fit pro naše požadavky:**
| Požadavek | Hodnocení | Poznámka |
|-----------|-----------|----------|
| Long-term memory | ⚠️ Omezené | Session parameters, ne long-term |
| Scénáře | ✅✅ Excelentní | Playbooks jsou právě pro toto |
| DB integrace | ✅ Ano | Webhooks k Cloud SQL / Firebase |
| Web search | ✅ Ano | Data store + web search |
| CZ/SK podpora | ✅ Ano | NLU podporuje češtinu |
| GA4 analytics | ✅ Ano | Native Google Analytics integrace |
| Azure hosting | ❌ Ne | Google Cloud only |

**Náklady:**
- Text requests: $0.007 per request
- Playbooks: $0.01 per playbook invocation
- Data store queries: $0.002 per query
- **Estimate @ 1,000 users (3 conv/měsíc)**: ~$300-400/měsíc

**Development complexity:** ⭐⭐ (Nízká-střední)

**Low-code vs Code:** Hybrid - GUI builder + custom fulfillment webhooks

**Doporučení pro architekturu:**
- **MVP**: ✅ Dobrá volba - rychlé, playbooks out-of-box
- **RAG**: ✅ Velmi dobré - Data stores support
- **Multi-agent**: ⚠️ Možné, ale ne native multi-agent orchestration

**Vendor lock-in:** ⚠️⚠️ Vysoký - Google Cloud ekosystém

**PROBLÉM: Azure hosting preference** - Dialogflow CX běží pouze na Google Cloud, což je konflikt s požadavkem na Azure hosting.

---

### 4. Google ADK (Agent Development Kit)

**Popis**: Framework pro budování multi-agent AI systemů s Gemini modely.

**Klíčové features:**
- Multi-agent orchestration
- Specialized agent teams
- Tool calling (funkce)
- Vertex AI integration
- Flexible agent architecture

**Fit pro naše požadavky:**
| Požadavek | Hodnocení | Poznámka |
|-----------|-----------|----------|
| Long-term memory | ✅ Ano | Custom implementation |
| Scénáře | ✅ Ano | Agent-per-scenario pattern |
| DB integração | ✅ Ano | Custom tools/functions |
| Web search | ✅ Ano | Google Search API |
| CZ/SK podpora | ✅ Ano | Gemini modely |
| GA4 analytics | ✅ Ano | Custom integration |
| Azure hosting | ⚠️ Možné | ADK lze hostovat kdekoli (containerized) |

**Náklady:**
- Framework je open-source (zdarma)
- Platí se jen za Gemini API calls
- Gemini 1.5 Pro: ~$3.50/MTok input, $10.50/MTok output
- **Comparable s Claude/GPT pricing**

**Development complexity:** ⭐⭐⭐⭐ (Vysoká - code-first)

**Low-code vs Code:** Pure code (Python/Node.js)

**Doporučení pro architekturu:**
- **MVP**: ❌ Overkill
- **RAG**: ⚠️ Možné, ale vyžaduje custom RAG
- **Multi-agent**: ✅✅ Ideální - navrženo právě pro multi-agent

**Vendor lock-in:** ⭐ Nízký - lze migrovat modely (Gemini → Claude/GPT)

**Výhoda:** Framework lze hostovat na Azure (containerized), jen API calls jdou na Google.

---

### 5. Custom Solution - LangChain/LangGraph na Azure

**Popis**: Code-first přístup s open-source frameworky, maximální flexibilita.

**Stack:**
- **Orchestration**: LangChain (RAG) nebo LangGraph (multi-agent)
- **LLM**: Claude 3.5 Sonnet (Anthropic API) nebo Azure OpenAI
- **Vector DB**: Azure AI Search nebo Pinecone
- **Backend**: FastAPI/Flask na Azure Container Apps
- **Database**: Azure SQL + Cosmos DB
- **Monitoring**: LangSmith + Azure App Insights

**Fit pro naše požadavky:**
| Požadavek | Hodnocení | Poznámka |
|-----------|-----------|----------|
| Long-term memory | ✅✅ Perfektní | Full control, custom implementation |
| Scénáře | ✅✅ Perfektní | LangGraph state machines |
| DB integrace | ✅✅ Perfektní | Direct SQL/ORM control |
| Web search | ✅ Ano | Tavily, SerpAPI, custom |
| CZ/SK podpora | ✅ Ano | Model dependent |
| GA4 analytics | ✅✅ Perfektní | Custom event tracking |
| Azure hosting | ✅✅ Perfektní | Full Azure deployment |

**Náklady:**
- Frameworks zdarma (open-source)
- LLM API dle consumption
- Azure services dle use
- **Comparable s přímými API calls + infrastructure**

**Development complexity:** ⭐⭐⭐⭐⭐ (Velmi vysoká - all custom)

**Low-code vs Code:** Pure code (Python primary)

**Doporučení pro architekturu:**
- **MVP**: ✅ Možné, ale možná zbytečně složité
- **RAG**: ✅✅ Výborné - LangChain RAG je industry standard
- **Multi-agent**: ✅✅ Perfektní - LangGraph multi-agent je state-of-art

**Vendor lock-in:** ⭐ Minimální - lze měnit všechny komponenty

**Výhody:**
- Maximální flexibilita
- Žádné platformové fees
- Best-in-class frameworks
- Velká komunita & dokumentace

**Nevýhody:**
- Nejvyšší development effort
- Nutnost vlastního DevOps
- Odpovědnost za security & scaling

---

## Framework Recommendation Matrix

| Framework | MVP | RAG Intermediate | Multi-Agent Pro | Azure Hosting | Lock-in Risk | Dev Time |
|-----------|-----|------------------|-----------------|---------------|--------------|----------|
| **Azure AI Foundry** | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ✅✅ Native | ⚠️ High | Medium |
| **Copilot Studio** | ⭐⭐⭐⭐ | ⭐⭐ | ⭐ | ✅ Yes | ⚠️⚠️ Very High | Low |
| **Dialogflow CX** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ❌ No (GCP) | ⚠️⚠️ High | Low-Med |
| **Google ADK** | ⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⚠️ Possible | ⭐ Low | High |
| **LangChain/Graph** | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ✅✅ Full | ⭐ Minimal | Very High |

---

## Finální doporučení

### Pro rychlý start (Rok 1 - do ledna 2026):

**🏆 DOPORUČENÍ: Azure AI Foundry + RAG Intermediate**

**Proč:**
1. ✅ **Timeline**: Lze stihnout do ledna 2026 (3 měsíce vývoje)
2. ✅ **Azure native**: Splňuje preferenci hostingu
3. ✅ **RAG out-of-box**: Azure AI Search integrace šetří čas
4. ✅ **Škálovatelnost**: Pokryje 320 users (rok 1) i 1,000 (rok 2)
5. ✅ **Prompt Flow**: Vizuální editor pro 3 scénáře = rychlejší iterace s kariérovými poradci
6. ✅ **Cost efficiency**: $254/měsíc při 320 users je přijatelné

**Implementační plán:**
- **Týden 1-2**: Setup Azure infrastruktury (AI Foundry, SQL, Cosmos, AI Search)
- **Týden 3-4**: Data ingestion (scraping/API z vs-weby.txt zdrojů) + chunking
- **Týden 5-7**: Prompt Flow scenarios (3 career counseling flows)
- **Týden 8-9**: RAG pipeline tuning (relevance, chunking strategy)
- **Týden 10-11**: Frontend chat widget + Azure integration
- **Týden 12**: Testing, GA4 integration, deployment
- **Týden 13**: Beta testing s reálnými studenty, fixes

**Alternative (pokud je Azure AI Foundry příliš complex):**
- **LangChain RAG** - větší control, stejná funkcionalita, o 2 týdny delší vývoj

---

### Pro dlouhodobý růst (Rok 2-3):

**🏆 DOPORUČENÍ: Migrace na Multi-Agent (LangGraph na Azure)**

**Kdy migrovat:**
- Při dosažení 500+ aktivních uživatelů měsíčně
- Když fixní náklady $605/měsíc budou proporcionální k počtu userů
- Pokud budete chtít přidat 4.+ scénář nebo komplexnější decision trees

**Proč LangGraph a ne Azure AI Foundry Agent Service:**
- LangGraph má lepší state management pro komplexní multi-turn scenarios
- Nižší vendor lock-in
- Větší flexibilita pro custom agents
- Aktivní komunita & best practices

---

## Guardrails proti zneužití

**Problém:** Studenti by mohli používat chatbota jako generic ChatGPT.

**Řešení:**

1. **System prompt guardrails**:
   ```
   You are a university selection counselor ONLY for Czech/Slovak universities.
   You MUST NOT answer questions unrelated to:
   - University selection, study programs, admission requirements
   - Career counseling related to higher education
   - Comparison of universities/faculties

   If user asks off-topic (homework, general knowledge, etc.), respond:
   "Jsem specializován pouze na výběr vysoké školy. Pro tuto otázku doporučuji použít obecný AI asistent."
   ```

2. **Rate limiting**:
   - Max 5 konverzací per user per den
   - Max 50 zpráv per konverzace
   - Cooldown 1 minuta mezi konverzacemi

3. **Monthly spend cap (Free tier rok 1)**:
   - $500/měsíc hard cap
   - Alert při 80% ($400)
   - Automatic pause služby při dosažení

4. **Semantic monitoring**:
   - Azure Content Safety API - detekuje off-topic queries
   - Flag users s >50% off-topic rate
   - Progressive restrictions (warning → 24h ban → permanent)

5. **Session timeout**:
   - Auto-close po 20 minutách neaktivity
   - Uložení konverzace, ale uvolnění resources

**Implementační náklady guardrails:** +$500 dev + ~$10/měsíc (Content Safety API)

---

## Poznámky k tokenům a cache

### Cache considerations

**Prompt cache limits (Claude)**:
- Max cache size: 100K tokens
- TTL: 5 minut (default), 1 hodina (extended)

**Naše strategie:**
- **Tier 1 (always cached, 5min TTL)**: System prompt + agent descriptions = 3K tokens
- **Tier 2 (session cache, extended 1h)**: DB schema + current scenario + user memory = 5K tokens
- **Tier 3 (no cache)**: Dynamic RAG results, user messages

**Pokud konverzace překročí 1h:**
- Auto-summarization každých 15 minut
- Summarized history je cachována (tier 2)
- Full history v Cosmos DB pro analytics

### Token budget per user/month (worst case)

**Konzervativní odhad:**
- 5 konverzací/měsíc (ne 3)
- 25 zpráv/konverzace (ne 20)
- Nižší cache hit rate (70% ne 85%)

**Result:**
- MVP: $0.28/user/měsíc
- RAG: $0.34/user/měsíc
- Multi-agent: $0.31/user/měsíc

**Safety margin:** Budget 1.5× worst case = $0.50/user/měsíc

---

## Závěr a next steps

### Doporučený přístup pro launch do ledna 2026:

1. **Architektura**: RAG Intermediate
2. **Framework**: Azure AI Foundry (Prompt Flow + AI Search)
3. **Model**: Claude 3.5 Sonnet (primární) + Claude 3 Haiku (embeddings fallback)
4. **Náklady**:
   - Vývoj: **$30,000** (60 dní @ 2 developers)
   - Měsíční (320 users): **~$254**
   - Měsíční (1,000 users): **~$410**
5. **Timeline**: 12 týdnů (launch prosinec 2025, testing leden 2026)

### Milestones:

- **Září 2025**: Kick-off, setup infrastruktury
- **Říjen 2025**: Data preparation, RAG pipeline, 1. scenario
- **Listopad 2025**: Všechny 3 scénáře, frontend, integrace
- **Prosinec 2025**: Testing, fixes, soft launch (beta)
- **Leden 2026**: Public launch, monitoring, optimizace

### Kritické faktory úspěchu:

1. ✅ Kvalita dat z DB (aktuální informace o VŠ)
2. ✅ Spolupráce s kariérovými poradci na scénářích
3. ✅ Guardrails proti zneužití (cost control)
4. ✅ CZ/SK language quality testing
5. ✅ GA4 analytics setup pro monitoring konverzí

---

## Dodatečné poznámky

### Web search - finální doporučení:
**Neimplementovat v MVP/ROK 1**, protože:
- Priorita je data z DB (která budete mít)
- Web search zvyšuje latenci (2-3s → 5-7s)
- Zvyšuje náklady o ~20%
- Riziko hallucination z nekvalitních webů

**Implementovat v ROK 2** pokud:
- Zjistíte, že students často ptají na data, která nejsou v DB
- Budete mít budget na monitoring kvality web search results

### GDPR compliance:
- User data v Cosmos DB - retention policy 24 měsíců
- User může požádat o smazání dat
- Conversation logs anonymizované po 3 měsících
- Žádné PII v LLM prompts (jen user_id)

  Vylepšené MVP: Multi-Playbook Architecture

  Jak by to fungovalo:

  User query
    ↓
  ┌─────────────────────────┐
  │  Intent Classifier      │
  │  (Claude Haiku)         │
  │  "Který playbook?"      │
  └─────────────────────────┘
    ↓
  ┌─────────────────────────────────────┐
  │  Playbook Selection:                │
  │  • career_counseling_1              │
  │  • career_counseling_2              │
  │  • career_counseling_3              │
  │  • university_info_search           │
  │  • program_comparison               │
  │  • general_guidance                 │
  └─────────────────────────────────────┘
    ↓
  Selected Playbook (Claude Sonnet)
    + Cached system prompt
    + User context
    ↓
  Response

  Výhody tohoto přístupu:

  ✅ Menší tokeny per request
  - Nemusíte posílat všechny 3 scénáře najednou
  - Jen relevantní playbook (1.5K tokens místo 3K)

  ✅ Lepší cache efficiency
  - Každý playbook má samostatnou cache
  - Pokud student často používá "career_counseling_1", ten má 95%
   cache hit rate
  - Celková cache hit rate vyšší než u monolitu

  ✅ Modularita bez complexity
  - Přidání 4. scénáře = přidání nového playbooku
  - Nepotřebujete upravovat existující prompty
  - Jednodušší A/B testing jednotlivých scénářů

  ✅ Cost savings
  - Intent classifier = Claude Haiku (~50 tokens input, 10 tokens
   output) = $0.00006
  - Playbook execution = pouze relevantní prompt

  Srovnání tokenů:

  | Přístup               | Input tokens | Output tokens |
  Náklady/conv |
  |-----------------------|--------------|---------------|-------
  -------|
  | Původní MVP (monolit) | 8,000        | 3,000         | $0.06
         |
  | Multi-Playbook MVP    | 5,200        | 3,000         | $0.042
         |
  | Úspora                | -35%         | 0%            | -30%
         |

  Implementace:

  1. Intent Classification (první API call - Haiku)
  system_prompt = """
  Classify user intent into one of these playbooks:
  - career_counseling_1: Career exploration (interests-based)
  - career_counseling_2: Skills assessment
  - career_counseling_3: Values-based career guidance  
  - university_info: Specific university/program questions
  - comparison: Compare universities/programs
  - general: General guidance

  Return only the playbook name.
  """

  2. Playbook Execution (druhý API call - Sonnet)
  playbook_prompts = {
      "career_counseling_1": {
          "system": "...[scenario 1 instructions]...",
          "cache": True,
          "cache_ttl": "1h"
      },
      "university_info": {
          "system": "...[search instructions + DB schema]...",
          "cache": True,
          "cache_ttl": "5m"
      }
  }

  # Load selected playbook
  selected = playbook_prompts[intent]
  response = claude.complete(
      system=selected["system"],  # Cached
      messages=conversation_history + [user_message]
  )

  Upgrade path:

  Toto je ideální stepping stone:
  1. Měsíc 1-3: Multi-Playbook MVP (rychle implementovat)
  2. Měsíc 4-6: Monitor které playbooks se používají,
  optimalizovat
  3. Měsíc 7+: Pokud potřeba → upgrade na full multi-agent

  Revidované náklady MVP s multi-playbook:

  Náklady per konverzace:
  - Intent classification (Haiku): $0.00006
  - Playbook execution (Sonnet): $0.042
  - Total: ~$0.042 (vs původní $0.06)

  Měsíční @ 1,000 users:
  - Variabilní: 1,000 × 3 × $0.042 = $126/měsíc (vs $190)
  - Fixní: $52 (beze změny)
  - TOTAL: $178/měsíc (vs $242) = -26% úspora

  Doporučení:

  Aktualizované MVP = Multi-Playbook Architecture

  Je to:
  - ✅ Stále jednoduché na implementaci (+ 2 dny vývoje)
  - ✅ 30% levnější než monolit
  - ✅ Škálovatelnější (snadno přidat playbooks)
  - ✅ Lepší připravené na přechod do multi-agent

  Tento přístup kombinuje best of both worlds - jednoduchost MVP
  + modularitu multi-agent systému.

  Máte pravdu, že původní "všechno v jednom promptu" není
  optimální. Multi-playbook je lepší volba pro MVP! 