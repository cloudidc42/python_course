# Part 92 - Elasticsearch & Full-Text Search

## บทนำ

Elasticsearch คือ distributed search and analytics engine ที่สร้างบน Apache Lucene ใช้สำหรับ full-text search, log analysis, metrics analysis และอื่น ๆ Elasticsearch เป็นส่วนหนึ่งของ Elastic Stack (ELK Stack: Elasticsearch, Logstash, Kibana)

### ทำไมต้องใช้ Elasticsearch?

- **Full-text search** ที่รวดเร็วและแม่นยำ
- **Near real-time** - ข้อมูลที่ index ค้นหาได้ภายใน 1 วินาที
- **Distributed** - scale ออก horizontally ได้ง่าย
- **Schema-flexible** - ไม่ต้องกำหนด schema ล่วงหน้าทั้งหมด
- **REST API** - ใช้งานง่าย ผ่าน HTTP
- **Rich query DSL** - query language ที่ยืดหยุ่น
- **Aggregations** - วิเคราะห์ข้อมูลได้ทันที

---

## 1. Core Concepts

### Index, Document, Shard, Replica

```
Elasticsearch Cluster
├── Node 1
│   ├── Shard 0 (Primary)     ← Index "products"
│   └── Shard 1 (Replica)
├── Node 2
│   ├── Shard 1 (Primary)
│   └── Shard 0 (Replica)
└── Node 3
    ├── Shard 2 (Primary)
    └── Shard 2 (Replica)
```

**ศัพท์สำคัญ**:
- **Index**: กลุ่มของ documents ที่มีลักษณะคล้ายกัน (เหมือน database table)
- **Document**: หน่วยข้อมูลพื้นฐาน ในรูปแบบ JSON (เหมือน row)
- **Shard**: แต่ละ index แบ่งออกเป็น shards เพื่อ distribute ข้อมูล
- **Replica**: สำเนา shard สำหรับ high availability
- **Mapping**: การกำหนด schema ของ index (เหมือน table schema)
- **Cluster**: กลุ่มของ Elasticsearch nodes

---

## 2. การติดตั้งและ Setup

### ติดตั้ง Elasticsearch

```bash
# Docker (แนะนำสำหรับ development)
docker run -d \
    --name elasticsearch \
    -p 9200:9200 \
    -e "discovery.type=single-node" \
    -e "xpack.security.enabled=false" \
    elasticsearch:8.11.0

# ทดสอบการเชื่อมต่อ
curl http://localhost:9200

# ติดตั้ง elasticsearch-py
pip install elasticsearch
pip install elasticsearch[async]  # สำหรับ async support
```

---

## 3. การเชื่อมต่อด้วย Python

### ตัวอย่างที่ 1: Basic Connection

```python
from elasticsearch import Elasticsearch
import json

# เชื่อมต่อ Elasticsearch
es = Elasticsearch(
    hosts=["http://localhost:9200"],
    # สำหรับ production ที่มี authentication:
    # http_auth=("username", "password"),
    # scheme="https",
    # port=443,
)

# ตรวจสอบ cluster health
health = es.cluster.health()
print(f"Cluster status: {health['status']}")
print(f"Number of nodes: {health['number_of_nodes']}")

# ข้อมูล cluster
info = es.info()
print(f"Elasticsearch version: {info['version']['number']}")

# ping
if es.ping():
    print("Elasticsearch is running!")
else:
    print("Cannot connect to Elasticsearch")
```

### ตัวอย่างที่ 2: Connection Configuration

```python
from elasticsearch import Elasticsearch
from elasticsearch.connection import create_ssl_context
import ssl

# การเชื่อมต่อ production พร้อม SSL
ssl_context = create_ssl_context()
ssl_context.check_hostname = False
ssl_context.verify_mode = ssl.CERT_NONE

# Connection Pool settings
es_config = Elasticsearch(
    hosts=[
        {"host": "es-node1", "port": 9200},
        {"host": "es-node2", "port": 9200},
        {"host": "es-node3", "port": 9200},
    ],
    maxsize=25,              # Pool size
    max_retries=3,           # Retry on failure
    retry_on_timeout=True,
    timeout=30,              # Request timeout
)

# Helper function
def get_es_client():
    return Elasticsearch(
        hosts=["http://localhost:9200"],
        retry_on_timeout=True,
        max_retries=3,
    )

es = get_es_client()
print("Connected to Elasticsearch")
```

---

## 4. Index Creation และ Mapping

### ตัวอย่างที่ 3: สร้าง Index พร้อม Mapping

```python
from elasticsearch import Elasticsearch

es = Elasticsearch(hosts=["http://localhost:9200"])

# สร้าง index พร้อม settings และ mapping
products_mapping = {
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0,
        "analysis": {
            "analyzer": {
                "thai_analyzer": {
                    "type": "custom",
                    "tokenizer": "standard",
                    "filter": ["lowercase", "stop"]
                },
                "autocomplete_analyzer": {
                    "type": "custom",
                    "tokenizer": "standard",
                    "filter": ["lowercase", "edge_ngram_filter"]
                }
            },
            "filter": {
                "edge_ngram_filter": {
                    "type": "edge_ngram",
                    "min_gram": 2,
                    "max_gram": 20
                }
            }
        }
    },
    "mappings": {
        "properties": {
            "id": {"type": "integer"},
            "name": {
                "type": "text",
                "analyzer": "standard",
                "fields": {
                    "keyword": {"type": "keyword"},  # สำหรับ exact match
                    "autocomplete": {
                        "type": "text",
                        "analyzer": "autocomplete_analyzer"
                    }
                }
            },
            "description": {
                "type": "text",
                "analyzer": "standard"
            },
            "category": {"type": "keyword"},
            "brand": {"type": "keyword"},
            "price": {"type": "float"},
            "original_price": {"type": "float"},
            "discount_percent": {"type": "integer"},
            "stock": {"type": "integer"},
            "rating": {"type": "float"},
            "review_count": {"type": "integer"},
            "tags": {"type": "keyword"},
            "created_at": {"type": "date"},
            "is_active": {"type": "boolean"},
            "attributes": {
                "type": "object",
                "properties": {
                    "color": {"type": "keyword"},
                    "size": {"type": "keyword"},
                    "weight": {"type": "float"}
                }
            },
            "location": {"type": "geo_point"}  # สำหรับ geo queries
        }
    }
}

# สร้าง index
index_name = "products"

if es.indices.exists(index=index_name):
    es.indices.delete(index=index_name)
    print(f"Deleted existing index: {index_name}")

response = es.indices.create(index=index_name, body=products_mapping)
print(f"Created index: {index_name}")
print(f"Acknowledged: {response['acknowledged']}")

# ตรวจสอบ mapping
mapping = es.indices.get_mapping(index=index_name)
print(f"\nMapping fields: {list(mapping[index_name]['mappings']['properties'].keys())}")
```

### ตัวอย่างที่ 4: Index Settings Management

```python
from elasticsearch import Elasticsearch

es = Elasticsearch(hosts=["http://localhost:9200"])

# ดู index settings
settings = es.indices.get_settings(index="products")
print("Index settings:")
print(f"  Shards: {settings['products']['settings']['index']['number_of_shards']}")

# อัปเดต index settings (บาง settings เปลี่ยนได้แบบ dynamic)
es.indices.put_settings(
    index="products",
    body={
        "index": {
            "refresh_interval": "30s",    # เพิ่ม performance ตอน bulk indexing
            "max_result_window": 50000    # เพิ่ม limit ของ from+size
        }
    }
)

# Index statistics
stats = es.indices.stats(index="products")
index_stats = stats['indices']['products']['total']
print(f"\nIndex stats:")
print(f"  Docs count: {index_stats['docs']['count']}")
print(f"  Store size: {index_stats['store']['size_in_bytes']} bytes")

# Refresh index (force near-real-time)
es.indices.refresh(index="products")

# Flush (commit to disk)
es.indices.flush(index="products")

# Optimize (force merge segments)
# es.indices.forcemerge(index="products", max_num_segments=1)
```

---

## 5. CRUD Operations

### ตัวอย่างที่ 5: Index (Create/Update) Documents

```python
from elasticsearch import Elasticsearch
from datetime import datetime

es = Elasticsearch(hosts=["http://localhost:9200"])

# สร้าง document ด้วย ID ที่กำหนด
response = es.index(
    index="products",
    id=1,
    body={
        "id": 1,
        "name": "MacBook Pro 14 inch",
        "description": "Powerful laptop with M3 chip, perfect for developers and creators",
        "category": "Laptops",
        "brand": "Apple",
        "price": 59900.0,
        "original_price": 65900.0,
        "discount_percent": 9,
        "stock": 50,
        "rating": 4.8,
        "review_count": 342,
        "tags": ["laptop", "apple", "m3", "professional"],
        "created_at": datetime.now().isoformat(),
        "is_active": True,
        "attributes": {
            "color": "Space Gray",
            "weight": 1.6
        }
    }
)
print(f"Indexed doc: {response['_id']}, result: {response['result']}")

# สร้าง document โดยให้ ES สร้าง ID อัตโนมัติ
response = es.index(
    index="products",
    body={
        "name": "iPhone 15 Pro",
        "category": "Smartphones",
        "price": 42900.0,
        "brand": "Apple"
    }
)
print(f"Auto ID: {response['_id']}")

# Bulk index - สร้างหลาย documents พร้อมกัน
from elasticsearch.helpers import bulk

products_data = [
    {
        "_index": "products",
        "_id": i,
        "_source": {
            "id": i,
            "name": f"Product {i}",
            "category": "Electronics",
            "price": float(i * 100),
            "brand": "TechBrand",
            "rating": 4.0 + (i % 5) * 0.2,
            "review_count": i * 10,
            "is_active": True,
            "created_at": datetime.now().isoformat()
        }
    }
    for i in range(2, 52)
]

success, failed = bulk(es, products_data, raise_on_error=False)
print(f"Bulk indexed: {success} success, {len(failed)} failed")
es.indices.refresh(index="products")
```

### ตัวอย่างที่ 6: Read, Update, Delete

```python
from elasticsearch import Elasticsearch
from elasticsearch.exceptions import NotFoundError

es = Elasticsearch(hosts=["http://localhost:9200"])

# GET - ดึง document ด้วย ID
try:
    doc = es.get(index="products", id=1)
    print(f"Document: {doc['_source']['name']}")
    print(f"Version: {doc['_version']}")
except NotFoundError:
    print("Document not found")

# EXISTS - ตรวจสอบว่า document มีอยู่หรือไม่
exists = es.exists(index="products", id=1)
print(f"Document exists: {exists}")

# GET specific fields
doc = es.get(
    index="products",
    id=1,
    _source=["name", "price", "category"]  # ดึงเฉพาะ fields ที่ต้องการ
)
print(f"Partial doc: {doc['_source']}")

# UPDATE - อัปเดต fields บางส่วน
es.update(
    index="products",
    id=1,
    body={
        "doc": {
            "price": 55900.0,
            "discount_percent": 15,
            "stock": 45
        }
    }
)

# UPDATE with script (Painless scripting)
es.update(
    index="products",
    id=1,
    body={
        "script": {
            "source": "ctx._source.review_count += params.count",
            "params": {"count": 5}
        }
    }
)

# UPSERT - update ถ้ามี, insert ถ้าไม่มี
es.update(
    index="products",
    id=999,
    body={
        "doc": {"name": "New Product", "price": 100.0},
        "doc_as_upsert": True  # สร้างใหม่ถ้ายังไม่มี
    }
)

# DELETE - ลบ document
try:
    es.delete(index="products", id=999)
    print("Document deleted")
except NotFoundError:
    print("Document not found for deletion")

# DELETE by query
es.delete_by_query(
    index="products",
    body={
        "query": {
            "term": {"brand": "TechBrand"}
        }
    }
)
print("Deleted all TechBrand products")
es.indices.refresh(index="products")
```

---

## 6. Full-Text Search Queries

### ตัวอย่างที่ 7: Match Query

```python
from elasticsearch import Elasticsearch

es = Elasticsearch(hosts=["http://localhost:9200"])

# สร้าง sample data ก่อน
from datetime import datetime
from elasticsearch.helpers import bulk

sample_products = [
    {"id": 1, "name": "Apple MacBook Pro 14", "category": "Laptops", "brand": "Apple", "price": 59900, "rating": 4.8, "description": "Professional laptop for developers", "is_active": True, "tags": ["laptop", "professional"], "created_at": "2024-01-01"},
    {"id": 2, "name": "Dell XPS 15 Laptop", "category": "Laptops", "brand": "Dell", "price": 45900, "rating": 4.6, "description": "Ultra-thin laptop with OLED display", "is_active": True, "tags": ["laptop", "ultrabook"], "created_at": "2024-01-05"},
    {"id": 3, "name": "iPhone 15 Pro Max", "category": "Smartphones", "brand": "Apple", "price": 52900, "rating": 4.9, "description": "Latest iPhone with titanium design and ProRes video", "is_active": True, "tags": ["smartphone", "5g"], "created_at": "2024-01-10"},
    {"id": 4, "name": "Samsung Galaxy S24 Ultra", "category": "Smartphones", "brand": "Samsung", "price": 47900, "rating": 4.7, "description": "Android flagship with S Pen and AI features", "is_active": True, "tags": ["smartphone", "android", "ai"], "created_at": "2024-01-15"},
    {"id": 5, "name": "Sony WH-1000XM5 Headphones", "category": "Audio", "brand": "Sony", "price": 13900, "rating": 4.8, "description": "Best noise cancelling wireless headphones", "is_active": True, "tags": ["headphones", "wireless", "anc"], "created_at": "2024-02-01"},
    {"id": 6, "name": "Apple AirPods Pro 2", "category": "Audio", "brand": "Apple", "price": 9990, "rating": 4.7, "description": "Active noise cancelling earbuds with spatial audio", "is_active": True, "tags": ["earbuds", "wireless", "anc"], "created_at": "2024-02-10"},
    {"id": 7, "name": "iPad Pro 12.9 M2", "category": "Tablets", "brand": "Apple", "price": 39900, "rating": 4.8, "description": "Professional tablet with M2 chip and mini-LED display", "is_active": True, "tags": ["tablet", "professional"], "created_at": "2024-03-01"},
    {"id": 8, "name": "Microsoft Surface Pro 9", "category": "Tablets", "brand": "Microsoft", "price": 42900, "rating": 4.5, "description": "2-in-1 tablet laptop with Windows 11", "is_active": True, "tags": ["tablet", "windows", "2in1"], "created_at": "2024-03-15"},
]

# Recreate index
if es.indices.exists(index="products"):
    es.indices.delete(index="products")

es.indices.create(index="products", body={
    "settings": {"number_of_shards": 1, "number_of_replicas": 0},
    "mappings": {
        "properties": {
            "id": {"type": "integer"},
            "name": {"type": "text", "fields": {"keyword": {"type": "keyword"}}},
            "category": {"type": "keyword"},
            "brand": {"type": "keyword"},
            "price": {"type": "float"},
            "rating": {"type": "float"},
            "description": {"type": "text"},
            "is_active": {"type": "boolean"},
            "tags": {"type": "keyword"},
            "created_at": {"type": "date"}
        }
    }
})

bulk(es, [{"_index": "products", "_id": p["id"], "_source": p} for p in sample_products])
es.indices.refresh(index="products")
print("Sample data indexed!")

# MATCH QUERY - full-text search บน field
result = es.search(
    index="products",
    body={
        "query": {
            "match": {
                "name": {
                    "query": "apple laptop",
                    "operator": "or",    # OR logic (default)
                    # "operator": "and"  # ต้องมีทุกคำ
                }
            }
        }
    }
)

print(f"\nMatch 'apple laptop':")
for hit in result['hits']['hits']:
    print(f"  [{hit['_score']:.2f}] {hit['_source']['name']}")

# MATCH_PHRASE - ค้นหาวลีที่ตรงกัน
result = es.search(
    index="products",
    body={
        "query": {
            "match_phrase": {
                "description": "noise cancelling"
            }
        }
    }
)
print(f"\nMatch phrase 'noise cancelling':")
for hit in result['hits']['hits']:
    print(f"  {hit['_source']['name']}")

# MULTI_MATCH - ค้นหาใน fields หลายๆ อัน
result = es.search(
    index="products",
    body={
        "query": {
            "multi_match": {
                "query": "professional display",
                "fields": ["name^2", "description"],  # name มีน้ำหนักมากกว่า 2 เท่า
                "type": "best_fields"
            }
        }
    }
)
print(f"\nMulti-match 'professional display':")
for hit in result['hits']['hits']:
    print(f"  [{hit['_score']:.2f}] {hit['_source']['name']}")
```

### ตัวอย่างที่ 8: Term, Range, Bool Queries

```python
from elasticsearch import Elasticsearch

es = Elasticsearch(hosts=["http://localhost:9200"])

# TERM QUERY - exact match (ไม่วิเคราะห์ text)
result = es.search(
    index="products",
    body={
        "query": {
            "term": {
                "brand": "Apple"  # keyword field ใช้ exact match
            }
        }
    }
)
print(f"Apple products: {result['hits']['total']['value']}")
for hit in result['hits']['hits']:
    print(f"  {hit['_source']['name']}")

# TERMS QUERY - match หลายค่า (เหมือน IN clause ใน SQL)
result = es.search(
    index="products",
    body={
        "query": {
            "terms": {
                "brand": ["Apple", "Sony"]
            }
        }
    }
)
print(f"\nApple or Sony: {result['hits']['total']['value']} products")

# RANGE QUERY
result = es.search(
    index="products",
    body={
        "query": {
            "range": {
                "price": {
                    "gte": 10000,   # greater than or equal
                    "lte": 50000,   # less than or equal
                }
            }
        }
    }
)
print(f"\nPrice 10,000-50,000: {result['hits']['total']['value']} products")

# Date range
result = es.search(
    index="products",
    body={
        "query": {
            "range": {
                "created_at": {
                    "gte": "2024-02-01",
                    "lte": "2024-12-31",
                    "format": "yyyy-MM-dd"
                }
            }
        }
    }
)
print(f"Added after Feb 2024: {result['hits']['total']['value']} products")

# BOOL QUERY - ผสม queries ต่าง ๆ
# must = AND (affects score)
# should = OR (affects score, not required)
# must_not = NOT (does not affect score)
# filter = AND (does not affect score, cacheable)
result = es.search(
    index="products",
    body={
        "query": {
            "bool": {
                "must": [
                    {"match": {"name": "Apple"}},
                ],
                "should": [
                    {"match": {"description": "professional"}},
                    {"match": {"description": "pro"}},
                ],
                "must_not": [
                    {"term": {"category": "Audio"}}
                ],
                "filter": [
                    {"range": {"price": {"gte": 1000}}},
                    {"term": {"is_active": True}}
                ],
                "minimum_should_match": 1
            }
        }
    }
)
print(f"\nBool query results: {result['hits']['total']['value']}")
for hit in result['hits']['hits']:
    print(f"  [{hit['_score']:.2f}] {hit['_source']['name']} - ฿{hit['_source']['price']:,.0f}")
```

### ตัวอย่างที่ 9: Fuzzy, Wildcard, Prefix Queries

```python
from elasticsearch import Elasticsearch

es = Elasticsearch(hosts=["http://localhost:9200"])

# FUZZY QUERY - ค้นหาแม้สะกดผิด
result = es.search(
    index="products",
    body={
        "query": {
            "fuzzy": {
                "name": {
                    "value": "Aple",    # สะกดผิด (Apple)
                    "fuzziness": "AUTO",  # AUTO, 0, 1, 2
                    "max_expansions": 50
                }
            }
        }
    }
)
print("Fuzzy 'Aple' (typo for Apple):")
for hit in result['hits']['hits']:
    print(f"  {hit['_source']['name']}")

# WILDCARD QUERY
result = es.search(
    index="products",
    body={
        "query": {
            "wildcard": {
                "name.keyword": {
                    "value": "*Pro*",
                    "case_insensitive": True
                }
            }
        }
    }
)
print("\nWildcard '*Pro*':")
for hit in result['hits']['hits']:
    print(f"  {hit['_source']['name']}")

# PREFIX QUERY - ขึ้นต้นด้วย prefix
result = es.search(
    index="products",
    body={
        "query": {
            "prefix": {
                "name.keyword": {
                    "value": "Apple",
                    "case_insensitive": True
                }
            }
        }
    }
)
print("\nPrefix 'Apple':")
for hit in result['hits']['hits']:
    print(f"  {hit['_source']['name']}")

# EXISTS QUERY - มี field นี้หรือไม่
result = es.search(
    index="products",
    body={
        "query": {
            "exists": {
                "field": "rating"
            }
        }
    }
)
print(f"\nDocuments with rating: {result['hits']['total']['value']}")

# IDS QUERY - ดึงด้วย IDs
result = es.search(
    index="products",
    body={
        "query": {
            "ids": {
                "values": ["1", "3", "5"]
            }
        }
    }
)
print("\nDocuments with IDs 1, 3, 5:")
for hit in result['hits']['hits']:
    print(f"  ID {hit['_id']}: {hit['_source']['name']}")
```

---

## 7. Aggregations

### ตัวอย่างที่ 10: Terms และ Stats Aggregations

```python
from elasticsearch import Elasticsearch

es = Elasticsearch(hosts=["http://localhost:9200"])

# TERMS AGGREGATION - นับจำนวนตาม category
result = es.search(
    index="products",
    body={
        "size": 0,  # ไม่ต้องดึง documents
        "aggs": {
            "by_category": {
                "terms": {
                    "field": "category",
                    "size": 10,
                    "order": {"_count": "desc"}
                },
                "aggs": {
                    "avg_price": {"avg": {"field": "price"}},
                    "max_price": {"max": {"field": "price"}},
                    "min_price": {"min": {"field": "price"}},
                    "product_count": {"value_count": {"field": "id"}}
                }
            }
        }
    }
)

print("Products by Category:")
for bucket in result['aggregations']['by_category']['buckets']:
    print(f"\n  Category: {bucket['key']} ({bucket['doc_count']} products)")
    print(f"    Avg price: ฿{bucket['avg_price']['value']:,.0f}")
    print(f"    Price range: ฿{bucket['min_price']['value']:,.0f} - ฿{bucket['max_price']['value']:,.0f}")

# STATS AGGREGATION - สถิติรวม
result = es.search(
    index="products",
    body={
        "size": 0,
        "aggs": {
            "price_stats": {
                "stats": {"field": "price"}
            },
            "rating_stats": {
                "extended_stats": {"field": "rating"}
            }
        }
    }
)

price_stats = result['aggregations']['price_stats']
print(f"\nPrice Statistics:")
print(f"  Count: {price_stats['count']}")
print(f"  Min: ฿{price_stats['min']:,.0f}")
print(f"  Max: ฿{price_stats['max']:,.0f}")
print(f"  Avg: ฿{price_stats['avg']:,.0f}")
print(f"  Sum: ฿{price_stats['sum']:,.0f}")

rating_stats = result['aggregations']['rating_stats']
print(f"\nRating Statistics:")
print(f"  Avg: {rating_stats['avg']:.2f}")
print(f"  Std deviation: {rating_stats['std_deviation']:.3f}")
```

### ตัวอย่างที่ 11: Date Histogram Aggregation

```python
from elasticsearch import Elasticsearch
from datetime import datetime

es = Elasticsearch(hosts=["http://localhost:9200"])

# สร้าง log data สำหรับตัวอย่าง
from elasticsearch.helpers import bulk

logs = [
    {"timestamp": "2024-01-15T10:00:00", "level": "ERROR", "service": "api", "message": "Database connection failed", "response_time": 5000},
    {"timestamp": "2024-01-15T10:05:00", "level": "INFO", "service": "api", "message": "Request processed", "response_time": 120},
    {"timestamp": "2024-01-15T11:00:00", "level": "WARN", "service": "worker", "message": "Queue is growing", "response_time": 800},
    {"timestamp": "2024-02-01T09:00:00", "level": "ERROR", "service": "api", "message": "Timeout error", "response_time": 30000},
    {"timestamp": "2024-02-01T09:30:00", "level": "INFO", "service": "api", "message": "Health check passed", "response_time": 5},
    {"timestamp": "2024-02-15T14:00:00", "level": "ERROR", "service": "worker", "message": "Job failed", "response_time": 0},
    {"timestamp": "2024-03-01T08:00:00", "level": "INFO", "service": "api", "message": "App started", "response_time": 200},
    {"timestamp": "2024-03-10T16:00:00", "level": "WARN", "service": "api", "message": "Slow query detected", "response_time": 2500},
]

if es.indices.exists(index="logs"):
    es.indices.delete(index="logs")

es.indices.create(index="logs", body={
    "settings": {"number_of_shards": 1, "number_of_replicas": 0},
    "mappings": {
        "properties": {
            "timestamp": {"type": "date"},
            "level": {"type": "keyword"},
            "service": {"type": "keyword"},
            "message": {"type": "text"},
            "response_time": {"type": "integer"}
        }
    }
})

bulk(es, [{"_index": "logs", "_source": log} for log in logs])
es.indices.refresh(index="logs")

# DATE HISTOGRAM AGGREGATION
result = es.search(
    index="logs",
    body={
        "size": 0,
        "aggs": {
            "logs_per_month": {
                "date_histogram": {
                    "field": "timestamp",
                    "calendar_interval": "month",
                    "format": "yyyy-MM"
                },
                "aggs": {
                    "by_level": {
                        "terms": {"field": "level"}
                    },
                    "avg_response_time": {
                        "avg": {"field": "response_time"}
                    },
                    "error_count": {
                        "filter": {"term": {"level": "ERROR"}},
                        "aggs": {
                            "count": {"value_count": {"field": "level"}}
                        }
                    }
                }
            }
        }
    }
)

print("Logs per Month:")
for bucket in result['aggregations']['logs_per_month']['buckets']:
    if bucket['doc_count'] > 0:
        print(f"\n  {bucket['key_as_string']}:")
        print(f"    Total logs: {bucket['doc_count']}")
        print(f"    Avg response time: {bucket['avg_response_time']['value']:.0f}ms")
        print(f"    Errors: {bucket['error_count']['count']['value']}")
        print(f"    By level: ", end="")
        for level_bucket in bucket['by_level']['buckets']:
            print(f"{level_bucket['key']}={level_bucket['doc_count']}", end=" ")
        print()

# PERCENTILE AGGREGATION
result = es.search(
    index="logs",
    body={
        "size": 0,
        "aggs": {
            "response_percentiles": {
                "percentiles": {
                    "field": "response_time",
                    "percents": [50, 75, 90, 95, 99]
                }
            }
        }
    }
)

print("\nResponse Time Percentiles:")
for pct, value in result['aggregations']['response_percentiles']['values'].items():
    print(f"  p{pct}: {value:.0f}ms")
```

### ตัวอย่างที่ 12: Nested Aggregations และ Pipeline Aggregations

```python
from elasticsearch import Elasticsearch

es = Elasticsearch(hosts=["http://localhost:9200"])

# Nested aggregations - หา top brands by revenue (price * review_count เป็น proxy)
result = es.search(
    index="products",
    body={
        "size": 0,
        "aggs": {
            "by_brand": {
                "terms": {
                    "field": "brand",
                    "size": 10
                },
                "aggs": {
                    "avg_price": {"avg": {"field": "price"}},
                    "avg_rating": {"avg": {"field": "rating"}},
                    "total_reviews": {"sum": {"field": "review_count"}},
                    "price_ranges": {
                        "range": {
                            "field": "price",
                            "ranges": [
                                {"to": 10000, "key": "budget"},
                                {"from": 10000, "to": 50000, "key": "mid-range"},
                                {"from": 50000, "key": "premium"}
                            ]
                        }
                    }
                }
            }
        }
    }
)

print("Brand Analysis:")
for bucket in result['aggregations']['by_brand']['buckets']:
    print(f"\n  {bucket['key']}:")
    print(f"    Products: {bucket['doc_count']}")
    print(f"    Avg price: ฿{bucket['avg_price']['value']:,.0f}")
    print(f"    Avg rating: {bucket['avg_rating']['value']:.1f}")
    
    ranges = {r['key']: r['doc_count'] for r in bucket['price_ranges']['buckets']}
    print(f"    Price distribution: {ranges}")

# FILTER AGGREGATION - วิเคราะห์เฉพาะ subset
result = es.search(
    index="products",
    body={
        "size": 0,
        "aggs": {
            "high_rated_by_category": {
                "filter": {
                    "range": {"rating": {"gte": 4.7}}
                },
                "aggs": {
                    "categories": {
                        "terms": {"field": "category"}
                    }
                }
            },
            "all_categories": {
                "terms": {"field": "category"}
            }
        }
    }
)

print("\nHigh-rated (>=4.7) products by category:")
for bucket in result['aggregations']['high_rated_by_category']['categories']['buckets']:
    print(f"  {bucket['key']}: {bucket['doc_count']} products")
```

---

## 8. Geo Queries

### ตัวอย่างที่ 13: Geo Distance และ Geo Bounding Box

```python
from elasticsearch import Elasticsearch
from elasticsearch.helpers import bulk

es = Elasticsearch(hosts=["http://localhost:9200"])

# สร้าง index สำหรับ restaurants
if es.indices.exists(index="restaurants"):
    es.indices.delete(index="restaurants")

es.indices.create(index="restaurants", body={
    "settings": {"number_of_shards": 1, "number_of_replicas": 0},
    "mappings": {
        "properties": {
            "name": {"type": "text"},
            "cuisine": {"type": "keyword"},
            "rating": {"type": "float"},
            "price_range": {"type": "keyword"},
            "location": {"type": "geo_point"},
            "delivery_available": {"type": "boolean"}
        }
    }
})

# Bangkok area restaurants
restaurants = [
    {"id": 1, "name": "Gaggan Anand", "cuisine": "Progressive Indian", "rating": 5.0, "price_range": "$$$$", "location": {"lat": 13.7481, "lon": 100.5381}, "delivery_available": False},
    {"id": 2, "name": "Jay Fai", "cuisine": "Thai Street Food", "rating": 4.9, "price_range": "$$$", "location": {"lat": 13.7532, "lon": 100.5022}, "delivery_available": False},
    {"id": 3, "name": "Nara Thai Cuisine", "cuisine": "Thai", "rating": 4.5, "price_range": "$$", "location": {"lat": 13.7419, "lon": 100.5320}, "delivery_available": True},
    {"id": 4, "name": "McDonald's Siam", "cuisine": "Fast Food", "rating": 3.8, "price_range": "$", "location": {"lat": 13.7455, "lon": 100.5340}, "delivery_available": True},
    {"id": 5, "name": "Le Normandie", "cuisine": "French", "rating": 4.9, "price_range": "$$$$", "location": {"lat": 13.7232, "lon": 100.5129}, "delivery_available": False},
    {"id": 6, "name": "Somboon Seafood", "cuisine": "Thai Seafood", "rating": 4.6, "price_range": "$$$", "location": {"lat": 13.7256, "lon": 100.5189}, "delivery_available": True},
]

bulk(es, [{"_index": "restaurants", "_id": r["id"], "_source": r} for r in restaurants])
es.indices.refresh(index="restaurants")

# GEO DISTANCE - หา restaurants ใกล้ Siam Paragon
user_location = {"lat": 13.7468, "lon": 100.5347}  # Siam Paragon

result = es.search(
    index="restaurants",
    body={
        "query": {
            "geo_distance": {
                "distance": "3km",
                "location": user_location
            }
        },
        "sort": [
            {
                "_geo_distance": {
                    "location": user_location,
                    "order": "asc",
                    "unit": "km"
                }
            }
        ]
    }
)

print(f"Restaurants within 3km of Siam Paragon:")
for hit in result['hits']['hits']:
    restaurant = hit['_source']
    distance = hit['sort'][0]
    print(f"  {restaurant['name']} ({restaurant['cuisine']}) - {distance:.2f}km - Rating: {restaurant['rating']}")

# GEO BOUNDING BOX
result = es.search(
    index="restaurants",
    body={
        "query": {
            "geo_bounding_box": {
                "location": {
                    "top_left": {"lat": 13.76, "lon": 100.49},
                    "bottom_right": {"lat": 13.72, "lon": 100.55}
                }
            }
        }
    }
)
print(f"\nRestaurants in bounding box: {result['hits']['total']['value']}")

# Combine geo with other filters
result = es.search(
    index="restaurants",
    body={
        "query": {
            "bool": {
                "filter": [
                    {
                        "geo_distance": {
                            "distance": "5km",
                            "location": user_location
                        }
                    },
                    {"term": {"delivery_available": True}},
                    {"range": {"rating": {"gte": 4.0}}}
                ]
            }
        },
        "sort": [
            {"rating": "desc"},
            {
                "_geo_distance": {
                    "location": user_location,
                    "order": "asc",
                    "unit": "km"
                }
            }
        ]
    }
)

print(f"\nNearby restaurants with delivery (rating >= 4.0):")
for hit in result['hits']['hits']:
    r = hit['_source']
    distance = hit['sort'][1]
    print(f"  ⭐{r['rating']} {r['name']} - {distance:.2f}km away")
```

---

## 9. Bulk Operations

### ตัวอย่างที่ 14: Efficient Bulk Indexing

```python
from elasticsearch import Elasticsearch
from elasticsearch.helpers import bulk, streaming_bulk, parallel_bulk
import json
import time

es = Elasticsearch(hosts=["http://localhost:9200"])

# Method 1: helpers.bulk (แนะนำสำหรับ large datasets)
def generate_actions(data_list, index_name):
    for item in data_list:
        yield {
            "_index": index_name,
            "_id": item.get("id"),
            "_source": item
        }

# สร้าง data
large_dataset = [
    {
        "id": i,
        "name": f"Product {i}",
        "category": ["Electronics", "Books", "Clothing", "Food"][i % 4],
        "price": float(i * 10 + 99),
        "rating": 3.0 + (i % 20) / 10,
        "is_active": i % 10 != 0
    }
    for i in range(1, 1001)
]

start = time.time()
success, failed = bulk(
    es,
    generate_actions(large_dataset, "bulk_test"),
    chunk_size=200,      # ส่ง 200 docs ต่อ request
    max_retries=3,
    raise_on_error=False,
    request_timeout=60
)
print(f"Bulk indexed {success} docs in {time.time()-start:.2f}s")
if failed:
    print(f"Failed: {len(failed)}")

es.indices.refresh(index="bulk_test")

# Method 2: streaming_bulk (สำหรับ memory efficiency)
def data_generator():
    """Generator function สำหรับ large datasets ที่ไม่ต้อง load ทั้งหมดใน memory"""
    for i in range(1001, 2001):
        yield {
            "_index": "bulk_test",
            "_id": i,
            "_source": {
                "id": i,
                "name": f"Product {i}",
                "category": "Generated",
                "price": float(i),
                "is_active": True
            }
        }

success_count = 0
for ok, info in streaming_bulk(es, data_generator(), chunk_size=100):
    if ok:
        success_count += 1
    else:
        print(f"Error: {info}")

print(f"Streaming bulk: {success_count} docs indexed")

# Bulk UPDATE
def update_actions():
    for i in range(1, 101):
        yield {
            "_op_type": "update",
            "_index": "bulk_test",
            "_id": i,
            "doc": {"price": float(i * 10 + 199)}  # ขึ้นราคา
        }

success, _ = bulk(es, update_actions())
print(f"Bulk updated: {success} docs")

# Bulk DELETE
def delete_actions():
    for i in range(901, 1001):
        yield {
            "_op_type": "delete",
            "_index": "bulk_test",
            "_id": i
        }

success, _ = bulk(es, delete_actions())
print(f"Bulk deleted: {success} docs")

es.indices.refresh(index="bulk_test")
count = es.count(index="bulk_test")
print(f"Total remaining: {count['count']}")
```

---

## 10. Index Aliases

### ตัวอย่างที่ 15: Index Aliases สำหรับ Zero-downtime Reindexing

```python
from elasticsearch import Elasticsearch
from elasticsearch.helpers import bulk
from datetime import datetime

es = Elasticsearch(hosts=["http://localhost:9200"])

# Pattern: ใช้ alias ชี้ไปยัง versioned index
# ทำให้ reindex ได้โดยไม่ต้อง downtime

def create_versioned_index(alias: str, version: int, mapping: dict) -> str:
    """สร้าง index พร้อม versioning"""
    index_name = f"{alias}_v{version}"
    
    if not es.indices.exists(index=index_name):
        es.indices.create(index=index_name, body=mapping)
        print(f"Created index: {index_name}")
    
    return index_name

def switch_alias(alias: str, old_index: str, new_index: str):
    """สลับ alias โดยไม่ downtime (atomic operation)"""
    actions = []
    
    # ลบ alias จาก old index
    if old_index:
        actions.append({"remove": {"index": old_index, "alias": alias}})
    
    # เพิ่ม alias ไปยัง new index
    actions.append({"add": {"index": new_index, "alias": alias}})
    
    es.indices.update_aliases(body={"actions": actions})
    print(f"Alias '{alias}' -> '{new_index}'")

mapping_v1 = {
    "settings": {"number_of_shards": 1, "number_of_replicas": 0},
    "mappings": {
        "properties": {
            "name": {"type": "text"},
            "price": {"type": "float"},
            "category": {"type": "keyword"}
        }
    }
}

# สร้าง index v1
index_v1 = create_versioned_index("catalog", 1, mapping_v1)

# Index data ใน v1
bulk(es, [
    {"_index": index_v1, "_id": i, "_source": {"name": f"Item {i}", "price": float(i * 10), "category": "A"}}
    for i in range(1, 6)
])
es.indices.refresh(index=index_v1)

# สร้าง alias ชี้ไปยัง v1
es.indices.put_alias(index=index_v1, name="catalog")
print(f"\nUsing alias 'catalog' -> v1")

# ค้นหาผ่าน alias
result = es.search(index="catalog", body={"query": {"match_all": {}}})
print(f"Documents via alias: {result['hits']['total']['value']}")

# Reindex เพื่อเพิ่ม field ใหม่
mapping_v2 = {
    "settings": {"number_of_shards": 1, "number_of_replicas": 0},
    "mappings": {
        "properties": {
            "name": {"type": "text"},
            "price": {"type": "float"},
            "category": {"type": "keyword"},
            "rating": {"type": "float"},    # field ใหม่
            "tags": {"type": "keyword"}      # field ใหม่
        }
    }
}

index_v2 = create_versioned_index("catalog", 2, mapping_v2)

# Reindex จาก v1 ไป v2
es.reindex(body={
    "source": {"index": index_v1},
    "dest": {"index": index_v2}
})
es.indices.refresh(index=index_v2)

# สลับ alias (atomic)
switch_alias("catalog", index_v1, index_v2)

# ตรวจสอบผ่าน alias
result = es.search(index="catalog", body={"query": {"match_all": {}}})
print(f"\nAfter reindex, documents via alias: {result['hits']['total']['value']}")

# ดู aliases ทั้งหมด
aliases = es.indices.get_alias(index="catalog*")
for idx, info in aliases.items():
    print(f"  Index: {idx} -> Aliases: {list(info['aliases'].keys())}")

# ลบ v1 หลัง reindex สำเร็จ
es.indices.delete(index=index_v1)
print(f"Deleted {index_v1}")
```

---

## 11. ตัวอย่างโปรแกรมจริง: Product Search Engine

### ตัวอย่างที่ 16: Complete Product Search Engine

```python
from elasticsearch import Elasticsearch
from elasticsearch.helpers import bulk
from typing import Optional, List, Dict, Any
from dataclasses import dataclass
import json

es = Elasticsearch(hosts=["http://localhost:9200"])

@dataclass
class SearchResult:
    total: int
    hits: List[Dict]
    aggregations: Dict
    took: int

class ProductSearchEngine:
    """
    Complete Product Search Engine
    Features:
    - Full-text search
    - Faceted search (category, brand, price range)
    - Sorting
    - Pagination
    - Autocomplete
    - Relevance boosting
    """
    
    INDEX_NAME = "products_search"
    
    def __init__(self, es_client):
        self.es = es_client
        self._setup_index()
    
    def _setup_index(self):
        """สร้าง index ด้วย optimized mapping"""
        if self.es.indices.exists(index=self.INDEX_NAME):
            return
        
        mapping = {
            "settings": {
                "number_of_shards": 1,
                "number_of_replicas": 0,
                "analysis": {
                    "analyzer": {
                        "product_analyzer": {
                            "type": "custom",
                            "tokenizer": "standard",
                            "filter": ["lowercase", "stop", "porter_stem"]
                        }
                    }
                }
            },
            "mappings": {
                "properties": {
                    "id": {"type": "keyword"},
                    "name": {
                        "type": "text",
                        "analyzer": "product_analyzer",
                        "boost": 3.0,
                        "fields": {
                            "keyword": {"type": "keyword"},
                            "suggest": {"type": "completion"}
                        }
                    },
                    "description": {"type": "text", "analyzer": "product_analyzer"},
                    "category": {"type": "keyword"},
                    "brand": {"type": "keyword"},
                    "price": {"type": "float"},
                    "original_price": {"type": "float"},
                    "discount_percent": {"type": "integer"},
                    "rating": {"type": "float"},
                    "review_count": {"type": "integer"},
                    "tags": {"type": "keyword"},
                    "is_active": {"type": "boolean"},
                    "in_stock": {"type": "boolean"},
                    "popularity_score": {"type": "float"}
                }
            }
        }
        
        self.es.indices.create(index=self.INDEX_NAME, body=mapping)
        print(f"Created search index: {self.INDEX_NAME}")
    
    def index_products(self, products: List[Dict]):
        """Index products"""
        actions = [
            {
                "_index": self.INDEX_NAME,
                "_id": p["id"],
                "_source": {
                    **p,
                    "popularity_score": p.get("rating", 0) * (p.get("review_count", 0) ** 0.5)
                }
            }
            for p in products
        ]
        
        success, failed = bulk(self.es, actions)
        self.es.indices.refresh(index=self.INDEX_NAME)
        return success, failed
    
    def search(
        self,
        query: str = "",
        categories: List[str] = None,
        brands: List[str] = None,
        min_price: float = None,
        max_price: float = None,
        min_rating: float = None,
        in_stock_only: bool = False,
        sort_by: str = "relevance",
        page: int = 1,
        page_size: int = 10
    ) -> SearchResult:
        """Full-featured search"""
        
        # Build filters
        filters = [{"term": {"is_active": True}}]
        
        if categories:
            filters.append({"terms": {"category": categories}})
        if brands:
            filters.append({"terms": {"brand": brands}})
        if in_stock_only:
            filters.append({"term": {"in_stock": True}})
        if min_price is not None or max_price is not None:
            price_range = {}
            if min_price is not None:
                price_range["gte"] = min_price
            if max_price is not None:
                price_range["lte"] = max_price
            filters.append({"range": {"price": price_range}})
        if min_rating is not None:
            filters.append({"range": {"rating": {"gte": min_rating}}})
        
        # Build query
        if query:
            must_query = {
                "multi_match": {
                    "query": query,
                    "fields": ["name^3", "brand^2", "description", "tags"],
                    "type": "best_fields",
                    "fuzziness": "AUTO",
                    "prefix_length": 2
                }
            }
        else:
            must_query = {"match_all": {}}
        
        # Sorting
        sort_options = {
            "relevance": ["_score", {"popularity_score": "desc"}],
            "price_asc": [{"price": "asc"}],
            "price_desc": [{"price": "desc"}],
            "rating": [{"rating": "desc"}],
            "newest": [{"id": "desc"}],
            "popularity": [{"popularity_score": "desc"}]
        }
        sort = sort_options.get(sort_by, ["_score"])
        
        # Pagination
        from_val = (page - 1) * page_size
        
        body = {
            "from": from_val,
            "size": page_size,
            "query": {
                "bool": {
                    "must": must_query,
                    "filter": filters,
                    "should": [
                        {"term": {"in_stock": True}},
                        {"range": {"rating": {"gte": 4.5}}}
                    ]
                }
            },
            "sort": sort,
            "aggs": {
                "categories": {
                    "terms": {"field": "category", "size": 10}
                },
                "brands": {
                    "terms": {"field": "brand", "size": 20}
                },
                "price_ranges": {
                    "range": {
                        "field": "price",
                        "ranges": [
                            {"to": 1000, "key": "Under ฿1,000"},
                            {"from": 1000, "to": 5000, "key": "฿1,000-5,000"},
                            {"from": 5000, "to": 20000, "key": "฿5,000-20,000"},
                            {"from": 20000, "to": 50000, "key": "฿20,000-50,000"},
                            {"from": 50000, "key": "Over ฿50,000"}
                        ]
                    }
                },
                "avg_price": {"avg": {"field": "price"}},
                "rating_distribution": {
                    "range": {
                        "field": "rating",
                        "ranges": [
                            {"from": 4.5, "key": "4.5+"},
                            {"from": 4.0, "to": 4.5, "key": "4.0-4.5"},
                            {"to": 4.0, "key": "Under 4.0"}
                        ]
                    }
                }
            },
            "highlight": {
                "fields": {
                    "name": {},
                    "description": {"number_of_fragments": 1}
                }
            }
        }
        
        result = self.es.search(index=self.INDEX_NAME, body=body)
        
        hits = []
        for hit in result['hits']['hits']:
            item = hit['_source'].copy()
            item['_score'] = hit['_score']
            item['_highlight'] = hit.get('highlight', {})
            hits.append(item)
        
        return SearchResult(
            total=result['hits']['total']['value'],
            hits=hits,
            aggregations=result.get('aggregations', {}),
            took=result['took']
        )
    
    def suggest(self, prefix: str, size: int = 5) -> List[str]:
        """Autocomplete suggestions"""
        result = self.es.search(
            index=self.INDEX_NAME,
            body={
                "suggest": {
                    "product_suggest": {
                        "prefix": prefix,
                        "completion": {
                            "field": "name.suggest",
                            "size": size,
                            "fuzzy": {"fuzziness": 1}
                        }
                    }
                }
            }
        )
        
        suggestions = []
        for option in result.get('suggest', {}).get('product_suggest', [{}])[0].get('options', []):
            suggestions.append(option['_source']['name'])
        return suggestions

# ทดสอบ
engine = ProductSearchEngine(es)

# Index sample products
sample_products = [
    {"id": "1", "name": "Apple MacBook Pro 14 M3", "description": "Professional laptop with M3 chip", "category": "Laptops", "brand": "Apple", "price": 59900, "rating": 4.8, "review_count": 342, "is_active": True, "in_stock": True, "tags": ["laptop", "apple", "m3"]},
    {"id": "2", "name": "Dell XPS 15 Laptop", "description": "Ultra-thin laptop with OLED display", "category": "Laptops", "brand": "Dell", "price": 45900, "rating": 4.6, "review_count": 215, "is_active": True, "in_stock": True, "tags": ["laptop", "oled"]},
    {"id": "3", "name": "iPhone 15 Pro Max", "description": "Latest iPhone with titanium design", "category": "Smartphones", "brand": "Apple", "price": 52900, "rating": 4.9, "review_count": 892, "is_active": True, "in_stock": True, "tags": ["iphone", "5g"]},
    {"id": "4", "name": "Samsung Galaxy S24 Ultra", "description": "Android flagship with AI features", "category": "Smartphones", "brand": "Samsung", "price": 47900, "rating": 4.7, "review_count": 456, "is_active": True, "in_stock": True, "tags": ["android", "ai"]},
    {"id": "5", "name": "Sony WH-1000XM5", "description": "Best noise cancelling headphones", "category": "Audio", "brand": "Sony", "price": 13900, "rating": 4.8, "review_count": 1205, "is_active": True, "in_stock": True, "tags": ["headphones", "anc"]},
    {"id": "6", "name": "Logitech MX Master 3", "description": "Ergonomic wireless mouse for productivity", "category": "Accessories", "brand": "Logitech", "price": 3990, "rating": 4.7, "review_count": 2341, "is_active": True, "in_stock": True, "tags": ["mouse", "wireless"]},
    {"id": "7", "name": "Samsung 27\" 4K Monitor", "description": "4K UHD monitor with HDR support", "category": "Monitors", "brand": "Samsung", "price": 18900, "rating": 4.5, "review_count": 189, "is_active": True, "in_stock": False, "tags": ["monitor", "4k", "hdr"]},
]

engine.index_products(sample_products)
print("Products indexed!")

# ค้นหา
print("\n=== Search: 'apple' ===")
results = engine.search(query="apple", page_size=5)
print(f"Found: {results.total} products (took {results.took}ms)")
for hit in results.hits:
    print(f"  [{hit['_score']:.2f}] {hit['name']} - ฿{hit['price']:,.0f}")

print("\n=== Search: Laptops under ฿50,000 ===")
results = engine.search(
    query="laptop",
    categories=["Laptops"],
    max_price=50000,
    sort_by="price_asc"
)
for hit in results.hits:
    print(f"  {hit['name']} - ฿{hit['price']:,.0f} (Rating: {hit['rating']})")

print("\n=== Facets (Categories) ===")
for bucket in results.aggregations.get('categories', {}).get('buckets', []):
    print(f"  {bucket['key']}: {bucket['doc_count']} products")
```

---

## 12. Log Analysis Dashboard

### ตัวอย่างที่ 17: Log Analysis System

```python
from elasticsearch import Elasticsearch
from elasticsearch.helpers import bulk
from datetime import datetime, timedelta
import random

es = Elasticsearch(hosts=["http://localhost:9200"])

# สร้าง index สำหรับ logs
if es.indices.exists(index="app_logs"):
    es.indices.delete(index="app_logs")

es.indices.create(index="app_logs", body={
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
    },
    "mappings": {
        "properties": {
            "timestamp": {"type": "date"},
            "level": {"type": "keyword"},
            "service": {"type": "keyword"},
            "host": {"type": "keyword"},
            "message": {"type": "text"},
            "error_type": {"type": "keyword"},
            "response_time": {"type": "integer"},
            "status_code": {"type": "integer"},
            "user_id": {"type": "keyword"},
            "endpoint": {"type": "keyword"},
            "method": {"type": "keyword"},
            "ip_address": {"type": "ip"},
            "session_id": {"type": "keyword"}
        }
    }
})

# สร้าง sample logs
services = ["api-gateway", "user-service", "payment-service", "notification-service"]
levels = ["INFO"] * 70 + ["WARN"] * 20 + ["ERROR"] * 10
endpoints = ["/api/users", "/api/products", "/api/orders", "/api/payments", "/api/auth"]
methods = ["GET", "GET", "GET", "POST", "PUT", "DELETE"]
error_types = ["DatabaseError", "TimeoutError", "ValidationError", "AuthError", None]

logs = []
base_time = datetime.now() - timedelta(days=7)

for i in range(500):
    log_time = base_time + timedelta(
        days=random.randint(0, 7),
        hours=random.randint(0, 23),
        minutes=random.randint(0, 59)
    )
    
    level = random.choice(levels)
    service = random.choice(services)
    endpoint = random.choice(endpoints)
    method = random.choice(methods)
    
    if level == "ERROR":
        response_time = random.randint(3000, 30000)
        status_code = random.choice([500, 502, 503, 504])
        error_type = random.choice(error_types[:-1])
        message = f"Error processing request: {error_type}"
    elif level == "WARN":
        response_time = random.randint(800, 3000)
        status_code = random.choice([400, 401, 403, 404])
        error_type = None
        message = f"Warning: slow response on {endpoint}"
    else:
        response_time = random.randint(10, 500)
        status_code = 200
        error_type = None
        message = f"Request processed: {method} {endpoint}"
    
    logs.append({
        "timestamp": log_time.isoformat(),
        "level": level,
        "service": service,
        "host": f"host-{random.randint(1, 5):02d}",
        "message": message,
        "error_type": error_type,
        "response_time": response_time,
        "status_code": status_code,
        "user_id": f"user_{random.randint(1, 100)}",
        "endpoint": endpoint,
        "method": method,
        "ip_address": f"10.0.{random.randint(0, 255)}.{random.randint(1, 254)}",
        "session_id": f"sess_{random.randint(1000, 9999)}"
    })

bulk(es, [{"_index": "app_logs", "_source": log} for log in logs])
es.indices.refresh(index="app_logs")
print(f"Indexed {len(logs)} logs")

class LogAnalyzer:
    def __init__(self, es_client, index: str = "app_logs"):
        self.es = es_client
        self.index = index
    
    def error_analysis(self, hours: int = 24) -> dict:
        """วิเคราะห์ errors ใน X ชั่วโมงที่ผ่านมา"""
        result = self.es.search(
            index=self.index,
            body={
                "size": 0,
                "query": {
                    "bool": {
                        "filter": [
                            {"term": {"level": "ERROR"}},
                            {"range": {"timestamp": {"gte": f"now-{hours}h"}}}
                        ]
                    }
                },
                "aggs": {
                    "by_service": {"terms": {"field": "service", "size": 10}},
                    "by_error_type": {"terms": {"field": "error_type", "size": 10}},
                    "by_hour": {
                        "date_histogram": {
                            "field": "timestamp",
                            "calendar_interval": "hour"
                        }
                    },
                    "avg_response_time": {"avg": {"field": "response_time"}},
                    "top_errors": {
                        "top_hits": {
                            "size": 3,
                            "sort": [{"timestamp": "desc"}],
                            "_source": ["message", "service", "error_type", "timestamp"]
                        }
                    }
                }
            }
        )
        
        aggs = result['aggregations']
        return {
            "total_errors": result['hits']['total']['value'],
            "by_service": {b['key']: b['doc_count'] for b in aggs['by_service']['buckets']},
            "by_error_type": {b['key']: b['doc_count'] for b in aggs['by_error_type']['buckets']},
            "avg_error_response_time": aggs['avg_response_time']['value'],
            "recent_errors": [h['_source'] for h in aggs['top_errors']['hits']['hits']]
        }
    
    def performance_report(self) -> dict:
        """รายงาน performance ของแต่ละ endpoint"""
        result = self.es.search(
            index=self.index,
            body={
                "size": 0,
                "aggs": {
                    "by_endpoint": {
                        "terms": {"field": "endpoint", "size": 20},
                        "aggs": {
                            "avg_response": {"avg": {"field": "response_time"}},
                            "p95_response": {
                                "percentiles": {
                                    "field": "response_time",
                                    "percents": [50, 90, 95, 99]
                                }
                            },
                            "error_rate": {
                                "filter": {"term": {"level": "ERROR"}},
                                "aggs": {
                                    "count": {"value_count": {"field": "level"}}
                                }
                            },
                            "total_requests": {"value_count": {"field": "endpoint"}}
                        }
                    }
                }
            }
        )
        
        report = []
        for bucket in result['aggregations']['by_endpoint']['buckets']:
            total = bucket['total_requests']['value']
            errors = bucket['error_rate']['count']['value']
            report.append({
                "endpoint": bucket['key'],
                "requests": total,
                "avg_response_ms": round(bucket['avg_response']['value']),
                "p95_response_ms": round(bucket['p95_response']['values']['95.0']),
                "error_rate": f"{errors/total*100:.1f}%"
            })
        
        return sorted(report, key=lambda x: x['avg_response_ms'], reverse=True)
    
    def search_logs(self, query: str, level: str = None, service: str = None) -> list:
        """ค้นหา logs"""
        filters = []
        if level:
            filters.append({"term": {"level": level}})
        if service:
            filters.append({"term": {"service": service}})
        
        result = self.es.search(
            index=self.index,
            body={
                "size": 20,
                "query": {
                    "bool": {
                        "must": {"match": {"message": query}},
                        "filter": filters
                    }
                },
                "sort": [{"timestamp": "desc"}],
                "highlight": {"fields": {"message": {}}}
            }
        )
        
        return [{
            "timestamp": h['_source']['timestamp'],
            "level": h['_source']['level'],
            "service": h['_source']['service'],
            "message": h['highlight'].get('message', [h['_source']['message']])[0]
        } for h in result['hits']['hits']]

# ทดสอบ
analyzer = LogAnalyzer(es)

print("\n=== Error Analysis (last 168h) ===")
error_report = analyzer.error_analysis(hours=168)
print(f"Total errors: {error_report['total_errors']}")
print(f"Avg error response time: {error_report['avg_error_response_time']:.0f}ms")
print("Errors by service:")
for service, count in error_report['by_service'].items():
    print(f"  {service}: {count}")
print("Recent errors:")
for err in error_report['recent_errors'][:2]:
    print(f"  [{err['service']}] {err['message']}")

print("\n=== Performance Report ===")
for endpoint in analyzer.performance_report()[:5]:
    print(f"  {endpoint['endpoint']:30s} avg:{endpoint['avg_response_ms']:5d}ms p95:{endpoint['p95_response_ms']:5d}ms errors:{endpoint['error_rate']}")

print("\n=== Search Logs ===")
results = analyzer.search_logs("Error", level="ERROR", service="payment-service")
print(f"Found {len(results)} payment errors")
```

---

## 13. แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Blog Search Engine

**โจทย์**: สร้าง Blog Search Engine ที่:
- Index blog posts พร้อม title, content, author, tags, published_at
- ค้นหา full-text ใน title และ content
- Filter ด้วย author และ tags
- Highlight คำที่ค้นเจอ
- แสดง aggregations (ยอดนิยมตาม tag, author)

```python
# เฉลย
from elasticsearch import Elasticsearch
from elasticsearch.helpers import bulk
from datetime import datetime

es = Elasticsearch(hosts=["http://localhost:9200"])

# Setup index
if es.indices.exists(index="blog"):
    es.indices.delete(index="blog")

es.indices.create(index="blog", body={
    "settings": {"number_of_shards": 1, "number_of_replicas": 0},
    "mappings": {
        "properties": {
            "title": {"type": "text", "boost": 2.0},
            "content": {"type": "text"},
            "author": {"type": "keyword"},
            "tags": {"type": "keyword"},
            "published_at": {"type": "date"},
            "views": {"type": "integer"},
            "likes": {"type": "integer"}
        }
    }
})

# Sample data
posts = [
    {"id": 1, "title": "Getting started with Python", "content": "Python is a versatile programming language perfect for beginners and experts alike.", "author": "Alice", "tags": ["python", "programming", "beginner"], "published_at": "2024-01-10", "views": 1500, "likes": 120},
    {"id": 2, "title": "Advanced Redis Caching", "content": "Learn how to implement efficient caching strategies with Redis in your Python applications.", "author": "Bob", "tags": ["redis", "caching", "python", "performance"], "published_at": "2024-02-15", "views": 890, "likes": 75},
    {"id": 3, "title": "Elasticsearch Full-Text Search", "content": "Elasticsearch provides powerful full-text search capabilities for your applications.", "author": "Alice", "tags": ["elasticsearch", "search", "python"], "published_at": "2024-03-01", "views": 2100, "likes": 180},
    {"id": 4, "title": "FastAPI Best Practices", "content": "Building high-performance REST APIs with FastAPI and Python async features.", "author": "Charlie", "tags": ["fastapi", "python", "api", "async"], "published_at": "2024-03-20", "views": 1750, "likes": 145},
    {"id": 5, "title": "Docker for Python Developers", "content": "Containerize your Python applications with Docker for consistent deployments.", "author": "Bob", "tags": ["docker", "python", "devops"], "published_at": "2024-04-05", "views": 1200, "likes": 95},
]

bulk(es, [{"_index": "blog", "_id": p["id"], "_source": p} for p in posts])
es.indices.refresh(index="blog")

def blog_search(query: str, author: str = None, tags: list = None):
    filters = []
    if author:
        filters.append({"term": {"author": author}})
    if tags:
        filters.append({"terms": {"tags": tags}})
    
    result = es.search(
        index="blog",
        body={
            "query": {
                "bool": {
                    "must": {"multi_match": {"query": query, "fields": ["title^2", "content"]}},
                    "filter": filters
                }
            },
            "highlight": {
                "fields": {"title": {}, "content": {"fragment_size": 150}}
            },
            "aggs": {
                "popular_tags": {"terms": {"field": "tags", "size": 10}},
                "by_author": {"terms": {"field": "author"}}
            }
        }
    )
    
    print(f"Found {result['hits']['total']['value']} posts for '{query}':")
    for hit in result['hits']['hits']:
        title_hl = hit.get('highlight', {}).get('title', [hit['_source']['title']])[0]
        print(f"  [{hit['_score']:.2f}] {title_hl}")
    
    print("Popular tags:")
    for bucket in result['aggregations']['popular_tags']['buckets'][:5]:
        print(f"  #{bucket['key']}: {bucket['doc_count']}")

blog_search("python caching", tags=["python"])
```

### แบบฝึกหัดที่ 2: Real-time Inventory Search

**โจทย์**: ระบบค้นหาสินค้าคลัง:
- ค้นหาด้วยชื่อ, SKU, หมวดหมู่
- Filter สินค้าที่ stock ต่ำกว่า threshold
- Alert เมื่อ stock เป็น 0
- รายงาน stock value รวม

```python
# เฉลย
from elasticsearch import Elasticsearch
from elasticsearch.helpers import bulk

es = Elasticsearch(hosts=["http://localhost:9200"])

if es.indices.exists(index="inventory"):
    es.indices.delete(index="inventory")

es.indices.create(index="inventory", body={
    "settings": {"number_of_shards": 1, "number_of_replicas": 0},
    "mappings": {
        "properties": {
            "sku": {"type": "keyword"},
            "name": {"type": "text"},
            "category": {"type": "keyword"},
            "price": {"type": "float"},
            "cost": {"type": "float"},
            "stock": {"type": "integer"},
            "reorder_point": {"type": "integer"},
            "warehouse": {"type": "keyword"}
        }
    }
})

items = [
    {"sku": "LAPTOP-001", "name": "Gaming Laptop", "category": "Electronics", "price": 45000, "cost": 30000, "stock": 5, "reorder_point": 10, "warehouse": "BKK-01"},
    {"sku": "MOUSE-001", "name": "Wireless Mouse", "category": "Accessories", "price": 1500, "cost": 800, "stock": 0, "reorder_point": 50, "warehouse": "BKK-01"},
    {"sku": "KEYBOARD-001", "name": "Mechanical Keyboard", "category": "Accessories", "price": 3500, "cost": 2000, "stock": 25, "reorder_point": 20, "warehouse": "BKK-02"},
    {"sku": "MONITOR-001", "name": "4K Monitor 27 inch", "category": "Electronics", "price": 18000, "cost": 12000, "stock": 3, "reorder_point": 5, "warehouse": "BKK-01"},
    {"sku": "HEADSET-001", "name": "Gaming Headset", "category": "Audio", "price": 4500, "cost": 2500, "stock": 30, "reorder_point": 15, "warehouse": "BKK-02"},
]

bulk(es, [{"_index": "inventory", "_id": i["sku"], "_source": i} for i in items])
es.indices.refresh(index="inventory")

# หา items ที่ต้องสั่งซื้อ (stock <= reorder_point)
result = es.search(
    index="inventory",
    body={
        "query": {
            "script": {
                "script": "doc['stock'].value <= doc['reorder_point'].value"
            }
        },
        "sort": [{"stock": "asc"}]
    }
)

print("Items needing reorder:")
for hit in result['hits']['hits']:
    item = hit['_source']
    status = "OUT OF STOCK" if item['stock'] == 0 else f"Low ({item['stock']} left)"
    print(f"  [{status}] {item['sku']}: {item['name']} - reorder point: {item['reorder_point']}")

# Total inventory value
result = es.search(
    index="inventory",
    body={
        "size": 0,
        "aggs": {
            "total_retail_value": {
                "sum": {
                    "script": "doc['price'].value * doc['stock'].value"
                }
            },
            "by_category": {
                "terms": {"field": "category"},
                "aggs": {
                    "stock_value": {
                        "sum": {"script": "doc['price'].value * doc['stock'].value"}
                    }
                }
            }
        }
    }
)

total = result['aggregations']['total_retail_value']['value']
print(f"\nTotal inventory retail value: ฿{total:,.2f}")
```

### แบบฝึกหัดที่ 3-8: (หัวข้อและ hints)

**แบบฝึกหัดที่ 3: E-commerce Filter System**
สร้างระบบ filter สินค้า E-commerce แบบ faceted search

**แบบฝึกหัดที่ 4: Job Search Engine**
สร้าง Job Search Engine ค้นหาตำแหน่งงาน ด้วย location, salary range, skills

**แบบฝึกหัดที่ 5: News Recommendation**
ระบบแนะนำข่าว More Like This บน Elasticsearch

```python
# แบบฝึกหัดที่ 5: More Like This Query
from elasticsearch import Elasticsearch

es = Elasticsearch(hosts=["http://localhost:9200"])

# More Like This - หาบทความที่คล้ายกัน
result = es.search(
    index="blog",
    body={
        "query": {
            "more_like_this": {
                "fields": ["title", "content", "tags"],
                "like": [
                    {"_index": "blog", "_id": "1"}  # find articles like post 1
                ],
                "min_term_freq": 1,
                "max_query_terms": 12,
                "min_doc_freq": 1
            }
        }
    }
)

print("Articles similar to post 1:")
for hit in result['hits']['hits']:
    print(f"  [{hit['_score']:.2f}] {hit['_source']['title']}")
```

**แบบฝึกหัดที่ 6: Security Log Analyzer**
วิเคราะห์ security logs หา suspicious IPs, brute force attacks

**แบบฝึกหัดที่ 7: Analytics Dashboard**
สร้าง analytics dashboard แสดง time-series metrics

**แบบฝึกหัดที่ 8: Percolator API**
ใช้ Percolator สำหรับ real-time document classification

```python
# แบบฝึกหัดที่ 8: Percolator - Reverse Search
# แทนที่จะค้นหา documents ด้วย query
# Percolator ทำตรงกันข้าม: หา queries ที่ match กับ document ใหม่
# ใช้สำหรับ: alerts, notifications, content classification

from elasticsearch import Elasticsearch

es = Elasticsearch(hosts=["http://localhost:9200"])

if es.indices.exists(index="alerts"):
    es.indices.delete(index="alerts")

es.indices.create(index="alerts", body={
    "mappings": {
        "properties": {
            "query": {"type": "percolator"},
            "name": {"type": "text"},
            "category": {"type": "keyword"},
            "threshold": {"type": "float"}
        }
    }
})

# Register alert queries
alert_queries = [
    {
        "name": "High Error Rate Alert",
        "category": "performance",
        "query": {
            "bool": {
                "must": [
                    {"term": {"level": "ERROR"}},
                    {"range": {"response_time": {"gte": 5000}}}
                ]
            }
        }
    },
    {
        "name": "Payment Service Down",
        "category": "critical",
        "query": {
            "bool": {
                "must": [
                    {"term": {"service": "payment-service"}},
                    {"term": {"level": "ERROR"}}
                ]
            }
        }
    }
]

for i, alert in enumerate(alert_queries):
    es.index(index="alerts", id=i+1, body=alert)

es.indices.refresh(index="alerts")

# Check ว่า log ใหม่ trigger alert ไหน
new_log = {
    "level": "ERROR",
    "service": "payment-service",
    "response_time": 8000,
    "message": "Database timeout"
}

result = es.search(
    index="alerts",
    body={
        "query": {
            "percolate": {
                "field": "query",
                "document": new_log
            }
        }
    }
)

print(f"Triggered alerts for new log:")
for hit in result['hits']['hits']:
    print(f"  ALERT: {hit['_source']['name']} (category: {hit['_source']['category']})")
```

---

## สรุป

Elasticsearch เป็น powerful search engine ที่เหมาะสำหรับ:

| Use Case | เหมาะสม |
|----------|---------|
| Full-text search | มาก |
| Log analysis | มาก |
| Real-time analytics | มาก |
| Product catalog | มาก |
| Geospatial queries | มาก |
| ACID transactions | น้อย (ใช้ RDBMS) |
| Complex joins | น้อย (ใช้ RDBMS) |

### Query Types Summary

```
Query DSL
├── Full-text queries
│   ├── match (analyzed text search)
│   ├── match_phrase (exact phrase)
│   ├── multi_match (multiple fields)
│   └── match_all (all documents)
├── Term-level queries
│   ├── term (exact keyword)
│   ├── terms (multiple values)
│   ├── range (numeric/date range)
│   ├── exists (field exists)
│   ├── prefix (starts with)
│   ├── wildcard (* ?)
│   └── ids (by document IDs)
├── Compound queries
│   ├── bool (must/should/must_not/filter)
│   ├── constant_score (fixed score)
│   └── function_score (custom scoring)
└── Specialized queries
    ├── fuzzy (typo tolerant)
    ├── more_like_this (similar docs)
    ├── geo_distance (by location)
    ├── geo_bounding_box (in area)
    └── percolate (reverse search)
```

### Best Practices

1. **ใช้ filter แทน query** เมื่อไม่ต้องการ relevance scoring (cache ได้)
2. **ตั้ง number_of_replicas=0** ตอน bulk indexing แล้วเพิ่มทีหลัง
3. **ใช้ bulk API** สำหรับ indexing จำนวนมาก
4. **หลีกเลี่ยง deep pagination** ใช้ search_after แทน from+size สูงๆ
5. **ใช้ keyword field** สำหรับ sorting และ aggregations
6. **Monitor index size** และทำ index lifecycle management
7. **ใช้ aliases** เพื่อ zero-downtime reindexing

```python
# Search After - efficient deep pagination
result = es.search(
    index="products",
    body={
        "size": 10,
        "query": {"match_all": {}},
        "sort": [
            {"price": "asc"},
            {"_id": "asc"}  # tie-breaker
        ]
    }
)

# ใช้ sort values ของ hit สุดท้ายสำหรับ page ถัดไป
last_hit = result['hits']['hits'][-1]
search_after = last_hit['sort']

next_page = es.search(
    index="products",
    body={
        "size": 10,
        "query": {"match_all": {}},
        "sort": [{"price": "asc"}, {"_id": "asc"}],
        "search_after": search_after  # ดึงหลัง cursor
    }
)
print(f"Next page: {len(next_page['hits']['hits'])} results")
```

---

## แหล่งเรียนรู้เพิ่มเติม

- [Elasticsearch Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/)
- [elasticsearch-py Documentation](https://elasticsearch-py.readthedocs.io/)
- [Elasticsearch: The Definitive Guide](https://www.elastic.co/guide/en/elasticsearch/guide/current/index.html)
- [Elastic Learning Center](https://www.elastic.co/training/)
