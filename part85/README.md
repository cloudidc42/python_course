# Part 85: Project - AI-Powered Document Assistant

## สารบัญ
1. [Project Overview](#overview)
2. [Architecture](#architecture)
3. [Backend - FastAPI](#backend)
4. [Document Processing](#document-processing)
5. [Vector Store with Chroma](#vector-store)
6. [Claude Integration](#claude-integration)
7. [Conversation Management](#conversation)
8. [API Endpoints](#api-endpoints)
9. [Frontend](#frontend)
10. [Docker Deployment](#docker)
11. [Testing](#testing)
12. [Setup Guide](#setup)

---

## 1. Project Overview {#overview}

### AI Document Assistant

สร้าง AI Document Assistant ที่สมบูรณ์ด้วย:

- **FastAPI** backend พร้อม REST API
- **Anthropic Claude** สำหรับ LLM
- **Document upload** - รองรับ PDF, DOCX, TXT
- **RAG** ด้วย Chroma vector store
- **Conversation history** ต่อเนื่อง
- **Multiple document support**
- **Source citations** ในคำตอบ
- **Streaming responses**
- **Docker** deployment

### Project Structure

```
ai-doc-assistant/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py              # FastAPI app
│   │   ├── config.py            # Settings
│   │   ├── models.py            # Pydantic models
│   │   ├── routers/
│   │   │   ├── documents.py     # Document endpoints
│   │   │   ├── conversations.py # Chat endpoints
│   │   │   └── health.py        # Health check
│   │   ├── services/
│   │   │   ├── document_service.py  # Document processing
│   │   │   ├── vector_service.py    # Vector store
│   │   │   ├── llm_service.py       # Claude integration
│   │   │   └── conversation_service.py
│   │   └── utils/
│   │       ├── text_splitter.py
│   │       └── file_parser.py
│   ├── tests/
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/              # Optional Next.js frontend
│   ├── app/
│   ├── components/
│   └── package.json
├── docker-compose.yml
└── README.md
```

---

## 2. Architecture {#architecture}

```
User Request
     │
     ▼
┌─────────────────────────────────────────┐
│           FastAPI Backend               │
│  ┌─────────┐  ┌──────────┐  ┌───────┐  │
│  │Document │  │   Chat   │  │Health │  │
│  │  Router │  │  Router  │  │Router │  │
│  └────┬────┘  └────┬─────┘  └───────┘  │
│       │             │                   │
│  ┌────▼────┐  ┌────▼─────────────────┐ │
│  │Document │  │  Conversation Service │ │
│  │Service  │  └──────────────────────┘ │
│  └────┬────┘          │                │
│       │         ┌─────▼────────┐       │
│  ┌────▼────┐   │  LLM Service  │       │
│  │ Vector  │   │  (Claude API) │       │
│  │Service  │   └──────────────┘       │
│  └────┬────┘                          │
│       │                               │
│  ┌────▼────────────────────┐         │
│  │    Chroma Vector Store   │         │
│  └─────────────────────────┘         │
└─────────────────────────────────────────┘
         │
    ┌────▼─────────────────┐
    │  Document Storage     │
    │  (local or S3)       │
    └──────────────────────┘
```

---

## 3. Backend - FastAPI {#backend}

### config.py

```python
# backend/app/config.py
from pydantic_settings import BaseSettings
from pathlib import Path
from typing import Optional

class Settings(BaseSettings):
    # App
    APP_NAME: str = "AI Document Assistant"
    APP_VERSION: str = "1.0.0"
    DEBUG: bool = False
    
    # API Keys
    ANTHROPIC_API_KEY: str
    
    # Models
    LLM_MODEL: str = "claude-3-5-sonnet-20241022"
    EMBEDDING_MODEL: str = "all-MiniLM-L6-v2"
    
    # Document Processing
    MAX_FILE_SIZE_MB: int = 50
    ALLOWED_EXTENSIONS: list = [".pdf", ".docx", ".txt", ".md"]
    CHUNK_SIZE: int = 500
    CHUNK_OVERLAP: int = 50
    
    # Vector Store
    CHROMA_PERSIST_DIR: str = "./chroma_db"
    COLLECTION_NAME: str = "documents"
    
    # Retrieval
    TOP_K_RESULTS: int = 5
    SIMILARITY_THRESHOLD: float = 0.3
    
    # Conversation
    MAX_CONVERSATION_HISTORY: int = 20
    MAX_CONTEXT_TOKENS: int = 100_000
    
    # Storage
    UPLOAD_DIR: str = "./uploads"
    
    # CORS
    ALLOWED_ORIGINS: list = ["http://localhost:3000", "http://localhost:8080"]
    
    class Config:
        env_file = ".env"

settings = Settings()

# Create directories
Path(settings.UPLOAD_DIR).mkdir(exist_ok=True)
Path(settings.CHROMA_PERSIST_DIR).mkdir(exist_ok=True)
```

### models.py

```python
# backend/app/models.py
from pydantic import BaseModel, Field, validator
from typing import Optional, List, Dict, Any
from datetime import datetime
from enum import Enum
import uuid

class DocumentStatus(str, Enum):
    UPLOADING = "uploading"
    PROCESSING = "processing"
    READY = "ready"
    ERROR = "error"

class MessageRole(str, Enum):
    USER = "user"
    ASSISTANT = "assistant"
    SYSTEM = "system"

# Document Models
class DocumentCreate(BaseModel):
    """Request model for document creation"""
    title: Optional[str] = None
    description: Optional[str] = None
    tags: List[str] = []

class DocumentMetadata(BaseModel):
    """Document metadata"""
    id: str = Field(default_factory=lambda: str(uuid.uuid4()))
    title: str
    filename: str
    file_type: str
    file_size: int
    description: Optional[str] = None
    tags: List[str] = []
    status: DocumentStatus = DocumentStatus.UPLOADING
    chunk_count: int = 0
    error_message: Optional[str] = None
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

class DocumentResponse(BaseModel):
    """Response model for document"""
    id: str
    title: str
    filename: str
    file_type: str
    status: DocumentStatus
    chunk_count: int
    created_at: datetime
    description: Optional[str] = None
    tags: List[str] = []

class DocumentList(BaseModel):
    """List of documents response"""
    documents: List[DocumentResponse]
    total: int

# Conversation Models
class Message(BaseModel):
    """Single message in conversation"""
    id: str = Field(default_factory=lambda: str(uuid.uuid4()))
    role: MessageRole
    content: str
    timestamp: datetime = Field(default_factory=datetime.utcnow)
    sources: List[Dict] = []
    metadata: Dict[str, Any] = {}

class ConversationCreate(BaseModel):
    """Create new conversation"""
    title: Optional[str] = None
    document_ids: List[str] = []
    system_prompt: Optional[str] = None

class ConversationMetadata(BaseModel):
    """Conversation metadata"""
    id: str = Field(default_factory=lambda: str(uuid.uuid4()))
    title: str = "New Conversation"
    document_ids: List[str] = []
    system_prompt: Optional[str] = None
    message_count: int = 0
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)

class ConversationResponse(BaseModel):
    """Conversation response"""
    id: str
    title: str
    document_ids: List[str]
    message_count: int
    created_at: datetime

# Chat Models
class ChatRequest(BaseModel):
    """Chat request"""
    message: str = Field(..., min_length=1, max_length=10000)
    conversation_id: Optional[str] = None
    document_ids: Optional[List[str]] = None
    stream: bool = False
    
    @validator('message')
    def message_not_empty(cls, v):
        if not v.strip():
            raise ValueError('Message cannot be empty')
        return v.strip()

class ChatResponse(BaseModel):
    """Chat response"""
    conversation_id: str
    message: Message
    sources: List[Dict[str, Any]] = []
    tokens_used: int = 0
    processing_time: float = 0.0

# Source citation
class SourceCitation(BaseModel):
    """Source citation in response"""
    document_id: str
    document_title: str
    chunk_id: str
    content: str
    relevance_score: float
    page_number: Optional[int] = None

# Health check
class HealthResponse(BaseModel):
    status: str
    version: str
    components: Dict[str, str]
    timestamp: datetime = Field(default_factory=datetime.utcnow)
```

### File Parser - utils/file_parser.py

```python
# backend/app/utils/file_parser.py
import io
import os
from pathlib import Path
from typing import Optional, Tuple
import logging

logger = logging.getLogger(__name__)

class FileParser:
    """Parse different file types to extract text"""
    
    SUPPORTED_TYPES = {
        '.pdf': '_parse_pdf',
        '.docx': '_parse_docx',
        '.doc': '_parse_docx',
        '.txt': '_parse_text',
        '.md': '_parse_text',
        '.csv': '_parse_csv',
        '.json': '_parse_json',
    }
    
    def parse(self, file_path: str) -> Tuple[str, dict]:
        """
        Parse file and return (text_content, metadata)
        
        Args:
            file_path: Path to file
            
        Returns:
            (extracted_text, metadata_dict)
        """
        path = Path(file_path)
        ext = path.suffix.lower()
        
        if ext not in self.SUPPORTED_TYPES:
            raise ValueError(f"Unsupported file type: {ext}")
        
        method_name = self.SUPPORTED_TYPES[ext]
        method = getattr(self, method_name)
        
        try:
            text, metadata = method(file_path)
            metadata['file_type'] = ext.lstrip('.')
            metadata['file_size'] = path.stat().st_size
            metadata['filename'] = path.name
            return text, metadata
        except Exception as e:
            logger.error(f"Error parsing {file_path}: {e}")
            raise
    
    def _parse_pdf(self, file_path: str) -> Tuple[str, dict]:
        """Parse PDF file"""
        # pip install pypdf2 or pdfplumber
        try:
            import pdfplumber
            
            text_parts = []
            metadata = {"page_count": 0}
            
            with pdfplumber.open(file_path) as pdf:
                metadata["page_count"] = len(pdf.pages)
                
                if pdf.metadata:
                    metadata["author"] = pdf.metadata.get("Author", "")
                    metadata["title"] = pdf.metadata.get("Title", "")
                    metadata["created"] = str(pdf.metadata.get("CreationDate", ""))
                
                for page_num, page in enumerate(pdf.pages, 1):
                    text = page.extract_text()
                    if text:
                        text_parts.append(f"[Page {page_num}]\n{text}")
            
            full_text = "\n\n".join(text_parts)
            return full_text, metadata
            
        except ImportError:
            # Fallback to PyPDF2
            import PyPDF2
            
            text_parts = []
            metadata = {}
            
            with open(file_path, 'rb') as f:
                reader = PyPDF2.PdfReader(f)
                metadata["page_count"] = len(reader.pages)
                
                for page_num, page in enumerate(reader.pages, 1):
                    text = page.extract_text()
                    if text:
                        text_parts.append(f"[Page {page_num}]\n{text}")
            
            return "\n\n".join(text_parts), metadata
    
    def _parse_docx(self, file_path: str) -> Tuple[str, dict]:
        """Parse DOCX file"""
        # pip install python-docx
        from docx import Document
        
        doc = Document(file_path)
        metadata = {
            "paragraph_count": len(doc.paragraphs),
            "section_count": len(doc.sections),
        }
        
        # Extract core properties
        try:
            props = doc.core_properties
            metadata["author"] = props.author or ""
            metadata["title"] = props.title or ""
        except:
            pass
        
        # Extract text from paragraphs
        text_parts = []
        current_heading = None
        
        for para in doc.paragraphs:
            if not para.text.strip():
                continue
            
            # Check if heading
            if para.style.name.startswith('Heading'):
                level = para.style.name.split()[-1]
                heading_prefix = '#' * int(level) if level.isdigit() else '#'
                text_parts.append(f"\n{heading_prefix} {para.text}\n")
                current_heading = para.text
            else:
                text_parts.append(para.text)
        
        # Extract tables
        for table in doc.tables:
            table_text = []
            for row in table.rows:
                row_text = " | ".join(cell.text.strip() for cell in row.cells)
                table_text.append(row_text)
            text_parts.append("\n" + "\n".join(table_text) + "\n")
        
        return "\n".join(text_parts), metadata
    
    def _parse_text(self, file_path: str) -> Tuple[str, dict]:
        """Parse plain text file"""
        with open(file_path, 'r', encoding='utf-8', errors='replace') as f:
            content = f.read()
        
        lines = content.split('\n')
        metadata = {
            "line_count": len(lines),
            "word_count": len(content.split()),
            "char_count": len(content),
        }
        
        return content, metadata
    
    def _parse_csv(self, file_path: str) -> Tuple[str, dict]:
        """Parse CSV file"""
        import csv
        
        rows = []
        with open(file_path, 'r', encoding='utf-8') as f:
            reader = csv.DictReader(f)
            headers = reader.fieldnames or []
            
            for i, row in enumerate(reader):
                if i == 0:
                    rows.append("Columns: " + ", ".join(headers))
                    rows.append("-" * 50)
                row_text = ", ".join(f"{k}: {v}" for k, v in row.items())
                rows.append(row_text)
        
        metadata = {
            "columns": headers,
            "row_count": len(rows) - 2,  # Subtract header rows
        }
        
        return "\n".join(rows), metadata
    
    def _parse_json(self, file_path: str) -> Tuple[str, dict]:
        """Parse JSON file"""
        import json
        
        with open(file_path, 'r', encoding='utf-8') as f:
            data = json.load(f)
        
        # Convert to readable text
        text = json.dumps(data, indent=2, ensure_ascii=False)
        
        metadata = {
            "type": type(data).__name__,
            "size": len(text),
        }
        
        if isinstance(data, list):
            metadata["item_count"] = len(data)
        elif isinstance(data, dict):
            metadata["keys"] = list(data.keys())[:10]
        
        return text, metadata
```

### Text Splitter - utils/text_splitter.py

```python
# backend/app/utils/text_splitter.py
import re
from typing import List, Dict, Optional
from dataclasses import dataclass, field

@dataclass
class Chunk:
    """A text chunk with metadata"""
    id: str
    content: str
    doc_id: str
    chunk_index: int
    start_char: int
    end_char: int
    metadata: Dict = field(default_factory=dict)

class RecursiveTextSplitter:
    """
    Recursive text splitter that tries different separators.
    Similar to LangChain's RecursiveCharacterTextSplitter.
    """
    
    def __init__(
        self,
        chunk_size: int = 500,
        chunk_overlap: int = 50,
        separators: Optional[List[str]] = None,
        keep_separator: bool = True,
    ):
        self.chunk_size = chunk_size
        self.chunk_overlap = chunk_overlap
        self.keep_separator = keep_separator
        
        self.separators = separators or [
            "\n\n",     # Paragraphs
            "\n",       # Lines
            ". ",       # Sentences
            "! ",
            "? ",
            "; ",
            ", ",       # Clauses
            " ",        # Words
            "",         # Characters
        ]
    
    def _split_text(self, text: str, separators: List[str]) -> List[str]:
        """Recursively split text"""
        final_chunks = []
        
        # Find the best separator
        separator = separators[-1]
        new_separators = []
        
        for i, sep in enumerate(separators):
            if sep == "":
                separator = sep
                break
            if sep in text:
                separator = sep
                new_separators = separators[i+1:]
                break
        
        # Split
        splits = re.split(re.escape(separator), text) if separator else list(text)
        
        # Process splits
        good_splits = []
        current = []
        current_len = 0
        
        for split in splits:
            if not split.strip():
                continue
            
            split_len = len(split.split())
            
            if split_len > self.chunk_size:
                # Recursively split large chunks
                if good_splits:
                    merged = self._merge_splits(good_splits, separator)
                    final_chunks.extend(merged)
                    good_splits = []
                    current_len = 0
                
                if new_separators:
                    sub_chunks = self._split_text(split, new_separators)
                    final_chunks.extend(sub_chunks)
                else:
                    final_chunks.append(split)
            else:
                if current_len + split_len > self.chunk_size and good_splits:
                    merged = self._merge_splits(good_splits, separator)
                    final_chunks.extend(merged)
                    good_splits = good_splits[-int(self.chunk_overlap/max(split_len, 1)):]
                    current_len = sum(len(s.split()) for s in good_splits)
                
                good_splits.append(split)
                current_len += split_len
        
        if good_splits:
            merged = self._merge_splits(good_splits, separator)
            final_chunks.extend(merged)
        
        return final_chunks
    
    def _merge_splits(self, splits: List[str], separator: str) -> List[str]:
        """Merge small splits into chunks"""
        docs = []
        current = []
        current_len = 0
        
        for split in splits:
            split_len = len(split.split())
            
            if current_len + split_len > self.chunk_size:
                if current:
                    joined = separator.join(current)
                    if joined.strip():
                        docs.append(joined)
                    
                    # Keep overlap
                    while current and current_len > self.chunk_overlap:
                        current_len -= len(current[0].split())
                        current.pop(0)
            
            current.append(split)
            current_len += split_len
        
        if current:
            joined = separator.join(current)
            if joined.strip():
                docs.append(joined)
        
        return docs
    
    def create_chunks(self, text: str, doc_id: str, 
                     metadata: Dict = None) -> List[Chunk]:
        """Split text and create Chunk objects"""
        raw_chunks = self._split_text(text, self.separators)
        
        chunks = []
        current_pos = 0
        
        for i, chunk_text in enumerate(raw_chunks):
            # Find position in original text
            start = text.find(chunk_text, current_pos)
            if start == -1:
                start = current_pos
            end = start + len(chunk_text)
            current_pos = max(0, end - (self.chunk_overlap * 5))
            
            chunk = Chunk(
                id=f"{doc_id}_chunk_{i:04d}",
                content=chunk_text.strip(),
                doc_id=doc_id,
                chunk_index=i,
                start_char=start,
                end_char=end,
                metadata={
                    **(metadata or {}),
                    "chunk_index": i,
                    "word_count": len(chunk_text.split()),
                }
            )
            chunks.append(chunk)
        
        return chunks
```

### Vector Service - services/vector_service.py

```python
# backend/app/services/vector_service.py
import logging
from typing import List, Dict, Optional, Tuple
from pathlib import Path

logger = logging.getLogger(__name__)

class VectorService:
    """
    Vector store service using ChromaDB.
    Handles embedding, storage, and retrieval.
    """
    
    def __init__(self, config):
        self.config = config
        self.client = None
        self.collection = None
        self.embedding_fn = None
        self._initialize()
    
    def _initialize(self):
        """Initialize ChromaDB and embedding model"""
        try:
            import chromadb
            from chromadb.config import Settings
            from sentence_transformers import SentenceTransformer
            
            # Initialize ChromaDB
            self.client = chromadb.Client(Settings(
                chroma_db_impl="duckdb+parquet",
                persist_directory=self.config.CHROMA_PERSIST_DIR,
                anonymized_telemetry=False
            ))
            
            # Get or create collection
            self.collection = self.client.get_or_create_collection(
                name=self.config.COLLECTION_NAME,
                metadata={"hnsw:space": "cosine"}
            )
            
            # Initialize embedding model
            self.embedding_model = SentenceTransformer(
                self.config.EMBEDDING_MODEL,
                device='cpu'  # Change to 'cuda' for GPU
            )
            
            logger.info(f"VectorService initialized with {self.count()} vectors")
            
        except ImportError as e:
            logger.warning(f"Failed to initialize ChromaDB: {e}. Using mock.")
            self._use_mock()
    
    def _use_mock(self):
        """Use mock implementation when ChromaDB not available"""
        # For testing without actual ChromaDB
        self._mock_storage = {}
        self._mock_embeddings = {}
    
    def embed(self, texts: List[str]) -> List[List[float]]:
        """Generate embeddings for texts"""
        if hasattr(self, 'embedding_model'):
            embeddings = self.embedding_model.encode(
                texts, 
                batch_size=32,
                normalize_embeddings=True
            )
            return embeddings.tolist()
        else:
            # Mock embeddings
            import random
            return [[random.random() for _ in range(384)] for _ in texts]
    
    def add_chunks(self, chunks: List[Dict]) -> bool:
        """Add document chunks to vector store"""
        if not chunks:
            return True
        
        # Prepare data
        ids = [c['id'] for c in chunks]
        texts = [c['content'] for c in chunks]
        metadatas = [c.get('metadata', {}) for c in chunks]
        
        # Generate embeddings
        embeddings = self.embed(texts)
        
        if hasattr(self, 'collection') and self.collection:
            # Add to ChromaDB in batches
            batch_size = 100
            for i in range(0, len(ids), batch_size):
                batch_ids = ids[i:i+batch_size]
                batch_texts = texts[i:i+batch_size]
                batch_metas = metadatas[i:i+batch_size]
                batch_embeds = embeddings[i:i+batch_size]
                
                self.collection.add(
                    ids=batch_ids,
                    documents=batch_texts,
                    metadatas=batch_metas,
                    embeddings=batch_embeds
                )
        else:
            # Mock storage
            for id, text, meta, emb in zip(ids, texts, metadatas, embeddings):
                self._mock_storage[id] = {"text": text, "metadata": meta}
                self._mock_embeddings[id] = emb
        
        logger.info(f"Added {len(chunks)} chunks to vector store")
        return True
    
    def search(
        self, 
        query: str,
        top_k: int = 5,
        doc_ids: Optional[List[str]] = None,
        threshold: float = 0.3
    ) -> List[Dict]:
        """Search for similar chunks"""
        query_embedding = self.embed([query])[0]
        
        if hasattr(self, 'collection') and self.collection:
            # Build filter
            where = None
            if doc_ids:
                where = {"doc_id": {"$in": doc_ids}}
            
            results = self.collection.query(
                query_embeddings=[query_embedding],
                n_results=min(top_k, self.count()),
                where=where,
                include=["documents", "metadatas", "distances"]
            )
            
            # Format results
            formatted = []
            if results["ids"][0]:
                for i, id in enumerate(results["ids"][0]):
                    distance = results["distances"][0][i]
                    similarity = 1 - distance  # Convert distance to similarity
                    
                    if similarity >= threshold:
                        formatted.append({
                            "id": id,
                            "content": results["documents"][0][i],
                            "metadata": results["metadatas"][0][i],
                            "score": similarity
                        })
            
            return sorted(formatted, key=lambda x: -x["score"])
        
        else:
            # Mock search
            import numpy as np
            
            if not self._mock_embeddings:
                return []
            
            query_vec = np.array(query_embedding)
            
            results = []
            for id, emb in self._mock_embeddings.items():
                emb_vec = np.array(emb)
                score = float(np.dot(query_vec, emb_vec) / (
                    np.linalg.norm(query_vec) * np.linalg.norm(emb_vec) + 1e-10
                ))
                
                # Filter by doc_id if specified
                meta = self._mock_storage[id]["metadata"]
                if doc_ids and meta.get("doc_id") not in doc_ids:
                    continue
                
                if score >= threshold:
                    results.append({
                        "id": id,
                        "content": self._mock_storage[id]["text"],
                        "metadata": meta,
                        "score": score
                    })
            
            return sorted(results, key=lambda x: -x["score"])[:top_k]
    
    def delete_document(self, doc_id: str) -> bool:
        """Delete all chunks for a document"""
        if hasattr(self, 'collection') and self.collection:
            self.collection.delete(where={"doc_id": doc_id})
        else:
            # Mock deletion
            to_delete = [id for id, data in self._mock_storage.items() 
                        if data["metadata"].get("doc_id") == doc_id]
            for id in to_delete:
                del self._mock_storage[id]
                del self._mock_embeddings[id]
        return True
    
    def count(self) -> int:
        """Count total vectors in store"""
        if hasattr(self, 'collection') and self.collection:
            return self.collection.count()
        return len(getattr(self, '_mock_storage', {}))
    
    def persist(self):
        """Persist vector store to disk"""
        if hasattr(self, 'client') and self.client:
            try:
                self.client.persist()
            except AttributeError:
                pass  # newer chromadb versions auto-persist
```

### LLM Service - services/llm_service.py

```python
# backend/app/services/llm_service.py
import anthropic
import logging
from typing import List, Dict, Optional, AsyncGenerator
import json
import time

logger = logging.getLogger(__name__)

class LLMService:
    """
    Service for interacting with Anthropic Claude API.
    Handles message formatting, streaming, and response parsing.
    """
    
    DEFAULT_SYSTEM_PROMPT = """You are a helpful AI assistant that answers questions 
based on provided documents.

Guidelines:
1. Answer ONLY based on the provided context
2. Cite your sources using [Doc: title] format
3. If the answer isn't in the context, clearly state: "This information is not in the provided documents"
4. Be accurate, concise, and helpful
5. For complex questions, structure your answer clearly
6. If asked about something outside the documents, respond: "I can only answer questions about the provided documents"
"""
    
    def __init__(self, config):
        self.config = config
        self.client = anthropic.Anthropic(api_key=config.ANTHROPIC_API_KEY)
        self.model = config.LLM_MODEL
        logger.info(f"LLMService initialized with model: {self.model}")
    
    def _format_context(self, retrieved_chunks: List[Dict]) -> str:
        """Format retrieved chunks as context"""
        if not retrieved_chunks:
            return ""
        
        context_parts = []
        for i, chunk in enumerate(retrieved_chunks, 1):
            meta = chunk.get("metadata", {})
            doc_title = meta.get("doc_title", "Unknown Document")
            page_info = f", page {meta['page_number']}" if "page_number" in meta else ""
            
            context_parts.append(
                f"[Source {i}: {doc_title}{page_info}]\n"
                f"{chunk['content']}"
            )
        
        return "\n\n---\n\n".join(context_parts)
    
    def _format_messages(
        self, 
        user_message: str,
        conversation_history: List[Dict],
        context: str
    ) -> List[Dict]:
        """Format messages for API call"""
        messages = []
        
        # Add conversation history (excluding current user message)
        for msg in conversation_history[-self.config.MAX_CONVERSATION_HISTORY:]:
            if msg["role"] in ["user", "assistant"]:
                messages.append({
                    "role": msg["role"],
                    "content": msg["content"]
                })
        
        # Add current message with context
        if context:
            user_content = f"""Based on the following documents:

{context}

---

User Question: {user_message}

Please answer the question based on the documents above. 
Cite specific sources using [Doc: title] format."""
        else:
            user_content = user_message
        
        messages.append({"role": "user", "content": user_content})
        return messages
    
    def generate(
        self,
        user_message: str,
        conversation_history: List[Dict] = None,
        retrieved_chunks: List[Dict] = None,
        system_prompt: Optional[str] = None,
        max_tokens: int = 2048,
        temperature: float = 0.3,
    ) -> Dict:
        """Generate response (non-streaming)"""
        start_time = time.time()
        
        context = self._format_context(retrieved_chunks or [])
        messages = self._format_messages(
            user_message,
            conversation_history or [],
            context
        )
        
        system = system_prompt or self.DEFAULT_SYSTEM_PROMPT
        
        try:
            response = self.client.messages.create(
                model=self.model,
                max_tokens=max_tokens,
                temperature=temperature,
                system=system,
                messages=messages
            )
            
            content = response.content[0].text
            
            return {
                "content": content,
                "model": response.model,
                "stop_reason": response.stop_reason,
                "usage": {
                    "input_tokens": response.usage.input_tokens,
                    "output_tokens": response.usage.output_tokens,
                    "total_tokens": response.usage.input_tokens + response.usage.output_tokens
                },
                "processing_time": time.time() - start_time
            }
            
        except anthropic.RateLimitError:
            logger.error("Rate limit exceeded")
            raise
        except anthropic.APIError as e:
            logger.error(f"API error: {e}")
            raise
    
    async def generate_stream(
        self,
        user_message: str,
        conversation_history: List[Dict] = None,
        retrieved_chunks: List[Dict] = None,
        system_prompt: Optional[str] = None,
        max_tokens: int = 2048,
    ) -> AsyncGenerator[str, None]:
        """Generate streaming response"""
        context = self._format_context(retrieved_chunks or [])
        messages = self._format_messages(
            user_message,
            conversation_history or [],
            context
        )
        
        system = system_prompt or self.DEFAULT_SYSTEM_PROMPT
        
        with self.client.messages.stream(
            model=self.model,
            max_tokens=max_tokens,
            system=system,
            messages=messages
        ) as stream:
            for text in stream.text_stream:
                yield text
    
    def extract_sources_from_response(
        self, 
        response_text: str,
        retrieved_chunks: List[Dict]
    ) -> List[Dict]:
        """Extract cited sources from response"""
        import re
        
        cited_titles = re.findall(r'\[Doc:\s*([^\]]+)\]', response_text)
        
        sources = []
        seen = set()
        
        for title in cited_titles:
            title = title.strip()
            if title in seen:
                continue
            seen.add(title)
            
            # Find matching chunk
            for chunk in retrieved_chunks:
                meta = chunk.get("metadata", {})
                if meta.get("doc_title", "").lower() == title.lower():
                    sources.append({
                        "doc_id": meta.get("doc_id"),
                        "doc_title": meta.get("doc_title"),
                        "chunk_id": chunk["id"],
                        "content": chunk["content"][:200] + "...",
                        "score": chunk.get("score", 0),
                    })
                    break
        
        return sources
    
    def count_tokens(self, messages: List[Dict], system: str = "") -> int:
        """Estimate token count"""
        try:
            response = self.client.messages.count_tokens(
                model=self.model,
                system=system,
                messages=messages
            )
            return response.input_tokens
        except Exception:
            # Fallback estimation
            total_text = system + " ".join(m["content"] for m in messages)
            return int(len(total_text.split()) * 1.3)
```

### Main Application - main.py

```python
# backend/app/main.py
from fastapi import FastAPI, HTTPException, UploadFile, File, Form, BackgroundTasks
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import StreamingResponse, JSONResponse
from fastapi.staticfiles import StaticFiles
from contextlib import asynccontextmanager
import logging
import uuid
import os
import asyncio
from pathlib import Path
from typing import Optional, List
import json
import shutil
from datetime import datetime

# Import app modules
from .config import settings
from .models import (
    DocumentMetadata, DocumentResponse, DocumentList, DocumentStatus,
    ConversationCreate, ConversationMetadata, ConversationResponse,
    ChatRequest, ChatResponse, Message, MessageRole, HealthResponse
)
from .utils.file_parser import FileParser
from .utils.text_splitter import RecursiveTextSplitter
from .services.vector_service import VectorService
from .services.llm_service import LLMService

# Setup logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

# In-memory stores (replace with database in production)
documents_store: dict = {}
conversations_store: dict = {}
messages_store: dict = {}  # conversation_id -> [messages]

# Services
file_parser = FileParser()
text_splitter = RecursiveTextSplitter(
    chunk_size=settings.CHUNK_SIZE,
    chunk_overlap=settings.CHUNK_OVERLAP
)

@asynccontextmanager
async def lifespan(app: FastAPI):
    """Application lifespan events"""
    # Startup
    logger.info("Starting AI Document Assistant...")
    
    # Initialize services
    app.state.vector_service = VectorService(settings)
    app.state.llm_service = LLMService(settings)
    
    logger.info(f"Vector store: {app.state.vector_service.count()} vectors")
    logger.info("Application ready!")
    
    yield
    
    # Shutdown
    logger.info("Shutting down...")
    if hasattr(app.state.vector_service, 'persist'):
        app.state.vector_service.persist()

# Create FastAPI app
app = FastAPI(
    title=settings.APP_NAME,
    version=settings.APP_VERSION,
    description="AI-powered document assistant with RAG capabilities",
    lifespan=lifespan
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.ALLOWED_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# ============= DOCUMENT ENDPOINTS =============

@app.post("/api/documents/upload", response_model=DocumentResponse)
async def upload_document(
    background_tasks: BackgroundTasks,
    file: UploadFile = File(...),
    title: Optional[str] = Form(None),
    description: Optional[str] = Form(None),
    tags: Optional[str] = Form(""),  # Comma-separated
):
    """Upload and process a document"""
    # Validate file
    ext = Path(file.filename).suffix.lower()
    if ext not in settings.ALLOWED_EXTENSIONS:
        raise HTTPException(
            status_code=400,
            detail=f"File type {ext} not supported. Allowed: {settings.ALLOWED_EXTENSIONS}"
        )
    
    if file.size and file.size > settings.MAX_FILE_SIZE_MB * 1024 * 1024:
        raise HTTPException(status_code=400, detail="File too large")
    
    # Generate document ID
    doc_id = str(uuid.uuid4())
    
    # Save file
    upload_path = Path(settings.UPLOAD_DIR) / f"{doc_id}{ext}"
    
    try:
        with open(upload_path, "wb") as buffer:
            content = await file.read()
            buffer.write(content)
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Failed to save file: {e}")
    
    # Create document metadata
    doc_title = title or Path(file.filename).stem
    tags_list = [t.strip() for t in tags.split(",") if t.strip()] if tags else []
    
    doc = DocumentMetadata(
        id=doc_id,
        title=doc_title,
        filename=file.filename,
        file_type=ext.lstrip("."),
        file_size=len(content),
        description=description,
        tags=tags_list,
        status=DocumentStatus.PROCESSING
    )
    documents_store[doc_id] = doc
    
    # Process in background
    background_tasks.add_task(
        process_document,
        doc_id=doc_id,
        file_path=str(upload_path),
        vector_service=app.state.vector_service
    )
    
    return DocumentResponse(
        id=doc.id,
        title=doc.title,
        filename=doc.filename,
        file_type=doc.file_type,
        status=doc.status,
        chunk_count=0,
        created_at=doc.created_at,
        description=doc.description,
        tags=doc.tags
    )

async def process_document(doc_id: str, file_path: str, vector_service: VectorService):
    """Background task to process document"""
    doc = documents_store.get(doc_id)
    if not doc:
        return
    
    try:
        logger.info(f"Processing document: {doc_id}")
        
        # Parse file
        text, metadata = file_parser.parse(file_path)
        
        if not text.strip():
            raise ValueError("Document has no extractable text")
        
        # Create chunks
        chunks = text_splitter.create_chunks(
            text=text,
            doc_id=doc_id,
            metadata={
                "doc_id": doc_id,
                "doc_title": doc.title,
                **metadata
            }
        )
        
        # Convert to dicts for vector service
        chunk_dicts = [
            {
                "id": chunk.id,
                "content": chunk.content,
                "metadata": {**chunk.metadata, "doc_id": doc_id, "doc_title": doc.title}
            }
            for chunk in chunks
        ]
        
        # Add to vector store
        vector_service.add_chunks(chunk_dicts)
        
        # Update metadata
        doc.status = DocumentStatus.READY
        doc.chunk_count = len(chunks)
        doc.updated_at = datetime.utcnow()
        
        logger.info(f"Document {doc_id} processed: {len(chunks)} chunks")
        
    except Exception as e:
        logger.error(f"Error processing document {doc_id}: {e}")
        doc.status = DocumentStatus.ERROR
        doc.error_message = str(e)
        doc.updated_at = datetime.utcnow()

@app.get("/api/documents", response_model=DocumentList)
async def list_documents(
    status: Optional[str] = None,
    tag: Optional[str] = None,
    limit: int = 50,
    offset: int = 0
):
    """List all documents with optional filtering"""
    docs = list(documents_store.values())
    
    # Filter
    if status:
        docs = [d for d in docs if d.status == status]
    if tag:
        docs = [d for d in docs if tag in d.tags]
    
    # Sort by created_at descending
    docs.sort(key=lambda x: x.created_at, reverse=True)
    
    total = len(docs)
    docs = docs[offset:offset+limit]
    
    return DocumentList(
        documents=[
            DocumentResponse(
                id=d.id, title=d.title, filename=d.filename,
                file_type=d.file_type, status=d.status,
                chunk_count=d.chunk_count, created_at=d.created_at,
                description=d.description, tags=d.tags
            )
            for d in docs
        ],
        total=total
    )

@app.get("/api/documents/{doc_id}", response_model=DocumentResponse)
async def get_document(doc_id: str):
    """Get document by ID"""
    doc = documents_store.get(doc_id)
    if not doc:
        raise HTTPException(status_code=404, detail="Document not found")
    
    return DocumentResponse(
        id=doc.id, title=doc.title, filename=doc.filename,
        file_type=doc.file_type, status=doc.status,
        chunk_count=doc.chunk_count, created_at=doc.created_at,
        description=doc.description, tags=doc.tags
    )

@app.delete("/api/documents/{doc_id}")
async def delete_document(doc_id: str):
    """Delete document"""
    doc = documents_store.get(doc_id)
    if not doc:
        raise HTTPException(status_code=404, detail="Document not found")
    
    # Delete from vector store
    app.state.vector_service.delete_document(doc_id)
    
    # Delete file
    ext = f".{doc.file_type}"
    file_path = Path(settings.UPLOAD_DIR) / f"{doc_id}{ext}"
    if file_path.exists():
        os.remove(file_path)
    
    del documents_store[doc_id]
    
    return {"message": "Document deleted"}

# ============= CONVERSATION ENDPOINTS =============

@app.post("/api/conversations", response_model=ConversationResponse)
async def create_conversation(conv: ConversationCreate):
    """Create new conversation"""
    # Validate document IDs
    for doc_id in conv.document_ids:
        if doc_id not in documents_store:
            raise HTTPException(status_code=400, detail=f"Document {doc_id} not found")
    
    conv_meta = ConversationMetadata(
        title=conv.title or "New Conversation",
        document_ids=conv.document_ids,
        system_prompt=conv.system_prompt
    )
    
    conversations_store[conv_meta.id] = conv_meta
    messages_store[conv_meta.id] = []
    
    return ConversationResponse(
        id=conv_meta.id,
        title=conv_meta.title,
        document_ids=conv_meta.document_ids,
        message_count=0,
        created_at=conv_meta.created_at
    )

@app.get("/api/conversations/{conv_id}/messages")
async def get_conversation_messages(conv_id: str):
    """Get conversation messages"""
    if conv_id not in conversations_store:
        raise HTTPException(status_code=404, detail="Conversation not found")
    
    messages = messages_store.get(conv_id, [])
    return {
        "conversation_id": conv_id,
        "messages": [msg.dict() for msg in messages],
        "count": len(messages)
    }

@app.post("/api/chat", response_model=ChatResponse)
async def chat(request: ChatRequest):
    """Main chat endpoint"""
    import time
    start_time = time.time()
    
    # Get or create conversation
    if request.conversation_id and request.conversation_id in conversations_store:
        conv = conversations_store[request.conversation_id]
        conv_id = request.conversation_id
    else:
        # Create new conversation
        doc_ids = request.document_ids or []
        conv = ConversationMetadata(
            document_ids=doc_ids,
            title=request.message[:50] + "..."
        )
        conversations_store[conv.id] = conv
        messages_store[conv.id] = []
        conv_id = conv.id
    
    # Get document IDs to search
    search_doc_ids = request.document_ids or conv.document_ids or None
    
    # Only search documents that are ready
    if search_doc_ids:
        ready_doc_ids = [
            doc_id for doc_id in search_doc_ids
            if doc_id in documents_store and 
            documents_store[doc_id].status == DocumentStatus.READY
        ]
    else:
        # Search all ready documents
        ready_doc_ids = [
            doc_id for doc_id, doc in documents_store.items()
            if doc.status == DocumentStatus.READY
        ]
    
    # Retrieve relevant chunks
    retrieved_chunks = []
    if ready_doc_ids or not search_doc_ids:
        retrieved_chunks = app.state.vector_service.search(
            query=request.message,
            top_k=settings.TOP_K_RESULTS,
            doc_ids=ready_doc_ids if ready_doc_ids else None,
            threshold=settings.SIMILARITY_THRESHOLD
        )
    
    # Get conversation history
    history = messages_store.get(conv_id, [])
    history_dicts = [{"role": msg.role.value, "content": msg.content} for msg in history]
    
    # Generate response
    if request.stream:
        # Streaming response
        async def stream_response():
            full_response = ""
            
            async for chunk in app.state.llm_service.generate_stream(
                user_message=request.message,
                conversation_history=history_dicts,
                retrieved_chunks=retrieved_chunks,
                system_prompt=conv.system_prompt
            ):
                full_response += chunk
                yield f"data: {json.dumps({'text': chunk})}\n\n"
            
            # Send final message with metadata
            user_msg = Message(role=MessageRole.USER, content=request.message)
            assistant_msg = Message(
                role=MessageRole.ASSISTANT,
                content=full_response,
                sources=app.state.llm_service.extract_sources_from_response(
                    full_response, retrieved_chunks
                )
            )
            
            messages_store[conv_id].append(user_msg)
            messages_store[conv_id].append(assistant_msg)
            conv.message_count += 2
            
            yield f"data: {json.dumps({'done': True, 'conversation_id': conv_id})}\n\n"
        
        return StreamingResponse(
            stream_response(),
            media_type="text/event-stream"
        )
    
    else:
        # Non-streaming response
        result = app.state.llm_service.generate(
            user_message=request.message,
            conversation_history=history_dicts,
            retrieved_chunks=retrieved_chunks,
            system_prompt=conv.system_prompt
        )
        
        # Create messages
        user_msg = Message(role=MessageRole.USER, content=request.message)
        
        sources = app.state.llm_service.extract_sources_from_response(
            result["content"], retrieved_chunks
        )
        
        assistant_msg = Message(
            role=MessageRole.ASSISTANT,
            content=result["content"],
            sources=sources
        )
        
        # Save to history
        messages_store[conv_id].append(user_msg)
        messages_store[conv_id].append(assistant_msg)
        conv.message_count += 2
        conv.updated_at = datetime.utcnow()
        
        return ChatResponse(
            conversation_id=conv_id,
            message=assistant_msg,
            sources=sources,
            tokens_used=result["usage"]["total_tokens"],
            processing_time=time.time() - start_time
        )

# ============= HEALTH CHECK =============

@app.get("/health", response_model=HealthResponse)
async def health_check():
    """System health check"""
    components = {}
    
    # Check vector store
    try:
        count = app.state.vector_service.count()
        components["vector_store"] = f"ok ({count} vectors)"
    except Exception as e:
        components["vector_store"] = f"error: {e}"
    
    # Check LLM
    components["llm"] = f"ok (model: {settings.LLM_MODEL})"
    
    # Storage
    upload_dir = Path(settings.UPLOAD_DIR)
    doc_count = len(list(upload_dir.glob("*"))) if upload_dir.exists() else 0
    components["storage"] = f"ok ({doc_count} files)"
    
    overall_status = "ok" if all("error" not in v for v in components.values()) else "degraded"
    
    return HealthResponse(
        status=overall_status,
        version=settings.APP_VERSION,
        components=components
    )

@app.get("/")
async def root():
    return {
        "name": settings.APP_NAME,
        "version": settings.APP_VERSION,
        "docs": "/docs",
        "health": "/health"
    }
```

---

## 9. Frontend (Optional) {#frontend}

```html
<!-- frontend/app/page.tsx (simplified Next.js/HTML version) -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Document Assistant</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
               background: #f0f2f5; height: 100vh; display: flex; flex-direction: column; }
        
        .header { background: #1a1a2e; color: white; padding: 1rem 2rem;
                  display: flex; align-items: center; gap: 1rem; }
        .header h1 { font-size: 1.5rem; }
        
        .main { display: flex; flex: 1; overflow: hidden; }
        
        .sidebar { width: 300px; background: white; border-right: 1px solid #e0e0e0;
                   display: flex; flex-direction: column; }
        
        .sidebar-section { padding: 1rem; border-bottom: 1px solid #e0e0e0; }
        .sidebar-section h3 { font-size: 0.9rem; color: #666; text-transform: uppercase;
                              letter-spacing: 0.5px; margin-bottom: 0.5rem; }
        
        .upload-area { border: 2px dashed #ccc; border-radius: 8px; padding: 1rem;
                       text-align: center; cursor: pointer; transition: border-color 0.2s; }
        .upload-area:hover { border-color: #1a1a2e; }
        
        .doc-list { flex: 1; overflow-y: auto; padding: 0.5rem; }
        .doc-item { display: flex; align-items: center; gap: 0.5rem; padding: 0.5rem;
                    border-radius: 6px; cursor: pointer; transition: background 0.2s;
                    margin-bottom: 0.25rem; }
        .doc-item:hover { background: #f0f2f5; }
        .doc-item.selected { background: #e8f0fe; }
        .doc-status { width: 8px; height: 8px; border-radius: 50%; flex-shrink: 0; }
        .status-ready { background: #4caf50; }
        .status-processing { background: #ff9800; }
        .status-error { background: #f44336; }
        
        .chat-area { flex: 1; display: flex; flex-direction: column; }
        
        .messages { flex: 1; overflow-y: auto; padding: 1rem; display: flex;
                    flex-direction: column; gap: 1rem; }
        
        .message { max-width: 80%; padding: 0.75rem 1rem; border-radius: 12px; }
        .message.user { background: #1a1a2e; color: white; align-self: flex-end;
                         border-bottom-right-radius: 4px; }
        .message.assistant { background: white; align-self: flex-start;
                              border-bottom-left-radius: 4px;
                              box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
        
        .sources { margin-top: 0.5rem; font-size: 0.8rem; color: #666; }
        .source-chip { display: inline-block; padding: 2px 8px; background: #e8f0fe;
                       color: #1a73e8; border-radius: 12px; margin: 2px; }
        
        .input-area { padding: 1rem; background: white; border-top: 1px solid #e0e0e0;
                      display: flex; gap: 0.5rem; }
        
        .input-area textarea { flex: 1; padding: 0.75rem; border: 1px solid #ddd;
                               border-radius: 8px; resize: none; font-size: 1rem;
                               font-family: inherit; }
        .input-area textarea:focus { outline: none; border-color: #1a1a2e; }
        
        .btn { padding: 0.75rem 1.5rem; border: none; border-radius: 8px; cursor: pointer;
               font-size: 1rem; transition: background 0.2s; }
        .btn-primary { background: #1a1a2e; color: white; }
        .btn-primary:hover { background: #2d2d4e; }
        .btn-primary:disabled { background: #ccc; cursor: not-allowed; }
        
        .typing-indicator { display: flex; gap: 4px; padding: 0.5rem; }
        .typing-dot { width: 8px; height: 8px; background: #666; border-radius: 50%;
                      animation: typing 1.4s infinite ease-in-out; }
        .typing-dot:nth-child(2) { animation-delay: 0.2s; }
        .typing-dot:nth-child(3) { animation-delay: 0.4s; }
        @keyframes typing { 0%, 60%, 100% { transform: translateY(0); }
                            30% { transform: translateY(-10px); } }
        
        .empty-state { flex: 1; display: flex; flex-direction: column; align-items: center;
                       justify-content: center; color: #999; gap: 1rem; }
    </style>
</head>
<body>
    <div class="header">
        <span>📄</span>
        <h1>AI Document Assistant</h1>
    </div>
    
    <div class="main">
        <!-- Sidebar -->
        <div class="sidebar">
            <div class="sidebar-section">
                <h3>Upload Document</h3>
                <div class="upload-area" id="uploadArea">
                    <p>Drop files here or click to upload</p>
                    <p style="font-size: 0.8rem; color: #999;">PDF, DOCX, TXT</p>
                    <input type="file" id="fileInput" accept=".pdf,.docx,.txt,.md" 
                           style="display:none" multiple>
                </div>
            </div>
            
            <div class="sidebar-section">
                <h3>Documents</h3>
            </div>
            <div class="doc-list" id="docList">
                <div style="text-align:center; color:#999; padding:1rem; font-size:0.9rem">
                    No documents yet
                </div>
            </div>
        </div>
        
        <!-- Chat Area -->
        <div class="chat-area">
            <div class="messages" id="messages">
                <div class="empty-state">
                    <span style="font-size:3rem">💬</span>
                    <h2>Start a conversation</h2>
                    <p>Upload documents and ask questions about them</p>
                </div>
            </div>
            
            <div class="input-area">
                <textarea id="messageInput" placeholder="Ask a question about your documents..." 
                          rows="2"></textarea>
                <button class="btn btn-primary" id="sendBtn" onclick="sendMessage()">
                    Send
                </button>
            </div>
        </div>
    </div>
    
    <script>
        const API_BASE = 'http://localhost:8000/api';
        let selectedDocIds = new Set();
        let currentConvId = null;
        let documents = {};
        
        // Upload area
        const uploadArea = document.getElementById('uploadArea');
        const fileInput = document.getElementById('fileInput');
        
        uploadArea.addEventListener('click', () => fileInput.click());
        uploadArea.addEventListener('dragover', (e) => {
            e.preventDefault();
            uploadArea.style.borderColor = '#1a1a2e';
        });
        uploadArea.addEventListener('dragleave', () => {
            uploadArea.style.borderColor = '#ccc';
        });
        uploadArea.addEventListener('drop', (e) => {
            e.preventDefault();
            uploadArea.style.borderColor = '#ccc';
            Array.from(e.dataTransfer.files).forEach(uploadFile);
        });
        fileInput.addEventListener('change', (e) => {
            Array.from(e.target.files).forEach(uploadFile);
        });
        
        async function uploadFile(file) {
            const formData = new FormData();
            formData.append('file', file);
            formData.append('title', file.name.replace(/\.[^/.]+$/, ''));
            
            try {
                const response = await fetch(`${API_BASE}/documents/upload`, {
                    method: 'POST',
                    body: formData
                });
                const doc = await response.json();
                documents[doc.id] = doc;
                renderDocList();
                
                // Poll for ready status
                pollDocStatus(doc.id);
            } catch (err) {
                console.error('Upload failed:', err);
                alert('Upload failed: ' + err.message);
            }
        }
        
        async function pollDocStatus(docId) {
            const maxAttempts = 30;
            let attempts = 0;
            
            const poll = async () => {
                attempts++;
                try {
                    const response = await fetch(`${API_BASE}/documents/${docId}`);
                    const doc = await response.json();
                    documents[docId] = doc;
                    renderDocList();
                    
                    if (doc.status === 'ready' || doc.status === 'error') {
                        return;
                    }
                    
                    if (attempts < maxAttempts) {
                        setTimeout(poll, 2000);
                    }
                } catch (err) {
                    console.error('Poll error:', err);
                }
            };
            
            setTimeout(poll, 2000);
        }
        
        function renderDocList() {
            const list = document.getElementById('docList');
            const docs = Object.values(documents);
            
            if (docs.length === 0) {
                list.innerHTML = '<div style="text-align:center;color:#999;padding:1rem;font-size:0.9rem">No documents yet</div>';
                return;
            }
            
            list.innerHTML = docs.map(doc => `
                <div class="doc-item ${selectedDocIds.has(doc.id) ? 'selected' : ''}" 
                     onclick="toggleDoc('${doc.id}')">
                    <div class="doc-status status-${doc.status}"></div>
                    <div style="flex:1;min-width:0">
                        <div style="font-size:0.9rem;font-weight:500;overflow:hidden;text-overflow:ellipsis;white-space:nowrap">
                            ${doc.title}
                        </div>
                        <div style="font-size:0.75rem;color:#999">${doc.status} · ${doc.chunk_count} chunks</div>
                    </div>
                    <button onclick="event.stopPropagation();deleteDoc('${doc.id}')" 
                            style="border:none;background:none;cursor:pointer;color:#999;font-size:1.2rem">×</button>
                </div>
            `).join('');
        }
        
        function toggleDoc(docId) {
            if (selectedDocIds.has(docId)) {
                selectedDocIds.delete(docId);
            } else {
                selectedDocIds.add(docId);
            }
            renderDocList();
        }
        
        async function deleteDoc(docId) {
            if (!confirm('Delete this document?')) return;
            await fetch(`${API_BASE}/documents/${docId}`, {method: 'DELETE'});
            delete documents[docId];
            selectedDocIds.delete(docId);
            renderDocList();
        }
        
        async function sendMessage() {
            const input = document.getElementById('messageInput');
            const msg = input.value.trim();
            if (!msg) return;
            
            input.value = '';
            document.getElementById('sendBtn').disabled = true;
            
            // Add user message
            addMessage('user', msg);
            
            // Add typing indicator
            const typingId = addTypingIndicator();
            
            try {
                const response = await fetch(`${API_BASE}/chat`, {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({
                        message: msg,
                        conversation_id: currentConvId,
                        document_ids: selectedDocIds.size > 0 ? Array.from(selectedDocIds) : null,
                        stream: false
                    })
                });
                
                const data = await response.json();
                
                removeTypingIndicator(typingId);
                currentConvId = data.conversation_id;
                
                addMessage('assistant', data.message.content, data.sources);
                
            } catch (err) {
                removeTypingIndicator(typingId);
                addMessage('assistant', '❌ Error: ' + err.message);
            }
            
            document.getElementById('sendBtn').disabled = false;
            input.focus();
        }
        
        function addMessage(role, content, sources = []) {
            const msgs = document.getElementById('messages');
            
            // Remove empty state
            const emptyState = msgs.querySelector('.empty-state');
            if (emptyState) emptyState.remove();
            
            const div = document.createElement('div');
            div.className = `message ${role}`;
            
            let html = `<div>${content.replace(/\n/g, '<br>')}</div>`;
            
            if (sources && sources.length > 0) {
                html += `<div class="sources">
                    Sources: ${sources.map(s => 
                        `<span class="source-chip">${s.doc_title || s.document_title}</span>`
                    ).join('')}
                </div>`;
            }
            
            div.innerHTML = html;
            msgs.appendChild(div);
            msgs.scrollTop = msgs.scrollHeight;
        }
        
        function addTypingIndicator() {
            const msgs = document.getElementById('messages');
            const id = 'typing-' + Date.now();
            const div = document.createElement('div');
            div.id = id;
            div.className = 'message assistant';
            div.innerHTML = '<div class="typing-indicator"><div class="typing-dot"></div><div class="typing-dot"></div><div class="typing-dot"></div></div>';
            msgs.appendChild(div);
            msgs.scrollTop = msgs.scrollHeight;
            return id;
        }
        
        function removeTypingIndicator(id) {
            const el = document.getElementById(id);
            if (el) el.remove();
        }
        
        // Send on Enter (Shift+Enter for newline)
        document.getElementById('messageInput').addEventListener('keydown', (e) => {
            if (e.key === 'Enter' && !e.shiftKey) {
                e.preventDefault();
                sendMessage();
            }
        });
        
        // Load existing documents on startup
        async function loadDocuments() {
            try {
                const response = await fetch(`${API_BASE}/documents`);
                const data = await response.json();
                data.documents.forEach(doc => { documents[doc.id] = doc; });
                renderDocList();
            } catch (err) {
                console.log('Could not load documents:', err);
            }
        }
        
        loadDocuments();
    </script>
</body>
</html>
```

---

## 10. Docker Deployment {#docker}

### Dockerfile

```dockerfile
# backend/Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Download NLTK data
RUN python -c "import nltk; nltk.download('punkt'); nltk.download('stopwords')"

# Copy application code
COPY . .

# Create directories
RUN mkdir -p uploads chroma_db

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s \
    CMD curl -f http://localhost:8000/health || exit 1

# Start application
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### requirements.txt

```text
# backend/requirements.txt
# Core
fastapi==0.104.1
uvicorn[standard]==0.24.0
pydantic==2.5.0
pydantic-settings==2.1.0
python-multipart==0.0.6

# Anthropic
anthropic==0.28.0

# Vector store
chromadb==0.4.18
sentence-transformers==2.2.2

# Document parsing
pypdf2==3.0.1
pdfplumber==0.10.3
python-docx==1.1.0

# NLP
nltk==3.8.1
spacy==3.7.2

# Utils
python-dotenv==1.0.0
aiofiles==23.2.1
httpx==0.25.2
tenacity==8.2.3  # Retry logic
loguru==0.7.2

# Dev
pytest==7.4.3
pytest-asyncio==0.21.1
httpx==0.25.2  # For testing
```

### docker-compose.yml

```yaml
# docker-compose.yml
version: '3.8'

services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - DEBUG=false
      - CHROMA_PERSIST_DIR=/data/chroma
      - UPLOAD_DIR=/data/uploads
    volumes:
      - app_data:/data
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  frontend:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./frontend/dist:/usr/share/nginx/html
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - backend
    restart: unless-stopped

volumes:
  app_data:
```

### nginx.conf

```nginx
# nginx.conf
server {
    listen 80;
    server_name localhost;
    
    # Frontend
    location / {
        root /usr/share/nginx/html;
        try_files $uri $uri/ /index.html;
    }
    
    # API proxy
    location /api/ {
        proxy_pass http://backend:8000/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    
    # WebSocket support (for streaming)
    location /ws/ {
        proxy_pass http://backend:8000/ws/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
    
    # SSE (streaming responses)
    location /api/chat/stream {
        proxy_pass http://backend:8000/api/chat/stream;
        proxy_set_header Connection '';
        proxy_http_version 1.1;
        chunked_transfer_encoding on;
        proxy_buffering off;
        proxy_cache off;
    }
}
```

---

## 11. Testing {#testing}

```python
# backend/tests/test_api.py
import pytest
import pytest_asyncio
from fastapi.testclient import TestClient
from pathlib import Path
import io
import json

# Mock the services before importing the app
import sys
from unittest.mock import MagicMock, AsyncMock, patch

@pytest.fixture
def client():
    """Create test client with mocked services"""
    
    with patch('app.services.vector_service.VectorService') as MockVector:
        with patch('app.services.llm_service.LLMService') as MockLLM:
            
            # Configure mocks
            MockVector.return_value.count.return_value = 0
            MockVector.return_value.search.return_value = []
            MockVector.return_value.add_chunks.return_value = True
            MockVector.return_value.delete_document.return_value = True
            
            MockLLM.return_value.generate.return_value = {
                "content": "This is a test response.",
                "model": "claude-3-5-sonnet-20241022",
                "stop_reason": "end_turn",
                "usage": {"input_tokens": 100, "output_tokens": 50, "total_tokens": 150},
                "processing_time": 0.5
            }
            MockLLM.return_value.extract_sources_from_response.return_value = []
            
            from app.main import app
            yield TestClient(app)

class TestHealth:
    def test_health_ok(self, client):
        response = client.get("/health")
        assert response.status_code == 200
        data = response.json()
        assert data["status"] in ["ok", "degraded"]
        assert "version" in data
    
    def test_root(self, client):
        response = client.get("/")
        assert response.status_code == 200

class TestDocuments:
    def test_list_documents_empty(self, client):
        response = client.get("/api/documents")
        assert response.status_code == 200
        data = response.json()
        assert data["total"] >= 0
        assert "documents" in data
    
    def test_upload_text_document(self, client, tmp_path):
        # Create test file
        content = "This is a test document with some content about Python programming."
        test_file = tmp_path / "test.txt"
        test_file.write_text(content)
        
        with open(test_file, 'rb') as f:
            response = client.post(
                "/api/documents/upload",
                files={"file": ("test.txt", f, "text/plain")},
                data={"title": "Test Document", "description": "Test"}
            )
        
        assert response.status_code == 200
        data = response.json()
        assert data["title"] == "Test Document"
        assert data["file_type"] == "txt"
        assert data["status"] in ["uploading", "processing", "ready"]
        
        return data["id"]
    
    def test_upload_invalid_type(self, client):
        response = client.post(
            "/api/documents/upload",
            files={"file": ("test.exe", b"binary content", "application/octet-stream")}
        )
        assert response.status_code == 400
    
    def test_get_nonexistent_document(self, client):
        response = client.get("/api/documents/nonexistent-id")
        assert response.status_code == 404

class TestConversations:
    def test_create_conversation(self, client):
        response = client.post("/api/conversations", json={
            "title": "Test Conversation",
            "document_ids": [],
        })
        assert response.status_code == 200
        data = response.json()
        assert "id" in data
        assert data["title"] == "Test Conversation"
    
    def test_get_messages_empty(self, client):
        # Create conversation first
        conv_response = client.post("/api/conversations", json={})
        conv_id = conv_response.json()["id"]
        
        response = client.get(f"/api/conversations/{conv_id}/messages")
        assert response.status_code == 200
        data = response.json()
        assert data["count"] == 0
    
    def test_get_messages_nonexistent(self, client):
        response = client.get("/api/conversations/nonexistent/messages")
        assert response.status_code == 404

class TestChat:
    def test_simple_chat(self, client):
        response = client.post("/api/chat", json={
            "message": "Hello, how are you?",
        })
        assert response.status_code == 200
        data = response.json()
        assert "conversation_id" in data
        assert "message" in data
        assert data["message"]["content"] == "This is a test response."
    
    def test_chat_with_conversation_id(self, client):
        # First message
        r1 = client.post("/api/chat", json={"message": "First message"})
        conv_id = r1.json()["conversation_id"]
        
        # Second message in same conversation
        r2 = client.post("/api/chat", json={
            "message": "Second message",
            "conversation_id": conv_id
        })
        assert r2.status_code == 200
        assert r2.json()["conversation_id"] == conv_id
    
    def test_empty_message(self, client):
        response = client.post("/api/chat", json={"message": ""})
        assert response.status_code == 422  # Validation error

class TestIntegration:
    def test_full_workflow(self, client, tmp_path):
        """Test complete workflow: upload -> chat"""
        # 1. Upload document
        content = """
        Python is a high-level programming language.
        It was created by Guido van Rossum in 1991.
        Python is known for its simple syntax and readability.
        """
        
        test_file = tmp_path / "python_doc.txt"
        test_file.write_text(content)
        
        with open(test_file, 'rb') as f:
            upload_resp = client.post(
                "/api/documents/upload",
                files={"file": ("python_doc.txt", f, "text/plain")},
                data={"title": "Python Guide"}
            )
        
        assert upload_resp.status_code == 200
        doc_id = upload_resp.json()["id"]
        
        # 2. Chat about the document
        chat_resp = client.post("/api/chat", json={
            "message": "Who created Python?",
            "document_ids": [doc_id]
        })
        
        assert chat_resp.status_code == 200
        assert "conversation_id" in chat_resp.json()

# Run tests
if __name__ == "__main__":
    pytest.main([__file__, "-v", "--tb=short"])
```

---

## 12. Setup Guide {#setup}

### Quick Start

```bash
# 1. Clone or create project
mkdir ai-doc-assistant && cd ai-doc-assistant

# 2. Backend setup
cd backend
python -m venv venv
source venv/bin/activate  # Linux/Mac
# venv\Scripts\activate    # Windows

pip install -r requirements.txt

# 3. Environment variables
cat > .env << EOF
ANTHROPIC_API_KEY=your_api_key_here
DEBUG=true
CHROMA_PERSIST_DIR=./chroma_db
UPLOAD_DIR=./uploads
LLM_MODEL=claude-3-5-sonnet-20241022
EMBEDDING_MODEL=all-MiniLM-L6-v2
CHUNK_SIZE=500
CHUNK_OVERLAP=50
MAX_CONVERSATION_HISTORY=20
EOF

# 4. Run development server
uvicorn app.main:app --reload --port 8000

# 5. Test the API
curl http://localhost:8000/health
curl http://localhost:8000/docs  # Swagger UI
```

### Docker Deployment

```bash
# Build and run with Docker Compose
cp .env.example .env
# Edit .env with your API key

docker-compose up --build -d

# Check logs
docker-compose logs -f backend

# Stop
docker-compose down
```

### API Testing

```python
# test_api_manual.py - Quick API test
import requests
import json

BASE = "http://localhost:8000/api"

# Health check
r = requests.get("http://localhost:8000/health")
print(f"Health: {r.json()['status']}")

# Upload document
with open("test.txt", "w") as f:
    f.write("Python was created by Guido van Rossum in 1991. It is a high-level language.")

with open("test.txt", "rb") as f:
    r = requests.post(
        f"{BASE}/documents/upload",
        files={"file": ("test.txt", f, "text/plain")},
        data={"title": "Python Guide"}
    )
doc = r.json()
print(f"Document: {doc['id']} - {doc['status']}")

# Wait for processing
import time
time.sleep(3)

# Chat
r = requests.post(f"{BASE}/chat", json={
    "message": "Who created Python?",
    "document_ids": [doc["id"]]
})
chat = r.json()
print(f"Answer: {chat['message']['content']}")
print(f"Sources: {chat['sources']}")
```

### Performance Optimization

```python
# backend/app/utils/performance.py
import functools
import time
import asyncio
from typing import Callable, Any
import logging

logger = logging.getLogger(__name__)

def retry(max_attempts=3, delay=1.0, backoff=2.0):
    """Retry decorator with exponential backoff"""
    def decorator(func):
        @functools.wraps(func)
        async def wrapper(*args, **kwargs):
            last_error = None
            for attempt in range(max_attempts):
                try:
                    return await func(*args, **kwargs)
                except Exception as e:
                    last_error = e
                    if attempt < max_attempts - 1:
                        wait = delay * (backoff ** attempt)
                        logger.warning(f"Attempt {attempt+1} failed: {e}. Retrying in {wait:.1f}s")
                        await asyncio.sleep(wait)
            raise last_error
        return wrapper
    return decorator

def cache(ttl_seconds=300):
    """Simple TTL cache decorator"""
    def decorator(func):
        _cache = {}
        
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            key = str(args) + str(sorted(kwargs.items()))
            now = time.time()
            
            if key in _cache:
                result, timestamp = _cache[key]
                if now - timestamp < ttl_seconds:
                    return result
            
            result = func(*args, **kwargs)
            _cache[key] = (result, now)
            
            # Cleanup old entries
            expired = [k for k, (_, t) in _cache.items() if now - t >= ttl_seconds]
            for k in expired:
                del _cache[k]
            
            return result
        
        wrapper.clear_cache = lambda: _cache.clear()
        return wrapper
    return decorator

class BatchProcessor:
    """Process items in batches for efficiency"""
    
    def __init__(self, process_fn: Callable, batch_size: int = 32):
        self.process_fn = process_fn
        self.batch_size = batch_size
    
    def process(self, items: list) -> list:
        results = []
        for i in range(0, len(items), self.batch_size):
            batch = items[i:i + self.batch_size]
            batch_results = self.process_fn(batch)
            results.extend(batch_results)
        return results
```

### Monitoring and Logging

```python
# backend/app/utils/monitoring.py
import time
import logging
from contextlib import contextmanager
from collections import defaultdict, deque
from datetime import datetime

logger = logging.getLogger(__name__)

class MetricsCollector:
    """Collect application metrics"""
    
    def __init__(self, window_seconds=3600):
        self.window = window_seconds
        self.request_times = deque()
        self.error_counts = defaultdict(int)
        self.token_usage = {"input": 0, "output": 0}
        self.document_count = 0
        self.query_count = 0
    
    @contextmanager
    def track_request(self, endpoint: str):
        """Track request timing"""
        start = time.time()
        try:
            yield
        except Exception as e:
            self.error_counts[endpoint] += 1
            raise
        finally:
            elapsed = time.time() - start
            now = time.time()
            self.request_times.append((now, endpoint, elapsed))
            
            # Remove old entries
            cutoff = now - self.window
            while self.request_times and self.request_times[0][0] < cutoff:
                self.request_times.popleft()
    
    def record_tokens(self, input_tokens: int, output_tokens: int):
        self.token_usage["input"] += input_tokens
        self.token_usage["output"] += output_tokens
    
    def get_stats(self) -> dict:
        now = time.time()
        cutoff = now - self.window
        
        recent = [(t, ep, elapsed) for t, ep, elapsed in self.request_times if t >= cutoff]
        
        if recent:
            avg_time = sum(e for _, _, e in recent) / len(recent)
            p95_time = sorted(e for _, _, e in recent)[int(len(recent) * 0.95)]
        else:
            avg_time = p95_time = 0
        
        return {
            "requests_per_hour": len(recent),
            "avg_response_time_ms": avg_time * 1000,
            "p95_response_time_ms": p95_time * 1000,
            "total_errors": sum(self.error_counts.values()),
            "token_usage": self.token_usage,
            "document_count": self.document_count,
            "query_count": self.query_count,
        }

# Global metrics instance
metrics = MetricsCollector()
```

---

## สรุป

โปรเจกต์ AI Document Assistant ใน Part 85 นี้ครอบคลุม:

### Components ทั้งหมด

1. **FastAPI Backend**
   - REST API endpoints (documents, conversations, chat)
   - Background processing สำหรับ document indexing
   - Streaming responses
   - Error handling และ validation
   - CORS และ security

2. **Document Processing**
   - รองรับ PDF, DOCX, TXT, MD, CSV, JSON
   - Recursive text splitting with overlap
   - Metadata extraction

3. **Vector Store (ChromaDB)**
   - Sentence Transformers embeddings
   - Cosine similarity search
   - Filtered retrieval by document ID
   - Persistent storage

4. **Claude Integration**
   - Messages API
   - Streaming responses
   - Source citation extraction
   - Conversation history management

5. **Frontend**
   - Single-page HTML app
   - Real-time chat UI
   - Document upload with drag & drop
   - Document selection for context

6. **Infrastructure**
   - Docker containerization
   - Nginx reverse proxy
   - Health monitoring
   - Production-ready configuration

### Lines of Code

| File | Approximate Lines |
|------|------------------|
| main.py | 250+ |
| models.py | 150+ |
| config.py | 60+ |
| file_parser.py | 150+ |
| text_splitter.py | 150+ |
| vector_service.py | 200+ |
| llm_service.py | 180+ |
| frontend/index.html | 200+ |
| docker-compose.yml | 40+ |
| requirements.txt | 25+ |
| tests | 200+ |
| **Total** | **1600+** |

### ขั้นต่อไป

สิ่งที่สามารถขยายได้:
- Add user authentication (JWT)
- Add database persistence (PostgreSQL)
- Add document versioning
- Support more file types (Excel, HTML)
- Add multi-language support
- Performance caching with Redis
- Full-text search with Elasticsearch

---

## จบ Python Course Parts 81-85

ยินดีด้วย! คุณได้เรียนรู้:

- **Part 81**: Deep Learning with PyTorch
- **Part 82**: NLP - Natural Language Processing
- **Part 83**: Computer Vision with OpenCV & PIL
- **Part 84**: LLM Integration - Anthropic API & OpenAI
- **Part 85**: Complete AI-Powered Application Project

---
*Part 85 - Project: AI Document Assistant | Python Course*
