---
name: clima
description: Consulta la temperatura y el estado del clima actual de una ciudad (por defecto Colima, México) usando Open-Meteo. Úsala cuando el usuario pida el clima, la temperatura o el estado del tiempo.
argument-hint: "[ciudad]"
---

# Clima

Obtiene el clima actual de una ciudad y lo reporta en español, en una sola línea.

## Pasos

1. Determina la ciudad: **Colima, México es siempre el valor por defecto**. Solo usa otra ciudad si el usuario la indica explícitamente en el argumento; en cualquier otro caso (sin argumento, o una petición genérica como "¿qué clima hace?"), usa Colima.
2. Obtén las coordenadas:
   - Colima, México: `latitude=19.2433`, `longitude=-103.725` (no hace falta geocodificar).
   - Otra ciudad: `curl -s "https://geocoding-api.open-meteo.com/v1/search?name=<ciudad>&count=1&language=es"` y toma `latitude` y `longitude` del primer resultado.
3. Consulta el clima actual:

   ```
   curl -s "https://api.open-meteo.com/v1/forecast?latitude=<lat>&longitude=<lon>&current=temperature_2m,apparent_temperature,relative_humidity_2m,weather_code,wind_speed_10m&timezone=auto"
   ```

4. Traduce `current.weather_code` (código WMO) a texto en español:

   | Código | Estado |
   |---|---|
   | 0 | Despejado |
   | 1, 2 | Mayormente despejado / parcialmente nublado |
   | 3 | Nublado |
   | 45, 48 | Niebla |
   | 51, 53, 55 | Llovizna |
   | 61, 63, 65 | Lluvia |
   | 66, 67 | Lluvia helada |
   | 71, 73, 75, 77 | Nieve |
   | 80, 81, 82 | Chubascos |
   | 85, 86 | Chubascos de nieve |
   | 95 | Tormenta eléctrica |
   | 96, 99 | Tormenta con granizo |

5. Responde en una línea con este formato:

   `<Ciudad>, <hora local>: <temp> °C (sensación <aparente> °C), <estado>, humedad <h> %, viento <v> km/h.`

## Notas

- Open-Meteo actualiza cada 15 minutos (`current.interval`), así que consultas repetidas pueden devolver la misma lectura.
- Si `curl` falla o la ciudad no se encuentra, dilo claramente en lugar de inventar datos.
- Todo el texto al usuario va en español.
