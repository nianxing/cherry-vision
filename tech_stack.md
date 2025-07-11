# Cherry Vision - 技术栈详细说明

基于DesignDoc要求，以下是各模块推荐的技术栈和依赖：

## 1. 硬件控制层 (`src/hardware/`)

### 无人机控制 (DJI Agras T40)
- **SDK**: DJI Mobile SDK 4.16 / DJI Onboard SDK 4.1
- **语言**: Python 3.8+ 
- **依赖**: 
  ```
  dji-sdk==4.16
  pyserial==3.5
  opencv-python==4.8.0
  ```

### 相机控制 (FLIR Blackfly S)
- **SDK**: PySpin (FLIR Spinnaker SDK)
- **语言**: Python 3.8+
- **依赖**:
  ```
  PySpin==3.0.0.118
  numpy==1.24.3
  opencv-python==4.8.0
  ```

## 2. 边缘计算层 (`src/edge/`)

### Jetson Nano 环境
- **操作系统**: JetPack 4.6.1 (Ubuntu 18.04)
- **Python**: 3.6.9
- **深度学习框架**: 
  - PyTorch 1.10.0 (ARM64)
  - TensorRT 8.2.1
- **依赖**:
  ```
  torch==1.10.0
  torchvision==0.11.0
  ultralytics==8.0.196
  tensorrt==8.2.1.8
  opencv-python==4.5.4
  pillow==8.3.2
  ```

## 3. AI模型层 (`src/ai/`)

### 果实检测 (YOLOv8)
- **框架**: Ultralytics YOLOv8
- **后端**: PyTorch 2.0+
- **训练工具**:
  ```
  ultralytics==8.0.196
  torch>=2.0.0
  torchvision>=0.15.0
  matplotlib>=3.2.2
  tensorboard>=2.4.1
  ```

### 质量分级 (ResNet-50)
- **框架**: PyTorch 2.0+
- **预训练**: ImageNet weights
- **依赖**:
  ```
  torch>=2.0.0
  torchvision>=0.15.0
  timm==0.9.7
  albumentations==1.3.1
  scikit-learn==1.3.0
  ```

### 数据处理
- **标注工具**: LabelImg, CVAT
- **数据增强**: Albumentations
- **数据管理**:
  ```
  pandas==2.0.3
  numpy==1.24.3
  albumentations==1.3.1
  labelimg==1.8.6
  ```

## 4. 云服务层 (`src/cloud/`)

### Azure 集成
- **计算**: Azure Machine Learning (Standard_NC6s_v3 for inference)
- **存储**: Azure Blob Storage (图像存储)
- **数据库**: Azure Database for PostgreSQL 14
- **消息队列**: Azure Service Bus
- **依赖**:
  ```
  azure-ai-ml==1.10.0
  azure-storage-blob==12.17.0
  azure-identity==1.14.0
  psycopg2-binary==2.9.7
  redis==4.6.0
  celery==5.3.1
  azure-servicebus==7.11.0
  ```

### API 服务
- **框架**: FastAPI 0.103+
- **异步**: asyncio, uvicorn
- **认证**: JWT
- **依赖**:
  ```
  fastapi==0.103.1
  uvicorn[standard]==0.23.2
  pydantic==2.3.0
  python-jose[cryptography]==3.3.0
  python-multipart==0.0.6
  sqlalchemy==2.0.20
  alembic==1.11.3
  ```

## 5. 前端层 (`frontend/`)

### React 应用
- **框架**: React 18.2.0
- **状态管理**: Redux Toolkit 1.9.5
- **路由**: React Router 6.15.0
- **UI组件**: Ant Design 5.9.0
- **可视化**: D3.js 7.8.5, Chart.js 4.4.0
- **依赖**:
  ```json
  {
    "react": "^18.2.0",
    "react-dom": "^18.2.0", 
    "@reduxjs/toolkit": "^1.9.5",
    "react-router-dom": "^6.15.0",
    "antd": "^5.9.0",
    "d3": "^7.8.5",
    "chart.js": "^4.4.0",
    "axios": "^1.5.0",
    "leaflet": "^1.9.4",
    "react-leaflet": "^4.2.1"
  }
  ```

### 构建工具
- **打包**: Webpack 5.88.0
- **编译**: Babel 7.22.0
- **类型检查**: TypeScript 5.2.0
- **代码质量**: ESLint, Prettier

## 6. 数据存储层

### 数据库设计
- **关系型**: PostgreSQL 14 (Azure Database for PostgreSQL)
- **缓存**: Redis 7.0 (Azure Cache for Redis)
- **对象存储**: Azure Blob Storage
- **时序数据**: InfluxDB 2.7 (监控指标)

### 数据模式
```sql
-- 果园信息表
CREATE TABLE orchards (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    location GEOMETRY(POINT, 4326),
    area_mu DECIMAL(10,2),
    cherry_variety VARCHAR(100),
    created_at TIMESTAMP DEFAULT NOW()
);

-- 图像表
CREATE TABLE images (
    id SERIAL PRIMARY KEY,
    orchard_id INTEGER REFERENCES orchards(id),
    file_path VARCHAR(500),
    capture_time TIMESTAMP,
    image_type VARCHAR(20), -- 'rgb' or 'nir'
    device_id VARCHAR(100),
    processed BOOLEAN DEFAULT FALSE
);

-- 检测结果表
CREATE TABLE detections (
    id SERIAL PRIMARY KEY,
    image_id INTEGER REFERENCES images(id),
    bbox_x INTEGER,
    bbox_y INTEGER, 
    bbox_width INTEGER,
    bbox_height INTEGER,
    confidence DECIMAL(4,3),
    quality_grade VARCHAR(10), -- J/JJ/JJJ
    created_at TIMESTAMP DEFAULT NOW()
);
```

## 7. 容器化和编排

### Docker 配置
- **基础镜像**: 
  - Edge: `nvcr.io/nvidia/l4t-pytorch:r32.7.1-pth1.10-py3`
  - API: `python:3.11-slim`
  - Frontend: `node:18-alpine`

### Kubernetes 部署
- **集群**: Azure Kubernetes Service (AKS) 1.27
- **监控**: Prometheus + Grafana
- **日志**: ELK Stack (Elasticsearch, Logstash, Kibana)
- **配置**:
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: cherry-vision-api
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: cherry-vision-api
    template:
      spec:
        containers:
        - name: api
          image: cherryregistry.azurecr.io/cherry-vision/api:latest
          resources:
            requests:
              memory: "512Mi"
              cpu: "500m"
            limits:
              memory: "1Gi" 
              cpu: "1000m"
  ```

## 8. 监控和运维

### 监控栈
- **指标收集**: Prometheus 2.45
- **可视化**: Grafana 10.1
- **告警**: AlertManager 0.26
- **日志**: ELK Stack 8.9
- **追踪**: Jaeger 1.47

### 关键指标
- 模型推理延迟 (<5秒目标)
- 检测准确率 (>95%目标)
- 分级准确率 (>90%目标)
- 系统可用性 (>99.5%目标)
- 硬件状态 (电池、存储、网络)

## 9. 开发工具链

### 代码质量
```bash
# Python 代码质量
black==23.7.0
flake8==6.0.0
mypy==1.5.1
pytest==7.4.2
pytest-cov==4.1.0

# JavaScript 代码质量  
eslint==8.47.0
prettier==3.0.2
@typescript-eslint/parser==6.5.0
jest==29.6.4
```

### CI/CD 流水线 (GitHub Actions)
```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: pip install -r requirements.txt
      - name: Run tests
        run: pytest tests/ --cov=src/
      
  build-and-deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Build Docker images
        run: docker build -t cherry-vision/api .
      - name: Deploy to Azure AKS
        run: kubectl apply -f deployment/kubernetes/
```

## 10. 性能优化

### 模型优化
- **量化**: INT8 quantization (TensorRT)
- **剪枝**: 结构化剪枝 (20% 参数减少)
- **蒸馏**: 教师-学生模型
- **批处理**: 动态批处理 (batch size 1-8)

### 系统优化
- **缓存策略**: Redis 缓存推理结果
- **负载均衡**: Azure Application Gateway + 多实例
- **CDN**: Azure CDN 静态资源分发
- **数据库**: 读写分离 + 连接池

这个技术栈设计确保系统能够满足DesignDoc中的所有性能和准确率要求，同时提供良好的可扩展性和维护性。 