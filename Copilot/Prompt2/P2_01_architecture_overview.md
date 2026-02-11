# Arquitectura General — Servicio de Calendario

## Visión general
Servicio de calendario accesible desde web y múltiples aplicaciones (móviles, tablets, desktops). Calendarios cifrados en repositorio y desencriptados bajo permiso.

```mermaid
graph TB
  subgraph Clients["Clientes"]
    Web["Web App"]
    Mobile["Mobile App"]
    Desktop["Desktop App"]
  end

  APIGW["API Gateway / Auth Proxy"]
  subgraph Backend["Servicios Backend"]
    Auth["Auth Service (OAuth2 / JWT)"]
    Calendar["Calendar Service (Negocio)"]
    Crypto["Crypto Service (Encrypt/Decrypt)"]
    Notify["Notification Service"]
    Sync["Sync Service (WebSockets / Push)"]
  end

  subgraph Storage["Persistencia"]
    DB["PostgreSQL (encrypted blobs)"]
    Blob["Object Storage (S3/Azure)"]
    Cache["Redis"]
  end

  Clients -->|HTTPS| APIGW
  APIGW --> Auth
  APIGW --> Calendar
  Calendar --> Crypto
  Calendar --> DB
  Calendar --> Blob
  Calendar --> Cache
  Notify -->|SMTP/Push| External["Push/Email Providers"]
  Sync --> Clients
  Auth --> DB
  Crypto --> DB
```

- Criterios clave: cifrado en reposo, autorización por recurso, sincronización en tiempo real, notificaciones.
- Microservicios recomendados por responsabilidad (Auth, Calendar, Crypto, Notify, Sync).