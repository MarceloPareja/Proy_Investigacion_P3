# Recomendaciones de Rendimiento y Gestión de Recursos

## Arquitectura de rendimiento
- Job System multithread para preparación de frame (visibility, animation skinning, equipment assembly)
- Render thread exclusivo para llamadas a la API gráfica
- Resource streaming (meshes/textures) con LRU cache

## Técnicas clave
- Frustum & Occlusion Culling: reducir trabajo de GPU
- LOD (tallas de malla y texturas) para objetos lejanos
- GPU Instancing para objetos repetidos (árboles, rocas)
- Merge de static geometry en chunks
- Texture atlases y array textures para minimizar switches
- Minimize state changes: sort by shader/material
- Use persistent mapped buffers para streaming de VBOs

## Memory management
- Pool allocators para vertex buffers y command buffers
- Arena allocators para datos temporales cada frame
- Evitar allocations en hot loops; reutilizar buffers

## Minimizar stalls
- Triple buffering de recursos CPU→GPU
- Evitar readbacks desde GPU; si necesario, usar staging buffers
- Asynchronous resource uploads en background threads

## Equipment assembly performance
- Precompute common equipment combinations (baked meshes) para conjuntos frecuentes
- Cache per-character combined meshes for visible combinations
- If dynamic blending required, use GPU skinning and morph targets

## Profiling & Telemetry
- Expose frame stats (draw calls, triangles, GPU time)
- Integrate with GPU profilers (Nsight, RenderDoc) and CPU profilers
- Tracing: OpenTelemetry / custom tracing for frame phases

## Scalabilidad multiplataforma
- Encapsular GPU-specific features behind backend adapters
- Provide fallbacks for missing features (instancing, specific texture formats)

## Testing
- Automated stress tests with many entities and randomized equipment
- Regression benchmarks for frame time and memory

## Dev-mode features
- Validation layers for driver issues
- Debug visualizers (draw call heatmap, overdraw view)
