# Setup Guide

Complete setup and configuration instructions for SnapAI.

## Prerequisites

- **Docker & Docker Compose** (easiest) or
- **Python 3.11+** and **Node.js 18+** (manual)
- Git
- 2GB RAM minimum
- Internet connection (for downloading models on first run)

## Option 1: Docker Compose (Recommended)

### Quick Start

```bash
# Clone the repository
git clone https://github.com/anam190/snapai.git
cd snapai

# Start all services
docker-compose up -d

# Check logs
docker-compose logs -f
```

Services will be available at:
- Frontend: http://localhost:3000
- Backend: http://localhost:8000
- API Docs: http://localhost:8000/docs

### Stopping Services

```bash
docker-compose down
```

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f backend
docker-compose logs -f frontend
```

## Option 2: Manual Setup

### Backend Setup

#### 1. Create Virtual Environment

```bash
cd backend
python -m venv venv

# Activate (Linux/Mac)
source venv/bin/activate

# Activate (Windows)
venv\Scripts\activate
```

#### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

#### 3. Database Setup

Set up PostgreSQL and create database:

```bash
# macOS with Homebrew
brew install postgresql@15
brew services start postgresql@15
createdb snapai

# Linux (Ubuntu/Debian)
sudo apt-get install postgresql
sudo -u postgres createdb snapai
sudo -u postgres createuser -P snapai

# Windows - Use PostgreSQL installer
```

#### 4. Environment Configuration

Create `.env` file in backend directory:

```bash
cp .env.example .env
```

Edit `.env` with your settings:

```
DATABASE_URL=postgresql://snapai:snapai@localhost:5432/snapai
REDIS_URL=redis://localhost:6379/0
HOST=0.0.0.0
PORT=8000
SECRET_KEY=your-secret-key-change-in-production
CORS_ORIGINS=http://localhost:3000
```

#### 5. Start Redis (Optional but Recommended)

```bash
# macOS with Homebrew
brew install redis
brew services start redis

# Docker
docker run -d -p 6379:6379 redis:7

# Linux
sudo apt-get install redis-server
sudo systemctl start redis-server
```

#### 6. Run Backend

```bash
# Development
uvicorn app:app --reload

# Production
gunicorn -w 4 -k uvicorn.workers.UvicornWorker app:app
```

Backend runs at: http://localhost:8000

### Frontend Setup

#### 1. Install Dependencies

```bash
cd frontend
npm install
```

#### 2. Environment Configuration

Create `.env.local`:

```
VITE_API_URL=http://localhost:8000
VITE_WS_URL=ws://localhost:8000
```

#### 3. Start Development Server

```bash
npm run dev
```

Frontend runs at: http://localhost:3000

## Database Initialization

### Create Tables

Option 1: Automatic (with alembic migrations)
```bash
cd backend
alembic upgrade head
```

Option 2: Manual (using SQLAlchemy)
```python
from database import engine, Base
from models import *

async def init_db():
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)

# Run: asyncio.run(init_db())
```

## Configuration

### Backend Configuration

Key environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_URL` | postgresql://... | PostgreSQL connection string |
| `REDIS_URL` | redis://localhost | Redis connection string |
| `SECRET_KEY` | your-key | JWT signing key |
| `ALGORITHM` | HS256 | JWT algorithm |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | 30 | Token expiration time |
| `CORS_ORIGINS` | http://localhost:3000 | Allowed CORS origins |
| `LOG_LEVEL` | INFO | Logging level |
| `HOST` | 0.0.0.0 | Server host |
| `PORT` | 8000 | Server port |

### Frontend Configuration

Key variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `VITE_API_URL` | http://localhost:8000 | Backend API URL |
| `VITE_WS_URL` | ws://localhost:8000 | WebSocket URL |

## Troubleshooting

### Port Already in Use

```bash
# Find and kill process
lsof -ti:8000  # Check port 8000
lsof -ti:3000  # Check port 3000

# Kill process
kill -9 <PID>

# Or use alternative port
export PORT=8001
```

### Database Connection Error

```bash
# Check PostgreSQL is running
psql -U snapai -d snapai -h localhost

# Check connection string
echo $DATABASE_URL

# Verify credentials
psql -U postgres -h localhost -c "CREATE DATABASE snapai;"
```

### Models Not Loading (NLP)

```bash
# First run requires internet to download models
# Check internet connection

# Verify transformers installation
python -c "from transformers import pipeline; print(pipeline)"

# Clear cache and retry
rm -rf ~/.cache/huggingface/
```

### Docker Issues

```bash
# Rebuild containers
docker-compose down -v
docker-compose build --no-cache
docker-compose up -d

# Check container status
docker ps
docker logs container_name
```

### WebSocket Connection Failed

```bash
# Ensure backend is running
curl http://localhost:8000/health

# Check CORS configuration
# Update CORS_ORIGINS in .env

# Verify WebSocket endpoint
curl http://localhost:8000/docs
```

## Testing

### Run Backend Tests

```bash
cd backend
pytest

# With coverage
pytest --cov=. --cov-report=html
```

### Run Frontend Tests

```bash
cd frontend
npm run test
```

## Performance Tuning

### Backend

```python
# Increase worker processes (production)
gunicorn -w 4 -k uvicorn.workers.UvicornWorker app:app

# Enable caching
REDIS_URL=redis://localhost:6379
```

### Database

```sql
-- Create indexes for common queries
CREATE INDEX idx_messages_conversation ON messages(conversation_id);
CREATE INDEX idx_messages_user ON messages(user_id);
CREATE INDEX idx_conversations_user ON conversations(user_id);
```

### Frontend

```bash
# Production build
npm run build

# Analyze bundle
npm run analyze
```

## Deployment

### Production Checklist

- [ ] Update `SECRET_KEY` to random value
- [ ] Set `ENV=production`
- [ ] Configure `CORS_ORIGINS` appropriately
- [ ] Enable HTTPS
- [ ] Set up database backups
- [ ] Configure monitoring/logging
- [ ] Set resource limits
- [ ] Enable rate limiting

### Deploy to Heroku

```bash
git push heroku main
```

### Deploy to AWS

See AWS deployment guide (coming soon)

## Monitoring

### Logs

```bash
# Backend
docker-compose logs -f backend

# Frontend
docker-compose logs -f frontend

# Database
docker-compose logs -f postgres
```

### Health Checks

```bash
# Backend health
curl http://localhost:8000/health

# API documentation
http://localhost:8000/docs

# Frontend
http://localhost:3000
```

---

For additional support, please open a GitHub issue or refer to the README.
