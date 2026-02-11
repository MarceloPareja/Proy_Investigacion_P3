# Arquitectura del Sistema de Calendario Multiplataforma

## 1. Visión General

Sistema de calendario empresarial multiplataforma que permite a usuarios acceder a sus calendarios desde múltiples dispositivos con sincronización en tiempo real, encriptación de datos sensibles y sistema de notificaciones.

## 2. Requisitos Funcionales

| Requisito | Descripción |
|-----------|-------------|
| RF-01 | Acceso web al calendario con autenticación |
| RF-02 | Aplicaciones nativas para múltiples plataformas |
| RF-03 | Encriptación de calendarios almacenados |
| RF-04 | Control de permisos por usuario |
| RF-05 | Sistema de alertas y notificaciones |
| RF-06 | Sincronización en tiempo real |
| RF-07 | Múltiples visualizaciones de calendarios |

## 3. Stack Tecnológico Recomendado

### 3.1 Backend

**Recomendación: Node.js con TypeScript + NestJS**

**Justificación:**
- Excelente rendimiento para aplicaciones I/O intensivas
- Ecosistema maduro para APIs RESTful y WebSockets
- TypeScript proporciona tipado estático para mantenibilidad
- NestJS ofrece arquitectura modular y escalable
- Gran soporte para aplicaciones en tiempo real

**Alternativas:**
- **Python/Django REST Framework**: Buena para lógica compleja, menos performante para tiempo real
- **Go (Golang)**: Excelente rendimiento y concurrencia, curva de aprendizaje más alta
- **Java/Spring Boot**: Robusto para enterprise, pero más pesado

### 3.2 Base de Datos

| Tipo | Tecnología | Uso |
|------|------------|-----|
| Primary | PostgreSQL | Datos estructurados, usuarios, calendarios |
| Cache | Redis | Sesiones, cacheo, cola de notificaciones |
| Time Series | InfluxDB | Métricas de uso, analytics |

### 3.3 Almacenamiento

**Encrypted Storage:**
- PostgreSQL con pgcrypto (encriptación a nivel BD)
- AWS KMS o HashiCorp Vault para gestión de claves
- cifrado AES-256 para archivos adjuntos

### 3.4 Frontend Web

**Recomendación: React con TypeScript**

**Justificación:**
- Componentes reutilizables para diferentes vistas
- Ecosistema rico de libraries de calendarios
- TypeScript para mantenibilidad
- Redux/Zustand para estado global

**Libraries recomendadas:**
- FullCalendar, react-big-calendar, react-calendar
- Socket.io-client para tiempo real

### 3.5 Aplicaciones Móviles

**Estrategia Multiplataforma:**

| Plataforma | Tecnología | Ventajas |
|------------|------------|----------|
| iOS | React Native / Swift | Rendimiento nativo |
| Android | React Native / Kotlin | Consistencia con web |
| Desktop | Electron / Tauri | Código compartido web |

### 3.6 Servicios Externos

| Servicio | Función |
|----------|----------|
| WebSocket Server | Sincronización en tiempo real |
| Push Notifications | Firebase Cloud Messaging (FCM), APNs |
| Email Service | SendGrid / AWS SES |
| Auth Service | OAuth 2.0 / OpenID Connect |
| Encryption Service | AWS KMS / HashiCorp Vault |

---

## 4. Arquitectura General

```
┌─────────────────────────────────────────────────────────────────────┐
│                        USERS & DEVICES                              │
├─────────────────────────────────────────────────────────────────────┤
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  │
│  │   Web   │  │  iOS    │  │Android  │  │ Desktop │  │  API    │  │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘  │
└───────┼────────────┼────────────┼────────────┼────────────┼───────┘
        │            │            │            │            │
        └────────────┴────────────┴────────────┴────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      API GATEWAY (Kong/AWS API Gateway)             │
│         ┌──────────────────────────────────────────────┐            │
│         │  - Rate Limiting                            │            │
│         │  - Authentication Validation                │            │
│         │  - Request Routing                           │            │
│         │  - SSL Termination                           │            │
│         └──────────────────────────────────────────────┘            │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
         ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
         │ Auth Service│  │ Calendar    │  │ Notification│
         │    (OAuth)  │  │   Service   │  │   Service   │
         └──────┬──────┘  └──────┬──────┘  └──────┬──────┘
                │                │                │
         ┌──────┴──────┐  ┌──────┴──────┐  ┌──────┴──────┐
         │             │  │             │  │             │
    ┌────▼────┐   ┌────▼────┐   ┌────▼────┐   ┌────▼────┐
    │  User   │   │ Calendar │   │  Event  │   │ Alert   │
    │ Service │   │ Service  │   │ Service │   │ Service │
    └────┬────┘   └────┬────┘   └────┬────┘   └────┬────┘
         │             │                │              │
         ▼             ▼                ▼              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        DATA LAYER                                    │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │  PostgreSQL │  │    Redis    │  │  Encrypted  │  │    Kafka    │ │
│  │  (Primary)  │  │   (Cache)   │  │   Storage   │  │   (Events)  │ │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 5. Patrones Arquitectónicos

### 5.1 Patrón Principal: Microservicios

```
┌─────────────────────────────────────────────────────────────────┐
│                    MICROSERVICES ARCHITECTURE                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌───────────────┐   ┌───────────────┐   ┌───────────────┐     │
│  │  Auth Service │   │ Calendar      │   │ Notification │     │
│  │               │   │ Service       │   │ Service       │     │
│  │  - Login      │   │  - CRUD        │   │  - Push       │     │
│  │  - Register   │   │  - Sharing     │   │  - Email      │     │
│  │  - Sessions   │   │  - Encryption  │   │  - SMS        │     │
│  │  - OAuth      │   │  - Search      │   │  - In-app     │     │
│  └───────┬───────┘   └───────┬───────┘   └───────┬───────┘     │
│          │                   │                   │              │
│  ┌───────┴───────┐   ┌───────┴───────┐   ┌───────┴───────┐     │
│  │  User Service │   │  Event Service│   │  Alert Service│     │
│  │               │   │               │   │               │     │
│  │  - Profiles   │   │  - CRUD       │   │  - Scheduling │     │
│  │  - Preferences│   │  - Reminders  │   │  - Delivery   │     │
│  │  - Permissions│   │  - Recurring  │   │  - Templates  │     │
│  └───────────────┘   └───────────────┘   └───────────────┘     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Patrones Complementarios

#### a) API Gateway Pattern
Centraliza el acceso a todos los microservicios

```typescript
// Conceptual
class ApiGateway {
    routes = {
        '/api/auth/*': 'auth-service',
        '/api/calendars/*': 'calendar-service',
        '/api/events/*': 'event-service',
        '/api/notifications/*': 'notification-service'
    }
}
```

#### b) Repository Pattern
Abstracción del acceso a datos

```typescript
interface CalendarRepository {
    findById(id: string): Promise<Calendar>;
    findByUser(userId: string): Promise<Calendar[]>;
    save(calendar: Calendar): Promise<void>;
    encryptAndSave(calendar: Calendar): Promise<void>;
}
```

#### c) Factory Pattern
Creación de objetos complejos (calendarios encriptados)

```typescript
class CalendarFactory {
    createEncryptedCalendar(userId: string, data: CalendarData): Calendar {
        const encrypted = this.encryptData(data);
        return new Calendar(userId, encrypted);
    }
}
```

#### d) Observer Pattern
Sincronización en tiempo real y notificaciones

```typescript
// WebSocket para sincronización
class CalendarSubject {
    private observers: WebSocket[] = [];
    
    notify(calendarId: string, event: CalendarEvent) {
        this.observers
            .filter(ws => ws.user.hasAccess(calendarId))
            .forEach(ws => ws.send(event));
    }
}
```

#### e) Strategy Pattern
Diferentes algoritmos de encriptación y notificación

```typescript
interface EncryptionStrategy {
    encrypt(data: string): string;
    decrypt(data: string): string;
}

interface NotificationStrategy {
    send(notification: Notification): Promise<void>;
}
```

#### f) Circuit Breaker Pattern
Tolerancia a fallos en servicios externos

```typescript
@Injectable()
class NotificationService {
    @CircuitBreaker({ timeout: 3000, errorThreshold: 50 })
    async sendPushNotification(userId: string, alert: Alert): Promise<void> {
        // Fallback si el servicio falla
    }
}
```

#### g) CQRS (Command Query Responsibility Segregation)
Separación de operaciones de lectura y escritura

```
┌─────────────────────────────────────────┐
│              COMMANDS (Writes)           │
│  ┌─────────────────────────────────┐    │
│  │ - CreateCalendar                │    │
│  │ - UpdateEvent                   │    │
│  │ - DeleteEvent                   │    │
│  │ - ShareCalendar                 │    │
│  │ - SetAlert                      │    │
│  └─────────────────────────────────┘    │
│                    │                    │
│                    ▼                    │
│            ┌─────────────┐              │
│            │  Event Bus  │──────────────┼──► Event Store
│            │  (Kafka)    │              │
│            └─────────────┘              │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│              QUERIES (Reads)             │
│  ┌─────────────────────────────────┐    │
│  │ - GetCalendar                   │    │
│  │ - GetEvents                     │    │
│  │ - GetAlerts                     │    │
│  │ - SearchCalendars               │    │
│  │ - GetSharedWith                 │    │
│  └─────────────────────────────────┘    │
│                    │                    │
│            ┌─────────────┐              │
│            │  Read DB    │              │
│            │ (Optimized) │              │
│            └─────────────┘              │
└─────────────────────────────────────────┘
```

---

## 6. Modelo de Dominio

### 6.1 Entidades Principales

```
┌─────────────────────────────────────────────────────────────┐
│                      CORE DOMAIN                             │
├─────────────────────────────────────────────────────────────┤
│  User                                                         │
│    - id: UUID                                                 │
│    - email: String                                            │
│    - password_hash: String                                   │
│    - profile: UserProfile                                     │
│    - preferences: UserPreferences                            │
│    - calendars: List<Calendar>                               │
│    - devices: List<Device>                                   │
├─────────────────────────────────────────────────────────────┤
│  Calendar                                                     │
│    - id: UUID                                                 │
│    - owner_id: UUID                                           │
│    - name: String                                             │
│    - description: String                                     │
│    - color: String                                            │
│    - is_encrypted: Boolean                                   │
│    - encryption_key_id: UUID                                 │
│    - shared_with: List<CalendarShare>                        │
│    - permissions: CalendarPermission                         │
│    - events: List<Event>                                      │
├─────────────────────────────────────────────────────────────┤
│  Event                                                        │
│    - id: UUID                                                 │
│    - calendar_id: UUID                                        │
│    - title: String                                           │
│    - description: String                                     │
│    - start_time: DateTime                                    │
│    - end_time: DateTime                                      │
│    - location: String                                        │
│    - is_all_day: Boolean                                     │
│    - recurrence: RecurrenceRule                               │
│    - reminders: List<Reminder>                                │
│    - attendees: List<Attendee>                               │
│    - status: Enum (CONFIRMED, TENTATIVE, CANCELLED)          │
├─────────────────────────────────────────────────────────────┤
│  Alert                                                        │
│    - id: UUID                                                 │
│    - event_id: UUID                                           │
│    - user_id: UUID                                            │
│    - type: Enum (EMAIL, PUSH, SMS, IN_APP)                    │
│    - trigger_time: DateTime                                  │
│    - trigger_before: Integer (minutes)                       │
│    - message: String                                          │
│    - is_active: Boolean                                       │
│    - was_sent: Boolean                                        │
├─────────────────────────────────────────────────────────────┤
│  Device                                                       │
│    - id: UUID                                                 │
│    - user_id: UUID                                            │
│    - device_token: String                                    │
│    - platform: Enum (IOS, ANDROID, WEB, DESKTOP)             │
│    - last_active: DateTime                                   │
│    - push_enabled: Boolean                                   │
├─────────────────────────────────────────────────────────────┤
│  CalendarShare                                                │
│    - id: UUID                                                 │
│    - calendar_id: UUID                                        │
│    - shared_with_user_id: UUID                               │
│    - permission: Enum (READ, WRITE, ADMIN)                    │
│    - shared_by: UUID                                          │
│    - created_at: DateTime                                    │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 Value Objects

```typescript
class EncryptedData {
    constructor(
        readonly ciphertext: string,
        readonly iv: string,
        readonly algorithm: string,
        readonly keyId: string
    ) {}
}

class CalendarPermission {
    constructor(
        readonly canView: boolean,
        readonly canEdit: boolean,
        readonly canShare: boolean,
        readonly canDelete: boolean
    ) {}
}

class RecurrenceRule {
    constructor(
        readonly frequency: 'DAILY' | 'WEEKLY' | 'MONTHLY' | 'YEARLY',
        readonly interval: number,
        readonly until?: Date,
        readonly count?: number,
        readonly daysOfWeek?: number[],
        readonly exceptions?: Date[]
    ) {}
}
```

---

## 7. Flujo de Autenticación y Encriptación

```
┌─────────────────────────────────────────────────────────────────────┐
│                    AUTHENTICATION & ENCRYPTION FLOW                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. USER LOGIN                                                      │
│  ┌─────────┐     ┌─────────────┐     ┌─────────────┐               │
│  │  User   │────►│ Auth Service │────►│  Token Gen  │               │
│  │  (Creds)│     │              │     │   (JWT)     │               │
│  └─────────┘     └─────────────┘     └──────┬──────┘               │
│                                              │                      │
│  ┌─────────┐     ┌─────────────┐            │                      │
│  │  JWT    │◄────│  Validate    │◄───────────┘                      │
│  │  Token  │     │  Credentials │                                   │
│  └────┬────┘     └─────────────┘                                   │
│       │                                                            │
│  2. CALENDAR ACCESS                                                 │
│       │                                                            │
│       ▼                                                            │
│  ┌─────────────┐     ┌─────────────────┐     ┌─────────────────┐  │
│  │   Request   │────►│ Permission      │────►│ Check User      │  │
│  │ Calendar +  │     │    Service       │     │ Permissions     │  │
│  │ JWT Token   │     │                 │     │                 │  │
│  └─────────────┘     └────────┬────────┘     └────────┬────────┘  │
│                                │                       │            │
│                                ▼                       ▼            │
│                       ┌─────────────────┐    ┌─────────────────┐  │
│                       │ Has Permission? │    │ Get Encryption  │  │
│                       │                 │    │ Key from Vault  │  │
│                       └────────┬────────┘    └────────┬────────┘  │
│                                │                       │            │
│                                ▼                       ▼            │
│                       ┌─────────────────┐    ┌─────────────────┐  │
│                       │ Return Calendar  │    │ Decrypt Calendar│  │
│                       │ (Encrypted)     │    │ Data            │  │
│                       └─────────────────┘    └────────┬────────┘  │
│                                                       │            │
│  3. RESPONSE                                          ▼            │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                     API Response                            │   │
│  │  {                                                         │   │
│  │    "calendar": { "id": "...", "name": "...", ... },        │   │
│  │    "decrypted_data": { "events": [...] },                  │   │
│  │    "permissions": { "canEdit": true, ... }                  │   │
│  │  }                                                         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 8. Sistema de Notificaciones

```
┌─────────────────────────────────────────────────────────────────────┐
│                      NOTIFICATION SYSTEM                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    ALERT TRIGGER                            │    │
│  │  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐    │    │
│  │  │  Event  │──►│  Check  │──►│ Trigger │──►│  Queue  │    │    │
│  │  │  Time   │   │  Alerts │   │  Alert  │   │  Alert  │    │    │
│  │  └─────────┘   └─────────┘   └─────────┘   └────┬────┘    │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                     │
│                              ▼                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    NOTIFICATION WORKER                       │    │
│  │  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐    │    │
│  │  │  Get    │──►│  Get    │──►│ Select  │──►│  Send   │    │    │
│  │  │ Pending │   │ User    │   │ Channel │   │ via     │    │    │
│  │  │ Alerts  │   │ Devices │   │         │   │ Channel │    │    │
│  │  └─────────┘   └─────────┘   └─────────┘   └────┬─────┘    │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                     │
│              ┌───────────────┼───────────────┐                      │
│              ▼               ▼               ▼                      │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐              │
│  │  Push       │   │   Email     │   │   In-App     │              │
│  │  Service    │   │   Service   │   │   Service    │              │
│  │             │   │             │   │             │              │
│  │  - FCM      │   │  - SendGrid │   │  - WebSocket │              │
│  │  - APNs     │   │  - Templates│   │  - Database  │              │
│  └──────┬──────┘   └──────┬──────┘   └──────┬──────┘              │
│         │                 │                 │                       │
│         ▼                 ▼                 ▼                       │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐              │
│  │   Devices   │   │   Email     │   │   Clients   │              │
│  │  (FCM/APNs) │   │  Inbox      │   │   (Real-time│              │
│  └─────────────┘   └─────────────┘   └─────────────┘              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 9. Sincronización en Tiempo Real

```
┌─────────────────────────────────────────────────────────────────────┐
│                    REAL-TIME SYNCHRONIZATION                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────┐    WebSocket    ┌─────────┐    Event Bus    ┌─────────┐│
│  │ Client  │◄──────────────►│  Sync   │◄──────────────►│  Other  ││
│  │         │                │ Service │                │Services ││
│  └─────────┘                └────┬────┘                └─────────┘│
│                                │                            │      │
│                                ▼                            │      │
│                        ┌─────────────┐                     │      │
│                        │  Redis      │                     │      │
│                        │  Pub/Sub    │─────────────────────┘      │
│                        │  Channel    │                            │
│                        └─────────────┘                            │
│                                                                      │
│  EVENT FLOW:                                                        │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                                                             │    │
│  │  User A updates event                                       │    │
│  │         │                                                    │    │
│  │         ▼                                                    │    │
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐      │    │
│  │  │   Client    │───►│  API        │───►│  Calendar   │      │    │
│  │  │  (Browser)  │    │  Gateway    │    │  Service    │      │    │
│  │  └─────────────┘    └─────────────┘    └──────┬──────┘      │    │
│  │                                               │             │    │
│  │              ┌─────────────────────────────────┘             │    │
│  │              │                                               │    │
│  │              ▼                                               │    │
│  │      ┌─────────────┐    ┌─────────────┐    ┌─────────────┐ │    │
│  │      │  Validate   │───►│   Update    │───►│  Publish    │ │    │
│  │      │  Request    │    │   Database  │    │  Event      │ │    │
│  │      └─────────────┘    └─────────────┘    └──────┬──────┘ │    │
│  │                                                   │        │    │
│  │              ┌───────────────────────────────────┘        │    │
│  │              │                                        │    │
│  │              ▼                                        │    │
│  │      ┌─────────────┐    ┌─────────────┐               │    │
│  │      │  Sync       │───►│  WebSocket  │───────────────┼────┘    │
│  │      │  Service     │    │  Broadcast  │               │         │
│  │      └─────────────┘    └──────┬──────┘               │         │
│  │                                 │                      │         │
│  │                    ┌────────────┼────────────┐         │         │
│  │                    ▼            ▼            ▼         │         │
│  │            ┌─────────────┐ ┌─────────────┐ ┌─────────────┐       │
│  │            │  User A    │ │  User B    │ │  User C    │       │
│  │            │  (Same     │ │  (Shared   │ │  (Other    │       │
│  │            │   Device)  │ │   Calendar)│ │   Device)  │       │
│  │            └─────────────┘ └─────────────┘ └─────────────┘       │
│  │                                                             │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 10. Consideraciones de Seguridad

### 10.1 Encriptación

```
┌─────────────────────────────────────────────────────────────────────┐
│                      ENCRYPTION ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                   DATA ENCRYPTION                          │    │
│  │                                                              │    │
│  │  Plaintext Data                                              │    │
│  │       │                                                      │    │
│  │       ▼                                                      │    │
│  │  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐       │    │
│  │  │   AES-256   │───►│   Add IV    │───►│   Store     │       │    │
│  │   Encryption  │    │  (Random)   │    │  Ciphertext │       │    │
│  │  └─────────────┘    └─────────────┘    └─────────────┘       │    │
│  │                                                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                   KEY MANAGEMENT                            │    │
│  │                                                              │    │
│  │  ┌─────────────────────────────────────────────────────┐    │    │
│  │  │              AWS KMS / HashiCorp Vault              │    │    │
│  │  │                                                      │    │    │
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │    │    │
│  │  │  │ Master Key  │  │ User Keys   │  │ Calendar    │ │    │    │
│  │  │  │ (Hardware)  │  │ (Per User)  │  │ Keys (DEK)  │ │    │    │
│  │  │  └─────────────┘  └─────────────┘  └─────────────┘ │    │    │
│  │  │                                                      │    │    │
│  │  │  - Key rotation automatic                            │    │    │
│  │  │  - Audit logging                                      │    │    │
│  │  │  - Access policies                                    │    │    │
│  │  └─────────────────────────────────────────────────────┘    │    │
│  │                                                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                   TRANSPORT ENCRYPTION                       │    │
│  │                                                              │    │
│  │  - TLS 1.3 for all communications                           │    │
│  │  - Certificate pinning for mobile apps                      │    │
│  │  - mTLS for service-to-service communication                 │    │
│  │                                                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 10.2 Control de Acceso

```
┌─────────────────────────────────────────────────────────────────────┐
│                      ACCESS CONTROL MATRIX                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Permission Levels:                                                  │
│  ┌─────────────┬────────┬─────────┬─────────┬─────────┬──────────┐ │
│  │  Resource   │ OWNER  │  ADMIN  │  EDITOR │  VIEWER │  PUBLIC  │ │
│  ├─────────────┼────────┼─────────┼─────────┼─────────┼──────────┤ │
│  │ Calendar    │  CRUD  │   CRUD  │   CRU   │    R    │    -     │ │
│  │ Event       │  CRUD  │   CRUD  │   CRU   │    R    │    -     │ │
│  │ Alert       │  CRUD  │   CRUD  │   CRU   │    R    │    -     │ │
│  │ Share       │   C    │    C    │    -    │    -    │    -     │ │
│  │ Settings    │  CRUD  │   R     │   RU    │    R    │    -     │ │
│  └─────────────┴────────┴─────────┴─────────┴─────────┴──────────┘ │
│                                                                      │
│  CRUD = Create, Read, Update, Delete                                │
│  CRU  = Create, Read, Update                                        │
│  RU   = Read, Update                                                │
│  R    = Read only                                                   │
│  -    = No access                                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 10.3 Medidas de Seguridad

- **Autenticación**: JWT con refresh tokens, OAuth 2.0
- **Encriptación**: AES-256 para datos, TLS 1.3 en tránsito
- **Key Management**: AWS KMS / HashiCorp Vault
- **Rate Limiting**: Por usuario y por endpoint
- **Input Validation**: Zod/Schema validation
- **SQL Injection Prevention**: ORM (Prisma/TypeORM)
- **XSS Prevention**: Content Security Policy
- **CSRF Protection**: Anti-CSRF tokens
- **Audit Logging**: Todas las acciones críticas
- **Session Management**: Redis-backed sessions con expiración
