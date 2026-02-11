# Arquitectura General — Sistema de Reproducción Multimedia Online

## Visión general
Cliente (app de reproducción) ↔ AppS (Application Server) ↔ Servidor Video (VS) / Servidor Audio (AS) ↔ Almacenamiento/CDN. AppS realiza transcodificación, agregado de subtítulos, compresión de audio y gestión de tienda/media.

```mermaid
graph TB
  subgraph Clients["Clientes"]
    App["Aplicación (PC/Móvil/Tablet)"]
    InstallerSite["Sitio de descarga del instalador"]
  end

  subgraph AppS["AppS - Application Server"]
    API["API / Auth / Session"]
    MediaStore["Media Store (compras/librería)"]
    Transcoder["Transcoder / Processor"]
    DRMSvc["DRM / Licencias"]
    DownloadSvc["Download Manager"]
    SyncSvc["Sync Manager (USB/Wireless)"]
  end

  subgraph MediaServers["Servidores de Media"]
    VS["Video Server (VS)"]
    AS["Audio Server (AS)"]
    Storage["Object Storage / Blob"]
    CDN["CDN / Edge Cache"]
  end

  App -->|HTTPS Request| API
  InstallerSite -->|GET installer| API
  API --> MediaStore
  API --> Transcoder
  Transcoder --> VS
  Transcoder --> AS
  MediaStore --> Storage
  Storage --> VS
  Storage --> AS
  VS --> CDN
  AS --> CDN
  CDN --> App
  API --> DRMSvc
  App -->|USB sync| SyncSvc
  SyncSvc --> MediaStore
```

Principios: separación de responsabilidades, transcodificación on-demand o pre-transcodificada, CDN para distribución, DRM opcional, soporte offline (descarga a librería).
