# Tier 2: FastAPI + Docker Deployment Guide (Railway)

This guide covers deploying the ShopSage FastAPI application with Docker to Railway.

## 🎯 Overview

FastAPI + Docker provides a production-ready API solution:
- RESTful API with automatic documentation
- Container-based deployment
- Easy scaling and management
- Professional API development experience

## 📋 Prerequisites

- UV installed (`curl -LsSf https://astral.sh/uv/install.sh | sh`)
- Docker Desktop installed
- Railway CLI installed
- GitHub account
- API keys (Tavily, OpenAI/Anthropic)

## 🐳 Local Development with Docker

### 1. Setup Environment

```bash
# Navigate to FastAPI directory
cd shopsage/tier2-fastapi

# Copy environment file
cp .env.example .env

# Edit .env with your API keys
nano .env
```

### 2. Build and Run with Docker Compose

```bash
# Build and start containers
docker-compose up --build

# Run in background
docker-compose up -d

# View logs
docker-compose logs -f

# Stop containers
docker-compose down
```

Your API will be available at:
- API: http://localhost:8000
- Interactive docs: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc

### 3. Test the API

```bash
# Health check
curl http://localhost:8000/health

# Get recommendation
curl -X POST http://localhost:8000/recommend \
  -H "Content-Type: application/json" \
  -d '{
    "question": "best laptop for programming under $1500"
  }'

# Search products
curl -X POST http://localhost:8000/search?query=gaming+mouse&max_results=5
```

## 🚂 Deploy to Railway

### 1. Install Railway CLI

```bash
# macOS
brew install railway

# Windows (PowerShell)
iwr -useb https://railway.app/install.sh | iex

# Linux
curl -fsSL https://railway.app/install.sh | sh
```

### 2. Prepare for Deployment

```bash
# Ensure you're in the FastAPI directory
cd shopsage/tier2-fastapi

# Copy core module to current directory
cp ../shopsage_core.py .

# Initialize git (if needed)
git init
git add .
git commit -m "Initial FastAPI ShopSage app"
```

### 3. Deploy to Railway

```bash
# Login to Railway
railway login

# Initialize new project
railway init

# When prompted:
# - Enter project name: shopsage-api
# - Select "Empty Project"

# Link to GitHub (recommended)
railway link

# Set environment variables
railway variables set TAVILY_API_KEY=tvly-xxxxxxxxxxxxx
railway variables set OPENAI_API_KEY=sk-xxxxxxxxxxxxx
railway variables set OPENAI_MODEL=gpt-4o-mini

# Deploy
railway up

# Get deployment URL
railway open
```

### 4. Alternative: Deploy via GitHub

1. Push to GitHub:
```bash
# Create GitHub repository
gh repo create shopsage-fastapi --public
git remote add origin https://github.com/YOUR_USERNAME/shopsage-fastapi.git
git push -u origin main
```

2. In Railway Dashboard:
   - Click "New Project"
   - Select "Deploy from GitHub repo"
   - Choose your repository
   - Railway auto-detects Dockerfile

3. Configure environment variables in Railway dashboard:
   - Go to Variables tab
   - Add each API key

## 🔧 Configuration

### Docker Configuration

Customize `Dockerfile` for production:
```dockerfile
# Multi-stage build for smaller image
FROM python:3.11-slim as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Railway Configuration

`railway.json` customization:
```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "DOCKERFILE",
    "dockerfilePath": "Dockerfile"
  },
  "deploy": {
    "numReplicas": 1,
    "healthcheckPath": "/health",
    "healthcheckTimeout": 10,
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 3,
    "cronSchedule": null,
    "region": "us-west1"
  }
}
```

## 📊 API Documentation

### Automatic Interactive Docs

FastAPI generates interactive documentation:
- Swagger UI: `https://your-app.railway.app/docs`
- ReDoc: `https://your-app.railway.app/redoc`

### Example API Calls

1. **Get Recommendation**
```bash
curl -X POST https://your-app.railway.app/recommend \
  -H "Content-Type: application/json" \
  -d '{
    "question": "best mechanical keyboard for coding"
  }'
```

2. **Search Products**
```bash
curl -X POST "https://your-app.railway.app/search?query=noise+cancelling+headphones&max_results=10"
```

3. **Analyze Products**
```bash
curl -X POST https://your-app.railway.app/analyze \
  -H "Content-Type: application/json" \
  -d '{
    "question": "Which is better for gaming?",
    "products": [...]
  }'
```

## 🛠️ Development Workflow

### Local Testing
```bash
# Run without Docker for faster development
uv sync
uv run uvicorn main:app --reload

# Run specific tests
pytest tests/test_api.py
```

### Docker Development
```bash
# Rebuild single service
docker-compose build shopsage-api

# Execute commands in container
docker-compose exec shopsage-api python -m pytest

# View container logs
docker logs shopsage-api -f
```

### Railway CLI Commands
```bash
# View logs
railway logs

# Open dashboard
railway open

# Run command in production
railway run python --version

# Restart deployment
railway restart

# View environment variables
railway variables
```

## 🔍 Troubleshooting

### Common Issues

1. **Port binding error**
```bash
# Check if port 8000 is in use
lsof -i :8000
# Kill process using port
kill -9 $(lsof -t -i:8000)
```

2. **Module import errors**
```bash
# Ensure shopsage_core.py is copied
cp ../shopsage_core.py .
# Or update Dockerfile to copy from parent
```

3. **Railway deployment fails**
```bash
# Check logs
railway logs

# Verify Dockerfile syntax
docker build -t test .

# Test locally first
docker run -p 8000:8000 test
```

4. **Environment variables not working**
```bash
# List all variables
railway variables

# Set variable with quotes if needed
railway variables set OPENAI_API_KEY="sk-xxxxx"
```

## 📈 Monitoring & Scaling

### Railway Metrics
- CPU usage
- Memory consumption
- Request count
- Response times

Access via Railway dashboard → Metrics tab

### Scaling Options
```bash
# Scale horizontally
railway scale --replicas 3

# Adjust resources in railway.json
"deploy": {
  "numReplicas": 3,
  "maxConcurrency": 100
}
```

### Add Logging
```python
import logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@app.post("/recommend")
async def recommend(query: ShoppingQuery):
    logger.info(f"Recommendation request: {query.question}")
    # ... rest of code
```

## 🎓 Teaching Notes

### Workshop Preparation
1. Pre-create Railway projects
2. Prepare Docker images
3. Test deployment pipeline
4. Have backup API keys

### Step-by-Step for Students
1. Start with local Docker
2. Test API with curl/Postman
3. Deploy to Railway
4. Monitor and debug

### Common Student Challenges
- Docker not installed → Use Docker Desktop
- Railway CLI issues → Use web dashboard
- API timeouts → Check rate limits
- CORS errors → Verify middleware config

## 🚀 Production Best Practices

### Security
```python
# Add rate limiting
from slowapi import Limiter
limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter

@app.post("/recommend")
@limiter.limit("10/minute")
async def recommend(query: ShoppingQuery):
    # ... code
```

### Error Handling
```python
# Global exception handler
@app.exception_handler(Exception)
async def global_exception_handler(request, exc):
    return JSONResponse(
        status_code=500,
        content={"detail": "Internal server error"}
    )
```

### Health Checks
```python
@app.get("/health")
async def health_check():
    # Add dependency checks
    checks = {
        "api": "healthy",
        "tavily": check_tavily_connection(),
        "llm": check_llm_connection()
    }
    return checks
```

## 🔗 Resources

- [FastAPI Documentation](https://fastapi.tiangolo.com)
- [Railway Documentation](https://docs.railway.app)
- [Docker Documentation](https://docs.docker.com)
- [FastAPI Best Practices](https://github.com/zhanymkanov/fastapi-best-practices)