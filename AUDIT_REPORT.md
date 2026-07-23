# 📊 Reporte de Auditoría y Static Analysis - CobranzApp

Este documento resume los hallazgos identificados en el código de **CobranzApp** (archivo `Script`) tras un análisis estático detallado. Se han clasificado por severidad para facilitar su revisión y resolución.

---

## 🔍 Resumen Ejecutivo
El sistema está estructurado de manera modular y robusta para la automatización en Google Workspace. Se realizó un análisis profundo que identificó una inconsistencia inicial en la clasificación del tag `_promesa`. Siguiendo el feedback del usuario de que **`_promesa` es inconsistente y no se requiere**, se procedió a **eliminarlo por completo tanto de las instrucciones de Gemini como de las validaciones de Javascript**.

---

## ✅ Resuelto: Eliminación del Tag Inconsistente `_promesa`

* **Estado:** **CORREGIDO / ELIMINADO**
* **Descripción:**
  Anteriormente, la IA de Gemini (`llamarGeminiPorLote`) estaba instruida para clasificar comentarios de promesas de pago bajo el tag `_promesa`. Sin embargo, debido a reportes de usuarios sobre su alta inconsistencia:
  1. Se eliminó la instrucción de clasificar `_promesa` del prompt en `llamarGeminiPorLote()`.
  2. Se actualizó el ejemplo de respuesta esperada en el prompt de Gemini para evitar el uso de `_promesa`.
  3. Se removió cualquier validación de `_promesa` en la función principal de procesamiento de comentarios (`procesarComentariosIA()`), garantizando que la IA se enfoque únicamente en tags altamente consistentes (`_pagado`, `_cancelado`, `_reexpedido`, `PENDIENTE`).

---

## ⚠️ Severidad Media: Discrepancias de Etiquetas e Ineficiencia (GAS Anti-patterns)

### 1. Discrepancias de Etiquetas de IA restantes (`_pt`, `_espera`)
* **Archivo/Línea:** `Script`, Función `procesarComentariosIA()` vs `llamarGeminiPorLote()`.
* **Descripción:**
  La validación en `procesarComentariosIA()` incluye `_pt` y `_espera`. Sin embargo:
  1. En el prompt de Gemini, se le pide clasificar pérdidas totales como `_cancelado` (no `_pt`).
  2. No existe ninguna categoría `_espera` en las instrucciones dadas a Gemini.
* **Impacto:**
  Estas ramas de código nunca se ejecutarán a menos que Gemini "alucine" o decida usar etiquetas no especificadas.
* **Recomendación:**
  Alinear exactamente las etiquetas del prompt con la validación de Javascript, o instruir formalmente a Gemini sobre el uso de `_espera` y `_pt`.

### 2. Operaciones de escritura individuales dentro de bucles (setValue en loops)
* **Archivo/Línea:** `Script`, Función `procesarComentariosIA()`.
* **Descripción:**
  Dentro del bucle de procesamiento de respuestas (hasta 50 por lote), el script realiza llamadas directas a la API de Sheets para actualizar celdas individuales:
  ```javascript
  hojaMaestra.getRange(item.fila, idxComentario + 1).setValue(nuevoComentario);
  ...
  hojaMaestra.getRange(item.fila, idxStatusIA + 1).setValue('_visto_ia');
  ```
* **Impacto:**
  Cada `setValue` realiza una petición síncrona a los servidores de Google Sheets. Esto degrada fuertemente el rendimiento y es la causa común del error de Google Apps Script: *"Exceeded maximum execution time"* (límite de 6 minutos) si el volumen de filas aumenta.
* **Recomendación:**
  Modificar el arreglo de datos en memoria y escribir por lotes/bloques continuos usando `setValues()`, o acumular rangos para realizar actualizaciones masivas si es posible.

---

## ℹ️ Severidad Baja: Hardcoding de Identificadores y Mantenibilidad

### 3. IDs de Google Drive y Libros Hardcodeados
* **Archivo/Línea:** Múltiples funciones.
  * Línea 56: `const ID_CARPETA_REPORTE = "1GNj0BCfFjIDNj9ETpm5oZHMsWOrPj0cn";`
  * Línea 1202: `const ID_CARPETA = '15pOR8HT5N2MfEr-hSJcLtbcGOCJd8JkS';`
  * Línea 1203: `const ID_CARPETA_HISTORIAL = '1rRlxxqukHsWwa-BGBQcH6-KA7EyHzo7e';`
  * Línea 1722: `var idLibro = '1tgROOAnGSGyeG0di86dSy9VqZv8bWmneWZNtYV05dGA';`
* **Descripción:**
  Identificadores clave de carpetas y hojas de cálculo están escritos directamente en el código fuente.
* **Impacto:**
  Si las carpetas son recreadas, migradas a unidades compartidas o si cambia de propietario la cuenta, el sistema fallará por completo hasta que un desarrollador edite el código fuente manualmente.
* **Recomendación:**
  Centralizar estos identificadores en la pestaña `Config` de Google Sheets o cargarlos utilizando `PropertiesService.getScriptProperties()`.
