# Banking Policy RAG Application - Deployment & Architecture Guide

## 1. System Architecture

### 1.1 High-Level Architecture Diagram

```
User Browser (TypeScript/React)
         │
         │ HTTP/REST (JSON)
         │
    ┌────▼─────────────────┐
    │   Load Balancer      │
    │ (Production: Nginx)  │
    └────┬─────────────────┘
         │
    ┌────▼──────────────────────────────────┐
    │  FastAPI Backend (Python)             │
    │  ┌──────────────────────────────────┐│
    │  │ Query Handler                   ││
    │  │ - Input validation              ││
    │  │ - Rate limiting                 ││
    │  │ - Error handling                ││
    │  └──────────────────────────────────┘│
    │                                      │
    │  ┌──────────────────────────────────┐│
    │  │ RAG Service (LangChain)         ││
    │  │ - Embedding generation          ││
    │  │ - Vector similarity search      ││
    │  │ - Prompt building               ││
    │  │ - LLM response generation       ││
    │  └──────────────────────────────────┘│
    └────┬──────────────────────────────────┘
         │
    ┌────┴─────────────┬──────────────────┐
    │                  │                  │
    ▼                  ▼                  ▼
┌────────────┐   ┌────────────────┐   ┌──────────────┐
│ Chroma DB  │   │ Hugging Face   │   │  Logging &   │
│ (Vector    │   │ (LLM Service)  │   │  Monitoring  │
│  Storage)  │   │                │   │              │
│ ┌────────┐ │   │ google/gemma-  │   │ - ELK Stack  │
│ │Embeddings│ │   │ 4-31B-it       │   │ - Prometheus │
│ └────────┘ │   │                │   │ - Grafana    │
│ ┌────────┐ │   └────────────────┘   └──────────────┘
│ │Metadata│ │
│ └────────┘ │
└────────────┘
```

### 1.2 Component Interactions

```
Query Lifecycle:

1. User submits question via React frontend
   │
2. Frontend sends HTTP POST to /api/query
   │
3. FastAPI receives request
   ├─ Validates input
   ├─ Applies rate limiting
   │
4. RAG Service processes query
   ├─ Generates embedding using HuggingFaceEmbeddings
   │
5. Vector Store Operation
   ├─ Searches Chroma DB for similar chunks
   ├─ Retrieves Top-K (3-5) most relevant chunks
   │
6. LLM Processing
   ├─ Constructs prompt with context
   ├─ Sends to Google Gemma 4 via HuggingFace API
   ├─ Receives streamed response
   │
7. Response Formatting
   ├─ Formats answer
   ├─ Attaches source citations
   ├─ Includes metadata (confidence, time, etc.)
   │
8. Frontend receives and displays response
   ├─ Renders formatted answer
   ├─ Shows source citations
   └─ Displays metadata
```

---

## 2. Development Environment Setup

### 2.1 Prerequisites
- Python 3.9+
- Node.js 16+
- Docker & Docker Compose
- Git
- Hugging Face API Key

### 2.2 Local Development Setup

#### Backend Setup
```bash
# Clone repository
git clone <repo-url>
cd RAG_chatbot/backend

# Create virtual environment
python -m venv venv

# Activate virtual environment
# Windows:
venv\Scripts\activate
# Mac/Linux:
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Create .env file
cp .env.example .env
# Edit .env and add your Hugging Face API key

# Run development server
python -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

#### Frontend Setup
```bash
cd RAG_chatbot/frontend

# Install dependencies
npm install

# Create .env.local file
echo "REACT_APP_API_URL=http://localhost:8000/api" > .env.local

# Start development server
npm start
```

---

## 3. Docker Deployment

### 3.1 Backend Dockerfile

```dockerfile
# syntax=docker/dockerfile:1
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY app/ ./app/

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:8000/api/health')"

# Run application
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 3.2 Frontend Dockerfile

```dockerfile
# Build stage
FROM node:18-alpine as builder

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

ENV REACT_APP_API_URL=/api

RUN npm run build

# Production stage
FROM node:18-alpine

WORKDIR /app

RUN npm install -g serve

COPY --from=builder /app/build ./build

EXPOSE 3000

CMD ["serve", "-s", "build", "-l", "3000"]
```

### 3.3 Docker Compose Configuration

```yaml
version: '3.8'

services:
  backend:
    build:
      context: ./backend
      dockerfile: docker/Dockerfile
    container_name: rag_backend
    ports:
      - "8000:8000"
    environment:
      - HUGGINGFACE_API_KEY=${HUGGINGFACE_API_KEY}
      - CHROMA_DB_PATH=/app/data/chroma_db
      - LOG_LEVEL=INFO
    volumes:
      - ./backend/data:/app/data
      - ./backend/logs:/app/logs
    networks:
      - rag_network
    restart: unless-stopped
    depends_on:
      - chroma

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: rag_frontend
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:8000/api
    networks:
      - rag_network
    restart: unless-stopped
    depends_on:
      - backend

  chroma:
    image: ghcr.io/chroma-core/chroma:latest
    container_name: chroma_db
    ports:
      - "8001:8000"
    environment:
      - IS_PERSISTENT=true
      - PERSIST_DIRECTORY=/data/chroma
    volumes:
      - chroma_data:/data/chroma
    networks:
      - rag_network
    restart: unless-stopped

volumes:
  chroma_data:

networks:
  rag_network:
    driver: bridge
```

### 3.4 Running with Docker Compose

```bash
# Build and start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down

# Stop and remove data
docker-compose down -v
```

---

## 4. Production Deployment

### 4.1 AWS Deployment (ECS + CloudFront)

#### Architecture
```
Internet
    │
    ▼
CloudFront CDN
    │ (Caches frontend assets)
    │
    ├─► S3 Bucket (Frontend static files)
    │
    └─► Application Load Balancer
        │
        ├─► ECS Task (FastAPI Backend)
        ├─► ECS Task (FastAPI Backend)
        └─► ECS Task (FastAPI Backend)
             │
             ├─► RDS (Optional: for metadata)
             └─► EFS (Chroma DB persistent storage)
                 │
                 └─► ECR (Container registry)
```

#### Deployment Steps

1. **Create ECR Repository**
   ```bash
   aws ecr create-repository --repository-name rag-backend
   aws ecr create-repository --repository-name rag-frontend
   ```

2. **Push Docker Images**
   ```bash
   # Build and tag
   docker build -t rag-backend:latest ./backend
   docker tag rag-backend:latest <aws-account-id>.dkr.ecr.<region>.amazonaws.com/rag-backend:latest
   
   # Push to ECR
   aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws-account-id>.dkr.ecr.<region>.amazonaws.com
   docker push <aws-account-id>.dkr.ecr.<region>.amazonaws.com/rag-backend:latest
   ```

3. **Create ECS Cluster and Services**
   ```bash
   # Create cluster
   aws ecs create-cluster --cluster-name rag-cluster
   
   # Register task definition
   aws ecs register-task-definition --cli-input-json file://task-definition.json
   
   # Create service
   aws ecs create-service \
     --cluster rag-cluster \
     --service-name rag-backend-service \
     --task-definition rag-backend:1 \
     --desired-count 3 \
     --launch-type FARGATE
   ```

4. **Configure Application Load Balancer**
   - Create ALB in EC2 dashboard
   - Create target groups for backend services
   - Configure listener rules

5. **Setup CloudFront Distribution**
   - Create distribution for frontend S3 bucket
   - Configure caching policies
   - Setup SSL/TLS certificates

### 4.2 GCP Deployment (Cloud Run)

```yaml
# clouddeploy.yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: rag-backend
spec:
  template:
    spec:
      containers:
        - image: gcr.io/PROJECT_ID/rag-backend:latest
          ports:
            - containerPort: 8000
          env:
            - name: HUGGINGFACE_API_KEY
              valueFrom:
                secretKeyRef:
                  name: hf-api-key
                  key: api-key
          resources:
            limits:
              cpu: "2"
              memory: "4Gi"
      serviceAccountName: rag-service-account
```

### 4.3 Azure Deployment (Container Instances)

```yaml
# azure-container-instances.yaml
apiVersion: 2019-12-01
name: rag-backend-container
properties:
  containers:
    - name: rag-backend
      properties:
        image: containerregistry.azurecr.io/rag-backend:latest
        resources:
          requests:
            cpu: 2.0
            memoryInGB: 4.0
        ports:
          - port: 8000
        environmentVariables:
          - name: HUGGINGFACE_API_KEY
            secureValue: <YOUR_API_KEY>
  osType: Linux
  ipAddress:
    type: Public
    ports:
      - protocol: TCP
        port: 8000
  imageRegistryCredentials:
    - server: containerregistry.azurecr.io
      username: <USERNAME>
      password: <PASSWORD>
```

---

## 5. Database & Storage

### 5.1 Chroma DB Persistence

#### Local Development (In-Memory)
```python
# app/config.py
from chromadb.config import Settings

if DEBUG:
    # In-memory for development
    chroma_settings = Settings(
        is_persistent=False,
        anonymized_telemetry=False
    )
```

#### Production (Persistent)
```python
# app/config.py
chroma_settings = Settings(
    is_persistent=True,
    persist_directory="/data/chroma_db",
    anonymized_telemetry=False,
    chroma_db_impl="duckdb+parquet"
)
```

#### Database Backup Strategy
```bash
# Backup Chroma DB
tar -czf chroma_backup_$(date +%Y%m%d_%H%M%S).tar.gz /data/chroma_db

# Restore from backup
tar -xzf chroma_backup_20240408_120000.tar.gz -C /data
```

### 5.2 Metadata Storage (Optional PostgreSQL)

For production, consider adding PostgreSQL for:
- Query history
- User feedback
- Document audit trails
- Performance analytics

```sql
-- schema.sql
CREATE TABLE documents (
    id UUID PRIMARY KEY,
    title VARCHAR(255),
    policy_type VARCHAR(100),
    upload_date TIMESTAMP,
    effective_date DATE,
    expires_date DATE,
    chunks INT,
    tokens INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE queries (
    id UUID PRIMARY KEY,
    question TEXT,
    answer TEXT,
    source_docs TEXT[],
    response_time_ms INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE user_feedback (
    id UUID PRIMARY KEY,
    query_id UUID REFERENCES queries(id),
    rating INT (1-5),
    feedback TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 6. Monitoring & Logging

### 6.1 Backend Logging Configuration

```python
# app/utils/logger.py
import logging
from pythonjsonlogger import jsonlogger

def setup_logger():
    logger = logging.getLogger()
    logHandler = logging.StreamHandler()
    
    formatter = jsonlogger.JsonFormatter(
        "(timestamp) (level) (name) (message)"
    )
    logHandler.setFormatter(formatter)
    logger.addHandler(logHandler)
    logger.setLevel(logging.INFO)
    
    return logger

logger = setup_logger()
```

### 6.2 Prometheus Metrics

```python
# app/utils/metrics.py
from prometheus_client import Counter, Histogram, Gauge

# Query metrics
query_counter = Counter(
    'rag_queries_total',
    'Total number of queries',
    ['status']
)

query_duration = Histogram(
    'rag_query_duration_seconds',
    'Query duration in seconds'
)

active_queries = Gauge(
    'rag_queries_active',
    'Currently active queries'
)

# Vector store metrics
embedding_duration = Histogram(
    'rag_embedding_duration_seconds',
    'Embedding generation duration'
)

vector_store_size = Gauge(
    'rag_vector_store_size',
    'Number of vectors in store'
)
```

### 6.3 Monitoring Stack (Docker Compose)

```yaml
prometheus:
  image: prom/prometheus:latest
  volumes:
    - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
  ports:
    - "9090:9090"

grafana:
  image: grafana/grafana:latest
  ports:
    - "3001:3000"
  environment:
    - GF_SECURITY_ADMIN_PASSWORD=admin

elasticsearch:
  image: docker.elastic.co/elasticsearch/elasticsearch:8.0.0
  environment:
    - discovery.type=single-node
  ports:
    - "9200:9200"

kibana:
  image: docker.elastic.co/kibana/kibana:8.0.0
  ports:
    - "5601:5601"
```

---

## 7. CI/CD Pipeline

### 7.1 GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy RAG Application

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          pip install -r backend/requirements.txt
          pip install pytest pytest-cov
      
      - name: Run tests
        run: pytest backend/tests/
      
      - name: Upload coverage
        uses: codecov/codecov-action@v2

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v2
      
      - name: Build Docker images
        run: |
          docker build -t rag-backend:latest ./backend
          docker build -t rag-frontend:latest ./frontend
      
      - name: Push to registry
        env:
          REGISTRY: ghcr.io
          IMAGE_NAME: ${{ github.repository }}
        run: |
          echo ${{ secrets.GITHUB_TOKEN }} | docker login $REGISTRY -u ${{ github.actor }} --password-stdin
          docker tag rag-backend:latest $REGISTRY/$IMAGE_NAME/rag-backend:latest
          docker push $REGISTRY/$IMAGE_NAME/rag-backend:latest

  deploy:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to production
        run: |
          # Deployment script
          echo "Deploying to production..."
```

---

## 8. Security Considerations

### 8.1 Environment Variables
```bash
# Never commit secrets; use environment files
HUGGINGFACE_API_KEY=<your_key>
SECRET_KEY=<your_secret>
DATABASE_URL=<db_url>
```

### 8.2 API Security Headers

```python
# app/middleware/auth.py
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.trustedhost import TrustedHostMiddleware

# Add CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://example.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Trust proxy headers
app.add_middleware(
    TrustedHostMiddleware,
    allowed_hosts=["example.com"]
)
```

### 8.3 Input Validation
```python
from pydantic import BaseModel, Field, validator

class QueryRequest(BaseModel):
    question: str = Field(..., min_length=3, max_length=1000)
    
    @validator('question')
    def validate_question(cls, v):
        # Remove potential injection attempts
        return v.strip()
```

---

## 9. Performance Optimization

### 9.1 Caching Strategy

```python
# Cache frequently queried policy sections
from functools import lru_cache

@lru_cache(maxsize=1024)
def get_cached_embedding(text: str):
    return embeddings_model.embed_query(text)
```

### 9.2 Async Processing

```python
# Use async for I/O bound operations
from fastapi import BackgroundTasks

@app.post("/api/documents/upload")
async def upload_document(file: UploadFile, background_tasks: BackgroundTasks):
    # Store file reference
    document_id = save_file(file)
    
    # Index in background
    background_tasks.add_task(index_document, document_id)
    
    return {"document_id": document_id, "status": "queued"}
```

### 9.3 Connection Pooling

```python
# Use connection pools for database connections
from sqlalchemy import create_engine

engine = create_engine(
    DATABASE_URL,
    poolclass=QueuePool,
    pool_size=20,
    max_overflow=40
)
```

---

## 10. Disaster Recovery

### 10.1 Backup Strategy
- Daily automated backups of Chroma DB
- Weekly full backups to S3/cloud storage
- Point-in-time recovery capability

### 10.2 Failover Plan
- Multi-region deployment
- Auto-scaling groups
- Health checks and automatic restarts
- Load balancing across instances

---

## 11. Cost Optimization

| Component | Dev Cost | Prod Cost |
|-----------|----------|-----------|
| Hugging Face API | Free tier | ~$0.01 per query |
| Cloud Infrastructure | Free/~$5/mo | ~$200-500/mo |
| Storage | Local | ~$20-50/mo |
| Monitoring | Free | ~$50-100/mo |
| **Total Monthly** | **~$5** | **~$270-650** |

---

## 12. Checklist Before Production Deployment

- [ ] All tests passing
- [ ] Security vulnerabilities scanned
- [ ] API rate limiting configured
- [ ] Logging and monitoring enabled
- [ ] Database backups tested
- [ ] SSL/TLS certificates configured
- [ ] Environment variables secured in vault
- [ ] Documentation complete
- [ ] Team trained on deployment process
- [ ] Incident response plan documented
- [ ] Performance benchmarks meet requirements
- [ ] Load testing completed successfully
