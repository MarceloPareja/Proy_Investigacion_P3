# Diagramas de Arquitectura - Sistema de Gestión de Conferencias

## 1. Diagrama de Arquitectura General (C4 Model - Container)

```mermaid
flowchart TB
    subgraph Users["Actores del Sistema"]
        Author["Autor"]
        Reviewer["Revisor"]
        Admin["Administrador"]
        Chair["Chair/Organizador"]
    end

    subgraph Web["Capa Web"]
        Browser["Navegador Web"]
        Mobile["App Móvil"]
    end

    subgraph Frontend["Frontend Application"]
        UI["React/Next.js Frontend"]
        Auth["Módulo de Autenticación"]
        Dashboard["Dashboard"]
    end

    subgraph API["Backend API - Django/FastAPI"]
        REST["REST API"]
        GraphQL["GraphQL Endpoint"]
        WS["WebSocket Server"]
        
        subgraph Services["Microservicios"]
            AuthSvc["Servicio Auth"]
            PaperSvc["Servicio Papers"]
            ReviewSvc["Servicio Reviews"]
            NotifSvc["Servicio Notificaciones"]
            ProgramSvc["Servicio Programa"]
        end
    end

    subgraph Infrastructure["Infraestructura"]
        LB["Load Balancer"]
        Cache["Redis Cache"]
        MQ["Message Queue"]
        
        subgraph Storage["Almacenamiento"]
            DB["PostgreSQL"]
            S3["S3/MinIO"]
            FS["File System"]
        end
        
        subgraph External["Servicios Externos"]
            Email["SMTP/SendGrid"]
            Antivirus["ClamAV"]
            PDF["PDF Processor"]
        end
    end

    %% Conexiones
    Author --> Browser
    Reviewer --> Browser
    Admin --> Browser
    Chair --> Mobile
    Chair --> Browser

    Browser --> UI
    Mobile --> UI

    UI --> Auth
    UI --> Dashboard
    Dashboard --> REST
    UI --> GraphQL
    Dashboard --> WS

    REST --> LB
    GraphQL --> LB
    WS --> LB

    LB --> AuthSvc
    LB --> PaperSvc
    LB --> ReviewSvc
    LB --> NotifSvc
    LB --> ProgramSvc

    AuthSvc --> Cache
    AuthSvc --> DB
    
    PaperSvc --> DB
    PaperSvc --> S3
    PaperSvc --> Antivirus
    PaperSvc --> MQ
    
    ReviewSvc --> DB
    ReviewSvc --> MQ
    
    NotifSvc --> MQ
    NotifSvc --> Email
    
    ProgramSvc --> DB
    ProgramSvc --> S3

    MQ --> PDF
```

---

## 2. Diagrama de Arquitectura de Capas (Layered Architecture)

```mermaid
flowchart TB
    subgraph Presentation["PRESENTATION LAYER"]
        Web["Django Views"]
        API["REST API Views"]
        WS["WebSocket Handlers"]
        Templates["HTML Templates"]
    end

    subgraph Business["BUSINESS LOGIC LAYER"]
        AuthBL["Authentication Service"]
        PaperBL["Paper Management"]
        ReviewBL["Review Workflow"]
        SelectionBL["Paper Selection"]
        ProgramBL["Program Builder"]
        NotifBL["Notification Service"]
    end

    subgraph Application["APPLICATION LAYER"]
        PaperRepo["Paper Repository"]
        ReviewRepo["Review Repository"]
        UserRepo["User Repository"]
        ConferenceRepo["Conference Repository"]
        NotifRepo["Notification Repository"]
    end

    subgraph Data["DATA LAYER"]
        PostgreSQL["PostgreSQL\n(Primary DB)"]
        Redis["Redis\n(Cache)"]
        S3["S3/MinIO\n(Files)"]
    end

    %% Flujo de datos
    Web --> AuthBL
    API --> AuthBL
    WS --> AuthBL

    AuthBL --> PaperBL
    PaperBL --> ReviewBL
    ReviewBL --> SelectionBL
    SelectionBL --> ProgramBL
    ProgramBL --> NotifBL

    PaperBL --> PaperRepo
    ReviewBL --> ReviewRepo
    AuthBL --> UserRepo
    SelectionBL --> ConferenceRepo
    NotifBL --> NotifRepo

    PaperRepo --> PostgreSQL
    ReviewRepo --> PostgreSQL
    UserRepo --> PostgreSQL
    ConferenceRepo --> PostgreSQL
    NotifRepo --> PostgreSQL

    AuthBL --> Redis
    PaperRepo --> S3
    ReviewRepo --> S3
```

---

## 3. Diagrama de Componentes del Sistema

```mermaid
flowchart TB
    subgraph Client["Client Layer"]
        WebBrowser["Navegador Web"]
        MobileApp["Aplicación Móvil"]
    end

    subgraph API["API Gateway / Load Balancer"]
        Nginx["Nginx"]
        AuthMiddleware["Auth Middleware"]
    end

    subgraph Core["Core Services"]
        subgraph Paper["Paper Management"]
            Submit["Submission Handler"]
            Validator["Document Validator"]
            Indexer["Text Indexer"]
            Storage["File Storage"]
        end

        subgraph Review["Review System"]
            Assigner["Assignment Engine"]
            Collector["Review Collector"]
            Calculator["Score Calculator"]
        end

        subgraph Selection["Selection Process"]
            Evaluator["Paper Evaluator"]
            Selector["Paper Selector"]
            Notifier["Author Notifier"]
        end

        subgraph Program["Program Builder"]
            Scheduler["Session Scheduler"]
            Exporter["Program Exporter"]
        end
    end

    subgraph Data["Data Layer"]
        Postgres["PostgreSQL\n- Users\n- Papers\n- Reviews\n- Conferences"]
        Redis["Redis\n- Sessions\n- Cache"]
        S3["S3/MinIO\n- PDF Documents\n- Versions"]
    end

    %% Conexiones
    WebBrowser --> Nginx
    MobileApp --> Nginx

    Nginx --> AuthMiddleware
    AuthMiddleware --> Submit
    AuthMiddleware --> Assigner
    AuthMiddleware --> Evaluator

    Submit --> Validator
    Validator --> Indexer
    Indexer --> Storage
    Storage --> S3

    Submit --> Postgres
    Assigner --> Postgres
    Collector --> Postgres
    Calculator --> Postgres
    Evaluator --> Postgres
    Selector --> Postgres
    Notifier --> Postgres
    Scheduler --> Postgres

    Submit --> Redis
    Assigner --> Redis
    Calculator --> Redis

    Collector --> Notifier
    Calculator --> Evaluator
    Evaluator --> Selector
    Selector --> Notifier
    Notifier --> Scheduler
    Scheduler --> Exporter
```

---

## 4. Diagrama de Arquitectura de Base de Datos

```mermaid
erDiagram
    USERS ||--o{ PAPERS : "submits"
    USERS ||--o{ REVIEWS : "writes"
    CONFERENCES ||--o{ PAPERS : "contains"
    CONFERENCES ||--o{ REVIEWERS : "assigns"
    PAPERS ||--o{ REVIEWS : "receives"
    PAPERS ||--o{ VERSIONS : "has"
    REVIEWERS ||--o{ REVIEW_ASSIGNMENTS : "receives"
    PAPERS ||--o{ REVIEW_ASSIGNMENTS : "assigned_to"
    SESSIONS ||--o{ PAPERS : "includes"
    
    USERS {
        uuid id PK
        string email
        string password_hash
        string full_name
        string affiliation
        enum role
        datetime created_at
        boolean active
    }
    
    CONFERENCES {
        uuid id PK
        string name
        string description
        datetime call_for_papers_date
        datetime submission_deadline
        datetime review_deadline
        datetime notification_date
        datetime final_program_date
        enum status
        uuid created_by
    }
    
    PAPERS {
        uuid id PK
        uuid conference_id FK
        string title
        text abstract
        string document_url
        uuid primary_author_id FK
        enum status
        decimal average_score
        datetime submitted_at
        datetime updated_at
    }
    
    REVIEWS {
        uuid id PK
        uuid paper_id FK
        uuid reviewer_id FK
        decimal originality_score
        decimal relevance_score
        decimal methodology_score
        decimal clarity_score
        decimal contribution_score
        text comments
        text confidential_comments
        enum recommendation
        enum status
        datetime submitted_at
    }
    
    REVIEW_ASSIGNMENTS {
        uuid id PK
        uuid paper_id FK
        uuid reviewer_id FK
        datetime assigned_date
        datetime due_date
        enum status
        boolean accepted
    }
    
    VERSIONS {
        uuid id PK
        uuid paper_id FK
        string version_number
        string document_url
        string change_notes
        datetime uploaded_at
    }
    
    SESSIONS {
        uuid id PK
        uuid conference_id FK
        string title
        datetime start_time
        datetime end_time
        string room
        enum session_type
    }
```

---

## 5. Diagrama de Flujo de Estados (State Machine)

```mermaid
stateDiagram-v2
    [*] --> DRAFT: Iniciar conference
    
    DRAFT --> OPEN: Publicar call for papers
    
    OPEN --> SUBMITTED: Autor sube paper
    OPEN --> CLOSED: Pasada fecha límite
    
    SUBMITTED --> UNDER_REVIEW: Asignar revisores
    SUBMITTED --> WITHDRAWN: Autor retira
    
    UNDER_REVIEW --> COMPLETED_ALL_REVIEWS: 3 reviews completados
    COMPLETED_ALL_REVIEWS --> SELECTED: Score >= umbral
    COMPLETED_ALL_REVIEWS --> REJECTED: Score < umbral
    COMPLETED_ALL_REVIEWS --> REVISION_REQUIRED: Requiere cambios
    
    REVISION_REQUIRED --> RESUBMITTED: Autor sube versión corregida
    RESUBMITTED --> UNDER_REVIEW: Re-evaluación
    
    SELECTED --> FINAL: Administrador approve
    REJECTED --> FINAL: Notificación enviada
    
    FINAL --> [*]: Programa publicado
    
    WITHDRAWN --> [*]: Paper eliminado
    
    note right of UNDER_REVIEW
        Cada paper requiere
        exactamente 3 revisores
    end note
    
    note right of COMPLETED_ALL_REVIEWS
        promedio = (originality + relevance + 
                    methodology + clarity + 
                    contribution) / 5
    end note
```

---

## 6. Diagrama de Flujo de Procesos (Business Process)

```mermaid
flowchart TD
    Start([Inicio del Proceso])
    
    subgraph CFP["Call for Papers"]
        A1[Crear conferencia]
        A2[Configurar fechas]
        A3[Publicar CFP]
    end
    
    subgraph Submission["Submisión de Papers"]
        B1[Autor registra datos]
        B2[Sube documento PDF]
        B3[Valida formato]
        B4[Registra submission]
        B5[Envía confirmación]
    end
    
    subgraph Assignment["Asignación de Revisores"]
        C1[Identificar papers pendientes]
        C2[Seleccionar revisores por expertise]
        C3[Asignar 3 revisores/paper]
        C4[Notificar revisores]
    end
    
    subgraph Review["Proceso de Revisión"]
        D1[Revisor acepta asignación]
        D2[Descarga paper]
        D3[Completa evaluación]
        D4[Envía review]
        D5[Actualizar score promedio]
    end
    
    subgraph Selection["Selección Final"]
        E1[Calcular scores finales]
        E2[Aplicar criterios selección]
        E3[Revisión administrativa]
        E4[Notificar resultados]
    end
    
    subgraph Final["Versión Final"]
        F1[Autores con revisiones suben versiones]
        F2[Admin aprueba versiones finales]
        F3[Generar programa]
        F4([Fin del proceso])
    end
    
    Start --> A1
    A1 --> A2
    A2 --> A3
    A3 --> B1
    
    B1 --> B2
    B2 --> B3
    B3 --> B4
    B4 --> B5
    B5 --> C1
    
    C1 --> C2
    C2 --> C3
    C3 --> C4
    C4 --> D1
    
    D1 --> D2
    D2 --> D3
    D3 --> D4
    D4 --> D5
    D5 --> E1
    
    E1 --> E2
    E2 --> E3
    E3 --> E4
    E4 --> F1
    
    F1 --> F2
    F2 --> F3
    F3 --> F4
```

---

## 7. Diagrama de Despliegue (Deployment)

```mermaid
flowchart TB
    subgraph Users["Usuarios"]
        Client["Navegador Web / Mobile"]
    end

    subgraph Cloud["Cloud Infrastructure"]
        subgraph CDN["CDN / WAF"]
            CloudFlare["CloudFlare"]
        end
        
        subgraph LB["Load Balancing"]
            ALB["Application Load Balancer"]
        end
        
        subgraph K8s["Kubernetes Cluster"]
            subgraph AppPods["Application Pods"]
                Frontend["Frontend Pod\n(React/Next.js)"]
                Backend["Backend Pods\n(Django x3)"]
                Worker["Worker Pods\n(Celery)"]
            end
            
            subgraph Services["Services"]
                FrontendSvc["Frontend Service"]
                BackendSvc["Backend Service"]
                WorkerSvc["Worker Service"]
            end
            
            subgraph Storage["Storage"]
                PV["Persistent Volume"]
            end
        end
        
        subgraph Data["Data Layer"]
            subgraph DB["PostgreSQL HA"]
                PrimaryDB["Primary DB"]
                ReplicaDB1["Read Replica 1"]
                ReplicaDB2["Read Replica 2"]
            end
            
            RedisCluster["Redis Cluster"]
        end
        
        subgraph ObjectStorage["Object Storage"]
            S3MinIO["S3/MinIO Bucket"]
        end
        
        subgraph Queue["Message Queue"]
            RabbitMQ["RabbitMQ"]
        end
        
        subgraph External["External Services"]
            SMTP["SMTP Service"]
            Antivirus["ClamAV Service"]
        end
    end
    
    %% Conexiones
    Client --> CloudFlare
    CloudFlare --> ALB
    ALB --> FrontendSvc
    ALB --> BackendSvc
    
    FrontendSvc --> Frontend
    BackendSvc --> Backend
    WorkerSvc --> Worker
    
    Frontend --> Backend
    
    Backend --> PrimaryDB
    Backend --> RedisCluster
    Backend --> S3MinIO
    Backend --> RabbitMQ
    
    Worker --> RabbitMQ
    Worker --> SMTP
    Worker --> Antivirus
    Worker --> S3MinIO
    
    PrimaryDB --> ReplicaDB1
    PrimaryDB --> ReplicaDB2
    
    RedisCluster --> PV
```

---

## 8. Diagrama de Secuencia - Submisión de Paper

```mermaid
sequenceDiagram
    participant Author
    participant Frontend
    participant Backend
    participant Database
    participant Storage
    participant Antivirus
    participant Email

    Author->>Frontend: Subir paper (metadata + PDF)
    
    Frontend->>Backend: POST /api/papers (multipart)
    
    Backend->>Backend: Validar archivo (tipo, tamaño)
    
    Backend->>Antivirus: Escanear documento
    
    alt Archivo infectado
        Antivirus-->>Backend: Virus detectado
        Backend-->>Frontend: Error 400: Archivo no válido
        Frontend-->>Author: Mostrar error
    else Archivo limpio
        Antivirus-->>Backend: Scan OK
        
        Backend->>Storage: Subir PDF a S3
        Storage-->>Backend: URL del documento
        
        Backend->>Database: Crear registro paper
        Database-->>Backend: Paper ID creado
        
        Backend->>Email: Enviar confirmación
        Email-->>Author: Email de confirmación
        
        Backend-->>Frontend: Paper creado (201)
        Frontend-->>Author: Confirmación visual
    end
```

---

## 9. Diagrama de Secuencia - Asignación de Revisores

```mermaid
sequenceDiagram
    participant Admin
    participant Backend
    participant Database
    participant Reviewer1
    participant Reviewer2
    participant Reviewer3
    participant Email

    Admin->>Backend: Iniciar asignación (conference_id)
    
    Backend->>Database: Obtener papers sin asignar
    Database-->>Backend: Lista de papers
    
    Backend->>Database: Obtener revisores disponibles
    Database-->>Backend: Lista de revisores con expertise
    
    loop Para cada paper
        Backend->>Backend: Seleccionar 3 revisores<br/>(algoritmo balanceado)
        
        Backend->>Database: Crear ReviewAssignments
        Database-->>Backend: Assignments creados
        
        Backend->>Email: Notificar Reviewer1
        Email-->>Reviewer1: Email con asignación
        
        Backend->>Email: Notificar Reviewer2
        Email-->>Reviewer2: Email con asignación
        
        Backend->>Email: Notificar Reviewer3
        Email-->>Reviewer3: Email con asignación
    end
    
    Backend-->>Admin: Asignación completada
```

---

## 10. Diagrama de Componentes - Patrón Observer para Notificaciones

```mermaid
flowchart TB
    subgraph Subject["Subject (Paper/Observable)"]
        PaperSubject["Paper Service"]
        Events["Event Bus"]
    end

    subgraph Observers["Observers (Subscribers)"]
        EmailNotif["Email Notification Service"]
        Webhook["Webhook Dispatcher"]
        Audit["Audit Logger"]
        Dashboard["Dashboard Updater"]
    end

    subgraph Events["Event Types"]
        PaperSubmitted["paper.submitted"]
        ReviewAssigned["review.assigned"]
        ReviewCompleted["review.completed"]
        PaperSelected["paper.selected"]
        PaperRejected["paper.rejected"]
        FinalVersion["paper.final_version"]
    end

    %% Flujo
    PaperSubject --> Events: Publica evento
    
    Events --> EmailNotif: Suscrito a todos
    Events --> Webhook: Suscrito a eventos relevantes
    Events --> Audit: Todos los eventos
    Events --> Dashboard: Actualizaciones de estado
    
    EmailNotif --> Author: Email
    EmailNotif --> Reviewer: Email
    
    Webhook --> ExternalSystems: HTTP callbacks
    
    Audit --> Database: Guardar logs
    
    Dashboard --> Frontend: WebSocket update
