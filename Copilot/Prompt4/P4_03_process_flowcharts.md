# Flujos de Proceso — Reproducción, Compra y Sync

## 1) Reproducción en Streaming (on-demand)

```mermaid
flowchart TD
  U[Usuario solicita reproducción] --> API[API Gateway]
  API --> Auth[Verificar sesión/permiso]
  Auth -->|ok| RH[Request Handler]
  RH --> CheckCache{¿Edge/Cache contiene asset?}
  CheckCache -->|Sí| CDN[Redirigir a CDN]
  CheckCache -->|No| Origin[Solicitar origen (VS/AS)]
  Origin -->|Puede pre-transcodificado| CDN
  Origin -->|Si no| Jobs[Crear job de transcodificación]
  Jobs --> Transcoder
  Transcoder --> Storage
  Storage --> CDN
  CDN --> App
```

## 2) Compra y descarga a la librería

```mermaid
flowchart TD
  User[Usuario] --> PurchaseUI[Interfaz de compra]
  PurchaseUI --> Billing[Billing Service]
  Billing -->|pago ok| MediaStore
  MediaStore --> LibrarySvc[Agregar a librería del usuario]
  LibrarySvc --> DownloadMgr[Prepara paquete DRM/licencia]
  DownloadMgr --> App[Cliente descarga y guarda offline]
```

## 3) Instalador y registro

```mermaid
flowchart TD
  WebUser --> InstallerSite
  InstallerSite -->|descarga| Installer
  Installer --> Device[Dispositivo]
  Device -->|instalación| App
  App -->|registro| AppS(API)
  AppS -->|crear cuenta| MediaStore
```

## 4) Sincronización de librería (USB)

```mermaid
flowchart TD
  PC[PC con librería] -->|USB| Mobile[Dispositivo móvil]
  PC --> AppS[Opcional: sync con MediaStore]
  Mobile --> LibraryLocal[Actualizar librería local]
```