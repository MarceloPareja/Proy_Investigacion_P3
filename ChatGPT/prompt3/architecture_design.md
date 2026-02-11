# gAPI — API Gráfica y Motor de Render

## Descripción general del sistema
gAPI es una API gráfica para soportar nuevos juegos de rol. Debe exponer una interfaz consumible desde **C++**, **C#** y **Java**, ocultando detalles del motor gráfico. Internamente, el motor traduce llamadas de alto nivel a comandos del motor y, para renderizar, se integra con **DirectX** u **OpenGL**, que finalmente interactúan con el sistema operativo.

El motor dibuja escenas siguiendo un pipeline:
1) Terreno → 2) Estructuras → 3) Personajes → 4) Post‑efectos.

Además, debe renderizar el equipamiento del personaje en un orden que evite **clipping** (superposición incorrecta).

## Arquitectura

### Estilo recomendado
**Arquitectura en capas + Bridge/Adapter hacia backends gráficos**, con núcleo de motor independiente de API pública y de DirectX/OpenGL.

### Patrones
- **Facade**: API estable para el desarrollador del juego.
- **Adapter/Bridge**: abstracción de `GraphicsBackend` con adaptadores DirectX/OpenGL.
- **Command**: cola de comandos de render (minimiza cambios de estado, batching).
- **Composite**: escena como grafo (scene graph).
- **Pipeline**: etapas de render (terreno/estructuras/personajes/postFX).
- **Strategy**: selección de backend y políticas de render.

### Lenguajes / stack (sugerencia)
- **Core Engine**: C++ (rendimiento y ecosistema de gráficos).
- **Bindings**:
	- C# vía wrapper (P/Invoke/C++/CLI) según plataforma.
	- Java vía JNI/JNA.

## Componentes principales

### API pública (Bindings)
- `gAPI` (interfaces idiomáticas por lenguaje)
- Mapeo de tipos (vectores, matrices, texturas, meshes)
- Validación de parámetros y “marshalling” hacia el core

### Núcleo del motor
- **Renderer Core**: coordina el pipeline de render.
- **Scene Graph**: entidades/objetos de escena.
- **Resource/Asset Manager**: carga y cache de texturas, meshes, shaders.
- **Render Queue**: comandos ordenados para ejecución eficiente.
- **Equipment Composer** (personajes): composición por slots para evitar clipping.

### Backends
- `DirectXBackend` / `OpenGLBackend` implementan la abstracción `GraphicsBackend`.
- Integración con OS/driver para presentación en pantalla.

## Flujo de datos

### Dibujar escena
1. Juego invoca `DrawScene(scene)` sobre gAPI.
2. API traduce a llamadas del core.
3. Core arma `RenderQueue` por etapas: terreno → estructuras → personajes → postFX.
4. Para personajes, `Equipment Composer` determina orden de “montaje” por slot.
5. Core ejecuta comandos sobre el backend seleccionado (DirectX u OpenGL).
6. Backend presenta el frame (swap buffers).

### Composición de equipamiento (evitar clipping)
- Definir **slots ordenados** y reglas de compatibilidad (debajo/encima).
- Generar una lista final de meshes/materiales a renderizar por personaje siguiendo el orden.
- Optimización: agrupar objetos no superpuestos para render concurrente/batching.

## Decisiones técnicas
- **Backend intercambiable** mediante interfaz estable (`GraphicsBackend`).
- **Orden del pipeline** fijo para garantizar determinismo visual.
- **Orden de equipamiento** controlado por tabla de slots (ej.: ropa → armadura → capa → mochila…).
- **Performance**: minimizar cambios de estado, culling, batching, y uso de render queue.

