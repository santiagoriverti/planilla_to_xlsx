# Contexto del proyecto — planilla_to_xlsx

## Objetivo

Convertir un PDF de planillas manuscritas escaneadas ("Seguimiento de Siembra a Cosecha") a un archivo Excel estructurado, usando Llama 4 Vision via Groq API, ejecutado en Google Colab.

---

## PDF a procesar

- **Archivo:** `Seguimiento de Siembra a Cosecha.pdf`
- **Ubicación:** Carpeta Downloads del usuario
- **Páginas:** 44 páginas de tablas manuscritas escaneadas
- **Orientación:** Horizontal (landscape), 2103×1488 px a 2.5x zoom

---

## Estructura de la planilla (verificada comparando PDF vs Excel)

Cada página tiene ~10-11 variedades con esta estructura de columnas:

| Cols | Columnas | Comportamiento |
|---|---|---|
| 1–7 | VARIEDAD, NRO_ESP, FECHA_MOJ, NRO_CAJ, PORC_FALL, FECHA_CH, Z_CH | Escritas UNA VEZ por variedad (en la primera sub-fila) |
| 8–18 | FECHA_TQE, NRO_T, NRO_C, LADO, TRASP, DIAS_7..28, COSECHA, RESTO | Se repiten en 1 a 3 sub-filas por variedad |

**Nota:** FECHA_CH y Z_CH están vacías (guiones) en muchas páginas — es correcto que aparezcan como NaN en el Excel.

---

## Modelo y API

- **Modelo:** `meta-llama/llama-4-scout-17b-16e-instruct` via Groq
- **max_tokens:** `8192` (máximo permitido por Groq para este modelo)
- **Ventana de contexto:** 131,072 tokens (input)
- **Precio (Dev tier):** $0.11/M tokens input · $0.34/M tokens output
- **Imágenes:** 100 DPI, JPEG calidad 85 (~11,000 tokens/página)

---

## Problema resuelto: límite de tokens

- Free tier de Groq: 500,000 tokens/día → solo 11 páginas/día a 200 DPI/PNG
- **Solución elegida:** upgrade al Dev tier de Groq (pago por uso, ~$0.25 para 44 págs)
- **Fix en código:** bajado a 100 DPI + JPEG como respaldo si se vuelve al free tier

---

## Ejecución parcial realizada (2026-05-28)

- Páginas 1–11: procesadas (pero con bugs — ver abajo)
- Páginas 12–44: pendientes (fallaron por límite TPD del free tier)
- **Pendiente:** volver a ejecutar todo desde página 1 con el código corregido y Dev tier activo

---

## Bugs encontrados (comparando PDF real vs Excel generado)

### Bug 1 — Filas con valores fusionados usando pipes ✅ CORREGIDO
- **Síntoma:** `FECHA_TQE = "4-3 | 5-3"`, `NRO_T = "6 | 7"`, etc.
- **Causa:** El modelo fusionaba las 2-3 sub-filas de una variedad en una sola fila usando ` | ` como separador
- **Fix:** `split_pipe_row()` en Celda 6 expande esas filas en múltiples filas separadas

### Bug 2 — Filas vacías con VARIEDAD = NaN ✅ CORREGIDO
- **Síntoma:** Fila con solo NRO_ESP=2 y todo lo demás vacío
- **Causa:** El modelo no podía leer el nombre de la variedad (probablemente en el borde de la imagen)
- **Fix:** Filtro que elimina filas con VARIEDAD vacía/NaN

### Bug 3 — Cols 1–7 vacías en sub-filas 2 y 3 ✅ CORREGIDO
- **Síntoma:** Sub-filas 2+ tienen VARIEDAD="" en vez de repetir el valor de la sub-fila 1
- **Fix:** `ffill` por PAGINA en Celda 6 propaga los valores hacia abajo

### Bug 4 — Valores apilados sin pipe en COSECHA/RESTO ⚠️ MEJORADO EN PROMPT
- **Síntoma:** `RESTO = "256  21  16"` — el modelo puso los COSECHA de 3 sub-filas en una sola celda con espacios
- **Ejemplo real:** Flandria, página 1 del Excel (vieja ejecución)
- **Fix:** Reglas 5 y 6 en el prompt prohíben explícitamente esto
- **Nota:** No es 100% corregible en post-procesado sin riesgo de romper otros datos

---

## Post-procesado en Celda 6 (orden de operaciones)

1. `split_pipe_row()` — expande filas con separador `|`
2. Filtro `VARIEDAD != NaN/""` — elimina filas vacías
3. `ffill` por PAGINA — propaga cols 1-7 hacia sub-filas sin esa info
4. `reset_index()` — índice limpio

---

## Posibles mejoras futuras

- [ ] Agregar parámetro `START_PAGE` para retomar desde una página específica
- [ ] Guardar resultados parciales para no perder progreso si Colab se desconecta
- [ ] Revisar manualmente las filas donde FECHA_TQE=NaN pero hay datos en otras columnas (e.g. Lalique, pág 3)
- [ ] Revisar manualmente filas donde COSECHA o RESTO tienen múltiples valores separados por espacios

---

## Notas sobre la API key de Groq

- La API key es la misma antes y después del upgrade al Dev tier
- **NUNCA commitear la API key al repositorio**

---

## Repositorio

- GitHub: https://github.com/santiagoriverti/planilla_to_xlsx
- Rama principal: `main`
