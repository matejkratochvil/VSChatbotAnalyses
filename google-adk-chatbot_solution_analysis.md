# Analýza řešení AI chatbotu pro projekt "Chci na vejšku"

## Přehled požadavků

Na základě specifikace v `AI_chat_specifikace.md` a analýzy dostupných příkladů z OpenAI Cookbook, navrhujeme několik úrovní řešení chatbotu pro pomoc studentům s výběrem vysoké školy.

### Klíčové požadavky:
- **Dlouhodobá paměť** - personalizované odpovědi na základě předchozích interakcí
- **Dynamické zpracování scénářů** - 3 připravené scénáře od kariérových poradců
- **Rychlost a kvalita** - rychlé a užitečné odpovědi
- **Web Search** (volitelné) - vyhledávání aktuálních informací
- **Podpora jazyků** - CZ a SK
- **Azure infrastruktura** (preferováno)
- **Zpětná vazba** - měření úspěšnosti
- **Prioritizace zdrojů** - DB > knowledge base > web search
- **Optimalizace nákladů** - cachování, flexibilní řešení

## Analýza dostupných technik z OpenAI Cookbook

### 1. Assistants API (Doporučeno pro MVP)
**Zdroj:** `Assistants_API_overview_python.ipynb`

**Klíčové vlastnosti:**
- Stateful konverzace (Threads)
- Vestavěné nástroje (Code Interpreter, File Search, Functions)
- Asynchronní zpracování
- Automatické řízení konverzace

**Implementace:**
```python
# Vytvoření asistenta
assistant = client.beta.assistants.create(
    name="Kariérový poradce",
    instructions="Jste kariérový poradce pomáhající studentům s výběrem vysoké školy...",
    model="gpt-4o",
    tools=[{"type": "code_interpreter"}, {"type": "file_search"}]
)

# Vytvoření vlákna konverzace
thread = client.beta.threads.create()

# Přidání zprávy a spuštění
message = client.beta.threads.messages.create(
    thread_id=thread.id,
    role="user",
    content="Potřebuji pomoc s výběrem vysoké školy"
)

run = client.beta.threads.runs.create(
    thread_id=thread.id,
    assistant_id=assistant.id
)
```

### 2. Function Calling s externími API
**Zdroj:** `Function_calling_finding_nearby_places.ipynb`

**Klíčové vlastnosti:**
- Integrace s externími databázemi
- Dynamické volání funkcí
- Profilování uživatelů
- Personalizované doporučení

**Implementace:**
```python
def search_universities(user_id, study_field, location_preference):
    # Načtení profilu uživatele
    customer_profile = fetch_customer_profile(user_id)
    
    # Vyhledání v databázi univerzit
    # Implementace logiky pro doporučení
    
    return university_recommendations

# Definice funkce pro LLM
tools = [
    {
        "type": "function",
        "function": {
            "name": "search_universities",
            "description": "Vyhledá vhodné univerzity na základě preferencí studenta",
            "parameters": {
                "type": "object",
                "properties": {
                    "study_field": {"type": "string"},
                    "location_preference": {"type": "string"}
                }
            }
        }
    }
]
```

### 3. RAG (Retrieval Augmented Generation)
**Zdroj:** `Embedding_Wikipedia_articles_for_search.ipynb`, `Code_search_using_embeddings.ipynb`

**Klíčové vlastnosti:**
- Vektorové vyhledávání v knowledge base
- Kontextově relevantní odpovědi
- Škálovatelnost pro velké databáze

**Implementace:**
```python
# Vytvoření embeddings pro univerzitní data
def create_university_embeddings(university_data):
    embeddings = []
    for university in university_data:
        embedding = client.embeddings.create(
            model="text-embedding-3-small",
            input=university['description']
        )
        embeddings.append(embedding.data[0].embedding)
    return embeddings

# Vyhledávání podobných univerzit
def search_similar_universities(query, university_embeddings, top_k=5):
    query_embedding = client.embeddings.create(
        model="text-embedding-3-small",
        input=query
    )
    
    similarities = cosine_similarity(query_embedding, university_embeddings)
    top_indices = np.argsort(similarities)[-top_k:][::-1]
    
    return top_indices
```

### 4. Context Summarization pro dlouhé konverzace
**Zdroj:** `Context_summarization_with_realtime_api.ipynb`

**Klíčové vlastnosti:**
- Automatické shrnování starších částí konverzace
- Zachování kontextu při dlouhých rozhovorech
- Optimalizace tokenů

**Implementace:**
```python
async def summarise_and_prune(ws, state):
    if state.latest_tokens >= SUMMARY_TRIGGER:
        old_turns, recent_turns = state.history[:-KEEP_LAST_TURNS], state.history[-KEEP_LAST_TURNS:]
        
        # Vytvoření shrnutí starších částí
        summary_text = await run_summary_llm(convo_text)
        
        # Aktualizace historie
        state.history[:] = [Turn("assistant", summary_id, summary_text)] + recent_turns
```

### 5. LangChain Agent Framework
**Zdroj:** `How_to_build_a_tool-using_agent_with_Langchain.ipynb`

**Klíčové vlastnosti:**
- Modulární architektura
- Kombinace více nástrojů
- Pokročilé řízení konverzace
- Memory management

## Návrh řešení - Úrovně implementace

### Úroveň 1: MVP (Minimální životaschopný produkt)
**Cíl:** Rychlé nasazení s minimálními náklady

**Architektura:**
- **Assistants API** jako základ
- **Statické scénáře** v promptu
- **Jednoduchá databáze** univerzit
- **Základní function calling** pro dotazy na DB

**Implementace:**
```python
# Základní asistent s vestavěnými scénáři
assistant = client.beta.assistants.create(
    name="Kariérový poradce MVP",
    instructions="""
    Jste kariérový poradce pomáhající studentům s výběrem vysoké školy.
    
    Máte k dispozici 3 scénáře:
    1. Scénář pro humanitní obory
    2. Scénář pro technické obory  
    3. Scénář pro přírodovědné obory
    
    Postupujte podle scénáře a pomozte studentovi zúžit výběr.
    """,
    model="gpt-4o-mini",  # Levnější model pro MVP
    tools=[{"type": "function", "function": search_universities_function}]
)
```

**Odhad nákladů (1 uživatel/měsíc):**
- Input tokens: ~15,000 (scénáře + konverzace)
- Output tokens: ~8,000
- **Celkové náklady: ~$0.15/měsíc**

### Úroveň 2: Rozšířené řešení
**Cíl:** Lepší personalizace a kvalita odpovědí

**Architektura:**
- **Assistants API** + **RAG**
- **Vektorová databáze** (Pinecone/Azure Search)
- **Function calling** pro dynamické dotazy
- **Základní memory** (ConversationBufferWindowMemory)

**Implementace:**
```python
# Kombinace RAG s Assistants API
def create_rag_assistant():
    # Vytvoření vektorové databáze
    vector_store = create_university_vector_store()
    
    # RAG nástroj
    rag_tool = {
        "type": "function",
        "function": {
            "name": "search_university_knowledge",
            "description": "Vyhledá relevantní informace o univerzitách",
            "parameters": {...}
        }
    }
    
    assistant = client.beta.assistants.create(
        name="Kariérový poradce RAG",
        instructions=enhanced_instructions,
        model="gpt-4o",
        tools=[rag_tool, search_db_tool, web_search_tool]
    )
```

**Odhad nákladů (1 uživatel/měsíc):**
- Input tokens: ~25,000
- Output tokens: ~12,000
- Embedding náklady: ~$0.05
- **Celkové náklady: ~$0.35/měsíc**

### Úroveň 3: Profesionální systém
**Cíl:** Plná funkcionalita s pokročilými funkcemi

**Architektura:**
- **Multi-agentní systém** (LangChain)
- **RAG** + **MCP servery**
- **Pokročilá memory** (ConversationSummaryBufferMemory)
- **Web scraping** pro aktuální data
- **Analytics** a monitoring

**Implementace:**
```python
# Multi-agentní systém
class CareerCounselorAgent:
    def __init__(self):
        self.rag_agent = RAGAgent()
        self.scenario_agent = ScenarioAgent()
        self.memory_agent = MemoryAgent()
        self.web_agent = WebSearchAgent()
    
    def process_query(self, user_input, user_context):
        # Rozhodnutí o tom, který agent použít
        if self.is_scenario_question(user_input):
            return self.scenario_agent.process(user_input, user_context)
        elif self.needs_current_data(user_input):
            return self.web_agent.process(user_input, user_context)
        else:
            return self.rag_agent.process(user_input, user_context)
```

**Odhad nákladů (1 uživatel/měsíc):**
- Input tokens: ~40,000
- Output tokens: ~20,000
- Embedding náklady: ~$0.10
- Web search náklady: ~$0.05
- **Celkové náklady: ~$0.70/měsíc**

## Detailní implementační plán

### Fáze 1: Příprava dat (2-3 týdny)
1. **Sběr dat o univerzitách**
   - Integrace s oficiálními zdroji (MŠMT, weby univerzit)
   - Strukturování dat podle požadovaných polí
   - Vytvoření embeddings pro RAG

2. **Příprava scénářů**
   - Spolupráce s kariérovými poradci
   - Testování a optimalizace scénářů
   - Převod do strukturovaného formátu

### Fáze 2: MVP implementace (3-4 týdny)
1. **Základní chatbot**
   - Assistants API integrace
   - Function calling pro databázi
   - Základní UI (web/chat widget)

2. **Databáze a backend**
   - Azure SQL Database
   - API pro správu dat
   - Základní analytics

### Fáze 3: Rozšíření funkcionalit (4-6 týdnů)
1. **RAG implementace**
   - Vektorová databáze (Azure Cognitive Search)
   - Optimalizace vyhledávání
   - Testování kvality odpovědí

2. **Memory a personalizace**
   - Ukládání konverzační historie
   - Profilování uživatelů
   - A/B testování

### Fáze 4: Pokročilé funkce (6-8 týdnů)
1. **Web search integrace**
   - Real-time data o univerzitách
   - Aktualizace informací
   - Zdrojování a citace

2. **Analytics a monitoring**
   - GA4 integrace
   - Měření úspěšnosti
   - Optimalizace na základě dat

## Nákladová analýza

### Pro 1,000 uživatelů/měsíc:

| Úroveň | Token náklady | Infrastruktura | Vývoj | Celkem/měsíc |
|--------|---------------|----------------|-------|--------------|
| MVP | $150 | $200 | $5,000 | $5,350 |
| Rozšířené | $350 | $400 | $8,000 | $8,750 |
| Profesionální | $700 | $800 | $15,000 | $16,500 |

### ROI analýza:
- **Rok 1:** 320 platících uživatelů × $10/měsíc = $38,400
- **Rok 2:** 1,000 uživatelů × $10/měsíc = $120,000
- **Rok 3:** 2,000 uživatelů × $10/měsíc = $240,000

## Doporučení

### Pro rychlé nasazení (leden 2026):
1. **Začít s MVP** - Assistants API + základní function calling
2. **Postupně přidávat funkce** - RAG, memory, web search
3. **Fokus na kvalitu dat** - investovat do přípravy scénářů a databáze univerzit

### Technologický stack:
- **Backend:** Python + FastAPI
- **Database:** Azure SQL Database + Azure Cognitive Search
- **AI:** OpenAI Assistants API + GPT-4o
- **Frontend:** React/Next.js
- **Hosting:** Azure App Service

### Klíčové metriky úspěchu:
- **Konverze:** % uživatelů, kteří dokončí scénář
- **Spokojenost:** Hodnocení zpětné vazby
- **Retention:** % uživatelů vracejících se
- **Náklady:** Token usage per user

Tento plán poskytuje flexibilní cestu od jednoduchého MVP po komplexní profesionální systém, s možností škálování podle potřeby a rozpočtu.