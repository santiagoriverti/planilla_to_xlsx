# planilla_to_xlsx

Convierte planillas manuscritas escaneadas (PDF) a Excel (.xlsx) usando inteligencia artificial (Llama 4 Vision via Groq).

Diseñado para planillas de **Seguimiento de Siembra a Cosecha**, pero adaptable a cualquier tabla manuscrita con estructura repetida.

---

## Abrir en Google Colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/santiagoriverti/planilla_to_xlsx/blob/main/seguimiento_siembra_cosecha.ipynb)

No necesitás instalar nada en tu computadora. Todo corre en la nube.

---

## Qué hace

- Lee un PDF de múltiples páginas con tablas manuscritas
- Convierte cada página a imagen
- Usa Llama 4 Vision (via Groq) para interpretar la escritura a mano y extraer los datos
- Genera un archivo `.xlsx` con todas las filas estructuradas y columnas identificadas

### Columnas que extrae

| Columna | Descripción |
|---|---|
| SEMANA | Número de semana (encabezado de página) |
| AÑO | Año (encabezado de página) |
| PAGINA | Número de página del PDF |
| VARIEDAD | Nombre de la variedad |
| NRO_ESP | Número de espacio |
| FECHA_MOJ | Fecha de mojado |
| NRO_CAJ | Número de caja |
| PORC_FALL | Porcentaje de falla |
| FECHA_CH | Fecha CH |
| Z_CH | Zona CH |
| FECHA_TQE | Fecha de toque |
| NRO_T | Número de toque |
| NRO_C | Número de caja (sub) |
| LADO | Lado |
| TRASP | Trasplante |
| DIAS_7 | Registro 7 días |
| DIAS_14 | Registro 14 días |
| DIAS_21 | Registro 21 días |
| DIAS_28 | Registro 28 días |
| COSECHA | Cosecha |
| RESTO | Resto |

---

## Requisitos

- Cuenta de Google (para usar Google Colab)
- API key de Groq **gratuita** (sin tarjeta de crédito)
- El archivo PDF con las planillas escaneadas

---

## Instrucciones paso a paso

### 1. Obtener la API key de Groq (gratis, sin tarjeta, 2 minutos)

1. Entrá a **[console.groq.com](https://console.groq.com)**
2. Creá una cuenta con tu Gmail
3. Hacé clic en **"API Keys"** → **"Create API key"**
4. Copiá la key generada (empieza con `gsk_...`)

> Groq ofrece un tier gratuito con 1000 requests/día y 30 requests/minuto — más que suficiente para procesar el PDF completo.

---

### 2. Abrir el notebook en Colab

Hacé clic en el botón:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/santiagoriverti/planilla_to_xlsx/blob/main/seguimiento_siembra_cosecha.ipynb)

O entrá a [colab.research.google.com](https://colab.research.google.com), luego `Archivo → Abrir notebook → GitHub` y buscá `santiagoriverti/planilla_to_xlsx`.

---

### 3. Pegar la API key

En la primera celda de código del notebook, pegá tu API key:

```python
API_KEY = "gsk_..."  # Pegá acá tu key
```

---

### 4. Ejecutar el notebook

Hacé clic en `Runtime → Run all` (o `Entorno de ejecución → Ejecutar todo`).

El proceso:
1. Instala las dependencias automáticamente
2. Te pide que selecciones el archivo PDF desde tu computadora
3. Convierte cada página a imagen
4. Procesa cada página con Llama 4 Vision (~3 segundos por página)
5. Para 44 páginas: aproximadamente **3-4 minutos**
6. Al terminar, descarga automáticamente el archivo `seguimiento_siembra_cosecha.xlsx`

---

### 5. Revisar el resultado

El Excel descargado contiene una fila por cada entrada de la tabla original. Si una variedad tenía múltiples sub-filas (distintas fechas de toque), aparece repetida con sus datos completos. Las columnas SEMANA y AÑO se extraen del encabezado de cada página.

---

## Tecnologías utilizadas

- [Groq](https://groq.com/) + [Llama 4 Scout Vision](https://groq.com/llama4/) — reconocimiento de texto manuscrito
- [PyMuPDF](https://pymupdf.readthedocs.io/) — conversión de PDF a imágenes
- [Pandas](https://pandas.pydata.org/) — estructuración de datos
- [OpenPyXL](https://openpyxl.readthedocs.io/) — generación del archivo Excel
- [Google Colab](https://colab.research.google.com/) — entorno de ejecución en la nube

---

## Adaptación a otras planillas

Para usar este notebook con otra estructura de tabla, modificá:

1. **El `PROMPT`** en la Celda 5: describí las columnas de tu planilla
2. **`column_order`** en la Celda 6: listá tus columnas en el orden deseado

---

## Licencia

MIT
