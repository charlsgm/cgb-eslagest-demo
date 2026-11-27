# Informe de rendimiento · Portada de movistar.es

**URL:** https://www.movistar.es/
**Fecha:** 26/11/2026 · **Herramienta:** Lighthouse 12 (Chromium headless), una pasada móvil y una escritorio. Datos de laboratorio, no de usuarios reales.

## Resultados

| Métrica | Móvil | Escritorio | Objetivo |
|---|---|---|---|
| **Puntuación** | **50** | **76** | ≥ 90 |
| FCP | 2,1 s | 1,0 s | < 1,8 s |
| **LCP** | **20,7 s** | 3,7 s | < 2,5 s |
| TBT | 960 ms | 80 ms | < 200 ms |
| CLS | 0 | 0,061 | < 0,1 |
| Speed Index | 4,1 s | 1,5 s | < 3,4 s |
| Time to Interactive | 23,2 s | 4,4 s | < 3,8 s |
| Peso total | 4,2 MB | 4,2 MB | < 1,6 MB |
| Peticiones | 69 | 84 | — |

El servidor responde rápido (~40 ms, caché de Next.js). El problema está en lo que se descarga después.

## Problema principal: imágenes del banner de Black Friday

De 4,2 MB totales, 3,5 MB son imágenes, casi todas del banner principal (`superbanner-home-black-friday…`):

| Tamaño | Formato | Prioridad |
|---|---|---|
| 738 KB | JPG | Alta |
| 711 KB | AVIF | Alta |
| 697 KB | JPG | Alta |
| 683 KB | AVIF | Alta |
| 172 KB | JPG | Alta |
| 160 KB | AVIF | Alta |

1. **Se descargan JPG y AVIF de la misma imagen**: el `<picture>` o el componente de imagen está mal montado.
2. **Se descargan todas las variantes de tamaño** (móvil y escritorio) en lugar de solo la necesaria.
3. **Compresión muy baja**: la ruta incluye `/low-compress/`; un AVIF de banner debería rondar 80–150 KB.
4. **13 imágenes con `fetchpriority="high"`**: compiten entre sí y retrasan el LCP.

Sin limitar la red, el LCP observado fue de ~0,6 s; los 20,7 s corresponden a la simulación de móvil medio con 4G lento, que es la que usa Google.

## Problema secundario: JavaScript en móvil

- Trabajo del hilo principal: 6,2 s; ejecución de JS: 3,1 s; TBT: 960 ms.
- Solo 437 KB de JS: el coste viene de la hidratación de Next.js y de las herramientas de medición (Adobe, AB Tasty, Dynatrace, CI360). `metrics.col.movistar.es` aporta 190 KB.

## Ahorros estimados por Lighthouse

- Mejorar formato y compresión de imágenes: ~930 KB
- Carga diferida de imágenes fuera de pantalla: ~870 KB
- Imágenes al tamaño mostrado: ~700 KB
- `preconnect` a dominios de terceros: ~260 ms
- JS sin usar, caché corta y JS duplicado: ahorros menores

## Recomendaciones (por impacto)

1. Prioridad alta solo para la primera diapositiva del banner; el resto con `loading="lazy"` y sin `fetchpriority`.
2. Corregir el `<picture>` (`<source media=… type="image/avif">` + un único `<img>` de reserva) para descargar un solo formato y tamaño.
3. Recomprimir los banners a ≤ 150 KB por imagen.
4. Diferir AB Tasty, CI360 y Dynatrace hasta después de la interactividad y reducir la hidratación.

Con los puntos 1–3, el LCP móvil debería bajar de ~20 s a 3–4 s y la puntuación superar 75.
