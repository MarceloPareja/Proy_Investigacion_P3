# Recomendaciones Técnicas y Patrones

## Lenguajes / Stack recomendado
- **Backend AppS**: Go (alto rendimiento) o Node.js/TypeScript (rápido desarrollo). Java es buena opción para ecosistemas grandes.
- **Workers Transcode**: Contenedores con FFmpeg (Linux), orquestados en Kubernetes
- **Frontend / Player**: Native apps (Swift/Kotlin) o cross-platform (React Native / Electron)
- **Storage**: S3 / Azure Blob + CDN
- **Broker**: RabbitMQ / Kafka para jobs
- **DB**: Postgres para metadata, Redis para cache

## Patrones arquitectónicos
- **Microservicios**: separación MediaStore, Transcoder, Billing, DownloadMgr, Sync
- **Event-driven**: jobs para transcodificación y notificaciones
- **CQRS**: separar escrituras (purchases, uploads) y lecturas (catálogo, librería)
- **Strangler pattern**: para migraciones incrementales

## Protocolos y formatos
- Streaming: HLS / MPEG-DASH
- Containers: MP4 fragmented / CMAF
- Audio codecs: AAC, Opus
- Subtitles: WebVTT / TTML

## Seguridad y DRM
- Integrar Widevine / PlayReady / FairPlay según plataformas
- Signed URLs, token-based auth
- Trazabilidad y logs para facturación y cumplimiento

## Escalado y rendimiento
- Autoscaling workers según queue depth
- CDN para descarga de segmentos
- Cache de manifests y metadatos en Redis

## Observabilidad
- Metrics (Prometheus), Tracing (OpenTelemetry), Logs centralizados (ELK)

## Recomendaciones operativas
- Canary releases para workers transcode
- Monitoreo de TTFB en CDN y latencias de transcode
- Políticas de retención y purga para costes
