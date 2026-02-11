# Arquitectura del Sistema de Reproducción de Multimedia Online

## 1. Visión General

Sistema de streaming multimedia que permite a usuarios reproducir, comprar y gestionar contenido de video y audio. El sistema incluye transcodificación en tiempo real, sincronización entre dispositivos, y soporte para múltiples plataformas (escritorio, móvil, web).

## 2. Requisitos del Sistema

### 2.1 Requisitos Funcionales

| Requisito | Descripción |
|-----------|-------------|
| RF-01 | Reproducción de video y audio desde servidores remotos |
| RF-02 | Servidor de aplicación (AppS) como orquestador central |
| RF-03 | Servidores dedicados de video (VS) y audio (AS) |
| RF-04 | Transcodificación de video (formatos, calidad, subtítulos) |
| RF-05 | Compresión y modificación de calidad de audio |
| RF-06 | Media Store para compra y descarga de contenido |
| RF-07 | Gestión de librería personal de medios |
| RF-08 | Sincronización entre dispositivos (USB, Wi-Fi, 4G) |
| RF-09 | Aplicación cliente multiplataforma |

### 2.2 Requisitos No Funcionales

| Requisito | Descripción |
|-----------|-------------|
| RNF-01 | Streaming con latencia mínima (< 2 segundos) |
| RNF-02 | Soporte para 4K, HDR, Dolby Atmos |
| RNF-03 | Escalabilidad para millones de usuarios concurrentes |
| RNF-04 | Disponibilidad 99.9% |
| RNF-05 | DRM para contenido protegido |
| RNF-06 | Cacheo inteligente para reproducción offline |

---

## 3. Stack Tecnológico Recomendado

### 3.1 Backend Principal

**Recomendación: Node.js con TypeScript + Go para servicios de alto rendimiento**

**Justificación:**
- Node.js: Excelente para APIs REST, WebSockets, y lógica de negocio
- Go: Ideal para servicios de transcodificación y streaming de alto rendimiento
- TypeScript: Tipado estático para mantenibilidad del código

**Alternativas:**
- **Java/Spring Boot**: Robusto pero más pesado
- **Python/FastAPI**: Buena alternativa con excelente soporte para IA
- **C++**: Para componentes críticos de rendimiento

### 3.2 Servidores de Media

| Componente | Tecnología | Justificación |
|------------|-----------|---------------|
| **Video Server** | Nginx + RTMP/HLS | Streaming de video optimizado |
| **Audio Server** | Icecast + HTTP Live Streaming | Streaming de audio de baja latencia |
| **Transcoding** | FFmpeg + GPU acceleration | Transcodificación rápida |
| **Storage** | AWS S3 + CloudFront CDN | Almacenamiento y distribución global |
| **Database** | PostgreSQL + Redis | Datos estructurados + cache |

### 3.3 Aplicación Cliente

| Plataforma | Tecnología | Uso |
|-----------|-----------|-----|
| **Windows/Mac** | Electron + React | Aplicación desktop |
| **iOS** | Swift | Aplicación iOS nativa |
| **Android** | Kotlin/Jetpack Compose | Aplicación Android nativa |
| **Web** | React + WebRTC | Reproductor web |

### 3.4 Herramientas de Media

| Herramienta | Función |
|-------------|---------|
| **FFmpeg** | Transcodificación de video/audio |
| **GStreamer** | Pipeline de media processing |
| **HandBrake** | Encoding de video |
| **Widevine/PlayReady** | DRM para contenido protegido |
| **HLS.js/video.js** | Reproductor web |

---

## 4. Arquitectura General del Sistema

```
┌─────────────────────────────────────────────────────────────────────┐
│                          CLIENT APPLICATIONS                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │
│  │   Desktop  │  │    Web      │  │   Mobile    │                 │
│  │  (Windows) │  │  (Browser)  │  │  (iOS/And)  │                 │
│  │            │  │             │  │             │                 │
│  │ Electron   │  │  React      │  │ Swift/Kotlin│                 │
│  │  App      │  │  Player     │  │   Native    │                 │
│  └─────┬─────┘  └──────┬──────┘  └──────┬──────┘                 │
│        │               │                 │                          │
│        └───────────────┼─────────────────┘                          │
│                        │                                            │
│                        ▼                                            │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                    CLIENT MEDIA PLAYER                       │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │  │
│  │  │   Video     │  │   Audio     │  │   DRM       │         │  │
│  │  │  Decoder    │  │  Decoder    │  │  Module     │         │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘         │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │  │
│  │  │   Cache     │  │   Buffer    │  │   Network   │         │  │
│  │  │  Manager    │  │  Manager    │  │  Handler    │         │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘         │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                        │                                            │
└────────────────────────┼──────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       API GATEWAY / LOAD BALANCER                    │
├─────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │  Kong API Gateway / AWS API Gateway                         │  │
│  │  - Rate Limiting                                           │  │
│  │  - Authentication                                          │  │
│  │  - Request Routing                                        │  │
│  │  - SSL Termination                                         │  │
│  └─────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
         ▼               ▼               ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│  AppS Server   │ │  CDN Edge      │ │  Auth Service   │
│  (Application  │ │  (CloudFront)  │ │  (OAuth/JWT)    │
│   Server)      │ │                │ │                 │
└───────┬─────────┘ └─────────────────┘ └────────┬────────┘
        │                                         │
        │    ┌────────────────────────────────────┘
        │    │
        ▼    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       AppS - APPLICATION SERVER                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    CORE SERVICES                            │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │   │
│  │  │   Media     │  │  Transcode │  │   Store     │        │   │
│  │  │  Catalog    │  │  Service   │  │  Service    │        │   │
│  │  │  Service    │  │            │  │            │        │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │   │
│  │  │   User      │  │   Sync      │  │   Purchase  │        │   │
│  │  │  Library    │  │  Service   │  │  Service    │        │   │
│  │  │  Service    │  │            │  │            │        │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │   │
│  │  │   Streaming │  │  Download   │  │   DRM      │        │   │
│  │  │  Service    │  │  Service   │  │  Service    │        │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────┬───────────────────────────────┘
                                      │
              ┌───────────────────────┼───────────────────────┐
              │                       │                       │
              ▼                       ▼                       ▼
┌─────────────────────┐   ┌─────────────────────┐   ┌─────────────────────┐
│      Video Server   │   │     Audio Server    │   │   Transcode Farm    │
│        (VS)         │   │       (AS)          │   │   (GPU Cluster)     │
│                     │   │                     │   │                     │
│  ┌─────────────────┐│   │  ┌─────────────────┐│   │  ┌─────────────────┐│
│  │   Nginx RTMP   ││   │  │   Icecast/     ││   │  │   FFmpeg        ││
│  │   HLS/DASH    ││   │  │   HLS Server   ││   │  │   GPU Workers   ││
│  └────────┬────────┘│   │  └────────┬───────┘│   │  └────────┬──────┘│
│           │         │   │           │        │   │           │       │
│           ▼         │   │           ▼        │   │           ▼       │
│  ┌─────────────────┐│   │  ┌─────────────────┐│   │  ┌─────────────────┐│
│  │  Origin Server  ││   │  │  Audio Storage ││   │  │   Queue         ││
│  │  (Segmented)   ││   │  │  (Flac/MP3)     ││   │  │   (RabbitMQ/    ││
│  └────────┬────────┘│   │  └────────┬───────┘│   │  │    Kafka)       ││
│           │         │   │           │        │   │  └────────┬──────┘│
│           ▼         │   │           ▼        │   │           ▼       │
│  ┌─────────────────┐│   │  ┌─────────────────┐│   │  ┌─────────────────┐│
│  │  CDN Integration│   │  │  CDN Integration│   │  │   Output        ││
│  │  (HLS/DASH)    ││   │  │  (HLS)         ││   │  │   Storage       ││
│  └─────────────────┘│   │  └─────────────────┘│   │  └─────────────────┘│
└─────────────────────┘   └─────────────────────┘   └─────────────────────┘
```

---

## 5. Patrones Arquitectónicos

### 5.1 Patrón Principal: Microservicios

```
┌─────────────────────────────────────────────────────────────────┐
│                    MICROSERVICIOS ARCHITECTURE                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌───────────────┐   ┌───────────────┐   ┌───────────────┐     │
│  │    Media      │   │   Transcode   │   │   Store       │     │
│  │   Catalog     │   │   Service     │   │   Service     │     │
│  │   Service     │   │               │   │               │     │
│  │               │   │  - FFmpeg     │   │  - Catalog    │     │
│  │  - Search     │   │  - GPU Queue  │   │  - Metadata   │     │
│  │  - Metadata   │   │  - Adaptive   │   │  - Purchase   │     │
│  │  - Discovery │   │    Bitrate     │   │  - DRM        │     │
│  └───────┬───────┘   └───────┬───────┘   └───────┬───────┘     │
│          │                   │                   │              │
│  ┌───────┴───────┐   ┌───────┴───────┐   ┌───────┴───────┐     │
│  │    User      │   │    Sync       │   │  Streaming    │     │
│  │   Library    │   │   Service     │   │  Service      │     │
│  │   Service    │   │               │   │               │     │
│  │               │   │  - Library   │   │  - HLS/DASH  │     │
│  │  - Library   │   │    Sync      │   │  - RTMP      │     │
│  │    CRUD      │   │  - USB Sync  │   │  - Segments  │     │
│  │  - Playlists │   │  - Conflict  │   │  - Adaptive  │     │
│  │              │   │    Resolution│   │    Streaming │     │
│  └───────────────┘   └───────────────┘   └───────────────┘     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Patrones Complementarios

#### a) API Gateway Pattern
Centralización de la comunicación con los clientes

```typescript
class ApiGateway {
    routes = {
        '/api/media/*': 'media-service',
        '/api/stream/*': 'streaming-service',
        '/api/library/*': 'library-service',
        '/api/store/*': 'store-service',
        '/api/sync/*': 'sync-service'
    }
    
    async handleRequest(request: Request): Promise<Response> {
        // Rate limiting, auth, routing
    }
}
```

#### b) CQRS Pattern
Separación de operaciones de lectura y escritura

```
┌─────────────────────────────────────────┐
│           COMMANDS (Writes)              │
│  ┌─────────────────────────────────┐   │
│  │ - Purchase media                │   │
│  │ - Upload media                  │   │
│  │ - Sync library                  │   │
│  │ - Create playlist               │   │
│  │ - Update preferences            │   │
│  └─────────────────────────────────┘   │
│                    │                    │
│                    ▼                    │
│            ┌─────────────┐              │
│            │  Command    │              │
│            │  Processor   │              │
│            └─────────────┘              │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│             EVENTS (Domain Events)       │
│  ┌─────────────────────────────────┐   │
│  │ - MediaPurchased                │   │
│  │ - MediaDownloaded               │   │
│  │ - LibrarySynced                  │   │
│  │ - PlaylistCreated               │   │
│  └─────────────────────────────────┘   │
│                    │                    │
│                    ▼                    │
│            ┌─────────────┐              │
│            │  Event Bus  │              │
│            │  (Kafka)    │              │
│            └─────────────┘              │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│              QUERIES (Reads)             │
│  ┌─────────────────────────────────┐   │
│  │ - Get media catalog             │   │
│  │ - Get streaming URL             │   │
│  │ - Get library contents          │   │
│  │ - Get playlists                 │   │
│  │ - Search media                  │   │
│  └─────────────────────────────────┘   │
│                    │                    │
│            ┌─────────────┐              │
│            │  Read DB    │              │
│            │  (Optimized)│              │
│            └─────────────┘              │
└─────────────────────────────────────────┘
```

#### c) Event Sourcing Pattern
Trazabilidad de cambios en la librería del usuario

```typescript
interface MediaEvent {
    type: 'PURCHASED' | 'DOWNLOADED' | 'SYNCED' | 'PLAYED';
    mediaId: string;
    userId: string;
    timestamp: Date;
    metadata: Record<string, any>;
}

class EventStore {
    async append(event: MediaEvent): Promise<void>;
    async getEvents(userId: string): Promise<MediaEvent[]>;
    async replay(userId: string): Promise<LibraryState>;
}
```

#### d) Publisher-Subscriber Pattern
Notificaciones en tiempo real

```typescript
class MediaEventPublisher {
    private subscribers: Map<string, Set<Subscriber>> = new Map();
    
    subscribe(eventType: string, subscriber: Subscriber): void;
    unsubscribe(eventType: string, subscriber: Subscriber): void;
    publish(event: MediaEvent): void;
}

// Uso para sincronización en tiempo real
publisher.subscribe('LIBRARY_UPDATED', wsClient);
```

#### e) Circuit Breaker Pattern
Tolerancia a fallos en servicios externos

```typescript
class StreamingService {
    @CircuitBreaker({ timeout: 5000, errorThreshold: 50 })
    async getStreamUrl(mediaId: string): Promise<StreamUrl> {
        // Fallback: usar cache o CDN alternativo
    }
}
```

#### f) Repository Pattern
Abstracción del acceso a datos de media

```typescript
interface IMediaRepository {
    findById(id: string): Promise<MediaItem>;
    findByGenre(genre: string): Promise<MediaItem[]>;
    search(query: string): Promise<MediaItem[]>;
    getPopular(limit: number): Promise<MediaItem[]>;
}
```

---

## 6. Pipeline de Transcodificación

### 6.1 Flujo de Transcodificación

```
┌─────────────────────────────────────────────────────────────────────┐
│                    TRANSCODING PIPELINE                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  INPUT SOURCES                                                      │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │  │
│  │  │  Raw Video  │  │  Encoded    │  │   DVD/Blu   │        │  │
│  │  │   (RAW)     │  │   (ProRes)  │  │    Ray      │        │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                         │                                            │
│                         ▼                                            │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                   INGESTION                                 │  │
│  │  ┌─────────────────────────────────────────────────────┐    │  │
│  │  │  - Validate input format                            │    │  │
│  │  │  - Extract metadata                                 │    │  │
│  │  │  - Generate thumbnails                               │    │  │
│  │  │  - Extract audio track                             │    │  │
│  │  │  - Extract subtitle tracks                         │    │  │
│  │  └─────────────────────────────────────────────────────┘    │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                         │                                            │
│                         ▼                                            │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                   TRANSCODE WORKERS                         │  │
│  │                                                               │  │
│  │  ┌─────────────────┐  ┌─────────────────┐                   │  │
│  │  │   GPU Worker   │  │   CPU Worker   │                   │  │
│  │  │  1             │  │  1             │                   │  │
│  │  │  - 4K HDR      │  │  - 1080p       │                   │  │
│  │  │  - HEVC        │  │  - H.264       │                   │  │
│  │  │  - Dolby Vision│  │  - VP9         │                   │  │
│  │  └─────────────────┘  └─────────────────┘                   │  │
│  │  ┌─────────────────┐  ┌─────────────────┐                   │  │
│  │  │   GPU Worker   │  │   CPU Worker   │                   │  │
│  │  │  2             │  │  2             │                   │  │
│  │  │  - 1080p HDR   │  │  - 720p        │                   │  │
│  │  │  - HEVC        │  │  - H.264       │                   │  │
│  │  └─────────────────┘  └─────────────────┘                   │  │
│  │                                                               │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                         │                                            │
│                         ▼                                            │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │              OUTPUT FORMATS                                  │  │
│  │                                                               │  │
│  │  VIDEO OUTPUTS:                                               │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │  │
│  │  │   4K HDR    │  │   1080p     │  │   720p      │        │  │
│  │  │  (HEVC)     │  │  (HEVC)     │  │  (H.264)    │        │  │
│  │  │  - Master   │  │  - Primary  │  │  - Mobile   │        │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │  │
│  │                                                               │  │
│  │  AUDIO OUTPUTS:                                               │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │  │
│  │  │  Dolby      │  │  Stereo     │  │  AAC        │        │  │
│  │  │  Atmos      │  │  (FLAC)     │  │  (Compressed│        │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │  │
│  │                                                               │  │
│  │  SUBTITLE OUTPUTS:                                           │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │  │
│  │  │  WebVTT     │  │    SRT      │  │  Embedded   │        │  │
│  │  │  (HLS)      │  │  (Optional) │  │  ( burned)  │        │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │  │
│  │                                                               │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                         │                                            │
│                         ▼                                            │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                   PACKAGING                                  │  │
│  │  ┌─────────────────────────────────────────────────────┐    │  │
│  │  │  - Fragment media into segments (2-10 seconds)      │    │  │
│  │  │  - Generate HLS (.m3u8) or DASH (.mpd) manifests     │    │  │
│  │  │  - Generate encryption keys (DRM)                    │    │  │
│  │  │  - Create multiple quality renditions                │    │  │
│  │  │  - Generate thumbnail sprites                        │    │  │
│  │  └─────────────────────────────────────────────────────┘    │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                         │                                            │
│                         ▼                                            │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                   CDN DISTRIBUTION                          │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │  │
│  │  │  Origin     │  │   Edge      │  │   Cache     │        │  │
│  │  │  Storage    │  │   Servers   │  │   Pop       │        │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 6.2 Configuración de Transcodificación

```typescript
interface TranscodeConfig {
    // Video settings
    video: {
        formats: VideoFormat[];
        resolutions: Resolution[];
        codec: Codec;
        bitrate: BitrateMode;
        fps: number;
        hdr: boolean;
    };
    
    // Audio settings
    audio: {
        formats: AudioFormat[];
        channels: number;
        bitrate: BitrateMode;
        compression: boolean;
    };
    
    // Subtitle settings
    subtitle: {
        formats: SubtitleFormat[];
        burning: boolean;
        languages: string[];
    };
}

// Example: Adaptive Bitrate Streaming
const streamingConfig: TranscodeConfig = {
    video: {
        formats: ['mp4', 'webm'],
        resolutions: [
            { width: 3840, height: 2160, bitrate: 15000, name: '4K' },
            { width: 1920, height: 1080, bitrate: 8000, name: '1080p' },
            { width: 1280, height: 720, bitrate: 5000, name: '720p' },
            { width: 854, height: 480, bitrate: 2500, name: '480p' }
        ],
        codec: 'hevc',
        bitrate: 'vbr',
        fps: 60,
        hdr: true
    },
    audio: {
        formats: ['aac', 'flac'],
        channels: 6,  // 5.1 surround
        bitrate: 'vbr',
        compression: true
    },
    subtitle: {
        formats: ['vtt', 'srt'],
        burning: false,
        languages: ['en', 'es', 'fr']
    }
};
```

---

## 7. Modelo de Datos

### 7.1 Entidades Principales

```
┌─────────────────────────────────────────────────────────────┐
│                      CORE ENTITIES                           │
├─────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐     │
│  │    MediaItem    │  │   Transcode    │  │   Streaming    │     │
│  │                 │  │   Rendition    │  │   Session      │     │
│  │  - id: UUID     │  │  - id: UUID    │  │  - id: UUID    │     │
│  │  - title: String│  │  - media_id   │  │  - user_id    │     │
│  │  - type: Enum  │  │  - quality    │  │  - media_id    │     │
│  │  - duration    │  │  - format     │  │  - position   │     │
│  │  - metadata    │  │  - bitrate    │  │  - start_time │     │
│  │  - thumbnail   │  │  - codec      │  │  - end_time   │     │
│  │  - price       │  │  - url        │  │  - completed  │     │
│  │  - drm_info    │  │  - file_size  │  │  - device_id  │     │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘     │
│                                                                      │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐     │
│  │   UserLibrary  │  │    Playlist    │  │   Download     │     │
│  │                 │  │                 │  │   Task         │     │
│  │  - id: UUID     │  │  - id: UUID    │  │  - id: UUID    │     │
│  │  - user_id     │  │  - user_id    │  │  - user_id    │     │
│  │  - items: []   │  │  - name       │  │  - media_id    │     │
│  │  - purchased   │  │  - items: []  │  │  - quality    │     │
│  │  - downloaded  │  │  - is_public  │  │  - format     │     │
│  │  - synced      │  │  - created_at │  │  - progress   │     │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘     │
│                                                                      │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐     │
│  │   Device       │  │   SyncTask     │  │   Purchase     │     │
│  │                 │  │                 │  │                 │     │
│  │  - id: UUID     │  │  - id: UUID    │  │  - id: UUID    │     │
│  │  - user_id     │  │  - source_dev  │  │  - user_id    │     │
│  │  - type: Enum  │  │  - target_dev │  │  - media_id    │     │
│  │  - library_id  │  │  - items: []  │  │  - price      │     │
│  │  - last_sync   │  │  - status     │  │  - payment_id │     │
│  │  - storage_cap │  │  - progress   │  │  - created_at │     │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘     │
│                                                                      │
└─────────────────────────────────────────────────────────────┘
```

---

## 8. Sistema de Sincronización

### 8.1 Flujo de Sincronización

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DEVICE SYNCHRONIZATION                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  SYNC MODES:                                                        │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                                                               │   │
│  │  1. CLOUD SYNC (Wi-Fi, 4G)                                  │   │
│  │     ┌─────────────────────────────────────────────────────┐ │   │
│  │     │  Device ──► Internet ──► Cloud Server ──► Devices   │ │   │
│  │     │    │              │                                   │ │   │
│  │     │    └──────────────┘                                   │ │   │
│  │     │              │                                         │ │   │
│  │     │         ┌────┴────┐                                   │ │   │
│  │     │         │ Library │                                   │ │   │
│  │     │         │  State  │                                   │ │   │
│  │     │         │  Sync   │                                   │ │   │
│  │     │         └─────────┘                                   │ │   │
│  │     └─────────────────────────────────────────────────────┘ │   │
│  │                                                               │   │
│  │  2. USB SYNC (Mobile ↔ PC)                                  │   │
│  │     ┌─────────────────────────────────────────────────────┐ │   │
│  │     │  Mobile Device ──USB Cable──► PC Application       │ │   │
│  │     │         │                                    │       │ │   │
│  │     │         │                                    ▼       │ │   │
│  │     │         │                            ┌───────────┐  │ │   │
│  │     │         │                            │  Library  │  │ │   │
│  │     │         │                            │  Compare  │  │ │   │
│  │     │         │                            │  & Merge  │  │ │   │
│  │     │         │                            └─────┬─────┘  │ │   │
│  │     │         │                                  │        │ │   │
│  │     │         │          ┌───────────────────────┼──────┐ │ │   │
│  │     │         │          │                       │      │ │ │   │
│  │     │         ▼          ▼                       ▼      │ │ │   │
│  │     │    ┌─────────────────────────────────────────┐   │ │ │   │
│  │     │    │         Bidirectional Sync              │   │ │ │   │
│  │     │    │  - Merge changes                        │   │ │ │   │
│  │     │    │  - Resolve conflicts                    │   │ │ │   │
│  │     │    │  - Update metadata                      │   │ │ │   │
│  │     │    └─────────────────────────────────────────┘   │ │ │   │
│  │     │                        │                         │ │ │   │
│  │     └────────────────────────┼─────────────────────────┘ │ │   │
│  │                              │                           │ │   │
│  │                              ▼                           │ │   │
│  │     ┌─────────────────────────────────────────────────┐ │ │   │
│  │     │              USB STORAGE TRANSFER              │ │ │   │
│  │     │  - Copy media files to device storage          │ │ │   │
│  │     │  - Update library database on device           │ │ │   │
│  │     │  - Verify integrity (checksum)                 │ │ │   │
│  │     └─────────────────────────────────────────────────┘ │ │   │
│  │                                                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  CONFLICT RESOLUTION STRATEGIES:                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  1. Last Write Wins (timestamp)                             │    │
│  │  2. User Selection (prompt for conflicts)                    │    │
│  │  3. Automatic Merge (metadata only, no data)                │    │
│  │  4. Keep Both (create duplicates with suffix)                │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 9. Protocolos de Streaming

### 9.1 Comparación de Protocolos

```
┌─────────────────────────────────────────────────────────────────────┐
│                    STREAMING PROTOCOLS                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────┬────────────┬────────────┬────────────┬───────────┐│
│  │  Protocol   │  Latency   │  Adaptive  │   DRM      │  Browser  ││
│  │              │            │  Bitrate   │   Support  │  Support  ││
│  ├─────────────┼────────────┼────────────┼────────────┼───────────┤│
│  │   HLS       │  10-30s    │    Yes     │    Yes     │    Yes    ││
│  │  (HTTP)     │            │            │            │           ││
│  ├─────────────┼────────────┼────────────┼────────────┼───────────┤│
│  │   DASH      │  10-30s    │    Yes     │    Yes     │    Yes    ││
│  │  (HTTP)     │            │            │            │           ││
│  ├─────────────┼────────────┼────────────┼────────────┼───────────┤│
│  │   RTMP      │   2-5s     │     No     │    Limited │    No     ││
│  │  (Flash)    │            │            │            │           ││
│  ├─────────────┼────────────┼────────────┼────────────┼───────────┤│
│  │  WebRTC     │  < 1s      │    Yes     │    Yes     │    Yes    ││
│  │  (UDP)      │  (real-time)│            │            │           ││
│  ├─────────────┼────────────┼────────────┼────────────┼───────────┤│
│  │   Smooth    │   2-5s     │    Yes     │    Yes     │   Edge    ││
│  │  Streaming  │            │            │            │  only     ││
│  └─────────────┴────────────┴────────────┴────────────┴───────────┘│
│                                                                      │
│  RECOMMENDATION:                                                     │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Primary: HLS + DASH (for broad compatibility)            │    │
│  │  Secondary: WebRTC (for live streaming)                    │    │
│  │  Fallback: Progressive Download (for offline)               │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 9.2 Estructura HLS

```
┌─────────────────────────────────────────────────────────────────────┐
│                    HLS STREAM STRUCTURE                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  MASTER PLAYLIST (.m3u8)                                             │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  #EXTM3U                                                       │   │
│  │  #EXT-X-VERSION:4                                              │   │
│  │                                                                 │   │
│  │  #EXT-X-STREAM-INF:BANDWIDTH=8000000,RESOLUTION=1920x1080     │   │
│  │  1080p/playlist.m3u8                                          │   │
│  │                                                                 │   │
│  │  #EXT-X-STREAM-INF:BANDWIDTH=5000000,RESOLUTION=1280x720      │   │
│  │  720p/playlist.m3u8                                           │   │
│  │                                                                 │   │
│  │  #EXT-X-STREAM-INF:BANDWIDTH=2500000,RESOLUTION=854x480       │   │
│  │  480p/playlist.m3u8                                           │   │
│  │                                                                 │   │
│  │  #EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="audio",NAME="Stereo"       │   │
│  │  audio/playlist.m3u8                                           │   │
│  └─────────────────────────────────────────────────────────────┘    │
│                         │                                            │
│                         ▼                                            │
│  MEDIA PLAYLIST (1080p/playlist.m3u8)                                │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  #EXTM3U                                                       │   │
│  │  #EXT-X-TARGETDURATION:10                                      │   │
│  │  #EXT-X-MEDIA-SEQUENCE:1                                       │   │
│  │  #EXT-X-PLAYLIST-TYPE:VOD                                      │   │
│  │                                                                 │   │
│  │  #EXTINF:10.000,                                               │   │
│  │  segment1.ts                                                    │   │
│  │                                                                 │   │
│  │  #EXTINF:10.000,                                               │   │
│  │  segment2.ts                                                    │   │
│  │                                                                 │   │
│  │  ...                                                           │   │
│  │                                                                 │   │
│  │  #EXT-X-ENDLIST                                                │   │
│  └─────────────────────────────────────────────────────────────┘    │
│                         │                                            │
│                         ▼                                            │
│  VIDEO SEGMENTS (segmentN.ts)                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Transport Stream (.ts) files                                │   │
│  │  - 2-10 seconds each                                         │   │
│  │  - Encrypted with AES-128                                     │   │
│  │  - Contain video + audio (multiplexed)                        │   │
│  │                                                                 │   │
│  │  Key File (.key)                                              │   │
│  │  #EXT-X-KEY:METHOD=AES-128,URI="encryption.key"              │   │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 10. Sistema de DRM

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DRM INTEGRATION                                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    DRM WORKFLOW                             │    │
│  │                                                               │   │
│  │  1. CONTENT PREPARATION                                     │   │
│  │     ┌─────────────────────────────────────────────────────┐ │   │
│  │     │  Raw Media ──► Encrypt (Widevine/PlayReady) ─►      │ │   │
│  │     │                 Encrypted Media + License           │ │   │
│  │     └─────────────────────────────────────────────────────┘ │   │
│  │                          │                                     │   │
│  │                          ▼                                     │   │
│  │  2. LICENSE ACQUISITION                                     │   │
│  │     ┌─────────────────────────────────────────────────────┐ │   │
│  │     │  Client ──Request License──► License Server       │ │   │
│  │     │                                    │              │ │   │
│  │     │                                    ▼              │ │   │
│  │     │                            ┌─────────────┐         │ │   │
│  │     │                            │  Validate  │         │ │   │
│  │     │                            │  Purchase  │         │ │   │
│  │     │                            │  & Rights │         │ │   │
│  │     │                            └─────┬─────┘         │ │   │
│  │     │                                  │               │ │   │
│  │     │                                  ▼               │ │   │
│  │     │                            ┌─────────────┐         │ │   │
│  │     │                            │   Return   │         │ │   │
│  │     │                            │  License   │         │ │   │
│  │     │                            │  (Keys)   │         │ │   │
│  │     │                            └───────────┘         │ │   │
│  │     └─────────────────────────────────────────────────────┘ │   │
│  │                          │                                     │   │
│  │                          ▼                                     │   │
│  │  3. PLAYBACK                                                           │
│  │     ┌─────────────────────────────────────────────────────┐         │
│  │     │  Encrypted Stream ──► DRM Module ──► Decrypt ──►   │         │
│  │     │                                    │                │         │
│  │     │                                    ▼                │         │
│  │     │                            ┌─────────────┐          │         │
│  │     │                            │   Decode   │          │         │
│  │     │                            │  & Render  │          │         │
│  │     │                            └───────────┘          │         │
│  │     └─────────────────────────────────────────────────────┘         │
│  │                                                                │
│  └──────────────────────────────────────────────────────────────      │
│                                                                      │
│  SUPPORTED DRM SYSTEMS:                                               │
│  ┌─────────────────────────────────────────────────────────────┐      │
│  │  - Widevine (Google) - Chrome, Android, TVs                  │      │
│  │  - FairPlay (Apple) - Safari, iOS, macOS                    │      │
│  │  - PlayReady (Microsoft) - Edge, Windows, Xbox              │      │
│  └─────────────────────────────────────────────────────────────┘      │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 11. Consideraciones de Seguridad

### 11.1 Medidas de Seguridad

| Capa | Medida | Implementación |
|------|--------|----------------|
| **Transporte** | TLS 1.3 | HTTPS/WSS para todas las comunicaciones |
| **Autenticación** | JWT + OAuth 2.0 | Tokens con expiración, refresh tokens |
| **DRM** | Widevine/FairPlay/PlayReady | Encriptación de contenido |
| **Storage** | Encriptación AES-256 | Archivos en reposo encriptados |
| **Watermarking** | Forensic Watermarks | Tracking de piratería |
| **Rate Limiting** | Por usuario/IP | Prevención de abuso |
| **Input Validation** | Schema validation | Prevención de inyecciones |

### 11.2 Arquitectura de Seguridad

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SECURITY ARCHITECTURE                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    EXTERNAL THREATS                        │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │    │
│  │  │  Piracy    │  │  Account   │  │   DDoS     │        │    │
│  │  │             │  │  Sharing   │  │  Attacks   │        │    │
│  │  │ - Watermark │  │  - Family  │  │  - Cloudflare│       │    │
│  │  │ - DRM       │  │    Sharing │  │  - Rate Limit│       │    │
│  │  │ - Legal    │  │  - Limit   │  │  - CDN      │        │    │
│  │  │   Actions  │  │    Devices │  │             │        │    │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                   SECURITY LAYERS                          │    │
│  │                                                               │   │
│  │  ┌─────────────────────────────────────────────────────┐    │   │
│  │  │  Layer 1: Network Security                         │    │   │
│  │  │  - WAF (Web Application Firewall)                  │    │   │
│  │  │  - DDoS Protection (Cloudflare)                   │    │   │
│  │  │  - TLS 1.3 Termination                             │    │   │
│  │  └─────────────────────────────────────────────────────┘    │   │
│  │                          │                                   │   │
│  │                          ▼                                   │   │
│  │  ┌─────────────────────────────────────────────────────┐    │   │
│  │  │  Layer 2: Application Security                     │    │   │
│  │  │  - API Gateway (Authentication, Rate Limiting)    │    │   │
│  │  │  - JWT Token Validation                           │    │   │
│  │  │  - Input Validation (Schema Validation)          │    │   │
│  │  │  - CORS Policies                                  │    │   │
│  │  └─────────────────────────────────────────────────────┘    │   │
│  │                          │                                   │   │
│  │                          ▼                                   │   │
│  │  ┌─────────────────────────────────────────────────────┐    │   │
│  │  │  Layer 3: Content Security                        │    │   │
│  │  │  - DRM Encryption (Widevine/FairPlay)            │    │   │
│  │  │  - Forensic Watermarking                          │    │   │
│  │  │  - License Management                             │    │   │
│  │  │  - Secure Decryption Module                       │    │   │
│  │  └─────────────────────────────────────────────────────┘    │   │
│  │                          │                                   │   │
│  │                          ▼                                   │   │
│  │  ┌─────────────────────────────────────────────────────┐    │   │
│  │  │  Layer 4: Data Security                           │    │   │
│  │  │  - AES-256 Encryption at Rest                     │    │   │
│  │  │  - Database Encryption                            │    │   │
│  │  │  - Key Management (AWS KMS/Vault)                │    │   │
│  │  │  - Audit Logging                                  │    │   │
│  │  └─────────────────────────────────────────────────────┘    │   │
│  │                                                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘