# Análisis de campañas + Informe visual (Meta Ads MCP)

> Todos los valores de este playbook (IDs de cuenta/página/píxel, números de teléfono, nombres como «Clínica Dental Aurora» o «Dra. Aurora Peña») son **ejemplos ficticios**. Reemplázalos por los de tu propia cuenta.

Guía reproducible para: (1) analizar campañas de un cliente vía MCP, y (2) generar un
informe HTML visual, offline y cliente-facing. Sirve para cualquier cliente con campañas de captación
de leads similares (dental, servicios).

---

## PARTE 1 — Análisis de campañas (Meta Ads MCP)

Todas las tools de Meta son `mcp__meta-ads__ads_*`. Cargarlas con ToolSearch antes de usarlas.
Si `tools/list` da timeout, arrancar con `MCP_TIMEOUT=120000 MCP_TOOL_TIMEOUT=120000 claude`.

### 1.1 Ubicar la cuenta
- `ads_get_ad_accounts` → lista cuentas. Filtrar por `is_ads_mcp_enabled:true` e `is_queryable:true`.
  Si `is_queryable:false`, mostrar `not_queryable_reason` y parar (cuenta marcada/deshabilitada).

### 1.2 Píxel / dataset (señales de conversión)
- `ads_get_datasets` (ad_account_id) → píxeles del cliente.
- `ads_get_dataset_details` → estado, `last_fired_time` (navegador) y `server_last_fired_time` (CAPI).
  También trae `first_party_cookie_enabled` y el **modo de carga**: la mayoría de eventos entra **por
  hora** (batch) y el de conversión suele entrar en **tiempo real** — es normal, no falta nada.
- `ads_get_dataset_stats` → volumen de eventos. Parámetro clave `event_source`:
  `WEB_ONLY` (píxel/navegador) vs `SERVER_ONLY` (API de Conversiones). Comparar ambos para
  confirmar que las conversiones llegan por **CAPI** (más fiable). Ventana máx. 28 días.
- `ads_get_dataset_quality` → **EMQ** (Event Match Quality, /10) y cobertura de claves.
  Eventos de conversión bien parametrizados suelen dar EMQ ~7.8 (100% teléfono/email/fbp/fbc).
  Los eventos de **navegación** (PageView, ScrollDepth, TimeOnPage) dan EMQ **naturalmente más bajo**
  (~6.2–6.4): llevan menos datos personales; es lo esperado, no lo leas como alerta.

**Concepto clave:** el evento que optimiza la campaña de landing (p. ej. `CompleteRegistration`,
personalizado) normalmente se dispara por **CAPI**, con el píxel como respaldo deduplicado.
`FormSubmit` es el evento previo; `CompleteRegistration` es el registro validado = el "resultado".
Nota: los eventos personalizados del dataset NO aparecen en `ads_get_customconversions`
(eso lista solo "custom conversions" formales); es normal.

### 1.2b Salud del píxel / alerta de caídas de eventos
Chequeo (ideal semanal) de que el píxel/dataset sigue sano. Responde 3 preguntas: ¿entran eventos?,
¿con qué EMQ?, ¿alguno cayó de forma alarmante?
- `ads_get_dataset_details(1122334455667788)` → nombre, si está activo, último evento (navegador y CAPI).
- `ads_get_dataset_quality(1122334455667788)` → **EMQ** por evento + cobertura de llaves. Escala: **≥8
  excelente, 6–7.9 aceptable, 4–5.9 bajo, <4 crítico**; lo que más lo sube es la cobertura de llaves
  fuertes (email, teléfono, external_id). Ojo: el EMQ es **foto de hoy**, sin histórico (no hay "caídas
  de EMQ" en el tiempo, solo estado actual).
- `ads_get_dataset_stats(1122334455667788, aggregation="event", event_name="Lead", start_time, end_time)`
  → volumen por evento. **Consultar evento por evento** (no todos juntos): en píxeles grandes la
  respuesta se corta. Usar `aggregation="event"` (NO `"event_total_counts"`, que duplica sin etiquetar).
  Ventana: últimos 7 días vs los 7 previos (una sola consulta de 14 días; el stats solo llega 28 días
  atrás, así que entra).

**Regla de alerta (evita falsos positivos):** un evento dispara ALERTA solo si cumple **las dos**:
1. Cayó **más del 20%** en los últimos 7 días vs los 7 previos.
2. La ventana previa tuvo **≥50 eventos** (piso de ruido). Razón: en conteos el ruido relativo ronda
   1/√N, así que una caída del 30% en un evento con 6 ocurrencias es puro azar. Eventos por debajo del
   piso se reportan (marcados "bajo volumen") pero no alertan. Un evento que existía y **desaparece**
   (0 esta semana) = caída de -100% → alerta fuerte si tenía volumen.

**Eventos a vigilar:** los de conversión del cliente — el estándar de Meta `CompleteRegistration`,
`Lead`, `Purchase`, `SubmitApplication`, o el personalizado que uses (en lead-gen/WhatsApp suele ser el
registro validado; `FormSubmit` es el paso previo). Si algo cae, **ponlo primero** en el informe y dilo
de frente; agrega 1–2 frases de lectura si aporta, pero no inventes causas.

### 1.3 Jerarquía y métricas — `ads_get_ad_entities`
La tool central. `level` = `account|campaign|adset|ad`.
- Para métricas hay que pasar rango: `date_preset` **o** `time_range` `{"since","until"}`.
- `time_increment`: `"monthly"`, `"1"` (diario), `"7"` (semanal), o `"all_days"`.
- `filtering`: `[{"field":"campaign.id","operator":"IN","value":[...]}]`.
- `sort`: p. ej. `amount_spent_descending`. `limit` alto (300–500) para histórico por creativo.
- Verificar campos dudosos con `ads_get_field_context` (resuelve alias: `spend`→`amount_spent`).

Campos válidos usados: `id, name, campaign_id, effective_status, objective, optimization_goal,
attribution_setting, amount_spent(=spend), impressions, reach, frequency, clicks, ctr, cpm,
results, cost_per_result, lead, daily_budget`.
- `results` y `cost_per_result` vienen con el nombre del evento entre paréntesis, p. ej.
  `"5 (CompleteRegistration)"` o `"4 (Messaging conversations started)"` → así se identifica el
  objetivo real de cada conjunto sin pedir `promoted_object` (que NO es fetchable a nivel adset).
- `optimization_goal`: `OFFSITE_CONVERSIONS` (landing/web) vs `REPLIES`/`CONVERSATIONS` (WhatsApp).

### 1.4 Creativos (imagen + copy)
- `ads_get_creatives` SIN `creative_ids` devuelve solo id/name/status. Para `body`(=primary text),
  `title`(=headline), `image_url`, `image_hash` → llamar CON `creative_ids` o `fields`.
- `ads_get_ad_preview` (ad_id, ad_format=`MOBILE_FEED_STANDARD`) → devuelve la imagen renderizada
  + copy. **Ojo:** si el anuncio es un post de Instagram promocionado, a veces NO devuelve la
  imagen (solo el iframe); en ese caso, sacar `image_url` del creative y descargarla con curl.
- Para **ver** una imagen: descargarla y usar Read sobre el archivo local.
- Mapeo útil: los anuncios nombrados `01 Imagen 0N` en Meta = tu creativo-N del lote.

### 1.5 Interpretación (lo que da valor real)
- **Hambreado vs. no convirtió:** distinguir un creativo que Meta casi no entregó (poco spend →
  no juzgable) de uno con muchas impresiones y 0 registros (creativo/copy que no caló). Este
  criterio es el que pide el cliente experto.
- **Concentración de presupuesto:** Meta suele volcar el gasto en 1 ganador temprano. Bueno
  mientras la audiencia es fresca; riesgo = sin reemplazo validado cuando ese ganador se fatiga.
- **Fatiga/saturación:** `frequency` sube y `cost_per_result` sube mes a mes con la misma
  audiencia. Es natural; es la señal para **refrescar creativos** (no antes).
- **Nivel de conciencia / ángulo:** cruzar performance con el ángulo del creativo (precio,
  autoridad, prótesis→implante, urgencia, cuotas…). El ángulo "producto + precio" a público
  N5 (casi decidido) suele cerrar barato.

### 1.5b Benchmarks (referencia general) + las 3 Q's
**Léelo antes de usar cualquier número:** estos benchmarks son **referencia general, NO verdad
absoluta.** Dependen del objetivo, el destino, la inversión, la cantidad de creativos, la calidad de la
landing y — sobre todo — de la clínica. La MISMA estrategia (misma imagen, copy y landing, cambiando
solo colores/marca) da resultados **drásticamente distintos** entre dos clínicas o ciudades. Son punto
de partida; **cada clínica construye su propio historial** de lo que le funciona, y el MCP sirve justo
para recoger esa data periódicamente y afinar.

**El costo por resultado es un PROMEDIO — desglósalo para ver la verdad.** El número "general" es el de la
**campaña sobre todo el rango de días**: un promedio que esconde el detalle. Míralo en varios niveles y
tendrás mucha más data para decidir:
- **Campaña, todo el periodo** → el promedio general (el titular del informe).
- **Campaña por día** → la **tendencia**: el costo por resultado sube o baja día a día (esto es lo que graficas).
- **Por conjunto de anuncios (adset)** → varía por público/segmentación/ubicación; revela qué adset arrastra el promedio.
- **Por anuncio, todo el periodo** → el promedio de cada anuncio: cuál rinde y cuál no.
- **Por anuncio y por día** → el **máximo detalle**: qué anuncio, qué día. Aquí está la data más rica para
  hallar el porqué (un creativo fatigado, un día flojo, un adset caro).
No te quedes en el promedio general: cuando algo se ve mal (o bien), **baja de nivel** hasta encontrar el
anuncio/día que lo explica. El mismo criterio aplica a CUALQUIER costo por resultado de las tablas de abajo.

**Costo por resultado — SIEMPRE por destino Y objetivo (el objetivo manda tanto como el destino):**

Landing (objetivo Clientes Potenciales) → evento `CompleteRegistration`, o el **personalizado** que uses
cuando el píxel tiene restricción y no deja el estándar (hace la misma acción). Válido con la landing en
**embudo (formulario ABAJO, info arriba):**

| Costo por registro | Lectura |
|---|---|
| < US$1 | excelente |
| ~ US$1 | bien |
| ~ US$2 | intermedio (ni bien ni mal) |
| > US$2 | empieza a empeorar |

Caveat de calidad: con el formulario **arriba** (primera pantalla) saldrían más registros pero de **mala
calidad** (el usuario no pasa el embudo, no se informa). El embudo con formulario abajo pre-filtra →
menos registros, pero calificados. Ese contexto es parte del benchmark.

WhatsApp (objetivo Clientes Potenciales / Ventas) → **costo por conversación iniciada:**

| Costo por conversación | Lectura |
|---|---|
| < US$0.30 | excelente |
| US$0.30 – 0.50 | muy bueno |
| US$0.50 – 0.70 | bueno |
| US$0.70 – 1.00 | medio |
| > US$1.00 | empieza a degradarse |

WhatsApp (objetivo **Interacción**): mucho más barato (~US$0.20–0.50), pero es **otra finalidad** (no
busca lead/venta) → **NO se compara** con la tabla de arriba.

Ventas (e-commerce, objetivo Ventas/Compra) → se mide por **ROAS** (retorno sobre el gasto), no por costo
por lead. El "ROAS bueno" **depende del margen** — no hay número universal. Define el **ROAS objetivo** con
el cliente (fórmula útil para ganar: `ROAS objetivo ≈ (1 / margen neto) × 1.5`; con margen 30% → ~5x). Y
juzga contra ESE objetivo: ≥ objetivo → 🟢 · ≥ 80% del objetivo → 🟡 (cerca, optimiza, no pauses) · < 80%
→ 🔴 (revisar margen/precio, auditar el embudo o pausar/reducir). Mira el **embudo e-commerce**: impresiones
→ clics → visitas a la landing → checkout (inicio de pago) → compra; el eslabón que más cae es dónde
optimizar (precio, página, oferta, o el creativo si el CTR es bajo).

**Destino según la habilidad de venta de quien atiende:** WhatsApp exige mucha destreza de cierre del
que responde el chat → conviene a clínicas que cierran bien. Si el equipo no vende bien, **landing con
embudo** (pre-filtra y la clínica contacta con los datos) rinde más. Elige el destino por esto, no por
moda.

**Puente landing → WhatsApp (post-registro):** tras el registro puedes cerrar por WhatsApp de dos modos:
(a) el prospecto escribe primero (botón wa.me en la página de gracias), o (b) el negocio lo contacta con
los datos del registro. Combina el pre-filtro del embudo con el cierre por chat.

**Métricas secundarias (referencia general, a calibrar):**

| Métrica | 🔴 | 🟡 | 🟢 |
|---|---|---|---|
| CTR (saliente / enlace) | <1% | 1–2% | >2% |
| Tasa conv. Leads/Clics (formulario) | <20% | 20–30% | >30% |
| Tasa conv. Leads/Visitas (sitio web) | <10% | 10–20% | >20% |
| Visitas/Clics salientes (carga+UX) | <50% | 50–70% | >70% |
| Video: reproducciones 3s / impresiones | <20% | 20–30% | >30% |
| Video: tiempo promedio | <3s | 3–6s | >6s |
| Calidad / Interacción / Conversión (Meta) | — | — | ABOVE_AVERAGE |

**Frecuencia — NO usar umbral fijo.** No es buena ni mala por sí sola: para que te conozcan hace falta
ver el anuncio varias veces (la frecuencia sube y luego el cliente actúa). La señal útil es la
**tendencia**: si la **frecuencia sube Y el costo por resultado sube** → fatiga → refrescar creativos.
Palanca clave: **excluir a los que ya convirtieron** (registrados) para no re-mostrarles el anuncio ni
inflar la frecuencia sin sentido.

**Regla anti-invención:** usa SOLO estos umbrales (o los que calibres con el cliente). NO uses
"promedios de la industria" de memoria — suelen ser de otro vertical/mercado/año y arruinan el
diagnóstico.

**Las 3 Q's (para el diagnóstico y el informe):**
1. **¿Qué pasó?** — resultado principal vs lo esperado (costo por registro/conversación), gasto, leads.
2. **¿Por qué pasó?** — baja por las métricas secundarias (CTR, tasa de conversión, video, tendencia de
   frecuencia, CPM) y encuentra el eslabón débil del embudo.
3. **¿Qué haremos?** — costo alto + embudo débil → optimizar el paso flojo; costo bueno → escalar (~+20%
   cada 3 días sin resetear aprendizaje); CTR bajo → nuevos creativos; frecuencia subiendo + costo
   subiendo → refrescar y excluir convertidos.

---

## PARTE 2 — Tus creativos (contexto de marca + imágenes fuente)

Ten tus creativos ya producidos (imágenes + copy con variantes). Para el informe conviene tener a mano,
por creativo: su ángulo, nivel de conciencia, estilo visual, elemento dominante, el copy
(headlines, primary texts) y la **imagen fuente limpia** subida a **una URL pública** (que se descarga
directo con curl, sin auth → mejor calidad que el preview de Meta).

Si tienes una fuente propia de contexto de marca (colores de la paleta, precios/ofertas, contactos,
guías de creativos, landings, doctores), úsala como **fuente de verdad para colores y ofertas** en el
informe. Si no, sáltate esta parte y toma esos datos del propio material del cliente.

---

## PARTE 3 — Informe HTML (formato y convenciones)

Objetivo: **un solo archivo .html, offline total**, cliente-facing, que venda el trabajo bien
y con honestidad. Se guarda en `~/Downloads` (acceso rápido del usuario) **y** copia en
`informes/aurora-2026-07/`.

### 3.1 Técnica offline (imágenes en base64 sin ensuciar el contexto)
1. Descargar imágenes fuente (tu URL pública de la imagen, o `image_url` de Meta).
2. Optimizar con PIL a **620px de ancho, JPEG q82** (mantener ratio). ~1.2 MB para 20 imágenes.
3. Escribir el HTML con **tokens** `%%LOGO%%`, `%%IMP1%%`…`%%ORT10%%` en los `src`.
4. Inyectar los `data:` URIs con un script Python que lee el HTML y reemplaza tokens en disco
   (así el base64 NUNCA pasa por el contexto del modelo). Ver `_fuente/regenerar.py`.
Resultado: ~1.6 MB, funciona sin conexión (doble clic).

### 3.2 Colores (CSS vars) — marco agencia + paleta cliente
- **Agencia:** azul `#042CE7`, navy `#0a1636`, verde `#10B981`, tinta `#0e1424`.
  Logo real en `logo-icon.png` (círculo azul con barras) → embeber en hero y footer.
- **Cliente:** su paleta propia (ej. bronce `#C39367`, crema `#FAF6F1`, crema2 `#FCEBE2`, oscuro
  `#2C2520`, salvia `#7D9B76`, neutro `#A9A29E`). Tipografías tipo Cormorant Garamond + Jost (fallback
  serif/sans del sistema, para no depender de fuentes externas). **Los colores del cliente salen de tu
  fuente de contexto de marca (o de su material) → cambian por cliente.**

### 3.3 Reglas de estilo (IMPORTANTES — feedback del cliente)
- **Sin emojis.** Usar iconos SVG inline (estilo feather), sin CDN. Incluso la estrella de
  "mejor rendimiento" es SVG, no `★`.
- **Sin guion largo (— em-dash ni – en-dash): delata IA.** Reemplazar por:
  `:` (explicación), `(` `)` (inciso), `·` (títulos/labels), `a` (rangos: "Marzo a Mayo",
  "US$5 a US$8"), `,` o `-` normal según el caso.
- **Moneda:** la cuenta opera en **USD** → mostrar `US$` primero (grande) y debajo, más pequeño,
  el equivalente en **RD$** como **aproximación** a **tasa promedio ~RD$60/US$1** (aclararlo).
- Iconos, charts y todo self-contained (sin librerías externas). Charts = SVG hechos a mano con
  JS a partir de datos embebidos.
- **Galería:** tarjetas con la imagen COMPLETA (`aspect-ratio` = ratio real, `object-fit:contain`,
  no `cover`). Click → **lightbox**: fondo glass (`backdrop-filter:blur`), imagen que crece desde
  el centro con `cubic-bezier(.2,.85,.25,1)`; cerrar con click fuera, botón ✕ o Esc.

### 3.4 Estructura narrativa del informe (secciones)
1. **Hero** (navy agencia): logo, "preparado para Clínica Dental Aurora", titular, periodo, 4 KPIs grandes + nota moneda.
2. **Resumen ejecutivo** (KPI cards).
3. **Filosofía**: "no usamos al cliente como conejillo de indias" (estrategia probada, medición CAPI).
4. **Evolución mes a mes**: 2 charts SVG (registros por servicio + costo por registro). Marcar la
   saturación como natural, no como fracaso.
5. **Fase 1 · Video**: enmarcar el arranque como éxito recorrido (días/semanas), no como prueba.
6. **Fase 2 · Imágenes**: cómo validaron el modelo (menor costo, más rápido, más ángulos).
7. **Optimización**: "lo que funciona no se toca"; refrescar solo al saturarse.
8. **Fase 3 · Refresco (actual)**: galería de los lotes nuevos por servicio, con ángulo + nivel de
   conciencia + señales tempranas. Tabla **día a día** del mes en curso, explicando la fase de
   aprendizaje (Meta necesita ~5 a 7 días) y que el refresco **optimiza sobre todo lo aprendido**
   (video e imágenes, todos los servicios) porque hereda audiencias/píxel/señales.
9. **Silla de tres patas** (reflexión de cierre): 33.3% Agencia (estrategia, creativos, gestión de
   anuncios y gasto) · 33.3% Gestión de citas (atención WhatsApp/redes, seguimiento, agendar) ·
   33.3% Clínica (recepción + personal odontológico; puntualidad y trato). Si una pata falla, se cae.
10. **Cierre** + footer agencia.

### 3.5 Editar / actualizar
**El HTML final (`informes/aurora-2026-07/Informe-*.html` y su copia en `~/Downloads`) es
AUTOCONTENIDO y se edita directo.** Para cambiar textos, cifras, colores o estilos, se edita ese
archivo tal cual (las imágenes y el logo ya van embebidos como `data:` base64). NO hay que
reconstruir nada. (No confundir con los temporales del scratchpad de sesión, que son efímeros.)

El bundle `_fuente/` (plantilla con tokens + `img/` + `regenerar.py`) es **opcional**, solo útil si
se quiere **cambiar imágenes en lote** sin lidiar con el base64: se sustituye `img/imp_N.jpg` u
`ort_N.jpg` (620px) y se corre `python3 _fuente/regenerar.py`. Para editar una sola imagen o
cualquier texto, editar el HTML final directamente es lo más rápido.

### 3.6 Archivo de datos crudos reproducible (companion `.md`)
Junto al informe, guarda un **`DATOS-detallados.md`** con el dataset crudo que usaste, para **no volver a
consultar Meta** después (menos llamadas a la API = más seguro y rápido). Documenta la procedencia en la
cabecera (fuente `ads_get_ad_entities`, cuenta, fecha de corte, si el mes va parcial). Estructura reutilizable:
- **A) Totales por servicio y mes** (nivel campaña).
- **B) Detalle por creativo y mes** — notación compacta `inversión · registros · costo/reg`
  (ej. `$337.85 · 184 · $1.84`); columnas = meses.
- **C) Lotes nuevos** (ángulo · nivel de conciencia · rendimiento temprano).
- **D) Píxel / señales** (EMQ por evento, WEB vs SERVER/CAPI, cobertura de llaves).

---

## Aprendizajes técnicos (trampas concretas encontradas)

- **Adset NO acepta** `promoted_object`, `destination_type`, `billing_event` en `ads_get_ad_entities`.
  Para saber el evento que optimiza, usar `optimization_goal` + el nombre que trae `results`
  (p. ej. `"5 (CompleteRegistration)"`).
- **`ads_get_ad_preview` a veces NO devuelve imagen** (solo `preview_html`/iframe) cuando el
  anuncio es un post de Instagram promocionado. Solución: `ads_get_creatives` con `creative_ids`
  → `image_url` → descargar con curl → Read. (Para el lote nuevo es mejor tu URL pública de la imagen.)
- **El `date_start` del objeto campaña puede engañar.** La campaña decía "1 de julio" pero el
  desglose **diario** (`time_increment:"1"`) mostró que solo hubo datos desde el **3 de julio**.
  Para "cuándo empezó de verdad", mirar el diario, no el `date_start`.
- **Mapeo Meta↔creativo confirmado con datos:** ad `01 Imagen 0N` = tu creativo-N. Verificarlo antes
  de atribuir performance a un ángulo. (Ej.: al inicio se creyó que el ganador implantes "Imagen 008"
  era el ángulo de prótesis; al revisar el lote se confirmó que es **creativo-8 = producto/titanio N5**.
  El nombre en la imagen puede diferir del ángulo interno del creativo.)
- **El sufijo del nombre data la cohorte/refresco:** `Imagen 0N` = tanda base; `Imagen 0N Mayo` =
  refresco de mayo. Distinguir cohortes por el sufijo evita mezclar tandas al comparar rendimiento.
- **Salidas grandes de MCP** (p. ej. un volcado de lote de ~60k chars) se guardan a archivo; leerlas con
  `jq` (no intentar volcarlas al contexto). Ej.: `jq -r '.creatives|sort_by(.index)[]|...' file`.
- **Reach NO es sumable** entre meses/creativos (personas únicas se solapan). Impresiones e
  inversión sí suman; para "personas alcanzadas" total, dar un aproximado.
- **Base64 en el HTML:** inyectar con script en disco, nunca pegar base64 en la conversación
  (satura el contexto). Redimensionar a 620px/q82 antes de incrustar.

## Datos de referencia del cliente (rellena con los tuyos — NO compartas datos de un cliente)
Anota por cliente lo que no quieras re-derivar: cuenta `1234567890123456`, píxel `1122334455667788`, evento de
optimización (p. ej. `CompleteRegistration` / `Lead` / `Purchase`, o el personalizado del cliente),
histórico de resultados (registros/leads, costo por resultado, inversión, impresiones), qué servicios y
formatos rinden mejor, lotes de creativos (`f1e2d3c4-b5a6-7890-1234-56789abcdef0`) y doctores/voceros.

Relacionado: el skill `crear-campanas-meta` (crear campañas), memoria del proyecto.
