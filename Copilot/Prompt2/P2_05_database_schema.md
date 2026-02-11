# Esquema de Datos — Calendario Encriptado

```mermaid
erDiagram
  USUARIO ||--o{ CALENDARIO : posee
  CALENDARIO ||--o{ EVENTO : contiene
  EVENTO ||--o{ ALERTA : tiene
  CALENDARIO ||--o{ ACL : permisos
  KEY ||--|| CALENDARIO : protege

  USUARIO {
    int id PK
    string email
    string password_hash
    string nombre
    enum rol
  }

  CALENDARIO {
    int id PK
    int usuario_id FK
    string nombre
    blob encrypted_blob
    int key_id FK
    timestamp fecha_creacion
    timestamp fecha_actualizacion
  }

  EVENTO {
    int id PK
    int calendario_id FK
    timestamp inicio
    timestamp fin
    string titulo
    text metadata_encrypted
    boolean all_day
  }

  ALERTA {
    int id PK
    int evento_id FK
    timestamp trigger_time
    enum canal "EMAIL|PUSH"
    boolean recurrent
  }

  ACL {
    int id PK
    int calendario_id FK
    int usuario_id FK
    enum permiso "READ|WRITE|OWNER"
  }

  KEY {
    int id PK
    string kms_key_id
    string key_meta
  }
```

- Los calendarios se guardan como blobs cifrados; los metadatos esenciales (owner, timestamps, A CL) están en claro para consultas.
- Las claves son referencias a KMS; nunca almacenar claves en texto plano.

## Índices sugeridos
- `calendario(usuario_id)`
- `evento(calendario_id, inicio)`
- `alerta(trigger_time)`

## Consultas típicas
- Obtener últimos eventos del calendario: paginación por `inicio`.
- Buscar eventos próximos 7 días: index por `inicio`.
