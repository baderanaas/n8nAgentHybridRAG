# Hybrid RAG Workflow with n8n and Supabase

A comprehensive document processing and retrieval system that combines semantic search with full-text search capabilities using n8n automation and Supabase vector database.

## Overview

This project implements a hybrid Retrieval-Augmented Generation (RAG) system that processes various document types from Google Drive, stores them in a Supabase vector database, and provides intelligent querying capabilities through a conversational AI agent.

## Architecture

The workflow consists of several interconnected components:

### 1. Document Processing Pipeline
- **File Monitoring**: Automatically detects new and updated files in Google Drive
- **Multi-Format Support**: Handles PDFs, Excel files, CSV files, and Google Docs
- **Data Extraction**: Extracts text content and structured data from documents
- **Vector Embedding**: Generates OpenAI embeddings for semantic search

### 2. Hybrid Search System
- **Vector Search**: Semantic similarity search using OpenAI embeddings
- **Full-Text Search**: PostgreSQL full-text search with ts_rank scoring
- **Reciprocal Rank Fusion (RRF)**: Combines results from both search methods
- **Configurable Weights**: Adjustable weighting for semantic vs. full-text results

### 3. AI Agent Interface
- **Conversational AI**: Powered by Groq's Qwen-QwQ-32B model
- **Memory Management**: Persistent conversation memory using Redis and Airtable
- **Tool Integration**: Multiple specialized tools for document retrieval and analysis
- **Context-Aware Responses**: Maintains user context and preferences

## Features

### Document Management
- ✅ Automatic Google Drive integration
- ✅ Support for PDF, Excel, CSV, and Google Docs
- ✅ Metadata extraction and storage
- ✅ Duplicate detection and handling
- ✅ Schema detection for structured data

### Search Capabilities
- ✅ Hybrid search combining semantic and keyword matching
- ✅ Document content retrieval by file ID
- ✅ Structured data querying with SQL
- ✅ Configurable search parameters

### AI Assistant
- ✅ Natural language document querying
- ✅ Persistent conversation memory
- ✅ Context-aware responses
- ✅ Multi-tool integration
- ✅ Personalized interactions

## Technology Stack

- **Automation**: n8n workflow automation
- **Database**: Supabase (PostgreSQL with vector extension)
- **Vector Embeddings**: OpenAI Ada-002
- **Language Model**: Groq Qwen-QwQ-32B
- **Memory Storage**: Redis + Airtable
- **File Storage**: Google Drive
- **API Integration**: RESTful webhooks

## Database Schema

### Core Tables

#### `documents`
Stores document chunks with vector embeddings:
```sql
CREATE TABLE documents (
  id BIGSERIAL PRIMARY KEY,
  content TEXT,
  metadata JSONB,
  embedding VECTOR(1536)
);
```

#### `document_metadata`
Stores file metadata and schemas:
```sql
CREATE TABLE document_metadata (
  id TEXT PRIMARY KEY,
  title TEXT,
  url TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  schema TEXT
);
```

#### `document_rows`
Stores structured data from CSV/Excel files:
```sql
CREATE TABLE document_rows (
  id SERIAL PRIMARY KEY,
  dataset_id TEXT REFERENCES document_metadata(id),
  row_data JSONB
);
```

### Hybrid Search Function

The system uses a custom PostgreSQL function that implements Reciprocal Rank Fusion:

```sql
CREATE OR REPLACE FUNCTION hybrid_search(
  query_text TEXT,
  query_embedding VECTOR(1536),
  match_count INT,
  full_text_weight FLOAT DEFAULT 1,
  semantic_weight FLOAT DEFAULT 1,
  rrf_k INT DEFAULT 50
)
RETURNS TABLE(id BIGINT, content TEXT, metadata JSONB)
```

## Setup Instructions

### Prerequisites
- n8n instance (cloud or self-hosted)
- Supabase project with vector extension enabled
- Google Drive API access
- OpenAI API key
- Groq API key
- Redis instance
- Airtable account

### Configuration Steps

1. **Database Setup**
   ```sql
   -- Enable vector extension
   CREATE EXTENSION IF NOT EXISTS vector;
   
   -- Run the provided schema creation queries
   ```

2. **n8n Workflow Import**
   - Import the provided workflow JSON
   - Configure all node credentials
   - Set up Google Drive folder monitoring

3. **API Keys Configuration**
   - OpenAI API key for embeddings
   - Groq API key for language model
   - Supabase connection details
   - Google Drive API credentials

4. **Memory System Setup**
   - Configure Redis connection
   - Set up Airtable base for memory storage
   - Create "Memories" table with required fields

### Environment Variables
```env
OPENAI_API_KEY=your_openai_key
GROQ_API_KEY=your_groq_key
SUPABASE_URL=your_supabase_url
SUPABASE_ANON_KEY=your_supabase_key
REDIS_URL=your_redis_url
AIRTABLE_API_KEY=your_airtable_key
```

## Usage

### Document Processing
1. Add documents to your monitored Google Drive folder
2. The system automatically processes and indexes new files
3. Structured data (CSV/Excel) is parsed and stored for querying

### Querying Documents
Send POST requests to the webhook endpoint:
```json
{
  "chatInput": "What are the key insights from the Q3 sales report?",
  "sessionId": "user_session_123"
}
```

### Available Tools

The AI agent has access to several specialized tools:

- **Hybrid RAG**: Searches documents using hybrid approach
- **List Documents**: Retrieves available documents and schemas
- **Get File Contents**: Fetches full text content by file ID
- **Query Table**: Executes SQL queries on structured data
- **Save Memory**: Stores conversation context

## Customization

### Search Parameters
Adjust hybrid search behavior:
- `full_text_weight`: Weight for keyword search (default: 1.0)
- `semantic_weight`: Weight for vector search (default: 1.0)
- `rrf_k`: RRF constant for score combination (default: 50)

### System Prompt
Customize the AI agent's behavior by modifying the system prompt in the Agent node.

### Document Types
Extend support for additional file types by adding new branches to the Switch node.

## Monitoring and Maintenance

### Health Checks
- Monitor n8n workflow execution status
- Check Supabase database performance
- Verify Redis memory usage
- Review API usage limits

### Performance Optimization
- Regularly update vector indexes
- Monitor embedding generation costs
- Optimize SQL queries for large datasets
- Implement caching strategies

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## Support

For issues and questions:
- Check the n8n community forums
- Review Supabase documentation
- Open an issue in this repository

## Roadmap

- [ ] Multi-language document support
- [ ] Advanced filtering and faceted search
- [ ] Document versioning and history
- [ ] Advanced analytics and insights
- [ ] Integration with additional file sources
- [ ] Enhanced security and access controls
