# AI Chatbot Solution Notes (Derivované ze zdrojů OpenAI Cookbook examples)

## 1. Přehled klíčových přístupů z cookbook (mapování na funkční požadavky)

Níže uvádím strukturované shrnutí relevantních metod z /openai-cookbook/examples (jen názvy + odvozené principy; samotné notebooky nejsou kopírovány). Každý blok obsahuje: Co řeší, Jak aplikovat v našem projektu, Na co si dát pozor, Skalování / optimalizace.

### 1.1 Základy práce s modely
- How_to_format_inputs_to_ChatGPT_models.ipynb / How_to_stream_completions.ipynb
  - Principy: systémové instrukce, role-based prompting, streaming pro nízkou latenci, explicitní output kontrola.
  - Aplikace: jednotný systémový prompt definující roli kariérního poradce + modulární injekce: (User Profile Summary + Context + Retrieval Evidence + Conversation Snippet Window).
  - Pozor: Nepřekrmovat systémový prompt statickými daty – místo toho referencovat externí paměť.
  - Optimalizace: Streaming + krátká konverzační okna + souhrn starších částí.

- How_to_count_tokens_with_tiktoken.ipynb / Prompt_Caching101.ipynb
  - Princip: Měření a optimalizace promptů; využití caching identických prefixů (OpenAI cache / vlastní hash layer).
  - Aplikace: Cache systémový prompt + uživatelský profil + scénářové instrukce.
  - Optimalizace: Vytvářet deterministicky stejné bloky = nižší variabilní náklady.

### 1.2 Funkce a nástroje (Tool / Function Calling)
- How_to_call_functions_with_chat_models.ipynb / Function_calling_finding_nearby_places.ipynb / Function_calling_with_an_OpenAPI_spec.ipynb / Using_tool_required_for_customer_service.ipynb
  - Princip: Model navrhuje volání funkcí dle schématu; funkce vrací strukturovaná data → vloží se zpět do kontextu.
  - Aplikace: Funkce: get_university(id), search_universities(filters), get_open_day_dates(faculty_id), get_admission_requirements(program_id), list_recommended_programs(user_profile_state), store_long_term_memory(event), retrieve_memory(query, scope), web_search(query) (volitelně), feedback_submit(conversation_id,rating,comment).
  - Pozor: Validovat parametry před exekucí (guardrail) + omezit fallback generativních odpovědí bez datových zdrojů.
  - Optimalizace: Po funkčním volání zestručnit výsledky (abstrakce) před předáním modelu.

- Structured_Outputs_Intro.ipynb / Structured_outputs_multi_agent.ipynb
  - Princip: Model generuje JSON dle definice → robustnější parsing.
  - Aplikace: Strukturované výstupy pro: (a) detekci intentu, (b) extrakci filtrů vyhledávání oborů, (c) generování mezi-souhrnu uživatele, (d) scénářové kroky.
  - Optimalizace: Definovat minimální schema, povinná pole, validace JSON → fallback re-run.

### 1.3 RAG (Retrieval Augmented Generation)
- Question_answering_using_embeddings.ipynb / Semantic_text_search_using_embeddings.ipynb / Using_embeddings.ipynb / Embedding_Wikipedia_articles_for_search.ipynb / Parse_PDF_docs_for_RAG.ipynb / RAG_with_graph_db.ipynb / Search_reranking_with_cross-encoders.ipynb / File_Search_Responses.ipynb
  - Princip: Vícekrokové: (1) Chunking → (2) Embedding store → (3) Retrieval (kNN) → (4) (Volitelně) rerank → (5) Kompozice promptu s evidence bloky.
  - Aplikace: Datové vrstvy: (a) Strukturovaná DB (fakulty, obory, termíny), (b) Textové dokumenty (popisy oborů, PDF), (c) Long-term user memory (vektor + symbolická). Při dotazu: Hybrid retrieval: BM25 (structured meta) + embedding kNN + rerank (volitelně cross-encoder nebo GPT krátké rerank volání) + deduplikace.
  - Graph DB (volitelné vyšší úroveň) pro vztahy (Univerzita→Fakulta→Program; Program→Požadavky; Program→OpenDay events). Může pomoci kontextovým dotazům typu: „Jak se liší X a Y z hlediska přijímaček?“ → extrahovat atributy a agregovat.
  - Optimalizace: Před vložením do promptu shrnout chunk na 1–2 věty + uvést citaci (ID / URL) pro transparentnost.

### 1.4 Práce s dlouhými vstupy a sumarizace
- Summarizing_long_documents.ipynb / Embedding_long_inputs.ipynb / Entity_extraction_for_long_documents.ipynb / Context_summarization_with_realtime_api.ipynb
  - Princip: Hierarchická sumarizace, extrakce entit, selektivní kontext.
  - Aplikace: Uživatelský profil = kontinuální souhrn (rolling summary) + klíčové preference (entities: oborové zájmy, lokace, finanční preference, studijní styl).
  - Optimalizace: Po každých ~6–8 uživatelských zprávách spustit summarizer funkci (levnější model) → aktualizovat memory_summary.

### 1.5 Klasifikace, extrakce, zlepšení přesnosti
- Classification_using_embeddings.ipynb / Zero-shot_classification_with_embeddings.ipynb / Named_Entity_Recognition_to_enrich_text.ipynb / Fine_tuning_for_function_calling.ipynb / Fine_tuned_classification.ipynb
  - Princip: Lehká inferenční pipeline před hlavním modelem (intent detection, scénář routing, extrakce filtrů).
  - Aplikace: Intent classes: {decision_guidance, factual_query, compare_programs, admission_requirements, open_day_info, feedback_phase, chit_chat, out_of_scope}.
  - Optimalizace: Nejprve embedding+cosine (rychlé), u hraničních případů fallback na model structured output.

### 1.6 Guardrails a kvalita
- Developing_hallucination_guardrails.ipynb / How_to_use_guardrails.ipynb / How_to_use_moderation.ipynb / Reproducible_outputs_with_the_seed_parameter.ipynb / Using_logprobs.ipynb
  - Princip: Vícevrstvý filtr (moderation → retrieval presence check → hallu detector → final answer).
  - Aplikace: Pokud odpověď obsahuje fakty bez citací nebo chybí retrieval evidence u dotazů typu factual_query, přegeneruj s instrukcí “attach_sources”.
  - Optimalizace: Logprobs + heuristika (dlouhé enumerace bez zdrojů) → flag.

### 1.7 Orchestrace a multi-agent přístupy
- Orchestrating_agents.ipynb / Structured_outputs_multi_agent.ipynb / How_to_build_a_tool-using_agent_with_Langchain.ipynb / object_oriented_agentic_approach
  - Princip: Router agent → Specialist agents (retriever, comparator, scenario coach, feedback collector, web_search).
  - Aplikace (pokročilý tier): Koordinátor volá funkční rozhraní; každý agent má striktní system prompt + limit tokenů.
  - Optimalizace: Minimalizovat chain-of-thought leak (nastavit reasoning budget / use reasoning model jen u komplexních srovnání).

### 1.8 Rate limiting a robustnost
- How_to_handle_rate_limits.ipynb / api_request_parallel_processor.py / batch_processing.ipynb
  - Princip: Exponenciální backoff, lokální fronta, priorita uživatelských realtime dotazů > asynchronní reindex.
  - Aplikace: Worker pro precomputing embeddings (cron) oddělen od inference API.

### 1.9 Analytika a evaluace
- Custom-LLM-as-a-Judge.ipynb / Unit_test_writing_using_a_multi-step_prompt.ipynb / Leveraging_model_distillation_to_fine-tune_a_model.ipynb
  - Princip: Automatizované skórování kvality odpovědí (relevance, factual grounding, helpfulness) na vzorku.
  - Aplikace: Týdenní batch eval: generuj test dotazy + known truth slices (DB). LLM judge (levnější model + rubric) → dashboard.

### 1.10 Multimodální / volitelné
- Tag_caption_images_with_GPT4V.ipynb / GPT_with_vision_for_video_understanding.ipynb (zde spíš budoucí rozšíření: např. analýza fotky kampusu – nízká priorita R1).

---
## 2. Návrhové vrstvy řešení

1. Frontend chat widget (CZ/SK autodetekce) + GA4 eventy.
2. API Gateway (Rate limit, Auth, Usage quota per user, Spend guard).
3. Conversation Orchestrator (Router: intent → pipeline). Modulární.
4. Retrieval Layer:
   - a) Structured DB (PostgreSQL / Azure SQL)
   - b) Vector Store (Azure AI Search / PostgreSQL pgvector / Qdrant)
   - c) (Tier Pro) Graph DB (Neo4j / Azure Cosmos DB Gremlin) pro komplexní vztahy.
5. Memory Layer:
   - Short-term: posledních N zpráv (window)
   - Mid-term: rolling summary (aktualizováno periodicky)
   - Long-term: vektorové embeddingy user events + klíčové preference tabulka
   - Scenario state: JSON (active_scenario, step_index, collected_slots).
6. Tools/Functions Layer (stateless wrappers) + Validation + Caching.
7. Guardrails Layer (moderation → intent policy → retrieval enforcement → hallucination check → final answer tagging sources).
8. Analytics & Feedback (store rating, auto-eval pipeline, token usage logs → cost dashboard).
9. Offline Workers (embedding ingestion, periodic re-summarization, evaluation batches, index refresh).

---
## 3. Evoluční roadmapa variant (od MVP po enterprise)

Varianty definované stupni zralosti. U každé: přístup, pokrytí požadavků, technologie, kdy upgradovat.

### 3.1 MVP (Level 0) – Nejjednodušší životaschopná verze
- Co je to: Jednoagentní chat bez skutečné dlouhodobé paměti; pouze kontextové okno + jednoduché dotazy do DB přes předdefinované funkce; základní retrieval (přímo SQL + fulltext/LIKE); žádný vektorový store.
- Funkční pokrytí: Rychlé odpovědi + základ scénářů (hardcoded flow skripty). Jazyková detekce na první zprávě. Zpětná vazba ukládána.
- Chybí: Personalizovaná dlouhodobá paměť, sofistikovaný RAG, guardrails na halucinace.
- Technologie: GPT-4o mini / Claude Haiku (levné), Azure Functions pro API, PostgreSQL.
- Kdy přejít dále: Jakmile požadujeme doporučování dle historie nebo robustní faktické odpovědi.

### 3.2 Level 1 – Základní RAG + Paměť
- Přidáno: Vektorový index oborových textů + user memory summary + embedding-based intent classification.
- Funkce: Lepší faktické odpovědi, personalizovaná doporučení (scéna: shrnutí preferencí → query embeddings).
- Technologie: text-embedding-3-small (levné), pgvector / Azure AI Search.

### 3.3 Level 2 – Scénářový orchestrátor + Guardrails
- Přidáno: Router (intent → {scenario_engine, factual_engine, compare_engine}); Hallucination gating; citace zdrojů; automatická sumarizace historie.
- Technologie: Strukturované výstupy (JSON schema), moderace API, heuristický hallu detektor.

### 3.4 Level 3 – Multi-agent modularita
- Přidáno: Specialist agents (ScenarioCoach, Retriever, Comparator, ProfileBuilder, FeedbackEvaluator). Každý limitovaný token budgetem. Plánování kroků + fallback strategie.
- Přínos: Vyšší přesnost v komplexních rozhodovacích dialozích (např. porovnej 3 programy v 5 dimenzích).

### 3.5 Level 4 – Rozšířená znalostní síť + Graph + Reranking
- Přidáno: Graph traversal pro vztahové dotazy; cross-encoder reranking pro navýšení relevance topK; atributové porovnání generované z normalizovaných polí.

### 3.6 Level 5 – Enterprise / Profesionální systém
- Přidáno: Multi-tenant governance, Hard spend guard, Prompt cache layer, Offline evaluation harness (LLM judge + drift detection), A/B test orchestrátor, Web search plugin (omezené domény whitelisted), reasoning model pro komplexní případy (on-demand), adaptive cost policy (levný model default, upgrade dle complexity heuristiky), automatická detekce datových mezer (unknown queries → ingestion backlog), MCP servery pro: (a) curated web scrape endpoints (oficiální weby), (b) registry státních dat, (c) PDF ingestion.
- Možný další stupeň: reinforcement fine tuning z reálné interakce (dlouhodobě, citlivě na etiku a bias).

---
## 4. Implementační kroky (step-by-step) – doporučená sekvence

### Fáze A: MVP (Týdny 1–4)
1. Specifikace schémat DB (universities, faculties, programs, events, admissions, user_profiles, conversations, feedback).
2. API kontrakt (REST/JSON) + Auth (JWT / session token).
3. Jednoduché funkce: search_programs(text), get_program(id), get_open_days(faculty_id), store_feedback().
4. System prompt + jazyková detekce (heuristika: první user msg → detect via model / simple langid lib).
5. Konverzační okno (posledních 10 zpráv); streaming odpovědí.
6. Logging token usage.
7. GA4 eventy (chat_start, chat_end, feedback_submitted).
8. Základ scénářů: Script table (scenario_id, step_index, prompt, slot_keys) + engine (lineární postup / přeskok při slotu vyplněném).

### Fáze B: RAG + Paměť (Týdny 5–8)
1. Extrakce textů (popisy oborů) → chunk (max 800 znaků), ukládání metadata.
2. Embedding ingest (batch worker) → vector store.
3. Retrieval pipeline (kNN top 8) → kontextový blok (citace + zhuštění).
4. User memory summary: funkce update_user_summary(conversation_slice) každých X zpráv.
5. Intent klasifikace (embedding nearest centroid) → fallback model JSON extrakce.
6. Přidat preference extraction (entities: location, field_interest, study_mode, cost_sensitivity).

### Fáze C: Guardrails + Vylepšené scénáře (Týdny 9–12)
1. Moderation API pre-check + post-check.
2. Hallucination check: if factual intent & no sources referenced → regenerate.
3. Structured outputs: definice JSON schema pro compare_request, scenario_progress.
4. Scenario adaptivity: dynamické větvení podle odpovědí (slot logic rules table).
5. Token optimalizace: prefix caching; summarization starších zpráv.

### Fáze D: Multi-agent a Reranking (Týdny 13–18)
1. Router agent (intent → plan) – structured output.
2. Specialist agents (RetrieverAgent, CompareAgent, ScenarioCoachAgent, ProfileAgent).
3. Reranking (volitelně cross-encoder / light model) → refine topK.
4. Evaluation harness (LLM judge + rubric: helpfulness, groundedness, clarity, personalization) – weekly batch.

### Fáze E: Enterprise rozšíření (Týdny 19–26)
1. Graph DB ingest + generování relationship edges.
2. Adaptive model policy (light vs advanced reasoning) – heuristika complexity score (#constraints + comparative depth).
3. Web search MCP (whitelist domains; caching fetched pages; TTL invalidation).
4. Spend guard (per user + global monthly) + soft warning, hard cut.
5. Prompt variant A/B test (tracking success via feedback + retention).
6. Drift detection: cluster novel queries → ingestion backlog.

---
## 5. Funkční požadavky – způsob implementace
(Mapování z AI_chat_specifikace.md)
1. Dlouhodobá paměť: Rolling summary + vector log of salient events + structured profile table. Aktualizace batch nebo event-driven. Retrieval prior to each answer: (a) summary, (b) top 3 memory events relevant (embedding similarity query on memory store).
2. Dynamické scénáře: Scenario engine (state machine). Tabulky: scenario_steps (conditions, next_step_if, prompt_template, required_slots). Data-driven přidávání scénářů.
3. Rychlost: Streaming + minimal context assembly + caching + short model default (upgrade only when needed for complex comparison). Parallel prefetch: while user čte odpověď, připrav další retrieval.
4. Web Search: Volitelný modul – spustit pouze při: (intent in {open_day_not_found, admission_gap, unknown_program}). Guard: domain allowlist + summarizer před vložením do promptu.
5. Jazyková podpora: detect_lang(user_first_msg) → set conversation.language; system prompt variant; user memory tagged s language; model odpovídá kontextově.
6. Infrastruktura: Azure App Service / Functions (API), Azure PostgreSQL, Azure OpenAI (nebo OpenAI public), Azure Cognitive Search (alternativa). Containerization (Bicep/Terraform skript později).
7. Zpětná vazba: Po X interakcích nebo explicitní ukončení – prompt “Ohodnoť užitečnost 1–5”. Uložit + volitelné textové vysvětlení.
8. Prioritizace zdrojů: Order: (a) DB structured, (b) vector docs, (c) memory, (d) web search fallback. V promptu explicitně vyjmenovat zdroje.
9. Optimalizace nákladů: viz Sekce 7 (token model); caching prefix, summarization, adaptive model selection.
10. Analytika: GA4 event taxonomy + interní metrics DB (table: analytics_events). Týdenní agregace.

---
## 6. Token & Cost odhady (scénáře)
Předpoklady (laditelné):
- Průměrná konverzace (kariérní guidance) 12 uživatelských zpráv, 12 odpovědí.
- Průměr input tokens / user msg (po úpravách) ~160 (uživatel 30–50, plus kontext memory ~110). Output ~220 tokens.
- Na 1 konverzaci: Input ≈ 12 * 160 = 1,920 tokens; Output ≈ 12 * 220 = 2,640 tokens → Celkem ~4,560 tokens.
- Měsíc na uživatele: průměr 2.2 konverzace (návrat) → ~10,000 tokens.
- Long-term memory ingest: každá konverzace 4 summarizační volání (levný model) po 300 input + 120 output = 420 * 4 = 1,680 tokens navíc (~+17%).
- Celkem / uživatele / měsíc ≈ 11,700 tokens (zaokrouhleno 12k) base case.

Modelové sazby (hypoteticky – aktualizovat dle aktuálních ceníků):
- GPT-4o mini: $0.15 / 1M input, $0.60 / 1M output.
- GPT-4o (standard): $2.50 / 1M input, $10 / 1M output.
- Embeddings (text-embedding-3-small): $0.02 / 1M tokens.
- Rerank / cross-encoder (pokud open-source hostovaný) ~ fixní infra.

Varianty nákladů / uživatele / měsíc (12k tokens):
A) 100% GPT-4o mini (40% input, 60% output poměr):
- Input: 4,800 tokens → 0.0048 * $0.15 ≈ $0.00072
- Output: 7,200 tokens → 0.0072 * $0.60 ≈ $0.00432
- Memory summarization (17% navíc totéž poměrově): ≈ $0.00085
- Embeddings (předpoklad 30 embedding volání * 512 tokens avg = 15,360 tokens): 0.01536 * $0.02 ≈ $0.000307
- Celkem ≈ $0.0062 / uživatele / měsíc (velmi nízké – validovat sazby!)

B) Mix: 80% mini, 20% GPT-4o (komplexnější odpovědi)
- Navýšení ~ ×3–4 → ≈ $0.02–0.025 / uživatele / měsíc.

C) High quality (100% GPT-4o) ~ ~$0.08–0.10 / uživatele / měsíc.

1 000 uživatelů (Level 2–3 mix):
- Variabilní LLM spend: ~ $20–30 / měsíc (rezerva ×2 = $60).
- Embedding reindex měsíčně (přidání nových dokumentů 200k tokens): 0.2 * $0.02 = $0.004 → zanedbatelné.
- Infra fixní: DB ($50), Vector store ($50), App hosting ($80), Monitoring+Logging ($40), Storage ($20) → ~ $240.
- Celkem s rezervou: $300 měsíčně (bez práce lidí).

Long-term memory růst: pokud průměrně 1k uživatelů × 12k tokens memory store = 12M tokens / rok → embedding budget ~ $0.24 / rok (levné). Hlavní driver je generace, ne embeddings.

Scaling k 2k uživatelům: LLM cost ~ $60; infra navýšení (větší DB tier) +50%. Stále řádově stovky USD.

---
## 7. Cost optimalizační techniky
1. Prompt caching (fixní prefix + scenario instructions hash).
2. Adaptive model selection (complexity classifier → flag high_cognitive_query True → use GPT-4o; jinak mini model).
3. Kontextové ořezávání: pouze posledních 4–6 výměn + summary.
4. Retrieval minimalizace: topK=5 baseline; rerank jen při comparative intent.
5. Memory densification: staré eventy slučovat do meta-summary (každých 10 konverzací).
6. Batch embeddings ingestion off-peak.
7. Citace = kratké ID reference, ne celý dokument → snížení tokenů.

---
## 8. Odhad vývoje (pracnost)
Hrubé MD (man-days) pro senior+mid tým (bez velké rezervy):
- Fáze A (MVP): 20–25 MD (backend 12, frontend 5, infra 3, PM/QA 3)
- Fáze B: 18–22 MD
- Fáze C: 20–24 MD
- Fáze D: 25–30 MD
- Fáze E: 30–40 MD
Celkem full roadmap: ~113–141 MD. Při 2 vývojářích (velká část paralelizovatelná) ~3.5–4.5 měsíce čistého času (+ testování + ladění) – v souladu s cílem do ledna 2026 (máme rezervu).

Rough náklad (interní účtovaná sazba hypoteticky 12 000 CZK / MD):
- MVP ~ 300k CZK
- Do Level 3 (A+B+C) ~ 300k + 220k + 240k ≈ 760k CZK
- Plný enterprise (A–E) ~ 1.4–1.6M CZK

---
## 9. Rizika a mitigace
1. Hallucinace: striktní retrieval enforcement; answer must cite >=1 source for factual intents.
2. Datová neúplnost: Unknown intent → log gap → manuální ingest pipeline.
3. Nákladový drift: měsíční cost dashboard; threshold alert.
4. Latence: Pre-warm model sessions; paralelní retrieval a intent klasifikace.
5. Jazyková kvalita (SK): test evaluace s nativními studenty + per-language fine instructions.
6. Scénářová rozšiřitelnost: Data-driven definice kroků (tabulky) – žádné hard-coded větvení v kódu.

---
## 10. Vzor architektonického request flow
User msg → (Lang detect) → Moderation → Intent classify (embedding) → If scenario_active: scenario_engine.step() gather needed slots → Retrieval (structured + vector + memory) → Compose prompt (system + summary + evidence + question) → Model call (adaptive model) → Hallu check → Post-process (citace formatting) → Store message + update memory queue → Return streaming tokens → After conversation end: solicit feedback.

---
## 11. Příklady modulárních komponent (inspirováno cookbook)
- intent_classifier.py: embedding centroid + threshold.
- memory_manager.py: add_event(), update_summary(), retrieve_relevant_events(query_text).
- retrieval_pipeline.py: build_query(filters) + vector_search(embedding) + merge_rank().
- scenario_engine.py: load_state(user_id) + apply_step(user_input) returns (prompt_addition, pending_slots).
- guardrails.py: run_moderation(), enforce_citations(answer, evidence_refs), hallucination_scan().
- cost_tracker.py: log_tokens(conversation_id, model, in_tokens, out_tokens).

---
## 12. Další možná budoucí vylepšení
- Fine-tuning malé varianty modelu pro stabilní scénářové odpovědi.
- Preference modeling (latent factors) → pokročilé doporučení (embedding space clustering programů + user vector).
- Realtime audio chat (Speech_transcription_methods.ipynb + Whisper) pro voice asistenci.
- Offline školní poradce mód (generace PDF shrnutí doporučení oborů + tabulkové porovnání).

---
## 13. Shrnutí
Navržený postup umožňuje inkrementálně přejít z minimálního chatbota na robustní multi-agent RAG systém s nízkými náklady. Přidaná hodnota roste hlavně v přesnosti doporučení, personalizaci a důvěryhodnosti (citace). Náklady na provoz při 1 000 uživatelích jsou relativně nízké; klíčová investice je ve vývoji a kuraci dat. Doporučuji začít MVP rychle, brzy nasbírat reálné konverzace pro iteraci intent a scénářů, následně přidat RAG a guardrails, poté teprve multi-agent orchestraci.

---
(End of document)
