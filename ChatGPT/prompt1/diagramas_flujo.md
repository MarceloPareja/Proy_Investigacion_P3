# Diagramas (Mermaid) — Sistema de Gestión de Conferencias

## Contexto (C4 - System Context)
```mermaid
flowchart LR
	Author[Autor] -->|Envía paper / version final| ConfSys[Sistema de Gestión de Conferencias]
	Reviewer[Revisor] -->|Envía informes| ConfSys
	Admin[Administrador / Chair] -->|Publica CFP, asigna revisores, decide| ConfSys

	ConfSys -->|Notifica resultados y recordatorios| Email[Servicio de Notificaciones]
	ConfSys -->|Guarda estado| DB[(Base de Datos)]
	ConfSys -->|Guarda PDFs| Store[(Almacenamiento de Archivos)]
```

## Contenedores (vista lógica)
```mermaid
flowchart TB
	subgraph Client[Clientes]
		UI[Web UI]
	end

	subgraph Backend[Backend]
		API[API HTTP]
		APP[Casos de Uso]
		DOM[Dominio]
	end

	UI --> API --> APP --> DOM
	APP --> DB[(PostgreSQL)]
	APP --> Store[(Object Storage)]
	APP --> Email[Email / Scheduler]
```

## Secuencia — Envío de paper
```mermaid
sequenceDiagram
	autonumber
	actor Autor
	participant UI as Web UI
	participant API as API
	participant App as Casos de Uso
	participant DB as DB
	participant Store as File Store

	Autor->>UI: Completa formulario + adjunta PDF
	UI->>API: POST /papers (metadata + file)
	API->>App: SubmitPaper
	App->>Store: Guardar PDF
	App->>DB: Registrar Submission + Paper
	DB-->>App: OK
	App-->>API: OK (paperId)
	API-->>UI: 201 Created
	UI-->>Autor: Confirmación
```

## Secuencia — Asignación y revisión
```mermaid
sequenceDiagram
	autonumber
	actor Admin as Admin/Chair
	actor Revisor
	participant UI as Web UI
	participant API as API
	participant App as Casos de Uso
	participant DB as DB
	participant Notif as Notificaciones

	Admin->>UI: Ejecuta distribución (fecha límite alcanzada)
	UI->>API: POST /assignments/run
	API->>App: AssignReviewers
	App->>DB: Crear 3 asignaciones por paper
	App->>Notif: Notificar revisores asignados

	Revisor->>UI: Abre paper asignado
	UI->>API: GET /papers/{id}
	API->>App: GetAssignedPaper
	App->>DB: Validar asignación
	App-->>API: Paper + link

	Revisor->>UI: Envía informe
	UI->>API: POST /reviews
	API->>App: SubmitReview
	App->>DB: Guardar ReviewReport
```

## Estado — Ciclo de vida del paper
```mermaid
stateDiagram-v2
	[*] --> Received
	Received --> UnderReview: Fecha de distribución
	UnderReview --> DecisionMade: Fecha de decisión
	DecisionMade --> Rejected
	DecisionMade --> Accepted
	DecisionMade --> ChangesRequested
	ChangesRequested --> FinalReceived: Autor envía camera-ready
	Accepted --> FinalReceived: (si se requiere camera-ready)
	FinalReceived --> ProgramFinal: Construcción programa
	Rejected --> [*]
	ProgramFinal --> [*]
```

