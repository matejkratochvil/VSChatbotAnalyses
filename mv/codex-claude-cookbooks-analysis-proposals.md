# Analýza repoužití Claude cookbooků pro VSChatBot

## Účel dokumentu
- Zhodnotit, které hotové příklady z `resources/repos_cookbooks/claude-cookbooks` a `resources/repos_cookbooks/claude-quickstarts` lze využít pro projekt VSChatBot.
- Popsat míru proveditelnosti, potřebné úpravy, očekávané přínosy a rizika.
- Navrhnout konkrétní integrační kroky a kombinace komponent.

## Shrnutí doporučení
- Vhodné je kombinovat několik modulů: orchestraci a evaluace z `claude-cookbooks/patterns/agents`, paměťové techniky z `tool_use/memory_cookbook`, RAG pipeline a contextual embeddings ze `skills/`, a rychlé UI/agent smyčky z `claude-quickstarts/agents` + `customer-support-agent`.
- Největší úsilí bude u vícejazyčnosti, mapování na česká data a přepojení na Azure infrastrukturu. Logika agentů a evaluací je však do značné míry přenositelná.

## Přehled repoužitelných zdrojů

### `claude-cookbooks/patterns/agents`
- **Obsah:** orchestrátor-worker a evaluator-optimizer workflow v `.ipynb`, utilitní funkce (`util.py`).
- **Proveditelnost:** vysoká; logika plánovače a předávání kontextu odpovídá požadavku na scénáře (kariérní poradce + specialista na informace z DB).
- **Úpravy:**
  - Přepsat systémové prompty do češtiny/slovenštiny.
  - Nahradit ukázkové nástroje za MCP nástroje s přístupem k interní DB (podle `codex-vschatbot-implementation.md`).
  - Doplňkový evaluator může vyhodnocovat kvalitu citací a vhodnost doporučení (nutné dodat dataset).
- **Rizika:** notebooková forma → nutnost převést do produkčního kódu; potřeba testovat s OpenAI/Claude modely dle licencí.

### `claude-cookbooks/tool_use/memory_cookbook` a `memory_tool.py`
- **Obsah:** strategie krátkodobé/dlouhodobé paměti, příklady shrnování, ukázka externího úložiště.
- **Proveditelnost:** vysoká; odpovídá požadavku na persistentní preference studentů a kvízové výsledky.
- **Úpravy:**
  - Integrace s vlastní DB přes MCP server.
  - Lokalizace promptů a pojmenování entit (např. "preferuje kombinaci IT a matematiky").
  - Přidat konverzní vrstvu pro práci s více jazyky.
- **Rizika:** potřeba důsledně omezit ukládání PII (soulad s GDPR); nutné testy ohledně objemu dat a tokenů.

### `claude-cookbooks/tool_use/customer_service_agent.ipynb`
- **Obsah:** agent se sadu nástrojů, fallbacky, strukturovaný dialog.
- **Proveditelnost:** střední; scénář "zákaznické podpory" lze přizpůsobit na "studijní poradnu".
- **Úpravy:**
  - Přemapovat sloty/intenty na studijní potřeby (výběr oboru, termíny, požadavky).
  - Rozšířit o RAG dotazy do našeho indexu a citace zdrojů.
- **Rizika:** nutnost přepsat UI a datové struktury; kontrolovat konzistenci s dlouhodobou pamětí.

### `claude-cookbooks/skills/retrieval_augmented_generation` a `contextual-embeddings`
- **Obsah:** kompletní RAG tutorial, evaluation skripty (Promptfoo), contextual embeddings pro přesnější retrieval.
- **Proveditelnost:** vysoká; přímo řeší potřebu RAG nad oficiálními weby škol.
- **Úpravy:**
  - Nahradit sample dataset daty z `vs-weby.txt` (po offline scrapingu).
  - Přizpůsobit evaluation pipeline na české dotazy; přidat testy pro SK.
  - Případně přepsat na Azure AI Search API (aktuálně Python knihovny pro open-source vektor DB).
- **Rizika:** promptfoo eval potřebuje nastavit Anthropic klíč; nutno ošetřit licenční omezení dat.

### `claude-cookbooks/skills/text_to_sql`
- **Obsah:** generování SQL dotazů, self-improvement a RAG pro strukturované dotazy.
- **Proveditelnost:** střední; lze využít pro dotazy typu "najdi školy s oborem X a požadavkem Y" nad relační DB.
- **Úpravy:**
  - Změna schématu na naše tabulky (programy, fakulty, termíny).
  - Lokalizace promptů (čeština/slovenština) a přidání post-validace (např. limit výsledků).
- **Rizika:** riziko generování neoptimálních dotazů → potřebná validační vrstva.

### `claude-cookbooks/skills/summarization`
- **Obsah:** techniky multi-dokumentové sumarizace.
- **Proveditelnost:** vysoká; vhodné pro generování shrnutí oborových profilů či follow-up emailů.
- **Úpravy:** jen lokalizace promptů a integrace s RAG výsledky.
- **Rizika:** kontrola halucinací – nutné používat citace z RAG.

### `claude-cookbooks/observability` (pokud bude potřeba)
- **Obsah:** logování, metriky, tracing.
- **Proveditelnost:** střední; možno napojit na GA4/App Insights.
- **Úpravy:** adaptovat exporty na Azure Monitor, definovat vlastní metriky (latence, citace, odchody).
- **Rizika:** propojení s existující telemetrií může vyžadovat custom adaptér.

### `claude-quickstarts/agents`
- **Obsah:** minimální implementace agentní smyčky v Pythonu (<300 řádků), integrace MCP.
- **Proveditelnost:** vysoká; vhodný start pro backend orchestraci nebo proof-of-concept.
- **Úpravy:**
  - Doplnit orchestraci dle scénářů VSChatBot (router + specialista).
  - Přidat naše MCP servery (DB, RAG vyhledávač).
  - Zapojit caching (prompt/context) dle požadavků.
- **Rizika:** chybí produkční featury (retry, telemetry), nutno doplnit.

### `claude-quickstarts/customer-support-agent`
- **Obsah:** Next.js UI s realtime myšlením, integrace na Amazon Bedrock knowledge bases, detekce nálady.
- **Proveditelnost:** střední; UI se hodí jako referenční design pro web chat (ukazuje citace a zdroje).
- **Úpravy:**
  - Přepojit z Bedrock na náš RAG (Azure AI Search / vlastní index).
  - Lokalizovat UI (čeština/slovenština) a přizpůsobit styl brandu.
  - Vyměnit mood detection za relevantní metriky (např. nespokojenost → ruční zásah poradce).
- **Rizika:** AWS specifické komponenty; je nutné je nahradit Azure analogií nebo udržovat multi-cloud.

### `claude-quickstarts/financial-data-analyst`
- **Obsah:** Next.js aplikace pro analýzu tabulkových dat, generování grafů, soubory upload.
- **Proveditelnost:** střední; užitečné pro modul vizualizací (např. porovnání škol/oborů).
- **Úpravy:**
  - Upravovat datový model na "studium" (např. dataset statistik přihlášek, úspěšnosti).
  - Omezit upload na interní zdroje nebo transformovat na precomputed grafy.
- **Rizika:** nepotřebujeme plnou funkcionalitu; vývoj by mohl být nadbytečný, pokud se zaměříme pouze na chat.

### `claude-quickstarts/computer-use-demo`
- **Obsah:** agent s ovládáním počítače přes Docker (beta funkce).
- **Proveditelnost:** nízká pro produkční chatbot; spíše experiment.
- **Možné využití:** offline skript pro kontrolu aktualizací webů (automatizované procházení stránek). Vyžaduje sandbox.
- **Rizika:** vysoká komplexita, bezpečnostní omezení.

## Integrace do návrhových variant (z `codex-vschatbot-solutions.md`)
- **MVP:** převzít `claude-quickstarts/agents` jako základ agentní smyčky + `skills/retrieval_augmented_generation` pro RAG; paměť pouze lehká (shrnutí) podle `tool_use/memory_cookbook`.
- **Scénářový konzultant:** využít orchestrátor-worker z `patterns/agents` + contextual embeddings pro přesnější dotazy; evaluace s Promptfoo.
- **Multiagent studio:** adaptovat evaluator-optimizer workflow k průběžné kontrole kvality; memory cookbook → centralizované profily; `customer-support-agent` UI pro vizualizaci interakcí a citací.
- **Enterprise++:** doplnit observability a extended thinking (pokročilé reasoning prompt šablony) pro audit, případně zvážit computer-use demo pro automatizované datové sběry.

## Doporučené kroky
1. **Proof-of-concept (2–3 týdny):** přenést `claude-quickstarts/agents` + RAG skills na naše data (minimální funkční chatbot) a ověřit kvalitu v češtině.
2. **Scénářový modul (1–2 měsíce):** implementovat orchestrátor z `patterns/agents`, navázat MCP nástroje a dlouhodobou paměť podle memory cookbook.
3. **Evaluace a telemetry (současně):** integrovat Promptfoo pipeline a observability pattern; sbírat feedback → validovat proti požadavkům.
4. **UI a citace:** adaptovat `customer-support-agent` front-end pro prezentaci citací a zpětné vazby.
5. **Rozšíření:** podle potřeby doplnit text-to-SQL modul a datové vizualizace (financial analyst), případně automatizovaný scraping.

## Identifikovaná rizika a mitigace
- **Jazyková lokalizace:** prompt engineering pro češtinu/SK → nutné testy a ruční review.
- **Licenční a přístupová omezení:** některé příklady vyžadují Anthropic API; při použití OpenAI modelů je nutný přepis knihoven.
- **Komplexita orchestrace:** notebooková forma vyžaduje refaktor do modulárních služeb; doporučeno vytvořit vlastní Python balíček s testy.
- **Bezpečnost a PII:** memory cookbook ukládá uživatelská data; nutná integrace s GDPR procesy (opt-in, anonymizace).

Tento dokument slouží jako podklad pro rozhodnutí, které komponenty z Claude ekosystému adaptovat v jednotlivých fázích vývoje VSChatBotu.
