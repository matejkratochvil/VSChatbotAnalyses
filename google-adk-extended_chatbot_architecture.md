# Rozšířená architektura AI chatbotu s pokročilými funkcemi

## Přehled rozšířené architektury

Tento dokument popisuje komplexní architekturu chatbotu, která zahrnuje všechny požadované pokročilé funkce: RAG, MCP servery, prompt caching, context caching, memory, citace zdrojů a automatické aktualizace dat.

## Architektura systému

```
┌─────────────────────────────────────────────────────────────────┐
│                        Frontend Layer                          │
├─────────────────────────────────────────────────────────────────┤
│  Web App (React/Next.js)  │  Mobile App  │  Chat Widget       │
└─────────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────────┐
│                      API Gateway Layer                        │
├─────────────────────────────────────────────────────────────────┤
│  Load Balancer  │  Rate Limiting  │  Authentication  │  CORS   │
└─────────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────────┐
│                    Microservices Layer                        │
├─────────────────────────────────────────────────────────────────┤
│  Chat Service  │  RAG Service  │  Memory Service  │  Analytics │
└─────────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────────┐
│                    AI/ML Processing Layer                     │
├─────────────────────────────────────────────────────────────────┤
│  OpenAI API  │  Embedding Service  │  Vector Search  │  Cache  │
└─────────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────────┐
│                      Data Layer                               │
├─────────────────────────────────────────────────────────────────┤
│  PostgreSQL  │  Vector DB  │  Redis Cache  │  File Storage    │
└─────────────────────────────────────────────────────────────────┘
                                │
┌─────────────────────────────────────────────────────────────────┐
│                    External Services Layer                    │
├─────────────────────────────────────────────────────────────────┤
│  MCP Servers  │  Web Scrapers  │  University APIs  │  Analytics│
└─────────────────────────────────────────────────────────────────┘
```

## 1. RAG (Retrieval Augmented Generation) Implementace

### 1.1 Vektorová databáze
```python
# Azure Cognitive Search konfigurace
class VectorSearchService:
    def __init__(self):
        self.search_client = SearchClient(
            endpoint=os.getenv("AZURE_SEARCH_ENDPOINT"),
            index_name="university-knowledge",
            credential=AzureKeyCredential(os.getenv("AZURE_SEARCH_KEY"))
        )
    
    async def search_relevant_documents(self, query: str, top_k: int = 5):
        # Vytvoření embedding pro dotaz
        query_embedding = await self.create_embedding(query)
        
        # Vektorové vyhledávání
        search_results = self.search_client.search(
            search_text=query,
            vector_queries=[
                VectorizedQuery(
                    vector=query_embedding,
                    k_nearest_neighbors=top_k,
                    fields="content_vector"
                )
            ],
            select=["id", "content", "metadata", "source_url"]
        )
        
        return [result for result in search_results]
```

### 1.2 Knowledge Base Management
```python
class KnowledgeBaseManager:
    def __init__(self):
        self.embedding_model = "text-embedding-3-small"
        self.chunk_size = 1000
        self.chunk_overlap = 200
    
    async def add_documents(self, documents: List[Document]):
        """Přidání nových dokumentů do knowledge base"""
        chunks = self.split_documents(documents)
        embeddings = await self.create_embeddings_batch(chunks)
        
        # Uložení do vektorové databáze
        await self.vector_db.add_documents(chunks, embeddings)
    
    async def update_document(self, doc_id: str, new_content: str):
        """Aktualizace existujícího dokumentu"""
        # Smazání staré verze
        await self.vector_db.delete_document(doc_id)
        
        # Přidání nové verze
        chunks = self.split_text(new_content)
        embeddings = await self.create_embeddings_batch(chunks)
        await self.vector_db.add_documents(chunks, embeddings, doc_id)
```

## 2. MCP (Model Context Protocol) Servery

### 2.1 Database MCP Server
```python
class DatabaseMCPServer:
    """MCP server pro přístup k databázi univerzit"""
    
    def __init__(self, db_connection):
        self.db = db_connection
        self.tools = {
            "search_universities": self.search_universities,
            "get_university_details": self.get_university_details,
            "get_admission_requirements": self.get_admission_requirements,
            "get_scholarships": self.get_scholarships
        }
    
    async def search_universities(self, query: dict) -> dict:
        """Vyhledání univerzit podle kritérií"""
        filters = self.build_filters(query)
        results = await self.db.execute(
            "SELECT * FROM universities WHERE " + filters,
            query.get("params", {})
        )
        return {"universities": results}
    
    async def get_university_details(self, university_id: str) -> dict:
        """Získání detailních informací o univerzitě"""
        university = await self.db.fetch_one(
            "SELECT * FROM universities WHERE id = %s", university_id
        )
        programs = await self.db.fetch_all(
            "SELECT * FROM study_programs WHERE university_id = %s", university_id
        )
        return {
            "university": university,
            "programs": programs
        }
```

### 2.2 Web Scraping MCP Server
```python
class WebScrapingMCPServer:
    """MCP server pro získávání aktuálních dat z webu"""
    
    def __init__(self):
        self.scrapers = {
            "university_websites": UniversityWebsiteScraper(),
            "admission_dates": AdmissionDatesScraper(),
            "scholarship_info": ScholarshipScraper(),
            "news_articles": NewsScraper()
        }
    
    async def scrape_university_data(self, university_url: str) -> dict:
        """Scraping dat z webu univerzity"""
        scraper = self.scrapers["university_websites"]
        data = await scraper.scrape(university_url)
        
        # Validace a čištění dat
        cleaned_data = self.clean_scraped_data(data)
        
        # Uložení do databáze
        await self.update_database(cleaned_data)
        
        return cleaned_data
    
    async def get_current_admission_dates(self, university_id: str) -> dict:
        """Získání aktuálních termínů přijímacích zkoušek"""
        scraper = self.scrapers["admission_dates"]
        dates = await scraper.get_admission_dates(university_id)
        return {"admission_dates": dates}
```

## 3. Prompt Caching

### 3.1 Redis-based Prompt Cache
```python
class PromptCacheService:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.cache_ttl = 3600  # 1 hodina
    
    async def get_cached_response(self, prompt_hash: str) -> Optional[str]:
        """Získání cached odpovědi"""
        cached = await self.redis.get(f"prompt:{prompt_hash}")
        return cached.decode() if cached else None
    
    async def cache_response(self, prompt_hash: str, response: str):
        """Uložení odpovědi do cache"""
        await self.redis.setex(
            f"prompt:{prompt_hash}",
            self.cache_ttl,
            response
        )
    
    def generate_prompt_hash(self, prompt: str, context: dict) -> str:
        """Generování hash pro prompt"""
        content = f"{prompt}:{json.dumps(context, sort_keys=True)}"
        return hashlib.md5(content.encode()).hexdigest()
```

### 3.2 Intelligent Caching Strategy
```python
class IntelligentCacheManager:
    def __init__(self):
        self.cache_service = PromptCacheService(redis_client)
        self.similarity_threshold = 0.85
    
    async def get_or_generate_response(self, prompt: str, context: dict) -> str:
        """Inteligentní cache s podobností"""
        # Hledání podobných cached odpovědí
        similar_response = await self.find_similar_cached_response(prompt, context)
        
        if similar_response:
            return similar_response
        
        # Generování nové odpovědi
        response = await self.generate_new_response(prompt, context)
        
        # Cache nové odpovědi
        prompt_hash = self.cache_service.generate_prompt_hash(prompt, context)
        await self.cache_service.cache_response(prompt_hash, response)
        
        return response
```

## 4. Context Caching

### 4.1 Conversation Context Manager
```python
class ConversationContextManager:
    def __init__(self):
        self.redis = redis_client
        self.max_context_length = 8000  # tokens
        self.summarization_threshold = 6000
    
    async def get_conversation_context(self, session_id: str) -> dict:
        """Získání kontextu konverzace"""
        context_data = await self.redis.get(f"context:{session_id}")
        if context_data:
            return json.loads(context_data)
        return {"messages": [], "summary": "", "user_profile": {}}
    
    async def update_conversation_context(self, session_id: str, new_message: dict):
        """Aktualizace kontextu konverzace"""
        context = await self.get_conversation_context(session_id)
        
        # Přidání nové zprávy
        context["messages"].append(new_message)
        
        # Kontrola délky kontextu
        if self.get_context_length(context) > self.summarization_threshold:
            await self.summarize_old_messages(context)
        
        # Uložení aktualizovaného kontextu
        await self.redis.setex(
            f"context:{session_id}",
            86400,  # 24 hodin
            json.dumps(context)
        )
    
    async def summarize_old_messages(self, context: dict):
        """Shrnutí starších zpráv pro úsporu tokenů"""
        old_messages = context["messages"][:-5]  # Zachovat posledních 5 zpráv
        
        if old_messages:
            summary = await self.create_summary(old_messages)
            context["summary"] = summary
            context["messages"] = context["messages"][-5:]  # Zachovat pouze posledních 5
```

## 5. Memory Management

### 5.1 Long-term Memory System
```python
class LongTermMemorySystem:
    def __init__(self):
        self.db = database_connection
        self.embedding_service = EmbeddingService()
    
    async def store_user_interaction(self, user_id: str, interaction: dict):
        """Uložení interakce uživatele"""
        # Vytvoření embedding pro vyhledávání
        interaction_embedding = await self.embedding_service.create_embedding(
            f"{interaction['query']} {interaction['response']}"
        )
        
        # Uložení do databáze
        await self.db.execute("""
            INSERT INTO user_interactions 
            (user_id, query, response, embedding, timestamp, metadata)
            VALUES (%s, %s, %s, %s, %s, %s)
        """, (
            user_id,
            interaction['query'],
            interaction['response'],
            interaction_embedding,
            datetime.now(),
            json.dumps(interaction.get('metadata', {}))
        ))
    
    async def get_relevant_memories(self, user_id: str, current_query: str, limit: int = 5):
        """Získání relevantních vzpomínek"""
        query_embedding = await self.embedding_service.create_embedding(current_query)
        
        # Vektorové vyhledávání v user interactions
        memories = await self.db.execute("""
            SELECT query, response, metadata, 
                   embedding <-> %s as distance
            FROM user_interactions 
            WHERE user_id = %s 
            ORDER BY distance 
            LIMIT %s
        """, (query_embedding, user_id, limit))
        
        return memories
```

### 5.2 User Profile Management
```python
class UserProfileManager:
    def __init__(self):
        self.db = database_connection
        self.preferences_cache = {}
    
    async def get_user_profile(self, user_id: str) -> dict:
        """Získání profilu uživatele"""
        profile = await self.db.fetch_one("""
            SELECT * FROM user_profiles WHERE user_id = %s
        """, user_id)
        
        if not profile:
            # Vytvoření nového profilu
            profile = await self.create_default_profile(user_id)
        
        return profile
    
    async def update_user_preferences(self, user_id: str, preferences: dict):
        """Aktualizace preferencí uživatele"""
        await self.db.execute("""
            UPDATE user_profiles 
            SET preferences = %s, updated_at = %s
            WHERE user_id = %s
        """, (json.dumps(preferences), datetime.now(), user_id))
        
        # Aktualizace cache
        self.preferences_cache[user_id] = preferences
    
    async def infer_preferences_from_interactions(self, user_id: str):
        """Odvození preferencí z interakcí"""
        interactions = await self.db.fetch_all("""
            SELECT query, response, metadata 
            FROM user_interactions 
            WHERE user_id = %s 
            ORDER BY timestamp DESC 
            LIMIT 100
        """, user_id)
        
        # Analýza interakcí pomocí LLM
        preferences = await self.analyze_interactions_for_preferences(interactions)
        
        # Uložení odvozených preferencí
        await self.update_user_preferences(user_id, preferences)
```

## 6. Source Citation System

### 6.1 Citation Manager
```python
class CitationManager:
    def __init__(self):
        self.db = database_connection
        self.source_cache = {}
    
    async def add_citation(self, response_id: str, source_info: dict):
        """Přidání citace k odpovědi"""
        citation = {
            "response_id": response_id,
            "source_type": source_info["type"],  # "database", "web", "knowledge_base"
            "source_url": source_info.get("url"),
            "source_title": source_info.get("title"),
            "relevance_score": source_info.get("relevance_score", 1.0),
            "extracted_at": datetime.now()
        }
        
        await self.db.execute("""
            INSERT INTO citations 
            (response_id, source_type, source_url, source_title, relevance_score, extracted_at)
            VALUES (%s, %s, %s, %s, %s, %s)
        """, tuple(citation.values()))
    
    async def get_citations_for_response(self, response_id: str) -> List[dict]:
        """Získání citací pro odpověď"""
        citations = await self.db.fetch_all("""
            SELECT * FROM citations 
            WHERE response_id = %s 
            ORDER BY relevance_score DESC
        """, response_id)
        
        return citations
    
    def format_citations_for_display(self, citations: List[dict]) -> str:
        """Formátování citací pro zobrazení"""
        if not citations:
            return ""
        
        formatted = "\n\n**Zdroje:**\n"
        for i, citation in enumerate(citations, 1):
            if citation["source_url"]:
                formatted += f"{i}. [{citation['source_title']}]({citation['source_url']})\n"
            else:
                formatted += f"{i}. {citation['source_title']}\n"
        
        return formatted
```

## 7. Automated Data Updates

### 7.1 Data Update Scheduler
```python
class DataUpdateScheduler:
    def __init__(self):
        self.scheduler = AsyncIOScheduler()
        self.web_scraper = WebScrapingMCPServer()
        self.knowledge_manager = KnowledgeBaseManager()
    
    def setup_scheduled_tasks(self):
        """Nastavení plánovaných úkolů"""
        # Denní aktualizace dat o univerzitách
        self.scheduler.add_job(
            self.update_university_data,
            'cron',
            hour=2,  # 2:00 AM
            id='daily_university_update'
        )
        
        # Týdenní aktualizace knowledge base
        self.scheduler.add_job(
            self.update_knowledge_base,
            'cron',
            day_of_week=0,  # Neděle
            hour=3,
            id='weekly_knowledge_update'
        )
        
        # Měsíční aktualizace scénářů
        self.scheduler.add_job(
            self.update_career_scenarios,
            'cron',
            day=1,  # První den v měsíci
            hour=4,
            id='monthly_scenario_update'
        )
    
    async def update_university_data(self):
        """Aktualizace dat o univerzitách"""
        universities = await self.get_all_universities()
        
        for university in universities:
            try:
                # Scraping aktuálních dat
                updated_data = await self.web_scraper.scrape_university_data(
                    university["website_url"]
                )
                
                # Aktualizace databáze
                await self.update_university_in_database(university["id"], updated_data)
                
                # Aktualizace knowledge base
                await self.knowledge_manager.update_document(
                    f"university_{university['id']}",
                    updated_data["description"]
                )
                
            except Exception as e:
                logger.error(f"Failed to update university {university['id']}: {e}")
    
    async def update_knowledge_base(self):
        """Aktualizace knowledge base"""
        # Získání nových dokumentů z různých zdrojů
        new_documents = await self.collect_new_documents()
        
        # Přidání do knowledge base
        await self.knowledge_manager.add_documents(new_documents)
        
        # Optimalizace vektorové databáze
        await self.optimize_vector_database()
```

### 7.2 Change Detection System
```python
class ChangeDetectionSystem:
    def __init__(self):
        self.db = database_connection
        self.notification_service = NotificationService()
    
    async def detect_website_changes(self, university_id: str, website_url: str):
        """Detekce změn na webu univerzity"""
        # Získání aktuálního obsahu
        current_content = await self.fetch_website_content(website_url)
        
        # Získání předchozího obsahu
        previous_content = await self.get_stored_content(university_id)
        
        # Porovnání obsahu
        if self.content_has_changed(previous_content, current_content):
            # Uložení nového obsahu
            await self.store_content(university_id, current_content)
            
            # Oznámení o změně
            await self.notification_service.notify_content_change(
                university_id, 
                self.extract_changes(previous_content, current_content)
            )
            
            # Aktualizace knowledge base
            await self.update_knowledge_base_with_changes(university_id, current_content)
    
    def content_has_changed(self, old_content: str, new_content: str) -> bool:
        """Kontrola, zda se obsah změnil"""
        # Použití hash pro rychlé porovnání
        old_hash = hashlib.md5(old_content.encode()).hexdigest()
        new_hash = hashlib.md5(new_content.encode()).hexdigest()
        
        return old_hash != new_hash
```

## 8. Monitoring a Analytics

### 8.1 Performance Monitoring
```python
class PerformanceMonitor:
    def __init__(self):
        self.metrics_collector = MetricsCollector()
        self.alerting_service = AlertingService()
    
    async def track_response_time(self, session_id: str, start_time: float, end_time: float):
        """Sledování času odpovědi"""
        response_time = end_time - start_time
        
        await self.metrics_collector.record_metric(
            "response_time",
            response_time,
            tags={"session_id": session_id}
        )
        
        # Alert při pomalých odpovědích
        if response_time > 5.0:  # 5 sekund
            await self.alerting_service.send_alert(
                f"Slow response detected: {response_time}s for session {session_id}"
            )
    
    async def track_token_usage(self, session_id: str, input_tokens: int, output_tokens: int):
        """Sledování použití tokenů"""
        await self.metrics_collector.record_metric(
            "input_tokens",
            input_tokens,
            tags={"session_id": session_id}
        )
        
        await self.metrics_collector.record_metric(
            "output_tokens", 
            output_tokens,
            tags={"session_id": session_id}
        )
```

### 8.2 User Analytics
```python
class UserAnalytics:
    def __init__(self):
        self.db = database_connection
        self.ga4_client = GA4Client()
    
    async def track_user_interaction(self, user_id: str, interaction: dict):
        """Sledování interakce uživatele"""
        # Uložení do databáze
        await self.db.execute("""
            INSERT INTO user_analytics 
            (user_id, event_type, event_data, timestamp)
            VALUES (%s, %s, %s, %s)
        """, (
            user_id,
            interaction["event_type"],
            json.dumps(interaction["data"]),
            datetime.now()
        ))
        
        # Odeslání do GA4
        await self.ga4_client.track_event(
            user_id=user_id,
            event_name=interaction["event_type"],
            parameters=interaction["data"]
        )
    
    async def get_conversion_metrics(self, date_from: datetime, date_to: datetime) -> dict:
        """Získání metrik konverze"""
        metrics = await self.db.fetch_one("""
            SELECT 
                COUNT(DISTINCT user_id) as total_users,
                COUNT(CASE WHEN event_type = 'scenario_completed' THEN 1 END) as completed_scenarios,
                COUNT(CASE WHEN event_type = 'feedback_positive' THEN 1 END) as positive_feedback
            FROM user_analytics 
            WHERE timestamp BETWEEN %s AND %s
        """, (date_from, date_to))
        
        return {
            "conversion_rate": metrics["completed_scenarios"] / metrics["total_users"],
            "satisfaction_rate": metrics["positive_feedback"] / metrics["completed_scenarios"]
        }
```

## 9. Bezpečnost a GDPR

### 9.1 Data Privacy Manager
```python
class DataPrivacyManager:
    def __init__(self):
        self.db = database_connection
        self.encryption_service = EncryptionService()
    
    async def anonymize_user_data(self, user_id: str):
        """Anonymizace dat uživatele"""
        # Anonymizace osobních údajů
        await self.db.execute("""
            UPDATE user_profiles 
            SET name = %s, email = %s, phone = %s
            WHERE user_id = %s
        """, (
            f"User_{user_id[:8]}",
            f"user_{user_id[:8]}@anonymized.com",
            None,
            user_id
        ))
        
        # Anonymizace interakcí
        await self.db.execute("""
            UPDATE user_interactions 
            SET query = %s, response = %s
            WHERE user_id = %s
        """, (
            "[ANONYMIZED]",
            "[ANONYMIZED]",
            user_id
        ))
    
    async def delete_user_data(self, user_id: str):
        """Smazání všech dat uživatele (GDPR)"""
        # Smazání všech záznamů uživatele
        await self.db.execute("DELETE FROM user_profiles WHERE user_id = %s", user_id)
        await self.db.execute("DELETE FROM user_interactions WHERE user_id = %s", user_id)
        await self.db.execute("DELETE FROM user_analytics WHERE user_id = %s", user_id)
        
        # Smazání z cache
        await self.redis.delete(f"context:{user_id}")
        await self.redis.delete(f"preferences:{user_id}")
```

## 10. Deployment a Scaling

### 10.1 Docker Configuration
```dockerfile
# Dockerfile pro chat service
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 10.2 Kubernetes Deployment
```yaml
# k8s-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: chatbot-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: chatbot-service
  template:
    metadata:
      labels:
        app: chatbot-service
    spec:
      containers:
      - name: chatbot
        image: chatbot:latest
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: chatbot-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: chatbot-secrets
              key: redis-url
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
```

Tato rozšířená architektura poskytuje kompletní řešení pro pokročilý AI chatbot s všemi požadovanými funkcemi, škálovatelností a bezpečnostními opatřeními.