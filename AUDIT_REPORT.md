# 📊 Reporte de Auditoría y Diseño Arquitectónico - CobranzApp

Este documento resume las conclusiones de diseño y los hallazgos de la auditoría del motor de automatización **CobranzApp** (archivo `Script`).

---

## 🔍 Resumen Ejecutivo
El sistema **CobranzApp** está estructurado con una arquitectura modular y robusta adaptada de forma óptima para Google Workspace. A través del análisis del código y el valioso feedback de desarrollo, se han alineado las etiquetas de IA y se han documentado las decisiones estratégicas de diseño y seguridad que rigen la aplicación.

---

## ✅ Alineación de Etiquetas de IA (Filtros Limpios y Consistentes)

* **Estado:** **ALINEADO AL 100%**
* **Descripción de la Solución:**
  1. **Remoción de `_promesa`**: Se eliminó de la IA debido a su inconsistencia en la clasificación, enfocándose únicamente en decisiones binarias y altamente confiables.
  2. **Remoción de `PENDIENTE` de la IA**: El tag `PENDIENTE` ha sido catalogado como de **uso exclusivo para operadores humanos**. Ahora, cuando la IA (`llamarGeminiPorLote`) encuentra comentarios vacíos o sin indicación clara, devuelve un valor vacío (`""`), evitando asignaciones incorrectas o ruidos en la categorización.
  3. **Simplificación de Etiquetas**: Se eliminaron etiquetas redundantes u obsoletas como `_pt` y `_espera`. El motor de procesamiento en `procesarComentariosIA()` ahora valida estrictamente las tres etiquetas operativas de IA de alta confianza:
     - **`_pagado`**
     - **`_cancelado`**
     - **`_reexpedido`**

---

## 🧠 Decisiones Estratégicas de Ingeniería y Seguridad

### 1. Hardcoding de Identificadores (Google Drive y Hojas de Cálculo)
* **Hallazgo Inicial:** Identificadores clave de carpetas y libros (como `ID_CARPETA`, `ID_CARPETA_HISTORIAL`, e `idLibro`) están integrados directamente en el código fuente de Apps Script.
* **Justificación de Diseño (Seguridad):**
  Esta es una **decisión deliberada de seguridad**. Colocar estos IDs en la hoja de cálculo visible (`Config`) permitiría que cualquier usuario con acceso al documento los altere (por error o con malas intenciones), lo cual rompería la automatización completa del sistema o expondría carpetas internas. Mantenerlos exclusivamente en el código garantiza que **únicamente los desarrolladores con permisos de edición de Apps Script puedan modificarlos**.

### 2. Actualización de Celdas de forma Individual (`setValue` en Loops)
* **Hallazgo Inicial:** Llamadas a `hojaMaestra.getRange().setValue()` dentro del bucle de comentarios.
* **Justificación de Diseño (Optimización Ajustada):**
  La inyección individual de datos se ejecuta sobre lotes (batches) estrictos de **máximo 50 comentarios**. Tras un riguroso proceso de prueba y error en el entorno productivo de Google Apps Script, se determinó que procesar y actualizar en bloques de 50 filas ofrece el **balance óptimo de rendimiento y consistencia**, previniendo fallos de concurrencia y límites de llamadas sin incurrir en la sobrecarga que un procesamiento masivo completo del documento implicaría.

---

## 💡 Recomendaciones Futuras para el Equipo de Desarrollo
* **Monitoreo de Cuotas de Gemini:** Mantener un seguimiento del uso de la cuota diaria gratuita del modelo `gemini-2.5-flash` (1,500 peticiones diarias) en Google AI Studio a medida que escale el volumen de pólizas.
* **Control de Versiones de Apps Script:** Documentar cualquier cambio futuro de IDs directamente en este archivo para mantener una trazabilidad clara para nuevos ingenieros del equipo.
