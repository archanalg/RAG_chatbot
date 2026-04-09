# Banking Policy RAG Application

A Retrieval-Augmented Generation (RAG) application that enables users to query banking policies using natural language and receive accurate, context-aware responses backed by actual policy documents.

## 📋 Project Overview

This application combines modern AI/ML technologies with a user-friendly interface to create an intelligent banking policy assistant. Users can:
- Ask natural language questions about banking policies
- Receive accurate answers with source citations
- Upload and manage policy documents
- Get confidence scores for answers

## 🛠 Tech Stack

- **Frontend**: TypeScript + React
- **Backend**: Python + FastAPI + LangChain
- **Vector Database**: Chroma DB
- **LLM/Embeddings**: Google Gemma 4 31B (via Hugging Face)
- **infrastructure**: Docker, Docker Compose

## 📁 Documentation

| Document | Purpose |
|----------|---------|
| [IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md) | Complete implementation roadmap with 6-phase approach, component details, and data flows |
| [API_SPECIFICATION.md](API_SPECIFICATION.md) | Comprehensive REST API documentation with all endpoints, request/response formats, and examples |
| [DEPLOYMENT_ARCHITECTURE.md](DEPLOYMENT_ARCHITECTURE.md) | Deployment strategies, Docker setup, CI/CD pipelines, monitoring, and production guidelines |

## 🚀 Quick Start

### Prerequisites
- Python 3.11
- Node.js 16+
- Docker & Docker Compose
- Hugging Face API Key (get one at https://huggingface.co/settings/tokens)

### Local Development

1. **Clone the repository**
   ```bash
   git clone <repo-url>
   cd RAG_chatbot
   ```

2. **Setup Backend**
   ```bash
   cd backend
   py -3.11 -m venv venv
   
   # Activate virtual environment
   # Windows:
   venv\Scripts\activate
   # Mac/Linux:
   source venv/bin/activate
   
   pip install -r requirements.txt
   
   # Create .env file
   cp .env.example .env
   # Edit .env and add your Hugging Face API key
   ```

3. **Setup Frontend**
   ```bash
   cd ../frontend
   npm install
   echo "REACT_APP_API_URL=http://localhost:8000/api" > .env.local
   ```

4. **Run with Docker Compose** (Recommended)
   ```bash
   # From project root
   docker-compose up -d
   
   # Backend: http://localhost:8000
   # Frontend: http://localhost:3000
   # Chroma DB: http://localhost:8001
   ```

5. **Run Manually**
   
   Terminal 1 - Backend:
   ```bash
   cd backend
   source venv/bin/activate
   python -m uvicorn app.main:app --reload
   ```
   
   Terminal 2 - Frontend:
   ```bash
   cd frontend
   npm start
   ```

### Accessing the Application

- **Web UI**: http://localhost:3000
- **API Documentation**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc

### Upload Sample Policy

1. Navigate to Admin page in web UI
2. Click "Upload Document"
3. Select a PDF or TXT file
4. Add title and select policy type
5. Click Upload

### Test Queries

Try asking:
- "What is the maximum loan amount for personal loans?"
- "What are the eligibility requirements?"
- "What is the interest rate for credit cards?"

## 📊 Architecture Overview

```
┌─────────────────────────────────┐
│   React Frontend (TypeScript)   │
│  - Query Interface              │
│  - Chat Display                 │
│  - Document Management          │
└──────────────┬──────────────────┘
               │ HTTP/REST
               │
┌──────────────▼──────────────────┐
│   FastAPI Backend (Python)      │
│  - Query Routing                │
│  - Request Validation           │
│  - RAG Pipeline Orchestration   │
└──────────────┬──────────────────┘
               │
    ┌──────────┼──────────┐
    │          │          │
    ▼          ▼          ▼
┌────────┐ ┌────────┐ ┌──────────┐
│ Chroma │ │Gemma-4 │ │LangChain │
│  DB    │ │(LLM)   │ │Components│
└────────┘ └────────┘ └──────────┘
```

## 🔄 Data Flow

1. **Query Processing**
   - User submits question via frontend
   - Backend validates and processes query
   - LangChain generates embedding
   - Chroma DB performs similarity search
   - Retrieved context sent to Gemma 4 LLM
   - Response formatted with citations

2. **Document Ingestion**
   - PDF/TXT uploaded via UI
   - Content extracted and cleaned
   - Text chunked into manageable pieces
   - Embeddings generated for each chunk
   - Vectors stored in Chroma DB with metadata

## 📋 Implementation Phases

| Phase | Duration | Deliverables |
|-------|----------|--------------|
| 1. Foundation | Week 1 | FastAPI skeleton, Docker setup, dependencies |
| 2. RAG Core | Week 2 | Embedding service, Vector store, LLM integration |
| 3. API Development | Week 3 | REST endpoints, document management |
| 4. Frontend | Week 3-4 | React UI, query interface, document upload |
| 5. Document Pipeline | Week 4 | PDF parsing, chunking, indexing |
| 6. Testing & Optimization | Week 5 | Unit tests, performance tuning, QA |
| 7. Deployment | Week 6 | Docker setup, CI/CD, documentation |

See [IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md) for detailed phases.

## 🔧 API Endpoints Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/query` | POST | Query banking policies |
| `/api/documents/upload` | POST | Upload policy documents |
| `/api/documents` | GET | List indexed documents |
| `/api/documents/{id}` | GET | Get document details |
| `/api/documents/{id}` | DELETE | Remove document |
| `/api/search` | POST | Semantic search |
| `/api/health` | GET | Health check |
| `/api/stats` | GET | System statistics |

See [API_SPECIFICATION.md](API_SPECIFICATION.md) for complete documentation.

## 🌐 Frontend Features

- **Query Interface**: Natural language question input
- **Response Display**: Formatted answers with markdown support
- **Source Citations**: Links to source documents with excerpts
- **Document Management**: Upload, view, and delete policies
- **Query History**: View previous queries and responses
- **Responsive Design**: Mobile-friendly UI

## ⚙️ Configuration

### Environment Variables (.env)

```env
# Hugging Face Configuration
HUGGINGFACE_API_KEY=your_api_key_here
HF_MODEL_ID=google/gemma-4-31B-it

# Backend
BACKEND_PORT=8000
DEBUG=true
LOG_LEVEL=INFO

# Chroma DB
CHROMA_DB_PATH=./data/chroma_db
CHROMA_RESET=false

# RAG Parameters
CHUNK_SIZE=1000
CHUNK_OVERLAP=200
TOP_K_RETRIEVAL=5

# Frontend
REACT_APP_API_URL=http://localhost:8000/api
```

## 🧪 Testing

### Backend Tests
```bash
cd backend
pytest tests/
pytest tests/ --cov=app  # With coverage
```

### Frontend Tests
```bash
cd frontend
npm test
npm run test:coverage
```

## 📦 Deployment

### Deployment Strategies

1. **Local Development**: Docker Compose (recommended for setup)
2. **AWS**: ECS with ALB and CloudFront
3. **Google Cloud**: Cloud Run managed containers
4. **Azure**: Container Instances
5. **Self-hosted**: Traditional VM deployment

See [DEPLOYMENT_ARCHITECTURE.md](DEPLOYMENT_ARCHITECTURE.md) for detailed deployment guides.

### Docker Commands

```bash
# Build images
docker-compose build

# Start services
docker-compose up -d

# View logs
docker-compose logs -f backend
docker-compose logs -f frontend

# Stop services
docker-compose down

# Remove all data
docker-compose down -v
```

## 📊 Performance Metrics

### Target Performance
- Query response time: < 3 seconds
- Embedding generation: < 500ms
- Vector search: < 200ms
- LLM generation: < 1.5s

### Scaling Considerations
- Supports thousands of policy documents
- Horizontal scaling for API servers
- Connection pooling for database
- Caching for frequent queries

## 🔒 Security

- Hugging Face API key protected via environment variables
- Input validation on all API endpoints
- CORS configured for frontend domain
- Rate limiting on API endpoints
- Request/response logging for audit trail
- HTTPS enforced in production

## 🛡️ Monitoring & Logging

- **Logging**: JSON format logs to stdout and files
- **Metrics**: Prometheus metrics for monitoring
- **Dashboards**: Grafana visualization
- **Health Checks**: Automated service health monitoring
- **Alerts**: Detection of errors and performance issues

## 📈 Future Enhancements

- [ ] Multi-language support for policies
- [ ] Advanced policy comparison features
- [ ] User authentication and role-based access
- [ ] Feedback loop for answer improvement
- [ ] Policy version control and change tracking
- [ ] Export capabilities (PDF, Excel)
- [ ] Policy recommendation engine
- [ ] Advanced analytics dashboard
- [ ] Integration with banking systems APIs

## 📞 Support & Contribution

### Troubleshooting

**Issue**: "Hugging Face API key not found"
- Ensure `.env` file is created in backend directory
- Verify `HUGGINGFACE_API_KEY` is set correctly
- Restart backend service after adding key

**Issue**: "Vector store connection error"
- Check if Chroma DB is running
- Reset Chroma DB: `docker-compose restart chroma`

**Issue**: "Frontend can't reach backend"
- Verify backend is running: `curl http://localhost:8000/api/health`
- Check `REACT_APP_API_URL` in `.env.local`

### Reporting Issues

1. Check [Issues](github.com/your-repo/issues) for existing reports
2. Create detailed issue with:
   - Steps to reproduce
   - Expected vs actual behavior
   - Error logs/screenshots
   - Environment details (OS, Python version, etc.)

### Contributing

1. Fork the repository
2. Create feature branch: `git checkout -b feature/your-feature`
3. Commit changes: `git commit -am 'Add feature'`
4. Push to branch: `git push origin feature/your-feature`
5. Submit pull request

## 📄 License

[Add your license here - e.g., MIT, Apache 2.0]

## 👥 Authors

- Your Name
- Your Team

## 🙏 Acknowledgments

- Google for Gemma 4 model
- Hugging Face for model hosting and inference
- Chroma for vector database
- LangChain for RAG orchestration

## 📚 Resources

- [LangChain Documentation](https://python.langchain.com/)
- [Chroma DB Documentation](https://docs.trychroma.com/)
- [Hugging Face Model Hub](https://huggingface.co/models)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [React Documentation](https://react.dev/)

## ⭐ Quick Links

- 📋 [Implementation Plan](IMPLEMENTATION_PLAN.md)
- 🔌 [API Specification](API_SPECIFICATION.md)
- 🚀 [Deployment Guide](DEPLOYMENT_ARCHITECTURE.md)
- 🐛 [Issue Tracker](github.com/your-repo/issues)
- 💬 [Discussions](github.com/your-repo/discussions)

---

**Last Updated**: April 8, 2026
**Current Version**: 1.0.0
**Status**: 🔄 Implementation Planning
