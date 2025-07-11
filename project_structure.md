# Cherry Vision - 代码文件结构设计

基于DesignDoc的系统架构，以下是推荐的代码文件结构：

```
cherry-vision/
├── README.md
├── requirements.txt
├── setup.py
├── .env.example
├── .gitignore
├── docker-compose.yml
├── Dockerfile
│
├── config/                          # 配置文件
│   ├── __init__.py
│   ├── settings.py                  # 全局配置
│   ├── drone_config.yaml            # 无人机配置
│   ├── camera_config.yaml           # 相机配置  
│   ├── model_config.yaml            # AI模型配置
│   └── azure_config.yaml            # Azure云服务配置
│
├── src/                             # 主要源代码
│   ├── __init__.py
│   │
│   ├── hardware/                    # 硬件控制模块
│   │   ├── __init__.py
│   │   ├── drone_controller.py      # DJI Agras T40控制
│   │   ├── camera_controller.py     # FLIR Blackfly S控制
│   │   ├── sensor_manager.py        # 传感器管理 (RGB + NIR)
│   │   └── dock_controller.py       # DJI Dock无人机舱控制
│   │
│   ├── edge/                        # 边缘计算模块 (Jetson Nano)
│   │   ├── __init__.py
│   │   ├── image_preprocessor.py    # 图像预处理 (裁剪、去噪、压缩)
│   │   ├── edge_inference.py        # 边缘推理 (YOLOv8本地推理)
│   │   ├── data_compressor.py       # 数据压缩 (90%数据减少)
│   │   └── upload_manager.py        # 上传到云端管理
│   │
│   ├── ai/                          # AI模型模块
│   │   ├── __init__.py
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   ├── yolo_detector.py     # YOLOv8果实检测模型
│   │   │   ├── resnet_classifier.py # ResNet-50质量分级模型
│   │   │   └── model_utils.py       # 模型工具函数
│   │   ├── training/
│   │   │   ├── __init__.py
│   │   │   ├── train_detector.py    # YOLOv8训练脚本
│   │   │   ├── train_classifier.py  # ResNet-50训练脚本
│   │   │   ├── data_augmentation.py # 数据增强
│   │   │   └── evaluation.py        # 模型评估
│   │   ├── inference/
│   │   │   ├── __init__.py
│   │   │   ├── detector_service.py  # 检测服务
│   │   │   ├── classifier_service.py # 分级服务
│   │   │   └── batch_processor.py   # 批处理推理
│   │   └── utils/
│   │       ├── __init__.py
│   │       ├── image_utils.py       # 图像处理工具
│   │       ├── bbox_utils.py        # 边界框处理
│   │       └── metrics.py           # 评估指标
│   │
│   ├── cloud/                       # 云服务模块
│   │   ├── __init__.py
│   │   ├── azure/
│   │   │   ├── __init__.py
│   │   │   ├── blob_manager.py      # Blob Storage存储管理
│   │   │   ├── ml_service.py        # Azure ML推理服务
│   │   │   ├── postgres_manager.py  # Azure Database管理
│   │   │   └── functions.py         # Azure Functions
│   │   ├── api/
│   │   │   ├── __init__.py
│   │   │   ├── routes.py            # API路由
│   │   │   ├── auth.py              # 认证
│   │   │   └── middleware.py        # 中间件
│   │   └── services/
│   │       ├── __init__.py
│   │       ├── image_service.py     # 图像处理服务
│   │       ├── analysis_service.py  # 分析服务
│   │       └── report_service.py    # 报告生成服务
│   │
│   ├── data/                        # 数据处理模块
│   │   ├── __init__.py
│   │   ├── collection/
│   │   │   ├── __init__.py
│   │   │   ├── image_collector.py   # 图像收集
│   │   │   └── annotation_tools.py  # 标注工具
│   │   ├── processing/
│   │   │   ├── __init__.py
│   │   │   ├── data_pipeline.py     # 数据流水线
│   │   │   ├── quality_assessment.py # 质量评估
│   │   │   └── statistics.py        # 统计分析
│   │   └── storage/
│   │       ├── __init__.py
│   │       ├── database_models.py   # 数据库模型
│   │       └── data_access.py       # 数据访问层
│   │
│   └── utils/                       # 通用工具
│       ├── __init__.py
│       ├── logger.py                # 日志工具
│       ├── exceptions.py            # 自定义异常
│       ├── validators.py            # 数据验证
│       └── helpers.py               # 辅助函数
│
├── frontend/                        # React前端
│   ├── public/
│   │   ├── index.html
│   │   └── favicon.ico
│   ├── src/
│   │   ├── components/
│   │   │   ├── Dashboard/
│   │   │   │   ├── Dashboard.jsx
│   │   │   │   ├── YieldHeatmap.jsx
│   │   │   │   ├── QualityPieChart.jsx
│   │   │   │   └── FilterPanel.jsx
│   │   │   ├── Reports/
│   │   │   │   ├── ReportGenerator.jsx
│   │   │   │   ├── ExportModal.jsx
│   │   │   │   └── ReportHistory.jsx
│   │   │   └── Common/
│   │   │       ├── Header.jsx
│   │   │       ├── Navigation.jsx
│   │   │       └── LoadingSpinner.jsx
│   │   ├── services/
│   │   │   ├── api.js               # API调用
│   │   │   ├── dataService.js       # 数据服务
│   │   │   └── exportService.js     # 导出服务
│   │   ├── utils/
│   │   │   ├── constants.js         # 常量
│   │   │   ├── formatters.js        # 数据格式化
│   │   │   └── validators.js        # 前端验证
│   │   ├── styles/
│   │   │   ├── global.css
│   │   │   └── components/
│   │   ├── App.jsx
│   │   └── index.js
│   ├── package.json
│   └── webpack.config.js
│
├── scripts/                         # 脚本文件
│   ├── setup/
│   │   ├── install_dependencies.sh  # 依赖安装
│   │   ├── setup_jetson.sh          # Jetson Nano配置
│   │   └── deploy_azure.sh          # Azure部署
│   ├── data/
│   │   ├── collect_images.py        # 图像收集脚本
│   │   ├── annotate_data.py         # 数据标注脚本
│   │   └── prepare_dataset.py       # 数据集准备
│   ├── training/
│   │   ├── train_models.py          # 模型训练主脚本
│   │   ├── evaluate_models.py       # 模型评估脚本
│   │   └── export_models.py         # 模型导出脚本
│   └── deployment/
│       ├── deploy_edge.py           # 边缘设备部署
│       ├── deploy_azure.py          # Azure云端部署
│       └── health_check.py          # 健康检查
│
├── tests/                           # 测试文件
│   ├── __init__.py
│   ├── unit/
│   │   ├── test_models.py           # 模型单元测试
│   │   ├── test_hardware.py         # 硬件单元测试
│   │   └── test_services.py         # 服务单元测试
│   ├── integration/
│   │   ├── test_pipeline.py         # 流水线集成测试
│   │   └── test_api.py              # API集成测试
│   └── e2e/
│       └── test_full_workflow.py    # 端到端测试
│
├── data/                            # 数据目录
│   ├── raw/                         # 原始数据
│   │   ├── images/
│   │   │   ├── rgb/
│   │   │   └── nir/
│   │   └── annotations/
│   ├── processed/                   # 处理后数据
│   │   ├── train/
│   │   ├── val/
│   │   └── test/
│   └── models/                      # 训练好的模型
│       ├── yolov8_cherry.pt
│       └── resnet50_grading.pt
│
├── docs/                            # 文档
│   ├── api/                         # API文档
│   │   ├── endpoints.md
│   │   └── schemas.md
│   ├── deployment/                  # 部署文档
│   │   ├── edge_deployment.md
│   │   ├── cloud_deployment.md
│   │   └── hardware_setup.md
│   ├── user_guide/                  # 用户指南
│   │   ├── dashboard_guide.md
│   │   └── operator_manual.md
│   └── development/                 # 开发文档
│       ├── model_training.md
│       ├── data_collection.md
│       └── troubleshooting.md
│
├── deployment/                      # 部署配置
│   ├── docker/
│   │   ├── Dockerfile.edge          # 边缘设备Docker
│   │   ├── Dockerfile.api           # API服务Docker
│   │   └── Dockerfile.frontend      # 前端Docker
│   ├── kubernetes/
│   │   ├── namespace.yaml
│   │   ├── api-deployment.yaml
│   │   └── frontend-deployment.yaml
│   ├── azure/
│   │   ├── arm-templates/           # ARM模板
│   │   ├── functions/               # Azure Functions
│   │   └── ml-endpoints/            # ML端点配置
│   └── ansible/
│       ├── edge_setup.yml
│       └── system_config.yml
│
└── monitoring/                      # 监控配置
    ├── grafana/
    │   └── dashboards/
    ├── prometheus/
    │   └── alerts.yml
    └── logs/
        └── logstash.conf
```

## 关键设计决策

### 1. 模块化架构
- **硬件层** (`hardware/`): 抽象化无人机和相机控制
- **边缘层** (`edge/`): Jetson Nano上的实时处理
- **AI层** (`ai/`): 独立的模型训练和推理模块
- **云层** (`cloud/`): Azure服务集成和API
- **前端层** (`frontend/`): React仪表板

### 2. 数据流设计
- 清晰的数据收集、处理、存储路径
- 支持离线和在线推理模式
- 批处理和实时处理并存

### 3. 部署友好
- Docker容器化支持
- Kubernetes编排配置
- CI/CD流水线集成

### 4. 可扩展性
- 插件化的模型架构
- 可配置的硬件支持
- 多环境部署支持

这个结构支持DesignDoc中提到的所有功能需求，包括>95%的检测精度、>90%的分级精度、<5秒的处理时间等性能目标。 