[上一章：雲原生安全實踐](135_雲原生安全實踐.md) | [下一章：持續集成與部署](137_持續集成與部署.md)

# Python DevOps實踐基礎 🔄

## 1. 自動化基礎設施

### 1.1 基礎設施即代碼

```python
from typing import Dict, List, Optional
import boto3
import yaml

class InfrastructureManager:
    """基礎設施管理器"""
    def __init__(self, region: str):
        self.ec2 = boto3.client('ec2', region_name=region)
        self.s3 = boto3.client('s3', region_name=region)
    
    def create_infrastructure(self,
                            config_file: str) -> bool:
        """創建基礎設施"""
        try:
            with open(config_file, 'r') as f:
                config = yaml.safe_load(f)
            
            # 創建 VPC
            vpc_id = self._create_vpc(config['vpc'])
            
            # 創建子網
            subnet_ids = self._create_subnets(
                vpc_id,
                config['subnets']
            )
            
            # 創建安全組
            security_group_id = self._create_security_group(
                vpc_id,
                config['security_group']
            )
            
            # 創建 EC2 實例
            instance_ids = self._create_instances(
                subnet_ids,
                security_group_id,
                config['instances']
            )
            
            return True
        except Exception as e:
            print(f"Failed to create infrastructure: {e}")
            return False
    
    def _create_vpc(self, vpc_config: Dict) -> str:
        """創建 VPC"""
        response = self.ec2.create_vpc(
            CidrBlock=vpc_config['cidr_block']
        )
        return response['Vpc']['VpcId']
    
    def _create_subnets(self,
                       vpc_id: str,
                       subnet_configs: List[Dict]) -> List[str]:
        """創建子網"""
        subnet_ids = []
        for subnet in subnet_configs:
            response = self.ec2.create_subnet(
                VpcId=vpc_id,
                CidrBlock=subnet['cidr_block'],
                AvailabilityZone=subnet['availability_zone']
            )
            subnet_ids.append(response['Subnet']['SubnetId'])
        return subnet_ids
```

### 1.2 配置管理

```python
import ansible_runner
from typing import Dict, List

class ConfigurationManager:
    """配置管理器"""
    def __init__(self,
                 inventory_file: str,
                 private_key_file: str):
        self.inventory_file = inventory_file
        self.private_key_file = private_key_file
    
    def apply_configuration(self,
                          playbook_file: str,
                          extra_vars: Optional[Dict] = None) -> bool:
        """應用配置"""
        try:
            result = ansible_runner.run(
                private_data_dir='/tmp/ansible',
                playbook=playbook_file,
                inventory=self.inventory_file,
                extravars=extra_vars,
                cmdline=f'--private-key={self.private_key_file}'
            )
            
            return result.status == 'successful'
        except Exception as e:
            print(f"Failed to apply configuration: {e}")
            return False
    
    def get_host_facts(self, host: str) -> Dict:
        """獲取主機信息"""
        try:
            result = ansible_runner.run(
                private_data_dir='/tmp/ansible',
                module='setup',
                host_pattern=host,
                inventory=self.inventory_file,
                cmdline=f'--private-key={self.private_key_file}'
            )
            
            return result.events[-1]['event_data']['res']
        except Exception as e:
            print(f"Failed to get host facts: {e}")
            return {}
```

## 2. 部署自動化

### 2.1 部署策略

```python
from kubernetes import client, config
from typing import Dict, List

class DeploymentManager:
    """部署管理器"""
    def __init__(self):
        config.load_kube_config()
        self.apps_v1 = client.AppsV1Api()
    
    def rolling_update(self,
                      name: str,
                      namespace: str,
                      image: str,
                      replicas: int = 3) -> bool:
        """滾動更新"""
        try:
            deployment = client.V1Deployment(
                metadata=client.V1ObjectMeta(
                    name=name,
                    namespace=namespace
                ),
                spec=client.V1DeploymentSpec(
                    replicas=replicas,
                    selector=client.V1LabelSelector(
                        match_labels={"app": name}
                    ),
                    template=client.V1PodTemplateSpec(
                        metadata=client.V1ObjectMeta(
                            labels={"app": name}
                        ),
                        spec=client.V1PodSpec(
                            containers=[
                                client.V1Container(
                                    name=name,
                                    image=image
                                )
                            ]
                        )
                    ),
                    strategy=client.V1DeploymentStrategy(
                        type="RollingUpdate",
                        rolling_update=client.V1RollingUpdateDeployment(
                            max_surge="25%",
                            max_unavailable="25%"
                        )
                    )
                )
            )
            
            self.apps_v1.patch_namespaced_deployment(
                name=name,
                namespace=namespace,
                body=deployment
            )
            return True
        except Exception as e:
            print(f"Failed to update deployment: {e}")
            return False
```

### 2.2 金絲雀發布

```python
class CanaryDeployment:
    """金絲雀發布管理器"""
    def __init__(self):
        config.load_kube_config()
        self.apps_v1 = client.AppsV1Api()
        self.core_v1 = client.CoreV1Api()
    
    def deploy_canary(self,
                     name: str,
                     namespace: str,
                     image: str,
                     canary_replicas: int = 1) -> bool:
        """部署金絲雀版本"""
        try:
            # 創建金絲雀部署
            canary_name = f"{name}-canary"
            deployment = client.V1Deployment(
                metadata=client.V1ObjectMeta(
                    name=canary_name,
                    namespace=namespace
                ),
                spec=client.V1DeploymentSpec(
                    replicas=canary_replicas,
                    selector=client.V1LabelSelector(
                        match_labels={
                            "app": name,
                            "version": "canary"
                        }
                    ),
                    template=client.V1PodTemplateSpec(
                        metadata=client.V1ObjectMeta(
                            labels={
                                "app": name,
                                "version": "canary"
                            }
                        ),
                        spec=client.V1PodSpec(
                            containers=[
                                client.V1Container(
                                    name=name,
                                    image=image
                                )
                            ]
                        )
                    )
                )
            )
            
            self.apps_v1.create_namespaced_deployment(
                namespace=namespace,
                body=deployment
            )
            return True
        except Exception as e:
            print(f"Failed to deploy canary: {e}")
            return False
    
    def promote_canary(self,
                      name: str,
                      namespace: str) -> bool:
        """提升金絲雀版本"""
        try:
            # 獲取金絲雀部署的配置
            canary_name = f"{name}-canary"
            canary = self.apps_v1.read_namespaced_deployment(
                name=canary_name,
                namespace=namespace
            )
            
            # 更新主部署
            self.apps_v1.patch_namespaced_deployment(
                name=name,
                namespace=namespace,
                body={
                    "spec": {
                        "template": canary.spec.template
                    }
                }
            )
            
            # 刪除金絲雀部署
            self.apps_v1.delete_namespaced_deployment(
                name=canary_name,
                namespace=namespace
            )
            
            return True
        except Exception as e:
            print(f"Failed to promote canary: {e}")
            return False
```

## 3. 監控與日誌

### 3.1 Prometheus 集成

```python
from prometheus_client import start_http_server, Counter, Gauge, Histogram
import time

class MetricsCollector:
    """指標收集器"""
    def __init__(self, port: int = 8000):
        self.port = port
        
        # 定義指標
        self.request_count = Counter(
            'app_request_count',
            'Application Request Count',
            ['method', 'endpoint']
        )
        
        self.response_time = Histogram(
            'app_response_time_seconds',
            'Application Response Time',
            ['method', 'endpoint']
        )
        
        self.memory_usage = Gauge(
            'app_memory_usage_bytes',
            'Application Memory Usage'
        )
    
    def start(self):
        """啟動指標服務器"""
        start_http_server(self.port)
    
    def record_request(self,
                      method: str,
                      endpoint: str):
        """記錄請求"""
        self.request_count.labels(
            method=method,
            endpoint=endpoint
        ).inc()
    
    def record_response_time(self,
                           method: str,
                           endpoint: str,
                           duration: float):
        """記錄響應時間"""
        self.response_time.labels(
            method=method,
            endpoint=endpoint
        ).observe(duration)
    
    def update_memory_usage(self,
                           usage: float):
        """更新內存使用"""
        self.memory_usage.set(usage)
```

### 3.2 ELK 集成

```python
from elasticsearch import Elasticsearch
from typing import Dict, List
import logging

class LogManager:
    """日誌管理器"""
    def __init__(self,
                 es_host: str,
                 es_port: int,
                 index_prefix: str):
        self.es = Elasticsearch([{
            'host': es_host,
            'port': es_port
        }])
        self.index_prefix = index_prefix
        
        # 配置日誌
        self.logger = logging.getLogger('app')
        self.logger.setLevel(logging.INFO)
    
    def index_log(self,
                  log_data: Dict) -> bool:
        """索引日誌"""
        try:
            self.es.index(
                index=f"{self.index_prefix}-{time.strftime('%Y.%m.%d')}",
                body=log_data
            )
            return True
        except Exception as e:
            print(f"Failed to index log: {e}")
            return False
    
    def search_logs(self,
                    query: Dict,
                    start_time: str,
                    end_time: str) -> List[Dict]:
        """搜索日誌"""
        try:
            response = self.es.search(
                index=f"{self.index_prefix}-*",
                body={
                    "query": {
                        "bool": {
                            "must": [
                                query,
                                {
                                    "range": {
                                        "@timestamp": {
                                            "gte": start_time,
                                            "lte": end_time
                                        }
                                    }
                                }
                            ]
                        }
                    }
                }
            )
            
            return [hit["_source"] for hit in response["hits"]["hits"]]
        except Exception as e:
            print(f"Failed to search logs: {e}")
            return []
```

## 練習題 🏃‍♂️

1. 實現完整的 CI/CD 流水線：
   - 代碼檢查
   - 單元測試
   - 構建打包
   - 自動部署
   - 環境管理

2. 開發基礎設施管理工具：
   - 資源創建
   - 配置管理
   - 狀態監控
   - 自動擴縮容
   - 災難恢復

3. 實現監控告警系統：
   - 指標收集
   - 日誌聚合
   - 告警規則
   - 通知集成
   - 故障分析

4. 創建部署管理系統：
   - 版本控制
   - 環境管理
   - 發布策略
   - 回滾機制
   - 審計跟踪

5. 開發自動化測試框架：
   - 單元測試
   - 集成測試
   - 性能測試
   - 安全測試
   - 測試報告

## 小結 📝

- 了解了 DevOps 的基本概念和實踐
- 掌握了基礎設施即代碼的實現方法
- 學會了不同的部署策略和自動化方案
- 理解了監控和日誌系統的重要性
- 掌握了 DevOps 工具鏈的使用方法

## 延伸閱讀 📚

1. DevOps 實踐指南
2. Infrastructure as Code 模式與實踐
3. Kubernetes 部署策略
4. 監控與可觀測性設計
5. 持續交付最佳實踐

[上一章：雲原生安全實踐](135_雲原生安全實踐.md) | [下一章：持續集成與部署](137_持續集成與部署.md) 