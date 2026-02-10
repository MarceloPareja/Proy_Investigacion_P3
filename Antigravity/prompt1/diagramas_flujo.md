
# Diagramas de Flujo y Procesos

Estos diagramas detallan los flujos de trabajo principales del sistema de conferencias, utilizando diagramas de secuencia y de estado UML.

## 1. Diagrama de Secuencia: Envío de Papers (Submission Phase)

Este flujo describe cómo un autor envía su trabajo inicial al sistema.

```mermaid
sequenceDiagram
    autonumber
    actor Autor
    participant WebApp as Frontend
    participant API as Backend API
    participant DB as Base de Datos
    participant FS as File Storage
    participant Email as Servicio de Email

    Note over Autor, Email: Fase de "Call for Papers"

    Autor->>WebApp: Iniciar envío de paper
    WebApp->>Autor: Mostrar formulario de envío (título, abstract, autores)
    Autor->>WebApp: Completar datos y adjuntar PDF
    WebApp->>API: POST /submissions (metadata + file)
    
    activate API
    API->>FS: Guardar archivo PDF
    FS-->>API: Retorna fileURL
    API->>DB: Crear registro de Paper (Status: SUBMITTED)
    DB-->>API: Retorna paperID
    
    par Notificación Asíncrona
        API->>Email: Encolar "EmailConfirmacionEnvio"
        Email->>Autor: Enviar correo de confirmación
    end

    API-->>WebApp: Retorna éxito (201 Created)
    deactivate API
    WebApp->>Autor: Mostrar confirmación en pantalla
```

## 2. Diagrama de Secuencia: Asignación y Revisión (Review Phase)

El administrador (Chair) asigna revisores y estos evalúan los papers.

```mermaid
sequenceDiagram
    autonumber
    actor Chair as Admin/Chair
    participant API as Backend API
    participant DB as Base de Datos
    actor Revisor
    participant Email as Servicio de Email

    Note over Chair, Revisor: Fase de Revisión

    %% Asignación
    Chair->>API: Asignar revisores a Paper X (Revisor A, B, C)
    activate API
    API->>DB: Crear asignaciones (ReviewAssignment)
    API->>Email: Notificar a Revisor (Nuevo paper asignado)
    deactivate API
    Email->>Revisor: Correo: "Tienes un nuevo paper para revisar"

    %% Revisión
    Revisor->>API: Obtener detalles del Paper X
    API->>DB: Consultar Paper y Archivo
    DB-->>API: Datos del paper
    API-->>Revisor: Mostrar paper y link de descarga

    Revisor->>API: Enviar evaluación (Score, Comentarios, Recomendación)
    activate API
    API->>DB: Guardar Review (Status: COMPLETED)
    API->>DB: Verificar si el paper tiene todas las revisiones (3/3)
    
    alt Todas las revisiones completas
        API->>DB: Actualizar estado del Paper (Status: REVIEWED)
        API->>Chair: Notificar (Paper listo para decisión)
    end
    deactivate API
```

## 3. Diagrama de Secuencia: Selección y Notificación (Decision Phase)

El Chair toma la decisión final basada en las revisiones y notifica a los autores.

```mermaid
sequenceDiagram
    autonumber
    actor Chair as Admin/Chair
    participant API as Backend API
    participant DB as Base de Datos
    participant Email as Servicio de Email
    actor Autor

    Note over Chair, Autor: Fase de Selección

    Chair->>API: Obtener lista de papers revisados
    API->>DB: Query papers con reviews completas
    DB-->>API: Lista de papers + Scores promedio
    API-->>Chair: Mostrar ranking de papers

    loop Para cada paper seleccionado
        Chair->>API: Marcar decisión (ACEPTADO / RECHAZADO / CAMBIOS_REQUERIDOS)
        activate API
        API->>DB: Actualizar estado del Paper
        deactivate API
    end

    Note right of Chair: Publicación de resultados

    Chair->>API: Publicar resultados (Trigger Notification)
    activate API
    API->>DB: Obtener todos los autores y decisiones
    
    loop Para cada autor
        API->>Email: Generar correo de resultado
        Email->>Autor: Enviar notificación (Aceptado/Rechazado)
    end
    deactivate API
```

## 4. Diagrama de Estado: Ciclo de Vida del Paper

Muestra los estados por los que pasa un paper desde su creación hasta la decisión final.

```mermaid
stateDiagram-v2
    [*] --> Draft: Autor inicia envío
    Draft --> Submitted: Autor confirma envío
    
    state "Revisión" as ReviewPhase {
        Submitted --> UnderReview: Asignación de revisores
        UnderReview --> Reviewed: Todas las revisiones completadas
    }

    Reviewed --> DecisionPending: Cierre de fase de revisión
    
    DecisionPending --> Accepted: Chair acepta el paper
    DecisionPending --> Rejected: Chair rechaza el paper
    DecisionPending --> RevisionRequired: Chair solicita cambios
    
    RevisionRequired --> Revised: Autor sube nueva versión
    Revised --> UnderReview: Vuelve a revisión (o decisión directa)
    
    Accepted --> CameraReady: Autor sube versión final
    CameraReady --> Published: Programa final generado
    
    Published --> [*]
    Rejected --> [*]
```
