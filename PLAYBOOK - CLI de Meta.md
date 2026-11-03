# PLAYBOOK — CLI de Meta (`meta-ads`) para el flujo creativos → Meta

> Todos los valores de este playbook (IDs de cuenta/página/píxel, números de teléfono, nombres como "Clínica Dental Aurora" o "Dra. Aurora Peña") son **ejemplos ficticios**. Reemplázalos por los de tu propia cuenta.

> Guía reproducible del **CLI de Meta**: qué es, qué NO es, cómo se instala, cómo se configura, cómo
> lo uso y lo interconecto con tus creativos y con la Graph API, su **alcance real** (qué hace y qué no),
> los **muros** encontrados y cómo sortearlos. Complementa a `PLAYBOOK-Crear-Campanas-Meta.md` (el del
> MCP). Crear la app y pasar el App Review: `GUIA - Crear la app de Meta y pasar el App Review (acceso
> completo al CLI).md`.

---

## 0. ACTUALIZACIÓN DE NOVIEMBRE DE 2026 (léela antes que el resto)

Este playbook se escribió en julio, con la app todavía en modo desarrollo. Lo que cambió:

- **El CLI oficial de Meta ya se instala:** `pip install meta-ads` (PyPI, versión 1.3.1 al 3 nov 2026,
  Python 3.12+) o `brew tap facebook/fb && brew trust facebook/fb && brew install facebook/fb/meta-ads`
  (macOS y Linux). Instala el comando **`meta`** (`meta --version`). La guía oficial, en
  `referencia-oficial-meta/en-2026-11/ads-cli--setup--get-started.md`. El resto de este playbook usa el
  CLI **comunitario** (`meta-ads-cli` 0.2.0), que fue el que operamos: las ideas valen igual, los
  comandos cambian.
- **El muro del modo desarrollo (§ 9) se levanta con el App Review.** Con `ads_management` aprobado y la
  app publicada, `/adcreatives` deja de dar el 1885183. Para comprobarlo, `validate_only` con un `page_id`
  **válido** (con uno inválido sale 1443121 y parece un falso desbloqueo). El paso a paso de la app, en la
  GUIA del App Review.
- **Marketing API Access Tier:** se concede solo con uso (500 llamadas a la API de Marketing en 15 días,
  con menos de un 15 % de errores) y se mantiene con uso. **Las llamadas del MCP no cuentan** (van por la
  app de Meta). Se ve en la cabecera `x-business-use-case-usage`, campo `ads_api_access_tier`
  (`development_access` = limitado, `standard_access` = completo).
- **Versión de la API:** la v21.0 caduca el 21 ene 2027. Usa v25 o posterior (`META_API_VERSION`).
- **Campos que la v25+ exige** y que, si faltan, dan un `100 Invalid parameter` sin más:
  `is_adset_budget_sharing_enabled` (campaña sin presupuesto propio), `advantage_audience` explícito y
  `bid_strategy`. El mensaje útil viene en `error_user_title` y `error_user_msg`: léelos siempre.
- **Límite de escritura:** error **613**, *"1 calls per 30 seconds"*. Una llamada `validate_only` gasta
  ese turno: validar y escribir seguidos falla. Y `validate_only` de un conjunto o un anuncio necesita ids
  reales de campaña y creativo (con id 0 salen falsos errores como "Invalid id: 0").
- **Errores que mienten sobre su causa:**

  | Error | Causa real |
  |---|---|
  | `act_act_... does not exist / missing permissions` | Un `act_` de sobra |
  | "Invalid Page ID" | Puede ser un dígito mal copiado: compruébalo con `me/accounts` |
  | 200 / 2490585 | Falta el rol de anunciante en la cuenta del cliente: el usuario del sistema lee pero no escribe |
  | 100 / 2446386 "Imagen no encontrada" | La imagen se subió a OTRA cuenta (el hash es el MD5 del archivo y sale igual en cualquier cuenta) |
  | `code 190` en `leadgen_forms` | Pide un token de PÁGINA, no un permiso nuevo |

- **Trampas del CLI comunitario 0.2.0:** pide `META_PAGE_ID` incluso para leer (`=0` vale);
  `/me/adaccounts` no lista cuentas de cliente que sí se pueden leer; su `insights` da solo totales (para
  resultados usa la Graph API con el campo `conversions`, nunca `actions`); `upload-image` sube a la
  cuenta por defecto si no le pasas la del cliente; y el entorno virtual guarda rutas absolutas: si mueves
  la carpeta, se rompe en silencio.
- **Por qué el CLI para el trabajo diario:** hace lo que el MCP no (leer `promoted_object` y
  `attribution_spec`, cambiar el creativo de un anuncio por la Graph API, mandar eventos a la API de
conversiones) y su uso mantiene el
  Access Tier.

---

## 1. QUÉ ES ESTE CLI (y qué NO es)

- Paquete instalado: **`meta-ads-cli`** (PyPI), versión probada **0.2.0**. Binario: **`meta-ads`**.
- **Importante:** su código apunta a **`github.com/attainmentlabs/meta-ads-cli`** → es una CLI
  **comunitaria (attainmentlabs)**, NO confirmadamente la "oficial de Meta". Es un **wrapper fino
  sobre la Graph API de Marketing** (endpoints `/adimages`, `/campaigns`, `/adsets`, `/adcreatives`,
  `/ads`).
- **La Ads CLI OFICIAL de Meta SÍ existe** (parte de sus "Ads AI Connectors", abril 2026). Docs:
  `developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/ads-cli-overview`
  (overview), la Get Started (instalación real) y la Command Reference en ese mismo árbol. Se documenta
  como paquete `meta-ads` (comando `meta ads …`), pero `pip install meta-ads` **NO resolvió en PyPI** en
  pruebas → confirma el comando real en la Get Started oficial antes de instalar. El `meta-ads-cli`
  comunitario de arriba funciona hoy y sube imágenes; la oficial se prefiere por soporte/funciones.
- **Oficial vs comunitario NO cambia el riesgo de baneo:** ambos usan la MISMA Graph API con tu mismo
  token. El baneo depende del USO (crear en PAUSA, bajo volumen, respetar rate limits), no del binario.
- **Orden de preferencia:** prefiere el CLI (oficial si instala; comunitario si no); usa curl a la Graph
  API cruda solo para lo que el CLI no cubra (es más frágil como primera opción). El token **SIEMPRE por
  variable de entorno**, nunca escrito en la línea de comando.
- **Su valor real frente al MCP:** **sube imágenes de forma nativa** (`POST act_1234567890123456/adimages`), lo
  que el MCP de Meta no hace. Ese es el motivo principal para usarlo.

## 2. REQUISITOS PREVIOS (el "gate" real)

1. **Token de Meta** con `ads_management` y acceso a la cuenta destino. Aquí: un **System User token**
   (`SystemUser-MiAgencia`) guardado en `.env` como `ACCESS_TOKEN`. Los System User tokens no
   expiran. (El MCP no expone token; por eso el CLI necesita uno propio.) Valídalo ANTES de usarlo:
   `GET /me` + `GET /me/permissions` confirman el tipo (System User) y los scopes
   (`ads_management`/`ads_read`/`business_management`); `debug_token` identifica la app.
2. **La app del token debe estar en modo LIVE** para poder **crear anuncios**. ⚠️ Éste es el muro más
   importante: con la app en **desarrollo** se pueden crear campaña y conjunto, pero **no anuncios**
   (ver §9). Verificar con `debug_token` qué app es el token.
3. **Acceso a la cuenta publicitaria** (el token debe verla). Un token puede ser válido y aun así no
   ver una cuenta de otro negocio (caso Clínica Dental Aurora).

## 3. INSTALACIÓN

```bash
cd "~/miagencia/creativos-aurora"
python3 -m venv .venv-meta-cli
source .venv-meta-cli/bin/activate
pip install --upgrade pip
pip install meta-ads-cli        # instala el binario `meta-ads`
meta-ads --version              # 0.2.0
meta-ads --help                 # ver comandos
```
(Python 3.12+; probado con 3.14.)

## 4. CONFIGURACIÓN (variables de entorno)

La CLI lee (hace `load_dotenv()` del `.env` del directorio actual, y `os.getenv`):
- **`META_ACCESS_TOKEN`** — el token.
- **`META_AD_ACCOUNT_ID`** — la cuenta **sin** el prefijo `act_` (ej. `1234567890123456`).
- **`META_PAGE_ID`** — la página FB (ej. `100200300400500`).
- Opcionales: `META_API_VERSION` (default `v21.0`), `META_ADS_MAX_DAILY_BUDGET_CENTS` (tope de
  seguridad de presupuesto).

Como nuestro `.env` usa `ACCESS_TOKEN` (no `META_ACCESS_TOKEN`), se exporta al vuelo sin tocar el
`.env`:
```bash
source .venv-meta-cli/bin/activate
set -a; source .env; set +a
export META_ACCESS_TOKEN="$ACCESS_TOKEN" \
       META_AD_ACCOUNT_ID=1234567890123456 \
       META_PAGE_ID=100200300400500
meta-ads account          # valida credenciales + cuenta
```

## 5. COMANDOS DISPONIBLES (v0.2.0)

| Comando | Para qué |
|---|---|
| `account` | Muestra la cuenta configurada (valida credenciales). |
| `upload-image IMG [--live] [-y]` | **Sube una imagen** y devuelve su `image_hash`. Sin `--live` es dry-run. |
| `create --config X.yaml [--dry-run] [-y]` | Crea campaña + 1 conjunto + N anuncios desde un YAML. |
| `validate --config X.yaml` | Valida el YAML sin llamar a la API. |
| `campaigns` / `adsets` / `ads` | Listan objetos de la cuenta. |
| `status <campaign_id>` | Estado de una campaña y sus anuncios. |
| `insights` | Métricas de cuenta/campaña/conjunto/anuncio. |
| `activate` / `pause` / `delete` / `bulk-status` | Cambiar estado. |
| `budget` | Cambiar presupuesto diario. |
| `setup` | Imprime snippet de configuración. |

## 6. FLUJO END-TO-END (cómo lo interconecto)

**Tus creativos (imágenes + copy) → CLI (sube imágenes) → Graph API (monta la campaña con evento) → Meta.**

1. **Creativos ya producidos:** ten por cada uno su copy (primary texts, headlines, descriptions) y su
   imagen. Súbela a una URL pública y descárgala a disco con `curl` (o parte directo del archivo local
   si ya lo tienes).
2. **CLI — subir imágenes nativo** (la pieza que el MCP no puede):
   ```bash
   for f in pd-01 pd-02 ... ; do
     meta-ads upload-image "$f.jpg" --live -y   # imprime "Image hash: a1b2c3d4e5f60718293a4b5c6d7e8f90"
   done
   ```
   Guardar el mapeo `nombre → hash` (ej. `hashes.txt`).
3. **Evento de conversión (si aplica):** asegurarse de que el pixel tenga el evento **fresco**
   (visitar la página que lo dispara, ej. `/gracias`). Verificar con
   `GET 1122334455667788/stats?aggregation=event`.
4. **Montar la campaña.** Dos opciones:
   - **Solo CLI** (`meta-ads create --config campaign.yaml`): sirve para campañas simples de
     tráfico/leads con `LINK_CLICKS`/`LANDING_PAGE_VIEWS`. **No** soporta evento de conversión, geo por
     pin, ni variantes (ver §7). Nota: `create` **vuelve a subir** las imágenes que le pases por
     `image:` (no acepta `image_hash`).
   - **CLI + Graph API** (lo usado): subir imágenes con el CLI y montar campaña/conjunto/anuncios con
     un script (`montar_campana.py`) para poder incluir `promoted_object` (evento), `advantage_audience:0`,
     etc. reusando los hashes del CLI.
5. **Todo en PAUSA** por defecto; activar solo con OK del usuario (gasta).

## 7. ALCANCE REAL DEL CLI v0.2.0 (leído del código fuente)

**Sí hace:**
- Subir imágenes nativo (`upload-image` → `/adimages`).
- Crear campaña (`objective` de: OUTCOME_TRAFFIC/AWARENESS/ENGAGEMENT/LEADS/SALES/APP_PROMOTION).
- Crear **1 conjunto** (targeting: `countries`, `age_min/max`, `genders`, `interests`, `platforms`,
  posiciones; `optimization_goal` de: LINK_CLICKS, IMPRESSIONS, REACH, LANDING_PAGE_VIEWS,
  APP_INSTALLS, OFFSITE_CONVERSIONS, LEAD_GENERATION).
- Crear N anuncios con creativo `link_data` (image_hash + link + message + name/headline +
  description + `call_to_action`).
- Listar, status, insights, activar/pausar/eliminar, presupuesto.

**NO hace (límites → se complementa con Graph API):**
- **`promoted_object`** (pixel + evento de conversión) → sin esto, `OFFSITE_CONVERSIONS` sería
  rechazado por Meta. Es lo que obligó a montar el conjunto por Graph API.
- **Geo por pin / radio** (solo `countries`).
- **Variantes de copy** (`asset_feed_spec`) — 1 texto por anuncio.
- **Múltiples conjuntos** por YAML (`ad_set` es singular).
- **Destino WhatsApp** nativo (arma `link_data` con `link`; no `destination_type: WHATSAPP`).

**Comparativa CLI vs MCP:**

| Capacidad | MCP de Meta | CLI / token propio |
|---|---|---|
| Subir imágenes | ❌ (image_hash / image_url) | ✅ **nativo** |
| Crear campaña / conjunto | ✅ | ✅ |
| `promoted_object` (evento conversión) | ✅ | ❌ (se hace por Graph API) |
| Geo por pin / variantes de copy | ✅ (variantes vía asset_feed_spec) | ❌ |
| Crear anuncios | ✅ (app de Meta en Live) | ⛔ **requiere la app propia en Live** |

## 8. COMPLEMENTO CON GRAPH API (lo que el CLI no cubre)

Para el caso "OUTCOME_LEADS web con evento `CompleteRegistration`" se montó por script
(`creativos-cli/montar_campana.py`), reusando los `image_hash` del CLI:
- **Campaña:** `POST act_1234567890123456/campaigns` con `objective=OUTCOME_LEADS`, `status=PAUSED`,
  `special_ad_categories=[]`, **`is_adset_budget_sharing_enabled="false"`** (obligatorio si no usas
  presupuesto de campaña).
- **Conjunto:** `POST act_1234567890123456/adsets` con `optimization_goal=OFFSITE_CONVERSIONS`,
  `billing_event=IMPRESSIONS`, `daily_budget`, `targeting` (país + edad + posiciones +
  `targeting_automation.advantage_audience=0`), y **`promoted_object`**:
  ```json
  {"pixel_id":"120210987654321000","custom_event_type":"OTHER","custom_event_str":"CompleteRegistration"}
  ```
  (Alternativa estándar: `{"pixel_id":"…","custom_event_type":"COMPLETE_REGISTRATION"}`.) El evento
  debe estar **fresco** en el pixel o Meta puede no aceptarlo.
- **Creativo + anuncio (×N):** `POST /adcreatives` con `object_story_spec.link_data`
  (`image_hash`, `link`, `message`, `name`, `description`, `call_to_action`), luego `POST /ads`
  (`adset_id`, `creative:{creative_id}`, `status=PAUSED`).

## 9. EL MURO DE LA APP EN DESARROLLO (crítico hasta pasar el App Review; ver § 0)

Al crear los **anuncios** (adcreatives) con el token propio saltó:
> `code 100 / subcode 1885183` — *"La publicación con contenido publicitario se creó con una app que
> se encuentra en modo de desarrollo. Debe estar en modo público para crear este anuncio."*

- **Causa:** el token pertenece a la app **`MiAgenciaApp`** (`988776655443322`), en
  **modo desarrollo**. Verificable con `GET /debug_token?input_token=<t>&access_token=<t>`.
- **Qué SÍ se puede con la app en dev:** crear campaña, crear conjunto (incl. `promoted_object`),
  editar nombre/estado/presupuesto/segmentación.
- **Qué NO:** crear/editar el **contenido** de anuncios (adcreatives / dark posts) — ni nuevos ni
  sobre anuncios existentes.
- **Anuncios creados a mano en Ads Manager sin publicar** quedan como **borradores locales del
  navegador**: NO aparecen en la API (`/ads` devuelve `[]`). Hay que publicarlos para verlos/editarlos
  por API — y aun así, ponerles imagen/copy por API requiere la app en Live.
- **Por qué el MCP no lo sufre:** usa la **app de Meta** (ya en Live).
- **Solución:** poner la app en **Live** (developers.facebook.com → app → Configuración básica:
  política de privacidad → toggle Development→Live; puede requerir negocio verificado y revisión de
  Meta). **Es un proceso aparte** con requisitos; no es un simple switch.
- **Tras ponerla en Live**, colgar los anuncios en el conjunto ya creado:
  ```bash
  source .venv-meta-cli/bin/activate
  set -a; source .env; set +a
  export META_ACCESS_TOKEN="$ACCESS_TOKEN" EXISTING_ADSET=120210987654321000
  python "creativos-cli/colgar_ads.py"
  ```

## 10. CATÁLOGO DE ERRORES → CAUSA → FIX

| Error | Causa | Fix |
|---|---|---|
| `#200 owner has NOT grant ads_management` | Token sin acceso a esa cuenta (otro negocio) | Que el dueño asigne el System User, o usar una cuenta propia |
| `pip install meta-ads` no encontrado | El nombre oficial no existe en PyPI | Usar `meta-ads-cli`; confirmar el oficial en doc de Meta |
| `Invalid parameter … is_adset_budget_sharing_enabled` | Falta el flag al no usar presupuesto de campaña | `is_adset_budget_sharing_enabled: "false"` |
| `OFFSITE_CONVERSIONS` rechazado | Falta `promoted_object` | Añadir `promoted_object` con pixel+evento (Graph API) |
| Evento no seleccionable / no fresco | El pixel no lo dispara hace tiempo | Disparar el evento (visitar la página) y verificar en `/stats` |
| **subcode 1885183** "app en modo de desarrollo" | App del token en dev | **Poner la app en Live** |
| `upload-image` sin `--live` no sube | Es dry-run por defecto | Añadir `--live -y` |

## 11. CHECKLIST REPRODUCIBLE

1. [ ] Token con `ads_management` + acceso a la cuenta; **app en Live**.
2. [ ] `pip install meta-ads-cli` en venv; `meta-ads account` OK.
3. [ ] Ten listos tus creativos (imágenes + copy); descargar imágenes a disco.
4. [ ] `meta-ads upload-image … --live` para cada imagen → guardar hashes.
5. [ ] (Si conversión) disparar y verificar el evento del pixel.
6. [ ] Montar: `meta-ads create` (simple) **o** script Graph API con `promoted_object` (conversión).
7. [ ] Todo en PAUSA; revisar previews/insights; activar solo con OK.

## 12. ARCHIVOS DEL PROYECTO (carpeta `creativos-cli/`)

- `campaign.yaml` — campaña + 10 anuncios con copy (formato del CLI y fuente del montaje).
- `hashes.txt` — `nombre image_hash` (subidos por CLI).
- `montar_campana.py` — crea campaña + conjunto (con evento) + intenta anuncios, por Graph API.
- `colgar_ads.py` — cuelga los 10 anuncios en un conjunto existente (para cuando la app esté Live).
- `pd-01…10-*.jpg` — imágenes finales. `hoja-contactos.jpg` — mosaico de revisión.
- Entorno: `../.venv-meta-cli/` (`meta-ads-cli 0.2.0`).

---

## 13. MCP vs CLI — dos MODELOS DE ACCESO (pros/contras y cuándo usar cada uno)

No son lo mismo ni compiten: usan **identidades y apps distintas**. Entenderlo evita confusiones como
"¿no es la misma app en dev?".

| | Identidad | App detrás | Estado app | Sube imágenes | Crea anuncios | Automatizable/headless |
|---|---|---|---|---|---|---|
| **MCP** | Login **personal** (OAuth) | App **de Meta** | **Live** | ❌ | ✅ | ❌ (requiere login navegador) |
| **CLI** | **System User** (token .env) | **Tu** app | **Dev** (→ hay que poner en Live) | ✅ | ⛔ hasta Live | ✅ (solo la carpeta+token) |

**Prueba de que son credenciales distintas:** el System User token **no ve Clínica Dental Aurora** (`#200`), pero
el **MCP sí operó en Clínica Dental Aurora**. → El MCP usa tu login personal (administra todos los negocios) + app
de Meta en Live; el CLI usa tu System User + tu app en dev.

**MCP — pros/contras:**
- ✅ Funciona hoy (campañas/conjuntos/anuncios; variantes de copy vía `asset_feed_spec`, geo, etc.).
- ✅ No exige app propia ni Live. Alcanza todo lo que administre tu usuario personal.
- ❌ **No sube imágenes** (workaround: una URL pública de tu imagen).
- ❌ **Login interactivo** por navegador → no headless; depende de sesión autenticada.

**CLI — pros/contras:**
- ✅ **Sube imágenes nativo**. **Portable/automatizable** (solo el token en la carpeta, sin login) →
  ideal para cron/servidor. Token no expira.
- ❌ Requiere **la app propia en Live** (proceso engorroso) para crear anuncios. El token solo ve las
  cuentas asignadas a su negocio. CLI v0.2.0 limitada (complementar con Graph API).

**Cuándo usar cada uno (recomendación):**
- **Operar ya, día a día, con login personal disponible** → **MCP** (con el workaround de imágenes).
- **Automatizar end-to-end sin login interactivo (headless, cron, "pasar la carpeta")** → **CLI**, una
  vez la app esté en **Live**.

**Estado 2026-07-04:** el objetivo inicial (MCP de Meta funcional) está **cumplido**. El
CLI quedaba **a medias** (funciona salvo crear anuncios) hasta que la app pasara a Live.
**Estado 2026-11:** App Review aprobado y Access Tier completo: el CLI crea anuncios. Ver § 0 y la GUIA
del App Review.
