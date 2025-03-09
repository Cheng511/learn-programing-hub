[上一章：容器化應用開發](132_容器化應用開發.md) | [下一章：雲原生存儲方案](134_雲端原生存儲方案.md)

# Python Kubernetes應用部署 ⎈

## 1. Kubernetes 基礎

### 1.1 核心概念

Kubernetes (K8s) 是一個開源的容器編排平台，提供：
- 容器部署
- 擴展
- 負載均衡
- 服務發現
- 自動恢復

### 1.2 基本組件

```python
from dataclasses import dataclass
from typing import List, Dict, Optional

@dataclass
class KubernetesResource:
    """Kubernetes 資源基類"""
    api_version: str
    kind: str
    metadata: Dict[str, str]
    spec: Dict

class KubernetesDeployment(KubernetesResource):
    """Kubernetes Deployment 配置"""
    def __init__(self, name: str, image: str, replicas: int = 1):
        super().__init__(
            api_version="apps/v1",
            kind="Deployment",
            metadata={"name": name},
            spec={
                "replicas": replicas,
                "selector": {
                    "matchLabels": {"app": name}
                },
                "template": {
                    "metadata": {
                        "labels": {"app": name}
                    },
                    "spec": {
                        "containers": [{
                            "name": name,
                            "image": image
                        }]
                    }
                }
            }
        )
```

## 2. Python 應用部署

### 2.1 部署配置

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: python-app
  template:
    metadata:
      labels:
        app: python-app
    spec:
      containers:
      - name: python-app
        image: python-app:latest
        ports:
        - containerPort: 8000
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 10
```

### 2.2 部署管理工具

```python
from kubernetes import client, config
from typing import Dict, List, Optional

class KubernetesManager:
    """Kubernetes 部署管理"""
    def __init__(self, context: Optional[str] = None):
        # 加載 kubeconfig
        if context:
            config.load_kube_config(context=context)
        else:
            config.load_kube_config()
        
        self.apps_v1 = client.AppsV1Api()
        self.core_v1 = client.CoreV1Api()
    
    def create_deployment(self,
                         name: str,
                         image: str,
                         namespace: str = "default",
                         replicas: int = 1,
                         ports: List[int] = None,
                         env_vars: Dict[str, str] = None) -> bool:
        """創建部署"""
        try:
            # 創建容器端口配置
            container_ports = []
            if ports:
                container_ports = [
                    client.V1ContainerPort(container_port=port)
                    for port in ports
                ]
            
            # 創建環境變量配置
            env = []
            if env_vars:
                env = [
                    client.V1EnvVar(name=k, value=v)
                    for k, v in env_vars.items()
                ]
            
            # 創建部署對象
            deployment = client.V1Deployment(
                metadata=client.V1ObjectMeta(name=name),
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
                                    image=image,
                                    ports=container_ports,
                                    env=env
                                )
                            ]
                        )
                    )
                )
            )
            
            # 創建部署
            self.apps_v1.create_namespaced_deployment(
                namespace=namespace,
                body=deployment
            )
            return True
            
        except client.rest.ApiException as e:
            print(f"Deployment creation failed: {e}")
            return False
    
    def scale_deployment(self,
                        name: str,
                        replicas: int,
                        namespace: str = "default") -> bool:
        """擴展部署"""
        try:
            # 獲取當前部署
            deployment = self.apps_v1.read_namespaced_deployment(
                name=name,
                namespace=namespace
            )
            
            # 更新副本數
            deployment.spec.replicas = replicas
            
            # 應用更新
            self.apps_v1.patch_namespaced_deployment(
                name=name,
                namespace=namespace,
                body=deployment
            )
            return True
            
        except client.rest.ApiException as e:
            print(f"Scaling failed: {e}")
            return False
```

## 3. 服務配置

### 3.1 服務定義

```python
class KubernetesService:
    """Kubernetes 服務管理"""
    def __init__(self, core_v1_api: client.CoreV1Api):
        self.core_v1 = core_v1_api
    
    def create_service(self,
                      name: str,
                      namespace: str = "default",
                      port: int = 80,
                      target_port: int = 8000,
                      service_type: str = "ClusterIP") -> bool:
        """創建服務"""
        try:
            service = client.V1Service(
                metadata=client.V1ObjectMeta(name=name),
                spec=client.V1ServiceSpec(
                    selector={"app": name},
                    ports=[
                        client.V1ServicePort(
                            port=port,
                            target_port=target_port
                        )
                    ],
                    type=service_type
                )
            )
            
            self.core_v1.create_namespaced_service(
                namespace=namespace,
                body=service
            )
            return True
            
        except client.rest.ApiException as e:
            print(f"Service creation failed: {e}")
            return False
```

## 4. 配置與密鑰管理

### 4.1 ConfigMap 管理

```python
class ConfigMapManager:
    """ConfigMap 管理工具"""
    def __init__(self, core_v1_api: client.CoreV1Api):
        self.core_v1 = core_v1_api
    
    def create_config_map(self,
                         name: str,
                         namespace: str = "default",
                         data: Dict[str, str] = None) -> bool:
        """創建 ConfigMap"""
        try:
            config_map = client.V1ConfigMap(
                metadata=client.V1ObjectMeta(name=name),
                data=data
            )
            
            self.core_v1.create_namespaced_config_map(
                namespace=namespace,
                body=config_map
            )
            return True
            
        except client.rest.ApiException as e:
            print(f"ConfigMap creation failed: {e}")
            return False
    
    def update_config_map(self,
                         name: str,
                         namespace: str = "default",
                         data: Dict[str, str] = None) -> bool:
        """更新 ConfigMap"""
        try:
            config_map = self.core_v1.read_namespaced_config_map(
                name=name,
                namespace=namespace
            )
            
            config_map.data = data
            
            self.core_v1.replace_namespaced_config_map(
                name=name,
                namespace=namespace,
                body=config_map
            )
            return True
            
        except client.rest.ApiException as e:
            print(f"ConfigMap update failed: {e}")
            return False
```

## 5. 監控與日誌

### 5.1 Pod 監控

```python
from datetime import datetime
import json

class PodMonitor:
    """Pod 監控工具"""
    def __init__(self, core_v1_api: client.CoreV1Api):
        self.core_v1 = core_v1_api
    
    def get_pod_status(self,
                      namespace: str = "default",
                      label_selector: str = None) -> List[Dict]:
        """獲取 Pod 狀態"""
        try:
            pods = self.core_v1.list_namespaced_pod(
                namespace=namespace,
                label_selector=label_selector
            )
            
            return [{
                "name": pod.metadata.name,
                "status": pod.status.phase,
                "start_time": pod.status.start_time,
                "ip": pod.status.pod_ip,
                "node": pod.spec.node_name,
                "containers": [
                    {
                        "name": cont.name,
                        "ready": cont.ready,
                        "restart_count": cont.restart_count
                    }
                    for cont in pod.status.container_statuses
                ]
            } for pod in pods.items]
            
        except client.rest.ApiException as e:
            print(f"Pod status check failed: {e}")
            return []
    
    def get_pod_logs(self,
                     pod_name: str,
                     namespace: str = "default",
                     container: str = None,
                     tail_lines: int = None) -> str:
        """獲取 Pod 日誌"""
        try:
            return self.core_v1.read_namespaced_pod_log(
                name=pod_name,
                namespace=namespace,
                container=container,
                tail_lines=tail_lines
            )
        except client.rest.ApiException as e:
            print(f"Log retrieval failed: {e}")
            return ""
```

## 練習題 🏃‍♂️

1. 部署一個完整的 Python Web 應用到 Kubernetes：
   - 創建部署配置
   - 設置服務暴露
   - 配置健康檢查
   - 實現自動擴展

2. 實現一個多服務應用部署：
   - 前端服務
   - 後端 API 服務
   - 數據庫服務
   - 消息隊列服務

3. 開發 Kubernetes 管理工具：
   - 部署管理
   - 服務管理
   - 配置管理
   - 日誌收集

4. 實現自動擴展系統：
   - CPU 使用率監控
   - 內存使用率監控
   - 自動擴展策略
   - 擴展事件通知

5. 構建完整的 CI/CD 流程：
   - 代碼構建
   - 容器打包
   - Kubernetes 部署
   - 部署驗證

## 小結 📝

- 了解了 Kubernetes 的基本概念
- 掌握了部署配置的編寫
- 學會了服務配置和管理
- 理解了配置和密鑰管理
- 掌握了監控和日誌收集

## 延伸閱讀 📚

1. Kubernetes 官方文檔
2. Python Kubernetes Client 文檔
3. Kubernetes Patterns
4. Cloud Native DevOps with Kubernetes
5. Kubernetes 最佳實踐指南

[上一章：容器化應用開發](132_容器化應用開發.md) | [下一章：雲原生存儲方案](134_雲原生存儲方案.md) 