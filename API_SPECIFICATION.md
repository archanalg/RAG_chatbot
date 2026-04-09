# Banking Policy RAG Application - API Specification

## Overview
REST API for the Banking Policy RAG Application. All endpoints use JSON for request/response bodies.

### Base URL
- **Development**: `http://localhost:8000`
- **Production**: `https://api.bankingpolicy.ai`

### Authentication
Currently public, but should implement API key authentication in production.

---

## 1. Query Endpoints

### 1.1 Query Banking Policy
**Endpoint**: `POST /api/query`

**Description**: Submit a natural language question about banking policies and receive a RAG-generated answer with source citations.

**Request**:
```json
{
  "question": "What is the maximum loan amount for personal loans?",
  "policy_type": "optional_filter",
  "conversation_id": "optional_session_id"
}
```

**Request Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| question | string | Yes | Natural language question about banking policies |
| policy_type | string | No | Filter results by policy type (e.g., "personal_loans", "credit_cards") |
| conversation_id | string | No | Session ID for maintaining conversation context |

**Response** (200 OK):
```json
{
  "answer": "The maximum loan amount for personal loans is INR 50 lakhs based on recent policy guidelines...",
  "sources": [
    {
      "document": "Personal Loan Policy v2.3",
      "section": "3.2 Loan Limits",
      "page": 5,
      "excerpt": "Maximum loan amount shall not exceed INR 50 lakhs...",
      "confidence": 0.95
    },
    {
      "document": "Lending Guidelines Updated 2024",
      "section": "2.1 Eligibility Criteria",
      "page": 3,
      "excerpt": "All personal loans must comply with maximum limits...",
      "confidence": 0.87
    }
  ],
  "metadata": {
    "response_time_ms": 1250,
    "chunks_retrieved": 5,
    "model_used": "google/gemma-4-31B-it",
    "timestamp": "2024-04-08T10:30:45Z"
  }
}
```

**Error Response** (400 Bad Request):
```json
{
  "error": "Invalid question",
  "detail": "Question must be at least 3 characters long",
  "status_code": 400
}
```

**Error Response** (503 Service Unavailable):
```json
{
  "error": "LLM service unavailable",
  "detail": "Unable to connect to Hugging Face API",
  "fallback": "Try rephrasing your question or contact support",
  "status_code": 503
}
```

---

## 2. Document Management Endpoints

### 2.1 Upload Policy Document
**Endpoint**: `POST /api/documents/upload`

**Description**: Upload a new banking policy document to be indexed in the vector store.

**Request**:
- Content-Type: `multipart/form-data`

**Form Data**:
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| file | File (PDF/TXT) | Yes | Banking policy document |
| title | string | Yes | Document title |
| policy_type | string | Yes | Classification (e.g., "loan_policy", "credit_card_policy", "kyc_policy") |
| effective_date | string | No | Date when policy takes effect (YYYY-MM-DD) |
| expires_date | string | No | Policy expiration date (YYYY-MM-DD) |

**Response** (200 OK):
```json
{
  "document_id": "doc_550e8400e29b41d4a716446655440000",
  "title": "Personal Loan Policy v2.3",
  "status": "indexed",
  "chunks_created": 45,
  "tokens_indexed": 12500,
  "message": "Document successfully uploaded and indexed",
  "indexing_time_ms": 3500
}
```

**Error Response** (400 Bad Request):
```json
{
  "error": "Invalid file format",
  "detail": "Only PDF and TXT files are supported",
  "status_code": 400
}
```

### 2.2 List Uploaded Documents
**Endpoint**: `GET /api/documents?skip=0&limit=50&policy_type=all`

**Description**: Retrieve list of all indexed banking policy documents.

**Query Parameters**:
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| skip | integer | No | 0 | Number of documents to skip (pagination) |
| limit | integer | No | 50 | Number of documents to return (max: 100) |
| policy_type | string | No | "all" | Filter by policy type |

**Response** (200 OK):
```json
{
  "total_documents": 12,
  "documents": [
    {
      "document_id": "doc_550e8400e29b41d4a716446655440000",
      "title": "Personal Loan Policy v2.3",
      "policy_type": "loan_policy",
      "upload_date": "2024-04-01T10:30:00Z",
      "effective_date": "2024-04-01",
      "expires_date": "2025-03-31",
      "chunks": 45,
      "tokens": 12500,
      "size_kb": 250
    },
    {
      "document_id": "doc_660e8400e29b41d4a716446655440001",
      "title": "Credit Card Policy 2024",
      "policy_type": "credit_card_policy",
      "upload_date": "2024-03-15T14:20:00Z",
      "effective_date": "2024-04-01",
      "expires_date": null,
      "chunks": 32,
      "tokens": 8900,
      "size_kb": 180
    }
  ],
  "pagination": {
    "skip": 0,
    "limit": 50,
    "has_more": false
  }
}
```

### 2.3 Get Document Details
**Endpoint**: `GET /api/documents/{document_id}`

**Description**: Retrieve detailed information about a specific document.

**Path Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| document_id | string | Unique identifier of the document |

**Response** (200 OK):
```json
{
  "document_id": "doc_550e8400e29b41d4a716446655440000",
  "title": "Personal Loan Policy v2.3",
  "policy_type": "loan_policy",
  "upload_date": "2024-04-01T10:30:00Z",
  "effective_date": "2024-04-01",
  "expires_date": "2025-03-31",
  "chunks": 45,
  "tokens": 12500,
  "size_kb": 250,
  "sections": [
    {
      "section_id": "sec_0001",
      "title": "1. Introduction and Scope",
      "chunks": 3
    },
    {
      "section_id": "sec_0002",
      "title": "2. Eligibility Criteria",
      "chunks": 5
    }
  ]
}
```

### 2.4 Delete Document
**Endpoint**: `DELETE /api/documents/{document_id}`

**Description**: Remove a document from the vector store.

**Path Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| document_id | string | Unique identifier of the document |

**Response** (200 OK):
```json
{
  "document_id": "doc_550e8400e29b41d4a716446655440000",
  "status": "deleted",
  "message": "Document and associated vectors removed",
  "chunks_deleted": 45
}
```

### 2.5 Update Document
**Endpoint**: `PUT /api/documents/{document_id}`

**Description**: Update document metadata (title, dates, etc.).

**Request**:
```json
{
  "title": "Personal Loan Policy v2.4",
  "effective_date": "2024-05-01",
  "expires_date": "2025-04-30",
  "policy_type": "loan_policy"
}
```

**Response** (200 OK):
```json
{
  "document_id": "doc_550e8400e29b41d4a716446655440000",
  "status": "updated",
  "message": "Document metadata updated successfully"
}
```

---

## 3. Search & Retrieval Endpoints

### 3.1 Semantic Search Documents
**Endpoint**: `POST /api/search`

**Description**: Perform vector similarity search across policy documents without RAG generation.

**Request**:
```json
{
  "query": "interest rate calculation method",
  "top_k": 5,
  "policy_type": "optional_filter"
}
```

**Response** (200 OK):
```json
{
  "query": "interest rate calculation method",
  "results": [
    {
      "rank": 1,
      "document": "Personal Loan Policy v2.3",
      "section": "4.3 Interest Rate Calculation",
      "confidence": 0.92,
      "excerpt": "Interest rates are calculated on a daily basis using the reducing balance method..."
    },
    {
      "rank": 2,
      "document": "Lending Guidelines Updated 2024",
      "section": "3.1 Rate Structure",
      "confidence": 0.85,
      "excerpt": "Base rate plus spread formula is applied as per RBI guidelines..."
    }
  ],
  "search_time_ms": 450
}
```

---

## 4. Health & System Endpoints

### 4.1 Health Check
**Endpoint**: `GET /api/health`

**Description**: Check API and service availability.

**Response** (200 OK):
```json
{
  "status": "healthy",
  "api": "running",
  "vector_store": "connected",
  "llm_service": "available",
  "timestamp": "2024-04-08T10:30:45Z"
}
```

### 4.2 System Statistics
**Endpoint**: `GET /api/stats`

**Description**: Get system statistics and performance metrics.

**Response** (200 OK):
```json
{
  "total_documents": 12,
  "total_chunks": 487,
  "total_tokens_indexed": 125400,
  "vector_store": {
    "name": "chroma",
    "size_mb": 45.3,
    "vectors": 487,
    "dimensions": 768
  },
  "performance": {
    "avg_query_time_ms": 1250,
    "avg_embedding_time_ms": 250,
    "avg_llm_generation_ms": 800,
    "p95_query_time_ms": 2100,
    "p99_query_time_ms": 2800
  },
  "api_metrics": {
    "total_queries": 1542,
    "successful_queries": 1520,
    "failed_queries": 22,
    "uptime_hours": 168
  }
}
```

---

## 5. Admin Endpoints

### 5.1 Reindex All Documents
**Endpoint**: `POST /api/admin/reindex`

**Description**: Rebuild the entire vector store (admin only).

**Query Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| force | boolean | Force reindexing even if documents haven't changed |

**Response** (200 OK):
```json
{
  "status": "reindexing_started",
  "documents_to_process": 12,
  "estimated_time_seconds": 300,
  "job_id": "job_550e8400e29b41d4a716446655440000"
}
```

### 5.2 Get Reindex Job Status
**Endpoint**: `GET /api/admin/reindex/job/{job_id}`

**Response** (200 OK):
```json
{
  "job_id": "job_550e8400e29b41d4a716446655440000",
  "status": "in_progress",
  "progress": {
    "documents_processed": 8,
    "total_documents": 12,
    "percentage": 66.7
  },
  "elapsed_time_seconds": 120,
  "estimated_remaining_seconds": 60
}
```

### 5.3 Clear Vector Store
**Endpoint**: `POST /api/admin/clear`

**Description**: Clear all documents from the vector store (DANGEROUS - requires confirmation).

**Query Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| confirm | string | Yes | Must be "DELETE_ALL_DATA" to confirm |

**Response** (200 OK):
```json
{
  "status": "vector_store_cleared",
  "documents_deleted": 12,
  "vectors_deleted": 487,
  "timestamp": "2024-04-08T10:30:45Z"
}
```

---

## 6. Error Handling

### Standard Error Response Format
```json
{
  "error": "Error Type",
  "detail": "Detailed error message",
  "status_code": 400,
  "timestamp": "2024-04-08T10:30:45Z",
  "request_id": "req_550e8400e29b41d4a716446655440000"
}
```

### Common HTTP Status Codes
| Code | Description |
|------|-------------|
| 200 | Success |
| 400 | Bad Request (invalid input) |
| 401 | Unauthorized (authentication required) |
| 403 | Forbidden (insufficient permissions) |
| 404 | Not Found (resource doesn't exist) |
| 409 | Conflict (duplicate document) |
| 429 | Too Many Requests (rate limited) |
| 500 | Internal Server Error |
| 503 | Service Unavailable (LLM service down) |

### Specific Error Codes
| Code | Meaning |
|------|---------|
| INVALID_QUESTION | Question format invalid or too short |
| DOCUMENT_NOT_FOUND | Referenced document doesn't exist |
| VECTOR_STORE_ERROR | Chroma DB connection issue |
| LLM_UNAVAILABLE | Google Gemma API unavailable |
| FILE_TOO_LARGE | Uploaded file exceeds size limit (100MB) |
| INVALID_FILE_FORMAT | File is not PDF or TXT |
| RATE_LIMIT_EXCEEDED | API rate limit exceeded |

---

## 7. Rate Limiting

All endpoints are rate limited per IP address:
- **Query Endpoint**: 100 requests per hour
- **Document Upload**: 10 requests per hour
- **Other Endpoints**: 1000 requests per hour

Rate limit headers included in responses:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 99
X-RateLimit-Reset: 1712586645
```

---

## 8. Request/Response Examples

### Example 1: Multi-turn Conversation
```
User Query 1: "What is the interest rate for personal loans?"
↓
Response: "The interest rate for personal loans ranges from 8.5% to 12.5%..."

User Query 2: "What about eligibility requirements?" (same conversation_id)
↓
Response: "Based on our personal loan policy, eligibility requirements include..."
(incorporates context from previous query)
```

### Example 2: Policy Type Filtering
```
POST /api/query
{
  "question": "What is the maximum credit limit?",
  "policy_type": "credit_card_policy"
}
↓
Response focuses only on credit card related policies
```

---

## 9. Frontend Integration Example (TypeScript)

```typescript
// api.ts
const API_BASE = "http://localhost:8000/api";

// Type definitions
interface QueryRequest {
  question: string;
  policy_type?: string;
  conversation_id?: string;
}

interface QueryResponse {
  answer: string;
  sources: Array<{
    document: string;
    section: string;
    page: number;
    excerpt: string;
    confidence: number;
  }>;
  metadata: {
    response_time_ms: number;
    chunks_retrieved: number;
    model_used: string;
    timestamp: string;
  };
}

// Query function
export const queryPolicy = async (
  request: QueryRequest
): Promise<QueryResponse> => {
  const response = await fetch(`${API_BASE}/query`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(request),
  });

  if (!response.ok) {
    throw new Error(`API Error: ${response.statusText}`);
  }

  return response.json();
};

// Document upload function
export const uploadDocument = async (
  file: File,
  title: string,
  policyType: string
): Promise<{ document_id: string; status: string }> => {
  const formData = new FormData();
  formData.append("file", file);
  formData.append("title", title);
  formData.append("policy_type", policyType);

  const response = await fetch(`${API_BASE}/documents/upload`, {
    method: "POST",
    body: formData,
  });

  return response.json();
};
```

---

## 10. CORS Configuration

Allowed Origins (Development):
- `http://localhost:3000`
- `http://localhost:8080`

Allowed Methods:
- GET, POST, PUT, DELETE, OPTIONS

Allowed Headers:
- Content-Type
- Authorization

---

## 11. OpenAPI/Swagger Documentation

The API supports Swagger UI at:
- **Development**: `http://localhost:8000/docs`
- **ReDoc**: `http://localhost:8000/redoc`

Automatically generated from FastAPI annotations.

---

## 12. Versioning Strategy

Current Version: `v1`

Future versions will be available at:
- `/api/v1/query` (current)
- `/api/v2/query` (future)

Backward compatibility maintained for at least 2 versions.
