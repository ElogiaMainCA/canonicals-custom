# 🛠️ Solución Custom para Canonicals Unificadas en PDP - C&A México

## 🎯 Objetivo del Documento

Proponer una **solución técnica y estratégica** que permita generar **etiquetas canónicas únicas, limpias y coherentes** en todas las páginas de producto (PDP) del sitio [cyamoda.com](https://www.cyamoda.com), eliminando la duplicidad, contradicciones y problemas derivados del manejo de parámetros dinámicos (`?pid=`, `?dwvar_`, etc.) o scripts externos.

---

## 🧩 Problema Detectado

Durante la auditoría técnica del sitio se identificó que muchas PDP están generando **etiquetas `<link rel="canonical">` duplicadas o conflictivas**. Esto ocurre principalmente por:

- Renderizado simultáneo desde plantillas base (Product-Show) y Experience Pages.
- Inyección de scripts de terceros (Yotpo, VWO, GTM) que alteran el DOM.
- Parámetros de personalización en URL (`?dwvar_`, `?pid=`, etc.) sin unificación canónica.

### ❌ Ejemplos de errores comunes:

html
<link rel="canonical" href="https://www.cyamoda.com/playera-tank-slim/3104446.html" />
<link rel="canonical" href="https://www.cyamoda.com/3104446.html" />
<link rel="canonical" href="https://www.cyamoda.com/playera-tank-slim/3104446.html?dwvar_3104446_color=VERDE" />
----

## 🔍 Impacto SEO

- Dificulta la correcta consolidación de señales SEO (autoridad, backlinks, clics).
- Provoca indexación errónea de URLs con parámetros.
- Aumenta el contenido duplicado interno.
- Disminuye el crawl budget en sitios de gran tamaño.
- Google puede ignorar todas las canonicals conflictivas si detecta ambigüedad.

---

## 🧠 Enfoque de la Solución

El objetivo es generar **una única etiqueta canonical por PDP** con las siguientes condiciones:

- Sin parámetros de sesión ni personalización.
- Basada en la URL amigable del producto.
- Definida desde el backend (SFCC) y no en el frontend (JS).
- Resistente a conflictos causados por scripts externos.

---

## 🧱 Arquitectura de la Solución

### 1. 📦 Backend (Demandware / SFCC)

#### Paso 1: Normalizar la lógica de generación de URL

- Utilizar la API `URLUtils.url('Product-Show', 'pid', product.ID)` como base.
- Convertir todas las URLs a su versión amigable limpia (sin parámetros ni fragmentos).
- En templates `.isml`, asegurarse de tener **una sola declaración** de canonical por producto:

html
<link rel="canonical" href="${URLUtils.http('Product-Show', 'pid', product.ID).toString()}" />

------------------------

Paso 2: Agregar lógica de validación en controller Product-Show.js

var productHelper = require('*/cartridge/scripts/helpers/productHelpers');

if (product && product.isVisible()) {
    res.setViewData({
        canonicalUrl: URLUtils.https('Product-Show', 'pid', product.ID)
    });
}
Y luego pasar esta variable a la plantilla:

<link rel="canonical" href="${canonicalUrl}" />
Paso 3: Redireccionamiento de URLs con parámetros

Implementar lógica de redirección 301 para evitar duplicados:

if (request.httpParameterMap.hasParameter('dwvar') || request.httpParameterMap.hasParameter('pid')) {
    var cleanUrl = URLUtils.https('Product-Show', 'pid', product.ID);
    response.redirect(cleanUrl);
}
2. 🧼 Frontend: Asegurar integridad del DOM
Paso 1: Auditar JavaScript

Revisar si herramientas como:

Yotpo
Visual Website Optimizer (VWO)
GTM (Google Tag Manager) Custom HTML Tags
... están inyectando etiquetas <link rel="canonical"> adicionales en tiempo de ejecución. Si es así:

Deshabilitar esas funciones o
Bloquear su ejecución mediante condiciones específicas, como:
if (!document.querySelector('link[rel="canonical"]')) {
    // Insertar solo si no existe
}
Paso 2: Establecer reglas de CSS para evitar visibilidad engañosa

Prevenir etiquetas <link> ocultas dentro de <noscript> o <template>.

🧪 Verificación Técnica

Herramientas Recomendadas
Screaming Frog SEO Spider – analizar canonical duplicadas.
Sitebulb – detección visual de conflictos.
Google Search Console – revisar indexación errónea de URLs con parámetros.
DevTools (Chrome) – con JS habilitado, verificar el DOM final con:
document.querySelectorAll('link[rel="canonical"]')
Comando de verificación en Google:
site:cyamoda.com inurl:3104446
Validar que solo se indexa una versión canónica del producto.

✅ Checklist de Implementación

Elemento	Estado
Unificación de canonical en backend	🟡 En progreso
Redirección de URLs con parámetros	🔲 Pendiente
Auditoría de scripts externos	🟡 En validación
Eliminación de canonical en JS	🔲 Pendiente
Pruebas en 3 productos piloto	🔲 Agendada
Revisión de indexación en GSC	🔲 Pendiente post-implementación
📌 Estado Ideal del HTML en PDP

<head>
  ...
  <link rel="canonical" href="https://www.cyamoda.com/playera-tank-slim/3104446.html" />
  <!-- Ninguna otra etiqueta canonical debe existir -->
</head>
📅 Plan de Despliegue Recomendado

Fase	Descripción	Tiempo estimado
1. Análisis de impacto	Revisión de cantidad de URLs duplicadas en GSC/Screaming Frog	2 días
2. Desarrollo y validación	Unificación de canonical y redirección en entorno staging	4 días
3. QA técnico	Pruebas con y sin parámetros, validación con DevTools	2 días
4. Despliegue	Push a producción, seguimiento de logs y tags	1 día
5. Monitoreo	Reindexación y consolidación de señales SEO	7 días
🧠 Conclusión

Implementar una solución personalizada para etiquetas canónicas unificadas es una acción crítica para proteger el posicionamiento orgánico del sitio.

Consolidar las URLs canónicas:

✅ Mejora la autoridad SEO.
✅ Evita contenido duplicado.
✅ Asegura una experiencia coherente para motores de búsqueda.
✅ Prepara la arquitectura para campañas futuras sin conflictos de indexación.
Esta acción debe ejecutarse antes de campañas de rebajas, lanzamientos de colección o picos estacionales de tráfico.
