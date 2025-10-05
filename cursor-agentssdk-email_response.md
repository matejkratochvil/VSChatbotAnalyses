# Odpověď na otázky z emailu

## Analýza byznysových aspektů

### Konkurence a výhody

**Konkurence:**
- Klasické chatboty (ChatGPT, Claude) - kvalitní a zdarma
- Web search schopnosti
- Rychlé odpovědi na obecné otázky

**Naše výhody:**
- **Specializované znalosti** - hluboké znalosti o českých/slovenských univerzitách
- **Personalizace** - dlouhodobá paměť a preference uživatelů
- **Strukturované scénáře** - připravené kariérové poradenství
- **Lokální kontext** - specifické informace o českém vzdělávacím systému

### Doporučení pro MVP

**Ano, doporučuji začít s MVP** z následujících důvodů:

1. **Rychlá validace konceptu** - testování zájmu uživatelů
2. **Nízké počáteční náklady** - $17,000-23,000/měsíc pro 1,000 uživatelů
3. **Rychlé nasazení** - 2-3 měsíce vývoje
4. **Postupná evoluce** - možnost přidávání funkcí podle potřeby

## Odpovědi na konkrétní body

### 1. Dlouhodobá paměť

**MVP přístup:**
- Cosmos DB pro ukládání konverzací
- Základní profilování uživatelů
- Shrnování starších konverzací pro úsporu tokenů

**Implementace:**
```python
# Uložení konverzace
conversation = {
    "user_id": user_id,
    "timestamp": datetime.utcnow(),
    "messages": messages,
    "user_preferences": extracted_preferences
}
cosmos_db.upsert_item(conversation)
```

### 2. Dynamické scénáře

**MVP řešení:**
- 3 statické scénáře v promptu
- Jednoduché rozhodovací stromy
- Možnost rozšíření o další scénáře

**Příklad implementace:**
```python
SCENARIOS = {
    "career_choice": {
        "questions": ["Jaké jsou vaše zájmy?", "Jaké předměty vás baví?"],
        "logic": "if interests == 'tech': suggest_computer_science()"
    },
    "university_selection": {
        "questions": ["Kde chcete studovat?", "Jaká je vaše finanční situace?"],
        "logic": "if location == 'prague': suggest_prague_universities()"
    }
}
```

### 3. Web Search

**MVP:**
- Function calling pro specifické dotazy
- Omezené na oficiální zdroje univerzit
- Fallback na databázi

**Implementace:**
```python
@function_tool
def search_university_info(query: str) -> str:
    """Vyhledání informací o univerzitě na webu"""
    # Search only official university websites
    official_sources = get_official_university_sources()
    results = search_web(query, sources=official_sources)
    return format_search_results(results)
```

### 4. CZ/SK podpora

**Implementace:**
```python
def detect_language(text: str) -> str:
    """Detekce jazyka uživatele"""
    # Simple language detection
    if any(word in text.lower() for word in czech_words):
        return "cs"
    elif any(word in text.lower() for word in slovak_words):
        return "sk"
    return "cs"  # default

def get_localized_response(response: str, language: str) -> str:
    """Lokalizace odpovědi"""
    if language == "sk":
        return translate_to_slovak(response)
    return response  # Czech is default
```

### 5. Zpětná vazba

**MVP implementace:**
```python
def request_feedback(conversation_id: str):
    """Požádání o zpětnou vazbu"""
    feedback_prompt = """
    Byla tato konverzace užitečná? 
    [1] Velmi užitečná
    [2] Užitečná  
    [3] Neutrální
    [4] Neužitečná
    [5] Velmi neužitečná
    """
    return feedback_prompt

def track_feedback(conversation_id: str, rating: int):
    """Sledování zpětné vazby"""
    analytics.track("feedback_received", {
        "conversation_id": conversation_id,
        "rating": rating,
        "timestamp": datetime.utcnow()
    })
```

### 6. LLM a spotřeba tokenů

**Optimalizace nákladů:**
- Prompt caching pro opakující se dotazy
- Context summarization pro dlouhé konverzace
- Použití GPT-4o-mini pro jednoduché úkoly
- Monitoring a alerting při překročení limitů

**Implementace:**
```python
class TokenOptimizer:
    def __init__(self):
        self.max_tokens_per_conversation = 8000
        self.cache_threshold = 0.8
    
    def optimize_conversation(self, messages: list) -> list:
        """Optimalizace konverzace pro úsporu tokenů"""
        total_tokens = self.count_tokens(messages)
        
        if total_tokens > self.max_tokens_per_conversation:
            # Summarize older messages
            messages = self.summarize_old_messages(messages)
        
        return messages
```

### 7. Analytika

**GA4 integrace:**
```python
def track_user_interaction(event_name: str, parameters: dict):
    """Sledování uživatelských interakcí"""
    gtag('event', event_name, {
        'conversation_id': parameters.get('conversation_id'),
        'user_id': parameters.get('user_id'),
        'scenario_type': parameters.get('scenario_type'),
        'completion_rate': parameters.get('completion_rate')
    })

# Klíčové metriky
METRICS = {
    'conversion_rate': 'conversations_started / conversations_completed',
    'success_rate': 'positive_feedback / total_feedback',
    'average_session_length': 'total_tokens / conversations',
    'scenario_completion': 'scenarios_completed / scenarios_started'
}
```

## Nákladová analýza

### MVP řešení (1,000 uživatelů)

**Měsíční náklady:**
- OpenAI API: $15,000-20,000
- Azure infrastruktura: $2,000-3,000
- **Celkem: $17,000-23,000**

**Roční náklady:**
- **$204,000-276,000**

### Střední řešení (1,000 uživatelů)

**Měsíční náklady:**
- OpenAI API: $25,000-35,000
- Azure infrastruktura: $5,000-8,000
- **Celkem: $30,000-43,000**

**Roční náklady:**
- **$360,000-516,000**

## Doporučení pro implementaci

### Fáze 1: MVP (2-3 měsíce)
1. Základní chatbot s Assistants API
2. Cosmos DB pro konverzace
3. 3 statické scénáře
4. Základní zpětná vazba
5. CZ/SK podpora

### Fáze 2: Rozšíření (4-6 měsíců)
1. RAG systém s Azure AI Search
2. Web search capabilities
3. Pokročilé scénáře
4. Detailní analytika
5. Optimalizace nákladů

### Fáze 3: Pokročilé funkce (6-12 měsíců)
1. Multi-agent systém
2. Real-time data updates
3. Pokročilá personalizace
4. A/B testing
5. Pokročilé analytics

## Závěr

**Doporučuji začít s MVP** z následujících důvodů:

1. **Rychlá validace** - testování zájmu uživatelů
2. **Nízké riziko** - rozumné počáteční náklady
3. **Rychlé nasazení** - možnost rychlého startu
4. **Postupná evoluce** - možnost přidávání funkcí

MVP poskytuje všechny klíčové funkce při rozumných nákladech a umožňuje postupnou evoluci podle potřeby a zpětné vazby uživatelů.