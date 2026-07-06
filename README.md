# Meta Ads MCP — Facebook, Instagram, WhatsApp · Guide, AI Playbooks, Skills & Official Docs

**Conecta tu IA (Claude Code, Gemini CLI, Antigravity, etc.) al servidor MCP OFICIAL de Meta y crea/analiza campañas de Meta Ads desde la terminal, en lenguaje natural — con la documentación de uso que la conexión sola NO te da.**

> El servidor MCP es el **oficial de Meta** (`https://mcp.facebook.com/ads`). Este repositorio **no** es el MCP: es todo el **conocimiento de uso** alrededor de él — qué hacer, qué no, y cómo sacarle provecho — destilado de pruebas reales (ensayo y error) para que arranques con algo que **funciona**, no con una URL a secas.

---

## Por qué esto (y no solo la URL del MCP)

Conectar el MCP oficial de Meta es fácil: es una URL. **Pero la conexión sola no te dice nada**: no te explica qué herramientas tiene, qué puede y qué NO puede hacer, ni cómo montar una campaña real sin estrellarte contra los mismos errores una y otra vez.

Eso es lo que trae este kit:

- **Playbooks de uso** con los "gotchas" ya resueltos (el orden correcto para crear un conjunto, cómo subir varias variantes de copy en un anuncio, la segmentación por pin sin que Meta la rechace, el catálogo de errores con su causa y su fix…).
- **Skills** (capacidades que la IA activa sola según lo que pidas): analizar campañas con benchmarks, armar estrategia, escribir copy, revisar el píxel, etc. — adaptadas tanto a **e-commerce** como a **negocios locales de servicios** (clínica, ferretería, zapatería… campañas sin tienda online).
- **Las documentaciones OFICIALES de Meta** descargadas en markdown limpio, para consultar el detalle exacto sin depender de que la IA pueda abrir las páginas de Meta.

Todo con **valores de ejemplo (placeholders)** — reemplázalos por los tuyos. **Cero datos reales de clientes.**

## Qué NO necesitas

- **No alojas ninguna aplicación.** El servidor MCP es el de Meta.
- **No necesitas un VPS** ni infraestructura propia para el MCP.
- **No hay nada propietario:** la conexión es pública y oficial de Meta. Puedes leer su documentación directamente ([Ads AI Connectors / MCP](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/)).

Lo único que este repo agrega es **la capa de conocimiento** encima de esa conexión.

## Con qué herramientas se usa

Cualquier agente de IA que pueda **leer los archivos de esta carpeta** y hablar con un servidor MCP:

- **Claude Code** — app de escritorio **o** la CLI (terminal).
- **Antigravity** — IDE o CLI.
- **Gemini CLI.**
- …y similares.

> En la práctica, todo esto se construyó y se probó con **Claude Code (CLI)** usando el modelo **Opus 4.8**. Con esa combinación es con la que hay más horas de vuelo.

---

## Primeros pasos

1. **Descarga o clona esta carpeta** y ábrela en tu terminal.
2. Arranca tu agente con un timeout amplio (el MCP de Meta tarda en cargar sus ~80 herramientas la primera vez). Con Claude Code:
   ```bash
   MCP_TIMEOUT=120000 MCP_TOOL_TIMEOUT=120000 claude
   ```
3. Escribe `/mcp`, elige **`meta-ads`** y **autoriza con tu cuenta de Meta** (una sola vez, por navegador). Es OAuth: no expones ningún token.
4. Pide en **lenguaje natural**: *"lista mis campañas"*, *"analiza la campaña X y hazme un informe"*, *"crea una campaña de clientes potenciales a WhatsApp, en pausa"*.

El `.mcp.json` ya viene con el servidor configurado. El `CLAUDE.md` se carga solo y le da a tu agente el mapa de todo esto.

> Si al conectar ves `-32001 Request timed out`: **no es error de credenciales**, es el timeout de la primera carga. Reinicia con el comando del paso 2.

---

## Qué puede hacer el MCP

Probado de punta a punta (todo se crea **EN PAUSA**, nada sale al aire sin tu confirmación):

- **Crear campañas, conjuntos de anuncios y anuncios.**
- **Subir creativos** con **imágenes** y **variedad de textos** (varias opciones de texto por anuncio).
- Informes e insights, catálogos, diagnóstico de señales del píxel, tests A/B.

> Probado con **imágenes + texto + variantes de texto**. **Video** aún no está probado.

### El detalle de subir imágenes (léelo)

A primeras, el MCP te dirá que **no puede subir la imagen del anuncio** — y es cierto que no expone la subida directa. **Pero SÍ se puede:** le pasas a Meta una **URL pública** de tu imagen (`image_url`) y Meta la descarga. Eso está documentado paso a paso en el playbook de creación.

¿De dónde sale esa URL pública? De donde tú quieras: tu propio hosting, un bucket, o **una herramienta propia** que exponga tus creativos. La IA (Claude Code) toma esa URL pública y se la entrega al MCP; el MCP la usa para montar el anuncio con imagen. *(El handoff de la URL lo hace tu agente, no el MCP.)*

---

## El caso de uso que lo justifica: campañas grandes en minutos

Imagina una campaña con **20 anuncios**, cada uno con:

- una **imagen** distinta,
- un **ángulo de venta** distinto (precio, autoridad, dolor, producto, urgencia, prueba social…),
- un **nivel de conciencia** distinto del público (desde el que apenas nota el problema hasta el que ya está por decidir).

¿Por qué 20 y no 3? Para **atacar muchos frentes a la vez** y dejar que Meta encuentre qué ángulo + nivel conecta con cada segmento. (Los playbooks y skills explican estos ángulos y niveles a detalle.)

Ahora súmale el **copy**: por cada anuncio, **5 textos principales + 5 títulos + 5 descripciones** = **15 copys por creativo**. Multiplícalo por 20 anuncios: **300 piezas de texto** para copiar, pegar y no equivocarte.

Eso, a mano, son **2–3 horas** de trabajo tedioso y mentalmente agotador. Con este flujo (IA + MCP), lo mismo se monta en **5–10 minutos**.

> Como referencia personal del autor: existe una herramienta aparte de generación de creativos que en ~5 minutos produce 20 creativos de alto impacto con su copy y una URL pública por pieza; Claude Code toma esas URLs y arma la campaña por el MCP. **No necesitas esa herramienta** — cualquier forma de exponer una URL pública (o tu propia app) sirve. Es solo un ejemplo de hasta dónde se puede automatizar.

---

## Seguridad (léelo antes de correr nada)

- **Todo se crea EN PAUSA.** Nada se activa (empieza a gastar) sin tu confirmación explícita.
- Esto **no es un bot de automatización masiva** ni un evasor de límites. Es tu criterio + una IA que ejecuta el trabajo pesado. Úsalo con cuidado, a volumen razonable.
- Respeta las [Políticas de publicidad de Meta](https://transparency.meta.com/es-la/policies/ad-standards/) y sus [límites de la API](https://developers.facebook.com/documentation/ads-commerce/marketing-api/overview/rate-limiting).

## El principio de fondo

La IA y el MCP son una **palanca**, no un reemplazo de tu criterio. Los fundamentos de por qué una campaña vende —cómo decide de comprar un ser humano— no cambian con la tecnología. Este kit te da poder de fuego; **la lectura humana del comportamiento la pones tú**. (Está desarrollado en el `CLAUDE.md`.)

---

## Qué hay en esta carpeta

| Archivo | Para qué |
|---|---|
| `CLAUDE.md` | Se carga solo en tu agente: el mapa + las reglas clave (empezar por aquí). |
| `INDICE - Documentacion.md` | Índice de todo el paquete. |
| `PLAYBOOK-Crear-Campanas-Meta.md` | Crear/montar/subir campañas y anuncios (variantes de copy, imagen por URL, geo por pin, WhatsApp, catálogo de errores). |
| `PLAYBOOK-Analizar-Campanas-Meta.md` | Analizar campañas + generar un informe HTML visual, con benchmarks por objetivo/destino y las "3 Q's". |
| `PLAYBOOK - CLI de Meta.md` | La vía avanzada por CLI (sube imágenes de forma nativa). |
| `.claude/skills/` | 6 skills que se auto-activan: `metodologia-3qs`, `monitor-pixel`, `analizar-anuncio`, `estrategia-campanas-meta`, `creativos-copy-dental`, `sugeridor-productos-catalogo`. |
| `referencia-oficial-meta/` | Las 13 documentaciones **oficiales de Meta** en markdown, para consulta offline. |
| `.mcp.json` | El servidor MCP oficial de Meta, ya configurado. |

---

## Aviso

Proyecto **no oficial** y **no afiliado a Meta**. "Meta", "Facebook", "Instagram" y "WhatsApp" son marcas de sus respectivos dueños. Las documentaciones oficiales incluidas son de Meta y se distribuyen aquí para consulta; la fuente autoritativa siempre es la documentación en vivo de Meta. Úsalo bajo tu propio criterio y responsabilidad, respetando las políticas de Meta.
