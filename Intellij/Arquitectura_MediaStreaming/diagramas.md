# Diagramas de Arquitectura - Sistema de Reproducción de Multimedia Online

## 1. Diagrama de Arquitectura General del Sistema

```mermaid
flowchart TB
    subgraph Clients["Aplicaciones Cliente"]
        Desktop["Desktop App\n(Windows/Mac)"]
        Web["Web Browser\n(React)"]
        Mobile["Mobile App\n(iOS/Android)"]
    end

    subgraph ClientLayer["Capa del Cliente"]
        Player["Media Player\nEngine"]
        Cache["Local Cache"]
        DRM["DRM Module"]
        Sync["Sync Manager"]
    end

    subgraph Gateway["API Gateway"]
        Kong["Kong API Gateway"]
        Auth["Auth Middleware"]
        RateLim["Rate Limiter"]
    end

    subgraph AppS["Application Server (AppS)"]
        subgraph CoreServices["Core Services"]
            MediaCat["Media Catalog\nService"]
            Store["Store Service"]
            Library["User Library\nService"]
            Purchase["Purchase\nService"]
        end
        
        subgraph StreamServices["Streaming Services"]
            Stream["Streaming\nCoordinator"]
            Transcode["Transcode\nService"]
            Download["Download\nService"]
        end
        
        subgraph SyncServices["Sync Services"]
            CloudSync["Cloud Sync\nService"]
            USBSync["USB Sync\nService"]
            Conflict["Conflict\nResolution"]
        end
    end

    subgraph MediaServers["Media Servers"]
        subgraph VideoServer["Video Server (VS)"]
            NginxRTMP["Nginx RTMP/HLS"]
            Origin["Origin Server"]
            Segmenter["Segmenter"]
        end
        
        subgraph AudioServer["Audio Server (AS)"]
            Icecast["Icecast HLS"]
            AudioStore["Audio Storage"]
        end
        
        subgraph TranscodeFarm["Transcode Farm"]
            GPUWorkers["GPU Workers\n(4K/HDR)"]
            CPUWorkers["CPU Workers\n(HD/SD)"]
            Queue["Job Queue"]
        end
    end

    subgraph Storage["Almacenamiento"]
        CDN["CDN\n(CloudFront)"]
        S3["S3 Bucket\n(Media Files)"]
        DB["PostgreSQL\n(Metadata)"]
        Redis["Redis\n(Cache)"]
    end

    %% Conexiones
    Desktop --> Player
    Web --> Player
    Mobile --> Player
    
    Player --> Cache
    Player --> DRM
    Player --> Sync
    
    Player --> Kong
    Kong --> Auth
    Auth --> RateLim
    RateLim --> MediaCat
    RateLim --> Store
    RateLim --> Library
    RateLim --> Stream
    
    MediaCat --> DB
    MediaCat --> Redis
    Store --> DB
    Library --> DB
    Stream --> Transcode
    Stream --> Download
    
    Transcode --> GPUWorkers
    Transcode --> CPUWorkers
    Transcode --> Queue
    
    Stream --> VideoServer
    Stream --> AudioServer
    
    VideoServer --> CDN
    AudioServer --> CDN
    CDN --> S3
    
    CloudSync --> CDN
    USBSync --> Sync
```

---

## 2. Diagrama de Arquitectura de Capas

```mermaid
flowchart TB
    subgraph Presentation["PRESENTATION LAYER"]
        DesktopApp["Desktop Application\n(Electron)"]
        WebApp["Web Application\n(React + video.js)"]
        MobileApp["Mobile Application\n(Native)"]
    end

    subgraph ClientLogic["CLIENT LOGIC LAYER"]
        PlayerEngine["Media Player Engine"]
        StreamHandler["Streaming Handler"]
        CacheManager["Cache Manager"]
        SyncEngine["Sync Engine"]
        DRMModule["DRM Module"]
    end

    subgraph API["API GATEWAY LAYER"]
        LoadBalancer["Load Balancer\n(Nginx)"]
        APIRouter["API Router"]
        AuthHandler["Authentication"]
        RateLimiter["Rate Limiting"]
        RequestValidator["Validation"]
    end

    subgraph Application["APPLICATION LAYER"]
        MediaCatalog["Media Catalog"]
        StoreService["Store Service"]
        LibraryService["Library Service"]
        SyncService["Sync Service"]
        StreamingService["Streaming Service"]
        DownloadService["Download Service"]
    end

    subgraph Domain["DOMAIN LAYER"]
        MediaDomain["Media Domain\n- Media Item\n- Transcode Job\n- Streaming Session"]
        UserDomain["User Domain\n- Library\n- Purchases\n- Devices"]
        StoreDomain["Store Domain\n- Cart\n- Orders\n- Payments"]
    end

    subgraph Infrastructure["INFRASTRUCTURE LAYER"]
        TranscodeInfra["Transcode Infrastructure\n- FFmpeg\n- GPU Cluster"]
        StorageInfra["Storage Infrastructure\n- S3\n- CDN"]
        CacheInfra["Cache Infrastructure\n- Redis\n- Local Cache"]
        DRMInfra["DRM Infrastructure\n- Widevine\n- FairPlay"]
    end

    %% Flujo
    DesktopApp --> PlayerEngine
    WebApp --> PlayerEngine
    MobileApp --> PlayerEngine
    
    PlayerEngine --> StreamHandler
    PlayerEngine --> CacheManager
    PlayerEngine --> SyncEngine
    PlayerEngine --> DRMModule
    
    StreamHandler --> LoadBalancer
    CacheManager --> LoadBalancer
    SyncEngine --> LoadBalancer
    DRMModule --> LoadBalancer
    
    LoadBalancer --> APIRouter
    APIRouter --> AuthHandler
    AuthHandler --> RateLimiter
    RateLimiter --> RequestValidator
    
    RequestValidator --> MediaCatalog
    RequestValidator --> StoreService
    RequestValidator --> LibraryService
    RequestValidator --> SyncService
    RequestValidator --> StreamingService
    
    MediaCatalog --> MediaDomain
    StoreService --> StoreDomain
    LibraryService --> UserDomain
    SyncService --> UserDomain
    StreamingService --> MediaDomain
    
    MediaCatalog --> TranscodeInfra
    MediaCatalog --> StorageInfra
    StorageInfra --> CacheInfra
    DRMModule --> DRMInfra
```

---

## 3. Diagrama de Componentes del Sistema

```mermaid
flowchart TB
    subgraph Client["Client Application"]
        Player["Media Player"]
        DownloadMgr["Download Manager"]
        LibraryMgr["Library Manager"]
        Settings["User Settings"]
    end

    subgraph API["API Layer"]
        REST["REST API"]
        WebSocket["WebSocket API"]
        StreamingAPI["Streaming API"]
    end

    subgraph Services["Business Logic Services"]
        subgraph Media["Media Services"]
            Catalog["Catalog Service"]
            Metadata["Metadata Service"]
            Search["Search Service"]
        end
        
        subgraph Transcode["Transcode Services"]
            JobMgr["Job Manager"]
            WorkerPool["Worker Pool"]
            QualityMgr["Quality Manager"]
        end
        
        subgraph Stream["Streaming Services"]
            SessionMgr["Session Manager"]
            AdaptiveMgr["Adaptive Bitrate\nManager"]
            HLSGen["HLS Generator"]
            DASHGen["DASH Generator"]
        end
        
        subgraph Store["Store Services"]
            Cart["Cart Service"]
            Payment["Payment Service"]
            Order["Order Service"]
        end
        
        subgraph Sync["Sync Services"]
            SyncMgr["Sync Manager"]
            ConflictRes["Conflict Resolver"]
            USBSync["USB Sync Adapter"]
        end
    end

    subgraph Data["Data Layer"]
        Postgres["PostgreSQL\nMetadata DB"]
        Redis["Redis\nCache"]
        S3["S3\nMedia Storage"]
        Kafka["Kafka\nEvent Queue"]
    end

    subgraph MediaInfra["Media Infrastructure"]
        FFmpeg["FFmpeg Cluster"]
        GPU["GPU Transcoders"]
        CDN["CDN Edge"]
        Origin["Origin Server"]
    end

    %% Conexiones
    Client --> Player
    Client --> DownloadMgr
    Client --> LibraryMgr
    
    Player --> REST
    Player --> StreamingAPI
    DownloadMgr --> REST
    LibraryMgr --> REST
    
    REST --> WebSocket
    WebSocket --> SessionMgr
    
    Catalog --> Search
    Search --> Metadata
    
    JobMgr --> WorkerPool
    WorkerPool --> FFmpeg
    WorkerPool --> GPU
    
    SessionMgr --> AdaptiveMgr
    AdaptiveMgr --> HLSGen
    AdaptiveMgr --> DASHGen
    HLSGen --> Origin
    DASHGen --> Origin
    
    Cart --> Payment
    Payment --> Order
    
    SyncMgr --> ConflictRes
    SyncMgr --> USBSync
    
    Catalog --> Postgres
    Search --> Redis
    SyncMgr --> Kafka
    
    Origin --> CDN
    CDN --> S3
```

---

## 4. Diagrama de Pipeline de Transcodificación

```mermaid
flowchart TB
    subgraph Input["Input Sources"]
        RawVideo["Raw Video\n(RAW/ProRes)"]
        EncodedVideo["Encoded Video\n(H.264/HEVC)"]
        PhysicalMedia["DVD/Blu-Ray"]
    end

    subgraph Ingestion["Ingestion Layer"]
        Validate["Validator\n- Format check\n- Integrity"]
        Extract["Metadata Extractor\n- Duration\n- Codecs\n- Bitrate"]
        Thumbnail["Thumbnail Generator\n- Key frames\n- Sprite"]
    end

    subgraph Transcode["Transcode Workers"]
        subgraph GPUPool["GPU Workers (Premium)"]
            GPU4K["4K HDR Worker\n- HEVC\n- Dolby Vision"]
            GPU1080["1080p HDR Worker\n- HEVC\n- HDR10"]
        end
        
        subgraph CPUPool["CPU Workers (Standard)"]
            CPU1080["1080p Worker\n- H.264\n- VP9"]
            CPU720["720p Worker\n- H.264\n- Efficient"]
            CPU480["480p Worker\n- H.264\n- Mobile"]
        end
    end

    subgraph Output["Output Formats"]
        subgraph VideoOut["Video Outputs"]
            FourK["4K HEVC\n(.mp4 + .ts)"]
            HD1080["1080p HEVC\n(.mp4 + .ts)"]
            HD720["720p H.264\n(.mp4 + .ts)"]
            SD480["480p H.264\n(.mp4 + .ts)"]
        end
        
        subgraph AudioOut["Audio Outputs"]
            Atmos["Dolby Atmos\n(.eac3)"]
            Surround["5.1 Surround\n(.aac)"]
            Stereo["Stereo\n(.aac + .mp3)"]
            Compressed["Compressed\n(.aac)"]
        end
        
        subgraph SubtitleOut["Subtitle Outputs"]
            VTT["WebVTT\n(HLS)"]
            SRT["SRT\n(Optional)"]
            Burned["Burned In\n(Optional)"]
        end
    end

    subgraph Packaging["Packaging"]
        Segment["Segmenter\n- 2-10 sec chunks"]
        Manifest["Manifest Generator\n- .m3u8\n- .mpd"]
        Encrypt["DRM Encrypt\n- Widevine\n- FairPlay"]
    end

    subgraph Distribution["Distribution"]
        Origin["Origin Storage\n(S3)"]
        CDN["CDN\n(CloudFront)"]
        Cache["Edge Cache"]
    end

    %% Flujo
    RawVideo --> Validate
    EncodedVideo --> Validate
    PhysicalMedia --> Validate
    
    Validate --> Extract
    Extract --> Thumbnail
    
    Extract --> GPU4K
    Extract --> GPU1080
    Extract --> CPU1080
    Extract --> CPU720
    Extract --> CPU480
    
    GPU4K --> FourK
    GPU1080 --> HD1080
    CPU1080 --> HD720
    CPU480 --> SD480
    
    Extract --> Atmos
    Extract --> Surround
    Extract --> Stereo
    Extract --> Compressed
    
    FourK --> Segment
    HD1080 --> Segment
    HD720 --> Segment
    SD480 --> Segment
    
    Atmos --> Segment
    Surround --> Segment
    Stereo --> Segment
    Compressed --> Segment
    
    Segment --> VTT
    Segment --> SRT
    Segment --> Burned
    
    VTT --> Manifest
    SRT --> Manifest
    Burned --> Manifest
    
    Segment --> Encrypt
    Encrypt --> Manifest
    
    Manifest --> Origin
    Origin --> CDN
    CDN --> Cache
```

---

## 5. Diagrama de Estados de Streaming

```mermaid
stateDiagram-v2
    [*] --> Idle: User selects media
    
    Idle --> Connecting: Play request
    Connecting --> LicenseAcq: DRM protected
    LicenseAcq --> Buffering: License OK
    
    Connecting --> Buffering: No DRM
    
    Buffering --> Playing: Buffer full
    Buffering --> Paused: User pause
    Buffering --> Connecting: Buffer low
    
    Playing --> Buffering: Buffer draining
    Playing --> Paused: User pause
    Playing --> Seeking: User seek
    Playing --> [*]: Media ended
    
    Seeking --> Buffering
    Paused --> Playing: User resume
    
    LicenseAcq --> Error: License failed
    Buffering --> Error: Connection lost
    
    Error --> Connecting: Retry
    Error --> [*]: Cancel
    
    note right of Buffering
        Adaptive Bitrate Switching
        - Quality based on bandwidth
        - Buffer level management
        - Pre-buffering strategy
    end note
    
    note right of LicenseAcq
        DRM Flow:
        1. Request license server
        2. Validate purchase
        3. Get content keys
        4. Initialize decryptor
    end note
```

---

## 6. Diagrama de Sincronización entre Dispositivos

```mermaid
flowchart TB
    subgraph Cloud["Cloud Sync Architecture"]
        CloudDB["Cloud Library DB"]
        SyncQueue["Sync Queue\n(Kafka)"]
        ConflictRes["Conflict Resolver"]
        StateMgr["State Manager"]
    end

    subgraph Device1["Dispositivo 1 (PC)"]
        LocalDB1["Local Library DB"]
        SyncAgent1["Sync Agent"]
        Storage1["Local Storage\n(Downloaded)"]
    end

    subgraph Device2["Dispositivo 2 (Mobile)"]
        LocalDB2["Local Library DB"]
        SyncAgent2["Sync Agent"]
        Storage2["Local Storage\n(Downloaded)"]
    end

    subgraph Device3["Dispositivo 3 (USB)"]
        USBStorage["USB Storage\n(Dump)"]
        USBSync["USB Sync Protocol"]
    end

    %% Flujo Cloud Sync
    SyncAgent1 --> SyncQueue: Push changes
    SyncQueue --> StateMgr: Process events
    StateMgr --> CloudDB: Update cloud state
    StateMgr --> SyncAgent2: Notify changes
    
    %% Conflict Resolution
    SyncAgent1 --> ConflictRes: Conflict detected
    ConflictRes --> StateMgr: Resolve & update
    StateMgr --> SyncAgent1: Resolved state
    StateMgr --> SyncAgent2: Resolved state

    %% USB Sync
    SyncAgent1 --> USBSync: Export to USB
    USBSync --> USBStorage: Write media files
    USBStorage --> USBSync: File list + metadata
    
    USBSync --> SyncAgent2: Import from USB
    SyncAgent2 --> LocalDB2: Merge library
    LocalDB2 --> Storage2: Copy media files
    
    %% Metadata sync
    LocalDB1 --> SyncAgent1: Check metadata
    SyncAgent1 --> StateMgr: Get latest state
    StateMgr --> LocalDB1: Sync metadata
    
    LocalDB2 --> SyncAgent2: Check metadata
    SyncAgent2 --> StateMgr: Get latest state
    StateMgr --> LocalDB2: Sync metadata
```

---

## 7. Diagrama de Base de Datos (ER)

```mermaid
erDiagram
    USERS ||--o{ PURCHASES : buys
    USERS ||--o{ LIBRARIES : owns
    USERS ||--o{ DEVICES : registers
    USERS ||--o{ PLAYLISTS : creates
    
    LIBRARIES ||--o{ LIBRARY_ITEMS : contains
    LIBRARIES ||--o{ DOWNLOADS : has
    DEVICES ||--o{ DOWNLOADS : stores
    
    MEDIA_ITEMS ||--o{ TRANSCODES : has
    MEDIA_ITEMS ||--o{ STREAM_SESSIONS : streamed
    PLAYLISTS ||--o{ PLAYLIST_ITEMS : includes
    
    DOWNLOADS ||--o{ SYNC_TASKS : synced
    
    USERS {
        uuid id PK
        string email
        string password_hash
        string subscription_type
        datetime created_at
        datetime last_login
        jsonb preferences
    }
    
    MEDIA_ITEMS {
        uuid id PK
        string title
        string type  -- VIDEO or AUDIO
        string genre
        integer duration_seconds
        jsonb metadata
        string thumbnail_url
        uuid uploader_id
        datetime released_at
        decimal price
        boolean is_explicit
        jsonb drm_info
    }
    
    TRANSCODES {
        uuid id PK
        uuid media_item_id FK
        string quality  -- 4K, 1080p, 720p, 480p
        string format  -- mp4, webm, ts
        string codec  -- hevc, h264, vp9
        integer bitrate
        string url
        integer file_size
    }
    
    LIBRARIES {
        uuid id PK
        uuid user_id FK
        string name
        datetime created_at
        datetime updated_at
    }
    
    LIBRARY_ITEMS {
        uuid id PK
        uuid library_id FK
        uuid media_item_id FK
        datetime added_at
        string playback_position
        boolean is_favorite
        jsonb user_data
    }
    
    DOWNLOADS {
        uuid id PK
        uuid library_item_id FK
        uuid device_id FK
        string quality
        string format
        string local_path
        datetime downloaded_at
        integer progress
        enum status
    }
    
    DEVICES {
        uuid id PK
        uuid user_id FK
        string device_name
        enum device_type
        string os_version
        datetime last_active
        string device_token
        integer max_storage
    }
    
    PLAYLISTS {
        uuid id PK
        uuid user_id FK
        string name
        string description
        boolean is_public
        datetime created_at
    }
    
    PLAYLIST_ITEMS {
        uuid id PK
        uuid playlist_id FK
        uuid media_item_id FK
        integer position
        datetime added_at
    }
    
    STREAM_SESSIONS {
        uuid id PK
        uuid media_item_id FK
        uuid user_id FK
        uuid device_id FK
        string quality
        string transcoded_url
        datetime started_at
        datetime ended_at
        integer bytes_streamed
    }
    
    PURCHASES {
        uuid id PK
        uuid user_id FK
        uuid media_item_id FK
        decimal amount
        string currency
        string payment_method
        datetime purchased_at
        string transaction_id
    }
```

---

## 8. Diagrama de Flujo de Compra

```mermaid
sequenceDiagram
    participant User
    participant App
    participant API
    participant Store
    participant Payment
    participant DRM
    participant Media

    User->>App: Browse media catalog
    App->>API: GET /media/{id}
    API-->>App: Media details + price
    
    User->>App: Add to cart
    App->>API: POST /cart/add
    API-->>App: Cart updated
    
    User->>App: Proceed to checkout
    App->>API: POST /cart/checkout
    API->>Store: Create order
    Store->>Payment: Process payment
    
    alt Payment successful
        Payment-->>Store: Payment confirmed
        Store-->>API: Order created
        API-->>App: Purchase confirmed
        
        App->>DRM: Initialize DRM license
        DRM-->>App: License acquired
        
        API->>Media: Enable download access
        Media-->>API: Download ready
        
        API-->>App: Purchase + DRM info
        App-->>User: Show purchased media
    else Payment failed
        Payment-->>Store: Payment failed
        Store-->>API: Order cancelled
        API-->>App: Payment error
        App-->>User: Show error message
    end
    
    Note over User,App: User can now stream or download
```

---

## 9. Diagrama de Streaming HLS/DASH

```mermaid
flowchart TB
    subgraph Client["Client Player"]
        Player["Video.js / Native Player"]
        ABR["ABR Controller"]
        Buffer["Buffer Manager"]
        DRMDecrypt["DRM Decryptor"]
    end

    subgraph Request["HTTP Requests"]
        MasterM3U8["GET /master.m3u8"]
        QualM3U8["GET /1080p.m3u8"]
        TSChunks["GET /segment1.ts"]
        KeyFile["GET /encryption.key"]
    end

    subgraph Server["CDN / Origin"]
        MasterPlaylist["Master Playlist\n.m3u8"]
        QualityPlaylists["Quality Playlists\n.m3u8 per quality"]
        Segments["TS Segments\n(2-10 seconds)"]
        KeyStore["DRM Key Store"]
    end

    %% Flujo de streaming
    Player --> ABR: User starts playback
    
    ABR --> MasterM3U8: Request master playlist
    MasterM3U8 --> Player: Return manifest
    
    Player --> ABR: Select quality based on bandwidth
    
    ABR --> QualM3U8: Request quality playlist
    QualM3U8 --> Player: Return segments list
    
    loop Playback loop
        Player --> Buffer: Request segment
        Buffer --> TSChunks: Fetch segment
        TSChunks --> Buffer: Segment data
        Buffer --> DRMDecrypt: Decrypt with key
        DRMDecrypt --> KeyFile: Request DRM key (if needed)
        KeyFile --> DRMDecrypt: Return key
        DRMDecrypt --> Player: Render frame
    end
    
    ABR --> Buffer: Monitor buffer level
    Buffer --> ABR: Report status
    
    ABR --> QualM3U8: Switch quality (up/down)
    QualM3U8 --> ABR: New segment list
    
    note right of ABR
        Adaptive Bitrate Logic:
        - High bandwidth → Switch to higher quality
        - Low bandwidth → Switch to lower quality
        - Buffer draining → Buffer more
        - Buffer full → Higher quality
    end note
```

---

## 10. Diagrama de Despliegue

```mermaid
flowchart TB
    subgraph CDN["Global CDN"]
        Edge["Edge Locations\n(Worldwide)"]
        Cache["Cache Layers"]
        OriginPull["Origin Pull"]
    end

    subgraph Cloud["Cloud Region"]
        subgraph K8s["Kubernetes Cluster"]
            subgraph APIPods["API Pods"]
                APIReplica1["API Replica 1"]
                APIReplica2["API Replica 2"]
                APIReplica3["API Replica 3"]
            end
            
            subgraph WorkerPods["Worker Pods"]
                TranscodePod["Transcode Workers"]
                SyncPod["Sync Workers"]
                DownloadPod["Download Workers"]
            end
            
            subgraph Services["Cluster Services"]
                K8sAPI["K8s API Service"]
                Ingress["Ingress Controller"]
            end
        end
        
        subgraph DB["Data Layer"]
            PostgresPrimary["PostgreSQL Primary"]
            PostgresReplica["PostgreSQL Replica"]
            RedisCluster["Redis Cluster"]
            Kafka["Kafka Cluster"]
        end
        
        subgraph Storage["Object Storage"]
            S3Origin["S3 Origin Bucket"]
            Glacier["Glacier Archive"]
        end
    end

    subgraph Transcode["Transcode Infrastructure"]
        GPUCluster["GPU Instances\n(P4d/P40)"]
        CPUCluster["CPU Instances\n(C6i)"]
        StoragePool["NVMe Storage\n(High IOPS)"]
    end

    subgraph Clients["Clients"]
        Desktop["Desktop Apps"]
        Mobile["Mobile Apps"]
        Web["Web Browsers"]
    end

    %% Conexiones
    Desktop --> Edge
    Mobile --> Edge
    Web --> Edge
    
    Edge --> Cache
    Cache --> OriginPull
    
    OriginPull --> K8sAPI
    Ingress --> APIReplica1
    Ingress --> APIReplica2
    Ingress --> APIReplica3
    
    APIReplica1 --> PostgresPrimary
    APIReplica1 --> RedisCluster
    APIReplica1 --> Kafka
    
    TranscodePod --> GPUCluster
    TranscodePod --> CPUCluster
    TranscodePod --> StoragePool
    
    StoragePool --> S3Origin
    S3Origin --> Glacier
    
    OriginPull --> S3Origin
    
    PostgresPrimary --> PostgresReplica
    RedisCluster --> Cache
```

---

## 11. Diagrama de Arquitectura DRM

```mermaid
flowchart TB
    subgraph ContentPrep["Content Preparation"]
        RawMedia["Raw Media File"]
        Encryptor["DRM Encryptor\n(Widevine/FairPlay)"]
        KeyServer["Key Server"]
        Packager["Content Packager"]
    end

    subgraph License["License Server"]
        AuthService["Auth Service"]
        PolicyEngine["Policy Engine"]
        KeyVault["Key Vault\n(Hardware Security)"]
        LicenseDB["License Database"]
    end

    subgraph Client["Client Device"]
        DRMModule["DRM Module"]
        LicenseReq["License Request"]
        Decryptor["Content Decryptor"]
        Player["Media Player"]
    end

    subgraph Distribution["Content Distribution"]
        CDN["CDN"]
        EncryptedContent["Encrypted Media\n(.mp4 + .ts)"]
        Manifests["DRM Manifests\n(.m3u8 + .key)"]
    end

    %% Flujo de preparación
    RawMedia --> Encryptor: Encrypt content
    Encryptor --> KeyServer: Request content key
    KeyServer --> KeyVault: Generate key
    KeyVault --> KeyServer: Return key
    KeyServer --> Encryptor: Content key + KID
    Encryptor --> Packager: Encrypted segments
    Packager --> Manifests: Generate manifest with KID
    Packager --> EncryptedContent: Segmented + encrypted
    
    EncryptedContent --> CDN
    Manifests --> CDN
    
    %% Flujo de licencia
    Player --> DRMModule: Initialize playback
    DRMModule --> LicenseReq: Build license request
    LicenseReq --> AuthService: Send license request
    
    AuthService --> PolicyEngine: Validate request
    PolicyEngine --> LicenseDB: Check entitlements
    LicenseDB -->> PolicyEngine: User permissions
    
    PolicyEngine --> KeyVault: Get content key
    KeyVault -->> PolicyEngine: Wrapped key
    
    PolicyEngine --> LicenseDB: Record license grant
    PolicyEngine -->> AuthService: License response
    
    AuthService -->> LicenseReq: License (encrypted key)
    LicenseReq -->> Decryptor: Decrypt content key
    Decryptor -->> Player: Decoded frames
```

---

## 12. Diagrama de Comparación de Protocolos de Streaming

```mermaid
flowchart LR
    subgraph Protocols["Streaming Protocols"]
        HLS["HLS\nHTTP Live Streaming"]
        DASH["DASH\nDynamic Adaptive Streaming"]
        RTMP["RTMP\nReal-Time Messaging"]
        WebRTC["WebRTC"]
        Progresive["Progressive Download"]
    end
    
    subgraph Characteristics["Characteristics"]
        subgraph Latency["Latency"]
            HighLat["10-30s\n(HLS/DASH)"]
            MedLat["2-5s\n(RTMP)"]
            LowLat["<1s\n(WebRTC)"]
        end
        
        subgraph Compatibility["Browser Compatibility"]
            WideComp["HLS/DASH\n(All modern browsers)"]
            LimitedComp["RTMP\n(Legacy only)"]
            RealTimeComp["WebRTC\n(Real-time apps)"]
        end
        
        subgraph Adaptive["Adaptive Bitrate"]
            ABR_HLS["HLS: Yes\n(.m3u8 playlists)"]
            ABR_DASH["DASH: Yes\n(.mpd manifest)"]
            ABR_RTMP["RTMP: No"]
            ABR_WebRTC["WebRTC: Limited"]
        end
        
        subgraph DRM["DRM Support"]
            DRM_HLS["HLS: Yes\n(Widevine/FairPlay)"]
            DRM_DASH["DASH: Yes\n(Widevine/PlayReady)"]
            DRM_RTMP["RTMP: Limited"]
            DRM_WebRTC["WebRTC: Yes\n(Encrypted extensions)"]
        end
    end
    
    %% Conexiones
    HLS --> HighLat
    HLS --> WideComp
    HLS --> ABR_HLS
    HLS --> DRM_HLS
    
    DASH --> HighLat
    DASH --> WideComp
    DASH --> ABR_DASH
    DASH --> DRM_DASH
    
    RTMP --> MedLat
    RTMP --> LimitedComp
    RTMP --> ABR_RTMP
    RTMP --> DRM_RTMP
    
    WebRTC --> LowLat
    WebRTC --> RealTimeComp
    WebRTC --> ABR_WebRTC
    WebRTC --> DRM_WebRTC
    
    Progresive --> HighLat
    Progresive --> WideComp
    Progresive --> ABR_WebRTC
    Progresive --> DRM_HLS
    
    note right of HLS
        RECOMMENDED:
        Primary: HLS + DASH
        Fallback: Progressive
    end note
```
