[上一章：雲端部署與服務](059_雲端部署與服務.md)

# Python 性能優化 ⚡

## 代碼優化

### 1. 數據結構選擇

```python
from collections import defaultdict, Counter, deque
import timeit
from typing import List, Dict, Set
import random

def compare_data_structures():
    """比較不同數據結構的性能"""
    # 準備測試數據
    data = [random.randint(1, 100) for _ in range(10000)]
    
    # 列表操作
    def list_operations():
        numbers = []
        for n in data:
            if n not in numbers:  # O(n)
                numbers.append(n)
        return len(numbers)
    
    # 集合操作
    def set_operations():
        numbers = set()
        for n in data:
            numbers.add(n)  # O(1)
        return len(numbers)
    
    # 字典操作
    def dict_operations():
        numbers = {}
        for n in data:
            numbers[n] = True  # O(1)
        return len(numbers)
    
    # 使用Counter
    def counter_operations():
        return len(Counter(data))
    
    # 計時比較
    times = {
        'list': timeit.timeit(list_operations, number=100),
        'set': timeit.timeit(set_operations, number=100),
        'dict': timeit.timeit(dict_operations, number=100),
        'counter': timeit.timeit(counter_operations, number=100)
    }
    
    return times

# 使用生成器
def generate_large_data():
    """使用生成器處理大量數據"""
    return (x * x for x in range(1000000))

# 使用deque
def compare_deque_list():
    """比較deque和list的性能"""
    # 準備測試數據
    n = 100000
    
    def list_operations():
        lst = []
        for i in range(n):
            lst.insert(0, i)  # O(n)
        for i in range(n):
            lst.pop(0)  # O(n)
    
    def deque_operations():
        d = deque()
        for i in range(n):
            d.appendleft(i)  # O(1)
        for i in range(n):
            d.popleft()  # O(1)
    
    times = {
        'list': timeit.timeit(list_operations, number=1),
        'deque': timeit.timeit(deque_operations, number=1)
    }
    
    return times

# 使用示例
def main():
    # 比較數據結構
    print("數據結構性能比較:")
    times = compare_data_structures()
    for structure, time in times.items():
        print(f"{structure}: {time:.4f} 秒")
    
    # 使用生成器
    data = generate_large_data()
    print("\n使用生成器處理大量數據:")
    print("前10個元素:", list(itertools.islice(data, 10)))
    
    # 比較deque和list
    print("\ndeque vs list性能比較:")
    times = compare_deque_list()
    for structure, time in times.items():
        print(f"{structure}: {time:.4f} 秒")

if __name__ == '__main__':
    main()
```

### 2. 算法優化

```python
import time
from functools import lru_cache
import numpy as np

# 使用緩存裝飾器
@lru_cache(maxsize=None)
def fibonacci_cached(n: int) -> int:
    """使用緩存的斐波那契數列計算"""
    if n < 2:
        return n
    return fibonacci_cached(n-1) + fibonacci_cached(n-2)

def fibonacci_iterative(n: int) -> int:
    """迭代方式計算斐波那契數列"""
    if n < 2:
        return n
    a, b = 0, 1
    for _ in range(2, n + 1):
        a, b = b, a + b
    return b

def compare_fibonacci_implementations(n: int):
    """比較不同實現方式的性能"""
    # 遞歸實現（帶緩存）
    start = time.time()
    result1 = fibonacci_cached(n)
    time1 = time.time() - start
    
    # 迭代實現
    start = time.time()
    result2 = fibonacci_iterative(n)
    time2 = time.time() - start
    
    return {
        'recursive_cached': {'time': time1, 'result': result1},
        'iterative': {'time': time2, 'result': result2}
    }

# 使用NumPy進行向量化運算
def vector_operations():
    """比較普通循環和向量化運算的性能"""
    # 準備數據
    size = 1000000
    data = list(range(size))
    array = np.array(data)
    
    # 使用循環
    def loop_operation():
        result = []
        for x in data:
            result.append(x * x + 2 * x + 1)
        return result
    
    # 使用NumPy向量化
    def numpy_operation():
        return array * array + 2 * array + 1
    
    # 計時比較
    start = time.time()
    result1 = loop_operation()
    time1 = time.time() - start
    
    start = time.time()
    result2 = numpy_operation()
    time2 = time.time() - start
    
    return {
        'loop': {'time': time1, 'result': result1[:5]},
        'numpy': {'time': time2, 'result': result2[:5].tolist()}
    }

# 使用示例
def main():
    # 比較斐波那契數列實現
    n = 35
    print(f"計算斐波那契數列第{n}項:")
    results = compare_fibonacci_implementations(n)
    for impl, data in results.items():
        print(f"{impl}: {data['time']:.6f} 秒, 結果: {data['result']}")
    
    # 比較向量化運算
    print("\n向量化運算比較:")
    results = vector_operations()
    for method, data in results.items():
        print(f"{method}: {data['time']:.6f} 秒")
        print(f"前5個結果: {data['result']}")

if __name__ == '__main__':
    main()
```

## 內存管理

### 1. 內存優化

```python
import sys
import gc
import weakref
from memory_profiler import profile

class LargeObject:
    def __init__(self, data):
        self.data = data

def show_memory_usage(obj):
    """顯示對象的內存使用情況"""
    return sys.getsizeof(obj)

@profile
def memory_intensive_operation():
    """內存密集型操作示例"""
    # 創建大量數據
    large_list = [LargeObject([i] * 1000) for i in range(1000)]
    
    # 處理數據
    result = []
    for obj in large_list:
        result.append(sum(obj.data))
    
    # 清理不需要的數據
    large_list.clear()
    gc.collect()
    
    return result

class Cache:
    """使用弱引用的緩存實現"""
    def __init__(self):
        self._cache = weakref.WeakValueDictionary()
    
    def get(self, key):
        return self._cache.get(key)
    
    def set(self, key, value):
        self._cache[key] = value

def optimize_memory_usage():
    """內存使用優化示例"""
    # 使用生成器而不是列表
    def generate_data():
        for i in range(1000000):
            yield i * i
    
    # 使用迭代器處理數據
    def process_data():
        total = 0
        for num in generate_data():
            total += num
        return total
    
    # 使用弱引用緩存
    cache = Cache()
    
    # 緩存計算結果
    result = process_data()
    cache.set('result', result)
    
    # 獲取緩存的結果
    cached_result = cache.get('result')
    
    return cached_result

# 使用示例
def main():
    # 顯示不同數據結構的內存使用
    data_structures = {
        'list': [i for i in range(1000)],
        'set': {i for i in range(1000)},
        'dict': {i: i for i in range(1000)}
    }
    
    print("內存使用情況:")
    for name, ds in data_structures.items():
        print(f"{name}: {show_memory_usage(ds)} bytes")
    
    # 運行內存密集型操作
    print("\n運行內存密集型操作...")
    result = memory_intensive_operation()
    print(f"操作完成，結果大小: {len(result)}")
    
    # 運行優化後的代碼
    print("\n運行優化後的代碼...")
    result = optimize_memory_usage()
    print(f"計算結果: {result}")

if __name__ == '__main__':
    main()
```

### 2. 垃圾回收

```python
import gc
import weakref
import time
from typing import Dict, List

class Resource:
    def __init__(self, name: str):
        self.name = name
        print(f"Resource {name} created")
    
    def __del__(self):
        print(f"Resource {self.name} destroyed")

def demonstrate_gc():
    """演示垃圾回收機制"""
    # 創建循環引用
    def create_cycle():
        l1 = []
        l2 = []
        l1.append(l2)
        l2.append(l1)
        return "Cycle created"
    
    # 強制垃圾回收
    print("Before collecting:")
    print(f"Garbage objects: {len(gc.get_objects())}")
    
    create_cycle()
    
    print("\nAfter creating cycle:")
    print(f"Garbage objects: {len(gc.get_objects())}")
    
    gc.collect()
    
    print("\nAfter garbage collection:")
    print(f"Garbage objects: {len(gc.get_objects())}")

class ResourceManager:
    """資源管理器示例"""
    def __init__(self):
        self._resources: Dict[str, weakref.ref] = {}
    
    def add_resource(self, name: str) -> Resource:
        """添加資源"""
        resource = Resource(name)
        self._resources[name] = weakref.ref(resource)
        return resource
    
    def get_resource(self, name: str) -> Resource:
        """獲取資源"""
        if name in self._resources:
            resource = self._resources[name]()
            if resource is not None:
                return resource
            else:
                del self._resources[name]
        return None
    
    def cleanup(self):
        """清理失效的引用"""
        for name, ref in list(self._resources.items()):
            if ref() is None:
                del self._resources[name]

def memory_leak_prevention():
    """內存洩漏預防示例"""
    # 使用資源管理器
    manager = ResourceManager()
    
    # 創建資源
    resource1 = manager.add_resource("resource1")
    resource2 = manager.add_resource("resource2")
    
    # 使用資源
    print("\n使用資源:")
    print(f"Resource1: {manager.get_resource('resource1').name}")
    print(f"Resource2: {manager.get_resource('resource2').name}")
    
    # 刪除一個資源
    del resource1
    
    # 清理
    manager.cleanup()
    
    # 檢查資源狀態
    print("\n資源狀態:")
    print(f"Resource1 exists: {manager.get_resource('resource1') is not None}")
    print(f"Resource2 exists: {manager.get_resource('resource2') is not None}")

# 使用示例
def main():
    # 演示垃圾回收
    print("垃圾回收演示:")
    demonstrate_gc()
    
    # 演示內存洩漏預防
    print("\n內存洩漏預防演示:")
    memory_leak_prevention()

if __name__ == '__main__':
    main()
```

## 並發優化

### 1. 多線程與多進程

```python
import threading
import multiprocessing
import time
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor
from typing import List, Callable
import math

def cpu_intensive_task(n: int) -> int:
    """CPU密集型任務"""
    return sum(i * i for i in range(n))

def io_intensive_task(seconds: float):
    """IO密集型任務"""
    time.sleep(seconds)
    return f"Slept for {seconds} seconds"

class PerformanceComparison:
    def __init__(self):
        self.numbers = [100000 + i for i in range(10)]
    
    def run_sequential(self, task: Callable, args: List):
        """順序執行"""
        start = time.time()
        results = [task(arg) for arg in args]
        end = time.time()
        return results, end - start
    
    def run_threaded(self, task: Callable, args: List,
                    max_workers: int = None):
        """多線程執行"""
        start = time.time()
        with ThreadPoolExecutor(max_workers=max_workers) as executor:
            results = list(executor.map(task, args))
        end = time.time()
        return results, end - start
    
    def run_multiprocess(self, task: Callable, args: List,
                        max_workers: int = None):
        """多進程執行"""
        start = time.time()
        with ProcessPoolExecutor(max_workers=max_workers) as executor:
            results = list(executor.map(task, args))
        end = time.time()
        return results, end - start
    
    def compare_execution_methods(self):
        """比較不同執行方法"""
        # CPU密集型任務比較
        print("CPU密集型任務比較:")
        
        results = {}
        
        # 順序執行
        cpu_results, cpu_time = self.run_sequential(
            cpu_intensive_task, self.numbers
        )
        results['sequential'] = cpu_time
        
        # 多線程執行
        cpu_results, cpu_time = self.run_threaded(
            cpu_intensive_task, self.numbers
        )
        results['threaded'] = cpu_time
        
        # 多進程執行
        cpu_results, cpu_time = self.run_multiprocess(
            cpu_intensive_task, self.numbers
        )
        results['multiprocess'] = cpu_time
        
        print("CPU密集型任務執行時間:")
        for method, time_taken in results.items():
            print(f"{method}: {time_taken:.4f} 秒")
        
        # IO密集型任務比較
        print("\nIO密集型任務比較:")
        io_args = [0.1] * 10  # 10個0.1秒的延遲
        
        results = {}
        
        # 順序執行
        io_results, io_time = self.run_sequential(
            io_intensive_task, io_args
        )
        results['sequential'] = io_time
        
        # 多線程執行
        io_results, io_time = self.run_threaded(
            io_intensive_task, io_args
        )
        results['threaded'] = io_time
        
        # 多進程執行
        io_results, io_time = self.run_multiprocess(
            io_intensive_task, io_args
        )
        results['multiprocess'] = io_time
        
        print("IO密集型任務執行時間:")
        for method, time_taken in results.items():
            print(f"{method}: {time_taken:.4f} 秒")

# 使用示例
def main():
    comparison = PerformanceComparison()
    comparison.compare_execution_methods()

if __name__ == '__main__':
    main()
```

### 2. 異步編程

```python
import asyncio
import aiohttp
import time
from typing import List, Dict
import random

async def async_task(task_id: int, delay: float) -> Dict:
    """異步任務示例"""
    await asyncio.sleep(delay)
    return {
        'task_id': task_id,
        'delay': delay,
        'timestamp': time.time()
    }

async def async_http_request(url: str) -> Dict:
    """異步HTTP請求"""
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return {
                'url': url,
                'status': response.status,
                'timestamp': time.time()
            }

class AsyncPerformanceTest:
    def __init__(self):
        self.tasks = [
            (i, random.uniform(0.1, 0.5))
            for i in range(10)
        ]
        self.urls = [
            'http://example.com',
            'http://google.com',
            'http://github.com'
        ] * 3
    
    async def run_tasks_sequential(self) -> List[Dict]:
        """順序執行任務"""
        results = []
        for task_id, delay in self.tasks:
            result = await async_task(task_id, delay)
            results.append(result)
        return results
    
    async def run_tasks_concurrent(self) -> List[Dict]:
        """並發執行任務"""
        tasks = [
            async_task(task_id, delay)
            for task_id, delay in self.tasks
        ]
        return await asyncio.gather(*tasks)
    
    async def run_http_requests_sequential(self) -> List[Dict]:
        """順序執行HTTP請求"""
        results = []
        for url in self.urls:
            try:
                result = await async_http_request(url)
                results.append(result)
            except Exception as e:
                results.append({
                    'url': url,
                    'error': str(e),
                    'timestamp': time.time()
                })
        return results
    
    async def run_http_requests_concurrent(self) -> List[Dict]:
        """並發執行HTTP請求"""
        tasks = [
            async_http_request(url)
            for url in self.urls
        ]
        return await asyncio.gather(*tasks, return_exceptions=True)
    
    async def compare_performance(self):
        """比較性能"""
        # 比較普通任務
        print("異步任務比較:")
        
        start = time.time()
        sequential_results = await self.run_tasks_sequential()
        sequential_time = time.time() - start
        
        start = time.time()
        concurrent_results = await self.run_tasks_concurrent()
        concurrent_time = time.time() - start
        
        print(f"順序執行: {sequential_time:.4f} 秒")
        print(f"並發執行: {concurrent_time:.4f} 秒")
        
        # 比較HTTP請求
        print("\nHTTP請求比較:")
        
        start = time.time()
        sequential_results = await self.run_http_requests_sequential()
        sequential_time = time.time() - start
        
        start = time.time()
        concurrent_results = await self.run_http_requests_concurrent()
        concurrent_time = time.time() - start
        
        print(f"順序執行: {sequential_time:.4f} 秒")
        print(f"並發執行: {concurrent_time:.4f} 秒")

# 使用示例
async def main():
    test = AsyncPerformanceTest()
    await test.compare_performance()

if __name__ == '__main__':
    asyncio.run(main())
```

## 練習題

1. **性能分析工具**
   實現一個性能分析工具：
   - 代碼執行時間統計
   - 內存使用監控
   - 函數調用追蹤
   - 性能報告生成

2. **並發處理框架**
   開發一個並發處理框架：
   - 任務調度
   - 資源管理
   - 錯誤處理
   - 性能監控

3. **緩存系統**
   實現一個高效的緩存系統：
   - 多級緩存
   - 過期策略
   - 並發控制
   - 性能優化

## 小提醒 💡

1. 代碼優化
   - 選擇合適的數據結構
   - 使用內置函數
   - 避免重複計算
   - 減少循環嵌套

2. 內存管理
   - 及時釋放資源
   - 避免循環引用
   - 使用生成器
   - 控制對象生命週期

3. 並發處理
   - 選擇合適的並發模型
   - 注意資源競爭
   - 處理異常情況
   - 優化任務粒度

4. 性能測試
   - 建立基準測試
   - 使用性能分析工具
   - 進行壓力測試
   - 持續優化改進

[上一章：雲端部署與服務](059_雲端部署與服務.md) 