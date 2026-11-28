---
name: clima
description: Consulta el clima actual y el pronóstico de los próximos días para Lima, Perú por defecto (o una ciudad indicada). Úsala cuando pida "el clima", "qué tiempo hace", "va a llover", temperatura o pronóstico.
argument-hint: "[ciudad opcional]"
allowed-tools: Bash(curl *)
---

# Clima local

Obtén el clima con wttr.in, que no requiere API key. Si no se indica ciudad, la ubicación por defecto es Lima, Perú.

## Pasos

1. Determina la ubicación:
   - Si el usuario pasó una ciudad en `$ARGUMENTS`, úsala (reemplaza espacios por `+`, ej. `Buenos+Aires`).
   - Si no, usa por defecto `Lima,Peru`.

2. Consulta los datos en JSON:

       curl -s "https://wttr.in/<ciudad>?format=j1&lang=es"

   Por defecto: `curl -s "https://wttr.in/Lima,Peru?format=j1&lang=es"`.

3. Del JSON usa:
   - `nearest_area[0]`: `areaName[0].value` y `country[0].value` (ubicación detectada).
   - `current_condition[0]`: `temp_C`, `FeelsLikeC`, `humidity`, `windspeedKmph`, `winddir16Point`, `precipMM`, `lang_es[0].value` (descripción).
   - `weather[0..2]`: `date`, `mintempC`, `maxtempC` y `hourly[*].chanceofrain` (usa el máximo como probabilidad de lluvia).

4. Responde siempre en español, de forma breve:
   - Ubicación consultada (Lima, Perú por defecto).
   - Condiciones actuales: temperatura, sensación térmica, humedad, viento.
   - Pronóstico de 3 días: mín/máx y probabilidad de lluvia.
   - Una recomendación corta (paraguas, abrigo, etc.) solo si aplica.

## Manejo de errores

- Si `curl` falla o devuelve algo que no es JSON, informa que no se pudo obtener el clima y sugiere reintentar más tarde o indicar otra ciudad.
- No inventes datos: si falta un campo, omítelo.
- Usa unidades métricas (°C, km/h, mm).
