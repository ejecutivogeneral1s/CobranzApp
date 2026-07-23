# 📊 Reporte de Auditoría y Static Analysis - CobranzApp

Este documento resume los hallazgos identificados en el código de **CobranzApp** (archivo `Script`) tras un análisis estático detallado. Se han clasificado por severidad para facilitar su revisión y resolución.

---

## 🔍 Resumen Ejecutivo
El sistema está estructurado de manera modular y robusta para la automatización en Google Workspace. Sin embargo, se ha detectado **1 Bug Crítico de Integración con IA**, así como áreas de mejora significativas en rendimiento, hardcoding de identificadores, y discrepancias en los diccionarios de clasificación de comentarios.

---

## 🛑 Severidad Alta: Bug Crítico de Integración con IA (`_promesa`)

### 1. Descarte de la etiqueta `_promesa` en la clasificación
* **Archivo/Línea:** `Script`, Función `procesarComentariosIA()` (Línea ~110-115) en relación con `llamarGeminiPorLote()` (Línea ~202).
* **Descripción:**
  En el prompt de Gemini (`llamarGeminiPorLote`), se instruye explícitamente al modelo a clasificar promesas de pago bajo la etiqueta **`_promesa`**:
  ```javascript
  - "_promesa" : Si el cliente prometió pagar en una fecha futura específica.
  ```
  Sin embargo, en el bucle que procesa las respuestas recibidas (`procesarComentariosIA`), la condición `if` que valida y guarda las etiquetas es la siguiente:
  ```javascript
  if (tagIA === "_pagado" || tagIA === "_cancelado" || tagIA === "_pt" || tagIA === "_reexpedido" || tagIA === "_espera") {
  ```
  **`_promesa` no está incluida en esta condición.**
* **Impacto:**
  Cuando Gemini clasifica correctamente un comentario como `_promesa`, la condición evalúa a `false`. El script cae en el bloque `else`, marcando la celda de control técnico como `_visto_ia` y **descartando por completo la etiqueta `_promesa`** sin agregarla al comentario ni guardarla en la bitácora/reporte.
* **Recomendación:**
  Añadir `tagIA === "_promesa"` a la validación de etiquetas válidas en `procesarComentariosIA()`.

---

## ⚠️ Severidad Media: Discrepancias de Etiquetas e Ineficiencia (GAS Anti-patterns)

### 2. Discrepancias de Etiquetas de IA (`_pt`, `_espera`)
* **Archivo/Línea:** `Script`, Función `procesarComentariosIA()` vs `llamarGeminiPorLote()`.
* **Descripción:**
  La validación en `procesarComentariosIA()` incluye `_pt` y `_espera`. Sin embargo:
  1. En el prompt de Gemini, se le pide clasificar pérdidas totales como `_cancelado` (no `_pt`).
  2. No existe ninguna categoría `_espera` en las instrucciones dadas a Gemini.
* **Impacto:**
  Estas ramas de código nunca se ejecutarán a menos que Gemini "alucine" o decida usar etiquetas no especificadas.
* **Recomendación:**
  Alinear exactamente las etiquetas del prompt con la validación de Javascript, o instruir formalmente a Gemini sobre el uso de `_espera` y `_pt`.

### 3. Operaciones de escritura individuales dentro de bucles (setValue en loops)
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

### 4. IDs de Google Drive y Libros Hardcodeados
* **Archivo/Línea:** Múltiples funciones.
  * Línea 56: `const ID_CARPETA_REPORTE = "1GNj0BCfFjIDNj9ETpm5oZHMsWOrPj0cn";`
  * Línea 1203: `const ID_CARPETA = '15pOR8HT5N2MfEr-hSJcLtbcGOCJd8JkS';`
  * Línea 1204: `const ID_CARPETA_HISTORIAL = '1rRlxxqukHsWwa-BGBQcH6-KA7EyHzo7e';`
  * Línea 1723: `var idLibro = '1tgROOAnGSGyeG0di86dSy9VqZv8bWmneWZNtYV05dGA';`
* **Descripción:**
  Identificadores clave de carpetas y hojas de cálculo están escritos directamente en el código fuente.
* **Impacto:**
  Si las carpetas son recreadas, migradas a unidades compartidas o si cambia de propietario la cuenta, el sistema fallará por completo hasta que un desarrollador edite el código fuente manualmente.
* **Recomendación:**
  Centralizar estos identificadores en la pestaña `Config` de Google Sheets o cargarlos utilizando `PropertiesService.getScriptProperties()`.

---

## 🛠️ Plan de Acción Propuesto

1. **Resolver el Bug Crítico:** Corregir la validación de `_promesa` en la función `procesarComentariosIA()` para asegurar que todas las promesas de pago identificadas por la IA se guarden correctamente.
2. **Corrección Opcional:** Aliviar las llamadas a `setValue` convirtiéndolas en un proceso más agrupado o documentando su límite para futuras optimizaciones de escala.
3. **Mantenibilidad:** Si se desea, mover los IDs de Drive hardcodeados hacia la configuración del script o del libro.
