# Implementační strategie splňující rozšířené požadavky

Dokument rozšiřuje varianty z `codex-vschatbot-solutions.md` o jednotný rámec schopností, které musí každá budoucí realizace zahrnout: (1) RAG, (2) MCP server pro přístup k databázovým datům, (3) prompt caching, (4) context caching, (5) paměť, (6) citace zdrojů, (7) offline ingestace aktualizovaných webových dat.

## MVP – „rychlý průvodce studiem“
- **RAG:** Azure AI Search ("On Your Data") s indexem nad statickými FAQ a exporty z `vs-weby`; embedding pipeline řízená skripty pro denní obnovu.
- **MCP server:** Jednoduchý Python MCP server nad REST API/SQL dotazy, publikující čtecí operace (např. seznam termínů DOD). Agent jej volá funkcí `tool_choice=required` u dotazů na interní data.
- **Prompt caching:** Využití OpenAI prompt caching pro statickou systémovou instrukci (scénář onboarding + guardrails). Cache ID se sdílí napříč sezeními.
- **Context caching:** Uchování sumarizovaného kontextu posledních N zpráv (L2 cache) v Redis/Azure Cache; při startu se dotahuje z DB místo znovugenerování.
- **Paměť:** Krátkodobá shrnutí konverzace (OpenAI trimming session) ukládaná do Cosmos DB/PostgreSQL per uživatele; přístup přes MCP.
- **Citace zdrojů:** Šablona odpovědi vynucuje JSON s polem `citace`, doplněným na základě metadata z AI Search (URL, datum). Frontend je vykreslí.
- **Offline update:** Azure Function 1× denně stahuje zdroje z `vs-weby`, validuje změny (hash, datum), reindexuje AI Search; chatbot jen čte aktuální index.

## Rozšířené řešení – „scénářový konzultant“
- **RAG:** Kombinace Azure AI Search (vektor + fulltext) a tabulkového RAG na PostgreSQL pomocí `pgvector`; orchestrátor vybírá zdroj dle typu dotazu.
- **MCP server:** Semantic Kernel Agent Framework exponuje interní moduly (uživatelský profil, kvízy) jako MCP capability; využívá Managed Identity pro bezpečný přístup k DB.
- **Prompt caching:** Sdílený systémový prompt + šablony pro scénáře se cachují (Claude prompt caching nebo OpenAI shared caches). Při změně scénářů se invalidují verzí.
- **Context caching:** Konverzační text se ukládá v Azure Table Storage + vektorový embedding pro rychlé znovuvložení. Orchestrátor kontroluje, zda lze vrátit z cache.
- **Paměť:** Hybrid – krátkodobá LLM sumarizace, dlouhodobé preference v DB; metadata (např. preferované obory) se vkládají do `memory` sekce systémového promptu.
- **Citace zdrojů:** Generátor pracuje v módu `strict json`, kde `citace` obsahuje seznam {"zdroj": URL, "popis": text, "timestamp": datum}. UI kontroluje existenci odkazů.
- **Offline update:** Data pipeline (Azure Data Factory) stahuje HTML/CSV z `vs-weby`, provádí extrakci (BeautifulSoup/Playwright), loguje dify; výsledky ukládá do Data Lake → trigger reindexace.

## Pokročilý multiagent – „kariérní poradenské studio“
- **RAG:** Víceindeksová strategie – obecná znalost (Azure AI Search), detailní tabulky (PostgreSQL), znalostní graf (Neo4j). Orchestrátor (manager agent) rozhoduje, které komponenty volat, a agreguje výsledky.
- **MCP server:** Vrstva MCP nad mikroslužbami (studijní programy, akce, analytics). Každý specializovaný agent má deklarativně povolené jen relevantní nástroje, volání auditována (App Insights).
- **Prompt caching:** Caching systémového promptu i scénářových skriptů (OpenAI `cached_prompt_id`) + dedikované cache pro eval agenty; invalidace řízena verzemi promptů.
- **Context caching:** Kontext jednotlivých agentů ukládán do centralizovaného store (Redis + Cosmos DB); shrnutí se sdílí mezi agenty pomocí "context tokens" (viz OpenAI multi-agent demo).
- **Paměť:** Trvalá dlouhodobá paměť (profil) + episodic memory. Memory agent periodicky generuje shrnutí, ukládá do vektorové DB; manager agent dotahuje při startu konverzace.
- **Citace zdrojů:** Každý agent je nucen vracet `citace` se `score` a typem zdroje. Před odesláním uživateli prochází odpověď validačním agentem, který kontroluje existenci a relevanci.
- **Offline update:** Data orchestrátor (ADF/Databricks) + LLM eval pipeline (Azure AI Evaluation) kontrolují nové verze stránek, extrahují strukturovaná data, generují diff reporty, po schválení aktualizují indexy.

## Enterprise++ – „multikanálový poradenský ekosystém“
- **RAG:** Kombinace znalostního grafu (Neo4j), dokumentových indexů (AI Search), reálných API (CRM) a event streamu. Planner agent využívá meta-ranker (RRF + reranker). 
- **MCP server:** Vrstvené MCP (HTTP, stdio) pro různé oddělení (CRM, marketing, statistiky). Kompliance agent monitoruje logy a enforceuje politiky.
- **Prompt caching:** Distribuované cache ID napříč modely (Azure OpenAI + proxy pro Claude) s fallbackem; Prompt Flow orchestruje invalidace dle verzování scénářů.
- **Context caching:** Hierarchická (per-kanál, per-agent). Hlavní orchestrátor si udržuje "context map" – sumarizace, které se revalidují pomocí LLM guard agenta.
- **Paměť:** Multi-store – vektorový knowledge vault, relační historické interakce, event sourcing pro konzistenci napříč kanály. Automatické zpracování preferencí do doporučovacích modelů.
- **Citace zdrojů:** Strukturovaná odpověď s citacemi + metadata (confidence, channel). Citace se zobrazují i v GA4 eventech a exportech pro poradce.
- **Offline update:** Automatizované crawler clustery (Azure Container Apps) sbírají nová data, validátor (LLM) generuje QA checklist, po schválení se aktualizují indexy + verzovaný changelog. Přidána notifikace pro agenty, že došlo k update.

---
Všechny varianty tak připravují infrastrukturu, která splňuje požadavek na RAG, MCP přístup k databázi, caching (prompt/context), paměť, citování zdrojů a oddělený offline mechanismus aktualizace znalostí.
