# Diseño de Arquitectura: Servicio de Calendario Multinacional

## 1. Introducción
Este documento detalla la arquitectura de software propuesta para un servicio de calendario seguro y escalable, diseñado para una empresa multinacional. El sistema permitirá el acceso desde múltiples plataformas (Web, Móvil, Tablet) y garantiza la privacidad de los datos mediante encriptación.

## 2. Estilo Arquitectónico
Se recomienda una **Arquitectura de Microservicios**.

### Justificación:
- **Escalabilidad Independiente**: El servicio de notificaciones y el de desencriptación pueden requerir recursos muy diferentes. Los microservicios permiten escalar cada componente según su demanda.
- **Multinacional**: Permite desplegar instancias regionales de ciertos servicios para reducir latencia.
- **Seguridad**: Aislar el módulo de desencriptación en un servicio protegido reduce la superficie de ataque.
- **Diversidad de Clientes**: Un API Gateway centralizado facilita la gestión de diferentes tipos de clientes (Web, iOS, Android).

## 3. Selección de Tecnología (Stack Recomendado)

### Backend
- **Lenguaje Principal (Core Logic & API Gateway):** **Node.js (TypeScript)** o **Go**.
  - *Por qué*: Node.js es excelente para operaciones I/O intensivas (como notificaciones y manejo de conexiones HTTP). Go ofrece un rendimiento superior y tipado estático, ideal para servicios de alta concurrencia.
- **Servicio de Criptografía (Calendar Service):** **Rust** o **Go**.
  - *Por qué*: El manejo de claves y desencriptación en vuelo requiere eficiencia y seguridad de memoria. Rust es ideal para evitar vulnerabilidades de memoria y maximizar el rendimiento de la CPU durante la desencriptación.

### Frontend
- **Web:** **React.js** o **Next.js**.
  - *Por qué*: Ecosistema maduro, manejo eficiente del estado (importante para un calendario interactivo) y capacidad de PWA.
- **Móvil/Tablet:** **Flutter** o **React Native**.
  - *Por qué*: Permite una base de código compartida para iOS y Android, reduciendo costos de desarrollo y mantenimiento mientras se mantiene un rendimiento nativo.

### Base de Datos & Almacenamiento
- **Base de Datos Principal:** **PostgreSQL**.
  - *Por qué*: Robusti, soporte para JSONB (útil para metadata flexible de eventos) y excelente integridad referencial para usuarios y permisos.
- **Almacenamiento de Claves:** **HashiCorp Vault** o **AWS KMS**.
  - *Por qué*: Nunca se deben guardar las claves de encriptación junto con los datos. Un gestor de secretos es vital.

### Mensajería (Notificaciones)
- **RabbitMQ** o **Apache Kafka**.
  - *Por qué*: Para desacoplar la generación de eventos (ej. "nueva reunión creada") del envío de notificaciones (email, push), permitiendo que el sistema de notificaciones procese colas sin bloquear al usuario.

## 4. Patrones Arquitectónicos

### A. API Gateway (BFF - Backend for Frontend)
Un único punto de entrada que enruta las peticiones, maneja la autenticación inicial (validación de tokens) y puede adaptar las respuestas para dispositivos móviles vs web si fuera necesario.

### B. Encryption/Decryption at Rest & in Transit (Encriptación)
- **Patrón Envelope Encryption**:
  - Cada calendario (o usuario) tiene una *Data Key* única encriptada por una *Master Key*.
  - La base de datos solo ve datos cifrados (`blob`).
  - Solo el Servicio de Calendario, tras verificar permisos, tiene acceso temporal a la *Data Key* para desencriptar en memoria y servir la vista al usuario.

### C. Observer / Pub-Sub (Notificaciones)
Cuando se crea o modifica un evento, se publica un mensaje en el bus de servicios. El Servicio de Notificaciones se "suscribe" a estos eventos para enviar alertas sin ralentizar la operación de guardado.

### D. CQRS (Command Query Responsibility Segregation) (Opcional pero recomendado)
Separar las operaciones de lectura (ver calendario) de las de escritura (crear evento).
- *Escritura*: Valida lógica, encripta y guarda.
- *Lectura*: Requiere un pipeline optimizado de desencriptación y agregación de citas.
