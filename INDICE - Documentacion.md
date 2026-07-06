# ÍNDICE — Qué trae este paquete

> Paquete standalone del **MCP de Meta** para Claude Code. Trae **documentación/playbooks** (lo que el
> agente lee a detalle por defecto) **y skills** (capacidades que se auto-activan según el pedido). Ambos
> se complementan.
>
> **Placeholders:** los ejemplos usan marcadores genéricos — reemplázalos por tus valores:
> `<AD_ACCOUNT_ID>`, `<PAGE_ID>`, `<PIXEL_ID>`, `<DATASET_ID>`, `<CAMPAIGN_ID>`, `wa.me/<NUMERO_WHATSAPP>`,
> `<CLIENTE>`.

## Empezar
- **`README.md`** — arranque paso a paso (terminal en esta carpeta, comando, autorizar `meta-ads`).
- **`CLAUDE.md`** — se carga solo: qué hay + la receta de variantes de copy + la regla PAUSA.

## Documentación / playbooks (léelos al operar el MCP)
- **`PLAYBOOK-Crear-Campanas-Meta.md`** — crear/montar/subir campañas y anuncios: variantes por
  `asset_feed_spec` (`DEGREES_OF_FREEDOM`), opt-out multi-anunciante, píxel `OFFSITE_CONVERSIONS`/UTM, geo
  por pin (`advantage_audience:0`), WhatsApp, imagen por URL pública. Todo en PAUSA. Con catálogo de errores.
- **`PLAYBOOK-Analizar-Campanas-Meta.md`** — analizar campañas por MCP + informe HTML: métricas
  (`ads_get_ad_entities`), benchmarks por destino y objetivo (landing/WhatsApp/Ventas-ROAS), salud del
  píxel, interpretación (hambreado vs no convirtió, fatiga, concentración de presupuesto).
- **`PLAYBOOK - CLI de Meta.md`** — el CLI de Meta (vía avanzada; sube imágenes nativo; muro del modo
  Development).

## Skills (se auto-activan según tu pedido, en `.claude/skills/`)
- **`monitor-pixel`** → *"revisa la salud del píxel"* (volumen, EMQ, alerta si un evento cae >20% con piso 50).
- **`metodologia-3qs`** → *"analiza mis campañas con las 3 Q's / por qué no convierte"* (benchmarks semáforo).
- **`analizar-anuncio`** → *"analiza este ad / deconstrúyelo"* (le pasas la imagen; Schwartz/LF8/NESB).
- **`estrategia-campanas-meta`** → *"arma mi estrategia / cómo reparto el presupuesto"* (embudo, tramos, exclusiones).
- **`creativos-copy-dental`** → *"escríbeme copy / súper prompts"* (dental; personaliza por cliente).
- **`sugeridor-productos-catalogo`** → *"qué productos de mi catálogo testear"* (e-commerce/DPA, solo lectura).

## Referencia oficial de Meta (en `referencia-oficial-meta/`)
Las **13 documentaciones oficiales de Meta** descargadas en markdown limpio, para **consultar** cuando necesites el detalle exacto (un parámetro, un permiso, un límite, una regla de las Condiciones). **Para operar mandan los playbooks y skills de arriba**; esto es la fuente autoritativa a mano. Están descargadas a propósito: una IA a veces **no puede abrir** las páginas de Meta (verificación anti-bot, login), así que aquí las tienes en local. Incluye:
- **CLI de anuncios**: overview, empezar, configuración, referencia de comandos, tutoriales, contenido de los anuncios, conjuntos de datos, insights.
- **Conectar agentes de IA con MCP**, **Condiciones de la plataforma**, **Parámetros de eventos del servidor (CAPI)**, **Permissions Reference**, **Uso de la API**.

## Principio rector — el criterio humano por encima de la IA
Detrás de cada compra hay una persona. Los fundamentos de por qué una campaña vende son **atemporales**;
la IA y Meta son herramientas para **aplicarlos**, no para reemplazar tu criterio. Este paquete (MCP,
playbooks, skills) es una palanca que amplifica tu lectura humana del comportamiento — eso es lo que la
optimización automática no ve. Prueba lo nuevo, pero no persigas el "hack" de moda sin entender cómo
funciona una campaña: los resultados consistentes salen de los fundamentos **+** la tecnología.

## Regla de oro
Todo se crea **EN PAUSA**. Nunca actives sin la confirmación explícita del usuario.
