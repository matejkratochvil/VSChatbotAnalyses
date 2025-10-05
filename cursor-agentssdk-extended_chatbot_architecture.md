# Rozšířená architektura AI chatbotu s pokročilými funkcemi

## Přehled architektury

Rozšířené řešení kombinuje nejlepší přístupy z OpenAI Cookbook s pokročilými funkcemi pro profesionální nasazení.

## Komponenty systému

### 1. RAG (Retrieval-Augmented Generation) System

**Implementace:**
```python
class RAGSystem:
    def __init__(self):
        self.embedding_model = "text-embedding-3-large"
        self.vector_db = ChromaDB()
        self.search_client = AzureSearchClient()
    
    def retrieve_documents(self, query, top_k=5):
        # Embedding query
        query_embedding = self.get_embedding(query)
        
        # Vector search
        vector_results = self.vector_db.similarity_search(
            query_embedding, 
            top_k=top_k
        )
        
        # Hybrid search s Azure AI Search
        search_results = self.search_client.search(
            query=query,
            top=top_k,
            filters=self.get_filters()
        )
        
        # Combine results
        return self.merge_results(vector_results, search_results)
```

**Výhody:**
- Kombinuje sémantické a full-text vyhledávání
- Automatické citace zdrojů
- Škálovatelnost
- Flexibilní filtrování

### 2. MCP (Model Context Protocol) Server

**Implementace:**
```python
class DatabaseMCPServer:
    def __init__(self):
        self.db_connection = AzureSQLConnection()
        self.cache = RedisCache()
    
    @mcp_tool
    def get_university_info(self, university_id: str) -> dict:
        """Získání informací o univerzitě"""
        cache_key = f"university:{university_id}"
        
        # Check cache first
        if cached := self.cache.get(cache_key):
            return cached
        
        # Query database
        result = self.db_connection.execute(
            "SELECT * FROM universities WHERE id = ?", 
            university_id
        )
        
        # Cache result
        self.cache.set(cache_key, result, ttl=3600)
        return result
    
    @mcp_tool
    def search_programs(self, criteria: dict) -> list:
        """Vyhledání studijních programů podle kritérií"""
        query = self.build_search_query(criteria)
        results = self.db_connection.execute(query)
        return results
```

**Funkce:**
- Databázové operace
- Caching výsledků
- Query optimization
- Error handling
- Logging a monitoring

### 3. Prompt Caching System

**Implementace:**
```python
class PromptCache:
    def __init__(self):
        self.redis = Redis()
        self.cache_ttl = 3600  # 1 hour
    
    def get_cached_response(self, prompt_hash: str) -> Optional[str]:
        """Získání cached odpovědi"""
        return self.redis.get(f"prompt:{prompt_hash}")
    
    def cache_response(self, prompt_hash: str, response: str):
        """Uložení odpovědi do cache"""
        self.redis.setex(
            f"prompt:{prompt_hash}", 
            self.cache_ttl, 
            response
        )
    
    def generate_prompt_hash(self, prompt: str, context: dict) -> str:
        """Generování hash pro prompt"""
        content = f"{prompt}:{json.dumps(context, sort_keys=True)}"
        return hashlib.md5(content.encode()).hexdigest()
```

**Výhody:**
- Snížení nákladů na tokeny
- Rychlejší odpovědi
- Snížení latence
- Lepší uživatelská zkušenost

### 4. Context Caching

**Implementace:**
```python
class ContextCache:
    def __init__(self):
        self.azure_cache = AzureCache()
        self.memory_threshold = 8000  # tokens
    
    def get_user_context(self, user_id: str) -> dict:
        """Získání kontextu uživatele"""
        context_key = f"context:{user_id}"
        return self.azure_cache.get(context_key) or {}
    
    def update_user_context(self, user_id: str, new_context: dict):
        """Aktualizace kontextu uživatele"""
        current_context = self.get_user_context(user_id)
        updated_context = self.merge_contexts(current_context, new_context)
        
        # Check if context is too large
        if self.get_token_count(updated_context) > self.memory_threshold:
            updated_context = self.summarize_context(updated_context)
        
        self.azure_cache.set(f"context:{user_id}", updated_context)
    
    def summarize_context(self, context: dict) -> dict:
        """Shrnutí kontextu pro úsporu tokenů"""
        # Use GPT-4o-mini for summarization
        summary_prompt = self.build_summary_prompt(context)
        summary = self.openai_client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": summary_prompt}]
        )
        return {"summary": summary.choices[0].message.content}
```

### 5. Memory System

**Implementace:**
```python
class MemorySystem:
    def __init__(self):
        self.cosmos_db = CosmosDB()
        self.embedding_model = "text-embedding-3-small"
    
    def store_conversation(self, user_id: str, conversation: dict):
        """Uložení konverzace"""
        conversation_doc = {
            "id": str(uuid.uuid4()),
            "user_id": user_id,
            "timestamp": datetime.utcnow(),
            "conversation": conversation,
            "embedding": self.get_embedding(conversation["content"])
        }
        self.cosmos_db.upsert_item(conversation_doc)
    
    def get_user_memory(self, user_id: str, query: str) -> list:
        """Získání relevantní paměti uživatele"""
        query_embedding = self.get_embedding(query)
        
        # Search similar conversations
        similar_conversations = self.cosmos_db.vector_search(
            query_embedding,
            filter={"user_id": user_id},
            top_k=5
        )
        
        return similar_conversations
    
    def get_user_preferences(self, user_id: str) -> dict:
        """Získání preferencí uživatele"""
        preferences = self.cosmos_db.get_item(
            f"preferences:{user_id}"
        )
        return preferences or {}
```

### 6. Citation Engine

**Implementace:**
```python
class CitationEngine:
    def __init__(self):
        self.source_tracker = SourceTracker()
        self.metadata_extractor = MetadataExtractor()
    
    def add_citations(self, response: str, sources: list) -> str:
        """Přidání citací do odpovědi"""
        cited_response = response
        
        for i, source in enumerate(sources, 1):
            citation = self.format_citation(source, i)
            cited_response += f"\n\n[{i}] {citation}"
        
        return cited_response
    
    def format_citation(self, source: dict, number: int) -> str:
        """Formátování citace"""
        return f"{source['title']} - {source['url']} (přístup: {source['date']})"
    
    def track_source_usage(self, source_id: str, query: str):
        """Sledování použití zdrojů"""
        self.source_tracker.log_usage(source_id, query)
```

### 7. Web Data Fetcher

**Implementace:**
```python
class WebDataFetcher:
    def __init__(self):
        self.scheduler = APScheduler()
        self.scrapers = {
            'universities': UniversityScraper(),
            'programs': ProgramScraper(),
            'events': EventScraper()
        }
    
    def schedule_data_updates(self):
        """Naplánování aktualizací dat"""
        # Daily updates for critical data
        self.scheduler.add_job(
            self.update_university_data,
            'cron',
            hour=2,
            minute=0
        )
        
        # Weekly updates for less critical data
        self.scheduler.add_job(
            self.update_program_data,
            'cron',
            day_of_week=0,
            hour=3
        )
    
    def update_university_data(self):
        """Aktualizace dat o univerzitách"""
        for scraper in self.scrapers.values():
            try:
                new_data = scraper.scrape()
                self.validate_data(new_data)
                self.store_data(new_data)
                self.notify_updates(new_data)
            except Exception as e:
                self.log_error(e)
    
    def validate_data(self, data: dict) -> bool:
        """Validace získaných dat"""
        required_fields = ['name', 'url', 'last_updated']
        return all(field in data for field in required_fields)
```

## Orchestrace systému

### Main Chatbot Class

```python
class UniversityAdvisorChatbot:
    def __init__(self):
        self.rag_system = RAGSystem()
        self.mcp_server = DatabaseMCPServer()
        self.prompt_cache = PromptCache()
        self.context_cache = ContextCache()
        self.memory_system = MemorySystem()
        self.citation_engine = CitationEngine()
        self.web_fetcher = WebDataFetcher()
        
        # Initialize OpenAI client
        self.openai_client = OpenAI()
    
    async def process_message(self, user_id: str, message: str) -> str:
        """Zpracování zprávy uživatele"""
        
        # Get user context and memory
        context = self.context_cache.get_user_context(user_id)
        memory = self.memory_system.get_user_memory(user_id, message)
        
        # Check prompt cache
        prompt_hash = self.prompt_cache.generate_prompt_hash(message, context)
        if cached_response := self.prompt_cache.get_cached_response(prompt_hash):
            return cached_response
        
        # Retrieve relevant documents
        documents = self.rag_system.retrieve_documents(message)
        
        # Build context for LLM
        llm_context = self.build_llm_context(
            message, context, memory, documents
        )
        
        # Generate response
        response = await self.generate_response(llm_context)
        
        # Add citations
        cited_response = self.citation_engine.add_citations(
            response, documents
        )
        
        # Cache response
        self.prompt_cache.cache_response(prompt_hash, cited_response)
        
        # Update user context and memory
        self.context_cache.update_user_context(user_id, {
            "last_message": message,
            "last_response": response
        })
        
        self.memory_system.store_conversation(user_id, {
            "user": message,
            "assistant": response,
            "timestamp": datetime.utcnow()
        })
        
        return cited_response
```

## Monitoring a Analytics

### Performance Monitoring

```python
class PerformanceMonitor:
    def __init__(self):
        self.metrics = {
            'response_time': [],
            'token_usage': [],
            'cache_hit_rate': 0,
            'error_rate': 0
        }
    
    def track_response_time(self, start_time: float, end_time: float):
        """Sledování času odpovědi"""
        response_time = end_time - start_time
        self.metrics['response_time'].append(response_time)
    
    def track_token_usage(self, input_tokens: int, output_tokens: int):
        """Sledování použití tokenů"""
        self.metrics['token_usage'].append({
            'input': input_tokens,
            'output': output_tokens,
            'total': input_tokens + output_tokens
        })
    
    def track_cache_hit(self, hit: bool):
        """Sledování cache hit rate"""
        if hit:
            self.metrics['cache_hit_rate'] += 1
    
    def get_metrics(self) -> dict:
        """Získání metrik"""
        return {
            'avg_response_time': np.mean(self.metrics['response_time']),
            'total_tokens': sum(t['total'] for t in self.metrics['token_usage']),
            'cache_hit_rate': self.metrics['cache_hit_rate'],
            'error_rate': self.metrics['error_rate']
        }
```

## Nákladová optimalizace

### Token Usage Optimization

```python
class TokenOptimizer:
    def __init__(self):
        self.compression_ratio = 0.7
        self.max_context_tokens = 8000
    
    def optimize_context(self, context: dict) -> dict:
        """Optimalizace kontextu pro úsporu tokenů"""
        if self.get_token_count(context) > self.max_context_tokens:
            # Compress context
            compressed = self.compress_context(context)
            return compressed
        return context
    
    def compress_context(self, context: dict) -> dict:
        """Komprese kontextu"""
        # Use GPT-4o-mini for compression
        compression_prompt = self.build_compression_prompt(context)
        compressed = self.openai_client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": compression_prompt}]
        )
        return {"compressed": compressed.choices[0].message.content}
```

## Bezpečnost a compliance

### GDPR Compliance

```python
class GDPRCompliance:
    def __init__(self):
        self.data_retention_days = 365
        self.anonymization_key = os.getenv('ANONYMIZATION_KEY')
    
    def anonymize_user_data(self, user_id: str, data: dict) -> dict:
        """Anonymizace uživatelských dat"""
        anonymized = data.copy()
        anonymized['user_id'] = self.hash_user_id(user_id)
        return anonymized
    
    def delete_user_data(self, user_id: str):
        """Smazání dat uživatele (GDPR right to be forgotten)"""
        # Delete from all systems
        self.cosmos_db.delete_user_data(user_id)
        self.redis.delete_user_data(user_id)
        self.vector_db.delete_user_data(user_id)
    
    def export_user_data(self, user_id: str) -> dict:
        """Export dat uživatele (GDPR data portability)"""
        return {
            'conversations': self.cosmos_db.get_user_conversations(user_id),
            'preferences': self.cosmos_db.get_user_preferences(user_id),
            'export_date': datetime.utcnow().isoformat()
        }
```

## Závěr

Rozšířená architektura poskytuje:

1. **RAG** - efektivní vyhledávání a citace
2. **MCP Server** - robustní databázové operace
3. **Prompt Caching** - úspora nákladů
4. **Context Caching** - optimalizace paměti
5. **Memory System** - dlouhodobá paměť
6. **Citation Engine** - transparentnost zdrojů
7. **Web Data Fetcher** - aktuální informace

Toto řešení je připraveno pro profesionální nasazení s vysokou dostupností, škálovatelností a nákladovou efektivitou.