# Contexto del proyecto — planilla_to_xlsx

## Objetivo

Convertir un PDF de planillas manuscritas escaneadas ("Seguimiento de Siembra a Cosecha") a un archivo Excel estructurado, usando Llama 4 Vision via Groq API, ejecutado en Google Colab.

---

## PDF a procesar

- **Archivo:** `Seguimiento de Siembra a Cosecha.pdf`
- **Ubicación:** Carpeta Downloads del usuario
- **Páginas:** 44 páginas de tablas manuscritas escaneadas
- **Tamaño:** ~25.5 MB

---

## Modelo de IA

- **Modelo:** `meta-llama/llama-4-scout-17b-16e-instruct` (Groq)
- **max_tokens:** `8192` — es el máximo permitido por Groq para este modelo
- **Ventana de contexto:** 131,072 tokens (input)
- **Precio (Dev tier):** $0.11/M tokens input · $0.34/M tokens output

---

## Problema principal: límite de tokens por día

### Síntoma
Al ejecutar el notebook, las primeras 11 páginas procesaban OK y luego fallaban todas con error 429:
```
Rate limit reached: Limit 500,000 TPD (tokens per day), Used 499,981
```

### Causa
- Free tier de Groq: límite de **500,000 tokens/día**
- A 200 DPI con PNG: cada imagen consume ~45,000 tokens → solo 11 páginas/día

### Solución aplicada en el notebook del repo
Cambiar en la Celda 4:
- `dpi=200` → `dpi=100`
- Formato PNG → **JPEG (calidad 85)**
- Resultado: ~11,000 tokens/página → 44 páginas entran en free tier

### Solución alternativa (elegida por el usuario)
El usuario hizo **upgrade al Dev tier de Groq** (pago por uso).
- Costo estimado para 44 páginas a 200 DPI: **~$0.25 USD**
- Con Dev tier puede ejecutar el notebook original (200 DPI / PNG) sin cambios

---

## Estado de la ejecución (2026-05-28)

| Páginas | Estado |
|---|---|
| 1–11 | ✅ Procesadas correctamente |
| 12–44 | ❌ Fallaron por límite TPD del free tier |

**Pendiente:** Volver a ejecutar el notebook completo con el Dev tier activo.

> **Nota:** Si se quieren conservar las filas ya extraídas (págs 1–11), agregar lógica para retomar desde la página 12. Si no importa, ejecutar desde el principio.

---

## Decisiones técnicas tomadas

| Decisión | Opción elegida | Alternativa descartada |
|---|---|---|
| Plataforma de ejecución | Google Colab | Local (Python no instalado) |
| Modelo de visión | Llama 4 Scout (Groq) | GPT-4o (OpenAI, ~$1.30 para 44 págs) |
| Tier de Groq | Dev (pago, ~$0.25 total) | Free (500K TPD, solo 11 págs/día) |
| DPI de imágenes (en repo) | 100 DPI + JPEG | 200 DPI + PNG (más tokens) |

---

## Posibles mejoras futuras

- [ ] Agregar parámetro `START_PAGE` para retomar desde una página específica sin reprocesar
- [ ] Guardar resultados parciales en Drive para no perder progreso si Colab se desconecta
- [ ] Validar que el JSON devuelto tenga todas las columnas esperadas antes de agregarlo
- [ ] Revisar manualmente las páginas con ERROR en el Excel final

---

## Notas sobre la API key de Groq

- La API key es la misma antes y después del upgrade al Dev tier
- El upgrade es a nivel de cuenta, no de key
- **Nunca commitear la API key al repositorio**

---

## Repositorio

- GitHub: https://github.com/santiagoriverti/planilla_to_xlsx
- Rama principal: `main`
