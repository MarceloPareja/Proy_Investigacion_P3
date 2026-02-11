# Diseño de Bindings — C++, C#, Java

## Principio general
- Exponer una **interfaz C ABI estable** (funciones `extern "C"`) como contrato público de gAPI.
- Implementar adaptadores/wrappers para cada lenguaje:
  - C++: uso directo de la API nativa (inline headers) o wrapper thin C++ class.
  - C#: P/Invoke (DllImport) o C++/CLI para integraciones profundas.
  - Java: JNI o JNA para llamadas nativas.
- Mantener reglas claras de ownership (quién libera memoria) y thread-safety.

## Ejemplo de capas
- `gapi_c.h` (C ABI): funciones de inicialización, creación de escena, subir mesh, crear personaje, renderFrame.
- Wrappers autogenerados / manuales:
  - `gapi_cpp.hpp` (header friendly C++)
  - `gapi_cs.dll` (C# wrapper + managed types)
  - `gapi_java.jar` (JNI layer)

## Consideraciones importantes
- Data marshaling: usar buffers binarios para meshes/textures para minimizar copia.
- Callbacks: permitir registro de callbacks desde cliente para eventos (onLoad, onError) con context pointers.
- Exceptions: no propagar excepciones C++ a través de C ABI; usar códigos de error y `gapi_get_last_error()`.
- Thread model: render thread único para las llamadas que tocan GPU; llamadas desde otros threads deben encolar comandos.
- Lifetime: objetos del engine referenciados por handles opacos (uint64_t) para seguridad de ABI.

## Estrategia de versiones y compatibilidad
- Mantener la C ABI lo más estable posible; versionar el shared library (semver)
- Exportar `gapi_get_api_version()` y `gapi_create_instance(version)`
- Deprecate con wrappers compatibles hacia atrás

## Herramientas de generación
- SWIG (opcional) para generación inicial de wrappers
- CBindGen / clang para generar headers C desde C++ si se desea

## Puntos de integración con el motor
- Inicialización: `gapi_init(params)` → alocar subsistemas
- Resource upload: `gapi_upload_mesh(handle, buffer)` (buffer binario)
- Scene ops: `gapi_add_entity(handle, descriptor)`
- Render loop: `gapi_render_frame()` → devuelve frame stats

## Seguridad y sandboxing
- Validar inputs desde bindings
- No confiar en tamaños pasados por cliente; verificar antes de upload
- Opcional: runtime checks en builds de debug para detectar misuse
