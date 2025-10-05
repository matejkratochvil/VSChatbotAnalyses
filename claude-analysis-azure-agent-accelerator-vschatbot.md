# Analýza Azure Accelerator projektů pro VSChatBot

**Datum:** 2025-10-04
**Analyzované projekty:**
1. Chat with Your Data Solution Accelerator
2. Azure Language OpenAI Conversational Agent Accelerator

**Požadavky:** AI_chat_specifikace.md - Chatbot "Chci na vejšku"

---

## Executive Summary

### Rychlé porovnání

| Aspekt | Chat with Your Data | Language Agent Accelerator | **Doporučení** |
|--------|---------------------|---------------------------|----------------|
| **Hlavní zaměření** | RAG přes dokumenty | Intent routing + RAG fallback | **Hybrid** |
| **Orchestrace** | 4 strategie (OpenAI, LangChain, SK, PromptFlow) | 6 routing strategií (CLU/CQA) | **Language Accelerator** |
| **Scénáře** | Dokumentové dotazy | Deterministické workflow + AI fallback | **Language Accelerator** |
| **Paměť konverzace** | PostgreSQL/CosmosDB | Chat History (basic) | **Chat with Your Data** |
| **Frontend** | React + TypeScript | React + TypeScript | **Oba podobné** |
| **Azure integrace** | Vysoká | Velmi vysoká | **Oba** |
| **Vhodnost pro MVP** | 7/10 | 9/10 | **Language Accelerator** |

### 🎯 Doporučení

**Použít HYBRID architekturu:**
- **Základ:** Azure Language OpenAI Conversational Agent Accelerator (CLU/CQA orchestrace)
- **Integrace:** Chat with Your Data komponenty pro RAG a memory management
- **Důvod:** Language Accelerator poskytuje **deterministické workflow** pro 3 kariérové scénáře, zatímco Chat with Your Data nabízí robustní **long-term memory** a **document ingestion**

---

## Projekt 1: Chat with Your Data Solution Accelerator

### Přehled

Komplexní RAG řešení pro konverzaci nad vlastními dokumenty s pokročilými funkcemi pro ingestion, chunking a retrieving.

### Architektura

```
┌─────────────────────────────────────────────────────────────┐
│                     Frontend (React)                         │
│                                                              │
│  - Document upload interface                                │
│  - Chat UI with history                                     │
│  - Admin panel for configuration                            │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│                  Backend Orchestrator                        │
│                                                              │
│  Strategie:                                                 │
│  1. OpenAI Functions - GPT function calling                 │
│  2. LangChain - Agent framework                             │
│  3. Semantic Kernel - MS orchestration                      │
│  4. Prompt Flow - Low-code ML pipeline                      │
└──────────────────┬──────────────────────────────────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
        ▼                     ▼
┌──────────────────┐  ┌──────────────────┐
│  Azure AI Search │  │   Azure OpenAI   │
│                  │  │                  │
│  - Vector index  │  │  - GPT-4         │
│  - Semantic rank │  │  - Embeddings    │
└──────────────────┘  └──────────────────┘
        │
        ▼
┌──────────────────────────────────────┐
│    Document Intelligence             │
│                                      │
│  - OCR                               │
│  - Layout analysis                   │
│  - Chunking strategies               │
└──────────────────────────────────────┘
        │
        ▼
┌──────────────────────────────────────┐
│  Database (PostgreSQL/CosmosDB)      │
│                                      │
│  - Chat history                      │
│  - User sessions                     │
│  - Document metadata                 │
└──────────────────────────────────────┘
```

### Klíčové funkce

#### 1. Document Ingestion
- **Podporované formáty:** PDF, DOCX, TXT, HTML, MD, XLSX, PPTX
- **Chunking strategie:**
  - Fixed-size overlap (default: 1024 tokens, overlap 128)
  - Layout-based (využívá Document Intelligence)
  - Semantic chunking
- **Push/Pull model:**
  - Push: Manuální upload přes admin UI
  - Pull: Integrated vectorization (automatické z Blob Storage)

**Kód reference:**
```python
# code/backend/batch/utilities/document_chunking/strategies.py
class ChunkingStrategy:
    LAYOUT = "layout"
    FIXED_SIZE_OVERLAP = "fixed_size_overlap"
    PAGE = "page"
```

#### 2. Orchestration Patterns

**4 implementované strategie:**

**A. OpenAI Functions** (`code/backend/batch/utilities/orchestrator/open_ai_functions.py`)
```python
def get_search_handler(config: ConfigHelper):
    return OpenAIFunctions(config)

# Výhody: Jednoduchý, rychlý
# Nevýhody: Méně kontroly
```

**B. LangChain** (`code/backend/batch/utilities/orchestrator/lang_chain_agent.py`)
```python
def create_agent(tools, llm):
    return create_react_agent(
        llm=llm,
        tools=tools,
        prompt=prompt
    )

# Výhody: Velká komunita, hodně šablon
# Nevýhody: Častá breaking changes
```

**C. Semantic Kernel** (`code/backend/batch/utilities/orchestrator/semantic_kernel.py`)
```python
# Microsoft framework s group chat orchestration
# Výhody: Azure integrace, plugins
# Nevýhody: Nový, menší komunita
```

**D. Prompt Flow** (`code/backend/batch/utilities/orchestrator/prompt_flow.py`)
```python
# Azure ML pipeline orchestration
# Výhody: Visual editor, experiment tracking
# Nevýhody: Vyžaduje Azure ML
```

#### 3. Long-term Memory (PostgreSQL)

**Database Schema:**
```sql
-- Conversation history
CREATE TABLE conversations (
    id UUID PRIMARY KEY,
    user_id VARCHAR(255),
    title TEXT,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

CREATE TABLE messages (
    id UUID PRIMARY KEY,
    conversation_id UUID REFERENCES conversations(id),
    role VARCHAR(50),
    content TEXT,
    created_at TIMESTAMP
);

-- User preferences
CREATE TABLE user_preferences (
    user_id VARCHAR(255) PRIMARY KEY,
    preferences JSONB
);
```

**Výhody PostgreSQL:**
- Relační data model
- JSONB support pro flexibilní metadata
- Full-text search indexing
- ACID transactions

#### 4. Admin Interface

Features:
- Document upload/management
- Prompt configuration
- System prompts editing
- Usage analytics dashboard
- User management

### Technology Stack

| Komponenta | Technologie | Verze |
|-----------|-------------|-------|
| Backend | Python | 3.11+ |
| Frontend | React + TypeScript | React 18 |
| Orchestration | OpenAI/LangChain/SK/PromptFlow | Latest |
| Database | PostgreSQL/CosmosDB | Flexible Server |
| Search | Azure AI Search | Standard S1 |
| Storage | Azure Blob Storage | Standard LRS |
| Hosting | Azure App Service | P1v3 |
| Functions | Azure Functions | Premium |

### Deployment

**Infra as Code:**
- **Bicep templates** v `infra/`
- **azd (Azure Developer CLI)** one-click deploy
- **Environment configs** pro dev/test/prod

**Příkaz:**
```bash
azd up
# Deployment time: ~15-20 minut
```

### Reusable Components pro VSChatBot

#### ✅ Přímo použitelné

1. **PostgreSQL memory schema** - pro dlouhodobou paměť uživatelů
   - Soubor: `infra/core/database/postgresql/`
   - Přínos: Konverzační historie, user preferences, quiz results

2. **Document chunking strategies** - pro zpracování univerzitních PDF
   - Soubor: `code/backend/batch/utilities/document_chunking/`
   - Přínos: Efektivní chunking prospektů, pravidel přijetí

3. **Azure AI Search integration** - pro RAG fallback
   - Soubor: `code/backend/batch/utilities/search/`
   - Přínos: Vyhledávání v univerzitní databázi

4. **React frontend base** - UI šablona
   - Soubor: `code/frontend/`
   - Přínos: Chat interface, file upload, history

#### ⚠️ Vyžaduje adaptaci

1. **Orchestration patterns** - moc obecné
   - Potřeba: Custom intent routing pro 3 scénáře

2. **Admin panel** - zaměřeno na dokumenty
   - Potřeba: Přizpůsobit pro správu scénářů, promptů

### Náklady (Azure)

**Pro 1,000 uživatelů/měsíc:**

| Služba | SKU | Měsíční náklad |
|--------|-----|----------------|
| Azure OpenAI (GPT-4) | Pay-as-you-go | $600-800 |
| Azure AI Search | Standard S1 | $250 |
| PostgreSQL | Flexible Server B2s | $50 |
| App Service | P1v3 | $80 |
| Storage | Standard LRS | $20 |
| **Celkem** | | **$1,000-1,200** |

---

## Projekt 2: Azure Language OpenAI Conversational Agent Accelerator

### Přehled

Pokročilý conversational agent s **deterministickým routingem** pomocí Azure AI Language (CLU/CQA) a **RAG fallbackem** pro dlouhý tail dotazů.

### Architektura

```
┌─────────────────────────────────────────────────────────────┐
│                  Frontend (React)                            │
└──────────────────┬──────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────────┐
│         UnifiedConversationOrchestrator                      │
│                                                              │
│  detect_language() → orchestrate() → route decision         │
└──────────┬──────────────────────────────────────────────────┘
           │
           ▼
    ┌──────────────────┐
    │  RouterType enum │
    │                  │
    │  1. TRIAGE_AGENT │──────────┐
    │  2. FUNCTION_CALLING        │
    │  3. CLU                     │
    │  4. CQA                     │
    │  5. ORCHESTRATION           │
    │  6. BYPASS                  │
    └──────────────────┘         │
                                 │
           ┌─────────────────────┴─────────────────┐
           │                                       │
           ▼                                       ▼
    ┌─────────────────┐                   ┌──────────────────┐
    │      CLU        │                   │       CQA        │
    │                 │                   │                  │
    │ Intent: 95%     │                   │ QA Pairs: 98%    │
    │ confidence      │                   │ confidence       │
    └────────┬────────┘                   └────────┬─────────┘
             │                                     │
             │ High confidence                     │
             ├─────────────┬───────────────────────┤
             │             │                       │
             ▼             ▼                       ▼
    ┌──────────────┐ ┌──────────────┐   ┌──────────────┐
    │ Custom Agent │ │ Custom Agent │   │  Direct      │
    │ OrderStatus  │ │ OrderRefund  │   │  Answer      │
    └──────────────┘ └──────────────┘   └──────────────┘
             │             │                       │
             └─────────────┴───────────────────────┘
                           │
                           │ Low confidence / No match
                           ▼
                  ┌──────────────────┐
                  │  Fallback: RAG   │
                  │  (Azure AI Search│
                  │   + OpenAI)      │
                  └──────────────────┘
```

### Klíčové funkce

#### 1. UnifiedConversationOrchestrator

**Centrální komponenta pro routing:**

```python
# src/backend/src/unified_conversation_orchestrator.py (92 řádků)

class UnifiedConversationOrchestrator:
    def __init__(
        self,
        router_type: RouterType,
        fallback_function: Callable[[str, str, str], dict]
    ):
        self.ta_client = TextAnalyticsClient(...)
        self.router = create_router(router_type)
        self.fallback_function = fallback_function

    def detect_language(self, text: str) -> str:
        """Detekce jazyka pomocí Azure AI Language"""
        result = self.ta_client.detect_language(documents=[text])
        return result[0].primary_language.iso6391_name

    def orchestrate(self, message: str, id: str = None) -> dict:
        """Orchestrace s routerem a fallbackem"""
        language = self.detect_language(text=message)
        routing_result = self.router(message, language, id)

        if routing_result is None or routing_result["error"] is not None:
            # Fallback to RAG
            fallback_result = self.fallback_function(message, language, id)
            return {
                "route": "fallback",
                "result": fallback_result
            }
        else:
            route = "clu" if routing_result["kind"] == "clu_result" else "cqa"
            return {
                "route": route,
                "result": routing_result
            }
```

**Výhody pro VSChatBot:**
- ✅ **Automatická detekce jazyka** - řeší CZ/SK požadavek
- ✅ **Fallback pattern** - RAG pro nepředvídatelné dotazy
- ✅ **Flexibilní routing** - 6 strategií

#### 2. Routing Strategies

**A. TRIAGE_AGENT (Doporučeno pro VSChatBot)**

Multi-agent orchestration s Semantic Kernel:

```python
# src/backend/src/semantic_kernel_orchestrator.py (387 řádků)

class SemanticKernelOrchestrator:
    async def initialize_agents(self) -> list:
        """Inicializace agentů z Azure AI Foundry"""

        # Translation agent pro CZ/SK/EN
        translation_agent = AzureAIAgent(
            client=self.client,
            definition=translation_agent_definition,
            description="A translation agent that translates to English"
        )

        # Triage agent používající CLU a CQA jako tools
        triage_agent = AzureAIAgent(
            client=self.client,
            definition=triage_agent_definition,
            description="A triage agent that routes inquiries"
        )

        # Custom business agents (pro VSChatBot: kariérové scénáře)
        order_status_agent = AzureAIAgent(
            client=self.client,
            definition=order_status_agent_definition,
            plugins=[OrderStatusPlugin()],
        )

        return [translation_agent, triage_agent, head_support_agent,
                order_status_agent, order_cancel_agent, order_refund_agent]

    async def create_agent_group_chat(self) -> None:
        """Group chat s custom selection a termination"""
        created_agents = await self.initialize_agents()

        self.orchestration = GroupChatOrchestration(
            members=created_agents,
            manager=CustomGroupChatManager(),  # Custom routing logic
        )
```

**Custom routing logic:**

```python
class CustomGroupChatManager(GroupChatManager):
    async def select_next_agent(self, chat_history, participant_descriptions):
        """Multi-agent orchestration routing"""
        last_message = chat_history[-1]

        # 1. User → TranslationAgent (CZ/SK → EN)
        if last_message.role == AuthorRole.USER:
            return route_user_message(participant_descriptions)

        # 2. TranslationAgent → TriageAgent (CLU/CQA detection)
        elif last_message.name == "TranslationAgent":
            return route_translation_message(last_message, participant_descriptions)

        # 3a. TriageAgent → CQA (vysoká konfidence)
        # 3b. TriageAgent → CLU intent → HeadSupportAgent
        elif last_message.name == "TriageAgent":
            parsed = json.loads(last_message.content)

            if parsed["type"] == "cqa_result":
                confidence = parsed["response"]["answers"][0]["confidenceScore"]
                if confidence >= cqa_confidence:
                    return StringResult(result="TranslationAgent")  # Přeložit odpověď
                else:
                    raise ValueError(f"Low CQA confidence: {confidence}")

            if parsed["type"] == "clu_result":
                intent = parsed["response"]["result"]["conversations"][0]["intents"][0]["name"]
                return StringResult(result="HeadSupportAgent")  # Route to custom agent

        # 4. HeadSupportAgent → Custom Agent (OrderStatus, OrderRefund, atd.)
        elif last_message.name == "HeadSupportAgent":
            parsed = json.loads(last_message.content)
            target_agent = parsed["target_agent"]
            return StringResult(result=target_agent)

        # 5. Custom Agent → TranslationAgent (EN → CZ/SK)
        elif last_message.name in ["OrderStatusAgent", "OrderRefundAgent", ...]:
            return StringResult(result="TranslationAgent")

    async def should_terminate(self, chat_history):
        """Ukončení po finální odpovědi TranslationAgent"""
        last_message = chat_history[-1]
        if last_message.name == "TranslationAgent" and len(chat_history) > 3:
            return BooleanResult(result=True)
        return BooleanResult(result=False)
```

**Jak to funguje pro VSChatBot:**

```
Uživatel (CZ): "Chci studovat informatiku, ale nevím kam."
    ↓
TranslationAgent: "I want to study informatics but don't know where."
    ↓
TriageAgent:
    - CLU call → Intent: "CareerCounseling" (confidence: 0.95)
    - Entities: { "field": "informatics" }
    ↓
HeadSupportAgent:
    - Routing logic → target_agent: "CareerCounselingInterestsAgent"
    ↓
CareerCounselingInterestsAgent:
    - Plugin function: run_career_scenario_interests(field="informatics")
    - Response (EN): "Based on informatics, I recommend these universities: ČVUT, MFF UK, FIT VUT..."
    ↓
TranslationAgent: "Na základě informatiky doporučuji tyto univerzity: ČVUT, MFF UK, FIT VUT..."
    ↓
Uživatel (CZ)
```

**B. CLU (Conversational Language Understanding)**

**Setup:**
```python
# infra/scripts/language/clu_setup.py
# Import intents z JSON

{
  "projectName": "contoso-outdoors-clu",
  "intents": [
    {
      "category": "OrderStatus",
      "examples": [
        "What is the status of my order?",
        "Where is my order?",
        "Track order 12345"
      ]
    },
    {
      "category": "RefundStatus",
      "examples": [
        "Where is my refund?",
        "Check refund for order 12"
      ]
    },
    {
      "category": "CancelOrder",
      "examples": [
        "Cancel order 100",
        "I want to cancel my order"
      ]
    }
  ],
  "entities": [
    {
      "category": "OrderId",
      "compositionSetting": "combine",
      "list": []
    }
  ]
}
```

**Pro VSChatBot - 3 kariérové scénáře:**
```json
{
  "intents": [
    {
      "category": "CareerCounselingInterests",
      "examples": [
        "Chci studovat informatiku",
        "Zajímá mě ekonomie",
        "Nevím co studovat, líbí se mi matematika"
      ]
    },
    {
      "category": "CareerCounselingSkills",
      "examples": [
        "Jsem dobrý v programování",
        "Umím dobře komunikovat",
        "Zajímá mě práce s lidmi"
      ]
    },
    {
      "category": "CareerCounselingValues",
      "examples": [
        "Chci pomáhat lidem",
        "Je pro mě důležitý work-life balance",
        "Zajímá mě kariéra s dopadem na společnost"
      ]
    },
    {
      "category": "UniversitySearch",
      "examples": [
        "Kdy jsou dny otevřených dveří na MFF UK?",
        "Jaké jsou podmínky přijetí na ČVUT?",
        "Které školy mají obor kybernetika?"
      ]
    },
    {
      "category": "ProgramComparison",
      "examples": [
        "Porovnej mi informatiku na ČVUT a VUT",
        "Která škola má lepší ekonomii?",
        "Rozdíl mezi FIT VUT a MFF UK"
      ]
    }
  ],
  "entities": [
    {
      "category": "University",
      "list": ["ČVUT", "MFF UK", "FIT VUT", "VŠE", ...]
    },
    {
      "category": "Field",
      "list": ["informatika", "ekonomie", "medicína", ...]
    }
  ]
}
```

**C. CQA (Custom Question Answering)**

Pre-defined QA pairs pro FAQ:

```python
# infra/scripts/language/cqa_setup.py

{
  "qnaList": [
    {
      "id": 1,
      "answer": "Our return policy allows returns within 30 days...",
      "source": "return_policy.md",
      "questions": [
        "What is your return policy?",
        "Can I return an item?",
        "How long do I have to return?"
      ]
    },
    {
      "id": 2,
      "answer": "Free shipping on orders over $50...",
      "source": "shipping.md",
      "questions": [
        "Do you offer free shipping?",
        "Shipping costs?",
        "How much is delivery?"
      ]
    }
  ]
}
```

**Pro VSChatBot:**
```json
{
  "qnaList": [
    {
      "id": 1,
      "answer": "Přijímací zkoušky probíhají v červnu. Konkrétní termíny se liší podle školy...",
      "questions": [
        "Kdy jsou přijímací zkoušky?",
        "Termíny přijímaček",
        "Kdy se konají přijímačky na vysoké školy?"
      ]
    },
    {
      "id": 2,
      "answer": "Maturita je základní podmínkou. Některé školy vyžadují také přijímací zkoušky...",
      "questions": [
        "Co potřebuji k přihlášce?",
        "Podmínky přijetí",
        "Jaké jsou požadavky pro VŠ?"
      ]
    }
  ]
}
```

#### 3. PII Detection & Redaction

```python
# src/backend/src/pii_redacter.py (164 řádků)

class PIIRedacter:
    def __init__(self):
        self.ta_client = TextAnalyticsClient(
            endpoint=os.environ.get("LANGUAGE_ENDPOINT"),
            credential=get_azure_credential()
        )

    def redact_pii(self, text: str, language: str = "en") -> dict:
        """Detekce a redakce PII před odesláním do LLM"""
        response = self.ta_client.recognize_pii_entities(
            documents=[text],
            language=language,
            categories_filter=[
                "Person", "Email", "PhoneNumber", "Address",
                "CreditCardNumber", "SSN"
            ]
        )

        redacted_text = response[0].redacted_text
        entities = [
            {
                "category": entity.category,
                "text": entity.text,
                "confidence": entity.confidence_score
            }
            for entity in response[0].entities
        ]

        return {
            "original": text,
            "redacted": redacted_text,
            "entities": entities
        }
```

**Výhoda pro VSChatBot:**
- Ochrana osobních údajů studentů (jméno, email, telefon)
- GDPR compliance
- Anonymizace před uložením do long-term memory

#### 4. Semantic Kernel Plugins

Custom business logic pro CLU intents:

```python
# src/backend/src/agents/order_status_plugin.py

from semantic_kernel import plugin

class OrderStatusPlugin:
    @plugin(name="check_order_status", description="Check order status")
    def check_order_status(self, order_id: str) -> str:
        # Database lookup
        order = db.query(f"SELECT * FROM orders WHERE id = {order_id}")

        if order.status == "shipped":
            return f"Order {order_id} has shipped. Tracking: {order.tracking_number}"
        elif order.status == "processing":
            return f"Order {order_id} is being processed. Expected ship date: {order.ship_date}"
        else:
            return f"Order {order_id} status: {order.status}"
```

**Pro VSChatBot:**
```python
# custom_plugins/career_counseling_plugin.py

class CareerCounselingInterestsPlugin:
    @plugin(
        name="run_career_scenario_interests",
        description="Run career counseling scenario based on student interests"
    )
    def run_career_scenario_interests(self, field: str, user_id: str) -> str:
        # 1. Načíst long-term memory studenta
        user_data = memory_service.get_user_data(user_id)

        # 2. Spustit scénář (z kariérových poradců)
        scenario_result = run_interests_scenario(
            field=field,
            previous_interests=user_data.get("interests", []),
            quiz_results=user_data.get("quiz_results", {})
        )

        # 3. Doporučit univerzity z DB
        universities = db_service.search_universities(
            field=field,
            criteria=scenario_result["criteria"]
        )

        # 4. Uložit do paměti
        memory_service.update_user_data(user_id, {
            "interests": scenario_result["interests"],
            "recommended_universities": universities
        })

        return f"Based on your interest in {field}, here are recommended universities: {universities}"
```

### Technology Stack

| Komponenta | Technologie | LOC | Reusable? |
|-----------|-------------|-----|-----------|
| Orchestrator | UnifiedConversationOrchestrator | 92 | ✅ 100% |
| SK Orchestrator | SemanticKernelOrchestrator | 387 | ✅ 80% |
| Routing | Function calling / CLU / CQA | 150 | ✅ 90% |
| PII Redaction | PIIRedacter | 164 | ✅ 100% |
| Custom Agents | Semantic Kernel Plugins | 100 | ⚠️ Custom |
| Frontend | React + TypeScript | ~2000 | ✅ 70% |
| **Total Backend** | | **~1300** | |

### Deployment

**Azure Services:**
- Azure AI Language (CLU + CQA + PII)
- Azure AI Agent Service (Semantic Kernel agents)
- Azure OpenAI (GPT-4o-mini, embeddings)
- Azure AI Search (RAG fallback)
- Azure Container Instances (hosting)

**Deployment:**
```bash
azd up
# Deployment time: 10-15 minut
# Post-deployment: 5 minut (CLU/CQA training)
```

### Reusable Components pro VSChatBot

#### ✅ Přímo použitelné (copy-paste)

1. **UnifiedConversationOrchestrator** (92 LOC)
   - Soubor: `src/backend/src/unified_conversation_orchestrator.py`
   - Přínos: **Kompletní routing logika** pro 3 scénáře
   - Změny: Žádné, jen config

2. **SemanticKernelOrchestrator** (387 LOC)
   - Soubor: `src/backend/src/semantic_kernel_orchestrator.py`
   - Přínos: **Multi-agent orchestration** s translation
   - Změny: Přejmenovat agenty, upravit routing

3. **PIIRedacter** (164 LOC)
   - Soubor: `src/backend/src/pii_redacter.py`
   - Přínos: **GDPR compliance** automaticky
   - Změny: Žádné

4. **CLU/CQA setup scripts**
   - Soubor: `infra/scripts/language/`
   - Přínos: **Automatické vytvoření** CLU/CQA projektů
   - Změny: Upravit JSON data (intents, QA pairs)

#### ⚠️ Vyžaduje adaptaci

1. **Custom Plugins** - vytvořit nové pro kariérové scénáře
   - Inspirace: `src/backend/src/agents/`
   - Nové: `CareerCounselingInterestsPlugin`, `CareerCounselingSkillsPlugin`, `CareerCounselingValuesPlugin`

2. **Frontend** - přizpůsobit UI
   - Inspirace: `src/frontend/`
   - Změny: Branding, layout

### Náklady (Azure)

**Pro 1,000 uživatelů/měsíc:**

| Služba | SKU | Měsíční náklad |
|--------|-----|----------------|
| Azure AI Language (CLU+CQA+PII) | S tier | $300 |
| Azure OpenAI (GPT-4o-mini) | Pay-as-you-go | $400-500 |
| Azure AI Agent Service | Pay-as-you-go | $200 |
| Azure AI Search (fallback) | Basic | $75 |
| Azure Container Instances | 1 vCPU, 1 GB | $30 |
| Storage | Standard LRS | $10 |
| **Celkem** | | **$1,015-1,115** |

**Poznámka:** Podobné náklady jako Chat with Your Data, ale **vyšší kvalita** díky CLU/CQA.

---

## Feature Mapping k požadavkům (AI_chat_specifikace.md)

### Požadavek 1: Dlouhodobá paměť

| Aspekt | Chat with Your Data | Language Accelerator | **Doporučení** |
|--------|---------------------|---------------------|----------------|
| **Database** | PostgreSQL (relační) | Basic (CosmosDB/memory) | **Chat with Data** |
| **Schema** | ✅ Conversations, Messages, Preferences | ⚠️ Simple key-value | **Použít PostgreSQL** |
| **Integrace s kvízy** | ✅ JSONB preferences | ⚠️ Vyžaduje custom | **PostgreSQL JSONB** |
| **Paměť napříč scénáři** | ✅ Přes user_id | ⚠️ Session-based | **PostgreSQL** |

**Implementace:**
```python
# Převzít z Chat with Your Data
# + Integrovat s Language Accelerator orchestratorem

class MemoryService:
    def __init__(self, db_connection_string):
        self.conn = psycopg2.connect(db_connection_string)

    def get_user_data(self, user_id: str) -> dict:
        """Načíst veškerá data uživatele"""
        query = """
            SELECT
                u.preferences,
                array_agg(c.id) as conversation_ids,
                array_agg(q.results) as quiz_results
            FROM users u
            LEFT JOIN conversations c ON c.user_id = u.user_id
            LEFT JOIN quiz_results q ON q.user_id = u.user_id
            WHERE u.user_id = %s
            GROUP BY u.user_id
        """
        return cursor.fetchone()

    def update_user_interests(self, user_id: str, interests: list):
        """Uložit zjištěné zájmy z kariérového scénáře"""
        query = """
            UPDATE users
            SET preferences = jsonb_set(
                preferences,
                '{interests}',
                %s::jsonb
            )
            WHERE user_id = %s
        """
        cursor.execute(query, (json.dumps(interests), user_id))
```

**Verdict:** ✅ **Použít PostgreSQL schema z Chat with Your Data**

---

### Požadavek 2: Dynamické zpracování scénářů

| Aspekt | Chat with Your Data | Language Accelerator | **Doporučení** |
|--------|---------------------|---------------------|----------------|
| **Scénáře** | ❌ Není zaměřeno | ✅ CLU intents | **Language Accelerator** |
| **Routing** | ⚠️ Obecný (RAG) | ✅ Deterministický (CLU) | **CLU routing** |
| **Rozšiřitelnost** | ❌ Vyžaduje kód | ✅ JSON konfigurace | **CLU JSON** |
| **Multi-step workflow** | ❌ Není | ✅ Semantic Kernel orchestration | **SK group chat** |

**Jak to bude fungovat:**

```
3 scénáře = 3 CLU intents:
1. CareerCounselingInterests
2. CareerCounselingSkills
3. CareerCounselingValues

Každý intent → Semantic Kernel Agent → Plugin → Multi-step conversation
```

**Příklad flow pro CareerCounselingInterests:**

```python
# infra/data/clu_import.json
{
  "intents": [
    {
      "category": "CareerCounselingInterests",
      "examples": [
        "Chci studovat informatiku",
        "Zajímá mě ekonomie",
        "Nevím co studovat, líbí se mi matematika",
        "Baví mě programování",
        ...  # 50+ příkladů
      ]
    }
  ],
  "entities": [
    {
      "category": "Field",
      "list": ["informatika", "ekonomie", "medicína", "právo", ...]
    }
  ]
}
```

```python
# custom_plugins/career_interests_plugin.py

class CareerCounselingInterestsPlugin:
    @plugin(name="run_interests_scenario")
    def run_interests_scenario(self, field: str, user_id: str) -> dict:
        """
        Scénář od kariérových poradců:
        1. Zjistit konkrétní zájmy studenta
        2. Probrat možnosti v daném oboru
        3. Zúžit výběr univerzit
        """

        # State machine pro multi-step konverzaci
        state = self.get_scenario_state(user_id)

        if state["step"] == 1:
            # Krok 1: Upřesnit zájmy
            response = f"Zajímá tě {field}. Jaké konkrétní oblasti v tomto oboru tě baví?"
            self.update_state(user_id, step=2)

        elif state["step"] == 2:
            # Krok 2: Diskuze o možnostech
            specific_interests = state["specific_interests"]
            response = f"Skvělé! V oblasti {specific_interests} můžeš studovat tyto programy..."
            self.update_state(user_id, step=3)

        elif state["step"] == 3:
            # Krok 3: Doporučení univerzit
            universities = db_service.search_universities(
                field=field,
                specialization=state["specific_interests"]
            )
            response = f"Doporučuji tyto univerzity: {universities}"
            self.update_state(user_id, step="completed")

        return {
            "response": response,
            "state": state,
            "completed": state["step"] == "completed"
        }
```

**Verdict:** ✅ **Použít CLU + Semantic Kernel Plugins z Language Accelerator**

---

### Požadavek 3: Rychlost a kvalita

| Aspekt | Chat with Your Data | Language Accelerator | **Doporučení** |
|--------|---------------------|---------------------|----------------|
| **Response time** | 2-4s (RAG overhead) | 1-2s (CLU) + 3-5s (fallback) | **Hybrid** |
| **Kvalita odpovědí** | ⚠️ RAG hallucinations | ✅ CLU (95% accuracy) + CQA (98%) | **Language Accelerator** |
| **Caching** | ⚠️ Manual | ⚠️ Manual | **Implementovat** |

**Optimalizace rychlosti:**

```python
# Použít Claude Haiku pro CLU routing (levné, rychlé)
# Použít GPT-4o-mini pro scénáře (kvalita, rychlost)
# Použít prompt caching (viz composite_architecture_blueprint.md)

class OptimizedOrchestrator(UnifiedConversationOrchestrator):
    def __init__(self, ...):
        # Fast routing model
        self.router_model = "claude-haiku-4-20250514"

        # Quality conversation model
        self.conversation_model = "gpt-4o-mini"

        # Aggressive caching
        self.cache_config = {
            "system_prompt": {"ttl": 3600},  # 1h
            "playbook_prompts": {"ttl": 3600},  # 1h
            "user_memory": {"ttl": 300}  # 5min
        }
```

**Verdict:** ✅ **Hybrid - CLU pro rychlost, GPT-4o-mini pro kvalitu**

---

### Požadavek 4: Web Search

| Aspekt | Chat with Your Data | Language Accelerator | **Doporučení** |
|--------|---------------------|---------------------|----------------|
| **Implementace** | ❌ Není | ❌ Není | **Custom MCP server** |
| **Prioritizace zdrojů** | ❌ N/A | ❌ N/A | **Custom logic** |

**Implementace (volitelné):**

Použít pattern z `composite_architecture_blueprint.md`:

```python
# mcp_servers/web_search_server.py

from mcp.server import Server

app = Server("web-search")

ALLOWED_DOMAINS = [
    "vysokeskoly.cz",
    "kampomaturite.cz",
    "scio.cz",
    "regvssp.msmt.cz",
    # + official university websites
]

@app.call_tool()
async def web_search(query: str, domain_filter: str = None):
    """
    Web search s prioritou oficiálních zdrojů
    """
    # 1. Search na allowed domains
    results = await brave_search(
        query=query,
        domains=ALLOWED_DOMAINS if not domain_filter else [domain_filter]
    )

    # 2. Fallback na general search (pokud nejsou výsledky)
    if not results:
        results = await brave_search(query=query)
        results = [r for r in results if is_trustworthy_source(r.domain)]

    return results
```

**Verdict:** ⚠️ **Zvážit nákladnost - možná není potřeba pro MVP**

---

### Požadavek 5: Podpora jazyků (CZ/SK)

| Aspekt | Chat with Your Data | Language Accelerator | **Doporučení** |
|--------|---------------------|---------------------|----------------|
| **Detekce jazyka** | ❌ Není | ✅ `detect_language()` | **Language Accelerator** |
| **Translation** | ❌ Není | ✅ TranslationAgent | **SK TranslationAgent** |
| **Úvodní zpráva** | ❌ Statická | ⚠️ Vyžaduje custom | **Custom logic** |

**Řešení úvodní zprávy:**

```python
# Server-side detection a response

WELCOME_MESSAGES = {
    "cs": "Ahoj! Jsem chatbot 'Chci na vejšku'. Pomohu ti vybrat vysokou školu...",
    "sk": "Ahoj! Som chatbot 'Chci na vejšku'. Pomôžem ti vybrať vysokú školu..."
}

@app.route("/api/chat/welcome")
async def get_welcome_message(request):
    # Detekce z browser locale nebo IP
    language = request.headers.get("Accept-Language", "cs")[:2]

    if language not in ["cs", "sk"]:
        language = "cs"  # Default

    return {
        "message": WELCOME_MESSAGES[language],
        "language": language
    }
```

**Verdict:** ✅ **Použít Language Accelerator + custom welcome logic**

---

### Požadavek 6: Azure infrastruktura

| Aspekt | Chat with Your Data | Language Accelerator | **Doporučení** |
|--------|---------------------|---------------------|----------------|
| **Deployment** | ✅ Azure App Service | ✅ Container Instances | **App Service** |
| **IaC** | ✅ Bicep | ✅ Bicep | **Kombinovat** |
| **Monitoring** | ✅ App Insights | ✅ App Insights | **Oba** |

**Verdict:** ✅ **Azure native - oba projekty jsou kompatibilní**

---

### Požadavek 7: Zpětná vazba

| Aspekt | Chat with Your Data | Language Accelerator | **Doporučení** |
|--------|---------------------|---------------------|----------------|
| **Feedback UI** | ✅ Thumbs up/down | ⚠️ Basic | **Chat with Data UI** |
| **Storage** | ✅ PostgreSQL | ❌ Není | **PostgreSQL** |
| **Analytics** | ⚠️ Basic | ❌ Není | **Custom GA4** |

**Schema:**
```sql
CREATE TABLE feedback (
    id UUID PRIMARY KEY,
    conversation_id UUID REFERENCES conversations(id),
    message_id UUID,
    rating INT CHECK (rating IN (1, -1)),  -- thumbs up/down
    comment TEXT,
    created_at TIMESTAMP
);
```

**Verdict:** ✅ **Použít feedback UI z Chat with Your Data + PostgreSQL**

---

### Požadavek 8: Prioritizace zdrojů dat

| Priorita | Zdroj | Jak implementovat |
|----------|-------|-------------------|
| 1 | **Naše DB** | Azure SQL Database s university data |
| 2 | **Knowledge base** | CQA (FAQ) + CLU (intents) |
| 3 | **Web search** | MCP web search server (volitelné) |

**Routing logika:**

```python
async def orchestrate_with_priority(message: str, user_id: str):
    # 1. Zkusit CLU intent detection
    clu_result = await clu_client.analyze(message)

    if clu_result.confidence > 0.8:
        if clu_result.intent == "UniversitySearch":
            # Priorita 1: DB query
            return await db_search(clu_result.entities)
        elif clu_result.intent in CAREER_SCENARIOS:
            # Priorita 2: CLU scenario
            return await run_scenario(clu_result.intent, user_id)

    # 2. Zkusit CQA (FAQ)
    cqa_result = await cqa_client.get_answer(message)

    if cqa_result.confidence > 0.7:
        # Priorita 2: CQA answer
        return cqa_result.answer

    # 3. Fallback: RAG přes DB + optional web search
    rag_result = await rag_search(message, sources=["database", "web"])
    return rag_result
```

**Verdict:** ✅ **Custom priority logic kombinující oba projekty**

---

### Požadavek 9: LLM a token optimalizace

| Aspekt | Chat with Your Data | Language Accelerator | **Doporučení** |
|--------|---------------------|---------------------|----------------|
| **Prompt caching** | ❌ Manual | ❌ Manual | **Implementovat** |
| **Model selection** | ⚠️ GPT-4 (drahé) | ✅ GPT-4o-mini | **GPT-4o-mini** |
| **Token tracking** | ⚠️ Basic | ⚠️ Basic | **Custom analytics** |

**Optimalizace (viz composite_architecture_blueprint.md):**

```python
# 3-level caching strategy

system = [
    {
        "type": "text",
        "text": BASE_SYSTEM_PROMPT,  # 2000 tokens
        "cache_control": {"type": "ephemeral"}  # Cache 1h
    },
    {
        "type": "text",
        "text": CAREER_SCENARIO_PROMPTS[scenario],  # 3000 tokens
        "cache_control": {"type": "ephemeral"}  # Cache 1h
    }
]

messages = [
    {
        "role": "user",
        "content": [
            {
                "type": "text",
                "text": f"User memory: {user_memory_summary}",  # 1000 tokens
                "cache_control": {"type": "ephemeral"}  # Cache 5min
            }
        ]
    },
    *conversation_history[-10:],  # 5000 tokens - no cache
    {"role": "user", "content": current_message}  # 100 tokens - no cache
]

# Total: 11,100 tokens
# Cached: 6,000 tokens (54%)
# Cost reduction: 90% na cached tokens
```

**Token estimates:**

| Konverzace | Input tokens | Output tokens | Cached | Náklad |
|-----------|--------------|---------------|--------|--------|
| První zpráva | 11,100 | 500 | 0% | $0.040 |
| Druhá zpráva | 11,200 | 500 | 54% | $0.008 |
| Třetí zpráva | 11,300 | 500 | 54% | $0.008 |
| **10-15 min konverzace (10 zpráv)** | | | | **$0.10** |

**1,000 uživatelů/měsíc:**
- Průměr: 3 konverzace/uživatel/měsíc
- Celkem: 3,000 konverzací
- **Měsíční náklad: $300**

**Verdict:** ✅ **Implementovat 3-level prompt caching + GPT-4o-mini**

---

### Požadavek 10: Analytika (GA4)

| Aspekt | Chat with Your Data | Language Accelerator | **Doporučení** |
|--------|---------------------|---------------------|----------------|
| **Tracking** | ⚠️ Basic App Insights | ⚠️ Basic | **Custom GA4** |
| **Metrics** | ❌ Není | ❌ Není | **Custom events** |

**GA4 Events:**

```typescript
// Frontend tracking

// Event 1: Chat started
gtag('event', 'chat_started', {
  user_id: userId,
  language: 'cs',
  timestamp: new Date().toISOString()
});

// Event 2: Message sent
gtag('event', 'message_sent', {
  user_id: userId,
  message_type: 'user',
  route: 'clu',  // nebo 'cqa', 'fallback'
  intent: 'CareerCounselingInterests'
});

// Event 3: Scenario completed
gtag('event', 'scenario_completed', {
  user_id: userId,
  scenario: 'CareerCounselingInterests',
  duration_seconds: 420,
  messages_count: 8
});

// Event 4: Chat abandoned
gtag('event', 'chat_abandoned', {
  user_id: userId,
  last_message_type: 'bot',
  duration_seconds: 120,
  messages_count: 3
});

// Event 5: Feedback submitted
gtag('event', 'feedback_submitted', {
  user_id: userId,
  conversation_id: conversationId,
  rating: 1,  // thumbs up
  comment: 'Velmi užitečné!'
});
```

**Tracking metrics:**

1. **Míra konverze:**
   - Started conversations / Total visitors
   - Completed scenarios / Started conversations

2. **Úspěšnost odpovědí:**
   - Positive feedback / Total feedback
   - Completed scenarios / Total scenarios

3. **Engagement:**
   - Average messages per conversation
   - Average conversation duration
   - Returning users rate

**Verdict:** ✅ **Custom GA4 implementation**

---

## Architektura Comparison

### Side-by-side Porovnání

| Dimenze | Chat with Your Data | Language Accelerator | **Hybrid (Doporučeno)** |
|---------|---------------------|---------------------|------------------------|
| **Architekturní pattern** | RAG-first | Intent routing → RAG fallback | ✅ Intent routing + RAG |
| **Orchestrace** | 4 strategie | 6 routing strategií | ✅ TRIAGE_AGENT + SK |
| **Deterministické workflow** | ❌ Není | ✅ CLU intents | ✅ CLU + custom plugins |
| **Long-term memory** | ✅ PostgreSQL (advanced) | ⚠️ Basic | ✅ PostgreSQL |
| **Multi-step scénáře** | ❌ Není | ✅ SK group chat | ✅ SK group chat |
| **Jazyková podpora** | ❌ Statická | ✅ TranslationAgent | ✅ TranslationAgent |
| **PII protection** | ❌ Není | ✅ PIIRedacter | ✅ PIIRedacter |
| **Document ingestion** | ✅ Advanced | ❌ Není | ⚠️ Možná nepotřeba |
| **Admin panel** | ✅ Full-featured | ❌ Není | ✅ Přizpůsobit |
| **Feedback system** | ✅ Built-in | ❌ Není | ✅ Built-in |
| **Code reusability** | 60% | 80% | ✅ 70% combined |
| **Development time** | 12-16 týdnů | 8-10 týdnů | ✅ 10-12 týdnů |
| **Azure services** | 8 služeb | 6 služeb | 9 služeb |
| **Měsíční náklad (1K users)** | $1,000-1,200 | $1,015-1,115 | $1,100-1,300 |

### Strengths/Weaknesses

#### Chat with Your Data

**✅ Strengths:**
1. **Production-ready RAG** - robustní document ingestion a retrieval
2. **Long-term memory** - PostgreSQL schema pro user data
3. **Admin panel** - UI pro správu dokumentů a konfiguraci
4. **Multiple orchestration** - 4 strategie (flexibility)
5. **Feedback system** - Built-in thumbs up/down

**❌ Weaknesses:**
1. **Není pro scénáře** - zaměřeno na RAG, ne na workflow
2. **Statická orchestrace** - všechno jde přes RAG
3. **Žádná jazyková podpora** - translation není
4. **Žádná PII ochrana** - GDPR compliance chybí

**Vhodnost pro VSChatBot:** 6/10 - Skvělé komponenty, ale chybí scénáře

---

#### Language Accelerator

**✅ Strengths:**
1. **Deterministické workflow** - CLU intents s 95% přesností
2. **Multi-agent orchestration** - Semantic Kernel group chat
3. **Jazyková podpora** - TranslationAgent pro CZ/SK/EN
4. **PII protection** - PIIRedacter pro GDPR
5. **Flexibilní routing** - 6 strategií
6. **High code reusability** - 80% lze použít

**❌ Weaknesses:**
1. **Basic memory** - není long-term memory
2. **Žádný admin panel** - správa přes JSON files
3. **Žádný feedback system** - chybí
4. **RAG je fallback** - méně robustní než Chat with Data

**Vhodnost pro VSChatBot:** 8/10 - Ideální pro scénáře, chybí memory

---

### Code Reusability Assessment

#### Z Chat with Your Data (60% reusable)

| Komponenta | LOC | Reusable? | Effort |
|-----------|-----|-----------|--------|
| PostgreSQL schema | - | ✅ 100% | Copy infra/ |
| Document chunking | 500 | ⚠️ 50% | Možná nepotřeba |
| Azure AI Search integration | 300 | ✅ 80% | Minor changes |
| Admin panel | 2000 | ✅ 70% | Rebrand, adapt |
| Feedback UI | 200 | ✅ 100% | Copy component |
| Orchestrator base | 400 | ❌ 0% | Nahradit Language Acc |

**Total reusable:** ~2,000 LOC z ~3,500 LOC

---

#### Z Language Accelerator (80% reusable)

| Komponenta | LOC | Reusable? | Effort |
|-----------|-----|-----------|--------|
| UnifiedConversationOrchestrator | 92 | ✅ 100% | Copy-paste |
| SemanticKernelOrchestrator | 387 | ✅ 90% | Rename agents |
| PIIRedacter | 164 | ✅ 100% | Copy-paste |
| CLU/CQA setup scripts | 200 | ✅ 100% | Update JSON data |
| Custom plugins (example) | 100 | ⚠️ 0% | Write new |
| Frontend | 2000 | ✅ 70% | Rebrand |

**Total reusable:** ~2,500 LOC z ~3,000 LOC

---

#### Hybrid Architecture (70% combined reusability)

**Total LOC estimate:** ~8,000 LOC
- **Reusable:** ~4,500 LOC (56%)
- **Custom:** ~3,500 LOC (44%)

**Breakdown:**
1. **From Language Accelerator (base):** 2,500 LOC
2. **From Chat with Your Data (components):** 2,000 LOC
3. **Custom development:**
   - Career counseling plugins: 500 LOC
   - Database service integration: 400 LOC
   - University search logic: 300 LOC
   - UI customization: 800 LOC
   - Analytics integration: 200 LOC
   - Testing: 1,300 LOC

---

## Recommended Approach: Hybrid Architecture

### Architektura Overview

```
┌───────────────────────────────────────────────────────────────────┐
│                     Frontend (React + TypeScript)                 │
│                                                                   │
│  Components:                                                      │
│  - Chat UI (from Language Accelerator)                           │
│  - Admin Panel (from Chat with Your Data)                        │
│  - Feedback Widget (from Chat with Your Data)                    │
│  - GA4 Tracking (custom)                                         │
└─────────────────────────────┬─────────────────────────────────────┘
                              │
                              ▼
┌───────────────────────────────────────────────────────────────────┐
│              UnifiedConversationOrchestrator                      │
│              (from Language Accelerator)                          │
│                                                                   │
│  1. detect_language(text) → "cs" / "sk"                          │
│  2. orchestrate(message, user_id)                                │
│     ├─ PIIRedacter.redact_pii(message) → redacted_message       │
│     ├─ router_type = TRIAGE_AGENT                               │
│     └─ routing_result = router(redacted_message, language, id)  │
└─────────────────────────────┬─────────────────────────────────────┘
                              │
        ┌─────────────────────┴─────────────────────┐
        │                                           │
        ▼                                           ▼
┌──────────────────────┐                  ┌──────────────────────┐
│  Semantic Kernel     │                  │   Fallback: RAG      │
│  Group Chat          │                  │                      │
│  Orchestration       │                  │  (from Chat with     │
│                      │                  │   Your Data)         │
│  Agents:             │                  │                      │
│  1. TranslationAgent │                  │  Azure AI Search     │
│  2. TriageAgent      │                  │  + GPT-4o-mini       │
│  3. HeadSupportAgent │                  │  + PostgreSQL docs   │
│  4. Career Interests │                  └──────────────────────┘
│  5. Career Skills    │
│  6. Career Values    │
│  7. UniversitySearch │
│  8. ProgramCompare   │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────────────────────────────────────────────────┐
│                    Azure AI Language                             │
│                                                                  │
│  CLU Project:                                                   │
│  - CareerCounselingInterests (intent)                           │
│  - CareerCounselingSkills (intent)                              │
│  - CareerCounselingValues (intent)                              │
│  - UniversitySearch (intent)                                    │
│  - ProgramComparison (intent)                                   │
│                                                                  │
│  CQA Project:                                                   │
│  - FAQ o přijímacích zkouškách                                  │
│  - FAQ o podmínkách přijetí                                     │
│  - FAQ o termínech                                              │
│  - FAQ o financování studia                                     │
└──────────────────────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────────────────────┐
│               Custom Semantic Kernel Plugins                     │
│                                                                  │
│  1. CareerCounselingInterestsPlugin                             │
│     - run_interests_scenario(field, user_id)                    │
│     - Uses: PostgreSQL memory, University DB                    │
│                                                                  │
│  2. CareerCounselingSkillsPlugin                                │
│     - run_skills_scenario(skills, user_id)                      │
│                                                                  │
│  3. CareerCounselingValuesPlugin                                │
│     - run_values_scenario(values, user_id)                      │
│                                                                  │
│  4. UniversitySearchPlugin                                      │
│     - search_universities(criteria)                             │
│     - get_admission_info(university, program)                   │
│                                                                  │
│  5. ProgramComparisonPlugin                                     │
│     - compare_programs(universities, program)                   │
└──────────────────────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────────────────────┐
│                      Data Layer                                  │
│                                                                  │
│  PostgreSQL (from Chat with Your Data):                         │
│  ├─ conversations (history)                                     │
│  ├─ messages                                                    │
│  ├─ users                                                       │
│  │  └─ preferences (JSONB)                                      │
│  │     ├─ interests: []                                         │
│  │     ├─ skills: []                                            │
│  │     ├─ values: []                                            │
│  │     └─ quiz_results: {}                                      │
│  ├─ feedback                                                    │
│  └─ universities (custom table)                                 │
│     ├─ id, name, location                                       │
│     ├─ programs (JSONB array)                                   │
│     └─ admission_info (JSONB)                                   │
│                                                                  │
│  Azure AI Search (from Chat with Your Data):                    │
│  └─ university_documents index                                  │
│     └─ PDFs, web pages, manuals                                 │
└──────────────────────────────────────────────────────────────────┘
```

### Komponenty k převzetí

#### 1. Z Language Accelerator (základ)

**Core orchestration:**
```bash
src/backend/src/
├── unified_conversation_orchestrator.py  # ✅ Copy 100%
├── semantic_kernel_orchestrator.py       # ✅ Copy 90%, rename agents
├── pii_redacter.py                       # ✅ Copy 100%
├── router/
│   ├── router_type.py                    # ✅ Copy 100%
│   ├── router_utils.py                   # ✅ Copy 100%
│   └── ...                               # ✅ Copy all routers
└── utils.py                              # ✅ Copy 100%
```

**Infrastructure:**
```bash
infra/
├── scripts/language/
│   ├── clu_setup.py                      # ✅ Copy, update JSON
│   ├── cqa_setup.py                      # ✅ Copy, update JSON
│   └── utils.py                          # ✅ Copy
├── data/
│   ├── clu_import.json                   # ⚠️ Write new (5 intents)
│   └── cqa_import.json                   # ⚠️ Write new (FAQ)
└── openapi_specs/                        # ✅ Copy all
```

**Frontend:**
```bash
src/frontend/
├── src/
│   ├── components/
│   │   └── ChatWindow.tsx                # ✅ Copy 80%, rebrand
│   └── App.tsx                           # ✅ Copy 70%
└── package.json                          # ✅ Copy, update
```

**Total from Language Accelerator:** ~2,500 LOC

---

#### 2. Z Chat with Your Data (komponenty)

**Database:**
```bash
infra/core/database/postgresql/
├── flexibleserver.bicep                  # ✅ Copy
└── scripts/
    └── init_schema.sql                   # ✅ Copy, extend with universities table
```

**SQL Schema extensions:**
```sql
-- Add to existing schema

CREATE TABLE universities (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    location VARCHAR(255),
    website VARCHAR(500),
    programs JSONB,  -- Array of programs
    admission_info JSONB,  -- Podmínky přijetí
    contact JSONB,  -- Kontaktní info
    open_days JSONB,  -- Dny otevřených dveří
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_universities_programs ON universities USING GIN (programs);
CREATE INDEX idx_universities_name ON universities(name);

-- Extend user preferences
ALTER TABLE users
ADD COLUMN IF NOT EXISTS preferences JSONB DEFAULT '{
  "interests": [],
  "skills": [],
  "values": [],
  "quiz_results": {},
  "recommended_universities": []
}'::jsonb;
```

**Admin Panel:**
```bash
code/frontend/
├── src/
│   ├── pages/
│   │   └── Admin.tsx                     # ✅ Copy 60%, adapt for scenarios
│   └── components/
│       ├── DocumentUpload.tsx            # ⚠️ Optional (university PDFs)
│       └── ConfigEditor.tsx              # ✅ Copy for prompt management
```

**Feedback:**
```bash
code/backend/
└── feedback/
    ├── feedback_handler.py               # ✅ Copy
    └── schema.sql                        # ✅ Copy (feedback table)
```

**Total from Chat with Your Data:** ~2,000 LOC

---

#### 3. Custom Development (nové)

**Career Counseling Plugins:**

```python
# custom_plugins/career_interests_plugin.py (200 LOC)

from semantic_kernel import plugin
from dataclasses import dataclass

@dataclass
class ScenarioState:
    step: int
    field: str
    specific_interests: list
    recommended_universities: list
    completed: bool

class CareerCounselingInterestsPlugin:
    def __init__(self, db_service, memory_service):
        self.db = db_service
        self.memory = memory_service

    @plugin(
        name="run_interests_scenario",
        description="Run career counseling scenario based on student interests"
    )
    async def run_interests_scenario(
        self,
        field: str,
        user_id: str,
        user_input: str = None
    ) -> dict:
        """
        Multi-step scenario flow:
        1. Identify broad field
        2. Drill down to specific interests
        3. Recommend universities
        """

        # Load scenario state from memory
        state = self.memory.get_scenario_state(user_id, "interests")

        if state is None:
            # Step 1: Initialize
            state = ScenarioState(
                step=1,
                field=field,
                specific_interests=[],
                recommended_universities=[],
                completed=False
            )

        # State machine
        if state.step == 1:
            response = await self._step1_identify_field(field, user_id)
            state.step = 2

        elif state.step == 2:
            response = await self._step2_drill_down(user_input, state, user_id)
            state.step = 3

        elif state.step == 3:
            response = await self._step3_recommend(state, user_id)
            state.completed = True

        # Save state
        self.memory.save_scenario_state(user_id, "interests", state)

        return {
            "response": response,
            "state": state.__dict__,
            "need_more_info": not state.completed
        }

    async def _step1_identify_field(self, field: str, user_id: str):
        # Load user's quiz results if available
        quiz_results = self.memory.get_user_data(user_id).get("quiz_results", {})

        prompt = f"""
        Student expressed interest in: {field}

        Quiz results: {quiz_results}

        Ask 2-3 follow-up questions to identify specific areas within {field}
        that the student is interested in.

        Response format (Czech):
        """

        return await generate_response(prompt)

    async def _step2_drill_down(self, user_input: str, state: ScenarioState, user_id: str):
        # Analyze user input to extract specific interests
        interests = await extract_interests(user_input, state.field)
        state.specific_interests = interests

        prompt = f"""
        Student is interested in: {state.field}
        Specific interests: {interests}

        Explain what programs/specializations are available in this area.
        Mention 3-5 top universities in Czech Republic.

        Response format (Czech):
        """

        return await generate_response(prompt)

    async def _step3_recommend(self, state: ScenarioState, user_id: str):
        # Query database for matching universities
        universities = await self.db.search_universities(
            field=state.field,
            specializations=state.specific_interests
        )

        state.recommended_universities = [u["name"] for u in universities]

        # Save to user memory
        self.memory.update_user_data(user_id, {
            "interests": state.specific_interests,
            "recommended_universities": state.recommended_universities
        })

        # Generate final recommendation
        prompt = f"""
        Based on student's interests in {state.field}
        (specifically: {', '.join(state.specific_interests)}),

        Here are the matching universities:
        {format_universities(universities)}

        Provide a personalized recommendation with:
        1. Top 3 universities and why
        2. Programs to consider
        3. Next steps (open days, application deadlines)

        Response format (Czech):
        """

        return await generate_response(prompt)
```

**Similarly:**
- `career_skills_plugin.py` (200 LOC)
- `career_values_plugin.py` (200 LOC)

**Database Service:**

```python
# services/database_service.py (400 LOC)

import asyncpg
from typing import List, Dict

class DatabaseService:
    def __init__(self, connection_string: str):
        self.pool = None
        self.connection_string = connection_string

    async def init_pool(self):
        self.pool = await asyncpg.create_pool(self.connection_string)

    async def search_universities(
        self,
        field: str = None,
        specializations: List[str] = None,
        location: str = None,
        min_rating: float = None
    ) -> List[Dict]:
        """
        Search universities with filters
        """
        query = """
            SELECT
                id, name, location, website,
                programs, admission_info, contact
            FROM universities
            WHERE 1=1
        """
        params = []

        if field:
            query += " AND programs @> %s::jsonb"
            params.append(json.dumps([{"field": field}]))

        if specializations:
            # JSONB array containment
            query += " AND programs @> %s::jsonb"
            params.append(json.dumps([{"specializations": specializations}]))

        if location:
            query += " AND location ILIKE %s"
            params.append(f"%{location}%")

        async with self.pool.acquire() as conn:
            rows = await conn.fetch(query, *params)
            return [dict(row) for row in rows]

    async def get_admission_info(
        self,
        university_id: str,
        program: str
    ) -> Dict:
        """
        Get admission requirements for specific program
        """
        query = """
            SELECT
                name,
                admission_info->%s as program_admission
            FROM universities
            WHERE id = %s
        """

        async with self.pool.acquire() as conn:
            row = await conn.fetchrow(query, program, university_id)
            return dict(row) if row else None

    async def compare_programs(
        self,
        university_ids: List[str],
        program: str
    ) -> List[Dict]:
        """
        Compare same program across multiple universities
        """
        query = """
            SELECT
                name,
                location,
                programs->%s as program_details,
                admission_info->%s as admission
            FROM universities
            WHERE id = ANY(%s)
        """

        async with self.pool.acquire() as conn:
            rows = await conn.fetch(query, program, program, university_ids)
            return [dict(row) for row in rows]
```

**UI Customization:**

```typescript
// frontend/src/components/ChatWindow.tsx (800 LOC custom)

// Extend Language Accelerator's ChatWindow with:
// 1. Scenario progress indicator
// 2. University cards display
// 3. Comparison table
// 4. Quiz results integration
// 5. Feedback widget

interface ScenarioProgress {
  scenario: 'interests' | 'skills' | 'values';
  step: number;
  totalSteps: number;
  completed: boolean;
}

const ChatWindow: React.FC = () => {
  const [scenarioProgress, setScenarioProgress] = useState<ScenarioProgress | null>(null);
  const [recommendedUniversities, setRecommendedUniversities] = useState<University[]>([]);

  // ... existing chat logic from Language Accelerator

  const handleMessage = async (message: string) => {
    const response = await api.sendMessage(message);

    // Check if response contains scenario progress
    if (response.state) {
      setScenarioProgress(response.state);
    }

    // Check if response contains university recommendations
    if (response.result?.recommended_universities) {
      const universities = await api.getUniversities(
        response.result.recommended_universities
      );
      setRecommendedUniversities(universities);
    }

    // ... rest of chat logic
  };

  return (
    <div className="chat-window">
      {scenarioProgress && (
        <ScenarioProgressBar progress={scenarioProgress} />
      )}

      <ChatMessages messages={messages} />

      {recommendedUniversities.length > 0 && (
        <UniversityCardsGrid universities={recommendedUniversities} />
      )}

      <ChatInput onSend={handleMessage} />

      <FeedbackWidget conversationId={conversationId} />
    </div>
  );
};
```

**Analytics Integration:**

```typescript
// services/analytics.ts (200 LOC)

import ReactGA from 'react-ga4';

export class AnalyticsService {
  constructor(measurementId: string) {
    ReactGA.initialize(measurementId);
  }

  trackChatStarted(userId: string, language: string) {
    ReactGA.event({
      category: 'Chat',
      action: 'started',
      label: language,
      value: userId
    });
  }

  trackMessageSent(
    userId: string,
    route: 'clu' | 'cqa' | 'fallback',
    intent?: string
  ) {
    ReactGA.event({
      category: 'Chat',
      action: 'message_sent',
      label: route,
      value: { userId, intent }
    });
  }

  trackScenarioCompleted(
    userId: string,
    scenario: string,
    duration: number,
    messageCount: number
  ) {
    ReactGA.event({
      category: 'Scenario',
      action: 'completed',
      label: scenario,
      value: { userId, duration, messageCount }
    });
  }

  trackChatAbandoned(
    userId: string,
    lastMessageType: 'user' | 'bot',
    duration: number,
    messageCount: number
  ) {
    ReactGA.event({
      category: 'Chat',
      action: 'abandoned',
      label: lastMessageType,
      value: { userId, duration, messageCount }
    });
  }

  trackFeedbackSubmitted(
    userId: string,
    conversationId: string,
    rating: number,
    comment?: string
  ) {
    ReactGA.event({
      category: 'Feedback',
      action: 'submitted',
      label: rating > 0 ? 'positive' : 'negative',
      value: { userId, conversationId, comment }
    });
  }
}
```

**Total custom development:** ~3,500 LOC

---

### Implementation Roadmap

#### Fáze 1: Foundation Setup (Týdny 1-2)

**Cíl:** Nastavit infrastrukturu a základní komponenty

**Úkoly:**

1. **Azure Resources Provisioning**
   ```bash
   # Deploy base infrastructure
   azd init -t hybrid-vschatbot
   azd up

   # Vytvoří:
   # - Azure AI Language (CLU + CQA + PII)
   # - Azure OpenAI (GPT-4o-mini deployment)
   # - PostgreSQL Flexible Server
   # - Azure AI Search
   # - App Service
   # - Application Insights
   # - Storage Account
   ```

2. **Database Schema Setup**
   ```sql
   -- Run init script
   psql -h <server> -U <user> -d vschatbot < init_schema.sql

   -- Includes:
   -- - conversations, messages (from Chat with Your Data)
   -- - users, preferences (extended)
   -- - universities (new)
   -- - feedback (from Chat with Your Data)
   ```

3. **Copy Core Components**
   ```bash
   # From Language Accelerator
   cp -r language-accelerator/src/backend/src/ backend/
   cp -r language-accelerator/infra/ infra/

   # From Chat with Your Data
   cp -r chat-with-data/infra/core/database/ infra/core/database/
   cp -r chat-with-data/code/frontend/ frontend/
   ```

**Deliverables:**
- ✅ Azure resources provisioned
- ✅ PostgreSQL schema deployed
- ✅ Base code structure ready

**Odhad času:** 2 týdny

---

#### Fáze 2: CLU/CQA Configuration (Týdny 3-4)

**Cíl:** Nastavit CLU intents a CQA FAQ

**Úkoly:**

1. **Define CLU Intents**
   ```json
   // infra/data/clu_import.json
   {
     "intents": [
       {
         "category": "CareerCounselingInterests",
         "examples": [
           // 50+ examples in Czech
         ]
       },
       {
         "category": "CareerCounselingSkills",
         "examples": [...]
       },
       {
         "category": "CareerCounselingValues",
         "examples": [...]
       },
       {
         "category": "UniversitySearch",
         "examples": [...]
       },
       {
         "category": "ProgramComparison",
         "examples": [...]
       }
     ],
     "entities": [
       {"category": "University", "list": [...]},
       {"category": "Field", "list": [...]},
       {"category": "Location", "list": [...]}
     ]
   }
   ```

2. **Define CQA FAQ**
   ```json
   // infra/data/cqa_import.json
   {
     "qnaList": [
       {
         "id": 1,
         "answer": "Přijímací zkoušky probíhají v červnu...",
         "questions": [
           "Kdy jsou přijímací zkoušky?",
           "Termíny přijímaček",
           ...
         ]
       },
       // 20-30 FAQ pairs
     ]
   }
   ```

3. **Deploy & Train**
   ```bash
   # Run setup scripts
   python infra/scripts/language/clu_setup.py
   python infra/scripts/language/cqa_setup.py

   # Training time: 10-20 minut
   ```

4. **Test CLU/CQA**
   ```bash
   # Test CLU
   curl -X POST https://<endpoint>/analyze \
     -d '{"text": "Chci studovat informatiku"}'

   # Expected: Intent = CareerCounselingInterests, confidence > 0.8

   # Test CQA
   curl -X POST https://<endpoint>/qna \
     -d '{"question": "Kdy jsou přijímačky?"}'

   # Expected: Answer = "Přijímací zkoušky...", confidence > 0.7
   ```

**Deliverables:**
- ✅ CLU project trained (5 intents)
- ✅ CQA project deployed (20-30 FAQ)
- ✅ Test results documented

**Odhad času:** 2 týdny

---

#### Fáze 3: Custom Plugins Development (Týdny 5-7)

**Cíl:** Implementovat kariérové scénáře a university search

**Úkoly:**

1. **Career Counseling Plugins**
   ```python
   # custom_plugins/career_interests_plugin.py
   # custom_plugins/career_skills_plugin.py
   # custom_plugins/career_values_plugin.py

   # Each ~200 LOC
   # Total: 600 LOC
   ```

2. **University Search Plugins**
   ```python
   # custom_plugins/university_search_plugin.py
   # custom_plugins/program_comparison_plugin.py

   # Total: 300 LOC
   ```

3. **Database Service**
   ```python
   # services/database_service.py
   # 400 LOC
   ```

4. **Memory Service**
   ```python
   # services/memory_service.py
   # Integration with PostgreSQL
   # 200 LOC
   ```

5. **Semantic Kernel Agent Configuration**
   ```python
   # Update semantic_kernel_orchestrator.py
   # Register new plugins, configure agents
   ```

6. **Testing**
   ```python
   # tests/test_career_plugins.py
   # tests/test_university_search.py
   # tests/integration/test_scenarios.py

   # Total: 500 LOC tests
   ```

**Deliverables:**
- ✅ 5 custom plugins implemented
- ✅ Database service operational
- ✅ Memory service integrated
- ✅ Unit + integration tests passing

**Odhad času:** 3 týdny

---

#### Fáze 4: Frontend Integration (Týdny 8-9)

**Cíl:** Přizpůsobit UI pro VSChatBot

**Úkoly:**

1. **Rebrand UI**
   ```typescript
   // Update branding, colors, logo
   // frontend/src/config/theme.ts
   ```

2. **Scenario Progress Component**
   ```typescript
   // frontend/src/components/ScenarioProgressBar.tsx
   // Visual indicator of multi-step scenario
   ```

3. **University Cards**
   ```typescript
   // frontend/src/components/UniversityCard.tsx
   // Display recommended universities
   ```

4. **Comparison Table**
   ```typescript
   // frontend/src/components/ProgramComparisonTable.tsx
   // Side-by-side university comparison
   ```

5. **Feedback Widget**
   ```typescript
   // Copy from Chat with Your Data
   // frontend/src/components/FeedbackWidget.tsx
   ```

6. **Analytics Integration**
   ```typescript
   // frontend/src/services/analytics.ts
   // GA4 event tracking
   ```

**Deliverables:**
- ✅ UI rebranded
- ✅ New components functional
- ✅ GA4 tracking active
- ✅ Responsive design

**Odhad času:** 2 týdny

---

#### Fáze 5: Testing & Optimization (Týdny 10-11)

**Cíl:** End-to-end testing a optimalizace

**Úkoly:**

1. **End-to-end Testing**
   ```python
   # tests/e2e/test_full_scenarios.py

   def test_career_interests_scenario():
       # 1. User starts chat
       # 2. CLU detects CareerCounselingInterests
       # 3. Multi-step conversation
       # 4. University recommendations
       # 5. Feedback submitted
       pass
   ```

2. **Prompt Optimization**
   ```python
   # Implement 3-level caching
   # config/prompt_config.py

   CACHE_CONFIG = {
       "system_prompt": {
           "ttl": 3600,  # 1h
           "content": BASE_SYSTEM_PROMPT
       },
       "scenario_prompts": {
           "ttl": 3600,  # 1h
           "content": SCENARIO_PROMPTS
       },
       "user_memory": {
           "ttl": 300,  # 5min
       }
   }
   ```

3. **Performance Testing**
   ```bash
   # Load test with 100 concurrent users
   locust -f tests/load/test_chat.py --users 100

   # Metrics to track:
   # - Response time (target: < 2s for CLU, < 5s for scenarios)
   # - Token usage
   # - Database query time
   # - Cache hit rate
   ```

4. **Cost Optimization**
   ```python
   # Analyze token usage
   # scripts/analyze_token_usage.py

   # Expected:
   # - Cache hit rate: > 50%
   # - Average tokens per conversation: 15,000
   # - Cost per conversation: $0.10
   ```

**Deliverables:**
- ✅ All e2e tests passing
- ✅ Prompt caching optimized
- ✅ Performance targets met
- ✅ Cost projections validated

**Odhad času:** 2 týdny

---

#### Fáze 6: Production Launch (Týden 12)

**Cíl:** Deploy do produkce a monitoring

**Úkoly:**

1. **Production Deployment**
   ```bash
   # Deploy to production environment
   azd deploy --environment production

   # Enable auto-scaling
   az webapp update --name vschatbot-prod \
     --resource-group vschatbot-rg \
     --min-instances 2 \
     --max-instances 10
   ```

2. **Monitoring Setup**
   ```bash
   # Application Insights alerts
   # - Response time > 5s
   # - Error rate > 5%
   # - Token usage anomalies

   # GA4 custom dashboards
   # - Conversion funnel
   # - Scenario completion rates
   # - User engagement
   ```

3. **Documentation**
   ```markdown
   # docs/
   ├── DEPLOYMENT.md
   ├── TROUBLESHOOTING.md
   ├── MONITORING.md
   └── USER_GUIDE.md
   ```

4. **Training**
   - Admin training (scenario management)
   - Support team training (troubleshooting)

**Deliverables:**
- ✅ Production deployment live
- ✅ Monitoring active
- ✅ Documentation complete
- ✅ Team trained

**Odhad času:** 1 týden

---

### Total Timeline: 12 týdnů (3 měsíce)

| Fáze | Týdny | Deliverable |
|------|-------|-------------|
| 1. Foundation | 1-2 | Infrastructure ready |
| 2. CLU/CQA | 3-4 | Intents trained |
| 3. Plugins | 5-7 | Scenarios working |
| 4. Frontend | 8-9 | UI complete |
| 5. Testing | 10-11 | Production-ready |
| 6. Launch | 12 | Live |

---

## Cost Analysis

### Development Costs

| Fáze | Týdny | Hodinová sazba | Hodiny | Náklad |
|------|-------|----------------|--------|--------|
| Foundation | 2 | $80 | 80 | $6,400 |
| CLU/CQA Config | 2 | $80 | 80 | $6,400 |
| Custom Plugins | 3 | $100 | 120 | $12,000 |
| Frontend | 2 | $80 | 80 | $6,400 |
| Testing | 2 | $80 | 80 | $6,400 |
| Launch | 1 | $80 | 40 | $3,200 |
| **Celkem** | **12** | | **480** | **$40,800** |

**Poznámka:** Cena je nižší než custom solution ($80-120K) díky 70% code reuse.

---

### Monthly Operating Costs (1,000 uživatelů)

| Služba | SKU | Popis | Měsíční náklad |
|--------|-----|-------|----------------|
| **Azure AI Language** | S tier | CLU + CQA + PII | $300 |
| **Azure OpenAI** | Pay-as-you-go | GPT-4o-mini (3K konverzací) | $400 |
| **Azure AI Agent Service** | Pay-as-you-go | Semantic Kernel orchestration | $150 |
| **PostgreSQL** | Flexible Server B2s | 2 vCPU, 4 GB RAM | $60 |
| **Azure AI Search** | Basic | RAG fallback index | $75 |
| **App Service** | P1v3 | 2 instances (auto-scale) | $160 |
| **Storage** | Standard LRS | Blobs, logs | $15 |
| **Application Insights** | Pay-as-you-go | Monitoring, logs | $40 |
| **Azure Container Registry** | Basic | Docker images | $5 |
| **Bandwidth** | Egress | Outbound traffic | $20 |
| **Backup** | GRS | Database backups | $25 |
| **Total** | | | **$1,250** |

**Breakdown by scenario:**

| Uživatelů | Konverzace/měsíc | Token cost | Azure services | **Total** |
|-----------|------------------|------------|----------------|-----------|
| 320 (Rok 1) | 960 | $130 | $800 | **$930** |
| 1,000 (Rok 2) | 3,000 | $400 | $850 | **$1,250** |
| 2,000 (Rok 3) | 6,000 | $800 | $1,100 | **$1,900** |

---

### ROI Analysis

**Investice:**
- Vývoj: $40,800
- Rok 1 provoz (320 users): $930 × 12 = $11,160
- **Total Rok 1:** $51,960

**Příjmy (předpoklad $10/user/měsíc):**
- Rok 1: 320 × $10 × 12 = $38,400
- Rok 2: 1,000 × $10 × 12 = $120,000
- Rok 3: 2,000 × $10 × 12 = $240,000

**Zisk:**
- Rok 1: -$13,560 (investice)
- Rok 2: +$105,000
- Rok 3: +$217,200

**Break-even:** Měsíc 16 (Rok 2, Q2)

---

## Závěr

### Shrnutí doporučení

**✅ Použít HYBRID ARCHITEKTURU:**

1. **Základ:** Azure Language OpenAI Conversational Agent Accelerator
   - UnifiedConversationOrchestrator
   - Semantic Kernel group chat
   - CLU/CQA routing
   - PII redaction
   - TranslationAgent

2. **Přidat komponenty z Chat with Your Data:**
   - PostgreSQL long-term memory
   - Admin panel
   - Feedback system
   - Azure AI Search pro RAG fallback

3. **Custom development:**
   - 5 Semantic Kernel plugins (career scenarios + university search)
   - Database service
   - Memory service
   - UI customization
   - GA4 analytics

**Výhody tohoto přístupu:**

| Kritérium | Hodnota |
|-----------|---------|
| **Code reuse** | 70% (4,500 / 8,000 LOC) |
| **Development time** | 12 týdnů |
| **Development cost** | $40,800 |
| **Monthly cost (1K users)** | $1,250 |
| **Deterministické workflow** | ✅ CLU 95% accuracy |
| **Long-term memory** | ✅ PostgreSQL |
| **Jazyková podpora** | ✅ CZ/SK automatic |
| **Rozšiřitelnost** | ✅ JSON config (no code) |
| **GDPR compliance** | ✅ PII redaction |
| **Production-ready** | ✅ Azure native |

**Proč tento přístup?**

1. **Best of both worlds** - kombinuje deterministický routing (Language Acc) s robustní memory (Chat with Data)
2. **Fast time-to-market** - 12 týdnů vs 20+ týdnů custom solution
3. **Cost-effective** - $40K vývoj vs $80-120K custom
4. **Proven patterns** - 2 production-tested accelerators
5. **Azure native** - perfektní integrace, škálovatelnost
6. **Extensible** - snadné přidávání scénářů přes CLU JSON

---

### Další kroky

1. **Schválení architektury** - review tohoto dokumentu
2. **Team assembly** - 2-3 developers, 1 DevOps
3. **Azure subscription setup** - quota check, resource limits
4. **Kickoff meeting** - Fáze 1 start (Týden 1)
5. **Sprint planning** - 2-week sprints, 6 sprints total

**Doporučený tým:**

| Role | FTE | Fáze |
|------|-----|------|
| Senior Backend Developer | 1.0 | 1-6 |
| Frontend Developer | 0.5 | 4-5 |
| DevOps Engineer | 0.3 | 1, 6 |
| QA Engineer | 0.3 | 5 |
| Project Manager | 0.2 | 1-6 |

**Total FTE:** ~2.3 FTE × 12 týdnů = ~27 person-weeks

---

## Přílohy

### A. CLU Intent Examples (Czech)

```json
{
  "intents": [
    {
      "category": "CareerCounselingInterests",
      "examples": [
        "Chci studovat informatiku",
        "Zajímá mě ekonomie",
        "Nevím co studovat, ale baví mě matematika",
        "Chtěl bych dělat něco s umělou inteligencí",
        "Zajímá mě programování",
        "Líbí se mi práce s daty",
        "Chci studovat něco technického",
        "Baví mě fyzika a chemie",
        "Zajímá mě medicína",
        "Chtěl bych být lékař",
        "Zajímá mě právo",
        "Chci studovat psychologii",
        "Baví mě jazyky",
        "Zajímá mě marketing",
        "Chci dělat něco kreativního"
      ]
    },
    {
      "category": "CareerCounselingSkills",
      "examples": [
        "Jsem dobrý v matematice",
        "Umím dobře programovat",
        "Jsem komunikativní",
        "Mám rád týmovou práci",
        "Umím dobře prezentovat",
        "Jsem analytický typ",
        "Jsem kreativní",
        "Umím řešit problémy",
        "Jsem detailista",
        "Umím vést lidi"
      ]
    },
    {
      "category": "CareerCounselingValues",
      "examples": [
        "Chci pomáhat lidem",
        "Je pro mě důležitý work-life balance",
        "Zajímá mě kariéra s dopadem na společnost",
        "Chci dobré finanční ohodnocení",
        "Důležitá je pro mě stabilita",
        "Chci cestovat",
        "Zajímá mě mezinárodní kariéra",
        "Chci být nezávislý",
        "Důležitý je pro mě osobní rozvoj"
      ]
    },
    {
      "category": "UniversitySearch",
      "examples": [
        "Kdy jsou dny otevřených dveří na MFF UK?",
        "Jaké jsou podmínky přijetí na ČVUT?",
        "Které školy mají obor kybernetika?",
        "Kde můžu studovat informatiku v Praze?",
        "Kolik stojí studium na VŠE?",
        "Jaké jsou termíny přihlášek na FIT VUT?",
        "Je na ČVUT těžké se dostat?",
        "Jaká je úspěšnost přijetí na medicínu?"
      ]
    },
    {
      "category": "ProgramComparison",
      "examples": [
        "Porovnej mi informatiku na ČVUT a VUT",
        "Která škola má lepší ekonomii?",
        "Rozdíl mezi FIT VUT a MFF UK",
        "Kde je lepší studovat právo?",
        "Porovnání medicíny na UK a Masarykově univerzitě"
      ]
    }
  ]
}
```

### B. CQA FAQ Examples (Czech)

```json
{
  "qnaList": [
    {
      "id": 1,
      "answer": "Přijímací zkoušky na vysoké školy v České republice probíhají většinou v červnu. Konkrétní termíny se liší podle školy a fakulty. Doporučuji sledovat oficiální stránky konkrétní školy, kam se chceš hlásit.",
      "questions": [
        "Kdy jsou přijímací zkoušky?",
        "Termíny přijímaček",
        "Kdy se konají přijímačky na vysoké školy?",
        "V jakém měsíci jsou přijímací zkoušky?"
      ]
    },
    {
      "id": 2,
      "answer": "Základní podmínkou je maturitní zkouška. Některé školy vyžadují specifické předměty u maturity (např. matematika pro technické obory). Dále může být požadována přijímací zkouška, NSZ (Národní srovnávací zkoušky) nebo pohovor. Každá škola má své vlastní podmínky.",
      "questions": [
        "Co potřebuji k přihlášce na VŠ?",
        "Podmínky přijetí na vysokou školu",
        "Jaké jsou požadavky pro VŠ?",
        "Co je potřeba k přijetí na vysokou?"
      ]
    },
    {
      "id": 3,
      "answer": "Studium na veřejných vysokých školách v České republice je zdarma, pokud studujete v rámci standardní doby studia. Poplatky se platí až při překročení standardní doby nebo při studiu více programů současně. Soukromé školy si účtují školné, které se obvykle pohybuje od 20 000 do 100 000 Kč ročně.",
      "questions": [
        "Kolik stojí vysoká škola?",
        "Je studium na VŠ zdarma?",
        "Musím platit školné?",
        "Jaké jsou poplatky za studium?"
      ]
    },
    {
      "id": 4,
      "answer": "Přihlášky na vysoké školy se podávají online přes systém školy, většinou od listopadu do února/března. Přesné termíny závisí na konkrétní škole. Obvykle můžeš podat přihlášky na více škol a oborů současně.",
      "questions": [
        "Kdy se podávají přihlášky?",
        "Termín podání přihlášky",
        "Do kdy můžu podat přihlášku?",
        "Jak se podává přihláška na VŠ?"
      ]
    },
    {
      "id": 5,
      "answer": "Národní srovnávací zkoušky (NSZ) jsou jednotné přijímací testy organizované Scio nebo Cermat. Některé školy je akceptují místo vlastních přijímacích zkoušek. Výhodou je, že můžeš jeden test použít pro přihlášky na více škol. NSZ se konají od února do června.",
      "questions": [
        "Co jsou NSZ?",
        "Co je národní srovnávací zkouška?",
        "Jak funguje NSZ?",
        "Musím dělat NSZ?"
      ]
    }
  ]
}
```

### C. University Database Schema

```sql
-- universities table
CREATE TABLE universities (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    short_name VARCHAR(50),
    type VARCHAR(50) CHECK (type IN ('public', 'private', 'state')),
    location VARCHAR(255),
    region VARCHAR(100),
    website VARCHAR(500),
    founded_year INTEGER,

    -- Contact info (JSONB)
    contact JSONB DEFAULT '{
      "email": "",
      "phone": "",
      "address": {
        "street": "",
        "city": "",
        "zip": ""
      }
    }'::jsonb,

    -- Programs (JSONB array)
    programs JSONB DEFAULT '[]'::jsonb,
    /* Example:
    [
      {
        "name": "Informatika",
        "degree": "Bc.",
        "field": "Computer Science",
        "language": "CS",
        "duration_years": 3,
        "admission_requirements": {
          "maturita": ["matematika", "informatika"],
          "exam_type": "NSZ",
          "min_score": 70
        },
        "tuition_fee": 0,
        "capacity": 200,
        "specializations": ["AI", "Cybersecurity", "Software Engineering"]
      }
    ]
    */

    -- Open days (JSONB array)
    open_days JSONB DEFAULT '[]'::jsonb,
    /* Example:
    [
      {
        "date": "2025-11-15",
        "type": "in-person",
        "registration_required": true,
        "url": "https://..."
      }
    ]
    */

    -- Statistics (JSONB)
    stats JSONB DEFAULT '{}'::jsonb,
    /* Example:
    {
      "students_total": 5000,
      "international_students_pct": 15,
      "acceptance_rate": 0.35,
      "employment_rate": 0.92
    }
    */

    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_universities_name ON universities(name);
CREATE INDEX idx_universities_location ON universities(location);
CREATE INDEX idx_universities_programs ON universities USING GIN (programs);
CREATE INDEX idx_universities_type ON universities(type);

-- Full-text search
CREATE INDEX idx_universities_search ON universities USING GIN (
    to_tsvector('czech', name || ' ' || COALESCE(location, ''))
);
```

### D. Semantic Kernel Agent Definitions

```json
// Azure AI Foundry agent definitions

{
  "agents": [
    {
      "id": "translation-agent",
      "name": "TranslationAgent",
      "description": "Translates user input from Czech/Slovak to English and responses back",
      "model": "gpt-4o-mini",
      "instructions": "You are a translation agent. Translate user input to English for processing. Translate final responses back to the original language (Czech or Slovak). Preserve the meaning and tone. Return JSON format: {\"response\": \"translated text\", \"original_language\": \"cs/sk\"}",
      "tools": []
    },
    {
      "id": "triage-agent",
      "name": "TriageAgent",
      "description": "Routes inquiries using CLU and CQA",
      "model": "claude-haiku-4-20250514",
      "instructions": "You are a triage agent. Use CLU to detect intents and CQA to answer FAQs. Return results in JSON format. If CLU detects an intent with confidence > 0.8, return {\"type\": \"clu_result\", \"response\": {...}}. If CQA answers with confidence > 0.7, return {\"type\": \"cqa_result\", \"response\": {...}}. Otherwise return null for fallback.",
      "tools": [
        {
          "type": "function",
          "function": {
            "name": "analyze_intent_clu",
            "description": "Analyze user intent using CLU",
            "parameters": {
              "type": "object",
              "properties": {
                "text": {"type": "string"},
                "language": {"type": "string"}
              }
            }
          }
        },
        {
          "type": "function",
          "function": {
            "name": "get_answer_cqa",
            "description": "Get answer from CQA knowledge base",
            "parameters": {
              "type": "object",
              "properties": {
                "question": {"type": "string"},
                "language": {"type": "string"}
              }
            }
          }
        }
      ]
    },
    {
      "id": "head-support-agent",
      "name": "HeadSupportAgent",
      "description": "Selects the appropriate custom agent based on CLU intent",
      "model": "gpt-4o-mini",
      "instructions": "Based on the CLU intent result, route to the appropriate custom agent. Intent mapping: CareerCounselingInterests → CareerInterestsAgent, CareerCounselingSkills → CareerSkillsAgent, CareerCounselingValues → CareerValuesAgent, UniversitySearch → UniversitySearchAgent, ProgramComparison → ProgramComparisonAgent. Return JSON: {\"target_agent\": \"AgentName\"}",
      "tools": []
    },
    {
      "id": "career-interests-agent",
      "name": "CareerInterestsAgent",
      "description": "Runs career counseling scenario based on student interests",
      "model": "gpt-4o-mini",
      "instructions": "You are a career counselor helping students discover their interests. Run multi-step conversation: 1) Identify broad field, 2) Drill down to specific interests, 3) Recommend universities. Use the plugin to access user memory and university database. Be conversational and supportive. Speak in Czech. Return JSON: {\"response\": \"message\", \"need_more_info\": true/false}",
      "tools": [
        {
          "type": "function",
          "function": {
            "name": "run_interests_scenario",
            "description": "Run interests-based career counseling scenario"
          }
        }
      ]
    },
    {
      "id": "university-search-agent",
      "name": "UniversitySearchAgent",
      "description": "Searches universities based on criteria",
      "model": "gpt-4o-mini",
      "instructions": "Search for universities matching user criteria. Use the database plugin. Provide detailed information about programs, admission requirements, open days. Speak in Czech. Return JSON: {\"response\": \"message\"}",
      "tools": [
        {
          "type": "function",
          "function": {
            "name": "search_universities",
            "description": "Search universities by field, location, etc."
          }
        },
        {
          "type": "function",
          "function": {
            "name": "get_admission_info",
            "description": "Get admission requirements for specific program"
          }
        }
      ]
    }
  ]
}
```

---

**Konec analýzy**

**Autor:** Claude (Anthropic)
**Verze:** 1.0
**Datum:** 2025-10-04
