# Project Structure & Configuration Templates

## 1. Project Tree Structure

```
RAG_chatbot/
│
├── README.md                           # Project overview and quick start
├── IMPLEMENTATION_PLAN.md             # Detailed implementation roadmap
├── API_SPECIFICATION.md               # Complete REST API documentation
├── DEPLOYMENT_ARCHITECTURE.md         # Deployment and operational guide
│
├── backend/
│   ├── venv/                          # Python virtual environment
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                    # FastAPI application entry point
│   │   ├── config.py                  # Configuration management
│   │   │
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   └── request_models.py      # Pydantic request/response models
│   │   │
│   │   ├── routes/
│   │   │   ├── __init__.py
│   │   │   ├── query.py               # Query endpoint handlers
│   │   │   └── documents.py           # Document management handlers
│   │   │
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── rag_service.py         # RAG pipeline orchestration
│   │   │   ├── embedding_service.py   # Embedding generation logic
│   │   │   ├── llm_service.py         # LLM interaction logic
│   │   │   └── vector_store.py        # Chroma DB operations
│   │   │
│   │   ├── utils/
│   │   │   ├── __init__.py
│   │   │   ├── document_loader.py     # PDF/document parsing
│   │   │   ├── text_processor.py      # Text chunking & preprocessing
│   │   │   └── logger.py              # Logging configuration
│   │   │
│   │   └── middleware/
│   │       ├── __init__.py
│   │       └── error_handler.py       # Error handling middleware
│   │
│   ├── tests/
│   │   ├── __init__.py
│   │   ├── conftest.py                # pytest configuration
│   │   ├── test_rag_service.py        # RAG service tests
│   │   ├── test_api_endpoints.py      # API endpoint tests
│   │   └── test_embedding.py          # Embedding tests
│   │
│   ├── data/
│   │   └── chroma_db/                 # Persistent Chroma DB storage
│   │
│   ├── logs/                          # Application logs
│   ├── docker/
│   │   ├── Dockerfile                 # Backend Docker image
│   │   └── .dockerignore
│   │
│   ├── requirements.txt               # Python dependencies
│   ├── .env.example                   # Environment variables template
│   └── .gitignore
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── QueryInput.tsx         # User query input component
│   │   │   ├── ResponseDisplay.tsx    # Response formatting and display
│   │   │   ├── DocumentUpload.tsx     # Document upload interface
│   │   │   ├── SourceCitations.tsx    # Source document display
│   │   │   ├── LoadingSpinner.tsx     # Loading indicator
│   │   │   └── ErrorBoundary.tsx      # Error handling component
│   │   │
│   │   ├── services/
│   │   │   ├── api.ts                 # API client with axios
│   │   │   └── queryService.ts        # Query business logic
│   │   │
│   │   ├── types/
│   │   │   └── index.ts               # TypeScript interfaces & types
│   │   │
│   │   ├── hooks/
│   │   │   ├── useQuery.ts            # Custom hook for queries
│   │   │   ├── useDocuments.ts        # Custom hook for documents
│   │   │   └── useFetch.ts            # Generic fetch hook
│   │   │
│   │   ├── pages/
│   │   │   ├── Chat.tsx               # Main query interface
│   │   │   ├── DocumentManager.tsx    # Document management page
│   │   │   ├── QueryHistory.tsx       # Query history page
│   │   │   └── Settings.tsx           # Settings page
│   │   │
│   │   ├── styles/
│   │   │   ├── index.css              # Global styles
│   │   │   ├── components.css         # Component-specific styles
│   │   │   └── tailwind.config.js     # Tailwind configuration
│   │   │
│   │   ├── App.tsx                    # Main app component
│   │   └── index.tsx                  # React entry point
│   │
│   ├── public/
│   │   ├── index.html                 # HTML template
│   │   └── favicon.ico
│   │
│   ├── package.json
│   ├── package-lock.json
│   ├── tsconfig.json                  # TypeScript configuration
│   ├── .env.local.example             # Environment variables template
│   └── .gitignore
│
├── docker-compose.yml                 # Multi-container orchestration
├── .gitignore
└── .env.example                       # Root environment variables

```

## 2. Configuration Files

### 2.1 Backend Requirements (requirements.txt)

```
# Core Framework
fastapi==0.108.0
uvicorn[standard]==0.27.0
pydantic==2.5.0
pydantic-settings==2.1.0

# LangChain & RAG
langchain==0.1.8
langchain-huggingface==0.0.23
langchain-chroma==0.1.0

# Vector Database
chromadb==0.4.21

# Embeddings
sentence-transformers==2.2.2

# LLM Integration
huggingface-hub==0.19.4
requests==2.31.0

# Document Processing
PyPDF2==3.0.1
python-multipart==0.0.6

# Database (Optional)
sqlalchemy==2.0.23
psycopg2-binary==2.9.9

# Utilities
python-dotenv==1.0.0
python-json-logger==2.0.7

# Testing
pytest==7.4.3
pytest-asyncio==0.21.1
pytest-cov==4.1.0
httpx==0.25.2

# Development
black==23.12.1
flake8==6.1.0
isort==5.13.2
mypy==1.7.1
```

### 2.2 Backend .env.example

```env
# ============================================
# HUGGING FACE CONFIGURATION
# ============================================
HUGGINGFACE_API_KEY=hf_your_api_key_here
HF_MODEL_ID=google/gemma-4-31B-it
HF_ENDPOINT_URL=https://api-inference.huggingface.co/models

# ============================================
# BACKEND CONFIGURATION
# ============================================
BACKEND_HOST=0.0.0.0
BACKEND_PORT=8000
DEBUG=true
ENVIRONMENT=development

# ============================================
# LOGGING CONFIGURATION
# ============================================
LOG_LEVEL=INFO
LOG_FILE=logs/app.log
LOG_FORMAT=json

# ============================================
# CHROMA DB CONFIGURATION
# ============================================
CHROMA_DB_PATH=./data/chroma_db
CHROMA_RESET=false
CHROMA_PERSISTENCE_ENABLED=true
CHROMA_ANONYMIZED_TELEMETRY=false

# ============================================
# RAG PIPELINE CONFIGURATION
# ============================================
# Chunk sizing for document processing
CHUNK_SIZE=1000
CHUNK_OVERLAP=200

# Retrieval parameters
TOP_K_RETRIEVAL=5
SIMILARITY_THRESHOLD=0.3

# Model parameters
EMBEDDING_MODEL_NAME=sentence-transformers/all-MiniLM-L6-v2
MAX_TOKENS=4096

# ============================================
# API CONFIGURATION
# ============================================
API_TITLE=Banking Policy RAG API
API_VERSION=1.0.0
API_DESCRIPTION=Retrieval-Augmented Generation API for Banking Policies

# CORS settings
CORS_ORIGINS=["http://localhost:3000","http://localhost:8080"]
CORS_ALLOW_CREDENTIALS=true
CORS_ALLOW_METHODS=["*"]
CORS_ALLOW_HEADERS=["*"]

# Rate limiting
RATE_LIMIT_PER_HOUR=100
RATE_LIMIT_PER_MINUTE=10

# ============================================
# DATABASE CONFIGURATION (Optional)
# ============================================
# DATABASE_URL=postgresql://user:password@localhost:5432/rag_db
# DATABASE_ECHO=false
# DATABASE_POOL_SIZE=20
# DATABASE_MAX_OVERFLOW=40

# ============================================
# SECURITY CONFIGURATION
# ============================================
SECRET_KEY=your-secret-key-here-change-in-production
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# ============================================
# DOCUMENT UPLOAD CONFIGURATION
# ============================================
MAX_UPLOAD_SIZE_MB=100
ALLOWED_FILE_TYPES=pdf,txt,docx
MAX_DOCUMENTS=1000
```

### 2.3 Frontend package.json

```json
{
  "name": "rag-banking-frontend",
  "version": "1.0.0",
  "description": "Banking Policy RAG Application Frontend",
  "private": true,
  "homepage": "./",
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-scripts": "5.0.1",
    "typescript": "^5.3.3",
    "axios": "^1.6.4",
    "react-markdown": "^9.0.1",
    "tailwindcss": "^3.4.1",
    "postcss": "^8.4.32",
    "autoprefixer": "^10.4.16",
    "zustand": "^4.4.2",
    "react-query": "^3.39.3",
    "react-router-dom": "^6.20.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.37",
    "@types/react-dom": "^18.2.15",
    "@types/node": "^20.10.5",
    "tailwindcss": "^3.4.1",
    "postcss": "^8.4.32",
    "autoprefixer": "^10.4.16",
    "eslint": "^8.56.0",
    "@typescript-eslint/eslint-plugin": "^6.15.0",
    "@typescript-eslint/parser": "^6.15.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "test:coverage": "react-scripts test --coverage --watchAll=false",
    "eject": "react-scripts eject",
    "lint": "eslint src/**/*.{ts,tsx}",
    "type-check": "tsc --noEmit"
  },
  "eslintConfig": {
    "extends": [
      "react-app"
    ]
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}
```

### 2.4 Frontend .env.local.example

```env
# Frontend Environment Configuration

# API Configuration
REACT_APP_API_URL=http://localhost:8000/api
REACT_APP_API_TIMEOUT=30000

# Feature Flags
REACT_APP_ENABLE_QUERY_HISTORY=true
REACT_APP_ENABLE_FEEDBACK=true
REACT_APP_ENABLE_ANALYTICS=false

# UI Configuration
REACT_APP_MAX_QUERY_LENGTH=1000
REACT_APP_MAX_RESPONSE_LENGTH=5000
REACT_APP_RESULTS_PER_PAGE=10

# Logging
REACT_APP_LOG_LEVEL=info
```

### 2.5 Docker Compose (docker-compose.yml)

```yaml
version: '3.8'

services:
  # Backend API Service
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
      - DEBUG=false
      - ENVIRONMENT=production
    volumes:
      - ./backend/data:/app/data
      - ./backend/logs:/app/logs
    networks:
      - rag_network
    restart: unless-stopped
    depends_on:
      - chroma
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/api/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  # Frontend Web Application
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: rag_frontend
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:8000/api
      - NODE_ENV=production
    networks:
      - rag_network
    restart: unless-stopped
    depends_on:
      - backend
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:3000"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Chroma Vector Database
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
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/api/v1/heartbeat"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Optional: Monitoring with Prometheus
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./docker/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    networks:
      - rag_network
    restart: unless-stopped
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'

  # Optional: Dashboards with Grafana
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_SECURITY_ADMIN_USER=admin
    volumes:
      - grafana_data:/var/lib/grafana
    networks:
      - rag_network
    restart: unless-stopped
    depends_on:
      - prometheus

volumes:
  chroma_data:
  prometheus_data:
  grafana_data:

networks:
  rag_network:
    driver: bridge
```

### 2.6 Backend Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements first for better caching
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY app/ ./app/

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/api/health || exit 1

# Run application
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## 3. First Steps

### Step 1: Create Backend Skeleton
```bash
# Create backend structure
mkdir -p backend/app/{models,routes,services,utils,middleware}
mkdir -p backend/tests
mkdir -p backend/data
mkdir -p backend/logs
mkdir -p backend/docker

# Create __init__.py files
touch backend/app/__init__.py
touch backend/app/models/__init__.py
touch backend/app/routes/__init__.py
touch backend/app/services/__init__.py
touch backend/app/utils/__init__.py
touch backend/app/middleware/__init__.py
```

### Step 2: Create Frontend Skeleton
```bash
# Create frontend structure
mkdir -p frontend/src/{components,services,types,hooks,pages,styles}
mkdir -p frontend/public

# Copy package.json from above
```

### Step 3: Initialize Git
```bash
git init
git add .
git commit -m "Initial project structure"
```

### Step 4: Configure Environment
```bash
# Copy environment templates
cp backend/.env.example backend/.env
cp frontend/.env.local.example frontend/.env.local

# Edit and add your Hugging Face API key
```

## 4. Development Workflow

### Daily Start
```bash
# Terminal 1: Backend
cd backend
source venv/bin/activate
python -m uvicorn app.main:app --reload

# Terminal 2: Frontend
cd frontend
npm start

# Access at http://localhost:3000
```

### Using Docker
```bash
docker-compose up -d
# Access at http://localhost:3000
```

## 5. Key Milestones

- [ ] Project structure created
- [ ] Python venv setup and dependencies installed
- [ ] React project initialized with TypeScript
- [ ] FastAPI skeleton with basic health endpoint
- [ ] Chroma DB integration (vector store)
- [ ] Embedding generation (Hugging Face)
- [ ] LLM integration (Gemma 4)
- [ ] RAG pipeline complete
- [ ] API endpoints implemented
- [ ] Frontend UI components built
- [ ] Document upload functionality
- [ ] Testing suite
- [ ] Docker containerization
- [ ] Deployment to production
