# Diagramas de Arquitectura: Sistema de Reproducción Multimedia Online

## 1. Diagrama de Contexto (C4 Nivel 1)
Muestra las interacciones de alto nivel entre los usuarios y los sistemas principales.

```mermaid
C4Context
    title Diagrama de Contexto del Sistema de Multimedia Online

    Person(customer, "Cliente", "Usuario que consume contenido multimedia.")
    
    System_Ext(website, "Sitio Web de Descarga", "Sitio para descargar el instalador de la aplicación.")
    System(app_client, "Aplicación Cliente", "Aplicación de escritorio o móvil para reproducir y gestionar media.")
    System(backend_apps, "AppS (Application Server)", "Servidor central que gestiona usuarios, catálogo y compras.")
    
    Rel(customer, website, "Descarga instalador desde")
    Rel(customer, app_client, "Usa para ver/escuchar contenido")
    Rel(app_client, backend_apps, "Solicita autenticación, catálogo y licencias", "HTTPS/REST")
    Rel(website, backend_apps, "Obtiene link de última versión", "HTTPS/REST")
```

## 2. Diagrama de Contenedores (C4 Nivel 2)
Detalla la arquitectura interna del sistema, separando los servidores de aplicación y medios.

```mermaid
C4Container
    title Diagrama de Contenedores - Sistema Multimedia

    Person(user, "Usuario", "Cliente del servicio")

    Container_Boundary(client_device, "Dispositivo del Cliente") {
        Container(desktop_app, "App de Escritorio", "Electron/C++", "Reproductor principal y gestor de librería.")
        Container(mobile_app, "App Móvil", "Flutter/Native", "Reproductor portátil.")
    }

    Container(web_site, "Sitio Web", "React/Static", "Portal de descarga del instalador.")

    Container_Boundary(cloud_system, "Backend System") {
        Container(api_gateway, "API Gateway", "Nginx/Kong", "Punto de entrada único.")
        Container(app_server, "App Server (AppS)", "Node.js/Go", "Lógica de negocio, usuarios, compras.")
        Container(video_server, "Video Server (VS)", "C++/Go", "Almacena, transcodifica y transmite video.")
        Container(audio_server, "Audio Server (AS)", "C++/Go", "Almacena, comprime y transmite audio.")
        
        ContainerDb(user_db, "Base de Datos de Usuarios", "PostgreSQL", "Perfiles, compras, historial.")
        ContainerDb(media_db, "Base de Datos de Medios", "MongoDB/NoSQL", "Metadatos técnicos, ubicaciones de archivos.")
        ContainerDb(file_storage, "Almacenamiento de Archivos", "S3/Blob Storage", "Archivos crudos y procesados.")
    }

    Rel(user, web_site, "Visita para descargar App")
    Rel(user, desktop_app, "Usa")
    Rel(user, mobile_app, "Usa")

    Rel(desktop_app, api_gateway, "API Requests", "HTTPS/JSON")
    Rel(mobile_app, api_gateway, "API Requests", "HTTPS/JSON")
    Rel(web_site, api_gateway, "Consulta versión", "HTTPS/JSON")

    Rel(api_gateway, app_server, "Enruta peticiones", "gRPC/HTTP")
    
    Rel(app_server, video_server, "Solicita procesamiento de video", "gRPC/Async")
    Rel(app_server, audio_server, "Solicita compresión de audio", "gRPC/Async")
    
    Rel(video_server, file_storage, "Lee/Escribe Video")
    Rel(audio_server, file_storage, "Lee/Escribe Audio")
    
    Rel(app_server, user_db, "Lee/Escribe Datos de Usuario")
    Rel(app_server, media_db, "Lee/Escribe Metadatos")

    Rel(desktop_app, video_server, "Stream Video", "HLS/DASH")
    Rel(desktop_app, audio_server, "Stream Audio", "HLS/DASH")
```

## 3. Diagrama de Componentes: Application Server (AppS) (C4 Nivel 3)
Muestra los módulos internos del servidor de aplicación.

```mermaid
classDiagram
    class AppServer {
        <<Container>>
    }
    
    class AuthController {
        +login()
        +register()
    }
    
    class CatalogManager {
        +searchMedia()
        +getDetails()
    }
    
    class PurchasingModule {
        +buyContent()
        +verifyLicense()
    }
    
    class MediaOrchestrator {
        +requestTranscoding()
        +requestCompression()
        +addSubtitles()
    }
    
    AppServer *-- AuthController
    AppServer *-- CatalogManager
    AppServer *-- PurchasingModule
    AppServer *-- MediaOrchestrator
    
    MediaOrchestrator --> VideoServerClient : gRPC
    MediaOrchestrator --> AudioServerClient : gRPC
```

## 4. Diagrama de Componentes: Cliente (Desktop/Mobile)
Muestra la estructura interna de la aplicación cliente y la sincronización.

```mermaid
componentDiagram
    package "Aplicación Cliente" {
        [UI Interfaz de Usuario] as UI
        [Reproductor Multimedia] as Player
        [Gestor de Librería Local] as LibLocal
        [Cliente API Store] as StoreClient
        [Gestor de Descargas] as Downloader
        [Sincronizador USB/Wi-Fi] as Sync
    }

    UI --> Player : Controla
    UI --> LibLocal : Visualiza contenido
    UI --> StoreClient : Navega tienda
    
    StoreClient --> Downloader : Inicia descarga
    Downloader --> LibLocal : Guarda archivo descargado
    
    Sync --> LibLocal : Lee contenido para móvil
    Sync --> MobileDevice : Transfiere archivos (USB/Wi-Fi)
```
