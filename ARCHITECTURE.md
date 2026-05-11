# SnapAI Architecture

Comprehensive technical documentation of the SnapAI system architecture, design patterns, and implementation details.

## System Overview

SnapAI is a modern, scalable real-time AI assistant platform built with:
- **Frontend**: React 18 with TypeScript
- **Backend**: FastAPI with async support
- **Database**: PostgreSQL for persistence
- **Cache**: Redis for performance
- **NLP**: Transformer-based models from Hugging Face

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   Frontend Layer                        │
│        React 18 | Zustand | WebSocket | Tailwind        │
└────────────────────────┬────────────────────────────────┘
                         │ HTTP/WS
┌────────────────────────▼────────────────────────────────┐
│                  API Gateway Layer                      │
│          FastAPI | CORS | Rate Limiting | Auth          │
└────────────────────────┬────────────────────────────────┘
                         │
          ┌──────────────┼──────────────┐
          │              │              │
    ┌─────▼──────┐  ┌───▼────┐  ┌──────▼─────┐
    │   Services │  │  NLP   │  │  External  │
    │   Layer    │  │ Engine │  │ Services   │
    └─────┬──────┘  └───┬────┘  └──────┬─────┘
          │              │              │
    ┌─────▼──────────────┼──────────────▼─────┐
    │       Data Access Layer                 │
    │  SQLAlchemy ORM | Query Builder | Cache │
    └─────┬──────────────┼──────────────┬─────┘
          │              │              │
    ┌─────▼──┐    ┌──────▼──┐    ┌──────▼──┐
    │   DB   │    │  Cache  │    │  Logs   │
    │   PG   │    │  Redis  │    │   ELK   │
    └────────┘    └─────────┘    └─────────┘
```

## Data Models

### User Model
```
User
├── id (UUID) - Primary key
├── username (str, unique)
├── email (str, unique)
├── password_hash (str)
├── full_name (str)
├── avatar_url (str, nullable)
├── is_active (bool)
├── created_at (datetime)
└── updated_at (datetime)

Relationships:
└── conversations (1 → many)
└── messages (1 → many)
```

### Conversation Model
```
Conversation
├── id (UUID) - Primary key
├── user_id (UUID) - Foreign key
├── title (str)
├── description (str, nullable)
├── metadata (JSON) - Extra data
├── created_at (datetime)
├── updated_at (datetime)
└── last_message_at (datetime, nullable)

Relationships:
├── user (many → 1)
└── messages (1 → many)
```

### Message Model
```
Message
├── id (UUID) - Primary key
├── conversation_id (UUID) - Foreign key
├── user_id (UUID) - Foreign key
├── content (str)
├── role (enum: 'user'|'assistant'|'system')
├── sentiment (float) - -1.0 to 1.0
├── intent (str, nullable)
├── metadata (JSON)
├── created_at (datetime)
└── updated_at (datetime)

Relationships:
├── conversation (many → 1)
└── user (many → 1)
```

### Skill Model
```
Skill
├── id (UUID) - Primary key
├── name (str, unique)
├── description (str)
├── handler (str) - Module path
├── is_active (bool)
├── created_at (datetime)
└── updated_at (datetime)
```

### Integration Model
```
Integration
├── id (UUID) - Primary key
├── user_id (UUID) - Foreign key
├── service_name (str)
├── config (JSON) - Service configuration
├── api_key_encrypted (str)
├── is_active (bool)
├── created_at (datetime)
└── updated_at (datetime)

Relationships:
└── user (many → 1)
```

## API Request/Response Flow

### Chat Message Flow

```
1. Client sends message via WebSocket
   {
     "type": "message",
     "content": "What's the weather?",
     "conversation_id": "uuid"
   }

2. Backend receives and validates

3. NLP Engine processes
   ├── Sentiment Analysis
   ├── Intent Classification
   └── Entity Extraction

4. Context Manager retrieves history
   └── Last 5 messages for context

5. Skill Engine processes
   ├── Matches intent to skill
   └── Executes handler

6. Response generated
   {
     "type": "response",
     "content": "It's sunny and 72°F",
     "metadata": {
       "intent": "weather_query",
       "sentiment": 0.5
     }
   }

7. Message saved to database

8. Response sent to client
```

## NLP Pipeline

```
Input Text
    │
    ▼
┌────────────────────┐
│ Sentiment Analysis │ (DistilBERT)
├────────────────────┤
│ Output: -1.0 to 1.0│
└────────────────────┘
    │
    ▼
┌────────────────────┐
│ Intent Detection   │ (Zero-shot)
├────────────────────┤
│ Output: intent_name│
└────────────────────┘
    │
    ▼
┌────────────────────┐
│ Entity Extraction  │ (Named Entity Recognition)
├────────────────────┤
│ Output: entities[] │
└────────────────────┘
    │
    ▼
 Final NLP State
```

### NLP Models

**Sentiment Analysis**
- Model: `distilbert-base-uncased-finetuned-sst-2-english`
- Input: Text
- Output: Sentiment (0-1)
- Latency: ~50ms

**Intent Classification**
- Model: `facebook/bart-large-mnli`
- Method: Zero-shot classification
- Input: Text + candidate labels
- Output: Intent + confidence
- Latency: ~100ms

**Entity Extraction**
- Model: `dslim/bert-base-NER`
- Input: Text
- Output: Entities with types
- Latency: ~80ms

## Authentication Flow

```
1. User sends credentials
   POST /api/users/login
   {
     "username": "user",
     "password": "pass"
   }

2. Backend validates
   ├── Check user exists
   ├── Verify password hash
   └── Check is_active

3. Generate JWT token
   ├── Payload: user_id, exp
   ├── Signing: HS256 algorithm
   └── Secret: environment variable

4. Return token
   {
     "access_token": "eyJ...",
     "token_type": "bearer",
     "expires_in": 1800
   }

5. Client stores token

6. Subsequent requests include
   Authorization: Bearer eyJ...

7. Backend validates token
   ├── Verify signature
   ├── Check expiration
   └── Extract user_id
```

## Caching Strategy

### Redis Cache Layers

```
Cache Hierarchy:
┌──────────────────┐
│   User Cache     │ (60 min TTL)
│  user:{user_id}  │
└──────────────────┘
         │
┌──────────────────┐
│ Session Cache    │ (30 min TTL)
│  session:{sid}   │
└──────────────────┘
         │
┌──────────────────┐
│  Query Cache     │ (15 min TTL)
│ query:{hash}     │
└──────────────────┘
```

### Cache Invalidation

```
On Update:
├── Invalidate user cache
├── Invalidate related sessions
└── Invalidate query cache

On Message:
├── Update conversation cache
└── Update user activity cache
```

## WebSocket Protocol

### Connection Flow

```
1. Client initiates WS connection
   ws://localhost:8000/api/chat/ws

2. Server accepts connection

3. Client sends hello message
   {
     "type": "hello",
     "conversation_id": "uuid",
     "token": "jwt_token"
   }

4. Server validates and confirms
   {
     "type": "connected",
     "status": "ready"
   }
```

### Message Types

```
CLIENT → SERVER:

{
  "type": "message",
  "content": "User input",
  "conversation_id": "uuid",
  "metadata": {}
}

{
  "type": "typing",
  "conversation_id": "uuid"
}

{
  "type": "ping"
}

SERVER → CLIENT:

{
  "type": "response",
  "content": "AI response",
  "metadata": {
    "intent": "...",
    "sentiment": 0.5
  }
}

{
  "type": "typing",
  "user": "assistant"
}

{
  "type": "error",
  "message": "Error description"
}

{
  "type": "pong"
}
```

## Error Handling

### Error Response Format

```json
{
  "detail": {
    "error": "error_code",
    "message": "Human readable message",
    "timestamp": "2026-05-11T00:00:00Z",
    "path": "/api/endpoint"
  }
}
```

### Error Codes

```
400 - Bad Request (validation error)
401 - Unauthorized (auth required)
403 - Forbidden (no permission)
404 - Not Found (resource not found)
409 - Conflict (resource exists)
429 - Too Many Requests (rate limit)
500 - Internal Server Error
503 - Service Unavailable
```

## Security Considerations

### Password Security
```
Input: password
  ↓
Bcrypt (rounds=12)
  ↓
Hash: $2b$12$...
```

### Token Security
```
JWT Payload:
{
  "user_id": "uuid",
  "exp": timestamp,
  "iat": timestamp
}

Signed with: SECRET_KEY
Algorithm: HS256
```

### CORS Policy
```
Allowed Origins: [configured via env]
Allowed Methods: [GET, POST, PUT, DELETE]
Allowed Headers: [Content-Type, Authorization]
Allow Credentials: true
```

## Monitoring & Observability

### Logging

```
Log Levels:
├── DEBUG - Development details
├── INFO - General information
├── WARNING - Warnings (should investigate)
├── ERROR - Errors (action needed)
└── CRITICAL - Critical (immediate action)

Log Format:
timestamp | level | module | message
```

### Metrics

```
Track:
├── Request count
├── Response time
├── Error rate
├── Active users
├── Message throughput
├── Cache hit ratio
└── Database query time
```

### Health Checks

```
GET /health
Response: {"status": "healthy"}

Components checked:
├── Database connectivity
├── Redis connectivity
├── NLP models loaded
└── API responsiveness
```

## Scalability Patterns

### Horizontal Scaling

```
Load Balancer
    │
    ├─→ Backend Pod 1
    ├─→ Backend Pod 2
    └─→ Backend Pod 3
    
Shared:
├── PostgreSQL (replicated)
├── Redis (cluster)
└── NLP Models (shared volume)
```

### Database Optimization

```
Indexes:
├── user_id on conversations
├── user_id, created_at on messages
├── conversation_id on messages
└── created_at on all tables

Connection Pooling:
├── Min: 5
├── Max: 20
└── Timeout: 30s
```

## Deployment Architecture

### Docker Compose Services

```
Frontend (Node 18):
├── Port: 3000
├── Volume: src/
└── Depends on: Backend

Backend (Python 3.11):
├── Port: 8000
├── Volume: /app/
├── Depends on: PostgreSQL, Redis

PostgreSQL:
├── Port: 5432
├── Volume: /var/lib/postgresql/data
└── Database: snapai

Redis:
├── Port: 6379
└── Volume: /data
```

### Environment Configuration

```
Development:
├── LOG_LEVEL: DEBUG
├── ENV: development
└── DEBUG: true

Production:
├── LOG_LEVEL: INFO
├── ENV: production
├── DEBUG: false
└── HTTPS: required
```

## Performance Optimization

### Backend Optimization
- Async/await for I/O operations
- Connection pooling for DB
- Redis caching layer
- Batch query operations
- NLP model quantization

### Frontend Optimization
- Code splitting
- Lazy loading
- Component memoization
- Bundle size optimization
- Image compression

### Database Optimization
- Query indexing
- Connection pooling
- Read replicas
- Partition large tables
- Archive old messages

---

See [SETUP.md](./SETUP.md) for deployment instructions.
