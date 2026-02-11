# Diagramas (Mermaid) — Multimedia Online

## Contexto
```mermaid
flowchart LR
	User[Usuario] --> Client[App Cliente]
	User --> Setup[Web Setup]

	Setup --> AppS[AppS]
	Client --> AppS

	AppS --> VS[Video Server (VS)]
	AppS --> AS[Audio Server (AS)]
	AppS --> Store[Media Store (en AppS)]
	AppS --> DB[(DB)]
	AppS --> Obj[(Object Storage)]
```

## Contenedores (AppS)
```mermaid
flowchart TB
	subgraph AppS[AppS]
		API[API + Auth]
		Orch[Playback Orchestrator]
		Trans[Transcoding Workers]
		MS[Media Store Service]
	end

	Client[Cliente] --> API
	API --> Orch
	Orch --> VS[VS]
	Orch --> AS[AS]
	Orch --> Trans
	Trans --> Obj[(Artefactos transcodificados)]
	API --> MS
	MS --> DB[(Catálogo/Compras/Librería)]
	MS --> Obj
```

## Secuencia — Reproducir video con subtítulos
```mermaid
sequenceDiagram
	autonumber
	actor Usuario
	participant Client as App Cliente
	participant API as AppS API
	participant Orch as Orquestador
	participant VS as Video Server
	participant AS as Audio Server
	participant Trans as Transcoding

	Usuario->>Client: Selecciona contenido + calidad + subtítulos
	Client->>API: POST /play (item, params)
	API->>Orch: StartPlayback
	Orch->>VS: FetchVideo(item)
	Orch->>AS: FetchAudio(item)
	Orch->>Trans: Transcode/OverlaySubs(params)
	Trans-->>Orch: Stream preparado
	Orch-->>API: Stream handle
	API-->>Client: Stream (chunks)
```

## Secuencia — Compra y descarga
```mermaid
sequenceDiagram
	autonumber
	actor Usuario
	participant Client as App Cliente
	participant API as AppS API
	participant Store as Media Store
	participant DB as DB
	participant Obj as Object Storage

	Usuario->>Client: Comprar contenido
	Client->>API: POST /store/purchase
	API->>Store: Purchase
	Store->>DB: Registrar compra + permisos
	Store-->>API: OK
	Client->>API: GET /store/download
	API->>Obj: Obtener archivo
	Obj-->>Client: Descarga
```

