# Sistema de Reproducción Multimedia Online (AppS + VS/AS)

## Descripción general del sistema
Sistema de reproducción multimedia donde el cliente ejecuta una aplicación que solicita y recibe video/música desde un **Servidor de Aplicación (AppS)** vía Internet. El AppS obtiene los archivos desde dos servidores dedicados: **Video Server (VS)** y **Audio Server (AS)**, potencialmente en hosts distintos.

Según el pedido del cliente, el AppS puede:
- Para video: **transcodificar formato**, ajustar calidad y/o agregar subtítulos.
- Para audio: **comprimir** y ajustar calidad.

Además existe un **sitio web de setup** para descargar el instalador. Tras instalación y registración, la app se comunica con un **Media Store** residente en AppS para comprar/descargar contenido y almacenarlo en una librería local. Dispositivos móviles pueden actualizar su librería vía USB desde una PC o descargar directo por Wi‑Fi/4G.

## Arquitectura

### Estilo recomendado
**Arquitectura orientada a servicios** centrada en un AppS que orquesta adquisición (VS/AS), procesamiento (transcoding) y entrega (stream/download).

Separación típica en AppS:
- **API/Session Service** (autenticación, sesiones, control de acceso).
- **Playback Orchestrator** (coordina VS/AS y el pipeline de transformación).
- **Transcoding Service** (workers para calidad/formato/subtítulos/compresión).
- **Media Store Service** (catálogo, compras, descargas, librería).

### Patrones
- **Proxy/Gateway**: AppS como punto único para clientes.
- **Pipelines**: etapas de procesamiento multimedia.
- **Asincronía** (jobs) para transcodificación pesada.
- **Circuit Breaker / Retries** para dependencias VS/AS.

### Tecnologías (sugerencia)
- AppS: Java/Spring Boot o Node.js (API) + workers (Go/Java/Python) para jobs.
- Procesamiento media: FFmpeg como herramienta de referencia.
- Storage: object storage para archivos y artefactos transcodificados.
- DB: PostgreSQL (usuarios, catálogo, compras, librerías).
- Cache: Redis (sesiones, tokens de descarga).

## Componentes principales

### Clientes
- App de reproducción (desktop/móvil/tablet): streaming y descargas.
- Sitio de setup: descarga instalador desde AppS.

### Servidores
- **AppS**:
	- API (login/registro, playback, store)
	- Orquestador
	- Transcoding workers
	- Media Store (catálogo + compras + librería)
- **VS**: servidor dedicado de video.
- **AS**: servidor dedicado de audio.

## Flujo de datos

### Streaming playback
1. Cliente solicita reproducir un item (video/audio).
2. AppS consulta VS/AS para obtener origen.
3. AppS decide transformaciones según pedido (formato/calidad/subtítulos).
4. AppS entrega stream al cliente (segmentos/chunks).

### Compra y descarga (Media Store)
1. Cliente autentica/autoriza compra.
2. AppS registra compra y habilita descarga.
3. Cliente descarga y guarda en su librería.

### Sincronización a móvil
- **USB**: copia desde librería en PC a dispositivo.
- **Inalámbrico (Wi‑Fi/4G)**: descarga directa desde Media Store.

## Decisiones técnicas
- **AppS como orquestador** para ocultar VS/AS y concentrar reglas de negocio.
- **Transcodificación**: síncrona para contenidos pequeños, asíncrona para pesados.
- **Subtítulos**: tratarlos como artefacto asociado al video y parte del pipeline.
- **Persistencia**: catálogo/compras/librerías en DB; binarios en object storage.
- **Resiliencia**: timeouts y reintentos ante VS/AS; control de concurrencia en workers.

