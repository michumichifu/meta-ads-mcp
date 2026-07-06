---
name: estrategia-campanas-meta
description: Arma la estructura y estrategia de una cuenta de Meta Ads (embudo de campañas, distribucion de presupuesto, publicos y exclusiones). Usalo cuando el usuario quiera armar su estrategia de Facebook/Instagram Ads, no sepa cuantas campañas crear, como repartir el presupuesto, que publicos usar, que excluir, o como estructurar el embudo (frio, retargeting, clientes). Aplica a Ventas, Clientes Potenciales (formulario o sitio web), Interaccion/WhatsApp y Reconocimiento/local. Los numeros son referencia a calibrar por cliente.
---

# Estructura y estrategia de campañas de Meta Ads

Cómo estructurar una cuenta de Meta Ads: cuántas campañas, con qué presupuesto, qué públicos y qué
exclusiones. **Los porcentajes y umbrales de abajo son una referencia de partida estándar — calíbralos
por cliente/mercado.** Complementa el "cómo técnico" de crear las campañas (skill `crear-campanas-meta`).

## Antes de armar — 4 preguntas
1. **Objetivo/tipo:** ¿Ventas (e-commerce) · Clientes Potenciales por formulario instantáneo · Clientes
   Potenciales por sitio web/landing · Interacción/Mensajes (WhatsApp) · Reconocimiento/marca local?
2. **¿Tiene datos para retargeting?** (>~1,000 visitas/mes, o lista de clientes/leads, o audiencia con
   interacción) → sí / no.
3. **Presupuesto mensual** (y moneda; si no dice, asume USD).
4. **¿Hay recompra?** (¿el cliente puede volver a comprar/tratarse?) → si NO, se omite la campaña de
   Ascensión.

## El embudo (3 etapas + 1 opcional)
- **Presentación (frío):** llegar a gente nueva. Varios conjuntos (abierto, intereses, similares/LAL).
- **Evaluación (retargeting caliente):** quienes ya interactuaron (video, web, IG/FB, formulario) en los
  últimos ~30 días. 1 conjunto.
- **Ascensión (clientes):** compradores/registrados + lista de clientes, para recompra/upsell. 1 conjunto
  (solo si hay recompra).
- **Conversión (opcional):** solo e-commerce con catálogo (carrito abandonado / DPA). Si no aplica, se
  omite.

## Distribución de presupuesto por tramo (referencia)
- **Presupuesto bajo (≈ ≤ US$600/mes):** ve a la fija con **2 campañas** → Presentación **80%** +
  Evaluación **20%**. Nada de Ascensión/Conversión (con poco presupuesto, dispersar mata el aprendizaje).
- **Con datos de retargeting (> ~US$600):** **3 campañas** → Presentación **60%** · Evaluación **20%** ·
  Ascensión **20%**. (Si hay catálogo e-commerce y presupuesto holgado, agrega Conversión y redistribuye
  ~50/20/20/10.)
- **Sin datos de retargeting:** **2 campañas** → Presentación **80%** + una campaña de **Retargeting 20%**
  que agrupa todos los públicos calientes en 1 conjunto.
- **Sin recompra:** quita Ascensión y reparte su % entre Presentación y Evaluación.
- Presupuesto diario por conjunto ≈ (Total × % de la campaña) ÷ nº de conjuntos ÷ 30. Redondea a 2
  decimales.

## Públicos y exclusiones
**Presentación (frío) — 3 conjuntos típicos:**
| Conjunto | Público | Exclusiones |
|---|---|---|
| Abierto | país/zona, H+M, rango de edad amplio | interacción FB/IG 90d, video +10s 90d, web 90d, convertidos 180d, lista de clientes |
| Intereses | + intereses relevantes | (las mismas) |
| Similares (LAL) | lookalike 1% de convertidos 180d y/o de la lista de clientes | (las mismas) |

**Evaluación (retargeting 30d) — 1 conjunto:** FB 30d + IG 30d + video medio 30d + web 30d (+ carrito/
inicio de pago si e-commerce). Exclusiones: convertidos 30d, lista de clientes. **No uses lookalike aquí.**

**Ascensión — 1 conjunto:** compradores/registrados 180d + lista de clientes (opcional: excluir a quienes
compraron muy reciente, según el ciclo de recompra).

**Regla de exclusión (clave):** excluye siempre a los **ya convertidos** (registrados/compradores) del
frío y de la evaluación — así no pagas por re-mostrarles el anuncio ni inflas la frecuencia. Ver la nota
de frecuencia en `analizar-campanas-meta`.

## Parámetros por objetivo
- **Clientes Potenciales — sitio web/landing:** objetivo Clientes Potenciales; evento de conversión = tu
  evento de registro en la web (estándar `CompleteRegistration`, o el personalizado si el píxel tiene
  restricción); optimiza a `OFFSITE_CONVERSIONS`.
- **Clientes Potenciales — formulario instantáneo:** objetivo Clientes Potenciales; resultado = formulario
  instantáneo de Meta (no necesita píxel para el lead).
- **Interacción / WhatsApp:** objetivo Interacción/Mensajes con la app WhatsApp; sin evento de conversión
  de píxel; el resultado es la conversación iniciada. (Requiere buena habilidad de cierre de quien
  atiende el chat — ver la nota de destino en `analizar-campanas-meta`.)
- **Ventas (e-commerce):** objetivo Ventas; evento Compra; catálogo para la campaña de Conversión.
- **Reconocimiento / local:** objetivo Reconocimiento en TODAS las campañas (no Ventas/Tráfico); público
  por radio; sin evento de conversión.

## CAUCIÓN de geo (aprendizaje propio — NO copiar "Advantage+ en todo")
Para campañas **locales/geográficas** (radio, ciudad, sector), **NO dejes Advantage+ Audience activo a
ciegas.** Meta, con Advantage+ Audience ON, trata la ubicación como "señal" y **re-inyecta al publicar** un
tipo de lugar descontinuado → error **#1870194** justo al activar. Regla:
- `targeting_automation.advantage_audience: 0`,
- ubicación por **`custom_locations`** con lat/long + radio (del pin del mapa; permite bajar el radio a ~4
  km, más fino que el mínimo de la GUI),
- `location_types: ["home","recent"]` ("viven o estuvieron recientemente").
Para campañas amplias (país) no aplica esta restricción.

## Creativos por campaña
- **Presentación (frío):** 3-6 creativos. Tipos: demostración, aspiracional, beneficios, educativo,
  split/comparación, humano, "prensa". Niveles de conciencia: inconsciente / problema / solución.
- **Evaluación / Ascensión / Retargeting:** 3-5 creativos. Tipos: testimonio, "prensa", producto,
  beneficios, promociones. Niveles: producto / decisión.
Para escribir el copy y los súper prompts, usa la skill `creativos-copy-dental`.

## Marco
Esta estructura es **referencia de partida**, no verdad absoluta. La MISMA estrategia rinde distinto entre
clínicas/ciudades — cada cliente construye su propio historial. Recoge data con el MCP (skill
`analizar-campanas-meta`) y ajusta presupuestos, públicos y umbrales con el tiempo. Todo se crea en PAUSA.
