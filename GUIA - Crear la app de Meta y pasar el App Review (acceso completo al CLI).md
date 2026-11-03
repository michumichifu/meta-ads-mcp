# GUIA - Crear la app de Meta y pasar el App Review (acceso completo al CLI)

> Todo lo que va entre `<...>` es un **valor de ejemplo**: cámbialo por el tuyo. Aquí no hay ningún id,
> dominio ni token real.
>
> Es **lo que hicimos** en 2026 para pasar dos App Review de Meta (anuncios y redes sociales) **sin
> ninguna aplicación con interfaz**, todo desde la terminal con un agente de IA. Lo que es criterio
> nuestro, y no regla de Meta, va marcado **(criterio)**.

---

## 0. Dos caminos para trabajar Meta con un agente

| | **MCP de Meta** | **CLI de Meta** (y la Graph API) |
|---|---|---|
| Costo | Gratis | Gratis, con trabajo previo |
| Qué hace falta | Una URL y tu login de Meta (OAuth) | Una **app propia** aprobada en el **App Review** y un **token de usuario del sistema** |
| Herramientas | Unas 100, listas | Toda la Marketing API |
| Límites | Algunos (ver `README.md`) | Casi ninguno, salvo los de llamadas |

El **CLI es un poco mejor**, pero exige crear una app en Facebook Developer, y para **publicarla** el
**portafolio (Business) dueño de la app tiene que estar verificado**.

🔴 La verificación es **del portafolio dueño de la app** (el tuyo), **no** de los portafolios de los
clientes. A un cliente no se le pide verificar nada: solo que **comparta sus activos** (cuenta
publicitaria, página, Instagram) con tu portafolio como socio y que dé a tu usuario del sistema el rol
que haga falta.

**Requisitos previos:**

- [ ] **Portafolio verificado** (Business Verification del dueño de la app). Sin esto el envío no avanza.
- [ ] Páginas públicas de **privacidad** y **eliminación de datos** (`https://<TU_DOMINIO>/privacidad/`,
  `https://<TU_DOMINIO>/eliminacion-de-datos/`).
- [ ] **Icono de 1024 x 1024** sin marcas de Meta, y un **correo de contacto** que leas.
- [ ] Un agente que ejecute comandos: **Claude Code (CLI o la app de escritorio de Claude)**,
  **Antigravity CLI** o similar. Con eso basta.

---

## 1. Qué desbloquea la aprobación

Con tu app **en modo desarrollo** ya puedes leer todo, **crear campañas y conjuntos**, **subir imágenes**
(`/adimages`) y cambiar nombre, estado, presupuesto y segmentación. Lo único bloqueado es **crear o
editar el creativo** (`/adcreatives`: imagen + copy + CTA):

```
code 100 / subcode 1885183: "...se creó con una app que se encuentra en modo de desarrollo.
Debe estar en modo público para crear este anuncio."
```

El App Review y la **publicación** quitan ese muro: **el anuncio entero por API**. El segundo escalón es
el **Access Tier** (sección 9), y 🔴 para él **no cuentan las llamadas del MCP**, solo las de **tu** token.

---

## 2. Crear la app en developers.facebook.com

1. **developers.facebook.com/apps -> Crear aplicación.** Nombre de 30 caracteres como máximo, 🔴 sin
   "Facebook", "Instagram", "IG", "Insta" ni "FB".
2. **Casos de uso** (se pueden marcar varios):
   - Anuncios: **"Crea y administra tus anuncios con la API de marketing"**. Los demás que quedan con el
     círculo vacío **no bloquean**: Meta solo revisa lo que envías.
   - Redes: **"Administra todo en tu página"**, **"Interactúa con los clientes en Messenger from Meta"** y
     **"Administrar mensajes y contenido en Instagram"**, en la misma app y el mismo envío.
3. **Empresa:** tu portafolio verificado. Las verificaciones (negocio y, si la tienes, proveedor de
   tecnología) **son del negocio, no de la app**: en una app nueva salen en verde y no se repiten.
4. **Configuración -> Información básica:** privacidad, eliminación de datos, categoría (pusimos
   "Utilidad y productividad"), icono (🔴 nos rechazaron uno **con fondo blanco**) y dominio
   `<TU_DOMINIO>`. "Condiciones del servicio" viene con `https://www.facebook.com/`; no teníamos página
   de términos y así quedó: **(criterio)** mejor tener una propia, y un correo de tu dominio.
5. **Plataforma:** abajo del todo, **"+ Añadir plataforma" -> Sitio web** con `https://<TU_DOMINIO>/`.
   Meta **no ofrece "herramienta de línea de comandos"**: se elige Website y **se explica al revisor**.
   Sin plataforma, el App Review **no deja escribir las instrucciones**.

🔴 **Dos trampas del panel:**
- **El dominio se añade pulsando la SUGERENCIA, nunca con Enter.** Enter guardó y **vació** privacidad,
  eliminación de datos, condiciones y categoría; en pantalla seguían, al recargar estaban en blanco.
- **Algo solo está guardado si sale al RECARGAR.** "Guardar cambios" y recargar para verlo.

**(criterio) Una app por función.** Para redes hicimos una app aparte: un bot que escribe a la gente
arriesga más una sanción, y así no tumba los anuncios ni vuelve a revisar lo ya aprobado (sección 7).
El costo: la base se pide otra vez, con su video.

---

## 3. El usuario del sistema y el token

1. **Se crea en el PORTAFOLIO**, no en la app: Configuración del negocio -> Usuarios -> Usuarios del
   sistema -> Añadir (Admin o Employee).
2. **Asignar activos:** la app (control total) y las cuentas, páginas, píxeles y catálogos que use.
3. **Generar identificador:** la app, caducidad **"Nunca"** y los permisos. Meta añade a la app los que
   le falten.
4. Al `.env` (permisos `600`, fuera de git). **Nunca en la línea de comandos ni en el chat.**

En la cuenta de **un cliente**, el cliente asigna a tu usuario del sistema el rol de **Anunciante** o
superior en **su** Configuración del negocio; sin eso el token lee, pero al crear da
`code 200 / subcode 2490585`.

```bash
set -a; source .env; set +a          # carga el token sin imprimir nada
G="https://graph.facebook.com/v26.0"; H="Authorization: Bearer $ACCESS_TOKEN"
curl -s -H "$H" "$G/me"; curl -s -H "$H" "$G/me/permissions"
curl -sG "$G/debug_token" --data-urlencode "input_token=$ACCESS_TOKEN" -H "$H"
```

`debug_token` debe decir `type: SYSTEM_USER`, el `app_id` de tu app, `is_valid: true` y `expires_at: 0`.
⚠️ **"granted" en `/me/permissions` no es "aprobado"**: antes del App Review es **acceso estándar**, que
solo vale con personas con rol en la app. ⚠️ Con field expansion (`{...}`), **`curl -g`**.

**(criterio) No regeneres un token que ya usan otras integraciones.** Cuando nos faltó un permiso para
una llamada de prueba, usamos un **token de usuario temporal** del Explorador de la Graph API (con la app
seleccionada; dura unas 2 horas), en otra variable, solo para esa llamada y su video.

---

## 4. Qué permisos pedir y cuáles NO

**Regla:** primero se mira qué hace de verdad la herramienta y **solo se pide (y se graba) lo que usa**.
Nunca se pide un permiso para luego buscar qué grabar.

**App de anuncios (8 items):**

| Permiso | Para qué | Video |
|---|---|---|
| `ads_management` | Crear y editar campañas, conjuntos y anuncios | Sí |
| `ads_read` | Leer métricas para informes | Sí |
| `business_management` | Operar los activos que comparten los clientes | Sí |
| `pages_show_list` | Elegir la página que firma el anuncio | Sí |
| `pages_read_engagement` | Usar publicaciones de la página como creativo | Sí |
| `leads_retrieval` | Descargar leads de formularios instantáneos | El formulario solo pidió texto |
| `public_profile` | Base | Solo confirmar |
| `Marketing API Access Tier` | El nivel de acceso (sección 9) | No |

Meta armó la cadena `ads_management -> pages_read_engagement -> pages_show_list`. No pedimos
`pages_manage_ads` (Meta no lo metió en la solicitud), `email`, `Business Asset User Profile Access`,
`catalog_management` ni `threads_business_basic`.

**App de redes (13 items, en dos envíos):**

| Permiso o función | Para qué |
|---|---|
| `pages_show_list`, `business_management`, `pages_read_engagement`, `public_profile` | Base |
| `pages_read_user_content` | Leer comentarios de publicaciones y anuncios |
| `pages_manage_engagement` | Responder como la página y ocultar |
| `pages_manage_metadata` | Suscribir la app a los webhooks (`feed`, `messages`) |
| `pages_messaging` | Messenger y la respuesta privada a un comentario |
| Business Asset User Profile Access (función) | Nombre y foto de quien escribe |
| `instagram_basic`, `instagram_manage_comments`, `instagram_manage_messages` | Cuenta, comentarios y mensajes de Instagram |
| `instagram_manage_engagement` | Me gusta a comentarios |

No pedimos **Page Public Content Access** (es para páginas ajenas), los **`instagram_business_*`** (son
del camino "Instagram Login": **una app usa Facebook Login o Instagram Login, no los dos**), Human Agent,
ni publicar o estadísticas. ⚠️ "Configuración con el inicio de sesión con Facebook" **no pone ninguna
pantalla de login**: solo elige la familia de permisos.

**(criterio)** Graba video de todos menos `public_profile` y el Access Tier: la doc lo pide para cada uno.
Se añaden en Casos de uso -> Personalizar -> fila -> **"Acciones" -> "Añadir a la revisión"**, y se
aprueban **para la app entera**, no por caso de uso.

---

## 5. Cómo escribimos los textos

### El texto de cada permiso

En inglés, siempre con la misma estructura: quiénes somos y qué es la app; qué hace **este** permiso y
para qué; de qué depende; y los límites (solo activos que gestionamos, no guardamos datos, una persona
escribe cada respuesta, si es verdad).

```text
<COMPANY> is a <TYPE OF BUSINESS> in <COUNTRY>. Our team uses this app as an internal
command-line tool to <MAIN JOB> for our own brand and for the businesses we work for,
which added our business as a partner in their Business portfolios.
We use <PERMISSION> to <CONCRETE ACTION>, so that <PURPOSE>. <It is a dependency of
<OTHER_PERMISSION>.> We only use it on assets we manage<, and we do not store the data>.
```

### Las instrucciones para el revisor

Salen **solo después de añadir la plataforma y RECARGAR** el App Review. Cinco campos:

| Campo | Qué pusimos |
|---|---|
| `instructions-web-2` (obligatorio) | El texto de abajo |
| `accesscode-web-1` | "Not applicable" y el porqué. **(criterio)** Vacío deja dudas |
| `accesscode-web-2` | Vacío (códigos de regalo de apps de tienda) |
| `geo-web-5` | Sin restricción geográfica |
| `documents-web-1` | 🔴 **Todos los videos** (`.mp4`, `.mov`, `.zip`, hasta 2 GB), aunque ya estén en cada permiso |

```text
WHAT THIS APP IS
"<APP NAME>" (App ID <APP_ID>) is the internal Marketing API client of <COMPANY>. We use it to
<WHAT IT DOES> on behalf of our clients, who share their assets with our Business as partners.

HOW IT IS OPERATED - THERE IS NO USER INTERFACE TO TEST
It is a server-to-server client: our team runs the API calls from the command line with a
Business System User token of this app. There is no Facebook Login and no end-user sign-up. We
registered our company website as the platform only because "command line tool" is not a
platform type Meta offers.

HOW TO REVIEW THIS APP
The attached screen recordings are the way to review it. Each one shows the starting state in
Meta's interface, the app making the API call from our terminal, and the result back in Meta.

WHAT EACH RECORDING DEMONSTRATES
- <permission>: <one line>
All write operations are created in PAUSED status and reviewed by a person before activation.
```

En Configuración -> Información básica -> "Proporciona instrucciones de prueba", "¿Está integrado el
inicio de sesión con Facebook?" -> **No**.

🔴 **Tres reglas que nos costaron tres redacciones:**
- **El envío describe lo que hacen los videos.** Escribimos que se operaba "desde un panel web y desde la
  terminal" y el panel no tenía esa función: una interfaz que el revisor no encuentra es motivo de rechazo.
- **Sin credenciales de prueba** si no hay pantalla donde usar los permisos.
- **La URL de la plataforma y la de las instrucciones, la misma.**

### Gestión de datos

- **Responsable:** tu empresa, con el país que **coincida con la verificación del negocio**.
- **Encargados:** tu **hosting** y, si los videos enseñan al agente leyendo comentarios o mensajes, **el
  proveedor de su modelo** (con Claude, Anthropic): si no se declara, los videos lo contradicen.
- **Países:** donde se tratan **hoy**, no "por si crecemos" (un país de la UE mete el RGPD encima).
- **Es una declaración jurada**; para Meta lo agregado o derivado sigue siendo dato de la plataforma.
- En una segunda app, "Responder automáticamente a partir de otra aplicación" copia la primera: repásalo.

---

## 6. Cómo grabamos los videos sin una app con interfaz

**La "app" es el token de tu usuario del sistema.** Al agente (lo hicimos con **Claude Code en la
terminal**; la app de escritorio de Claude o Antigravity CLI sirven igual) se le pega una instrucción en
inglés, hace la llamada a la Graph API, y en pantalla se ven **el comando y la respuesta**; luego se
enseña **el cambio en Meta**. Nadie programa ni teclea comandos. 🔴 **No se graba el MCP de Meta**: es
la app de Meta y no demuestra la tuya.

**El patrón: antes -> la app actúa -> después.**

| Permiso | Antes | La app actúa | Después |
|---|---|---|---|
| `pages_show_list` | Configuración del negocio -> Páginas | Lista las páginas | Las mismas |
| `business_management` | Configuración del negocio -> Cuentas | Lista las cuentas | Las mismas |
| `pages_read_engagement` | La página | Token de página + 3 publicaciones | Las mismas |
| `ads_read` | Administrador, últimos 30 días | Insights, mismo rango | Mismas cifras |
| `ads_management` | Administrador, Campañas | Crea una campaña EN PAUSA | Refresco: aparece |
| `pages_manage_engagement` | Cuenta personal comenta | Responde como la página | Recarga: sale |
| `pages_messaging` | Cuenta personal comenta | Privado y luego respuesta | Llega a Messenger |
| `pages_manage_metadata` | Solo la terminal | Suscribe la página | `success: true` |

En comentarios y mensajes, **una cuenta personal escribe como un cliente** y **la página responde por el
agente**. Cada video crea su propio comentario o mensaje, y la instrucción apunta a **"lo más reciente"**,
así no hay ids que copiar (⚠️ una publicación fijada no cuenta: se usa la de debajo).

**Forma (doc de Meta):** **un permiso = un video**; **10 a 40 s**, **1080p**, **sin audio**, **cursor
grande**, con el ratón; **monitor de 1440 px o menos**; **todo en inglés** (primero al agente:
`From now on, respond in English.`); **anota** cada video con su permiso; el **cómo** en el video, el
**porqué** en el formulario; y **al menos una llamada correcta por permiso en los 30 días previos**.

**Lo que hunde un video:**
- 🔴 **El token en pantalla** (el video se le envía a Meta): nada de `cat .env`, `echo`, `env` ni
  `access_token=` en una URL; las llamadas con `-H "Authorization: Bearer $ACCESS_TOKEN"`. Revisa el
  video entero antes de subirlo; si se coló, **se rota el token** y se regraba.
- 🔴 **Con la app en desarrollo, en `ads_management` se crea una CAMPAÑA, nunca un anuncio** (rebotaría
  con `1885183`).
- 🔴 **La cuenta del Administrador distinta de la del prompt**: nos pasó; se creó, pero en otra cuenta.
- ⚠️ **Sin el ID exacto y la orden de no explorar**, el agente busca entre cuentas: un video llegó a 5 min.
- ⚠️ **Sin los campos obligatorios en la instrucción** (`is_adset_budget_sharing_enabled=false`), la
  primera llamada rebotó: 44 s y un error rojo en cámara.
- ⚠️ **Sin el refresco** no demuestra nada. Sesión limpia, sin chats ni correo en pantalla.

**Las instrucciones que se pegan al agente:**

```text
Using the ACCESS_TOKEN in .env (our System User, app <APP_ID>), make exactly one Graph API
call and show the result in a small table. Use exactly this account act_<AD_ACCOUNT_ID>, do not
search for any other, never print the token and do not explore the project:
GET /act_<AD_ACCOUNT_ID>/insights?fields=impressions,reach,clicks,spend,cpc,ctr&date_preset=last_30d
```

```text
Using the ACCESS_TOKEN in .env (our System User, app <APP_ID>), in ad account act_<AD_ACCOUNT_ID>
create ONE campaign in PAUSED status with these exact fields: name 'TEST - App Review', objective
OUTCOME_TRAFFIC, special_ad_categories [], is_adset_budget_sharing_enabled false. Show the id,
name and status in a small table. Use exactly this account, do not explore the project, do not
create an ad set or an ad, do not activate anything.
```

```text
Using the <TOKEN_VARIABLE> in .env (a user access token of our app <APP_ID>), on our Instagram
account <IG_USER_ID>: (1) get its most recent post with GET /<IG_USER_ID>/media?fields=id&limit=1,
(2) read its comments with GET /{that media id}/comments?fields=id,username,text,timestamp and show
them, (3) like the most recent comment with POST /<IG_USER_ID>/likes and comment_id={that comment
id}. Show the result. Never print the token and do not explore the project.
```

⚠️ Lo de una página pide **token de página** (si no, `190` o `210`): `GET /<PAGE_ID>?fields=access_token`.

**Cómo se suben:** en **Revisión -> Revisión de la aplicación**, en cada permiso: descripción, video,
"Confirmo que..." y Guardar (sus "checks" son tareas, no aprobación); y todos juntos en `documents-web-1`. `pages_messaging` pide además página e instrucciones para que el
revisor le escriba (🔴 si lo hace, **alguien tiene que contestarle desde la página**);
`instagram_manage_messages`, el usuario de Instagram y la experiencia (pusimos **Live Agent**).

---

## 7. Cómo se envía

1. Cada permiso con texto, video y confirmación; verificación, configuración, gestión de datos e
   instrucciones, en verde.
2. 🔴 **Meta tiene que CONTAR la llamada de prueba de cada permiso.** En **Revisión -> Tests** sale
   "0 de las 1 llamadas a la API requeridas" hasta que la cuenta; Meta avisa de que *puede tardar hasta
   24 horas* y de que *los tests valen 30 días*. Hasta entonces "Enviar a revisión" sigue bloqueado, y
   hacer más llamadas no lo acelera.
3. **Enviar.** Ya no se puede editar ni cancelar (las instrucciones al revisor sí se pueden actualizar).
4. **Aprobado: publicar** (menú Publicar -> botón de abajo a la derecha; "Sin publicar" pasa a
   "Publicada"). 🔴 **Publicar va DESPUÉS de aprobar, y aprobado no es en vivo**: hasta publicar, lo
   aprobado no rige fuera de la app.

- ⚠️ Con una solicitud en revisión **no sale un bloque "Sin enviar"** para otra: va en un segundo envío.
- 🔴 **Reenviar en una app aprobada vuelve a revisar lo aprobado** ("Acceso actual para la renovación":
  *"Revisaremos tanto las solicitudes nuevas como las aprobadas previamente"*). Nos salió todo
  "Renewed"; **(criterio)** reenvía con lo nuevo sobrado.
- ⚠️ No uses "API setup with Unified login" -> "Ir a la revisión": trae permisos de más marcados.

---

## 8. Los tiempos que tuvimos

La pantalla dice *"La mayoría de los envíos se revisan en un plazo de 20 días"*.

| Envío | Enviado | Respuesta | Tardó |
|---|---|---|---|
| Anuncios, 8 items | 26 ago 2026 | 5 sep: 7 aprobados, Access Tier rechazado. Publicada el 6 | 10 días |
| Access Tier, reenviado con unas 1.100 llamadas | 24 sep 2026, 23:52 | 25 sep, antes de las 9:36: aprobado; los 7 "Renewed" | Menos de 10 h |
| Redes, 9 items (app creada el día antes) | 18 oct 2026 | 1 nov: aprobados los 9; publicada ese día | 14 días |
| Redes, 4 de Instagram | Subido el 1 nov 2026, esperando que Meta contara la llamada | Pendiente al escribir esto | |

⚠️ El aviso llegó a **las notificaciones de Facebook** (lo vimos un día tarde). Un rechazo se lee en
**Ver detalles -> Revisar comentarios**.

---

## 9. El Marketing API Access Tier

**Se concede por un contador**, no por el texto: **500 llamadas a la Marketing API en los últimos 15
días, con menos del 15 % de error en las últimas 500**, medidas en la **ventana anterior a la revisión**.
Nuestro rechazo: *"Our records do not show a sufficient number of Ads API calls in the last 15 days by
this application."* ⚠️ El globo "Requisitos" salía en verde porque mira el acumulado. **No se reescribe
nada:** se trabaja de verdad con tu token 15 días (nosotros, con el CLI a diario en vez del MCP) y se
reenvía. 🔴 **Las llamadas fallidas cuentan en el 15 %**, también las de `validate_only`.

**Cómo se lee, sin navegador**, en la cabecera de cualquier llamada a una cuenta:

```bash
curl -s -D - -o /dev/null -H "Authorization: Bearer $ACCESS_TOKEN" \
  "https://graph.facebook.com/v26.0/act_<AD_ACCOUNT_ID>?fields=name" | grep -i x-business-use-case-usage
# x-business-use-case-usage: {"<AD_ACCOUNT_ID>":[{"type":"ads_management","call_count":0,...,"ads_api_access_tier":"standard_access"}]}
```

- `development_access` = limitado; `standard_access` = **completo**. 🔴 La doc renombró los niveles
  ("Standard Access" ahora es *Limited*, "Advanced" es *Full*), pero **la cabecera no**: `standard_access`
  es el ALTO.
- ⚠️ `call_count` es el **porcentaje de cuota**, no el número de llamadas. ❌ El campo
  `marketing_api_access_tier` de la app **no existe**.

| Por hora y cuenta | Limited | Full |
|---|---|---|
| Ads Management | `300 + 40 x anuncios activos` | `100.000 + 40 x anuncios activos` |
| Ads Insights | `600 + 400 x anuncios activos` | `190.000 + 400 x anuncios activos` |
| Puntuación máxima | 60 | 9.000 |
| Usuarios del sistema | 1 + 1 admin | 10 + 1 admin |
| Business Manager API | Sin gestionar por API cuentas, permisos de usuarios ni Páginas | Completa, con Catálogo |
| Crear Páginas por API | No | No |

Lo de la Business Manager API es **capacidad**, no cuota. **No cambia** que el cliente tenga que darte
acceso: la API deja **pedirlo**, no saltárselo.

🔴 **Se mantiene con uso:** la doc oficial pide seguir con **500 llamadas cada 15 días y menos del 15 %
de error**. No dice cada cuánto lo revisa: **vigila la cabecera**. Fuente:
`https://developers.facebook.com/documentation/ads-commerce/marketing-api/get-started/authorization.md`.

---

## 10. Ya aprobado: el acceso completo al CLI

**Comprobar que el muro cayó, sin crear nada** (`validate_only` valida y descarta):

```bash
curl -s -X POST "https://graph.facebook.com/v26.0/act_<AD_ACCOUNT_ID>/adcreatives" \
  -H "Authorization: Bearer $ACCESS_TOKEN" -d "name=PRUEBA_ESTADO_APP" \
  --data-urlencode 'object_story_spec={"page_id":"<PAGE_ID>","link_data":{"image_hash":"<IMAGE_HASH>","link":"https://<TU_DOMINIO>/","message":"x","name":"x","call_to_action":{"type":"LEARN_MORE"}}}' \
  -d 'execution_options=["validate_only"]'
```

`{"success": true}` = desbloqueado; `1885183` = sigue en desarrollo; ⚠️ `1443121` es un **falso
negativo** (`page_id` inválido: Meta no llegó a mirar el modo). Sobre `/campaigns`, dice si ya puedes
escribir en la cuenta de un cliente (`2490585` = falta el rol). Publicar no prueba nada.

**El CLI oficial de Meta:**

```bash
pip install meta-ads        # Python 3.12+; o: brew tap facebook/fb && brew trust facebook/fb && brew install facebook/fb/meta-ads
export ACCESS_TOKEN=...     # mejor en un .env: ACCESS_TOKEN, AD_ACCOUNT_ID (con o sin act_), BUSINESS_ID opcional
meta auth status
meta ads adaccount current
meta ads campaign list
```

Las variables de la terminal mandan sobre el `.env`. La doc pide que el token lleve `business_management`,
`ads_management`, `pages_show_list`, `pages_read_engagement`, `pages_manage_ads` y `read_insights`, más lo
que tu caso necesite (`referencia-oficial-meta/en-2026-11/ads-cli--setup--get-started.md` y
`ads-cli--setup--configuration.md`). ⚠️ **No es el que usamos estos meses**: cuando lo probamos, `pip
install meta-ads` no resolvía. Sigue su doc vigente.

**El comunitario que usamos, `meta-ads-cli` 0.2.0** (`pip install meta-ads-cli`, comando `meta-ads`):
- Lee **`META_ACCESS_TOKEN`**, **`META_AD_ACCOUNT_ID`** y **`META_PAGE_ID`**, no los nombres del oficial.
- 🔴 **El id de cuenta SIN `act_`**: con él responde *"Object with ID 'act_act_...' does not exist...
  missing permissions"*, que parece de permisos y no lo es.
- Su API por defecto es **v21.0, que caduca el 21 ene 2027**: fija `META_API_VERSION` en una vigente
  (nuestros scripts pasaron a `v26.0`) y **valida las escrituras con `validate_only` antes de subir**.
- `upload-image` sin `--live` es un ensayo. 🔴 Su **entorno virtual guarda rutas absolutas**: al mover la
  carpeta dejó de arrancar.

**Desde v25, tres campos obligatorios** (fallan con un `Invalid parameter` mudo; el mensaje útil está en
**`error_user_title`** y **`error_user_msg`**):

| Al crear | Campo | Valor |
|---|---|---|
| Campaña | `is_adset_budget_sharing_enabled` | `false` si el presupuesto va en el conjunto |
| Conjunto | `targeting.targeting_automation.advantage_audience` | `0` o `1`, explícito |
| Conjunto | `bid_strategy` | p. ej. `LOWEST_COST_WITHOUT_CAP` |

---

## 11. Errores frecuentes

| Síntoma | Causa | Solución |
|---|---|---|
| `subcode 1885183` | App en desarrollo | App Review y publicar |
| `subcode 2490585` | Sin rol de anunciante en la cuenta | Que el dueño se lo asigne |
| No deja pegar las instrucciones | Falta la plataforma | Sitio web y **recargar** |
| Campos vacíos en Información básica | Enter en "Dominios" | Sugerencia y **recargar** |
| "Enviar a revisión" gris | Llamada de prueba sin contar | Revisión -> Tests; hasta 24 h |
| Access Tier rechazado o perdido | Menos de 500 llamadas en 15 días | Uso real y reenviar |
| `code 190` / `210` en datos de página | Hace falta token de página | `GET /<PAGE_ID>?fields=access_token` |
| `code 10` en Messenger | Acceso estándar | Aprobación y app publicada |
| `code 100 / subcode 33` | El token no lleva el permiso | Otro token (sección 3) |
| Aprobado, pero sin datos nuevos | App sin publicar | Publicar |

---

*No oficial ni afiliado a Meta. Pantallas del panel en español, 2026: Meta las cambia sin avisar.*
