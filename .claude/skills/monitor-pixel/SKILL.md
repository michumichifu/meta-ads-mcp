---
name: monitor-pixel
description: Monitorea la salud de un Pixel (conjunto de datos / dataset) de Meta Ads — volumen de eventos, calidad de coincidencia (EMQ) y una ALERTA si algun evento cayo mas del 20% en los ultimos 7 dias vs los 7 previos. Usalo SIEMPRE que el usuario mencione revisar/auditar/monitorear el pixel, salud del pixel, eventos del pixel, EMQ o calidad de coincidencia, caida de eventos, alerta de eventos, conjunto de datos / dataset de Meta, o sospeche que el pixel dejo de registrar o perdio calidad.
---

# Monitor de píxel de Meta

Revisa la salud de un Pixel (Meta lo llama ahora "conjunto de datos" / *dataset*) y avisa cuando algo se
rompe. Responde 3 preguntas: **¿entran los eventos?**, **¿con qué calidad de coincidencia?**, y **¿alguno
cayó de forma alarmante esta semana?**

## Requisito: el `<DATASET_ID>` (ID del píxel)
Necesita un **dataset ID**. Si el usuario lo dio, úsalo directo. Si no, pídelo (Events Manager → el
conjunto de datos → el número bajo el nombre). No lo adivines: correr el reporte sobre el píxel
equivocado es peor que no correrlo.

## Flujo
1. **Ventana y corte** (bash): `NOW=$(date +%s)`, `MIDPOINT=$((NOW-604800))` (hace 7 días),
   `START=$((NOW-1209600))` (hace 14 días). Consultas UNA ventana de 14 días; se parte en dos mitades de 7.
   (`ads_get_dataset_stats` solo mira **28 días atrás**, así que 7d vs 7d entra sin problema.)
2. **Identidad y estado:** `ads_get_dataset_details(<DATASET_ID>)` → nombre, si está activo, último evento
   (navegador y servidor/CAPI).
3. **Calidad (EMQ):** `ads_get_dataset_quality(<DATASET_ID>)` → EMQ por evento + cobertura de llaves
   (email, teléfono, external_id…).
4. **Volumen — una consulta POR evento:** para cada evento de `quality`, llama
   `ads_get_dataset_stats(<DATASET_ID>, aggregation="event", event_name="<EVENTO>", start_time=START,
   end_time=NOW)`. **Evento por evento** (no todos juntos): en píxeles grandes la respuesta se corta.
   Usa `aggregation="event"` (NO `"event_total_counts"`, que duplica sin etiquetar).
5. **Calcular caídas** (con cuidado, no "a ojo"): suma los buckets por evento y repártelos en las dos
   ventanas de 7 días según el timestamp; calcula el cambio %.

## La regla de alerta (evita falsos positivos)
Un evento dispara **ALERTA** solo si cumple **las dos**:
1. Cayó **más del 20%** en los últimos 7 días vs los 7 previos.
2. La ventana previa tuvo **≥ 50 eventos** (piso de ruido). Razón estadística: en conteos el ruido
   relativo ronda **1/√N**; un evento con 6 ocurrencias puede "caer 30%" solo por azar — alertar sobre eso
   entrena al usuario a ignorar el reporte. Los eventos bajo el piso se reportan (marcados "bajo volumen")
   pero **no alertan**. Un evento que existía y **desaparece** (0 esta semana) = caída de -100% → alerta
   fuerte si tenía volumen.

## Cómo leer el EMQ
Puntaje 0–10 por evento: **≥8 excelente · 6–7.9 aceptable · 4–5.9 bajo · <4 crítico**. Lo que más lo sube
es la cobertura de **llaves fuertes** (email, teléfono, external_id) — marca las de cobertura débil. Ojo:
el EMQ es **foto de hoy**, sin histórico → no detecta "caídas de EMQ" en el tiempo, solo el estado actual.

## Presentar
**Si hay alertas, ponlas primero** y dilo de frente — el punto es que una caída no pase desapercibida.
Agrega 1–2 frases de lectura si aporta (p. ej. si la caída coincide con un cambio conocido en la web),
pero **no inventes causas**. Pensado también para correr como tarea programada (semanal): ahí el
`<DATASET_ID>` debe venir en el prompt.
