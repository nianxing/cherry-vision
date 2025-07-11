# DesignDoc: Fruit Detection and Quality Grading System for DDC Cherry Orchards

**Author**: Cindy 
**Date**: July 11, 2025  
**Version**: 1.0  
**Status**: Draft  
**Prepared for**: David del Curto (DDC)  
**Intended Audience**: Development Team, AI Engineers, Cursor Code Assistant  

---

## 1. Objective
Design and implement a fruit detection and quality grading module for DDC’s cherry orchards, enabling automated cherry counting (>95% accuracy) and quality grading (J/JJ/JJJ, >90% accuracy) across 10,000 mu (1,650 acres or 6,670,000 m²) over a 6-month harvest season. The system uses drones, ground multispectral cameras, and AI models (YOLOv8 for detection, ResNet-50 for grading), delivering real-time reports (<1 min/orchard) to optimize harvest planning and market pricing.

---

## 2. Background
- **Context**: DDC requires a pre-harvest estimation system to predict cherry yield and quality (J/JJ/JJJ grades) for export planning, targeting high-value markets (e.g., Chinese New Year). The system must handle complex orchard environments (dense canopies, variable lighting) and scale to 10,000 mu.
- **Requirements**:
  - **Fruit Detection**: Count cherries with <5% error (>95% accuracy).
  - **Quality Grading**: Classify cherries into J (premium), JJ (medium), JJJ (substandard) based on color, size, and defects, with >90% accuracy.
  - **Performance**: Image processing <5 sec/frame, report generation <1 min/orchard.
  - **Output**: Web dashboard with yield/quality reports (heatmaps, pie charts), exportable as CSV/PDF.
  - **Scalability**: Support multiple cherry varieties (e.g., Bing, Lapins) and future orchard expansion.
- **Existing Solutions**:
  - **AgEagle (Apples)**: YOLOv5 achieves 96% counting accuracy, 91% grading accuracy.
  - **PrecisionHawk (Citrus)**: Faster R-CNN achieves 94% counting, 89% defect detection.
  - **Fonterra (Pasture)**: Multispectral + CNN achieves >90% yield prediction, scalable cloud architecture.

---

## 3. System Design

### 3.1 Architecture Overview
The system integrates hardware (drones, cameras), edge computing, AI models, cloud processing, and a web dashboard. Key components:

- **Image Acquisition**:
  - Drones (4x DJI Agras T40, RGB + multispectral, 12 MP/5 MP): Cover 495 acres (2,001,000 m²) each.
  - Ground Cameras (20x FLIR Blackfly S, RGB + NIR): Cover 82.5 acres (333,500 m²) each, addressing dense canopies.
- **Edge Computing**:
  - NVIDIA Jetson Nano (per drone/camera): Preprocess images (crop, denoise, compress).
- **AI Analysis**:
  - YOLOv8: Detect and count cherries.
  - ResNet-50: Grade cherries (J/JJ/JJJ based on color, size, defects).
- **Cloud Platform**:
  - AWS (S3 for storage, SageMaker for inference, RDS for metadata): Aggregate results, generate reports.
- **Frontend**:
  - React + D3.js: Web dashboard for yield/quality visualization.
- **Optional Drone Hangar**:
  - DJI Dock: Automated storage/charging, reduces 10% maintenance costs.

### 3.2 Data Flow
1. **Acquisition**: Drones and cameras capture RGB + NIR images (JPEG/TIFF, 1-2 MB/frame).
2. **Edge Preprocessing**: Jetson Nano crops cherry regions, denoises, compresses images.
3. **AI Processing**:
   - YOLOv8 (edge/cloud): Detects cherries, outputs bounding boxes and counts.
   - ResNet-50 (cloud): Grades cherries (J/JJ/JJJ) based on color (RGB), size (pixel area), defects (NIR).
4. **Data Aggregation**: AWS S3 stores images, RDS stores results, Lambda triggers report generation.
5. **Visualization**: React dashboard displays yield (heatmap), quality (pie chart), exports CSV/PDF.

### 3.3 Component Details
#### 3.3.1 Fruit Detection (YOLOv8)
- **Model**: YOLOv8 (Ultralytics, PyTorch-based).
- **Input**: RGB + NIR images (1080p, preprocessed).
- **Output**: Bounding boxes (x, y, width, height), cherry count, confidence score (threshold: 0.9).
- **Performance**: <2 sec/frame (edge), <1 sec/frame (cloud), >95% accuracy.
- **Training**:
  - Dataset: 50,000 cherry images (10,000 initial + 40,000 pilot), covering Bing/Lapins, 30-50% occlusion, day/cloudy conditions.
  - Augmentation: Flip, scale, brightness adjustment.
  - Pretraining: COCO dataset, fine-tune on cherry data.

#### 3.3.2 Quality Grading (ResNet-50)
- **Model**: ResNet-50 (PyTorch, 50-layer CNN).
- **Input**: Cropped cherry images (224x224 pixels, RGB + NIR).
- **Output**: J/JJ/JJJ classification, confidence score (threshold: 0.85).
- **Grading Criteria** (to be finalized with DDC):
  - J: Red (R>0.8), diameter >25mm, no defects.
  - JJ: Red (R: 0.6-0.8), diameter 20-25mm, minor defects.
  - JJJ: Red (R<0.6), diameter <20mm, visible defects.
- **Performance**: <3 sec/frame (cloud), >90% accuracy.
- **Training**:
  - Dataset: 50,000 images (J:JJ:JJJ = 3:5:2).
  - Augmentation: Color jitter, rotation, scaling.
  - Pretraining: ImageNet, fine-tune on cherry data.

#### 3.3.3 Edge Processing
- **Hardware**: NVIDIA Jetson Nano (4GB RAM, GPU).
- **Tasks**:
  - Crop cherry regions (color/shape thresholds).
  - Denoise (Gaussian blur).
  - Compress to JPEG (1-2 MB/frame).
- **Performance**: <5 sec/frame, 90% data reduction for upload.

#### 3.3.4 Cloud Platform
- **Storage**: AWS S3 (images), RDS (PostgreSQL, metadata/counts/grades).
- **Inference**: AWS SageMaker (YOLOv8 + ResNet-50, GPU instances).
- **Pipeline**: AWS Lambda triggers inference and report generation (<1 min/orchard).
- **Scalability**: Supports 100 drones, 10,000 mu.

#### 3.3.5 Frontend
- **Framework**: React (UI), D3.js (visualization).
- **Features**:
  - Heatmap: Yield distribution by orchard.
  - Pie Chart: Quality grades (J/JJ/JJJ).
  - Filters: Orchard/variety selection.
  - Export: CSV/PDF reports.
- **Performance**: <1 sec page load, 100% export success.

---

## 4. Implementation Plan
### 4.1 Phase 1: Pilot Preparation (0-3 Months)
- **Tasks**:
  - Procure hardware: 1 drone ($5,936), 2 cameras ($29,680), optional hangar ($88,800), AWS ($7,000/month).
  - Collect 10,000 cherry images (RGB + NIR), annotate 5,000 (LabelImg/CVAT).
  - Train YOLOv8 (detection) and ResNet-50 (grading) on initial dataset.
  - Deploy edge (Jetson Nano) and cloud (SageMaker) pipelines.
  - Develop React dashboard prototype.
- **Deliverables**: Pilot hardware deployed, initial models (90% counting, 85% grading accuracy), basic dashboard.
- **Team**: 2 AI engineers, 1 drone operator, 1 frontend dev, 1 backend dev.
- **Budget**: $120,167 (no hangar), $235,607 (with hangar).

### 4.2 Phase 2: Pilot Testing & Optimization (3-6 Months)
- **Tasks**:
  - Test on 1,000 mu (165 acres), collect 40,000 additional images.
  - Fine-tune models: >95% counting, >90% grading accuracy.
  - Optimize edge processing (<5 sec/frame), cloud pipeline (<1 min/report).
  - Train DDC operators (2 weeks).
- **Deliverables**: Optimized models, pilot report (accuracy, efficiency), trained operators.
- **Team**: Add 1 data annotator.
- **Budget**: $20,000 (data annotation).

### 4.3 Phase 3: Full Deployment (6-9 Months)
- **Tasks**:
  - Procure remaining hardware: 3 drones ($17,808), 18 cameras ($267,120), AWS ($42,000/6 months).
  - Deploy across 10,000 mu, integrate multi-orchard data.
  - Enhance dashboard: Multi-orchard filters, predictive analytics.
  - Provide 6-month support (phone + on-site).
- **Deliverables**: Full system covering 10,000 mu, comprehensive dashboard, support plan.
- **Team**: Add 1 ops engineer.
- **Budget**: $565,744.

---

## 5. Dependencies
- **Hardware**:
  - Drones: DJI Agras T40 (4x, $5,936 each).
  - Cameras: FLIR Blackfly S (20x, $14,840 each).
  - Edge: NVIDIA Jetson Nano (24x, ~$100 each).
  - Optional Hangar: DJI Dock ($88,800).
- **Software**:
  - Frameworks: PyTorch (YOLOv8, ResNet-50), React, D3.js.
  - Cloud: AWS (S3, SageMaker, RDS, Lambda).
  - Tools: LabelImg/CVAT (annotation), TensorRT (model optimization).
- **Data**:
  - 50,000 cherry images (RGB + NIR, 10,000 initial + 40,000 pilot).
  - Pretrained models: COCO (YOLOv8), ImageNet (ResNet-50).
- **External References**:
  - **Open Source**: YOLOv8 (`github.com/ultralytics/ultralytics`), ResNet (`pytorch/vision`).
  - **Datasets**: Fruit-360 (`kaggle.com/datasets/moltean/fruits`), PlantVillage (`kaggle.com/datasets/plantvillage/plantvillage-dataset`).
  - **Papers**: Horticulture Research (2021, `nature.com/hortres`), IEEE Access (2023, Cherry Detection, `ieeexplore.ieee.org`).
  - **Patents**: US10902566B2 (AgEagle), CN111507973A (Jiangsu University, `patents.google.com`).

---

## 6. Risks and Mitigations
- **Risk 1: Low Accuracy in Complex Environments** (dense canopies, lighting variations).
  - **Mitigation**: Use ground cameras (20x) to supplement drones, train on 50,000 diverse images (30-50% occlusion, day/cloudy).
- **Risk 2: Grading Criteria Misalignment** (J/JJ/JJJ standards).
  - **Mitigation**: Collaborate with DDC to define standards (color: RGB values, size: diameter, defects: NIR thresholds), validate in pilot.
- **Risk 3: Processing Latency** (>5 sec/frame).
  - **Mitigation**: Optimize YOLOv8 with TensorRT, use AWS SageMaker GPU instances, preprocess on Jetson Nano.
- **Risk 4: Insufficient Data** (cherry-specific images).
  - **Mitigation**: Collect 10,000 initial images, augment with 40,000 in pilot, use semi-supervised learning (Fruit-360 pretraining).

---

## 7. Success Criteria
- **Accuracy**:
  - Counting: <5% error (>95% accuracy).
  - Grading: >90% accuracy (J/JJ/JJJ).
- **Performance**:
  - Image processing: <5 sec/frame (edge + cloud).
  - Report generation: <1 min/orchard.
- **Scalability**: Supports 10,000 mu, extensible to other fruits.
- **User Experience**:
  - Dashboard satisfaction: >90%.
  - Operator training: >95% independent operation rate.
- **ROI**:
  - Revenue increase: 10-15% (~$1M, assuming $10M DDC revenue).
  - Cost savings: 8-12% (~$800,000) via reduced scouting/planning.

---

## 8. References
- **Open Source**:
  - YOLOv8: `github.com/ultralytics/ultralytics`
  - ResNet-50: `pytorch/vision`
  - OpenWeedLocator: `github.com/geezacoleman/OpenWeedLocator`
- **Datasets**:
  - Fruit-360: `kaggle.com/datasets/moltean/fruits`
  - PlantVillage: `kaggle.com/datasets/plantvillage/plantvillage-dataset`
- **Papers**:
  - Horticulture Research (2021): `nature.com/hortres`
  - IEEE Access (2023, Cherry Detection): `ieeexplore.ieee.org`
  - MDPI (2022, Fruit Detection): `mdpi.com`
- **Patents**:
  - US10366308B2 (IntelliFruit): `patents.google.com`
  - US10902566B2 (AgEagle): `patents.google.com`
  - CN111507973A (Jiangsu University): `patents.google.com`
- **Case Studies**:
  - AgEagle (Apples): 96% counting, 91% grading.
  - PrecisionHawk (Citrus): 94% counting, 89% grading.
  - Fonterra (Pasture): >90% yield prediction.

---

## 9. Next Steps
1. **Confirm Requirements**: Validate J/JJ/JJJ grading criteria with DDC (color, size, defect thresholds).
2. **Data Collection**: Start 10,000 cherry image collection (RGB + NIR, DJI Agras T40, FLIR Blackfly S).
3. **Model Development**:
   - Clone YOLOv8 (`pip install ultralytics`), fine-tune on cherry data.
   - Clone ResNet-50 (`pytorch/vision`), train on J/JJ/JJJ labels.
4. **Pilot Deployment**: Deploy on 1,000 mu, test accuracy/performance.
5. **Legal Review**: Check patents (US10902566B2, CN111507973A) for potential licensing.

---

**Note for Cursor**: This DesignDoc provides a blueprint for implementing the fruit detection and quality grading module. Use the referenced open-source code (YOLOv8, ResNet-50) and datasets (Fruit-360, custom cherry data) to generate model training scripts, edge preprocessing logic, and React dashboard components. Suggest code snippets for YOLOv8 fine-tuning, ResNet-50 classification, and AWS Lambda pipeline integration. Highlight any missing details (e.g., specific cherry varieties, orchard layouts) for further clarification.
