[上一章：數據庫分片與集群](113_數據庫分片與集群.md) | [下一章：大數據處理基礎](115_大數據處理基礎.md)

# Python NoSQL數據庫應用 🔄

## 1. MongoDB 基礎應用

### 1.1 連接與基本操作

```python
from pymongo import MongoClient
from typing import Dict, List, Any
from datetime import datetime

class MongoDBManager:
    def __init__(self, host: str = 'localhost', port: int = 27017):
        self.client = MongoClient(host, port)
        
    def insert_document(self, db_name: str, collection_name: str, document: Dict):
        """插入單個文檔"""
        db = self.client[db_name]
        collection = db[collection_name]
        return collection.insert_one(document)
    
    def find_documents(self, db_name: str, collection_name: str, query: Dict) -> List[Dict]:
        """查詢文檔"""
        db = self.client[db_name]
        collection = db[collection_name]
        return list(collection.find(query))
    
    def update_document(self, db_name: str, collection_name: str, query: Dict, update: Dict):
        """更新文檔"""
        db = self.client[db_name]
        collection = db[collection_name]
        return collection.update_one(query, {'$set': update})
```

### 1.2 聚合操作

```python
class MongoAggregation:
    def __init__(self, db_manager: MongoDBManager):
        self.db_manager = db_manager
    
    def aggregate_pipeline(self, db_name: str, collection_name: str, pipeline: List[Dict]):
        """執行聚合管道操作"""
        db = self.db_manager.client[db_name]
        collection = db[collection_name]
        return list(collection.aggregate(pipeline))
    
    def group_by_example(self, db_name: str, collection_name: str, group_field: str):
        """分組聚合示例"""
        pipeline = [
            {'$group': {
                '_id': f'${group_field}',
                'count': {'$sum': 1},
                'avg_value': {'$avg': '$value'}
            }},
            {'$sort': {'count': -1}}
        ]
        return self.aggregate_pipeline(db_name, collection_name, pipeline)
```

## 2. Redis 應用

### 2.1 快取管理

```python
import redis
from typing import Any, Optional
import json

class RedisCache:
    def __init__(self, host: str = 'localhost', port: int = 6379, db: int = 0):
        self.redis = redis.Redis(host=host, port=port, db=db)
    
    def set_cache(self, key: str, value: Any, expire: int = 3600):
        """設置快取"""
        if isinstance(value, (dict, list)):
            value = json.dumps(value)
        self.redis.setex(key, expire, value)
    
    def get_cache(self, key: str) -> Optional[Any]:
        """獲取快取"""
        value = self.redis.get(key)
        if value:
            try:
                return json.loads(value)
            except json.JSONDecodeError:
                return value.decode()
        return None
```

### 2.2 分布式鎖

```python
import time
from typing import Callable

class RedisLock:
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client
    
    def acquire_lock(self, lock_name: str, expire: int = 10) -> bool:
        """獲取分布式鎖"""
        return bool(self.redis.set(
            f'lock:{lock_name}',
            str(time.time()),
            ex=expire,
            nx=True
        ))
    
    def release_lock(self, lock_name: str) -> bool:
        """釋放分布式鎖"""
        return bool(self.redis.delete(f'lock:{lock_name}'))
    
    def with_lock(self, lock_name: str, expire: int = 10):
        """上下文管理器形式的鎖"""
        def decorator(func: Callable):
            def wrapper(*args, **kwargs):
                if not self.acquire_lock(lock_name, expire):
                    raise Exception("無法獲取鎖")
                try:
                    return func(*args, **kwargs)
                finally:
                    self.release_lock(lock_name)
            return wrapper
        return decorator
```

## 3. Cassandra 應用

### 3.1 基本操作

```python
from cassandra.cluster import Cluster
from cassandra.query import SimpleStatement
from typing import List, Dict

class CassandraManager:
    def __init__(self, contact_points: List[str], keyspace: str):
        self.cluster = Cluster(contact_points)
        self.session = self.cluster.connect(keyspace)
    
    def execute_query(self, query: str, parameters: tuple = None):
        """執行CQL查詢"""
        statement = SimpleStatement(query)
        return self.session.execute(statement, parameters)
    
    def batch_insert(self, table: str, data: List[Dict]):
        """批量插入數據"""
        query = f"INSERT INTO {table} ({{columns}}) VALUES ({{placeholders}})"
        
        if not data:
            return
        
        columns = ','.join(data[0].keys())
        placeholders = ','.join(['%s'] * len(data[0]))
        query = query.format(columns=columns, placeholders=placeholders)
        
        batch = []
        for item in data:
            batch.append(tuple(item.values()))
        
        prepared = self.session.prepare(query)
        for params in batch:
            self.session.execute(prepared, params)
```

## 4. 混合持久化策略

### 4.1 多數據庫整合

```python
from typing import Dict, Any
import json

class MultiDBManager:
    def __init__(self, mongo_manager: MongoDBManager, redis_cache: RedisCache):
        self.mongo = mongo_manager
        self.cache = redis_cache
    
    def get_data(self, key: str, collection: str) -> Optional[Dict]:
        """獲取數據，優先從快取獲取"""
        # 嘗試從快取獲取
        cached_data = self.cache.get_cache(key)
        if cached_data:
            return cached_data
        
        # 從MongoDB獲取
        data = self.mongo.find_documents('db_name', collection, {'_id': key})
        if data:
            # 存入快取
            self.cache.set_cache(key, data[0])
            return data[0]
        
        return None
    
    def save_data(self, key: str, collection: str, data: Dict):
        """保存數據到多個數據庫"""
        # 保存到MongoDB
        self.mongo.insert_document('db_name', collection, data)
        
        # 更新快取
        self.cache.set_cache(key, data)
```

## 練習題 🏃

1. 實現一個使用 MongoDB 的簡單博客系統。
2. 使用 Redis 實現一個分布式計數器。
3. 設計一個使用 Cassandra 的時間序列數據存儲系統。
4. 實現一個混合使用 MongoDB 和 Redis 的購物車系統。
5. 創建一個支持多種 NoSQL 數據庫的數據同步工具。

## 小結 📝

- 學習了 MongoDB 的基本操作和聚合功能
- 掌握了 Redis 的快取和分布式鎖應用
- 理解了 Cassandra 的使用場景和操作方法
- 學會了混合使用多種 NoSQL 數據庫
- 了解了不同 NoSQL 數據庫的應用場景

## 延伸閱讀 📚

1. MongoDB 最佳實踐指南
2. Redis 設計與實現
3. Cassandra 權威指南
4. NoSQL 數據庫選型指南
5. 分布式數據庫系統原理

[上一章：數據庫分片與集群](113_數據庫分片與集群.md) | [下一章：大數據處理基礎](115_大數據處理基礎.md) 