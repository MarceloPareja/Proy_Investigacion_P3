# Diagramas de Arquitectura - Sistema de Calendario Multiplataforma

## 1. Diagrama de Arquitectura General del Sistema

```mermaid
flowchart TB
    subgraph Clients["Clientes"]
        Web["Web App\n(React)"]
        iOS["iOS App\n(Swift/React Native)"]
        Android["Android App\n(Kotlin/React Native)"]
        Desktop["Desktop App\n(Electron)"]
        ThirdParty["API Externa"]
    end

    subgraph Gateway["API Gateway"]
        Kong["Kong API Gateway"]
        AuthGW["Auth Middleware"]
        RateLimiter["Rate Limiter"]
        SSL["SSL Termination"]
    end

    subgraph Services["Microservicios"]
        subgraph Core["Servicios Core"]
            AuthSVC["Auth Service\n(Login, OAuth, Sessions)"]
            UserSVC["User Service\n(Profiles, Preferences)"]
            CalendarSVC["Calendar Service\n(CRUD, Encryption)"]
            EventSVC["Event Service\n(Events, Reminders)"]
        end
        
        subgraph Sharing["Servicios de Colaboración"]
            ShareSVC["Sharing Service\n(Permissions, Access)"]
            SyncSVC["Sync Service\n(Real-time, Conflict)"]
        end
        
        subgraph Notifications["Servicios de Notificación"]
            AlertSVC["Alert Service\n(Scheduling, Triggers)"]
            PushSVC["Push Service\n(FCM, APNs)"]
            EmailSVC["Email Service\n(Templates, SendGrid)"]
        end
    end

    subgraph Data["Capa de Datos"]
        subgraph Storage["Persistencia"]
            Postgres[(PostgreSQL\nUsers, Calendars,\nEvents, Alerts)]
            Redis[(Redis\nSessions, Cache,\nPub/Sub)]
            S3[("S3/MinIO\nEncrypted\nAttachments")]
            Vault[("Vault/KMS\nEncryption Keys")]
        end
        
        subgraph MessageBus["Message Queue"]
            Kafka["Apache Kafka"]
        end
    end

    %% Conexiones
    Web --> Kong
    iOS --> Kong
    Android --> Kong
    Desktop --> Kong
    ThirdParty --> Kong
    
    Kong --> SSL
    SSL --> AuthGW
    AuthGW --> RateLimiter
    RateLimiter --> AuthSVC
    RateLimiter --> UserSVC
    RateLimiter --> CalendarSVC
    RateLimiter --> EventSVC
    RateLimiter --> SyncSVC
    RateLimiter --> AlertSVC
    
    AuthSVC --> Redis
    AuthSVC --> Postgres
    
    UserSVC --> Postgres
    
    CalendarSVC --> Postgres
    CalendarSVC --> S3
    CalendarSVC --> Vault
    
    EventSVC --> Postgres
    EventSVC --> Kafka
    
    SyncSVC --> Redis
    SyncSVC --> Kafka
    
    AlertSVC --> Kafka
    AlertSVC --> Redis
    
    PushSVC --> AlertSVC
    EmailSVC --> AlertSVC
```

---

## 2. Diagrama de Arquitectura de Capas

```mermaid
flowchart TB
    subgraph Presentation["PRESENTATION LAYER"]
        WebViews["React Components"]
        MobileViews["Native Views"]
        API["REST/GraphQL API"]
        WebSocketAPI["WebSocket API"]
    end

    subgraph Application["APPLICATION LAYER"]
        AuthApp["Auth Controller"]
        CalendarApp["Calendar Controller"]
        EventApp["Event Controller"]
        AlertApp["Alert Controller"]
        SyncApp["Sync Controller"]
    end

    subgraph Domain["DOMAIN LAYER"]
        CalendarDomain["Calendar Domain\n- Calendar Entity\n- Calendar Service\n- Calendar Factory"]
        EventDomain["Event Domain\n- Event Entity\n- Recurrence Rules\n- Event Service"]
        UserDomain["User Domain\n- User Entity\n- Preferences\n- Permissions"]
        AlertDomain["Alert Domain\n- Alert Entity\n- Notification Rules"]
    end

    subgraph Infrastructure["INFRASTRUCTURE LAYER"]
        %% Repositories
        CalendarRepo["Calendar Repository\n(Encrypted)"]
        EventRepo["Event Repository"]
        UserRepo["User Repository"]
        AlertRepo["Alert Repository"]
        
        %% External Services
        EncryptionSVC["Encryption Service\n(AES-256, KMS)"]
        PushSVC["Push Notification Service"]
        EmailSVC["Email Service"]
        StorageSVC["Cloud Storage Service"]
        
        %% Infrastructure
        Kafka["Kafka Producer/Consumer"]
        RedisCache["Redis Cache"]
    end

    subgraph Data["DATA LAYER"]
        PostgreSQL["PostgreSQL"]
        Redis["Redis"]
        S3["S3 Bucket"]
        Vault["Vault/KMS"]
    end

    %% Flujo de datos
    WebViews --> API
    MobileViews --> API
    API --> WebSocketAPI
    
    API --> AuthApp
    API --> CalendarApp
    API --> EventApp
    API --> AlertApp
    API --> SyncApp
    
    WebSocketAPI --> SyncApp
    
    AuthApp --> UserDomain
    CalendarApp --> CalendarDomain
    EventApp --> EventDomain
    AlertApp --> AlertDomain
    
    CalendarDomain --> CalendarRepo
    EventDomain --> EventRepo
    UserDomain --> UserRepo
    AlertDomain --> AlertRepo
    
    CalendarRepo --> EncryptionSVC
    EventRepo --> Kafka
    AlertRepo --> Kafka
    
    EncryptionSVC --> Vault
    Kafka --> PushSVC
    Kafka --> EmailSVC
    
    CalendarRepo --> PostgreSQL
    EventRepo --> PostgreSQL
    UserRepo --> PostgreSQL
    AlertRepo --> PostgreSQL
    
    Kafka --> RedisCache
    SyncApp --> RedisCache
```

---

## 3. Diagrama de Componentes Detallado

```mermaid
flowchart TB
    subgraph Client["Client Applications"]
        WebClient["Web Client\n(React)"]
        MobileClient["Mobile Client\n(React Native)"]
        DesktopClient["Desktop Client\n(Electron)"]
    end

    subgraph API["API Layer"]
        REST["REST API Endpoints"]
        GraphQL["GraphQL Endpoint"]
        WS["WebSocket Handler"]
        
        subgraph Middleware["Middleware Stack"]
            AuthMid["Authentication"]
            RateMid["Rate Limiting"]
            ValidationMid["Validation"]
            LoggingMid["Logging"]
        end
    end

    subgraph Services["Business Logic Services"]
        subgraph AuthModule["Authentication Module"]
            Login["Login Handler"]
            Register["Registration"]
            OAuth["OAuth 2.0 Provider"]
            Session["Session Manager"]
            JWT["JWT Token Generator"]
        end
        
        subgraph CalendarModule["Calendar Module"]
            CRUD["Calendar CRUD"]
            Encrypt["Encryption Handler"]
            Share["Sharing Manager"]
            Version["Version Control"]
        end
        
        subgraph EventModule["Event Module"]
            EventCRUD["Event CRUD"]
            Recurrence["Recurrence Engine"]
            Attendee["Attendee Manager"]
            Reminder["Reminder Service"]
        end
        
        subgraph AlertModule["Alert Module"]
            Scheduler["Alert Scheduler"]
            Trigger["Alert Trigger"]
            ChannelRouter["Channel Router"]
            Template["Message Template"]
        end
        
        subgraph SyncModule["Sync Module"]
            Realtime["Real-time Sync"]
            Conflict["Conflict Resolver"]
            Offline["Offline Support"]
        end
    end

    subgraph DataAccess["Data Access Layer"]
        %% Repositories
        CalendarDB["Calendar Repository"]
        EventDB["Event Repository"]
        UserDB["User Repository"]
        AlertDB["Alert Repository"]
        SessionDB["Session Repository"]
        
        %% Cache
        Cache["Redis Cache Layer"]
        
        %% Storage
        FileStore["File Storage\n(S3)"]
        KeyStore["Key Storage\n(Vault)"]
    end

    %% Conexiones
    WebClient --> REST
    WebClient --> WS
    MobileClient --> REST
    MobileClient --> GraphQL
    DesktopClient --> REST
    
    REST --> Middleware
    GraphQL --> Middleware
    WS --> Middleware
    
    Middleware --> AuthModule
    Middleware --> CalendarModule
    Middleware --> EventModule
    Middleware --> AlertModule
    Middleware --> SyncModule
    
    AuthModule --> SessionDB
    AuthModule --> Cache
    
    CalendarModule --> CalendarDB
    CalendarModule --> Encrypt
    CalendarModule --> FileStore
    CalendarModule --> KeyStore
    
    EventModule --> EventDB
    EventModule --> Recurrence
    EventModule --> Attendee
    
    AlertModule --> AlertDB
    AlertModule --> Scheduler
    AlertModule --> ChannelRouter
    
    SyncModule --> Realtime
    SyncModule --> Conflict
    SyncModule --> Cache
    
    CalendarDB --> Cache
    EventDB --> Cache
    AlertDB --> Cache
```

---

## 4. Diagrama de Base de Datos (ER)

```mermaid
erDiagram
    USERS ||--o{ CALENDARS : owns
    USERS ||--o{ DEVICES : registers
    USERS ||--o{ ALERTS : configures
    CALENDARS ||--o{ EVENTS : contains
    CALENDARS ||--o{ SHARES : shared_with
    EVENTS ||--o{ REMINDERS : has
    EVENTS ||--o{ RECURRENCE : follows
    EVENTS ||--o{ ATTENDEES : includes
    CALENDARS ||--o{ ENCRYPTION_KEYS : encrypted_by
    
    USERS {
        uuid id PK
        string email
        string password_hash
        string first_name
        string last_name
        jsonb preferences
        datetime created_at
        datetime last_login
        boolean active
        enum timezone
    }
    
    CALENDARS {
        uuid id PK
        uuid user_id FK
        string name
        string description
        string color
        boolean is_default
        boolean is_encrypted
        uuid encryption_key_id
        datetime created_at
        datetime updated_at
        jsonb settings
    }
    
    ENCRYPTION_KEYS {
        uuid id PK
        uuid user_id FK
        string key_version
        string encrypted_dek
        string key_id
        datetime created_at
        datetime rotated_at
    }
    
    SHARES {
        uuid id PK
        uuid calendar_id FK
        uuid shared_with_user_id FK
        enum permission
        uuid shared_by_user_id FK
        datetime created_at
    }
    
    EVENTS {
        uuid id PK
        uuid calendar_id FK
        string title
        text description
        string location
        datetime start_time
        datetime end_time
        boolean is_all_day
        string timezone
        enum status
        string color
        datetime created_at
        datetime updated_at
    }
    
    RECURRENCE {
        uuid id PK
        uuid event_id FK
        enum frequency
        integer interval
        datetime until
        integer count
        jsonb days_of_week
        jsonb exceptions
    }
    
    REMINDERS {
        uuid id PK
        uuid event_id FK
        integer minutes_before
        enum reminder_type
        boolean was_sent
        datetime sent_at
    }
    
    ATTENDEES {
        uuid id PK
        uuid event_id FK
        string email
        string name
        enum response_status
        boolean is_organizer
    }
    
    DEVICES {
        uuid id PK
        uuid user_id FK
        string device_token
        enum platform
        string app_version
        datetime last_active
        boolean push_enabled
    }
    
    ALERTS {
        uuid id PK
        uuid user_id FK
        uuid event_id FK
        enum alert_type
        datetime trigger_time
        string message
        boolean is_active
        boolean was_sent
        jsonb metadata
    }
```

---

## 5. Diagrama de Estados (Event State Machine)

```mermaid
stateDiagram-v2
    [*] --> DRAFT: Create event
    
    DRAFT --> CONFIRMED: Confirm event
    DRAFT --> TENTATIVE: Set tentative
    DRAFT --> CANCELLED: Delete event
    
    CONFIRMED --> TENTATIVE: Change to tentative
    CONFIRMED --> CANCELLED: Cancel event
    CONFIRMED --> UPDATED: Modify event
    CONFIRMED --> COMPLETED: Event passed
    
    TENTATIVE --> CONFIRMED: Accept invitation
    TENTATIVE --> CANCELLED: Decline invitation
    
    UPDATED --> CONFIRMED: Save changes
    
    CANCELLED --> [*]
    COMPLETED --> [*]
    
    note right of DRAFT
        Initial state
        Not visible to attendees yet
    end note
    
    note right of CONFIRMED
        Visible to all attendees
        Reminders are active
    end note
    
    note right of CANCELLED
        Still visible with
        "cancelled" status
        Previous versions preserved
    end note
```

---

## 6. Diagrama de Flujo de Autenticación

```mermaid
sequenceDiagram
    participant User
    participant Client
    participant API
    participant AuthSvc
    participant Redis
    participant KeyVault
    
    User->>Client: Ingresar credenciales
    
    Client->>API: POST /auth/login (email, password)
    API->>AuthSvc: Validar credenciales
    AuthSvc->>Redis: Verificar cuenta no bloqueada
    
    alt Credenciales válidas
        AuthSvc-->>API: Usuario validado
        API->>KeyVault: Obtener/clave de encriptación del usuario
        KeyVault-->>API: Clave de usuario (cifrada)
        
        API->>Redis: Crear session
        Redis-->>API: Session ID
        
        API->>API: Generar JWT tokens
        API-->>Client: { access_token, refresh_token, user_data }
        
        Client-->>User: Login exitoso
    else Credenciales inválidas
        AuthSvc-->>API: Error de autenticación
        API-->>Client: 401 Unauthorized
        Client-->>User: Mostrar error
    end
    
    Note over User,KeyVault: Flujo de acceso a calendario encriptado
    
    User->>Client: Solicitar calendario
    
    Client->>API: GET /calendars/:id (with JWT)
    API->>AuthSvc: Validar JWT
    AuthSvc-->>API: Token válido
    
    API->>API: Verificar permisos del usuario
    API->>KeyVault: Solicitar clave de calendario
    KeyVault-->>API: Clave desencriptada
    
    API->>API: Desencriptar datos del calendario
    API-->>Client: Calendario (datos desencriptados)
    Client-->>User: Mostrar calendario
```

---

## 7. Diagrama de Flujo de Notificaciones

```mermaid
sequenceDiagram
    participant Event
    participant AlertSvc
    participant Kafka
    participant Worker
    participant PushSvc
    participant EmailSvc
    participant User
    
    Event->>AlertSvc: Evento creado/actualizado
    AlertSvc->>AlertSvc: Crear alertas según preferencias
    
    loop Para cada alerta configurada
        AlertSvc->>AlertSvc: Calcular trigger_time
        AlertSvc->>Kafka: Publicar alerta programada
    end
    
    Kafka->>Worker: Consumir mensaje de alerta
    
    Worker->>Worker: Verificar trigger_time alcanzado
    
    alt Tiempo de trigger alcanzado
        Worker->>Worker: Obtener detalles del evento
        Worker->>Worker: Seleccionar canal preferido
        Worker->>Worker: Generar mensaje
        
        alt Canal