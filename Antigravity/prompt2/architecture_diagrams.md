# Diagramas de Arquitectura: Servicio de Calendario

Este documento contiene los diagramas arquitectónicos generados con Mermaid.js para visualizar la estructura y el comportamiento del sistema.

## 1. Diagrama de Contexto del Sistema (System Context)
Muestra cómo los usuarios interactúan con el sistema a través de diferentes plataformas.

```mermaid
C4Context
    title Diagrama de Contexto - Servicio de Calendario
    
    Person(user, "Usuario", "Empleado de la multinacional que usa el calendario.")
    
    System_Boundary(apps, "Aplicaciones Cliente") {
        System(web, "Aplicación Web", "Interfaz accesible vía navegador.")
        System(mobile, "App Móvil/Tablet", "Aplicación nativa para iOS/Android.")
    }
    
    System(calendar_system, "Sistema Backend de Calendario", "Gestiona eventos, usuarios, seguridad y notificaciones.")
    
    Rel(user, web, "Usa", "HTTPS")
    Rel(user, mobile, "Usa", "Touch/Gestos")
    Rel(web, calendar_system, "API Calls", "JSON/HTTPS")
    Rel(mobile, calendar_system, "API Calls", "JSON/HTTPS")
```

## 2. Diagrama de Contenedores (Container Diagram)
Detalla la arquitectura de microservicios propuesta.

```mermaid
C4Container
    title Diagrama de Contenedores - Backend de Microservicios
    
    Container(api_gateway, "API Gateway", "Go / Kong", "Punto de entrada único. Enrutamiento, Rate Limiting, SSL Termination.")
    
    System_Boundary(services, "Microservicios") {
        Container(auth_service, "Auth Service", "Node.js / TypeScript", "Maneja login, tokens JWT y roles.")
        Container(calendar_core, "Calendar Core Service", "Go / Rust", "Lógica de negocio. Maneja creación y lectura de eventos. **Realiza la desencriptación.**")
        Container(notification_service, "Notification Service", "Node.js", "Envía emails y notificaciones push.")
    }
    
    System_Boundary(data, "Persistencia") {
        ContainerDb(user_db, "User Database", "PostgreSQL", "Almacena perfiles y credenciales (hased).")
        ContainerDb(calendar_db, "Calendar Database", "PostgreSQL / MongoDB", "Almacena eventos de calendario **ENCRIPTADOS**.")
        ContainerDb(queue, "Event Bus", "RabbitMQ / Kafka", "Cola de mensajes para desacoplar notificaciones.")
    }
    
    Rel(api_gateway, auth_service, "Valida Auth", "gRPC")
    Rel(api_gateway, calendar_core, "Proxies Requests", "REST/gRPC")
    Rel(calendar_core, calendar_db, "Lee/Escribe (Encriptado)", "TCP/IP")
    Rel(calendar_core, queue, "Publica Eventos (Created/Updated)", "AMQP")
    Rel(queue, notification_service, "Consume Eventos", "AMQP")
    Rel(auth_service, user_db, "Lee/Escribe", "SQL")
```

## 3. Diagrama de Componentes (Component Diagram)
Enfocado en la estructura interna del servicio de calendario y su módulo de seguridad.

```mermaid
classDiagram
    class CalendarController {
        +getEvents(userId, dateRange)
        +createEvent(userId, eventData)
    }
    
    class SecurityModule {
        <<Service>>
        -MasterKey
        +encrypt(data, userKey)
        +decrypt(data, userKey)
    }
    
    class CalendarService {
        +retrieveAndDecrypt(userId)
        +validatePermissions(userId, calendarId)
    }
    
    class Repository {
        +saveEncrypted(blob)
        +findEncrypted(userId)
    }
    
    CalendarController --> CalendarService : Invoca
    CalendarService --> SecurityModule : Usa para desencriptar
    CalendarService --> Repository : Usa para persistir
```

## 4. Diagrama de Secuencia: Flujo de Visualización Seguro
Muestra el proceso crítico de lectura y desencriptación.

```mermaid
sequenceDiagram
    participant U as Usuario
    participant App as App Cliente
    participant GW as API Gateway
    participant Auth as Auth Service
    participant Cal as Calendar Service
    participant DB as Database

    U->>App: Abrir Calendario
    App->>GW: GET /api/v1/events (Token)
    GW->>Auth: Validar Token & Permisos
    Auth-->>GW: Token Válido (User ID)
    GW->>Cal: Solicitar Eventos (User ID)
    
    Note over Cal, DB: Inicio de Fase Segura
    Cal->>DB: Query eventos encriptados
    DB-->>Cal: Retorna datos cifrados (Blob)
    Cal->>Cal: Desencriptar datos (AES-GCM)
    Cal-->>GW: Retorna eventos planos (JSON)
    Note over Cal, DB: Fin de Fase Segura
    
    GW-->>App: Respuesta JSON
    App-->>U: Muestra Calendario
```
