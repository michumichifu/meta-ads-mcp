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

## Dos caminos para automatizar Meta: MCP o CLI

Hoy hay **dos** formas de que un agente de IA cree y analice campañas de Meta. Este kit documenta las dos:

| | **MCP de Meta** | **CLI de Meta** (y la Graph API) |
|---|---|---|
| Costo | **Gratis** | Gratis, pero con trabajo previo |
| Qué necesitas | Nada: una URL + tu login (OAuth) | Una **app propia en Facebook Developer**, aprobada en el **App Review**, y un token de usuario del sistema. Para publicar la app, el **portafolio dueño de la app tiene que estar verificado** |
| Herramientas | ~100 (crear, informes, píxel, públicos, catálogo, pruebas A/B, benchmarks…) | Toda la API de Marketing |
| Límites | Algunos: por esta vía un creativo publicado no se edita, no lee `promoted_object` ni `tracking_specs`, no manda eventos a la API de conversiones | Casi ninguno; respetar los límites de llamadas |
| Instalar | Nada | `pip install meta-ads` o `brew install facebook/fb/meta-ads` (macOS/Linux) |

- **El MCP** es el camino para empezar: es lo que cubre la mayor parte de este kit. Desde noviembre de 2026 **también sube imágenes y videos** (antes había que pasar una URL pública).
- **El CLI es un poco mejor** para el trabajo diario y para lo que el MCP no cubre, pero exige la app de Facebook Developer. Cómo la creamos, qué permisos pedimos, cómo escribimos los textos, cómo grabamos los videos solo con el agente de IA (sin construir ninguna interfaz), cuánto tardó Meta y cómo llegamos al acceso completo: **`GUIA - Crear la app de Meta y pasar el App Review (acceso completo al CLI).md`**. El uso del CLI, en **`PLAYBOOK - CLI de Meta.md`**.

Regla simple: **empieza por el MCP**; pasa al CLI cuando choques con uno de sus límites o cuando operes muchas cuentas a diario.

## Con qué herramientas se usa

Cualquier agente de IA que pueda **leer los archivos de esta carpeta** y hablar con un servidor MCP:

- **Claude Code** — app de escritorio **o** la CLI (terminal).
- **Antigravity** — IDE o CLI.
- **Gemini CLI.**
- …y similares.

**Sistemas operativos:** funciona en **Windows, macOS y Linux**. Lo puedes usar por la **terminal** (Claude Code CLI) o con la **app de escritorio de Claude**, que integra Claude Code y funciona muy bien.

> En la práctica, todo esto se construyó y se probó con **Claude Code (CLI)** usando modelos **Opus** (el 4.8 al principio; esta actualización de noviembre de 2026, con Opus 5.5). Con esa combinación es con la que hay más horas de vuelo.

---

## Primeros pasos

1. **Descarga o clona esta carpeta** y ábrela con tu agente de IA:
   - **App de escritorio de Claude:** ábrela como carpeta/proyecto. **No ejecutas ningún comando:** ya trae un `.claude/settings.json` que aplica un timeout amplio (el MCP de Meta tarda en cargar sus ~100 herramientas la primera vez).
   - **Claude Code por terminal:** entra a la carpeta y ejecuta `claude`. El timeout amplio también se aplica solo (está en `.claude/settings.json`).
   - **Gemini CLI, Antigravity u otra herramienta:** ábrela con la tuya; si el MCP tarda en la primera carga, dale un momento (y, si hace falta, sube el timeout en la config de esa herramienta).
2. Escribe `/mcp`, elige **`meta-ads`** y **autoriza con tu cuenta de Meta** (una sola vez, por navegador). Es OAuth: no expones ningún token.
3. **Si tu negocio atiende en una ciudad o en un radio (una clínica, un restaurante, un taller, cualquier servicio local): dale a tu agente la ubicación con un puntero, UNA sola vez.** Es el único paso manual, y es el que decide dónde sale tu publicidad:
   1. Entra al **Administrador de anuncios** como si fueras a crear una campaña y avanza hasta el **conjunto de anuncios**, sección **Ubicaciones**.
   2. Pulsa **"Colocar pin"** (el icono del alfiler, junto al mapa) y haz clic en el mapa **exactamente donde está tu negocio** (o el centro de la zona que quieres cubrir).
   3. El pin aparece en la lista como un par de números, por ejemplo `19.4545, -70.6876`. **Esas son las coordenadas**: el primero es la latitud y el segundo la longitud. Cópialas.
   4. Sal sin publicar nada y pégaselas a tu agente: *"mi negocio está en 19.4545, -70.6876, quiero un radio de 5 km"*. El agente las guarda en `MI-NEGOCIO.md` y las usa en todas las campañas, sin volver a preguntarte.

   **Por qué con puntero y no con el nombre de la ciudad:** el MCP de Meta no tiene buscador de lugares, y un nombre puede caer en otra ciudad que se llama igual (hay varias "Santiago", varias "San José"...). Con el puntero, el anuncio sale exactamente donde tú dijiste, y además puedes bajar el radio hasta **4 km**, cuando por ciudad el Administrador no baja de unos 16. *Si no encuentras el botón, vale también Google Maps: clic derecho sobre el punto y la primera línea del menú son las coordenadas; un clic y quedan copiadas.*

   **Si vendes a todo un país o tienes una tienda online, sáltate este paso:** basta con decirle a tu agente el país.
4. Pide en **lenguaje natural**: *"lista mis campañas"*, *"analiza la campaña X y hazme un informe"*, *"crea una campaña de clientes potenciales a WhatsApp, en pausa"*.

El `.mcp.json` ya trae el servidor configurado y el `.claude/settings.json` el **timeout amplio por defecto**. El `CLAUDE.md` se carga solo y le da a tu agente el mapa de todo esto.

> Si aun así ves `-32001 Request timed out` al conectar: **no es error de credenciales**, es la primera carga (el MCP lista sus ~100 herramientas). Vuelve a abrir la carpeta o reinicia tu agente y dale un momento.

> **Disponibilidad (importante):** el MCP de Meta todavía **NO** está habilitado en todas las cuentas publicitarias. Meta lo va activando de forma **gradual y automática** (sin avisar). Si una cuenta no aparece o Meta rechaza el acceso, no te desesperes: prueba con **otra cuenta** (a veces una del mismo negocio sí la tiene y otra no) y **revisa cada pocos días o cada semana** — se activa solo.

---

## Qué puede hacer el MCP

Probado de punta a punta (todo se crea **EN PAUSA**, nada sale al aire sin tu confirmación):

- **Crear campañas, conjuntos de anuncios y anuncios.**
- **Subir creativos** con **imágenes** y **variedad de textos** (varias opciones de texto por anuncio).
- Informes e insights, benchmarks frente a anunciantes parecidos, detección de anomalías, historial de cambios de la cuenta.
- Píxel y conjuntos de datos (calidad de coincidencia, volumen, reglas de evento), públicos personalizados y similares, catálogos, pruebas A/B y de conversion lift, la Biblioteca de anuncios.

El inventario completo de las ~100 herramientas, agrupadas y con lo que cambió desde julio: **`REFERENCIA - Herramientas del MCP de Meta.md`**.

> Probado con **imágenes + texto + variantes de texto**. **Video** aún no está probado.

### El detalle de subir imágenes (léelo)

**Esto cambió en noviembre de 2026.** Hasta entonces el MCP no subía la imagen del anuncio y había que pasarle a Meta una **URL pública** (`image_url`). Hoy existe **`ads_creative_upload_media`**, que sube **imágenes y videos** al banco de tu cuenta por URL pública o eligiendo el archivo de tu equipo (en las apps que muestran "MCP Apps"), y devuelve el `image_hash` para el anuncio. Meta la está activando poco a poco: si en tu cuenta responde "gradually rolled out", la URL pública sigue funcionando. Los dos caminos están en el playbook de creación.

¿De dónde sale una URL pública? De donde tú quieras: tu propio hosting, un bucket, o **una herramienta propia** que exponga tus creativos. Enlaces de Drive, Dropbox o Canva que piden iniciar sesión **no** sirven.

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
- **No compartas datos sensibles** (de clientes o pacientes) pegándolos en herramientas o guías de terceros. Trabaja solo con lo necesario.
- **Límites de Meta, en simple:** cada llamada suma "puntos" (leer ≈ 1 punto; crear o editar ≈ 3 puntos). Hay un tope por cuenta; si lo superas de golpe, Meta te frena unos minutos. No dispares muchas acciones en ráfaga: espaciarlas evita el bloqueo. Detalle oficial en `referencia-oficial-meta/meta-uso-de-la-api.md`.

## El principio de fondo

La IA y el MCP son una **palanca**, no un reemplazo de tu criterio. Los fundamentos de por qué una campaña vende —cómo decide de comprar un ser humano— no cambian con la tecnología. Este kit te da poder de fuego; **la lectura humana del comportamiento la pones tú**. (Está desarrollado en el `CLAUDE.md`.)

---

## Qué hay en esta carpeta

| Archivo | Para qué |
|---|---|
| `CLAUDE.md` | Se carga solo en tu agente: el mapa + las reglas clave (empezar por aquí). |
| `INDICE - Documentacion.md` | Índice de todo el paquete. |
| `PLAYBOOK-Crear-Campanas-Meta.md` | Crear/montar/subir campañas y anuncios (variantes de copy, subir la imagen, geo por pin, WhatsApp, catálogo de errores). Empieza por su § 0: lo que cambió desde julio. |
| `PLAYBOOK-Analizar-Campanas-Meta.md` | Analizar, leer y escalar campañas en marcha (métricas que engañan, aprendizaje, presupuesto semanal, conversiones de WhatsApp por la API de conversiones, Biblioteca de anuncios) + informe HTML. |
| `PLAYBOOK - CLI de Meta.md` | El camino del CLI y la Graph API: instalar, configurar, errores y límites. |
| `GUIA - Crear la app de Meta y pasar el App Review (acceso completo al CLI).md` | Paso a paso de la app de Facebook Developer: permisos, textos, videos grabados con el agente, envío, tiempos y el Access Tier. |
| `REFERENCIA - Herramientas del MCP de Meta.md` | Las ~100 herramientas del MCP, agrupadas, y lo que todavía no hace. |
| `REFERENCIA - Condiciones de Meta del 20 nov 2026.md` | Qué cambia en las condiciones de anuncios de Meta y qué te toca revisar. |
| `.claude/skills/` | 6 skills que se auto-activan: `metodologia-3qs`, `monitor-pixel`, `analizar-anuncio`, `estrategia-campanas-meta`, `creativos-copy-dental`, `sugeridor-productos-catalogo`. |
| `referencia-oficial-meta/` | Las documentaciones **oficiales de Meta** en markdown, para consulta offline: las 13 traducidas de julio y, en `en-2026-11/`, las 18 páginas actuales de Ads AI Connectors (CLI y servidor MCP) tal cual, en inglés. |
| `.mcp.json` | El servidor MCP oficial de Meta, ya configurado. |

---

## Aviso

Proyecto **no oficial** y **no afiliado a Meta**. "Meta", "Facebook", "Instagram" y "WhatsApp" son marcas de sus respectivos dueños. Las documentaciones oficiales incluidas son de Meta y se distribuyen aquí para consulta; la fuente autoritativa siempre es la documentación en vivo de Meta. Úsalo bajo tu propio criterio y responsabilidad, respetando las políticas de Meta.
