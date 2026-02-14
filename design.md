# System Design Document
## AI-Based Real-Time Visual Forensics & Injury Detection System

**Project**: Visual Forensics & Injury Detection for Indian Army and Disaster Response  
**Version**: 1.0  
**Date**: February 14, 2026  
**Status**: Design Phase

---

## 1. System Architecture

### 1.1 High-Level Architecture

```
┌─────────────────┐
│  Input Layer    │
│ (Mobile/Web/    │
│  Edge Device)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   API Gateway   │
│ (Authentication,│
│   Rate Limit)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  AI Engine      │
│ ┌─────────────┐ │
│ │ Tamper/     │ │
│ │ Deepfake    │ │
│ │ Detection   │ │
│ └─────────────┘ │
│ ┌─────────────┐ │
│ │ Injury      │ │
│ │ Detection   │ │
│ └─────────────┘ │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Decision Layer  │
│ (Risk Scoring,  │
│  Alert Logic)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Output Dashboard│
│ (Reports, Alerts│
│  Visualization) │
└─────────────────┘
```

### 1.2 Component Architecture

**Client Layer**
- Mobile App (Android/iOS) - React Native or Flutter
- Web Dashboard - React.js with real-time updates
- Edge Device Interface - Lightweight Python client

**Backend Services**
- API Server - Node.js/Express or Python/FastAPI
- Authentication Service - JWT-based auth
- File Storage Service - S3-compatible storage
- Database - PostgreSQL for metadata, MongoDB for logs

**AI Model Service**
- Model Inference Server - Python/FastAPI with GPU support
- Model Registry - MLflow or custom versioning
- Preprocessing Pipeline - OpenCV, PIL
- Post-processing & Analytics

**Infrastructure**
- Load Balancer - Nginx or cloud-native
- Message Queue - Redis/RabbitMQ for async processing
- Monitoring - Prometheus + Grafana
- Logging - ELK Stack or cloud logging

### 1.3 Deployment Modes

**Mode 1: Cloud Deployment** (Primary)
- Hosted on AWS/Azure/GCP
- Auto-scaling for variable load
- High availability with multi-region support

**Mode 2: Edge Deployment** (Offline/Field Operations)
- Raspberry Pi 4 or Jetson Nano
- Local model inference
- Sync to cloud when connectivity available
- Reduced model size (quantization, pruning)

**Mode 3: Hybrid Mode**
- Edge preprocessing + cloud inference
- Fallback to edge-only in network failure
- Priority queue for critical cases

---

## 2. Component Breakdown

### 2.1 Frontend Components

**Mobile Application**
- Camera capture with real-time preview
- Image upload from gallery
- Offline queue management
- Push notification for alerts
- Report viewing and export (PDF)
- Multi-language support (Hindi, English, regional)

**Web Dashboard**
- Admin panel for user management
- Real-time case monitoring
- Analytics and statistics
- Bulk upload interface
- Model performance metrics
- Audit log viewer

**Key UI Screens**
1. Capture Screen - Camera interface with guidelines
2. Analysis Screen - Loading state with progress
3. Results Screen - Visual indicators, confidence scores
4. History Screen - Past analyses with filters
5. Settings Screen - Preferences, offline mode toggle

### 2.2 Backend Components

**API Endpoints**
```
POST   /api/v1/auth/login
POST   /api/v1/auth/refresh
POST   /api/v1/analysis/upload
GET    /api/v1/analysis/{id}
GET    /api/v1/analysis/history
POST   /api/v1/analysis/batch
GET    /api/v1/reports/{id}
POST   /api/v1/feedback
GET    /api/v1/models/status
```

**Authentication & Authorization**
- JWT tokens with refresh mechanism
- Role-based access control (RBAC)
  - Admin: Full access
  - Operator: Upload and view
  - Viewer: Read-only access
- API key for edge devices
- Session management with timeout

**Logging & Monitoring**
- Request/response logging
- Error tracking with stack traces
- Performance metrics (latency, throughput)
- Model inference time tracking
- User activity audit logs

### 2.3 AI Engine Components

**Component 1: Visual Forensics (Tamper/Deepfake Detection)**

*Purpose*: Detect image manipulation, deepfakes, and authenticity issues

*Model Architecture* (Conceptual):
- Base: EfficientNet-B4 or Vision Transformer (ViT)
- Custom head for binary/multi-class classification
- Attention mechanism to highlight manipulated regions
- Ensemble approach for higher accuracy

*Detection Capabilities*:
- Copy-move forgery
- Splicing detection
- Deepfake face detection
- Metadata inconsistency analysis
- Compression artifact analysis

*Output*:
- Authenticity score (0-100%)
- Confidence level
- Heatmap of suspicious regions
- Metadata report

**Component 2: Injury Detection**

*Purpose*: Identify visible injuries for triage and documentation

*Model Architecture* (Conceptual):
- Object detection: YOLOv8 or Faster R-CNN
- Segmentation: U-Net or Mask R-CNN for precise localization
- Classification head for injury type

*Detection Capabilities*:
- Visible wounds (cuts, lacerations)
- Bruising and contusions
- Burns (degree estimation - conceptual)
- Swelling detection
- Fracture indicators (deformity, unnatural angles)
- Bleeding severity estimation

*Output*:
- Bounding boxes with injury type
- Severity score (mild/moderate/severe)
- Body part identification
- Triage priority recommendation

**Preprocessing Pipeline**
```python
# Conceptual flow
1. Image validation (format, size, corruption check)
2. Resize and normalize
3. Color space conversion if needed
4. Noise reduction (optional)
5. Augmentation for robustness (rotation, brightness)
6. Batch preparation
```

**Inference Pipeline**
```python
# Conceptual flow
1. Load image from request
2. Run preprocessing
3. Model inference (parallel for both models)
4. Post-processing (NMS, threshold filtering)
5. Generate visualization overlays
6. Compile results JSON
7. Store results and return response
```

### 2.4 Data Layer

**Database Schema** (Simplified)

```sql
-- Users table
users (
  id, username, email, role, 
  created_at, last_login
)

-- Analysis records
analyses (
  id, user_id, image_url, 
  forensics_result, injury_result,
  status, created_at, metadata
)

-- Model versions
model_versions (
  id, model_name, version, 
  accuracy, deployed_at, active
)

-- Audit logs
audit_logs (
  id, user_id, action, 
  resource, timestamp, ip_address
)
```

**File Storage**
- Original images: S3/MinIO with encryption
- Processed images: Separate bucket with retention policy
- Reports: PDF storage with signed URLs
- Model artifacts: Versioned storage

**Dataset Management**
- Training data: Labeled datasets with version control
- Validation split: 80/10/10 (train/val/test)
- Data augmentation pipeline
- Annotation tools integration (Label Studio, CVAT)

---

## 3. Data Flow

### 3.1 End-to-End Flow

**Step 1: Image Capture/Upload**
```
User captures image → Client validates (size, format) 
→ Compress if needed → Upload to API with metadata
```

**Step 2: API Processing**
```
API receives request → Authenticate user 
→ Validate payload → Generate unique analysis ID
→ Store image in object storage → Queue for processing
→ Return analysis ID to client
```

**Step 3: AI Processing**
```
Worker picks job from queue → Download image
→ Run preprocessing → Parallel inference:
  ├─ Forensics model inference
  └─ Injury detection model inference
→ Post-process results → Generate visualizations
→ Calculate risk scores
```

**Step 4: Decision Layer**
```
Aggregate results → Apply business rules:
  - If authenticity < 50%: Flag as suspicious
  - If severe injury detected: High priority alert
  - If multiple injuries: Escalate to supervisor
→ Generate alert if needed → Store final results
```

**Step 5: Response & Notification**
```
Update analysis status → Send push notification
→ Client polls/receives webhook → Display results
→ User can view report, export PDF, provide feedback
```

### 3.2 Error Handling & Fallback

**Network Failure (Client Side)**
- Queue uploads locally with SQLite
- Retry with exponential backoff
- Show offline indicator
- Sync when connection restored

**Processing Failure (Server Side)**
- Retry logic with max 3 attempts
- Dead letter queue for failed jobs
- Alert admin on repeated failures
- Graceful degradation (return partial results)

**Model Inference Failure**
- Fallback to previous model version
- Return error with retry suggestion
- Log failure for debugging
- Health check endpoint for monitoring

**Weak Network Handling**
- Adaptive image compression
- Progressive upload with resume capability
- Thumbnail preview first, full analysis la
