# Diseño de Arquitectura de Software - gAPI

Este documento describe la arquitectura de software propuesta para **gAPI** (Game API), una interfaz gráfica diseñada para facilitar el desarrollo de juegos de rol de fantasía en tercera persona.

## 1. Selección de Lenguaje y Tecnologías

### Lenguaje Principal: **C++ (Moderno: C++17/20)**

**Justificación:**
*   **Rendimiento Crítico**: Los motores gráficos requieren acceso de bajo nivel a la memoria y al hardware (GPU). C++ es el estándar de la industria para este propósito.
*   **Interoperabilidad**: Directa integración con APIs gráficas nativas como DirectX (COM/C++) y OpenGL (C).
*   **Control de Memoria**: Esencial para gestionar grandes cantidades de assets (texturas, modelos) y para implementar el sistema de *Equipment Composition* sin overhead del Garbage Collector.

### Bindings / Wrappers
Para cumplir el requerimiento de soportar **C#** y **Java**, se implementará una capa de bindings:
*   **C#**: Utilizando **C++/CLI** (en Windows) o **P/Invoke** con una API tipo C plana exportada desde la DLL de gAPI. Esto permite su uso en motores como Unity o frameworks .NET.
*   **Java**: Utilizando **JNI (Java Native Interface)** o **JNA**. JNI ofrece mejor rendimiento pero es más complejo; JNA es más simple pero puede tener overhead. Dado que es un motor gráfico, se recomienda una implementación cuidadosa con JNI o herramientas como **SWIG** para generar los bindings automáticamente.

---

## 2. Patrones Arquitectónicos y de Diseño

Para cumplir con los requerimientos de abstracción, flexibilidad de backend (DirectX/OpenGL) y composición de escenas, se proponen los siguientes patrones:

### A. Patrón Facade (Fachada)
*   **Propósito**: Proveer una interfaz unificada y simplificada a los lenguajes cliente (C++, C#, Java).
*   **Aplicación**: La clase `GAPIEngine` o `IGraphicsContext` será el único punto de entrada para que el programador del juego "dibuje" o cargue escenas, ocultando la complejidad de la gestión de recursos y selección de API gráfica.

### B. Patrón Bridge (Puente)
*   **Propósito**: Desacoplar la abstracción (gAPI) de su implementación (DirectX u OpenGL).
*   **Aplicación**:
    *   `RendererAPI` (Interfaz Abstracta): Define métodos como `DrawMesh`, `SetShader`, `ClearScreen`.
    *   `DXRenderer` (Implementación Concreta): Implementa usando llamadas DirectX.
    *   `GLRenderer` (Implementación Concreta): Implementa usando llamadas OpenGL.
    *   Esto permite cambiar de backend en tiempo de ejecución o compilación sin afectar el código del juego.

### C. Patrón Composite (Compositor)
*   **Propósito**: Tratar a objetos individuales y a composiciones de objetos de manera uniforme.
*   **Aplicación**: Gestión de la **Escena**.
    *   `SceneNode`: Clase base.
    *   `Terrain`, `Structure`, `Character`: Hojas o nodos compuestos.
    *   Un `Character` puede ser un nodo compuesto que contiene a sus partes y equipamiento (`Equipment`), permitiendo transformaciones (movimiento, rotación) en conjunto.

### D. Patrón Strategy (Estrategia)
*   **Propósito**: Definir una familia de algoritmos, encapsular cada uno, y hacerlos intercambiables.
*   **Aplicación**:
    *   **Post-Procesamiento**: Diferentes efectos (Bloom, Blur, ColorCorrection) pueden implementarse como estrategias aplicadas al final del pipeline.
    *   **Rendering Path**: Elegir entre *Forward Rendering* (más simple) o *Deferred Rendering* (mejor para muchas luces) según el hardware.

### E. Patrón Factory / Abstract Factory
*   **Propósito**: Crear familias de objetos relacionados sin especificar sus clases concretas.
*   **Aplicación**: Crear recursos (Texturas, Shaders, Buffers) específicos para DirectX u OpenGL según la configuración inicial.

---

## 3. Manejo de Equipamiento y Layering (Z-Fighting / Clipping Prevention)

El requerimiento de evitar "clipping" en el equipamiento del personaje (ropa, armadura, capas) se abordará mediante un sistema de **Capas de Equipamiento (Equipment Slots & Layers)**.

*   No se confiará solo en el Z-buffer (depth test) estándar, ya que mallas muy cercanas (piel y ropa ajustada) producen *z-fighting*.
*   **Solución**:
    1.  **Depth Bias / Offset**: Aplicar un ligero desplazamiento en el buffer de profundidad para las capas superiores.
    2.  **Stencil Buffer**: Usar máscaras para recortar la geometría de la capa inferior si está totalmente cubierta (optimización).
    3.  **Orden de Dibujado (Draw Order)**: Dibujar estrictamente desde adentro hacia afuera:
        1.  Cuerpo Base (Skin)
        2.  Ropa Interior / Ajustada
        3.  Armadura / Túnica
        4.  Capas / Mochilas (Back)
        5.  Accesorios Externos.

---

## 4. Pipeline de Renderizado (Resumen)

El ciclo principal del motor (`GameLoop`) seguirá este flujo para cada frame:

1.  **Update**: Actualizar lógica de juego, física y entradas.
2.  **Culling**: Determinar qué objetos de la escena son visibles por la cámara.
3.  **Render - Geometry Pass**:
    *   **Terreno**: Dibujar heightmap/malla del terreno con texturas splatting.
    *   **Estructuras**: Dibujar edificios, muros, props estáticos.
    *   **Personajes (Dynamic)**:
        *   Para cada personaje, iterar sobre sus componentes de equipamiento en orden de capa.
        *   Aplicar esqueletos/animaciones (Skinning) uniformemente a todas las partes equipadas.
4.  **Render - Post-Processing Pass**:
    *   Tomar el frame renderizado.
    *   Aplicar sombras (si no fueron en pase previo), iluminación global, corrección de color.
5.  **Present**: Intercambiar buffers (Double Buffering) para mostrar en pantalla.
