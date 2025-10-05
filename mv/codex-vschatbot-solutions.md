# Návrhy řešení pro VSChatBot

## Klíčové požadavky z `VSChatBot.md`
- Dlouhodobá paměť a personalizace (historie interakcí, preference z kvízů mimo chat).
- Dynamické scénáře vedené kariérovými poradci s možností snadného rozšiřování.
- Rychlé, kvalitní odpovědi v češtině/slovenštině; prioritizace vlastní databáze a oficiálních zdrojů.
- Volitelný web search, vícejazyčný onboarding, hostování preferovaně na Azure.
- Zpětná vazba po konverzaci, GA4 analytika, sledování nákladů a cachování promptů.
- Odhady tokenů, nákladů a vývoje (nutné do následného rozpracování).

## Varianty řešení

### 1. MVP – "rychlý průvodce studiem"
- **Cíl:** Ověřit hodnotu služby s minimem komponent, rychlá integrace do existujícího webu.
- **Architektura:**
  - Frontend: jednoduchý chat widget (React/Next.js podle vzoru `openai-cs-agents-demo/ui`).
  - Backend: Python FastAPI + OpenAI Chat Completions (`gpt-4o-mini`) hostované na Azure App Service.
  - Paměť: krátkodobá konverzační historie s trimming/sumarizací (`session_memory` pattern z OpenAI CookBook).
  - Data: staticky připravené FAQ + CSV s termíny (ruční export z `vs-weby`), embedované do Azure AI Search "on your data" nebo lokálního FAISS.
- **Scénáře:**
  - Předpřipravený "scénář průvodce výběrem školy" jako systémový prompt; ostatní dotazy přes RAG.
  - Automatická identifikace jazyka (Language Detection middleware z BotBuilder vzorků).
- **Analytika:** GA4 eventy + App Insights latence, ruční export zpětné vazby.
- **Výhody:** rychlá implementace (4–6 týdnů), nízké náklady, možnost nasadit před sezónou.
- **Limity:** žádná perzistentní dlouhodobá paměť, scénáře obtížně rozšiřitelné, omezená moderace.

### 2. Rozšířené řešení – "scénářový konzultant"
- **Cíl:** Přidat dlouhodobou paměť, modulární scénáře a automatizovanou správu dat.
- **Architektura:**
  - Orchestrátor v Azure Functions nebo Container Apps s modulárním routerem (inspirace `Azure-Language-OpenAI-Conversational-Agent-Accelerator`).
  - Paměť: kombinace vektorového úložiště (Azure AI Search / PostgreSQL pgvector) + relační DB (uživatelské profily, preference, logy kvízů).
  - Scénáře: definované JSON/YAML workflow (intent → kroky → kontrolní otázky), editovatelné poradci.
  - LLM: GPT-4o pro generování, menší model (GPT-4o-mini/Claude 3 Haiku) pro klasifikaci intentu.
  - Data pipeline: plánovaný scrape z `vs-weby`, transformace (Azure Data Factory / Logic Apps) → indexace do AI Search, kontrola aktuálnosti.
- **Funkce navíc:**
  - Zpětná vazba jako explicitní krok s uložení do DB a GA4 eventu.
  - Guardrails: relevance + jailbreak filtry (z `openai-cs-agents-demo`), PII redakce (Azure AI Language PII API).
  - Vícejazyčnost: BotBuilder překladové middleware / detekce jazyka + per-language system prompt.
- **Analytika:** GA4 + Application Insights + custom dashboard (Power BI / Azure Monitor workbook) pro tokeny a náklady.
- **Výhody:** škálovatelné scénáře, dlouhodobý profil, lepší kontrola kvality.
- **Limity:** vyšší komplexita, nutnost průběžné správy dat a pravidelných evaluací.

### 3. Pokročilý multiagent – "kariérní poradenské studio"
- **Cíl:** Vytvořit modulární multi-agent ekosystém s odbornými rolemi, plnou správou dat a evaluací.
- **Architektura:**
  - Hlavní orchestrátor (Semantic Kernel Process Framework) řídí triáž → specializované agenty (scénářový průvodce, informační vyhledávač, validátor zdrojů, concierge).
  - MCP server proxy nad interními API (např. plánované informační akce, CRM) podle vzoru `agent-openai-python-banking-assistant`.
  - Dlouhodobá paměť: kombinace
    - Vektorový index (Azure AI Search + chunking strategie z `chat-with-your-data`),
    - Relační DB (PostgreSQL) pro preference a historii kroků,
    - Shrnutí profilů pomocí memory cookbook (Claude) pro rychlý kontext.
  - Datový orchestrátor: ingestion pipeline (Azure Data Factory + Functions) s pravidelným crawlingem oficiálních zdrojů, deduplikace, verze.
  - Hodnocení: automatické evaluace (Azure AI Evaluation SDK, Langfuse) + ruční QA na kritických scénářích.
  - Web search: řízený agent s whitelistem oficiálních domén, fallback při chybě dat, logování zdrojů.
- **UX:**
  - Webová aplikace s vizualizací průběhu (Next.js + timeline), modul zpětné vazby, export shrnutí.
  - Personalizované onboarding flows, možnost navázat na minulé návštěvy.
- **Governance a bezpečnost:**
  - App Insights + GA4 + vlastní datamart nákladů (Azure Cost Management API).
  - PII redakce, role-based přístup, Key Vault, CI/CD (GitHub Actions + `azd`).
- **Výhody:** vysoká kvalita odpovědí, auditovatelnost, flexibilita pro nové agenty (např. Slovensko vs. Česko).
- **Limity:** vyšší investice, nutnost dedikovaného týmu pro data + AI ops.

### 4. Enterprise++ – "multikanálový poradenský ekosystém"
- **Cíl:** Využít nej pokročilejší agentní praktiky, multimodální vstupy a více cloudových modelů.
- **Architektura:**
  - Koordinace "týmu agentů" (Google Agent Team pattern) – plánovač, výzkumník, poradce, compliance důstojník, analytik.
  - Multi-model strategie: Azure OpenAI (GPT-4o) + Claude 3.5 Sonnet přes secure proxy (využití prompt caching, paralelní tool calls), eventuálně vlastní Mistral/Mixtral pro offline fallback.
  - Znaloťní graf (Neo4j, podle Hugging Face RAG s knowledge graph) pro relace mezi školami, obory a požadavky + kombinace se vektory a strukturovanou generací.
  - Omnikanál: integrace s Teams/WhatsApp (BotBuilder), web, mobil, volitelně voice (OpenAI voice agent blueprint) a AR/VR chatbot.
  - Real-time observabilita (OpenTelemetry → Data Explorer), automatické guardrail testy (Claude tool evaluation, Azure Content Safety), A/B testy promptů (Prompt Flow). 
  - Automatizované workflow pro aktualizaci dat (crawler → validace → LLM-as-judge QA) a notifikace při zjištění změn (např. termíny DOD).
- **Pokročilé funkce:**
  - Personalizované doporučování oborů s komb. RAG + scoring model (Azure Machine Learning AutoML),
  - Doporučené akce (email follow-up, generování pracovních listů) – integrace s CRM.
  - Toky pro kariérové poradce (live handoff), integrace s calendly/Teams meetingy.
- **Výhody:** prémiový zážitek, vysoká míra automatizace, připravenost na velký růst.
- **Limity:** vysoké CAPEX/OPEX, potřeba robustní bezpečnostní a provozní politiky.

## Porovnání variant (vysoká úroveň)
- **Náročnost:** MVP (nízká) → Enterprise++ (velmi vysoká); doba dodání 1–2 měsíce vs. 9–12 měsíců.
- **Náklady na provoz:** škálují s počtem komponent (jednoduché RAG vs. multi-agent + multi-model).
- **Kvalita odpovědí:** roste s mírou orchestrací, validace a datové kvality; advanced varianty umožňují personalizaci a komplexní scénáře.
- **Rizika:** MVP – kvalita dat a halucinace; Enterprise – komplexita systému, governance, integrace více týmů.

## Doporučený postup
1. Ověřit MVP na omezené skupině (interní kariéroví poradci) a sbírat zpětnou vazbu.
2. Paralelně připravit datovou pipeline (scraping oficiálních zdrojů) a evaluace, aby byl přechod na variantu 2 plynulý.
3. Po úspěšném MVP rozhodnout, zda škálovat na multiagentní řešení (varianta 3) podle kapacity týmu a rozpočtu.
4. Enterprise++ plánovat pouze v případě potřeby omnikanálového rozšíření a výrazného růstu uživatelů.
