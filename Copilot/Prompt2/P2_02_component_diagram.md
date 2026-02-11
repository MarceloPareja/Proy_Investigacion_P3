# Diagrama de Componentes — Servicio de Calendario

```mermaid
graph TB
  subgraph Frontend["Frontend / Apps"]
    Web["UI Web (React/Vue)"]
    Mobile["App iOS/Android (Flutter/ReactNative)"]
    Desktop["Electron / Native"]
  end

  subgraph Infra["Infra / Gateway"]
    APIGW["API Gateway (Auth, Rate Limit)"]
    CDN["CDN"]
  end

  subgraph Services["Microservicios"]
    Auth["Auth Service (OAuth2/JWT)"]
    Calendar["Calendar Service (CRUD, Views)"]
    Crypto["Crypto Service (Key mgmt, Decrypt) "]
    Sync["Sync Service (WS / Push) "]
    Notify["Notification Service (Email/Push)"]
  end

  subgraph Data["Datos"]
    Postgres["PostgreSQL (metadata)"]
    ObjectStorage["S3/Blob (attachments) "]
    Redis["Redis (sessions, locks)"]
    KMS["Key Management Service (AWS KMS / Azure Key Vault)"]
  end

  Web --> APIGW
  Mobile --> APIGW
  Desktop --> APIGW
  APIGW --> Auth
  APIGW --> Calendar
  Calendar --> Crypto
  Calendar --> Postgres
  Calendar --> ObjectStorage
  Calendar --> Redis
  Crypto --> KMS
  Auth --> Postgres
  Notify --> External["Push/Email Providers"]
  Sync --> APIGW
```

Responsabilidades principales:
- `Auth`: login, token, MFA opcional.
- `Calendar`: CRUD de calendarios, reglas de permisos, operaciones de vista (desencriptado bajo permiso).
- `Crypto`: gestión de claves, cifrado/descifrado por usuario/objeto.
- `Sync`: mantener vistas actualizadas en múltiples clientes.
- `Notify`: programar y enviar alertas y recordatorios.