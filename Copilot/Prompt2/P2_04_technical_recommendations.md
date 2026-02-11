# Recomendaciones Técnicas — Servicio de Calendario

## Lenguajes y Frameworks recomendados
- Backend: **Go** (alto rendimiento, concurrencia) o **Node.js/TypeScript** si se prioriza velocidad de desarrollo.
- Frontend: **React** o **Flutter** (si buscas un único codebase móvil+web con Flutter Web).

## Por qué elegir Go vs Node.js
- Go: mejor para alto throughput, concurrencia y servicios de sincronización en tiempo real.
- Node.js: amplio ecosistema, rapidez para prototipado y manejo de JSON.

## Patrones arquitectónicos a implementar
- **Microservicios por responsabilidad**: Auth, Calendar, Crypto, Notify, Sync.
- **Patrón de Capas**: API → Servicios → Repositorios → Persistencia.
- **CQRS** (opcional): separar lectura (vistas) y escritura para optimizar sincronización y consultas.
- **Event-Driven**: eventos para sincronización y notificaciones (Kafka/RabbitMQ).
- **Zero Trust / Encrypted-at-rest**: claves gestionadas en KMS.
- **Circuit Breaker / Retry Policy** para integraciones externas

## Seguridad
- KMS para keys (AWS KMS / Azure Key Vault)
- Cifrado de calendario por usuario/objeto
- RBAC y scopes OAuth2
- MFA opcional para accesos sensibles
- Logs de auditoría y trazabilidad

## Infraestructura recomendada
- Kubernetes para orquestación
- Managed Postgres (RDS / Azure DB)
- Object Storage (S3 / Azure Blob)
- Redis para caché y locks
- Broker de eventos (Kafka o RabbitMQ)

## Asincronía y Escalado
- Worker queue (Celery / Sidekiq / Go workers) para jobs de notificación y tareas pesadas
- WebSocket / Push gateway (APNs / FCM) para Sync Service

## Observabilidad
- Metrics: Prometheus + Grafana
- Tracing: OpenTelemetry
- Logs centralizados: ELK / Loki

## Consideraciones de privacidad
- Minimizar datos en texto plano
- TTL para backups no persistentes
- Políticas para revocar acceso y rotación de claves
