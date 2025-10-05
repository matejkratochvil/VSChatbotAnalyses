#  Návrhy a rozbor řešení chatbota „Chci na vejšku“

## [Koncepční checklist]()

1.        **Uchování kontextu** – navrhnout long‑term paměť (kolekce kvízů, preferencí, předchozích dotazů) a způsob jejího vyhledávání.

2.        **Dynamické scénáře** – navázat předpřipravené kariérní scénáře a obecné dotazy na datové zdroje a rozhodovací logiku.

3.        **Zdroje dat** – stanovit prioritní pořadí (interní DB, RAG/knowledge base, web search) a řešit aktualizaci a citování zdrojů.

4.        **Technická infrastruktura** – vybrat platformu (Azure AI Foundry, Google ADK, Copilot Studio nebo open‑source), databáze, vyhledávání, orchestraci agentů a jazykové mutace.

5.        **Optimalizace nákladů** – odhad vstupních a výstupních tokenů, výdaje na paměť, provoz databází a dlouhodobě udržitelný model (např. prompt caching).

6.        **Analytika a strážci** – implementovat sběr metrik (míra konverze, užitečnost odpovědí) a guardraily (omezení dotazů mimo zadanou doménu).

7.        **Vývoj a testování** – zvolit metodiku (MVP → rozšíření), odhadnout kapacitu týmu a rozpočet.

## [Shrnutí vstupních požadavků]()

Projekt požaduje chatbota, který pomůže středoškolákům vybrat univerzitu a obor. Chatbot má podporovat dlouhodobou paměť, dynamické kariérové scénáře, více jazyků (CZ/SK), prioritizovat interní databázi, případně využít web‑search pro chybějící informace a zůstat nákladově efektivní. Vysoký důraz je kladen na rychlé reakce a kvalitu odpovědí.

## [Datové zdroje a paměť]()

### [Interní databáze]()

Projekt má vlastní databázi (školy, obory, přijímací řízení, harmonogramy, předměty, výsledky kvízů). Tato data budou primárním zdrojem. Lze uložit do relační DB (např. PostgreSQL/Azure SQL) nebo do schématu v datalake (Iceberg) a zpřístupnit vyhledávacím enginem (Azure AI Search / Elastic / Vespa). Repliky nebo snapshoty lze přenášet do vyhledávacího indexu pro rychlý dotaz.

### [Paměť a dlouhodobé znalosti]()

Pro cross‑session paměť se využijí nástroje poskytované platformou:

·      **Azure AI Foundry** – konverzace jsou organizovány do trvalých Threadů; zprávy se ukládají do úložiště a asistent automaticky spravuje kontextový rozsah (odřízne staré zprávy, aby se vešel do kontextu modelu)[\[1\]](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/assistants#:~:text=tune%20their%20personality%20and%20capabilities,in%20the%20Messages%20they%20create). Vývojář pouze připojuje message k existujícímu Threadu; není účtován zvláštní poplatek za používání Threadů[\[2\]](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/assistants#:~:text=tune%20their%20personality%20and%20capabilities,in%20the%20Messages%20they%20create). Pro perzistentní ukládání historie se používá Cosmos DB[\[3\]](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/architecture/baseline-azure-ai-foundry-chat#:~:text=3,tools%2C%20and%20connected%20knowledge%20stores), u webhook integrací je nutné perzistovat thread_id pro každého uživatele[\[4\]](https://learn.microsoft.com/en-us/answers/questions/2283126/azure-ai-foundry-agent-how-to-add-memory#:~:text=In%20Azure%20AI%20Foundry%2C%20context,Thread%20ID%20between%20webhook%20invocations).

·      **Google ADK** – rozlišuje krátkodobou paměť (Session/State) a dlouhodobou znalost (MemoryService). BaseMemoryService umožňuje přidat konverzaci do knowledge store a vyhledávat relevantní úryvky[\[5\]](https://google.github.io/adk-docs/sessions/memory/#:~:text=The%20,Its%20primary%20responsibilities%20are). K dispozici jsou dvě implementace: InMemoryMemoryService (bez perzistence) a VertexAiMemoryBankService (perzistentní, s extrakcí významných informací a sémantickým vyhledáváním)[\[6\]](https://google.github.io/adk-docs/sessions/memory/#:~:text=Choosing%20the%20Right%20Memory%20Service%C2%B6). Při použití Vertex AI je potřeba mít projekt a Agent Engine[\[7\]](https://google.github.io/adk-docs/sessions/memory/#:~:text=Vertex%20AI%20Memory%20Bank%C2%B6).

·      **Copilot Studio (Microsoft)** – orientováno na koncové uživatele; „Memory“ funkce zaznamenává interakce a vytváří osobní profil. Uživatel může zapnout/vypnout ukládání nebo odstranit konkrétní vzpomínku[\[8\]](https://www.microsoft.com/en-us/microsoft-copilot/for-individuals/do-more-with-ai/general-ai/ai-that-doesnt-just-remember-it-gets-you#:~:text=How%20does%20the%20new%20Memory,feature%20in%20Copilot%20work). Pro obchodní implementace je vhodnější Azure AI Foundry; článek slouží jako důkaz, že Copilot podporuje personalizovanou paměť.

·      **Open‑source** – frameworky jako LangChain nebo smolagents umožňují vlastní implementaci paměti s vektorem, knowledge graphy apod. Hugging Face cookbook popisuje použití multi‑agentních systémů s RAG a znalostními grafy[\[9\]](https://huggingface.co/learn/cookbook/en/agents#:~:text=2,refinement%20%26%20Source%20selection), včetně obousměrného vyhledávání a generování strukturovaných odpovědí[\[10\]](https://huggingface.co/learn/cookbook/en/structured_generation#:~:text=RAG%20with%20source%20highlighting%20using,Structured%20generation).

## [Možné architektury chatbota]()

### [1. **MVP – jednoduchý RAG chat pro statické dotazy**]()

**Přístup**: Data z interní DB jsou exportována do indexu (např. Azure AI Search). Chatbot používá jeden LLM (např. GPT‑3.5 nebo menší GPT‑4 model) a retrieval‑augmented generation (RAG). Uživatel položí otázku → systém vyhledá relevantní dokumenty v indexu → vytvoří prompt (dotaz + úryvky) → LLM vygeneruje odpověď a uvede citace. Neimplementuje se paměť; každá relace je nezávislá.

**Funkční požadavky**:

·      **Dynamické scénáře**: Nepodporované; bot odpovídá na jednoduché dotazy.

·      **Dlouhodobá paměť**: Ne.

·      **Jazyky**: Použití menšího LLM s multi‑lingual support; detekce jazyka a generování v CZ/SK.

·      **Analytika**: Pomocí GA4 logovat spuštění a ukončení konverzace.

·      **Guardraily**: Omezit počet tokenů v dotazu; odmítnout dotazy mimo scope.

**Tokenový odhad**:
• Jedna konverzace = 10 dotazů (100 tokenů na dotaz, 200 tokenů odpověď) → 3 000 vstupních tokenů a 6 000 výstupních tokenů měsíčně na uživatele. Cena při ceníku OpenAI (GPT‑5: 1,25 USD/M input, 10 USD/M output) ≈ **0,063 USD/měsíc** na uživatele; pro 1 000 uživatelů asi **63 USD/měsíc**[\[1\]](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/assistants#:~:text=tune%20their%20personality%20and%20capabilities,in%20the%20Messages%20they%20create).
• Náklady na indexování a hosting vyhledávače (Azure AI Search Basic, ~50 USD/m).
• Vývoj: 1–2 vývojáři × 2 měsíce (cca 320 h).
• Odhad vývoje ≈ 20 000 USD.

**Výhody**: Rychlá implementace, nízké náklady, možnost testovat se studenty.
**Nevýhody**: Bez paměti a bez kariérních scénářů; omezená personalizace.

### [2. **Rozšířený RAG chatbot s pamětí a předdefinovanými scénáři**]()

**Přístup**: Postavit nad MVP orchestraci scénářů a dlouhodobou paměť. V Azure AI Foundry by se pro každého uživatele vytvořil persistentní thread; LLM by se spouštěl v režimu Assistant API se specifikovanou max. délkou promptu[\[11\]](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/assistants#:~:text=provides%20a%20solution%20for%20these,messages%20to%20it%20as%20users). Do DB se ukládají předepsané kariérní scénáře (strom rozhodnutí); agent dynamicky klade otázky, ukládá preference do state a na konci navrhne vhodný obor.
Paměť obsahuje i výsledky kvízů a předchozí volby. Pro vyhledávání slouží RAG index; pokud se dotaz netýká scénáře, agent použije vyhledávání.
Implementuje se prompt caching pro opakované části promptu, čímž se sníží spotřeba tokenů (podle dokumentace lze uložit často používané části promptu a přidávat je do konverzace jen referencí).

**Funkční požadavky**:

·      **Dynamické scénáře**: LLM se vyvolá s instrukcemi, jak postupovat, a využívá nástroj „decision tree“ uložený v DB.

·      **Dlouhodobá paměť**: Persistentní thread (Azure) nebo VertexAiMemoryBankService (Google) pro vyhledávání v minulých konverzacích[\[5\]](https://google.github.io/adk-docs/sessions/memory/#:~:text=The%20,Its%20primary%20responsibilities%20are).

·      **Web‑search**: Implementováno jako externí nástroj, využívá oficiální weby; agent volá search jen pokud DB nemá informace.

·      **Jazyky**: LLM detekuje jazyk a odpovídá v CZ/SK; úvodní zpráva se zvolí podle parametru.

·      **Analytika a zpětná vazba**: GA4 měří, kolik studentů dokončí scénář; po ukončení se zobrazí dotazník.

**Tokenový odhad**:
• Díky paměti a scénářům je konverzace delší (12 otázek, 150 tokenů dotaz, 300 tokenů odpověď). Při třech konverzacích/měsíc ≈ **0,115 USD/měsíc** na uživatele (1 000 uživatelů → ~115 USD/měsíc)[\[1\]](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/assistants#:~:text=tune%20their%20personality%20and%20capabilities,in%20the%20Messages%20they%20create).
• Paměťové vyhledávání generuje dalších 500–1 000 tokenů vstupu na konverzaci (0,0006–0,0013 USD).
• Fixní náklady: Azure AI Search S1 + Cosmos DB s replikami (~200 USD/m), logování, funkce.
• Vývoj: 3–4 vývojáři × 4 měsíce; odhad nákladů 60 000 USD.

**Výhody**: Personalizace, scénáře od kariérních poradců, úspora tokenů díky prompt cachingu, pro uživatele plynulý dialog.
**Nevýhody**: Vyšší složitost a provozní náklady; nutnost navrhnout strukturu scénářů a upravovat podle zpětné vazby.

### [3. **Multi‑agentní systém s nástroji a RAG**]()

**Přístup**: Nasadit orchestraci více agentů – jeden agent vede rozhovor (Dialog Agent), jiný vyhledává v DB a RAG indexu (Retrieval Agent), další provádí web‑search, a specializovaný „Scénářový Agent“ řídí kariérní rozhodovací strom. Google ADK ukazuje, jak agenti mohou sdílet session_state a používat paměťové služby[\[12\]](https://google.github.io/adk-docs/sessions/memory/#:~:text=The%20,Its%20primary%20responsibilities%20are). Při dotazu agent může volat funkce (např. load_memory nebo search_memory) a podle kontextu spouštět další agenty. LangChain a smolagents popisují architekturu, kde každý agent optimalizuje určité dílčí úkoly a používá RAG a knowledge graph[\[9\]](https://huggingface.co/learn/cookbook/en/agents#:~:text=2,refinement%20%26%20Source%20selection).

**Funkční požadavky**:

·      **Dynamické scénáře**: Scénářový agent reaguje na uživatelovy preference a rozhoduje, který krok (otázka) nastane.

·      **Dlouhodobá paměť**: Použití Vertex AI Memory Bank (Google) nebo kombinace vlastního vektorového úložiště + datového grafu. U ADK lze kombinovat více paměťových služeb a vyhledávat paralelně[\[13\]](https://google.github.io/adk-docs/sessions/memory/#:~:text=information%20to%20formulate%20its%20final,answer%20to%20the%20user).

·      **Web‑search**: Realizováno jako agent s přísnými guardraily (vyhledává pouze na oficiálních webech).

·      **Knowledge graph**: Pro vztahy (škola–fakulta–program–obor) se vytvoří graf (např. Neo4j), který umožní vícekrokové dotazy a doporučování (využívá se RAG s grafem[\[14\]](https://huggingface.co/learn/cookbook/en/rag_with_knowledge_graphs_neo4j#:~:text=Enhancing%20RAG%20Reasoning%20with%20Knowledge,Graphs)).

·      **Tokenový odhad**: Multi‑agentní workflow přidává overhead (cca +30 % tokenů na orchestraci). Pro 1 000 uživatelů může měsíční účet za GPT‑5 dosáhnout ~150–200 USD.

·      **Infrastruktura**: Vyžaduje orchestrátor (např. Google Vertex AI Agents Engine, Azure Functions nebo vlastní orchestraci), memory bank, vektorový index, grafovou DB a funkční konektory.

·      **Vývoj**: 4–5 vývojářů × 6 měsíců; odhad 100 000 USD.

**Výhody**: Vysoká flexibilita, modulární rozšíření, možnost snadno přidávat nové scénáře nebo agenty, výkonné vyhledávání v grafu a RAG.
**Nevýhody**: Vyšší složitost implementace, nutnost orchestrace a monitoringu, vyšší tokenová spotřeba.

### [4. **Profesionalizovaný systém se znalostním grafem, strukturovanou generací a MCP servery**]()

**Přístup**: Vrcholný návrh kombinuje multi‑agentní systém s knowledge graph a RAG, navíc obsahuje cross‑linguální překlad, generování strukturovaných odpovědí a externí micro‑services (MCP). Podle smolagents se do RAG integruje knowledge graph a iterativní dotazování, aby agent generoval přesnější odpovědi[\[9\]](https://huggingface.co/learn/cookbook/en/agents#:~:text=2,refinement%20%26%20Source%20selection). Strukturovaná generace vyžaduje, aby model vrátil JSON s odpovědí a citacemi, což usnadňuje audit a redukci halucinací[\[10\]](https://huggingface.co/learn/cookbook/en/structured_generation#:~:text=RAG%20with%20source%20highlighting%20using,Structured%20generation). K překladům lze použít dedikovaný překladový agent. Pomocí MCP serverů lze volat interní API (dotazy do DB, vyhledávání).

**Funkční požadavky**:

·      **Dynamické scénáře**: Řídí orchestrátor, který spouští podsystémy podle stavového modelu.

·      **Dlouhodobá paměť**: Kombinace Vertex AI Memory Bank + vlastního knowledge graphu; staré konverzace se extrahují a přidávají do grafu (caching)[\[15\]](https://google.github.io/adk-docs/sessions/memory/#:~:text=Vertex%20AI%20Memory%20Bank%C2%B6).

·      **RAG + graf**: Dotazy se nejprve vyhledají ve vektorovém indexu; pokud je dotaz komplexní, orchestrátor analyzuje graf (multi‑hop reasoning)[\[14\]](https://huggingface.co/learn/cookbook/en/rag_with_knowledge_graphs_neo4j#:~:text=Enhancing%20RAG%20Reasoning%20with%20Knowledge,Graphs).

·      **Web‑search**: Spouští se specializovaný agent s guardrails – prioritu mají oficiální zdroje (msmt.gov.cz, regvssp, vysokeskoly.cz).

·      **Structure and citations**: Model generuje strukturovaný JSON se zdroji; zobrazuje uživateli i citované odstavce.

·      **Tokenový odhad**: Každá konverzace ~20 interakcí, navíc overhead na orchestraci; měsíčně cca 0,2 USD na uživatele.

·      **Infrastruktura**: Při 1 000 uživatelích vyžaduje robustní orchestraci (Kubernetes, Ray), vektorové servery (Weaviate/Vespa), graf (Neo4j) a funkční mikro‑služby.

·      **Vývoj**: 5–6 vývojářů × 8 měsíců; odhad 150 000–200 000 USD.

**Výhody**: Nejvyšší kvalita odpovědí, škálovatelnost, strukturované odpovědi a auditelnost.
**Nevýhody**: Nejnákladnější a nejsložitější implementace; vyžaduje experty na RAG, grafy a orchestraci.

## [Porovnání frameworků a platforem]()

|   Platforma/framework  |    Podpora paměti  |    Scénáře a nástroje  |    Kompatibilita (CZ/SK)  |    Komplexita implementace  |    Nákladovost  |
|---|---|---|---|---|---|
|   **Azure AI Foundry**  |    Perzistentní Threads automaticky spravují kontext, ukládání probíhá v Cosmos DB[\[3\]](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/architecture/baseline-azure-ai-foundry-chat#:~:text=3,tools%2C%20and%20connected%20knowledge%20stores); nutno perzistovat thread_id při webhook integraci[\[4\]](https://learn.microsoft.com/en-us/answers/questions/2283126/azure-ai-foundry-agent-how-to-add-memory#:~:text=In%20Azure%20AI%20Foundry%2C%20context,Thread%20ID%20between%20webhook%20invocations).  |    Podporuje tool‑calling a paralelní nástroje (code interpreter, file search), integrační funkce; Assistant API umožňuje persistentní vlákna a konfiguraci max_prompt_tokens pro kontrolu context window[\[1\]](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/assistants#:~:text=tune%20their%20personality%20and%20capabilities,in%20the%20Messages%20they%20create).  |    Modely GPT‑4/GPT‑5 mají dobrou podporu češtiny a slovenštiny; nutno upravit prompt; úvodní zprávu lze lokalizovat.  |    Střední – nutná znalost Azure, Thread management a bezpečnost (RBAC, privátní endpoints).  |    Spotřeba tokenů je hlavní variabilní náklad; fixně se platí za AI Search, Cosmos DB a hosting (~200–300 USD/m při 1 000 uživatelích).  |   
|   **Google ADK + Vertex AI**  |    Rozlišení Session/State (krátkodobé) a MemoryService (dlouhodobé); VertexAiMemoryBankService poskytuje persistenci a sémantické vyhledávání[\[6\]](https://google.github.io/adk-docs/sessions/memory/#:~:text=Choosing%20the%20Right%20Memory%20Service%C2%B6).  |    ADK umožňuje definovat agenty, nástroje a multi‑agentní orchestrace; paměť se přidává pomocí memory_service a lze kombinovat více služeb[\[13\]](https://google.github.io/adk-docs/sessions/memory/#:~:text=information%20to%20formulate%20its%20final,answer%20to%20the%20user).  |    Lze využít Gemini‑Pro/Flash, které podporují češtinu; je třeba implementovat lokalizační logiku.  |    Střední až vyšší; vyžaduje vytvoření Agent Engine, autentizaci a správy GCP; multi‑agentní design.  |    Cena Vertex AI Memory Bank a vyhledávání navíc; tokenová cena modelu (Gemini Flash ~0,35 USD/M input, 0,7 USD/M output) je nižší než GPT‑5.  |   
|   **Copilot Studio (Power Virtual Agents)**  |    Nabízí „Memory“ pro personalizované rady; uživatel může paměť vypnout nebo smazat[\[8\]](https://www.microsoft.com/en-us/microsoft-copilot/for-individuals/do-more-with-ai/general-ai/ai-that-doesnt-just-remember-it-gets-you#:~:text=How%20does%20the%20new%20Memory,feature%20in%20Copilot%20work).  |    Zaměřeno na jednoduché konverzační agenty; scénáře se staví vizuálně; integrace s databází přes Power Automate.  |    Podpora CZ/SK závisí na modelu; generativní odpovědi v češtině jsou omezené, chybí pokročilá RAG.  |    Nízká – low‑code prostředí, rychlý prototyp.  |    Licencování (Microsoft 365 Copilot) je drahé na uživatele; nepodporuje custom modely, takže náklady mohou být vyšší než provoz vlastního back‑endu.  |   
|   **Open‑source (LangChain, smolagents)**  |    Nutno implementovat vlastní paměť (např. vektorová DB + knowledge graph).  |    Plná kontrola – možnost multi‑agentního systému, knowledge graph, RAG; smolagents popisují iterativní dotazování a multi‑hop reasoning[\[9\]](https://huggingface.co/learn/cookbook/en/agents#:~:text=2,refinement%20%26%20Source%20selection).  |    LLM musí být zvolen a nasazen; pro CZ/SK se doporučují modely GPT‑4/5 nebo open‑source Llama v kombinaci s RAG.  |    Vysoká – vyžaduje vývoj infrastruktury, orchestraci a bezpečnost.  |    Flexibilní; nejvyšší náklady tvoří spotřeba tokenů a správa infrastrukturních komponent.  |   
## [Přibližné náklady při 1 000 uživatelích]()

|   Odhadovaný model  |    Input tokeny/user/měsíc  |    Output tokeny/user/měsíc  |    Cena na uživatele/měsíc (GPT‑5)  |    Celkem 1 000 uživatelů  |    Přídavné náklady (paměť, DB, search)  |
|---|---|---|---|---|---|
|   **MVP (10 interakcí)**  |    ~3 000  |    ~6 000  |    ~0,063 USD  |    ~63 USD  |    Azure AI Search (≈50 USD/m), serverless hosting (~50 USD/m)  |   
|   **Rozšířený RAG**  |    ~5 400  |    ~10 800  |    ~0,115 USD  |    ~115 USD  |    Cosmos DB + AI Search (~200 USD/m), web‑search volání (~50 USD/m)  |   
|   **Multi‑agentní systém**  |    ~7 000  |    ~14 000  |    ~0,17 USD  |    ~170 USD  |    vektorová DB (Weaviate Cluster ~100 USD/m), Neo4j (~150 USD/m), orchestrátor (Kubernetes ~200 USD/m)  |   
|   **Profesionalizovaný systém**  |    ~10 000  |    ~20 000  |    ~0,25 USD  |    ~250 USD  |    knowledge graph, streaming, micro‑services, monitoring (celkem 500–700 USD/m)  |   
Fixní náklady zahrnují vývojové prostředí, experimentální pipeline (CI/CD), logování, GA4 a doménu. Variabilní náklady jsou zejména tokeny LLM, vyhledávání, úložiště a přenos dat.

## [Doporučení]()

·      **Pilotní fáze** – Postavit **MVP** s jednoduchým RAG, abyste rychle otestovali kvalitu odpovědí a poptávku. Připravte datový model pro univerzity, fakulty a obory, a exportujte jej do vyhledávacího indexu. Zavést guardrail pro standardní dotazy, aby uživatelé nepoužívali bota místo ChatGPT.

·      **Iterativní rozšíření** – Po pilotu přejít na **rozšířený RAG** se scénáři a persistentní pamětí. Uložené výsledky kvízů a preference pomohou personalizovat doporučení. Zároveň implementujte GA4 pro sledování, kolik studentů dokončí scénář.

·      **Pokročilá verze** – Pokud bude poptávka a rozpočet, zvažte **multi‑agentní systém**, který umožní přidávat další rozhodovací scénáře, web‑search a knowledge graph. Tato verze nabízí nejlepší kvalitu doporučení a škálovatelnost, ale je výrazně náročnější na vývoj a provoz.

·      **Výběr platformy** – Pro firemní prostředí a integraci s Azure doporučuji **Azure AI Foundry**: umožňuje perzistentní vlákna[\[1\]](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/assistants#:~:text=tune%20their%20personality%20and%20capabilities,in%20the%20Messages%20they%20create), RAG s Azure AI Search, monitorování a bezpečnost. Pro multi‑agentní experimenty lze zvažovat **Google ADK** s Vertex AI, který poskytuje modulární paměť a agentní orchestraci[\[12\]](https://google.github.io/adk-docs/sessions/memory/#:~:text=The%20,Its%20primary%20responsibilities%20are). Low‑code **Copilot Studio** lze využít jen pro prototyp, protože neumožňuje pokročilý RAG a není nákladově efektivní pro velký provoz.

---

 [\[1\]](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/assistants#:~:text=tune%20their%20personality%20and%20capabilities,in%20the%20Messages%20they%20create) [\[2\]](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/assistants#:~:text=tune%20their%20personality%20and%20capabilities,in%20the%20Messages%20they%20create) [\[11\]](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/assistants#:~:text=provides%20a%20solution%20for%20these,messages%20to%20it%20as%20users) Azure OpenAI in Azure AI Foundry Models Assistants API concepts - Azure OpenAI | Microsoft Learn

[https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/assistants](https://learn.microsoft.com/en-us/azure/ai-foundry/openai/concepts/assistants)

[\[3\]](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/architecture/baseline-azure-ai-foundry-chat#:~:text=3,tools%2C%20and%20connected%20knowledge%20stores) Baseline Azure AI Foundry Chat Reference Architecture - Azure Architecture Center | Microsoft Learn

[https://learn.microsoft.com/en-us/azure/architecture/ai-ml/architecture/baseline-azure-ai-foundry-chat](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/architecture/baseline-azure-ai-foundry-chat)

[\[4\]](https://learn.microsoft.com/en-us/answers/questions/2283126/azure-ai-foundry-agent-how-to-add-memory#:~:text=In%20Azure%20AI%20Foundry%2C%20context,Thread%20ID%20between%20webhook%20invocations) Azure AI Foundry Agent: How to Add Memory - Microsoft Q&A

[https://learn.microsoft.com/en-us/answers/questions/2283126/azure-ai-foundry-agent-how-to-add-memory](https://learn.microsoft.com/en-us/answers/questions/2283126/azure-ai-foundry-agent-how-to-add-memory)

[\[5\]](https://google.github.io/adk-docs/sessions/memory/#:~:text=The%20,Its%20primary%20responsibilities%20are) [\[6\]](https://google.github.io/adk-docs/sessions/memory/#:~:text=Choosing%20the%20Right%20Memory%20Service%C2%B6) [\[7\]](https://google.github.io/adk-docs/sessions/memory/#:~:text=Vertex%20AI%20Memory%20Bank%C2%B6) [\[12\]](https://google.github.io/adk-docs/sessions/memory/#:~:text=The%20,Its%20primary%20responsibilities%20are) [\[13\]](https://google.github.io/adk-docs/sessions/memory/#:~:text=information%20to%20formulate%20its%20final,answer%20to%20the%20user) [\[15\]](https://google.github.io/adk-docs/sessions/memory/#:~:text=Vertex%20AI%20Memory%20Bank%C2%B6) Memory - Agent Development Kit

[https://google.github.io/adk-docs/sessions/memory/](https://google.github.io/adk-docs/sessions/memory/)

[\[8\]](https://www.microsoft.com/en-us/microsoft-copilot/for-individuals/do-more-with-ai/general-ai/ai-that-doesnt-just-remember-it-gets-you#:~:text=How%20does%20the%20new%20Memory,feature%20in%20Copilot%20work) AI That Doesn’t Just Remember, It Gets You | Microsoft Copilot

[https://www.microsoft.com/en-us/microsoft-copilot/for-individuals/do-more-with-ai/general-ai/ai-that-doesnt-just-remember-it-gets-you](https://www.microsoft.com/en-us/microsoft-copilot/for-individuals/do-more-with-ai/general-ai/ai-that-doesnt-just-remember-it-gets-you)

[\[9\]](https://huggingface.co/learn/cookbook/en/agents#:~:text=2,refinement%20%26%20Source%20selection) Build an agent with tool-calling superpowers using smolagents - Hugging Face Open-Source AI Cookbook

[https://huggingface.co/learn/cookbook/en/agents](https://huggingface.co/learn/cookbook/en/agents)

[\[10\]](https://huggingface.co/learn/cookbook/en/structured_generation#:~:text=RAG%20with%20source%20highlighting%20using,Structured%20generation) RAG with source highlighting using Structured generation - Hugging Face Open-Source AI Cookbook

[https://huggingface.co/learn/cookbook/en/structured_generation](https://huggingface.co/learn/cookbook/en/structured_generation)

[\[14\]](https://huggingface.co/learn/cookbook/en/rag_with_knowledge_graphs_neo4j#:~:text=Enhancing%20RAG%20Reasoning%20with%20Knowledge,Graphs) Enhancing RAG Reasoning with Knowledge Graphs - Hugging Face Open-Source AI Cookbook

[https://huggingface.co/learn/cookbook/en/rag_with_knowledge_graphs_neo4j](https://huggingface.co/learn/cookbook/en/rag_with_knowledge_graphs_neo4j)

  