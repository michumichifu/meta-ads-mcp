---
name: metodologia-3qs
description: Analiza campañas de Meta (Facebook/Instagram) Ads con la metodologia de las 3 Q's (Que paso / Por que paso / Que haremos) y benchmarks semaforo, a partir de metricas (del MCP o de un reporte exportado de Ads Manager en CSV/Excel). Usalo para analizar o revisar campañas, diagnosticar por que no convierten, evaluar ROAS/CPL/CPM/CPC/CTR/frecuencia/costo por conversacion, auditar el embudo, detectar fatiga y recomendar que pausar, escalar u optimizar. No para crear campañas.
---

# Metodología 3 Q's — diagnóstico de campañas Meta

Marco para analizar campañas y decir con claridad **qué hacer**. Tres preguntas, en orden:
**1) ¿Qué pasó?** · **2) ¿Por qué pasó?** · **3) ¿Qué haremos?** Funciona sobre las métricas que tengas
(del MCP `ads_get_ad_entities`, o de un CSV/Excel exportado de Ads Manager).

## Antes de juzgar: el objetivo del negocio (por defecto la tabla, opcional afinar)
Un "bueno" genérico no existe: el costo por resultado depende del servicio, la ubicación, el tipo de
creativo, el tipo de campaña, el objetivo y el destino — de la economía de cada cliente. Por eso **NUNCA
bloquees el diagnóstico** esperando que el usuario te dé un número. Procede así:
1. **Por defecto, usa los benchmarks documentados** (las tablas por destino y objetivo de
   `PLAYBOOK-Analizar-Campanas-Meta.md` §1.5b: costo por registro en landing, costo por conversación en
   WhatsApp, ROAS en ventas). Con eso ya coloreas el semáforo y entregas las 3 Q's completas — no falta nada.
2. **Aclara SIEMPRE** que ese umbral es una **referencia general** que varía según los factores de arriba,
   y que el número que de verdad manda es el **objetivo propio del cliente**.
3. **Ofrece la opción, sin frenar el informe:** "¿Uso los objetivos por defecto de la documentación, o me
   pasas el objetivo de este cliente para afinar el semáforo a su economía?" Si te da un objetivo del
   cliente, recalcula con ese; si no, deja el de la tabla y sigue. No lo conviertas en un requisito.

Referencias para calcular el objetivo del cliente (solo cuando quiera afinarlo):
- **Ventas:** ROAS objetivo (`ROAS objetivo ≈ (1 / margen neto) × 1.5` para ganar; margen 30% → ~5x).
- **Clientes Potenciales (form o web):** CPL objetivo (`≈ ticket × tasa de cierre × margen × 0.3`).
- **WhatsApp:** costo por conversación objetivo. **Reconocimiento:** evalúa por CPM y alcance vs periodos.

## Regla inquebrantable — la tabla manda
Usa **SOLO** estos umbrales (o los que calibres con el cliente). **NO** uses "promedios de la industria"
de memoria — suelen ser de otro vertical/mercado/año y **arruinan el diagnóstico**. Si un umbral te
parece exigente, no lo cambies a ojo.

**Resultado principal (por objetivo):**
- **ROAS** vs objetivo: ≥ objetivo 🟢 · ≥ 80% 🟡 · < 80% 🔴.
- **CPL** vs objetivo: ≤ objetivo 🟢 · ≤ 120% 🟡 · > 120% 🔴 (igual para costo por conversación).

**Métricas secundarias (referencia — calíbralas por cuenta):**
| Métrica | 🔴 | 🟡 | 🟢 |
|---|---|---|---|
| CTR (saliente / enlace) | <1% | 1–2% | >2% |
| Tasa conv. Leads/Clics (formulario) | <20% | 20–30% | >30% |
| Tasa conv. Leads/Visitas (sitio web) | <10% | 10–20% | >20% |
| Visitas a landing / Clics salientes | <50% | 50–70% | >70% |
| Video: reproducciones 3s / impresiones | <20% | 20–30% | >30% |
| Video: tiempo promedio | <3s | 3–6s | >6s |
| Métricas de calidad (Meta) | — | — | ABOVE_AVERAGE |

**Frecuencia:** no es buena ni mala por sí sola (para que te conozcan hace falta verte varias veces). La
señal útil es la **tendencia**: si **frecuencia sube Y costo por resultado sube** → fatiga → refrescar
creativos. Palanca: **excluir a los ya convertidos** para no re-mostrarles el anuncio.

## El costo por resultado es un promedio — léelo en varios niveles
El "costo por resultado general" es el de la **campaña sobre todo el rango de días** (un promedio que
esconde el detalle). Desglósalo para tener más data: **campaña por día** = la tendencia (sube/baja, lo que
graficas) · **por conjunto (adset)** = variación por público/ubicación · **por anuncio** = qué pieza rinde
· **por anuncio y por día** = el máximo detalle (qué anuncio, qué día). Cuando algo se ve mal o bien, baja
de nivel hasta el anuncio/día que lo explica. Detalle en `PLAYBOOK-Analizar-Campanas-Meta.md` §1.5b.

## Las 3 Q's
1. **¿Qué pasó?** — el resultado principal vs el objetivo (ej. "CPL US$X vs objetivo US$Y → Z% por encima
   🔴"), gasto, resultados.
2. **¿Por qué pasó?** — baja por las métricas secundarias (CTR, tasa de conversión, video, tendencia de
   frecuencia, CPM) y encuentra el **eslabón 🔴 del embudo** (impresiones → clics → landing → checkout →
   compra/registro).
3. **¿Qué haremos?** — según el diagnóstico:
   - Resultado ≥ objetivo → **escalar** (subir presupuesto ~20% cada 3 días, sin resetear el aprendizaje).
   - Resultado entre 80% y objetivo → **optimizar** el paso del embudo con % más bajo; estás cerca, no pauses.
   - Resultado < 80% del objetivo → auditar urgente el embudo; revisar margen/precio; pausar o reducir.
   - CTR bajo → nuevos creativos (gancho/oferta más clara). Tasa de conversión baja → simplificar el
     formulario o mejorar la landing/oferta. Frecuencia subiendo + costo subiendo → refrescar y excluir
     convertidos.

Para el **cómo técnico** de traer las métricas por el MCP y hacer un informe visual, ver el playbook
`PLAYBOOK-Analizar-Campanas-Meta.md`. Para la salud del píxel, el skill `monitor-pixel`.
