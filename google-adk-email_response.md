# Odpověď na otázky z emailu

## Analýza byznysových aspektů

### Konkurence a výhody
Souhlasím s analýzou, že naším hlavním konkurentem jsou kvalitní bezplatné chatboty s přístupem k webu. Naše klíčové výhody:

1. **Specializované znalosti** - hluboké znalosti o českém a slovenském vysokém školství
2. **Personalizace** - dlouhodobá paměť a profilování uživatelů
3. **Strukturované scénáře** - připravené postupy od kariérových poradců
4. **Aktuální data** - přímé napojení na oficiální zdroje (MŠMT, univerzity)

### Doporučení pro MVP

**Ano, doporučuji začít s MVP** z následujících důvodů:

1. **Rychlé nasazení** - termín leden 2026 je skutečně ambiciózní
2. **Validace konceptu** - ověření zájmu uživatelů před velkou investicí
3. **Iterativní vývoj** - postupné přidávání funkcí na základě feedbacku
4. **Kontrola nákladů** - MVP umožní lepší odhad skutečných nákladů

## Odpovědi na konkrétní body

### 1. Dlouhodobá paměť
**MVP přístup:** 
- Ukládání kontextu v Azure Cosmos DB jako key-value store
- Základní profilování uživatele (preference, předchozí dotazy)
- Implementace GDPR compliance od začátku

**Implementace:**
```python
# Základní memory management pro MVP
class SimpleMemoryManager:
    def __init__(self, cosmos_client):
        self.db = cosmos_client
    
    async def store_user_context(self, user_id: str, context: dict):
        await self.db.upsert_item({
            "id": f"context_{user_id}",
            "user_id": user_id,
            "context": context,
            "timestamp": datetime.now()
        })
    
    async def get_user_context(self, user_id: str) -> dict:
        try:
            item = await self.db.read_item(f"context_{user_id}")
            return item.get("context", {})
        except:
            return {}
```

### 2. Dynamické zpracování scénářů
**MVP přístup:**
- 3 pevně definované scénáře v promptu
- Jednoduché rozhodovací stromy
- Možnost snadného přidávání nových scénářů

**Implementace:**
```python
SCENARIOS = {
    "humanitni": {
        "name": "Humanitní obory",
        "questions": [
            "Máte zájem o jazyky a literaturu?",
            "Zajímá vás historie a kultura?",
            "Chcete pracovat s lidmi?"
        ],
        "recommendations": ["FF UK", "FF MU", "FF UP"]
    },
    "technicke": {
        "name": "Technické obory", 
        "questions": [
            "Baví vás matematika a fyzika?",
            "Zajímá vás programování?",
            "Chcete řešit praktické problémy?"
        ],
        "recommendations": ["ČVUT", "VUT", "FIT ČVUT"]
    }
}
```

### 3. Web Search
**MVP doporučení:**
- **NEPOUŽÍVAT** v MVP z důvodu nákladů a složitosti
- Místo toho: pravidelné offline aktualizace databáze
- Implementovat až po ověření základní funkčnosti

### 4. Jazyková podpora
**MVP přístup:**
- Detekce jazyka z uživatelského vstupu
- Automatické přepínání mezi CZ/SK
- Jednoduchá implementace bez složitých jazykových modelů

### 5. Zpětná vazba
**MVP implementace:**
- Jednoduché tlačítko "Poslat shrnutí na email"
- Automatické sledování dokončení scénářů
- Základní metriky (dokončení, čas strávený)

## Nákladová analýza pro MVP

### Token usage (1 uživatel/měsíc):
- **Input tokens:** ~12,000 (scénáře + konverzace)
- **Output tokens:** ~6,000
- **Model:** GPT-4o-mini (levnější varianta)
- **Náklady:** ~$0.12/měsíc

### Pro 1,000 uživatelů:
- **Token náklady:** $120/měsíc
- **Infrastruktura:** $200/měsíc (Azure)
- **Celkem:** $320/měsíc

### ROI kalkulace:
- **Příjem:** 1,000 × $10 = $10,000/měsíc
- **Náklady:** $320/měsíc
- **Zisk:** $9,680/měsíc (96.8% margin)

## Implementační plán

### Fáze 1: Příprava (4 týdny)
1. **Týden 1-2:** Sběr a strukturování dat o univerzitách
2. **Týden 3:** Příprava 3 scénářů s kariérovými poradci
3. **Týden 4:** Základní databáze a API

### Fáze 2: MVP vývoj (6 týdnů)
1. **Týden 1-2:** Assistants API integrace
2. **Týden 3-4:** Function calling pro databázi
3. **Týden 5-6:** Frontend a testování

### Fáze 3: Nasazení a optimalizace (4 týdny)
1. **Týden 1-2:** Beta testování s omezenou skupinou
2. **Týden 3-4:** Opravy a optimalizace na základě feedbacku

## Doporučení pro vývojový tým

### Technologický stack pro MVP:
- **Backend:** Python + FastAPI
- **Database:** Azure SQL Database
- **AI:** OpenAI Assistants API + GPT-4o-mini
- **Frontend:** React/Next.js
- **Hosting:** Azure App Service

### Klíčové metriky pro MVP:
1. **Konverze:** % uživatelů, kteří dokončí scénář
2. **Spokojenost:** Hodnocení v emailu
3. **Retention:** % uživatelů vracejících se
4. **Náklady:** Token usage per user

### Rizika a mitigace:
1. **Riziko:** Vysoké náklady na tokeny
   - **Mitigace:** Použití GPT-4o-mini, prompt caching
2. **Riziko:** Nízká kvalita odpovědí
   - **Mitigace:** Pečlivé testování scénářů, A/B testování
3. **Riziko:** Technické problémy
   - **Mitigace:** Jednoduchá architektura, důkladné testování

## Závěr

**Doporučuji začít s MVP** z důvodu:
- Rychlého nasazení do ledna 2026
- Nízkých počátečních nákladů
- Možnosti validace konceptu
- Postupného přidávání funkcí

MVP by měl být **jednoduchý, ale funkční** - zaměřit se na kvalitu základních funkcí spíše než na množství pokročilých funkcí. Po úspěšném nasazení MVP můžeme postupně přidávat RAG, web search a další pokročilé funkce.

**Klíčové pro úspěch:** Kvalitní data o univerzitách a dobře připravené scénáře od kariérových poradců.