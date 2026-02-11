# Diagrama de Componentes — AppS y Servicios

```mermaid
graph TB
  subgraph Client["Cliente"]
    App["Player App"]
  end

  subgraph AppS["Application Server (AppS)"]
    API["API Gateway / Auth"]
    RequestHandler["Request Handler"]
    MediaStore["Media Store (catalog, purchases)"]
    Transcoder["Transcoder / Worker Pool"]
    SubtitleSvc["Subtitle Service"]
    AudioProcessor["Audio Processor"]
    DownloadMgr["Download Manager / DRM"]
    LibrarySvc["User Library Service"]
    Billing["Billing / Purchase Service"]
    Sync["Sync Service (USB / Network)"]
    Jobs["Job Queue (Distribución transcodes)"]
  end

  subgraph MediaInfra["Infra de Media"]
    VS["Video Server (origin)"]
    AS["Audio Server (origin)"]
    Storage["Object Storage (S3/Blob)"]
    CDN["CDN / Edge"]
  end

  App --> API
  API --> RequestHandler
  RequestHandler --> MediaStore
  RequestHandler --> LibrarySvc
  RequestHandler --> DownloadMgr
  RequestHandler --> Jobs
  Jobs --> Transcoder
  Jobs --> AudioProcessor
  Jobs --> SubtitleSvc
  Transcoder --> Storage
  AudioProcessor --> Storage
  Storage --> VS
  Storage --> AS
  VS --> CDN
  AS --> CDN
  CDN --> App
  Billing --> MediaStore
  Sync --> LibrarySvc
```

Notas:
- `Jobs` usa broker (RabbitMQ/Kafka) para escalabilidad.
- `Transcoder` puede usar worker autoscaling (FFmpeg containers).
- `DownloadMgr` aplica DRM/licencias y empaqueta descargas para librería offline.
