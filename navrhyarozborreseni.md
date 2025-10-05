# Návrhy a rozbor řešení chatbota „Chci na vejšku"

## Koncepční checklist

1. **Uchování kontextu** – navrhnout long‑term paměť (kolekce kvízů, preferencí, předchozích dotazů) a způsob jejího vyhledávání.
2. **Dynamické scénáře** – navázat předpřipravené kariérní scénáře a obecné dotazy na datové zdroje a rozhodovací logiku.
3. **Zdroje dat** – stanovit prioritní pořadí (interní DB, RAG/knowledge base, web search) a řešit aktualizaci a citování zdrojů.
4. **Technická infrastruktura** – vybrat platformu (Azure AI Foundry, Google ADK, Copilot Studio nebo open‑source), databáze, vyhledávání, orchestraci agentů a jazykové mutace.
5. **Optimalizace nákladů** – odhad vstupních a výstupních tokenů, výdaje na paměť, provoz databází a dlouhodobě udržitelný model (např. prompt caching).
6. **Analytika a strážci** – implementovat sběr metrik (míra konverze, užitečnost odpovědí) a guardraily (omezení dotazů mimo zadanou doménu).
7. **Vývoj a testování** – zvolit metodiku (MVP → rozšíření), odhadnout kapacitu týmu a rozpočet.

---

## Shrnutí vstupních požadavků

Projekt požaduje chatbota, který pomůže středoškolákům vybrat univerzitu a obor. Chatbot má podporovat dlouhodobou paměť, dynamické kariérové scénáře, více jazyků (CZ/SK), prioritizovanou citaci zdrojů a analytiku.

---

## Datové zdroje a paměť

### Interní databáze

Projekt má vlastní databázi (školy, obory, přijímací řízení, harmonogramy, předměty, výsledky kvízů). Tato data budou primárním zdrojem. Lze uložit do relační DB (např. PostgreSQL).

### Paměť a dlouhodobé znalosti

Pro cross‑session paměť se využijí nástroje poskytované platformou:

- **Azure AI Foundry** – konverzace jsou organizovány do trvalých Threadů; zprávy se ukládají do úložiště a asistent automaticky spravuje kontextový rozsah (odřízne staré zprávy).
- **Google ADK** – rozlišuje krátkodobou paměť (Session/State) a dlouhodobou znalost (MemoryService). BaseMemoryService umožňuje přidat konverzaci do knowledge store a vyhledávat v ní.
- **Copilot Studio (Microsoft)** – orientováno na koncové uživatele; „Memory“ funkce zaznamenává interakce a vytváří osobní profil. Uživatel může zapnout/vypnout ukládání paměti.
- **Open‑source** – frameworky jako LangChain nebo smolagents umožňují vlastní implementaci paměti s vektorem, knowledge graphy apod. Hugging Face cookbook popisuje použití multi‑agentních workflow.

---

## Možné architektury chatbota

### 1. MVP – jednoduchý RAG chat pro statické dotazy

**Přístup**: Data z interní DB jsou exportována do indexu (např. Azure AI Search). Chatbot používá jeden LLM (např. GPT‑3.5 nebo menší GPT‑4 model) a retrieval‑augmented generation (RAG).

**Funkční požadavky**:
- **Dynamické scénáře**: Nepodporované; bot odpovídá na jednoduché dotazy.
- **Dlouhodobá paměť**: Ne.
- **Jazyky**: Použití menšího LLM s multi‑lingual support; detekce jazyka a generování v CZ/SK.
- **Analytika**: Pomocí GA4 logovat spuštění a ukončení konverzace.
- **Guardraily**: Omezit počet tokenů v dotazu; odmítnout dotazy mimo scope.

**Tokenový odhad**:
- Jedna konverzace = 10 dotazů (100 tokenů na dotaz, 200 tokenů odpověď) → 3 000 vstupních tokenů a 6 000 výstupních tokenů měsíčně na uživatele. Cena při ceníku OpenAI (GPT‑4).
- Náklady na indexování a hosting vyhledávače (Azure AI Search Basic, ~50 USD/m).
- Vývoj: 1–2 vývojáři × 2 měsíce (cca 320 h).
- Odhad vývoje ≈ 20 000 USD.

**Výhody**: Rychlá implementace, nízké náklady, možnost testovat se studenty.  
**Nevýhody**: Bez paměti a bez kariérních scénářů; omezená personalizace.

---

### 2. Rozšířený RAG chatbot s pamětí a předdefinovanými scénáři

**Přístup**: Postavit nad MVP orchestraci scénářů a dlouhodobou paměť. V Azure AI Foundry by se pro každého uživatele vytvořil persistentní thread; LLM by se spouštěl v režimu Assistant.  
Paměť obsahuje i výsledky kvízů a předchozí volby. Pro vyhledávání slouží RAG index; pokud se dotaz netýká scénáře, agent použije vyhledávání.
Implementuje se prompt caching pro opakované části promptu, čímž se sníží spotřeba tokenů.

**Funkční požadavky**:
- **Dynamické scénáře**: LLM se vyvolá s instrukcemi a využívá „decision tree“ uložený v DB.
- **Dlouhodobá paměť**: Persistentní thread (Azure) nebo VertexAiMemoryBankService (Google) pro vyhledávání v minulých konverzacích.
- **Web‑search**: Implementováno jako externí nástroj, využívá oficiální weby; agent volá search jen pokud DB nemá informace.
- **Jazyky**: LLM detekuje jazyk a odpovídá v CZ/SK; úvodní zpráva se zvolí podle parametru.
- **Analytika a zpětná vazba**: GA4 měří, kolik studentů dokončí scénář; po ukončení se zobrazí dotazník.

**Tokenový odhad**:
- Delší konverzace (12 otázek, 150 tokenů dotaz, 300 tokenů odpověď). Při třech konverzacích/měsíc ≈ **0,115 USD/měsíc** na uživatele (1 000 uživatelů ≈ 115 USD/m).
- Paměťové vyhledávání generuje dalších 500–1 000 tokenů vstupu na konverzaci (0,0006–0,0013 USD).
- Fixní náklady: Azure AI Search S1 + Cosmos DB s replikami (~200 USD/m), logování, funkce.
- Vývoj: 3–4 vývojáři × 4 měsíce; odhad nákladů 60 000 USD.

**Výhody**: Personalizace, scénáře od kariérních poradců, úspora tokenů díky prompt cachingu, pro uživatele plynulý dialog.  
**Nevýhody**: Vyšší složitost a provozní náklady; nutnost navrhnout strukturu scénářů a upravovat podle zpětné vazby.

---

### 3. Multi‑agentní systém s nástroji a RAG

**Přístup**: Nasadit orchestraci více agentů – jeden agent vede rozhovor (Dialog Agent), jiný vyhledává v DB a RAG indexu (Retrieval Agent), další provádí web‑search, a specializovaný agent řeší scénáře.

**Funkční požadavky**:
- **Dynamické scénáře**: Scénářový agent reaguje na uživatelovy preference a rozhoduje, který krok (otázka) nastane.
- **Dlouhodobá paměť**: Vertex AI Memory Bank (Google) nebo kombinace vlastního vektorového úložiště + datového grafu.
- **Web‑search**: Realizováno jako agent s přísnými guardraily (vyhledává pouze na oficiálních webech).
- **Knowledge graph**: Pro vztahy (škola–fakulta–program–obor) se vytvoří graf (např. Neo4j), který umožní vícekrokové dotazy a doporučování (využívá se RAG s grafem).
- **Tokenový odhad**: Multi‑agentní workflow přidává overhead (cca +30 % tokenů na orchestraci). Pro 1 000 uživatelů může měsíční účet za GPT‑5 dosáhnout ~150–200 USD.
- **Infrastruktura**: Vyžaduje orchestrátor (např. Google Vertex AI Agents Engine, Azure Functions nebo vlastní orchestraci), memory bank, vektorový index, grafovou DB a funkční konektory.
- **Vývoj**: 4–5 vývojářů × 6 měsíců; odhad 100 000 USD.

**Výhody**: Vysoká flexibilita, modulární rozšíření, možnost snadno přidávat nové scénáře nebo agenty, výkonné vyhledávání v grafu a RAG.  
**Nevýhody**: Vyšší složitost implementace, nutnost orchestrace a monitoringu, vyšší tokenová spotřeba.

---

### 4. Profesionalizovaný systém se znalostním grafem, strukturovanou generací a MCP servery

**Přístup**: Vrcholný návrh kombinuje multi‑agentní systém s knowledge graph a RAG, navíc obsahuje cross‑linguální překlad, generování strukturovaných odpovědí a externí micro‑services.

**Funkční požadavky**:
- **Dynamické scénáře**: Řídí orchestrátor, který spouští podsystémy podle stavového modelu.
- **Dlouhodobá paměť**: Kombinace Vertex AI Memory Bank + vlastního knowledge graphu; staré konverzace se extrahují a přidávají do grafu (caching).
- **RAG + graf**: Dotazy se nejprve vyhledají ve vektorovém indexu; pokud je dotaz komplexní, orchestrátor analyzuje graf (multi‑hop reasoning).
- **Web‑search**: Spouští se specializovaný agent s guardrails – prioritu mají oficiální zdroje (msmt.gov.cz, regvssp, vysokeskoly.cz).
- **Structure and citations**: Model generuje strukturovaný JSON se zdroji; zobrazuje uživateli i citované odstavce.
- **Tokenový odhad**: Každá konverzace ~20 interakcí, navíc overhead na orchestraci; měsíčně cca 0,2 USD na uživatele.
- **Infrastruktura**: Při 1 000 uživatelích vyžaduje robustní orchestraci (Kubernetes, Ray), vektorové servery (Weaviate/Vespa), graf (Neo4j) a funkční mikro‑služby.
- **Vývoj**: 5–6 vývojářů × 8 měsíců; odhad 150 000–200 000 USD.

**Výhody**: Nejvyšší kvalita odpovědí, škálovatelnost, strukturované odpovědi a auditelnost.  
**Nevýhody**: Nejnákladnější a nejsložitější implementace; vyžaduje experty na RAG, grafy a orchestraci.

---

## Porovnání frameworků a platforem

| Platforma/framework | Podpora paměti | Scénáře a nástroje | Kompatibilita (CZ/SK) | Komplexita implementace | Nákladovost |
|---|---|---|---|---|---|
| **Azure AI Foundry** | Perzistentní Threads automaticky spravují kontext, ukládání probíhá v Cosmos DB | Složitější scénáře lze implementovat přes orchestrace a custom tools | Dobrá CZ/SK podpora, možnost vlastních modelů | Střední | $ |
| **Google ADK + Vertex AI** | Rozlišení Session/State (krátkodobé) a MemoryService (dlouhodobé); VertexAiMemoryBankService poskytuje persistenci a sémantické vyhledávání | Silná podpora nástrojů, orchestrace více agentů | Dobrá CZ/SK podpora, nutné ověřit model | Vyšší | $$ |
| **Copilot Studio (Power Virtual Agents)** | Nabízí „Memory“ pro personalizované rady; uživatel může paměť vypnout nebo smazat | Scénáře přes Power Automate flows, omezení v nástrojích | Omezená CZ/SK podpora, hlavně pro firmy | Nízká | $ |
| **Open‑source (LangChain, smolagents)** | Nutno implementovat vlastní paměť (např. vektorová DB + knowledge graph) | Plná kontrola – možnost multi‑agentního systému, knowledge graph | Závisí na modelu a lokalizaci | Vysoká | $$ |

---

## Přibližné náklady při 1 000 uživatelích

| Odhadovaný model           | Input tokeny/user/měsíc | Output tokeny/user/měsíc | Cena na uživatele/měsíc (GPT‑5) | Celkem 1 000 uživatelů | Přídavné náklady (paměť, DB, orchestrace) |
|---------------------------|------------------------|--------------------------|-----------------------------------|------------------------|--------------------------------------------|
| **MVP (10 interakcí)**    | ~3 000                 | ~6 000                   | ~0,063 USD                        | ~63 USD                | Azure AI Search (≈50 USD/m), serverless hosting (~50 USD/m) |
| **Rozšířený RAG**         | ~5 400                 | ~10 800                  | ~0,115 USD                        | ~115 USD               | Cosmos DB + AI Search (~200 USD/m), web‑search volání (~50 USD/m) |
| **Multi‑agentní systém**  | ~7 000                 | ~14 000                  | ~0,17 USD                         | ~170 USD               | vektorová DB (Weaviate Cluster ~100 USD/m), Neo4j (~150 USD/m), orchestrátor (Kubernetes ~200 USD/m) |
| **Profesionalizovaný systém** | ~10 000             | ~20 000                  | ~0,25 USD                         | ~250 USD               | knowledge graph, streaming, micro‑services, monitoring (celkem 500–700 USD/m) |

Fixní náklady zahrnují vývojové prostředí, experimentální pipeline (CI/CD), logování, GA4 a doménu. Variabilní náklady jsou zejména tokeny LLM, vyhledávání, úložiště a přenos dat.

---

## Doporučení

- **Pilotní fáze** – Postavit **MVP** s jednoduchým RAG, abyste rychle otestovali kvalitu odpovědí a poptávku. Připravte datový model pro univerzity, fakulty a obory, a exportujte do indexu.
- **Iterativní rozšíření** – Po pilotu přejít na **rozšířený RAG** se scénáři a persistentní pamětí. Uložené výsledky kvízů a preference pomohou personalizovat doporučení.
- **Pokročilá verze** – Pokud bude poptávka a rozpočet, zvažte **multi‑agentní systém**, který umožní přidávat další rozhodovací scénáře, web‑search a knowledge graph.
- **Výběr platformy** – Pro firemní prostředí a integraci s Azure doporučuji **Azure AI Foundry**: umožňuje perzistentní vlákna.

---

### Reference

- [Azure AI Foundry – OpenAI Assistants](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/assistants)
- [Baseline Azure AI Foundry Chat](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/architecture/baseline-azure-ai-foundry-chat)
- [Azure AI Foundry Agent – Memory](https://learn.microsoft.com/en-us/answers/questions/2283126/azure-ai-foundry-agent-how-to-add-memory)
- [Google ADK – Sessions and Memory](https://google.github.io/adk-docs/sessions/memory/)
- [Copilot Studio – Memory feature](https://www.microsoft.com/en-us/microsoft-copilot/for-individuals/do-more-with-ai/general-ai/ai-that-doesnt-just-remember-it-gets-you)
- [Hugging Face – Agents Cookbook](https://huggingface.co/learn/cookbook/en/agents)
- [Hugging Face – Structured Generation](https://huggingface.co/learn/cookbook/en/structured_generation)
- [Hugging Face – RAG with Knowledge Graphs Neo4j](https://huggingface.co/learn/cookbook/en/rag_with_knowledge_graphs_neo4j).