# Diagramas de Componentes y Arquitectura Técnica

## 1. DIAGRAMA C4 - Vista de Contexto

```
┌─────────────────────────────────────────────────────────────────────┐
│                      SISTEMA DE CONFERENCIA                          │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │                   APLICACIÓN WEB                               │ │
│  │                                                                │ │
│  │  Portal Autores | Portal Revisores | Panel Admin              │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
       ▲                    ▲                    ▲
       │                    │                    │
       │ Navega             │ Evaluación         │ Administra
       │                    │                    │
    ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
    │    AUTOR     │  │   REVISOR    │  │ ADMINISTRADOR│
    └──────────────┘  └──────────────┘  └──────────────┘
       │                    │
       │ Consultas          │ Consultas
       └────────┬───────────┘
                ▼
        ┌─────────────────┐
        │ SISTEMA DE EMAIL│
        │ (Notificaciones)│
        └─────────────────┘
```

## 2. DIAGRAMA DE COMPONENTES

```
┌──────────────────────────────────────────────────────────────────────┐
│                      CAPA DE PRESENTACIÓN                            │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────────┐  ┌────────────────────┐  ┌──────────────┐  │
│  │  PORTAL AUTORES    │  │ PORTAL REVISORES   │  │ PANEL ADMIN  │  │
│  ├────────────────────┤  ├────────────────────┤  ├──────────────┤  │
│  │ React/Vue.js       │  │ React/Vue.js       │  │ React/Vue.js │  │
│  │                    │  │                    │  │              │  │
│  │ Componentes:       │  │ Componentes:       │  │ Componentes: │  │
│  │ - Dashboard        │  │ - Dashboard        │  │ - Dashboard  │  │
│  │ - Envío Papers     │  │ - Envío Reviews    │  │ - Usuarios   │  │
│  │ - Historial        │  │ - Listado Papers   │  │ - Reportes   │  │
│  │ - Perfil           │  │ - Perfil           │  │ - Distribución│ │
│  └────────────────────┘  └────────────────────┘  └──────────────┘  │
│         │                        │                      │            │
└─────────┼────────────────────────┼──────────────────────┼────────────┘
          │                        │                      │
          └────────────────────────┼──────────────────────┘
                                   │
                REST/JSON/Axios    │
                                   ▼
┌──────────────────────────────────────────────────────────────────────┐
│                   CAPA DE APLICACIÓN (API)                           │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    API GATEWAY                              │   │
│  │  (Rate Limiting, Auth, CORS, Logging)                       │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌─────────────────────┐  ┌─────────────────────┐  ┌────────────┐  │
│  │ CONTROLLADOR AUTH   │  │ CONTROLADOR PAPERS  │  │ CONTROLADOR│  │
│  │                     │  │                     │  │ ADMIN      │  │
│  │ - /auth/login       │  │ - POST /papers      │  │            │  │
│  │ - /auth/register    │  │ - GET /papers       │  │ - Distrib. │  │
│  │ - /auth/refresh     │  │ - GET /papers/{id}  │  │ - Reportes │  │
│  │ - /auth/logout      │  │ - PUT /papers/{id}  │  │            │  │
│  │ - /auth/me          │  │ - DELETE /papers    │  │            │  │
│  └─────────────────────┘  └─────────────────────┘  └────────────┘  │
│                                                                      │
│  ┌─────────────────────┐  ┌─────────────────────┐  ┌────────────┐  │
│  │ CONTROLADOR REVIEWS │  │ CONTROLADOR NOTIF.  │  │ CONTROLADOR│  │
│  │                     │  │                     │  │ PROGRAMA   │  │
│  │ - GET /reviews      │  │ - GET /notif        │  │            │  │
│  │ - POST /reviews     │  │ - POST /notif/send  │  │ - Build    │  │
│  │ - PUT /reviews/{id} │  │ - DELETE /notif     │  │ - Export   │  │
│  │ - GET /stats        │  │ - GET /notif/stats  │  │            │  │
│  └─────────────────────┘  └─────────────────────┘  └────────────┘  │
│         │                        │                      │            │
└─────────┼────────────────────────┼──────────────────────┼────────────┘
          │                        │                      │
          │ Delegation            │                       │
          │                       │                       │
          └───────────┬───────────┴───────────────────────┘
                      ▼
┌──────────────────────────────────────────────────────────────────────┐
│           CAPA DE LÓGICA DE NEGOCIO (Services)                       │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ ┌──────────┐ │
│  │PaperService  │  │ReviewService │  │SelectService │ │NotifServ │ │
│  │              │  │              │  │              │ │          │ │
│  │- Submit      │  │- Create      │  │- SelectPapers│ │- Send    │ │
│  │- Validate    │  │- Submit      │  │- Decide      │ │- Queue   │ │
│  │- Update      │  │- Compile     │  │- Calculate   │ │- Track   │ │
│  │- Upload Ver. │  │- Stats       │  │- Change Req. │ │          │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ └──────────┘ │
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ ┌──────────┐ │
│  │UserService   │  │ValidationServ│  │ProgramService│ │CacheServ │ │
│  │              │  │              │  │              │ │          │ │
│  │- Create      │  │- Validate    │  │- Build       │ │- Get     │ │
│  │- Import      │  │- Format      │  │- Schedule    │ │- Set     │ │
│  │- Profile     │  │- Sanitize    │  │- Export      │ │- Invalidate
│  │- Auth        │  │- Check Rules │  │- Gen Report  │ │          │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ └──────────┘ │
│                    │                                                │
└────────────────────┼────────────────────────────────────────────────┘
                     │
                     │ ORM / Queries
                     ▼
┌──────────────────────────────────────────────────────────────────────┐
│              CAPA DE ACCESO A DATOS (Repositories)                   │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ ┌──────────┐ │
│  │PaperRepo     │  │ReviewRepo    │  │UserRepo      │ │DecisRepo │ │
│  │              │  │              │  │              │ │          │ │
│  │- findById    │  │- findById    │  │- findById    │ │- findById│ │
│  │- findByAuth  │  │- findByPaper │  │- findByEmail │ │- findByP │ │
│  │- save        │  │- save        │  │- save        │ │- save    │ │
│  │- update      │  │- update      │  │- update      │ │- update  │ │
│  │- delete      │  │- delete      │  │- delete      │ │- delete  │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ └──────────┘ │
│                    │                                                │
└────────────────────┼────────────────────────────────────────────────┘
                     │
              ┌──────┴──────┬──────────┬──────────┐
              ▼             ▼          ▼          ▼
        ┌─────────────┐ ┌────────┐ ┌──────┐ ┌──────────┐
        │ PostgreSQL  │ │ Redis  │ │ S3   │ │RABBITMQ/Q│
        │ (Primary DB)│ │(Cache) │ │(Files)│ │ (Queue)  │
        └─────────────┘ └────────┘ └──────┘ └──────────┘
```

## 3. DIAGRAMA DE SERVICIOS PRINCIPALES

```
┌───────────────────────────────────────────────────────────────────────┐
│                        DOMINIO: PAPER                                │
├───────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  PaperService (Facade)                                               │
│  ├─ submitPaper(author, file, metadata)                              │
│  │   ├─ ValidationService.validateFormat(file)                       │
│  │   ├─ ValidationService.validateMetadata(metadata)                 │
│  │   ├─ PaperRepository.save(paper)                                  │
│  │   └─ generate_event(PaperSubmitted)                               │
│  │                                                                    │
│  ├─ updatePaperVersion(paperId, newFile)                             │
│  │   ├─ PaperRepository.findById(paperId)                            │
│  │   ├─ ValidationService.validateFormat(newFile)                    │
│  │   ├─ PaperRepository.update(paper)                                │
│  │   └─ generate_event(PaperVersionUpdated)                          │
│  │                                                                    │
│  ├─ getPapersByAuthor(authorId)                                      │
│  │   ├─ CacheService.get("papers:author:{authorId}")                 │
│  │   └─ PaperRepository.findByAuthor(authorId)                       │
│  │                                                                    │
│  └─ getPapersByStatus(status, filters)                               │
│      ├─ ValidationService.validateStatus(status)                     │
│      └─ PaperRepository.findByStatus(status)                         │
│                                                                       │
└───────────────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────────────┐
│                        DOMINIO: REVIEW                               │
├───────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ReviewService (Facade)                                              │
│  ├─ createReview(paperId, reviewerId)                                │
│  │   ├─ PaperRepository.findById(paperId)                            │
│  │   ├─ UserRepository.findById(reviewerId)                          │
│  │   ├─ ReviewRepository.save(review)                                │
│  │   └─ generate_event(ReviewCreated)                                │
│  │                                                                    │
│  ├─ submitReview(reviewId, reviewData)                               │
│  │   ├─ ReviewRepository.findById(reviewId)                          │
│  │   ├─ ValidationService.validateReviewFormat(reviewData)           │
│  │   ├─ ReviewRepository.update(review)                              │
│  │   └─ generate_event(ReviewSubmitted)                              │
│  │                                                                    │
│  ├─ getReviewStatistics(paperId)                                     │
│  │   ├─ ReviewRepository.findByPaper(paperId)                        │
│  │   ├─ Calculate: scoreAvg, consensus, etc.                         │
│  │   └─ return statistics                                            │
│  │                                                                    │
│  └─ compileReviews(paperId)                                          │
│      ├─ ReviewRepository.findByPaper(paperId)                        │
│      ├─ Calculate decisions                                          │
│      └─ Generate report                                              │
│                                                                       │
└───────────────────────────────────────────────────────────────────────┘

┌───────────────────────────────────────────────────────────────────────┐
│                      DOMINIO: SELECCIÓN                              │
├───────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  SelectionService (Facade)                                           │
│  ├─ selectPapers(criteria)                                           │
│  │   ├─ get all evaluated papers                                     │
│  │   ├─ apply selection criteria                                     │
│  │   ├─ DecisionRepository.save(decisions)                           │
│  │   └─ generate_event(PapersSelected)                               │
│  │                                                                    │
│  ├─ calculatePaperScore(paperId)                                     │
│  │   ├─ ReviewRepository.findByPaper(paperId)                        │
│  │   ├─ weights = get_weights()                                      │
│  │   ├─ score = weighted_average(reviews, weights)                   │
│  │   └─ return score                                                 │
│  │                                                                    │
│  ├─ identifyRequiredChanges(paperId)                                 │
│  │   ├─ ReviewRepository.findByPaper(paperId)                        │
│  │   ├─ extract suggested_changes                                    │
│  │   ├─ merge common changes                                         │
│  │   └─ return changes                                               │
│  │                                                                    │
│  └─ acceptanceCriteria(paperId)                                      │
│      ├─ get paper score                                              │
│      ├─ check consensus (>= 2 aceptaciones)                          │
│      ├─ check quality threshold                                      │
│      └─ return decision                                              │
│                                                                       │
└───────────────────────────────────────────────────────────────────────┘
```

## 4. DIAGRAMA DE FLUJO DE DATOS

```
┌─────────────────┐
│   AUTOR ENVÍA   │
│     PAPER       │
└────────┬────────┘
         │
         ▼
    ┌────────────────────────────────┐
    │ PaperController.submit_paper() │
    └────────────────────────────────┘
         │
         ▼
    ┌────────────────────────────────┐
    │ PaperService.submitPaper()     │
    │ - Validación                   │
    │ - Guardar en BD                │
    │ - Generar evento               │
    └────────────────────────────────┘
         │
         ├─────────┬──────────┐
         ▼         ▼          ▼
    ┌────────┐ ┌────────┐ ┌──────────┐
    │AuditLog│ │EventLog│ │Database  │
    └────────┘ └────────┘ └──────────┘
         │         │
         ├─────────┘
         ▼
    ┌─────────────────────────────────┐
    │ Listener: PaperSubmittedEvent   │
    │ - Enviar confirmación email     │
    │ - Update cache                  │
    │ - Trigger workflows             │
    └─────────────────────────────────┘
         │
         ├─────────┬──────────────┐
         ▼         ▼              ▼
    ┌─────────┐ ┌──────┐ ┌─────────────┐
    │    e-mail│ │Cache │ │WorkflowMgr  │
    └─────────┘ └──────┘ └─────────────┘
```

## 5. DIAGRAMA DE PROCESOS ASÍNCRONOS

```
                        EVENT QUEUE (RabbitMQ/Celery)
                               ▲
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
        │                      │                      │
        ▼                      ▼                      ▼
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│ PaperService │      │ReviewService │      │SelectService │
├──────────────┤      ├──────────────┤      ├──────────────┤
│              │      │              │      │              │
│Emit Events:  │      │Emit Events:  │      │Emit Events:  │
│              │      │              │      │              │
│- PaperSubmit │      │- ReviewSubmit│      │- Decision    │
│- PaperAccepted│     │- ReviewOverdue│     │- ChangeReq.  │
│- VersionUpd. │      │              │      │              │
└──────────────┘      └──────────────┘      └──────────────┘
        │                      │                      │
        └──────────────────────┼──────────────────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │  Event Listeners    │
                    ├─────────────────────┤
                    │                     │
                    │ Background Workers: │
                    │                     │
                    │ - EmailWorker       │
                    │ - NotificationWorker│
                    │ - PDFGeneratorWorker│
                    │ - ReportWorker      │
                    │ - ArchiveWorker     │
                    │                     │
                    └────────────┬────────┘
                                 │
                    ┌────────────┬┴─────────┬──────────┐
                    ▼            ▼         ▼          ▼
            ┌─────────────┐  ┌────────┐ ┌──────┐ ┌──────────┐
            │   SMTP      │  │Database│ │S3    │ │Analytics │
            └─────────────┘  └────────┘ └──────┘ └──────────┘
```

## 6. DIAGRAMA UML DE CLASES PRINCIPALES

```
┌──────────────────────────────────┐
│           <<Entity>>             │
│            USER                  │
├──────────────────────────────────┤
│ - id: int (PK)                   │
│ - email: string (UNIQUE)         │
│ - nombre: string                 │
│ - apellido: string               │
│ - password_hash: string          │
│ - rol: UserRole enum             │
│ - afiliacion: string             │
│ - especialidad: string           │
│ - created_at: datetime           │
│ - updated_at: datetime           │
├──────────────────────────────────┤
│ + getId()                        │
│ + getRole()                      │
│ + hasPermission(action)          │
│ + getFullName()                  │
└──────────────────────────────────┘
        ▲           ▲           ▲
        │           │           │
        │           │           │
   ┌────┴──┐   ┌────┴──┐   ┌───┴───┐
   │Author │   │Reviewer│   │Admin  │
   └───────┘   └────────┘   └───────┘

┌──────────────────────────────────┐
│          <<Entity>>              │
│           PAPER                  │
├──────────────────────────────────┤
│ - id: int (PK)                   │
│ - title: string                  │
│ - abstract: text                 │
│ - author_id: int (FK)            │
│ - co_authors: json               │
│ - file_path: string              │
│ - version: int                   │
│ - status: PaperStatus enum       │
│ - keywords: array<string>        │
│ - submission_date: datetime      │
│ - updated_at: datetime           │
├──────────────────────────────────┤
│ + getId()                        │
│ + getAuthor(): User              │
│ + getReviews(): Review[]          │
│ + getStatus()                    │
│ + canBeModified()                │
│ + getVersion()                   │
└──────────────────────────────────┘
        │              
        │ 1 to *
        ▼
┌──────────────────────────────────┐
│          <<Entity>>              │
│           REVIEW                 │
├──────────────────────────────────┤
│ - id: int (PK)                   │
│ - paper_id: int (FK)             │
│ - reviewer_id: int (FK)          │
│ - score: decimal (1-10)          │
│ - executive_summary: text        │
│ - strengths: text                │
│ - weaknesses: text               │
│ - comments: text                 │
│ - recommendation: enum           │
│ - suggested_changes: json        │
│ - submission_date: datetime      │
│ - status: ReviewStatus enum      │
├──────────────────────────────────┤
│ + getId()                        │
│ + getReviewer(): User            │
│ + getPaper(): Paper              │
│ + getRecommendation()            │
│ + getSuggestedChanges()           │
└──────────────────────────────────┘
        │
        │ 1 to *
        ▼
┌──────────────────────────────────┐
│          <<Entity>>              │
│          DECISION                │
├──────────────────────────────────┤
│ - id: int (PK)                   │
│ - paper_id: int (FK) (UNIQUE)    │
│ - decision: DecisionType enum    │
│ - committee_comments: text       │
│ - required_changes: json         │
│ - decision_date: datetime        │
│ - average_score: decimal         │
│ - acceptance_count: int          │
│ - rejection_count: int           │
├──────────────────────────────────┤
│ + getId()                        │
│ + getDecision()                  │
│ + getPaper(): Paper              │
│ + requiresChanges()              │
│ + getChangesList()               │
└──────────────────────────────────┘
```

## 7. DIAGRAMA DE SEGURIDAD

```
┌────────────────────────────────────────────────────────┐
│              CAPAS DE SEGURIDAD                        │
├────────────────────────────────────────────────────────┤
│                                                        │
│  NIVEL 1: Red & Transport                             │
│  ┌──────────────────────────────────────────────────┐ │
│  │ - HTTPS/TLS 1.3                                  │ │
│  │ - Firewall                                       │ │
│  │ - DDoS Protection                                │ │
│  │ - SSL Certificates                              │ │
│  └──────────────────────────────────────────────────┘ │
│                                                        │
│  NIVEL 2: API Gateway & Authentication               │
│  ┌──────────────────────────────────────────────────┐ │
│  │ - Rate Limiting                                  │ │
│  │ - JWT Validation                                 │ │
│  │ - CORS Configuration                             │ │
│  │ - Request Logging                                │ │
│  └──────────────────────────────────────────────────┘ │
│                                                        │
│  NIVEL 3: Authorization & Permissions                 │
│  ┌──────────────────────────────────────────────────┐ │
│  │ - Role-Based Access Control (RBAC)               │ │
│  │ - Attribute-Based Access Control (ABAC)          │ │
│  │ - Permission Validation                          │ │
│  │ - Data Ownership Checks                          │ │
│  └──────────────────────────────────────────────────┘ │
│                                                        │
│  NIVEL 4: Data & Validation                           │
│  ┌──────────────────────────────────────────────────┐ │
│  │ - Input Validation                               │ │
│  │ - SQL Injection Prevention                       │ │
│  │ - Cross-Site Scripting (XSS) Prevention          │ │
│  │ - CSRF Token Validation                          │ │
│  │ - Data Sanitization                              │ │
│  └──────────────────────────────────────────────────┘ │
│                                                        │
│  NIVEL 5: File & Storage Security                     │
│  ┌──────────────────────────────────────────────────┐ │
│  │ - MIME Type Validation                           │ │
│  │ - File Size Limits                               │ │
│  │ - Antivirus Scanning                             │ │
│  │ - Encrypted Storage                              │ │
│  │ - Access Controls                                │ │
│  └──────────────────────────────────────────────────┘ │
│                                                        │
│  NIVEL 6: Auditing & Monitoring                       │
│  ┌──────────────────────────────────────────────────┐ │
│  │ - Comprehensive Logging                          │ │
│  │ - Audit Trail                                    │ │
│  │ - Anomaly Detection                              │ │
│  │ - Security Alerts                                │ │
│  │ - Compliance Checks                              │ │
│  └──────────────────────────────────────────────────┘ │
│                                                        │
└────────────────────────────────────────────────────────┘
```

## 8. DIAGRAMA DE DESPLIEGUE

```
┌──────────────────────────────────────────────────────────────────┐
│                    PRODUCCIÓN (KUBERNETES)                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              INGRESS / LOAD BALANCER                    │   │
│  │  (nginx-ingress, manages routing & SSL)                │   │
│  └────────────┬────────────────────────────────────────────┘   │
│               │                                                 │
│               ▼                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              SERVICE: API-Gateway (ClusterIP)           │   │
│  └────────────┬────────────────────────────────────────────┘   │
│               │                                                 │
│       ┌───────┼───────┬─────────┐                               │
│       │       │       │         │                               │
│       ▼       ▼       ▼         ▼                               │
│  ┌────────┐ ┌────────┐ ┌─────────┐ ┌──────────┐               │
│  │  POD   │ │  POD   │ │  POD    │ │   POD    │               │
│  │ API-1  │ │ API-2  │ │ API-3   │ │Worker(bg)│ x2             │
│  │(Replica│ │(Replica│ │(Replica │ │Instances │               │
│  │  Set)  │ │  Set)  │ │  Set)   │ │          │               │
│  └────────┘ └────────┘ └─────────┘ └──────────┘               │
│       │       │         │                                       │
│       └───────┼─────────┘                                       │
│               │                                                 │
│       ┌───────┴──────────┬─────────┐                            │
│       │                  │         │                            │
│       ▼                  ▼         ▼                            │
│  ┌─────────────┐   ┌─────────┐  ┌──────┐                      │
│  │  StatefulSet    │   │Service │  │Secret│                      │
│  │PostgreSQL   │   │ Redis   │  │ConfigMap                    │
│  │(Primary+Rep)│   │(Cluster)│  │Secrets(pwd, keys)           │
│  └─────────────┘   └─────────┘  └──────┘                      │
│                                                                  │
│  ┌──────────────────────────────────────────────────────── ┐   │
│  │  PERSISTENT VOLUMES (Storage)                           │   │
│  │  ├─ PostgreSQL data (/var/lib/postgresql)              │   │
│  │  ├─ Application files (/uploads)                       │   │
│  │  └─ Backups (/backups)                                 │   │
│  └──────────────────────────────────────────────────────── ┘   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘

SERVICIOS EXTERNOS:
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  S3/MinIO    │  │ Email Service│  │ CDN (Cloudflare)│
│  File Storage│  │ (SendGrid)   │  │ Static Assets   │
└──────────────┘  └──────────────┘  └──────────────┘
```

## 9. DIAGRAMA DE INTEGRACIÓN CON SERVICIOS EXTERNOS

```
┌────────────────────────────────────────────────────────┐
│           SISTEMA DE CONFERENCIAS                      │
├────────────────────────────────────────────────────────┤
│                    │                                   │
│   ┌────────────────┼────────────────┐                 │
│   │                │                │                 │
│   ▼                ▼                ▼                 │
│
│  ┌──────────────────┐  ┌──────────────────┐           │
│  │ Email Provider   │  │ Cloud Storage    │           │
│  │  (SendGrid/AWS   │  │  (S3 / MinIO)    │           │
│  │   SES/Gmail)     │  │                  │           │
│  └──────────────────┘  └──────────────────┘           │
│       │                       │                       │
│       │ - Notifications       │ - Paper Files         │
│       │ - Reminders           │ - Version History     │
│       │ - Confirmations       │ - Generated Reports   │
│       │                       │ - Backups             │
│       │                       │                       │
│
│  ┌──────────────────┐  ┌──────────────────┐           │
│  │ Analytics Mix    │  │ Monitoring &     │           │
│  │  (Mixpanel/GA)   │  │ Logging          │           │
│  │                  │  │ (Datadog/ELK)    │           │
│  └──────────────────┘  └──────────────────┘           │
│       │                       │                       │
│       │ - User Events         │ - Performance Metrics │
│       │ - Usage Patterns      │ - Error Logs          │
│       │ - Funnel Analysis     │ - Audit Trail         │
│       │                       │ - Alerts              │
│       │                       │                       │
│
│  ┌──────────────────┐  ┌──────────────────┐           │
│  │ Authentication   │  │ CDN / Cache      │           │
│  │  (Auth0/Okta)    │  │  (Cloudflare)    │           │
│  │                  │  │                  │           │
│  └──────────────────┘  └──────────────────┘           │
│       │                       │                       │
│       │ - OAuth               │ - Static Assets       │
│       │ - SAML                │ - DDoS Protection     │
│       │ - 2FA                 │ - Edge Caching        │
│       │                       │                       │
└────────────────────────────────────────────────────────┘
```

## 10. MATRIZ DE COMPONENTES Y RESPONSABILIDADES

```
┌─────────────────────┬──────────────┬──────────────┬──────────────┐
│ COMPONENTE          │ RESPONSABLE  │ TECNOLOGÍA   │ ESCALABILIDAD│
├─────────────────────┼──────────────┼──────────────┼──────────────┤
│ API Gateway         │ DevOps       │ nginx/HAProxy│ Horizontal   │
├─────────────────────┼──────────────┼──────────────┼──────────────┤
│ Backend API         │ Backend Dev  │ Python/Django│ Horizontal   │
├─────────────────────┼──────────────┼──────────────┼──────────────┤
│ Database (Primary)  │ DBA          │ PostgreSQL   │ Vertical     │
├─────────────────────┼──────────────┼──────────────┼──────────────┤
│ Database (Replica)  │ DBA          │ PostgreSQL   │ Read Scale   │
├─────────────────────┼──────────────┼──────────────┼──────────────┤
│ Cache Layer         │ DevOps       │ Redis        │ Horizontal   │
├─────────────────────┼──────────────┼──────────────┼──────────────┤
│ Message Queue       │ DevOps       │ RabbitMQ     │ Horizontal   │
├─────────────────────┼──────────────┼──────────────┼──────────────┤
│ File Storage        │ Cloud Ops    │ S3/MinIO     │ Unlimited    │
├─────────────────────┼──────────────┼──────────────┼──────────────┤
│ Frontend (SPA)      │ Frontend Dev │ React/Vue    │ CDN          │
├─────────────────────┼──────────────┼──────────────┼──────────────┤
│ Email Service       │ Ops          │ SendGrid/AWS │ Managed      │
├─────────────────────┼──────────────┼──────────────┼──────────────┤
│ Monitoring          │ DevOps       │ Prometheus   │ Managed      │
└─────────────────────┴──────────────┴──────────────┴──────────────┘
```
