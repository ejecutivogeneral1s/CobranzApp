# 🤖 CobranzApp: Motor de Automatización de Cobranza

> Sistema automatizado para el control de cobranza de pólizas de seguros. Integra Google Workspace y la API de Gemini (IA) para leer historiales, inferir pagos conciliados, y generar/enviar reportes consolidados a promotorías y agentes sin intervención manual.

## 🛠️ Stack Tecnológico
* **Lenguaje Core:** JavaScript (ES6) / Google Apps Script
* **Base de Datos:** Google Sheets
* **Almacenamiento:** Google Drive API (Enrutamiento dinámico)
* **Notificaciones:** Gmail API
* **Inteligencia Artificial:** Google Gemini 2.5 Flash API (Zero-shot classification)

## 🚀 Funcionalidades Principales

* **🧠 Procesamiento de Lenguaje Natural (NLP):** Conexión con Gemini 2.5 Flash para analizar y clasificar comentarios de agentes de forma masiva (lotes de 50) usando un output estricto en JSON para evitar alucinaciones.
* **📂 Enrutamiento Dinámico en Drive:** Algoritmo que construye y navega automáticamente por estructuras jerárquicas de carpetas (`Año > Mes > Día > Vendedor`) para almacenar los reportes generados.
* **⚙️ Pipeline de Reportes:** Segmentación de una base de datos maestra en reportes individuales (XLSX) inyectando fórmulas y formatos condicionales complejos directamente desde el código.
* **🕵️‍♂️ Inferencia de Datos:** Lógica de negocio capaz de deducir recibos conciliados comparando deltas temporales entre la base de datos histórica y la extracción actual del CRM.
* **📧 Distribución Automatizada:** Despliegue automatizado de notificaciones vía correo electrónico usando Gmail API, adjuntando blobs de datos convertidos al vuelo a Excel.

## 🏗️ Arquitectura del Proyecto

El sistema se compone de módulos independientes orquestados para ejecutarse en cadena o bajo demanda:

* **`procesarReportesQuattroXLSX()`:** El motor de integración (ETL). Ingiere reportes en bruto, aplica retención de datos, audita catálogos (grupos y claves) y sincroniza la base de datos principal sin duplicar registros.
* **`procesarComentariosIA()`:** El analista virtual. Extrae gestiones pendientes, consulta a la API de Gemini mediante prompts estructurados y actualiza la base de datos con las clasificaciones.
* **`generarReportes(...)`:** Los constructores. Filtran los datos en memoria, aplican reglas de negocio (vendedores, ramos, antigüedad), inyectan fórmulas/formato condicional y generan archivos `.xlsx` limpios.
* **`obtenerOCrearRutaDinamica()`:** El gestor de archivos. Algoritmo recursivo que garantiza que cada reporte se guarde en una estructura de carpetas estandarizada en Drive.
