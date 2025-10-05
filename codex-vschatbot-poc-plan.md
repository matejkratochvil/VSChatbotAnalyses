# PoC plán pro VSChatBot – repoužití Claude vzorů

## 1. Klíčové komponenty PoC
- **Agentní smyčka a orchestrátor:** Start z `claude-quickstarts/agents` (agent/tool loop) s rozšířením o routing dle scénářů (triáž → specialista).
- **RAG vrstva s citacemi:** Vektorový index + contextual embeddings podle `claude-cookbooks/skills/retrieval_augmented_generation` a `contextual-embeddings`; využít metadata pro citace.
- **Paměť a profily uživatelů:** Shrnutí a dlouhodobé preference z `claude-cookbooks/tool_use/memory_cookbook`/`memory_tool.py`, napojení na naši DB přes MCP.
- **Zpětná vazba a evaluace:** Promptfoo evaluace z `skills/retrieval_augmented_generation/evaluation`, plus ruční kontrola citací.
- **Základní UI/telemetrie:** Převzít klíčové prvky z `claude-quickstarts/customer-support-agent` (zobrazení zdrojů) jen jako referenci pro integrační tým.

## 2. Mapování na zdrojové příklady
| Požadovaná schopnost | Zdroj | Co převzít | Úpravy |
| --- | --- | --- | --- |
| Agentní smyčka | `claude-quickstarts/agents/agent.py` | Základní loop, integrace MCP | Lokalizace promptů, doplnění routeru, logging |
| RAG | `skills/retrieval_augmented_generation/guide.ipynb`, `contextual-embeddings/guide.ipynb` | Pipeline embeddings + retrieval, evaluace | Předělat na Azure AI Search/PostgreSQL, čeština/SK dataset |
| Citace | Stejné RAG notebooky + `customer-support-agent` UI | Formátování odpovědí se zdroji | JSON výstup + front-end anotace |
| Paměť | `tool_use/memory_cookbook.ipynb`, `memory_tool.py` | Shrnovací vzory, API pro memory tool | Ochrana PII, per-user klíče, integrace s DB |
| Evaluace | `skills/retrieval_augmented_generation/evaluation` | Promptfoo konfigurace | Lokální dataset, OpenAI/Claude přepínač |
| Caching | `tool_use/parallel_tools_claude_3_7_sonnet.ipynb` (context reuse), Anthropic docs | Praktiky pro paralelu a opětovné využití promptů | Zpřístupnit přes naše orchestrátory |

## 3. Backlog PoC úkolů
1. **Inicializace repo a infrastruktury** – vytvořit PoC modul, připravit prostředí (Python 3.10, Anthropic/OpenAI SDK, Azure přístup). *Odhad: 2 dny.*
2. **Portace agentní smyčky** – přenést `agent.py`, přidat scénářový router, logging. *3 dny.*
3. **MCP vrstva pro DB a paměť** – vytvořit prototyp MCP serveru nad PostgreSQL a pamětí, zajistit autentizaci. *4 dny.*
4. **RAG pipeline na českých datech** – ingestovat sample z `vs-weby`, vytvořit embeddings/index, vracet citace. *5 dny.*
5. **Paměťová integrace** – implementovat krátkodobé shrnutí + dlouhodobý profil per uživatel. *4 dny.*
6. **Prompt/context caching** – nakonfigurovat caching pro systémový prompt a shrnutí konverzace. *2 dny.*
7. **Evaluace kvality** – nastavit Promptfoo scénáře, spustit testy, dokumentovat výsledky. *3 dny.*
8. **Frontend prototyp** – jednoduchý Next.js/React chat zobrazující citace a sběr zpětné vazby. *4 dny.*
9. **Dokumentace a handover** – popsat konfiguraci, omezení, další kroky. *2 dny.*

Rámcový čas PoC: ~29 člověkodnů (cca 6 týdnů dvoučlenného týmu). Úkoly 2–5 jsou kritické, zbytek lze paralelizovat po zprovoznění základů.
