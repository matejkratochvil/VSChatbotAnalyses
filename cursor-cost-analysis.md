# Detailní analýza nákladů chatbotu pro výběr vysoké školy

## Přehled nákladů podle řešení

### 1. MVP Řešení - Základní RAG Chatbot

#### Token kalkulace (měsíčně na uživatele)
**Předpoklady:**
- 10 konverzací měsíčně na uživatele
- 5 zpráv v průměrné konverzaci
- 1,000 tokenů na vstupní zprávu
- 600 tokenů na výstupní odpověď

**Výpočet:**
- Input tokeny: 10 × 5 × 1,000 = 50,000 tokenů
- Output tokeny: 10 × 5 × 600 = 30,000 tokenů
- **Celkem: 80,000 tokenů/měsíc/uživatel**

#### Náklady při 1,000 uživatelích (měsíčně)

| Služba | Konfigurace | Cena/měsíc |
|--------|-------------|------------|
| **Azure OpenAI** | GPT-4o-mini, 80M tokenů | $240 |
| **Azure AI Search** | Standard S1, 1 index | $250 |
| **Azure Container Instances** | 1 vCPU, 1GB RAM | $50 |
| **Azure Storage** | Standard LRS, 100GB | $5 |
| **Azure Container Registry** | Basic tier | $5 |
| **Celkem** | | **$550** |

#### Fixní náklady (ročně)
- Azure Container Registry: $60
- Monitoring (Application Insights): $120
- **Celkem fixní náklady: $180/rok**

#### Variabilní náklady (měsíčně při 1,000 uživatelích)
- Tokeny: $240
- Compute: $55
- Storage: $5
- **Celkem variabilní: $300/měsíc**

---

### 2. Střední řešení - Rozšířený Chatbot s pamětí

#### Token kalkulace (měsíčně na uživatele)
**Předpoklady:**
- 12 konverzací měsíčně na uživatele
- 6 zpráv v průměrné konverzaci
- 1,200 tokenů na vstupní zprávu (včetně kontextu paměti)
- 700 tokenů na výstupní odpověď

**Výpočet:**
- Input tokeny: 12 × 6 × 1,200 = 86,400 tokenů
- Output tokeny: 12 × 6 × 700 = 50,400 tokenů
- **Celkem: 136,800 tokenů/měsíc/uživatel**

#### Náklady při 1,000 uživatelích (měsíčně)

| Služba | Konfigurace | Cena/měsíc |
|--------|-------------|------------|
| **Azure OpenAI** | GPT-4o, 136.8M tokenů | $410 |
| **Azure AI Search** | Standard S1, 2 indexy | $300 |
| **Azure Cosmos DB** | Provisioned, 400 RU/s | $100 |
| **Azure Container Apps** | 2 vCPU, 4GB RAM | $80 |
| **Redis Cache** | Standard C1 | $50 |
| **Azure Functions** | Consumption plan | $30 |
| **Azure Storage** | Standard LRS, 500GB | $25 |
| **Azure Container Registry** | Standard tier | $20 |
| **Celkem** | | **$1,015** |

#### Fixní náklady (ročně)
- Azure Container Registry: $240
- Monitoring (Application Insights): $200
- Security Center: $100
- **Celkem fixní náklady: $540/rok**

#### Variabilní náklady (měsíčně při 1,000 uživatelích)
- Tokeny: $410
- Compute: $160
- Storage: $125
- **Celkem variabilní: $695/měsíc**

---

### 3. Pokročilé řešení - Multi-agentní systém

#### Token kalkulace (měsíčně na uživatele)
**Předpoklady:**
- 15 konverzací měsíčně na uživatele
- 7 zpráv v průměrné konverzaci
- 1,500 tokenů na vstupní zprávu (včetně kontextu a web search)
- 800 tokenů na výstupní odpověď

**Výpočet:**
- Input tokeny: 15 × 7 × 1,500 = 157,500 tokenů
- Output tokeny: 15 × 7 × 800 = 84,000 tokenů
- **Celkem: 241,500 tokenů/měsíc/uživatel**

#### Náklady při 1,000 uživatelích (měsíčně)

| Služba | Konfigurace | Cena/měsíc |
|--------|-------------|------------|
| **Azure OpenAI** | GPT-4o, 241.5M tokenů | $725 |
| **Azure AI Search** | Standard S2, 3 indexy | $450 |
| **Azure Cosmos DB** | Provisioned, 1000 RU/s | $200 |
| **Azure Container Apps** | 4 vCPU, 8GB RAM | $160 |
| **Azure Functions** | Premium plan | $100 |
| **Redis Cache** | Standard C2 | $100 |
| **Web Scraping Service** | Custom solution | $100 |
| **Azure Storage** | Standard LRS, 1TB | $50 |
| **Azure Container Registry** | Premium tier | $50 |
| **Azure Cognitive Services** | Translation, PII | $30 |
| **Celkem** | | **$1,915** |

#### Fixní náklady (ročně)
- Azure Container Registry: $600
- Monitoring (Application Insights): $300
- Security Center: $200
- Backup & Recovery: $150
- **Celkem fixní náklady: $1,250/rok**

#### Variabilní náklady (měsíčně při 1,000 uživatelích)
- Tokeny: $725
- Compute: $360
- Storage: $200
- **Celkem variabilní: $1,285/měsíc**

---

### 4. Enterprise řešení - Plně autonomní systém

#### Token kalkulace (měsíčně na uživatele)
**Předpoklady:**
- 20 konverzací měsíčně na uživatele
- 8 zpráv v průměrné konverzaci
- 2,000 tokenů na vstupní zprávu (optimalizované)
- 1,000 tokenů na výstupní odpověď (vysoce kvalitní)

**Výpočet:**
- Input tokeny: 20 × 8 × 2,000 = 320,000 tokenů
- Output tokeny: 20 × 8 × 1,000 = 160,000 tokenů
- **Celkem: 480,000 tokenů/měsíc/uživatel**

#### Náklady při 1,000 uživatelích (měsíčně)

| Služba | Konfigurace | Cena/měsíc |
|--------|-------------|------------|
| **Azure AI Foundry** | Multi-model approach | $800 |
| **Custom Models** | Fine-tuned models | $300 |
| **Azure AI Search** | Standard S3, 5 indexů | $750 |
| **Azure Cosmos DB** | Provisioned, 2000 RU/s | $400 |
| **Microservices** | 8 vCPU, 16GB RAM | $300 |
| **Advanced Cache** | Redis Enterprise | $150 |
| **Monitoring** | Application Insights + Log Analytics | $200 |
| **Azure Storage** | Premium LRS, 2TB | $100 |
| **Azure Container Registry** | Premium tier | $100 |
| **Azure Cognitive Services** | Full suite | $100 |
| **Celkem** | | **$3,250** |

#### Fixní náklady (ročně)
- Azure Container Registry: $1,200
- Monitoring & Observability: $600
- Security Center: $400
- Backup & Recovery: $300
- Compliance Tools: $200
- **Celkem fixní náklady: $2,700/rok**

#### Variabilní náklady (měsíčně při 1,000 uživatelích)
- Tokeny: $1,100
- Compute: $750
- Storage: $300
- **Celkem variabilní: $2,150/měsíc**

---

## Detailní analýza nákladů podle počtu uživatelů

### MVP Řešení

| Počet uživatelů | Tokeny/měsíc | Náklady/měsíc | Náklady/rok |
|-----------------|--------------|---------------|-------------|
| 100 | 8M | $55 | $660 |
| 320 | 25.6M | $176 | $2,112 |
| 1,000 | 80M | $550 | $6,600 |
| 2,000 | 160M | $1,100 | $13,200 |

### Střední řešení

| Počet uživatelů | Tokeny/měsíc | Náklady/měsíc | Náklady/rok |
|-----------------|--------------|---------------|-------------|
| 100 | 13.7M | $102 | $1,224 |
| 320 | 43.8M | $325 | $3,900 |
| 1,000 | 136.8M | $1,015 | $12,180 |
| 2,000 | 273.6M | $2,030 | $24,360 |

### Pokročilé řešení

| Počet uživatelů | Tokeny/měsíc | Náklady/měsíc | Náklady/rok |
|-----------------|--------------|---------------|-------------|
| 100 | 24.2M | $192 | $2,304 |
| 320 | 77.3M | $613 | $7,356 |
| 1,000 | 241.5M | $1,915 | $22,980 |
| 2,000 | 483M | $3,830 | $45,960 |

### Enterprise řešení

| Počet uživatelů | Tokeny/měsíc | Náklady/měsíc | Náklady/rok |
|-----------------|--------------|---------------|-------------|
| 100 | 48M | $325 | $3,900 |
| 320 | 153.6M | $1,040 | $12,480 |
| 1,000 | 480M | $3,250 | $39,000 |
| 2,000 | 960M | $6,500 | $78,000 |

---

## Optimalizace nákladů

### 1. Prompt Caching
**Úspora:** 30-40% tokenů
- Implementace Redis cache pro opakující se dotazy
- Cache TTL: 1 hodina
- Úspora při 1,000 uživatelích: $120-160/měsíc

### 2. Model Selection
**Úspora:** 20-30% nákladů
- GPT-4o-mini pro jednoduché dotazy
- GPT-4o pouze pro komplexní scénáře
- Úspora při 1,000 uživatelích: $100-150/měsíc

### 3. Batch Processing
**Úspora:** 15-25% nákladů
- Seskupování podobných dotazů
- Batch API calls
- Úspora při 1,000 uživatelích: $75-125/měsíc

### 4. Regional Optimization
**Úspora:** 10-20% nákladů
- Výběr nejlevnějších regionů
- Edge caching
- Úspora při 1,000 uživatelích: $50-100/měsíc

---

## ROI Analýza

### Předpoklady
- Průměrná cena za uživatele: $10/měsíc
- Provozní náklady: 30% z příjmů
- Marketing náklady: 20% z příjmů
- Vývoj a údržba: 15% z příjmů

### ROI při 1,000 uživatelích (střední řešení)

| Metrika | Hodnota |
|---------|---------|
| **Měsíční příjmy** | $10,000 |
| **Měsíční náklady** | $1,015 |
| **Provozní náklady** | $3,000 |
| **Marketing náklady** | $2,000 |
| **Vývoj a údržba** | $1,500 |
| **Celkové náklady** | $7,515 |
| **Měsíční zisk** | $2,485 |
| **ROI** | 33% |

### Break-even analýza

| Řešení | Break-even (uživatelé) | Break-even (měsíce) |
|--------|------------------------|-------------------|
| MVP | 55 | 1 |
| Střední | 102 | 2 |
| Pokročilé | 192 | 3 |
| Enterprise | 325 | 4 |

---

## Doporučení pro optimalizaci nákladů

### Fáze 1 (Měsíce 1-3): MVP
- Začít s MVP řešením
- Implementovat základní monitoring
- Sledovat skutečné použití

### Fáze 2 (Měsíce 4-6): Optimalizace
- Implementovat prompt caching
- Optimalizovat model selection
- Přidat batch processing

### Fáze 3 (Měsíce 7-12): Rozšíření
- Upgrade na střední řešení
- Implementovat dlouhodobou paměť
- Přidat pokročilé scénáře

### Fáze 4 (Rok 2+): Scale
- Upgrade na pokročilé řešení
- Implementovat multi-agentní systém
- Přidat web search

---

## Závěr

**Doporučené řešení:** Střední řešení s postupným upgradem
- **Počáteční investice:** $35,000-50,000
- **Měsíční náklady při 1,000 uživatelích:** $1,015
- **Break-even:** 2 měsíce
- **ROI:** 33%

Toto řešení poskytuje optimální poměr funkcionality, nákladů a flexibility pro budoucí rozšíření.