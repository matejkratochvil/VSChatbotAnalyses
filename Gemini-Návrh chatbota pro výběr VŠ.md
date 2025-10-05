

# **Strategický a Technický Návrh: Vývoj AI Chatbota pro Kariérní Poradenství Studentů**

## **Shrnutí pro Vedení**

Tento dokument předkládá komplexní strategický a technický plán pro vývoj, nasazení a provoz pokročilého AI chatbota, jehož cílem je poskytovat studentům personalizované poradenství při výběru vysoké školy a studijního oboru. Projekt si klade za cíl vytvořit špičkový nástroj, který bude integrován s proprietární databází vzdělávacích institucí, bude si pamatovat interakce s uživateli pro zajištění kontinuity a bude schopen dynamicky provázet studenty poradenskými scénáři.

### **Klíčová Doporučení**

Po důkladné analýze funkčních požadavků, časového rámce a strategických cílů je doporučeným postupem realizace **Návrhu B: Škálovatelný Profesionální Systém**. Tato architektura představuje optimální rovnováhu mezi počáteční investicí, dlouhodobou udržitelností a flexibilitou pro budoucí rozšiřování.  
Jako základní vývojový framework je doporučen **Microsoft Semantic Kernel**. Jeho strukturovaný, softwarově-inženýrský přístup a nativní, prvotřídní integrace s ekosystémem Microsoft Azure z něj činí strategicky nejvhodnější volbu, která minimalizuje technický dluh a zajišťuje robustnost a spravovatelnost řešení v produkčním prostředí.  
Odhadované celkové náklady na vlastnictví (TCO) pro doporučený Návrh B v tříletém horizontu jsou podrobně rozpracovány v závěrečné části tohoto dokumentu.

### **Hlavní Zjištění**

Analýza odhalila zásadní kompromisy mezi rychlostí uvedení na trh, provozními náklady a dlouhodobou flexibilitou. Zatímco zjednodušený přístup (Minimum Viable Product \- MVP) umožňuje rychlé ověření konceptu, jeho architektura nese inherentní omezení v oblasti škálovatelnosti a nákladové efektivity, zejména u mechanismů pro dlouhodobou paměť a správu scénářů. Takový přístup by nevyhnutelně vedl k nutnosti nákladné přestavby systému v budoucnu.  
Naopak doporučená profesionální architektura, ačkoliv vyžaduje vyšší počáteční investici do vývoje, je navržena s ohledem na budoucí růst. Využívá pokročilé techniky, jako je směrování modelů (model routing) a sofistikovaná správa paměti, které vedou k výrazně nižším a lépe kontrolovatelným provozním nákladům v dlouhodobém horizontu. Tento přístup přímo podporuje klíčový požadavek na snadné rozšiřování poradenských scénářů bez nutnosti zásahů do kódu aplikace.

### **Přehled Plánu Realizace**

Navrhovaný projektový plán je rozdělen do čtyř hlavních fází s cílem dokončit vývoj a testování do ledna 2026\.

1. **Fáze 1: Základy a Jádro RAG (6 měsíců):** Vytvoření infrastruktury, implementace datového pipeline a základní funkce odpovídání na dotazy z databáze.  
2. **Fáze 2: Paměť a Scénáře (6 měsíců):** Implementace pokročilé správy paměti a prvního poradenského scénáře.  
3. **Fáze 3: Iterace a Zabezpečení (6 měsíců):** Vývoj zbývajících scénářů, uživatelské testování a implementace bezpečnostních a nákladových opatření.  
4. **Fáze 4: Předprodukční Příprava a Nasazení (3 měsíce):** Finální testování, audit a nasazení do produkčního prostředí.

---

## **Část I: Návrhy Architektury Řešení**

Tato část představuje tři odlišné architektonické návrhy, které se liší komplexností, náklady a schopnostmi. Každý návrh je analyzován z hlediska splnění funkčních požadavků projektu, což umožňuje jasné pochopení souvisejících kompromisů.

### **1.1. Návrh A: Rychlé MVP (Minimum Viable Product)**

Tento přístup klade důraz na maximální rychlost vývoje a minimalizaci počátečních nákladů. Cílem je dodat základní funkčnost, která postačí pro první testování s uživateli a validaci tržního potenciálu.

#### **Architektonický Nákres a Technologický Stack**

* **Výpočetní Vrstva:** Jedna instance služby Azure App Service v plánu Basic (B1/B2), na které poběží monolitická aplikace napsaná v Pythonu (s využitím frameworků jako Flask nebo FastAPI) nebo Node.js. Tento plán představuje nákladově efektivní řešení pro počáteční, středně velký provoz.1  
* **Data a Paměť:** Databáze Azure Cosmos DB v režimu **Serverless**. Tento model je ideální pro nepředvídatelný provoz v rané fázi projektu, protože náklady jsou účtovány pouze za provedené operace, čímž se eliminuje platba za nevyužitou, předem alokovanou kapacitu.3 Dlouhodobá paměť bude implementována jako jednoduchý, chronologicky řazený záznam konverzační historie pro každého uživatele.  
* **Inteligentní Jádro (RAG):** Služba Azure AI Search v plánu **Basic**. Tento plán poskytuje dedikované zdroje a dostatečnou úložnou kapacitu (až 15 GB) pro počáteční databázi univerzit, což je významný posun oproti sdílenému plánu Free.5 Vyhledávání bude založeno na kombinaci klíčových slov a základního vektorového vyhledávání.  
* **Jazykový Model (LLM):** Bude použit jeden, vysoce nákladově efektivní model, jako je **OpenAI GPT-4o-mini** nebo **Claude 3.5 Haiku**. Tyto modely nabízejí nejlepší poměr mezi výkonem a nízkou cenou pro všeobecné úlohy.9

#### **Implementace Funkčních Požadavků**

* **Dlouhodobá Paměť:** Při každém dotazu uživatele se z Cosmos DB načte kompletní historie konverzace a připojí se na začátek kontextu (promptu) pro LLM. Tento přístup je jednoduchý na implementaci, ale rychle se stává neefektivním z hlediska spotřeby tokenů.  
* **Dynamické Scénáře:** Tři úvodní poradenské scénáře budou pevně naprogramovány jako stavové automaty nebo sada pravidel přímo v kódu aplikace. Toto řešení je rigidní, ale jeho počáteční implementace je rychlá.  
* **Integrace s Databází:** Aplikace bude obsahovat dedikovaný modul s funkcemi pro přímé dotazování databáze univerzit prostřednictvím Azure AI Search. LLM bude instruován, aby tyto funkce volal, když detekuje relevantní dotaz od uživatele.

#### **Analýza Nákladů a Odhad Vývoje**

* **Model Spotřeby Tokenů:** Náklady na LLM API budou modelovány na základě průměrné konverzace o 10 tazích (uživatel-bot) 12 a průměrné velikosti kontextu 2000 tokenů na jeden tah (zahrnující historii, nový dotaz a odpověď).  
* **Náklady na Infrastrukturu:** Bude proveden detailní rozpis nákladů na Azure App Service, Cosmos DB (Serverless) a AI Search (Basic).  
* **Odhad Vývoje:** Na základě srovnání s průmyslovými daty pro "středně komplexní chatboty" 14 se odhaduje délka vývoje na 4–6 měsíců s malým týmem.

#### **Nevyhnutelný Strop MVP Řešení**

Zvolená implementace paměti, která při každém dotazu posílá celou historii konverzace, vede k rychlému nárůstu počtu vstupních tokenů, což přímo zvyšuje provozní náklady a v konečném důsledku narazí na limit kontextového okna jazykového modelu. Tento model je finančně neudržitelný, protože náklady na interakci s vracejícím se uživatelem budou exponenciálně vyšší než s novým. Podobně, pevné zakódování scénářů je v přímém rozporu s požadavkem na jejich budoucí snadné rozšiřování. Každý nový scénář by vyžadoval zásah programátora a nasazení nové verze celé aplikace, což je pomalé, nákladné a náchylné k chybám. Návrh A je tedy validní strategií pro rychlé ověření trhu, ale nepředstavuje udržitelnou architekturu. V případě úspěchu služby bude nutné počítat s rozsáhlou a nákladnou přestavbou na architekturu podobnou Návrhu B.

### **1.2. Návrh B: Škálovatelný Profesionální Systém**

Tato architektura je navržena pro produkční nasazení a klade důraz na rovnováhu mezi výkonem, škálovatelností a dlouhodobou udržovatelností. Zavádí orchestrační vrstvu pro efektivní správu komplexních procesů.

#### **Architektonický Nákres a Technologický Stack**

* **Výpočetní Vrstva:** Azure App Service v plánu **Premium (P1v3)**. Tento plán poskytuje vyšší výkon, pokročilé síťové funkce (např. integraci s VNet) a možnosti automatického škálování, které jsou nezbytné pro zvládání produkčního provozu.1  
* **Data a Paměť:** Azure Cosmos DB s **automatickým škálováním propustnosti (Autoscale Provisioned Throughput)**. Tento model je nákladově efektivnější než standardní alokovaná propustnost pro zátěže s předvídatelnými špičkami (např. v období podávání přihlášek na VŠ). Automaticky navyšuje a snižuje počet Request Units (RU) podle aktuální potřeby, čímž optimalizuje náklady.15 Paměť bude řešena hybridním modelem: plná historie pro několik posledních zpráv a souhrn starších konverzací generovaný pomocí LLM, uložený v samostatném databázovém poli.  
* **Inteligentní Jádro (RAG):** Azure AI Search v plánu **Standard (S1)**. Přechod na tento plán je klíčový, protože odemyká funkci **Semantic Ranker**. Tento nástroj dramaticky zvyšuje relevanci vyhledaných výsledků nad rámec prosté vektorové podobnosti, což vede k výrazně kvalitnějším odpovědím chatbota.6  
* **Orchestrační Framework:** Jádrem aplikace bude moderní, programátorský (code-first) framework jako **Microsoft Semantic Kernel** nebo **LangChain**. Tento framework bude řídit konverzační tok, používání nástrojů (dotazy do databáze, web search) a správu paměti.  
* **Strategie LLM:** Bude implementován přístup **směrování modelů (model routing)**. Jednoduchý a rychlý model (např. GPT-4o-mini) bude použit pro klasifikaci záměru uživatele a jednoduché dotazy do databáze. Výkonnější a schopnější model (např. OpenAI GPT-4o nebo Claude 3.5 Sonnet) bude volán pouze pro komplexní a nuancované poradenské scénáře. Tímto způsobem se optimalizují náklady i kvalita výstupů.10

#### **Implementace Funkčních Požadavků**

* **Dlouhodobá Paměť:** Správu paměti zajišťuje orchestrační framework. Načte nedávnou historii a "dlouhodobý souhrn" z Cosmos DB. Po skončení konverzace použije další volání LLM k aktualizaci tohoto souhrnu, čímž udržuje kontext poskytovaný hlavnímu modelu stručný a relevantní.  
* **Dynamické Scénáře:** Scénáře jsou definovány jako konfigurovatelné "řetězce" (chains) nebo "plánovače" (planners) v rámci orchestračního frameworku. Nové scénáře lze přidávat nebo upravovat aktualizací konfiguračních souborů nebo šablon promptů, často bez nutnosti nasazovat novou verzi celé aplikace. Tím je plně vyhověno požadavku na flexibilitu.  
* **Web Search (Volitelné):** Pokud bude tato funkce implementována, bude představovat další "nástroj" dostupný pro orchestrátor. Systém by využíval **Bing Search API prostřednictvím služby Grounding v Azure AI Foundry** 17, přičemž prompty by byly navrženy tak, aby prioritizovaly oficiální webové stránky univerzit.  
* **Nákladové a Bezpečnostní Mantinely:** Před LLM API bude nasazena brána **Azure API Management (APIM)**, která bude vynucovat tvrdé měsíční limity útraty. Logika v orchestračním frameworku bude omezovat délku konverzace a celkový počet tokenů na jednu relaci.

#### **Analýza Nákladů a Odhad Vývoje**

* **Model Spotřeby Tokenů:** Strategie směrování modelů bude zohledněna v kalkulaci, což výrazně sníží průměrné náklady na konverzaci ve srovnání s MVP. Náklady budou modelovány odděleně pro jednoduché a komplexní typy konverzací.  
* **Náklady na Infrastrukturu:** Vyšší náklady na App Service (Premium) a AI Search (Standard) budou detailně rozepsány a zdůvodněny přidanou hodnotou a pokročilými funkcemi.  
* **Odhad Vývoje:** Odhadovaná délka vývoje je 9–12 měsíců. Ačkoliv je to delší než u MVP, tento přístup buduje robustnější a rozšiřitelnější základ, což snižuje náklady na budoucí úpravy a refaktoring.19

### **1.3. Návrh C: Pokročilý Multi-agentní Systém**

Tento návrh představuje nejmodernější, do budoucna orientovanou architekturu, navrženou pro maximální modularitu, specializaci a škálovatelnost. Nepohlíží na chatbota jako na jedinou entitu, ale jako na tým spolupracujících AI agentů.

#### **Architektonický Nákres a Technologický Stack**

* **Výpočetní Vrstva:** Architektura založená na mikroslužbách, nasazená na **Azure Kubernetes Service (AKS)** nebo **Azure Container Apps**. Tento přístup poskytuje maximální kontrolu nad škálováním, odolností a nezávislým nasazováním jednotlivých komponent (agentů).  
* **Platforma pro Vývoj Agentů:** Pro návrh, správu a hostování jednotlivých agentů bude využita platforma **Azure AI Foundry** a její Agent Service.21 To poskytuje ucelené prostředí pro budování a provoz multi-agentního systému.  
* **Klíčoví Agenti:**  
  1. **Orchestrátor (Root Agent):** Vstupní bod pro uživatele. Jeho jediným úkolem je porozumět záměru uživatele a delegovat úkol na příslušného specializovaného agenta. Tento koncept je dobře popsán v rámci frameworků jako Google ADK.22  
  2. **Kariérní Poradce:** Specializovaný agent s pečlivě vyladěným promptem a přístupem k nejvýkonnějšímu LLM (např. GPT-4o) pro řešení nuancovaných poradenských scénářů.  
  3. **Databázový Agent:** Agent, jehož jedinými nástroji jsou funkce pro dotazování indexu v Azure AI Search. Pro svůj provoz by využíval levnější a rychlejší model optimalizovaný pro používání nástrojů.  
  4. **Web Search Agent (Volitelný):** Agent dedikovaný pro provádění webových vyhledávání a sumarizaci výsledků.  
  5. **Paměťový Agent:** Agent běžící na pozadí, zodpovědný za správu a sumarizaci historie konverzací uživatele. Potenciálně by mohl budovat znalostní graf preferencí uživatele v čase.  
* **Sdílené Služby:** Agenti by interagovali se sdílenými službami Azure AI Search (Standard S1/S2) a Azure Cosmos DB (Autoscale).

#### **Implementace Funkčních Požadavků**

* **Dynamické Scénáře:** Přidání nové schopnosti (např. "asistent pro žádosti o stipendium") se stává otázkou vývoje a nasazení nového, specializovaného agenta. Prompt Orchestrátora se pouze aktualizuje, aby o tomto novém členovi týmu věděl. To představuje nejvyšší možnou úroveň flexibility.  
* **Dlouhodobá Paměť:** Paměťový Agent může implementovat sofistikované techniky, jako je vytváření znalostního grafu zájmů studenta (např. (uživatel)-\[má\_zájem\_o\]-\>(obor: 'Umělá inteligence'), (uživatel)-\[preferuje\]-\>(univerzity\_v: 'Praha')). Tento graf lze následně dotazovat pro poskytování vysoce kontextuálních odpovědí v budoucích interakcích.

#### **Analýza Nákladů a Odhad Vývoje**

* **Náklady na Infrastrukturu:** Náklady na AKS/Container Apps jsou komplexnější, ale při větším měřítku mohou být efektivnější. Budou modelovány na základě potřebného počtu uzlů/replik.  
* **Odhad Vývoje:** Odhadovaná délka vývoje je 12–18 měsíců. Tento přístup má vyšší počáteční složitost a náklady kvůli režii spojené s mikroslužbami a nutnosti navrhnout komunikační protokoly mezi agenty. V dlouhodobém horizontu však nabízí bezkonkurenční udržovatelnost a škálovatelnost.19

#### **Budoucnost a Specializace**

Multi-agentní přístup zásadně odděluje jednotlivé funkcionality. To znamená, že agent Kariérního Poradce může být vylepšen novějším, výkonnějším LLM, aniž by to ovlivnilo jednoduchého a nákladově efektivního Databázového Agenta. Tato modularita dramaticky zjednodušuje údržbu a budoucí vylepšení. V rychle se vyvíjejícím světě AI, kde se neustále objevují nové modely a techniky, je tato schopnost izolovaných upgradů klíčová. Umožňuje rychleji a bezpečněji adoptovat nové technologie, minimalizuje riziko zanesení chyb do celého systému a snižuje komplexitu testování. Návrh C představuje nejvyšší počáteční investici, ale nabízí nejnižší dlouhodobé provozní tření a největší přizpůsobivost budoucím pokrokům v oblasti umělé inteligence. Je to strategicky nejzdravější volba pro společnost, která vnímá tohoto chatbota jako klíčové, dlouhodobé aktivum.

#### **Tabulka 1: Srovnávací Matice Architektonických Návrhů**

| Kritérium | Návrh A: Rychlé MVP | Návrh B: Škálovatelný Profesionální Systém | Návrh C: Pokročilý Multi-agentní Systém |
| :---- | :---- | :---- | :---- |
| **Hlavní Architektura** | Monolitická aplikace | Monolit s orchestračním frameworkem | Mikroslužby (agenti) |
| **Klíčové Technologie** | App Service (Basic), CosmosDB (Serverless), AI Search (Basic) | App Service (Premium), CosmosDB (Autoscale), AI Search (Standard), Semantic Kernel | AKS/Container Apps, Azure AI Foundry, Semantic Kernel |
| **Škálovatelnost (1-5)** | 2 | 4 | 5 |
| **Flexibilita (1-5)** | 1 | 4 | 5 |
| **Počáteční Čas Vývoje** | 4–6 měsíců | 9–12 měsíců | 12–18 měsíců |
| **Odhadované Náklady na Vývoj** | Nízké | Střední | Vysoké |
| **Odhad. Měsíční Náklady @ 1000 Uživ.** | Střední (rostoucí) | Nízké (stabilní) | Nízké (optimalizované) |

---

## **Část II: Detailní Analýza Klíčových Funkčních Komponent**

Tato část poskytuje podrobnou technickou analýzu nejdůležitějších funkčních požadavků a zkoumá implementační strategie, které jsou relevantní především pro doporučený Návrh B a pokročilý Návrh C.

### **2.1. Architektura Dlouhodobé Paměti**

Hlavní výzvou je najít rovnováhu mezi relevancí kontextu, provozními náklady a zajištěním plynulého zážitku pro vracejícího se uživatele.

* **Možnost 1: Plná Historie (přístup pro MVP):**  
  * **Metoda:** Ukládání všech zpráv mezi uživatelem a botem do jednoho dokumentu v Cosmos DB. Celá historie se připojuje ke každému novému promptu.  
  * **Výhody:** Jednoduchá implementace.  
  * **Nevýhody:** Finančně neudržitelné kvůli neustále rostoucímu počtu vstupních tokenů. Rychle narazí na limit kontextového okna modelu.  
* **Možnost 2: Sumarizace (profesionální přístup):**  
  * **Metoda:** Po každé konverzaci se provede samostatné volání LLM, které shrne klíčové informace, preference a výsledky. Následující konverzace začíná s tímto souhrnem a několika (např. 5–10) posledními zprávami z minulé interakce.  
  * **Výhody:** Udržuje velikost kontextového okna pod kontrolou a náklady předvídatelné.  
  * **Nevýhody:** Samotné generování souhrnu spotřebovává tokeny. Existuje riziko ztráty nuancí a důležitých detailů během sumarizace.  
* **Možnost 3: Hybridní RAG \+ Znalostní Graf (pokročilý přístup):**  
  * **Metoda:** Během konverzace se pomocí LLM extrahují strukturované entity a vztahy (např. (uživatel)-\[preferuje\]-\>(obor: 'AI'), (uživatel)-\[zajímal\_se\_o\]-\>(univerzita: 'ČVUT')). Tyto informace (trojice) se ukládají do grafové databáze nebo strukturovaného pole v Cosmos DB. Pro další konverzaci se z tohoto grafu načtou relevantní fakta a vloží se do promptu.  
  * **Výhody:** Nejvýkonnější a kontextuálně nejbohatší metoda. Vytváří detailní, dotazovatelný profil uživatele.  
  * **Nevýhody:** Nejvyšší složitost implementace.

### **2.2. Návrh Dynamického Scénářového Enginu**

Systém musí uživatele provádět komplexními, vícekrokovými poradenskými scénáři způsobem, který je přirozený a snadno rozšiřitelný i pro osoby bez technických znalostí.

* **Možnost 1: Pravidlové Stavové Automaty (přístup pro MVP):**  
  * **Metoda:** Logika scénářů je pevně zakódována v aplikaci. Pokud uživatel řekne X, zeptej se na Y. Pokud odpoví Z, přejdi do stavu Q.  
  * **Výhody:** Rychlá implementace pro malý, pevně daný počet scénářů.  
  * **Nevýhody:** Křehké, neškálovatelné a jakákoli změna vyžaduje zásah programátora.  
* **Možnost 2: Orchestrace Řízená Frameworkem (profesionální přístup):**  
  * **Metoda:** Využití funkcí v rámci frameworků jako LangChain (Chains) nebo Semantic Kernel (Planners) pro definici toku. LLM dostane cíl (např. "pomoz uživateli zúžit výběr na 3 potenciální obory") a sadu nástrojů/kroků, které může použít. Framework řídí jejich provádění.  
  * **Výhody:** Vysoce flexibilní. Scénáře lze definovat v konfiguraci nebo šablonách promptů, což umožňuje snadné aktualizace.  
  * **Nevýhody:** Vyžaduje expertizu ve zvoleném frameworku. Ladění může být složité.24  
* **Možnost 3: Autonomní Plánování Agenta (pokročilý přístup):**  
  * **Metoda:** Agent Kariérního Poradce dostane obecný cíl a personu. Autonomně se rozhoduje, jaké otázky položit a jaké informace poskytnout, aby dosáhl svého cíle. Nejedná se o předdefinovaný "scénář", ale o konverzaci orientovanou na cíl.  
  * **Výhody:** Nejpřirozenější a lidsky nejpodobnější interakce. Dokáže se přizpůsobit neočekávaným vstupům od uživatele.  
  * **Nevýhody:** Méně předvídatelné, vyžaduje robustní mantinely, aby agent neodbočoval od tématu, a spoléhá na nejpokročilejší (a nejdražší) LLM.

### **2.3. Optimalizace Vyhledávání Dat (RAG)**

Klíčovým úkolem je zajistit, aby informace načtené z databáze byly přesné, relevantní a úplné.

* **Zpracování a Segmentace Dat (Chunking):**  
  * Prvním krokem je zpracování zdrojových dat (informace o univerzitách, katalogy předmětů atd.) do prohledávatelného formátu.  
  * Pro robustní parsování různých formátů dokumentů (PDF, DOCX) a jejich převod do čistého formátu Markdown bude využita služba **Azure AI Document Intelligence**.25  
  * Bude použita metoda **sémantické segmentace (Semantic Chunking)**. Místo rozdělení textu na segmenty pevné velikosti bude text dělen na základě sémantického významu (např. podle odstavců nebo sekcí). Tím se zachovává kontext a dosahuje se lepších výsledků při vyhledávání.25  
* **Strategie Indexování v Azure AI Search:**  
  * Nezbytný je přístup **hybridního vyhledávání (Hybrid Search)**. Index bude obsahovat jak původní text (pro vyhledávání podle klíčových slov), tak vektorové reprezentace (embeddings) textových segmentů (pro sémantické vyhledávání).  
  * To systému umožní najít dokumenty, které obsahují přesná klíčová slova (jako specifický kód předmětu), i dokumenty, které jsou koncepčně příbuzné, ale nemusí sdílet stejná slova.  
* **Role Funkce Semantic Ranker:**  
  * Tato funkce, dostupná v plánech Standard a vyšších služby Azure AI Search, je klíčovou komponentou pro dosažení vysoké kvality.6  
  * Poté, co je pomocí klíčových slov a vektorového vyhledávání nalezena počáteční sada výsledků, Semantic Ranker použije sofistikovaný model hlubokého učení k přeuspořádání přibližně 50 nejlepších výsledků podle jejich skutečné koncepční relevance k dotazu.  
  * Tento dvoufázový proces je zásadní pro překonání limitů čistě vektorového vyhledávání. Zatímco vektorové vyhledávání najde dokumenty, které jsou *podobné*, Semantic Ranker pomáhá najít dokumenty, které skutečně *odpovídají na otázku uživatele*. To se přímo promítá do kvalitnějších a užitečnějších odpovědí od chatbota.

### **2.4. Implementace Nákladových a Bezpečnostních Mantinelů**

Systém musí být chráněn před zneužitím a nekontrolovatelnými náklady, zejména v úvodním období, kdy bude služba poskytována zdarma.

* **Mechanismy Kontroly Nákladů:**  
  * **Měsíční Strop Útraty:** Bude nakonfigurována instance **Azure API Management (APIM)** s politikou měsíční kvóty. Jakmile bude dosažen definovaný limit útraty, APIM přestane předávat požadavky na LLM API, čímž efektivně zastaví službu pro daný měsíc, jak bylo požadováno.  
  * **Limity na Konverzaci:** Aplikační logika bude sledovat počet tahů a celkový počet spotřebovaných tokenů v jedné relaci. Při překročení stanovené hranice bude konverzace elegantně ukončena, aby se předešlo zneužití.  
  * **Caching:** Bude implementováno cachování pro identické, často se opakující dotazy (např. "Jaké jsou přijímací podmínky na informatiku na ČVUT?"). To lze provést na aplikační vrstvě nebo pomocí služby jako Azure Cache for Redis. Koncept "prompt caching", popsaný v dokumentaci Claude, je podobně výkonným mechanismem.26  
* **Prevence Zneužití a Bezpečnost (Guardrails):**  
  * **Inženýrství Systémového Promptu:** "Meta-prompt" neboli systémový prompt bude obsahovat striktní instrukce pro chatbota, které definují jeho roli a omezení. Bude obsahovat fráze jako: "Jsi akademický poradce pro české a slovenské vysoké školy. Nesmíš odpovídat na otázky mimo toto téma. Pokud se uživatel zeptá na nesouvisející téma, zdvořile odmítni odpovědět a vrať konverzaci zpět k výběru VŠ."  
  * **Moderace Obsahu:** Pro vstupy od uživatelů i výstupy z modelu lze využít službu **Azure AI Content Safety** k detekci a filtrování škodlivého nebo nevhodného obsahu.21  
  * **Validace Vstupů/Výstupů:** Aplikační vrstva bude validovat strukturu dat před odesláním do LLM a po přijetí odpovědi, zejména při použití funkcí pro volání nástrojů. Tím se předejde útokům typu "prompt injection" a dalším zranitelnostem.

---

## **Část III: Hodnocení Vývojových Frameworků a Platforem**

Tato část poskytuje konečné doporučení ohledně klíčového technologického stacku a přechází od otázky "co" (architektura) k otázce "jak" (implementace).

### **3.1. Spravované Platformy (Low-Code/No-Code)**

* **Microsoft Copilot Studio:**  
  * **Analýza:** Výkonná low-code platforma pro tvorbu botů v ekosystému Microsoftu. Její cenový model je založen na "Copilot Credits", přičemž generativní AI akce stojí více kreditů.27 Nabízí vynikající integraci s Microsoft 365\.  
  * **Vhodnost pro Projekt:** Ačkoliv umožňuje rychlý počáteční vývoj, jeho grafické rozhraní a předdefinované komponenty se mohou stát omezujícím faktorem pro komplexní, na míru šitou logiku, kterou vyžadují dynamické scénáře a sofistikovaná správa paměti. Rozšiřitelnost je možná, ale může být složitá.29 Představuje potenciální "strop flexibility".  
* **Google Dialogflow CX:**  
  * **Analýza:** Platforma pro konverzační AI založená na stavových automatech. Je silná ve strukturovaných konverzacích (jako rezervace letenky) a má cenový model založený na počtu požadavků.31 Nyní zahrnuje i generativní funkce jako "Playbooks" a "Data Store Handlers".33  
  * **Vhodnost pro Projekt:** Podobně jako Copilot Studio, exceluje ve strukturovaných tocích, ale může být méně vhodná pro vysoce dynamickou, generativní povahu kariérních poradenských scénářů. Paradigma stavového automatu se může stát těžkopádným při rostoucí komplexitě scénářů.

### **3.2. Programátorské (Code-First) Orchestrační Frameworky**

* **LangChain:**  
  * **Analýza:** Nejvyspělejší a nejrozšířenější framework s obrovským ekosystémem integrací pro modely, vektorové databáze a nástroje.34 Jeho základními abstrakcemi jsou "Chains" (řetězce) a "Agents" (agenti). Je chválen pro svou flexibilitu a rychlé prototypování, ale někdy kritizován za přílišnou abstrakci nebo "magii", což může komplikovat ladění a produkční nasazení.24  
  * **Integrace s Azure:** Dobrá, s dedikovanými balíčky pro Azure OpenAI, Azure AI Search a Cosmos DB.37  
* **Microsoft Semantic Kernel:**  
  * **Analýza:** Odpověď Microsoftu na LangChain, navržená se strukturovanějším, softwarově-inženýrským přístupem. Jeho klíčovými koncepty jsou "Plugins" (kolekce "Functions") a "Planners", které automaticky sestavují sekvenci funkcí k dosažení cíle.40 Je chválen za čistý design a silnou podporu pro C\#/.NET, což z něj činí přirozenou volbu pro podniky již investované do Microsoft stacku.24  
  * **Integrace s Azure:** Nativní a prvotřídní. Je od základu navržen pro bezproblémovou spolupráci s AI ekosystémem Azure.  
* **LlamaIndex:**  
  * **Analýza:** Zatímco LangChain je univerzální orchestrační framework, LlamaIndex je úzce zaměřen na to, aby byl nejlepším nástrojem pro "R" v RAG: Retrieval (vyhledávání). Nabízí pokročilejší a specializovanější funkce pro zpracování dat, indexaci a komplexní transformace dotazů nad daty.43  
  * **Vhodnost pro Projekt:** Není přímým konkurentem LangChain/SK pro celou aplikační logiku, ale může být použit *v kombinaci* s nimi. Například by bylo možné použít Semantic Kernel pro celkovou orchestraci a LlamaIndex pro specifický úkol dotazování databáze univerzit. To by však přidalo další závislost do projektu.

#### **Rozhodující Faktor: Filozofie Orchestrace a Soulad s Ekosystémem**

Volba mezi LangChain a Semantic Kernel není pouze otázkou funkcí, ale filozofie. LangChain nabízí obrovskou, rozmanitou sadu nástrojů ideální pro rychlé experimentování. Semantic Kernel nabízí strukturovanější, předvídatelnější a pro podnikové prostředí připravenější přístup k začlenění AI funkcionality do stávajících softwarových paradigmat. Projekt vyžaduje robustní, udržitelný systém s plány na budoucí rozšiřování, což upřednostňuje předvídatelné softwarově-inženýrské postupy před rychlým, potenciálně neuspořádaným prototypováním. Spolu s explicitní preferencí pro ekosystém Azure se Semantic Kernel jeví jako strategicky nejvhodnější volba. Jako produkt Microsoftu, navržený pro integraci se službami Azure a podnikovými vývojovými postupy, jeho designová filozofie "pluginů" a "funkcí" čistě mapuje na existující softwarové koncepty, jako jsou knihovny a metody.40 Tím se snižuje riziko spojené s rychlými změnami v open-source projektech a zajišťuje se dlouhodobá stabilita.  
**Microsoft Semantic Kernel je proto doporučeným frameworkem pro Návrh B a C.**

#### **Tabulka 2: Srovnávací Karta Hodnocení Frameworků**

| Kritérium | LangChain | Microsoft Semantic Kernel | LlamaIndex (jako orchestrátor) |
| :---- | :---- | :---- | :---- |
| **Integrace s Azure** | 4/5 (Dobrá, komunitní) | 5/5 (Nativní, prvotřídní) | 3/5 (Základní) |
| **Flexibilita/Rozšiřitelnost** | 5/5 (Extrémně flexibilní) | 4/5 (Strukturovaná flexibilita) | 3/5 (Zaměřeno na RAG) |
| **Ekosystém a Komunita** | 5/5 (Největší) | 3/5 (Rostoucí, podniková) | 4/5 (Silná v RAG oblasti) |
| **Připravenost pro Produkci** | 3/5 (Vyžaduje opatrnost) | 5/5 (Navrženo pro podniky) | 4/5 (Stabilní v RAG) |
| **Křivka Učení** | 3/5 (Může být matoucí) | 4/5 (Jasné koncepty) | 4/5 (Jasné zaměření) |
| **Možnosti Ladění** | 2/5 (Často kritizováno) | 4/5 (Strukturovanější) | 4/5 (Přehledné) |

### **3.3. Integrované Platformy pro Vývoj AI**

* **Azure AI Foundry:**  
  * **Analýza:** Integrovaná platforma na Azure pro celý životní cyklus AI, od výběru a ladění modelů po vývoj agentů, nasazení a monitorování.21 Funguje jako centrální hub neboli "továrna na agenty". Není to samotný framework, ale platforma pro  
    *provozování* aplikací postavených na frameworcích jako Semantic Kernel.  
  * **Doporučení:** Toto by měla být centrální platforma použitá pro správu projektu, zejména pro Návrhy B a C. Poskytuje nezbytné MLOps schopnosti (monitorování, hodnocení, bezpečnost) pro produkční systém.  
* **Google Vertex AI Agent Builder:**  
  * **Analýza:** Konkurenční platforma od Googlu, která nabízí podobné schopnosti pro budování a nasazování generativních AI aplikací a agentů na Google Cloud.45 Je úzce integrována s modely Googlu (Gemini) a jeho infrastrukturou.  
  * **Doporučení:** Ačkoliv se jedná o výkonnou platformu, neodpovídá stanovené preferenci projektu pro infrastrukturu Azure. Pro tento projekt neexistuje pádný důvod upřednostnit ji před nativní platformou Azure AI Foundry.

---

## **Část IV: Konsolidovaný Plán a Finanční Projekce**

Tato závěrečná část syntetizuje architektonickou a finanční analýzu do konkrétního projektového plánu.

### **4.1. Navrhovaný Časový Plán Projektu (do ledna 2026\)**

Doporučuje se fázový přístup, který umožňuje iterativní vývoj a zapracování zpětné vazby.

* **Fáze 1: Základy a Jádro RAG (6 měsíců)**  
  * Nastavení infrastruktury Azure (AI Foundry, Cosmos DB, AI Search).  
  * Vývoj datového pipeline pro zpracování databáze univerzit.  
  * Implementace základního RAG cyklu pomocí Semantic Kernel pro odpovídání na jednoduché, faktické dotazy z databáze.  
  * Vývoj počátečního uživatelského rozhraní a backendové služby.  
* **Fáze 2: Implementace Paměti a Scénářů (6 měsíců)**  
  * Implementace hybridní dlouhodobé paměti a logiky sumarizace.  
  * Vývoj prvního ze tří kariérních poradenských scénářů s využitím plánovače v Semantic Kernel.  
  * Integrace mechanismu pro zpětnou vazbu od uživatelů a základní analytiky (backendové události pro GA4 46).  
* **Fáze 3: Iterace a Zabezpečení (6 měsíců)**  
  * Vývoj zbývajících dvou scénářů.  
  * Rozsáhlé uživatelské testování a zapracování zpětné vazby.  
  * Implementace robustních nákladových a bezpečnostních mantinelů (APIM, inženýrství promptů).  
  * Ladění výkonu a optimalizace.  
* **Fáze 4: Předprodukční Příprava a Nasazení (3 měsíce)**  
  * Finální zajištění kvality a oprava chyb.  
  * Bezpečnostní audit.  
  * Nasazení do produkčního prostředí a finální integrace s hlavní aplikací.  
  * Spuštění (do ledna 2026).

### **4.2. Komplexní Tříletý Přehled Nákladů**

Tato sekce poskytuje klíčový finanční přehled nezbytný pro strategické plánování a rozpočtování.

* **Předpoklady pro Modelování Nákladů:**  
  * **Růst Uživatelů:** Rok 1: 320, Rok 2: 1000, Rok 3: 2000 platících uživatelů.  
  * **Vzor Použití:** Každý uživatel provede měsíčně 2 "komplexní" konverzace (15 tahů) a 5 "jednoduchých" konverzací (5 tahů).  
  * **Odhady Tokenů:** Založeno na analýze typů konverzací, velikosti promptů (včetně paměti) a volbě modelů (s využitím strategie směrování pro Návrhy B a C).  
  * **Náklady na Vývoj:** Založeno na týmu 4–5 inženýrů (architekt, 2x AI/backend, 1x frontend, 1x QA) se sazbami pro střední/východní Evropu ($50–$80/hod) pro nákladově efektivnější odhad.14  
  * **Údržba:** Odhadována na 15–20 % počátečních nákladů na vývoj, ročně.47

#### **Tabulka 3: Tříletá Projekce Celkových Nákladů na Vlastnictví (TCO) v USD**

| Nákladová Položka | Návrh A: Rychlé MVP | Návrh B: Profesionální Systém | Návrh C: Multi-agentní Systém |
| :---- | :---- | :---- | :---- |
|  | **Rok 1 / 2 / 3** | **Rok 1 / 2 / 3** | **Rok 1 / 2 / 3** |
| **Jednorázové Náklady na Vývoj (CapEx)** | $40,000 | $120,000 | $180,000 |
| **Provozní Náklady (OpEx) \- Ročně** |  |  |  |
| *Infrastruktura Azure* | $1,200 / $1,500 / $2,000 | $4,500 / $5,000 / $6,000 | $7,000 / $8,000 / $10,000 |
| *LLM API (OpenAI/Claude)* | $2,500 / $8,000 / $16,500 | $1,500 / $4,800 / $9,800 | $1,200 / $3,800 / $7,800 |
| *Údržba a Podpora (15%)* | $6,000 / $6,000 / $6,000 | $18,000 / $18,000 / $18,000 | $27,000 / $27,000 / $27,000 |
| **Celkové Roční Náklady** | $49,700 / $15,500 / $24,500 | $144,000 / $27,800 / $33,800 | $215,200 / $38,800 / $44,800 |
| **Celkové TCO za 3 Roky** | **$89,700** | **$205,600** | **$298,800** |

Tato projekce jasně ukazuje, jak se počáteční investice promítají do dlouhodobých provozních nákladů. Zatímco Návrh A má nejnižší počáteční náklady, jeho provozní náklady na LLM API rostou nejrychleji s počtem uživatelů kvůli neefektivní správě paměti. Naopak Návrh B a C, ačkoliv vyžadují vyšší počáteční investici, dosahují díky pokročilým optimalizačním technikám výrazně lepších provozních nákladů v pozdějších letech, což z nich činí strategicky výhodnější investici pro dlouhodobý a udržitelný provoz služby.

#### **Works cited**

1. Pricing – App Service for Linux | Microsoft Azure, accessed on October 4, 2025, [https://azure.microsoft.com/en-us/pricing/details/app-service/linux/](https://azure.microsoft.com/en-us/pricing/details/app-service/linux/)  
2. Azure App Service on Windows pricing, accessed on October 4, 2025, [https://azure.microsoft.com/en-us/pricing/details/app-service/windows/](https://azure.microsoft.com/en-us/pricing/details/app-service/windows/)  
3. Pricing Model for Azure Cosmos DB, accessed on October 4, 2025, [https://docs.azure.cn/en-us/cosmos-db/how-pricing-works](https://docs.azure.cn/en-us/cosmos-db/how-pricing-works)  
4. Pricing Model for Azure Cosmos DB \- Microsoft Learn, accessed on October 4, 2025, [https://learn.microsoft.com/en-us/azure/cosmos-db/how-pricing-works](https://learn.microsoft.com/en-us/azure/cosmos-db/how-pricing-works)  
5. Azure AI Search Pricing \- TrustRadius, accessed on October 4, 2025, [https://www.trustradius.com/products/azure-ai-search/pricing](https://www.trustradius.com/products/azure-ai-search/pricing)  
6. Azure AI Search pricing, accessed on October 4, 2025, [https://azure.microsoft.com/en-us/pricing/details/search/](https://azure.microsoft.com/en-us/pricing/details/search/)  
7. Service Limits for Tiers and SKUs \- Azure AI Search | Microsoft Learn, accessed on October 4, 2025, [https://learn.microsoft.com/en-us/azure/search/search-limits-quotas-capacity](https://learn.microsoft.com/en-us/azure/search/search-limits-quotas-capacity)  
8. Azure AI Search Cost: Pricing Explained \- BytePlus, accessed on October 4, 2025, [https://www.byteplus.com/en/topic/404880](https://www.byteplus.com/en/topic/404880)  
9. Claude 3.5 Haiku \- Anthropic, accessed on October 4, 2025, [https://www.anthropic.com/claude/haiku](https://www.anthropic.com/claude/haiku)  
10. Claude AI Pricing: Choosing the Right Model \- PromptLayer Blog, accessed on October 4, 2025, [https://blog.promptlayer.com/claude-ai-pricing-choosing-the-right-model/](https://blog.promptlayer.com/claude-ai-pricing-choosing-the-right-model/)  
11. How to Calculate OpenAI API Price for GPT-4, GPT-4o and GPT-3.5 Turbo?, accessed on October 4, 2025, [https://www.analyticsvidhya.com/blog/2024/12/openai-api-cost/](https://www.analyticsvidhya.com/blog/2024/12/openai-api-cost/)  
12. 35+ Chatbot Statistics You Need to Know for 2025 \- LocaliQ, accessed on October 4, 2025, [https://localiq.com/blog/chatbot-statistics/](https://localiq.com/blog/chatbot-statistics/)  
13. 80+ Chatbot Statistics & Trends in 2025 \[Usage, Adoption Rates\] \- Tidio, accessed on October 4, 2025, [https://www.tidio.com/blog/chatbot-statistics/](https://www.tidio.com/blog/chatbot-statistics/)  
14. Cost to Build a Chatbot: In-Depth Pricing Guide for 2025 \- Cleveroad, accessed on October 4, 2025, [https://www.cleveroad.com/blog/chatbot-development-cost/](https://www.cleveroad.com/blog/chatbot-development-cost/)  
15. Azure Cosmos DB pricing, accessed on October 4, 2025, [https://azure.microsoft.com/en-us/pricing/details/cosmos-db/autoscale-provisioned/](https://azure.microsoft.com/en-us/pricing/details/cosmos-db/autoscale-provisioned/)  
16. Azure Cosmos DB Pricing \- Cost Breakdown & Savings Guide \- Pump, accessed on October 4, 2025, [https://www.pump.co/blog/azure-cosmos-db-pricing](https://www.pump.co/blog/azure-cosmos-db-pricing)  
17. Grounding with Bing Pricing Details \- Microsoft, accessed on October 4, 2025, [https://www.microsoft.com/en-us/bing/apis/grounding-pricing](https://www.microsoft.com/en-us/bing/apis/grounding-pricing)  
18. Microsoft ends Bing Search APIs on August 11, alternative costs 40-483% more \- PPC Land, accessed on October 4, 2025, [https://ppc.land/microsoft-ends-bing-search-apis-on-august-11-alternative-costs-40-483-more/](https://ppc.land/microsoft-ends-bing-search-apis-on-august-11-alternative-costs-40-483-more/)  
19. AI Chatbot Development Cost In 2025: Pricing & Key Factors \- KumoHQ, accessed on October 4, 2025, [https://www.kumohq.co/blog/ai-chatbot-development-cost](https://www.kumohq.co/blog/ai-chatbot-development-cost)  
20. How Much Does a Chatbot Really Cost in 2025? A Straightforward Guide to Pricing, Building, and Saving Big | Quickchat AI, accessed on October 4, 2025, [https://quickchat.ai/post/how-much-does-chatbot-cost](https://quickchat.ai/post/how-much-does-chatbot-cost)  
21. Azure AI Foundry documentation | Microsoft Learn, accessed on October 4, 2025, [https://learn.microsoft.com/en-us/azure/ai-foundry/](https://learn.microsoft.com/en-us/azure/ai-foundry/)  
22. Agent Team \- Agent Development Kit \- Google, accessed on October 4, 2025, [https://google.github.io/adk-docs/tutorials/agent-team/](https://google.github.io/adk-docs/tutorials/agent-team/)  
23. How Much Do AI Chatbots Cost? Estimates for 2025 \- Crescendo.ai, accessed on October 4, 2025, [https://www.crescendo.ai/blog/how-much-do-chatbots-cost](https://www.crescendo.ai/blog/how-much-do-chatbots-cost)  
24. LangChain vs Semantic Kernel \- Getting Started with Artificial Intelligence, accessed on October 4, 2025, [https://www.gettingstarted.ai/langchain-vs-semantic-kernel/](https://www.gettingstarted.ai/langchain-vs-semantic-kernel/)  
25. Retrieval-Augmented Generation (RAG) with Azure AI Document Intelligence, accessed on October 4, 2025, [https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept/retrieval-augmented-generation?view=doc-intel-4.0.0](https://learn.microsoft.com/en-us/azure/ai-services/document-intelligence/concept/retrieval-augmented-generation?view=doc-intel-4.0.0)  
26. 10 Tips to Reduce Chatbot Response Time \- Quidget AI, accessed on October 4, 2025, [https://quidget.ai/blog/ai-automation/10-tips-to-reduce-chatbot-response-time/](https://quidget.ai/blog/ai-automation/10-tips-to-reduce-chatbot-response-time/)  
27. Microsoft 365 Copilot Pricing – AI Agents | Copilot Studio, accessed on October 4, 2025, [https://www.microsoft.com/en-us/microsoft-365-copilot/pricing/copilot-studio](https://www.microsoft.com/en-us/microsoft-365-copilot/pricing/copilot-studio)  
28. Customize Copilot and Create Agents | Microsoft Copilot Studio, accessed on October 4, 2025, [https://www.microsoft.com/en/microsoft-copilot/microsoft-copilot-studio](https://www.microsoft.com/en/microsoft-copilot/microsoft-copilot-studio)  
29. Custom engine agents for Microsoft 365 overview, accessed on October 4, 2025, [https://learn.microsoft.com/en-us/microsoft-365-copilot/extensibility/overview-custom-engine-agent](https://learn.microsoft.com/en-us/microsoft-365-copilot/extensibility/overview-custom-engine-agent)  
30. Microsoft Copilot Alternatives for Enterprise AI Teams \- Dynamiq, accessed on October 4, 2025, [https://www.getdynamiq.ai/post/microsoft-copilot-alternatives-for-enterprise-ai-teams](https://www.getdynamiq.ai/post/microsoft-copilot-alternatives-for-enterprise-ai-teams)  
31. Dialogflow pricing \- Google Cloud, accessed on October 4, 2025, [https://cloud.google.com/dialogflow/pricing](https://cloud.google.com/dialogflow/pricing)  
32. Dialog Flow CX, pricing, by session? by request? \[closed\] \- Stack Overflow, accessed on October 4, 2025, [https://stackoverflow.com/questions/68671074/dialog-flow-cx-pricing-by-session-by-request](https://stackoverflow.com/questions/68671074/dialog-flow-cx-pricing-by-session-by-request)  
33. accessed on January 1, 1970, [https://cloud.google.com/dialogflow/cx/docs/quick/build-agent-playbook\#about\_playbooks](https://cloud.google.com/dialogflow/cx/docs/quick/build-agent-playbook#about_playbooks)  
34. LangChain vs LlamaIndex: A Detailed Comparison \- DataCamp, accessed on October 4, 2025, [https://www.datacamp.com/blog/langchain-vs-llamaindex](https://www.datacamp.com/blog/langchain-vs-llamaindex)  
35. LangChain vs LlamaIndex \- Reddit, accessed on October 4, 2025, [https://www.reddit.com/r/LangChain/comments/1bbog83/langchain\_vs\_llamaindex/](https://www.reddit.com/r/LangChain/comments/1bbog83/langchain_vs_llamaindex/)  
36. New to RAG, LangChain or something else? \- Reddit, accessed on October 4, 2025, [https://www.reddit.com/r/Rag/comments/1mmgijg/new\_to\_rag\_langchain\_or\_something\_else/](https://www.reddit.com/r/Rag/comments/1mmgijg/new_to_rag_langchain_or_something_else/)  
37. Get started with Serverless AI Chat with RAG using LangChain.js \- Microsoft Learn, accessed on October 4, 2025, [https://learn.microsoft.com/en-us/azure/developer/javascript/ai/get-started-app-chat-template-langchainjs](https://learn.microsoft.com/en-us/azure/developer/javascript/ai/get-started-app-chat-template-langchainjs)  
38. Should I use Azure OpenAI or LangChain with OpenAI API for enterprise security and cost efficiency? \- Microsoft Learn, accessed on October 4, 2025, [https://learn.microsoft.com/en-ie/answers/questions/2236992/should-i-use-azure-openai-or-langchain-with-openai](https://learn.microsoft.com/en-ie/answers/questions/2236992/should-i-use-azure-openai-or-langchain-with-openai)  
39. Unleashing LLM Potential with RAG: Building Intelligent Assistants with Azure and LangChain \- Presidio, accessed on October 4, 2025, [https://www.presidio.com/technical-blog/unleashing-llm-potential-with-rag-building-intelligent-assistants-with-azure-and-langchain/](https://www.presidio.com/technical-blog/unleashing-llm-potential-with-rag-building-intelligent-assistants-with-azure-and-langchain/)  
40. Langchain vs. Semantic Kernel \- Medium, accessed on October 4, 2025, [https://medium.com/@heyamit10/langchain-vs-semantic-kernel-d7e5de87c288](https://medium.com/@heyamit10/langchain-vs-semantic-kernel-d7e5de87c288)  
41. Semantic Kernel VS LangChain: The Ultimate AI Development Framework Guide \- Kanerika, accessed on October 4, 2025, [https://kanerika.com/blogs/semantic-kernel-vs-langchain/](https://kanerika.com/blogs/semantic-kernel-vs-langchain/)  
42. Langchain or Semantic Kernel? : r/aipromptprogramming \- Reddit, accessed on October 4, 2025, [https://www.reddit.com/r/aipromptprogramming/comments/1chka3k/langchain\_or\_semantic\_kernel/](https://www.reddit.com/r/aipromptprogramming/comments/1chka3k/langchain_or_semantic_kernel/)  
43. Llamaindex vs Langchain: What's the difference? \- IBM, accessed on October 4, 2025, [https://www.ibm.com/think/topics/llamaindex-vs-langchain](https://www.ibm.com/think/topics/llamaindex-vs-langchain)  
44. LlamaIndex vs. LangChain: Which RAG Tool is Right for You? \- n8n Blog, accessed on October 4, 2025, [https://blog.n8n.io/llamaindex-vs-langchain/](https://blog.n8n.io/llamaindex-vs-langchain/)  
45. Vertex AI Platform | Google Cloud, accessed on October 4, 2025, [https://cloud.google.com/vertex-ai](https://cloud.google.com/vertex-ai)  
46. Send Measurement Protocol events to Google Analytics, accessed on October 4, 2025, [https://developers.google.com/analytics/devguides/collection/protocol/ga4/sending-events](https://developers.google.com/analytics/devguides/collection/protocol/ga4/sending-events)  
47. Chatbot Development Cost Guide: From Planning to ROI \- Clavax Technologies LLC, accessed on October 4, 2025, [https://www.clavax.com/blog/chatbot-development-cost-guide-from-planning-to-roi/](https://www.clavax.com/blog/chatbot-development-cost-guide-from-planning-to-roi/)  
48. Chatbot Development Cost: Guide to Budgeting and Key Factors, accessed on October 4, 2025, [https://topflightapps.com/ideas/chatbot-development-cost/](https://topflightapps.com/ideas/chatbot-development-cost/)