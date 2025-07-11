# Cherry Vision - 实施计划

基于DesignDoc的三阶段计划，以下是详细的开发实施路线图：

## 阶段一：试点准备 (0-3个月)

### 1.1 环境搭建 (第1-2周)

#### 代码仓库初始化
```bash
# 创建项目骨架
mkdir cherry-vision && cd cherry-vision
git init
pip install cookiecutter
cookiecutter https://github.com/audreyfeldroy/cookiecutter-pypackage

# 按照project_structure.md创建目录结构
mkdir -p src/{hardware,edge,ai,cloud,data,utils}
mkdir -p frontend/src/{components,services,utils}
mkdir -p scripts/{setup,data,training,deployment}
mkdir -p tests/{unit,integration,e2e}
mkdir -p deployment/{docker,kubernetes,azure,ansible}
```

#### 开发环境配置
```bash
# Python环境 (后端开发)
pyenv install 3.11.5
pyenv virtualenv 3.11.5 cherry-vision
pyenv activate cherry-vision
pip install -r requirements.txt

# Node.js环境 (前端开发)
nvm install 18.17.0
nvm use 18.17.0
cd frontend && npm install

# Jetson Nano环境配置
# 在边缘设备上安装JetPack 4.6.1
sudo apt update && sudo apt install -y python3-pip
pip3 install torch torchvision ultralytics
```

#### 硬件采购和配置
- [x] 采购1台DJI Agras T40无人机 ($5,936)
- [x] 采购2台FLIR Blackfly S相机 ($29,680)
- [x] 采购24台Jetson Nano开发板
- [x] 配置Azure账户和服务

### 1.2 数据收集模块 (第3-4周)

#### 硬件控制开发
```python
# src/hardware/drone_controller.py
class DroneController:
    def __init__(self, config_path: str):
        self.sdk = DJISDKManager()
        self.config = load_config(config_path)
    
    async def capture_images(self, flight_plan: FlightPlan) -> List[str]:
        """执行飞行计划并采集图像"""
        pass
    
    def get_battery_status(self) -> BatteryStatus:
        """获取电池状态"""
        pass

# src/hardware/camera_controller.py  
class CameraController:
    def __init__(self, camera_id: str):
        self.system = PySpin.System.GetInstance()
        self.camera = self.system.GetCameras()[camera_id]
        
    def capture_rgb_nir(self) -> Tuple[np.ndarray, np.ndarray]:
        """同时采集RGB和NIR图像"""
        pass
```

#### 数据收集脚本
```python
# scripts/data/collect_images.py
async def main():
    # 目标：收集10,000张樱桃图像
    drone = DroneController("config/drone_config.yaml")
    cameras = [CameraController(i) for i in range(20)]
    
    collector = ImageCollector(drone, cameras)
    await collector.collect_dataset(
        target_images=10000,
        varieties=["Bing", "Lapins"],
        conditions=["sunny", "cloudy", "dawn", "dusk"]
    )
```

**第4周交付物：**
- ✅ 完成硬件控制模块 (`src/hardware/`)
- ✅ 收集1,000张测试图像
- ✅ 基础数据管道 (`src/data/collection/`)

### 1.3 AI模型开发 (第5-8周)

#### YOLOv8检测模型训练
```python
# src/ai/training/train_detector.py
from ultralytics import YOLO

def train_cherry_detector():
    # 使用COCO预训练权重
    model = YOLO('yolov8n.pt')
    
    # 训练配置
    results = model.train(
        data='data/cherry_dataset.yaml',
        epochs=100,
        batch=16,
        imgsz=640,
        device='cuda',
        project='runs/detect',
        name='cherry_yolov8'
    )
    
    # 验证结果 (目标: >90% mAP@0.5)
    metrics = model.val()
    return model, metrics

# data/cherry_dataset.yaml
train: data/processed/train/images
val: data/processed/val/images
test: data/processed/test/images

nc: 1  # 类别数量
names: ['cherry']  # 类别名称
```

#### ResNet-50分级模型训练  
```python
# src/ai/training/train_classifier.py
import torch.nn as nn
from torchvision import models

class CherryGradeClassifier(nn.Module):
    def __init__(self, num_classes=3):  # J/JJ/JJJ
        super().__init__()
        self.backbone = models.resnet50(pretrained=True)
        self.backbone.fc = nn.Linear(2048, num_classes)
        
    def forward(self, x):
        return self.backbone(x)

def train_grade_classifier():
    model = CherryGradeClassifier()
    
    # 数据加载器
    train_loader = create_dataloader(
        'data/processed/train/',
        batch_size=32,
        transform=get_train_transforms()
    )
    
    # 训练循环
    optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
    criterion = nn.CrossEntropyLoss()
    
    for epoch in range(50):
        train_epoch(model, train_loader, optimizer, criterion)
        val_acc = validate(model, val_loader)
        
        # 目标: >85% 验证准确率 (试点阶段)
        if val_acc > 0.85:
            save_model(model, f'models/resnet50_grade_epoch{epoch}.pt')
```

**第8周交付物：**
- ✅ YOLOv8检测模型 (>90% mAP@0.5)
- ✅ ResNet-50分级模型 (>85% 准确率)
- ✅ 模型训练/评估管道

### 1.4 边缘计算开发 (第9-10周)

#### Jetson Nano优化
```python
# src/edge/edge_inference.py
import tensorrt as trt
from ultralytics import YOLO

class EdgeInference:
    def __init__(self):
        # TensorRT优化的YOLOv8模型
        self.detector = YOLO('models/yolov8_cherry_trt.engine')
        self.classifier = self.load_trt_classifier()
        
    def process_image(self, image: np.ndarray) -> DetectionResult:
        """边缘推理：<5秒处理目标"""
        start_time = time.time()
        
        # 1. 预处理
        preprocessed = self.preprocess(image)
        
        # 2. YOLOv8检测
        detections = self.detector(preprocessed)
        
        # 3. 提取樱桃区域
        cherry_crops = self.extract_cherries(image, detections)
        
        # 4. 质量分级
        grades = []
        for crop in cherry_crops:
            grade = self.classifier(crop)
            grades.append(grade)
        
        processing_time = time.time() - start_time
        
        return DetectionResult(
            detections=detections,
            grades=grades,
            processing_time=processing_time,
            meets_sla=processing_time < 5.0
        )

# src/edge/data_compressor.py
class DataCompressor:
    def compress_for_upload(self, image: np.ndarray, 
                          detections: List) -> bytes:
        """压缩数据用于上传 (目标: 90%减少)"""
        # 只保留检测区域的高分辨率图像
        # 其他区域降采样
        compressed = self.selective_compression(image, detections)
        return compressed
```

**第10周交付物：**
- ✅ 边缘推理模块 (<5秒处理)
- ✅ TensorRT模型优化
- ✅ 数据压缩上传模块

### 1.5 云平台和API开发 (第11-12周)

#### FastAPI后端服务
```python
# src/cloud/api/routes.py
from fastapi import FastAPI, UploadFile, BackgroundTasks
from src.cloud.services.analysis_service import AnalysisService

app = FastAPI(title="Cherry Vision API", version="1.0.0")

@app.post("/api/v1/images/upload")
async def upload_image(
    file: UploadFile,
    orchard_id: int,
    background_tasks: BackgroundTasks
):
    """上传图像并触发分析"""
    # 保存到Azure Blob Storage
    blob_path = await upload_to_blob_storage(file)
    
    # 异步处理
    background_tasks.add_task(
        analyze_image,
        blob_path,
        orchard_id
    )
    
    return {"status": "uploaded", "path": blob_path}

@app.get("/api/v1/orchards/{orchard_id}/analysis")
async def get_analysis(orchard_id: int):
    """获取果园分析结果"""
    service = AnalysisService()
    results = await service.get_orchard_analysis(orchard_id)
    
    return {
        "total_cherries": results.total_count,
        "quality_distribution": {
            "J": results.grade_j_count,
            "JJ": results.grade_jj_count, 
            "JJJ": results.grade_jjj_count
        },
        "yield_prediction": results.yield_tons,
        "confidence": results.confidence
    }
```

#### Azure集成
```python
# src/cloud/azure/ml_service.py
from azure.ai.ml import MLClient
from azure.ai.ml.entities import Model, ManagedOnlineEndpoint, ManagedOnlineDeployment

class AzureMLService:
    def __init__(self):
        self.ml_client = MLClient.from_config()
        
    def deploy_models(self):
        """部署YOLOv8和ResNet-50到Azure ML"""
        # YOLOv8检测端点
        detector_endpoint = ManagedOnlineEndpoint(
            name="cherry-detector-endpoint",
            description="Cherry detection using YOLOv8"
        )
        self.ml_client.online_endpoints.begin_create_or_update(detector_endpoint)
        
        detector_deployment = ManagedOnlineDeployment(
            name="cherry-detector-deployment",
            endpoint_name="cherry-detector-endpoint",
            model="cherry-yolov8:1",
            instance_type="Standard_NC6s_v3",
            instance_count=1
        )
        
        # ResNet-50分级端点
        classifier_endpoint = ManagedOnlineEndpoint(
            name="cherry-classifier-endpoint",
            description="Cherry grading using ResNet-50"
        )
        self.ml_client.online_endpoints.begin_create_or_update(classifier_endpoint)
        
        classifier_deployment = ManagedOnlineDeployment(
            name="cherry-classifier-deployment", 
            endpoint_name="cherry-classifier-endpoint",
            model="cherry-resnet50:1",
            instance_type="Standard_NC6s_v3",
            instance_count=1
        )
        
        return detector_deployment, classifier_deployment
```

**第12周交付物：**
- ✅ FastAPI后端服务
- ✅ Azure服务集成 (Blob Storage, ML, PostgreSQL)
- ✅ API文档和测试

## 阶段二：试点测试与优化 (3-6个月)

### 2.1 试点部署 (第13-16周)

#### 1,000亩试点果园部署
```bash
# 部署脚本
scripts/deployment/deploy_pilot.py
```

#### 数据收集扩充
- 目标：额外收集40,000张图像
- 覆盖更多场景：
  - 不同成熟度 (30%, 50%, 70%, 90%)
  - 不同遮挡程度 (10%, 30%, 50%)
  - 不同光照条件 (早晨, 中午, 黄昏)
  - 不同天气 (晴天, 阴天, 小雨后)

### 2.2 模型优化 (第17-20周)

#### 准确率提升
```python
# 目标指标提升：
# YOLOv8: 90% → >95% mAP@0.5
# ResNet-50: 85% → >90% 准确率

# 优化策略：
# 1. 数据增强
# 2. 模型集成
# 3. 难例挖掘
# 4. 半监督学习
```

#### 性能优化
```python
# 目标：
# 边缘推理: 5秒 → <5秒
# 云端推理: 2分钟 → <1分钟
# 报告生成: 5分钟 → <1分钟

# 优化方法：
# 1. 模型剪枝和量化
# 2. 批处理优化
# 3. 缓存策略
# 4. 并行处理
```

### 2.3 前端开发 (第21-24周)

#### React仪表板
```jsx
// frontend/src/components/Dashboard/Dashboard.jsx
import React, { useState, useEffect } from 'react';
import { YieldHeatmap } from './YieldHeatmap';
import { QualityPieChart } from './QualityPieChart';
import { FilterPanel } from './FilterPanel';

const Dashboard = () => {
  const [data, setData] = useState(null);
  const [filters, setFilters] = useState({
    orchard: 'all',
    dateRange: 'last30days',
    variety: 'all'
  });

  useEffect(() => {
    fetchAnalysisData(filters).then(setData);
  }, [filters]);

  return (
    <div className="dashboard">
      <FilterPanel 
        filters={filters} 
        onFilterChange={setFilters} 
      />
      
      <div className="dashboard-content">
        <YieldHeatmap data={data?.yieldMap} />
        <QualityPieChart data={data?.qualityDist} />
        
        <div className="metrics-cards">
          <MetricCard 
            title="总产量预测"
            value={`${data?.totalYield || 0} 吨`}
            trend={data?.yieldTrend}
          />
          <MetricCard 
            title="优质果率 (J级)"
            value={`${data?.premiumRate || 0}%`}
            trend={data?.qualityTrend}
          />
          <MetricCard
            title="平均准确率"
            value={`${data?.accuracy || 0}%`}
            trend={data?.accuracyTrend}
          />
        </div>
      </div>
    </div>
  );
};
```

**第24周交付物：**
- ✅ 优化的AI模型 (>95% + >90% 准确率)
- ✅ 完整的React仪表板
- ✅ 试点测试报告

## 阶段三：全面部署 (6-9个月)

### 3.1 规模化部署 (第25-32周)

#### 硬件扩展
- 额外采购3台无人机
- 额外采购18台地面相机
- 部署到全部10,000亩果园

#### 系统集成
```python
# 多果园数据聚合
class MultiOrchardManager:
    def __init__(self):
        self.orchards = self.load_orchards()
        
    async def generate_consolidated_report(self) -> Report:
        """生成整体10,000亩报告"""
        tasks = []
        for orchard in self.orchards:
            task = self.analyze_orchard(orchard)
            tasks.append(task)
            
        results = await asyncio.gather(*tasks)
        
        return ConsolidatedReport(
            total_area=10000,
            total_yield=sum(r.yield for r in results),
            avg_quality=np.mean([r.quality_score for r in results]),
            orchards=results
        )
```

### 3.2 高级功能开发 (第33-36周)

#### 预测分析
```python
# src/ai/models/yield_predictor.py
class YieldPredictor:
    def __init__(self):
        self.model = self.load_lstm_model()
        
    def predict_harvest_schedule(self, 
                               current_data: OrchardData,
                               weather_forecast: WeatherData) -> HarvestPlan:
        """基于当前数据和天气预报预测最佳收获时间"""
        pass
        
    def optimize_pricing_strategy(self, 
                                yield_prediction: YieldData,
                                market_data: MarketData) -> PricingStrategy:
        """基于产量预测优化定价策略"""
        pass
```

#### 移动端应用
```typescript
// 简化的移动端React Native应用
// 供农场工人现场使用
const FieldApp = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Dashboard" component={FieldDashboard} />
        <Stack.Screen name="Capture" component={QuickCapture} />
        <Stack.Screen name="Results" component={InstantResults} />
      </Stack.Navigator>
    </NavigationContainer>
  );
};
```

### 3.3 运维和监控 (第37-39周)

#### 监控系统完善
```yaml
# monitoring/prometheus/alerts.yml
groups:
- name: cherry-vision-alerts
  rules:
  - alert: ModelAccuracyDrop
    expr: model_accuracy < 0.90
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "模型准确率下降"
      
  - alert: ProcessingLatencyHigh
    expr: processing_latency_seconds > 5
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "处理延迟超过5秒阈值"
      
  - alert: DroneOffline
    expr: drone_status == 0
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: "无人机离线"
```

#### 自动化运维
```python
# scripts/deployment/auto_scaling.py
class AutoScaler:
    def scale_inference_endpoints(self, load_metrics: Dict):
        """根据负载自动扩缩Azure ML端点"""
        if load_metrics['cpu_utilization'] > 80:
            self.scale_up()
        elif load_metrics['cpu_utilization'] < 30:
            self.scale_down()
```

**最终交付物 (第39周)：**
- ✅ 覆盖10,000亩的完整系统
- ✅ >95% 检测准确率，>90% 分级准确率
- ✅ <5秒边缘处理，<1分钟报告生成
- ✅ 完整的Web和移动端界面
- ✅ 自动化运维和监控系统
- ✅ 用户培训和文档
- ✅ 6个月技术支持计划

## 关键里程碑和风险缓解

### 里程碑检查点
- **第4周**: 硬件控制和数据收集验证
- **第8周**: AI模型基线达标 (>90% + >85%)
- **第12周**: 端到端流水线运行
- **第24周**: 试点性能达标 (>95% + >90%)
- **第36周**: 全量部署完成
- **第39周**: 系统验收

### 风险缓解策略
1. **技术风险**: 每2周技术评审，及时调整方案
2. **性能风险**: 持续基准测试，性能监控
3. **数据风险**: 多样化数据收集，数据质量检查
4. **部署风险**: 分阶段部署，回滚机制
5. **运维风险**: 自动化监控，24/7支持

这个实施计划确保项目按时交付，满足DesignDoc中的所有技术要求和性能指标。 