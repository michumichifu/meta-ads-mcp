# REFERENCIA — Las herramientas del MCP de Meta (inventario de noviembre de 2026)

> **Fuente:** la lista de herramientas que el servidor `https://mcp.facebook.com/ads` expone **en vivo**
> (consultada el 3 nov 2026) cruzada con la documentación oficial de Meta, que está copiada tal cual en
> `referencia-oficial-meta/en-2026-11/`. Son **100 herramientas**; la doc oficial lista 91 de ellas. Las que
> solo están en vivo van marcadas con **(solo en vivo)**: existen y funcionan, pero Meta aún no las
> documenta, así que su comportamiento puede cambiar.
>
> 🔴 **Meta añade herramientas sin avisar.** Cuando dudes de si algo existe, pídele a tu agente la lista
> de herramientas del MCP (`tools/list`) antes de decir que no se puede. En julio de 2026 el MCP **no
> subía imágenes**; hoy sí.

---

## 0. Lo que cambió desde la primera versión de este kit (jul 2026)

| Antes (jul 2026) | Ahora (nov 2026) |
|---|---|
| ~20 herramientas útiles, sin doc oficial del servidor de anuncios | 100 herramientas y **10 páginas oficiales** del "Ads MCP Server" |
| **No subía imágenes**: había que pasar una URL pública como `image_url` | `ads_creative_upload_media` sube **imágenes y videos** por URL o desde archivo local |
| No se podía crear un evento del píxel | `ads_pixel_event_*` y `ads_pixel_parameter_*` (Event Setup Tool) |
| Sin benchmarks del sector | `ads_insights_industry_benchmark`, `ads_insights_performance_vs_peers`, `ads_insights_auction_ranking_benchmarks` |
| Sin públicos personalizados | `ads_create_custom_audience` y compañía (incluye similares y listas de clientes con datos cifrados) |
| Solo OAuth | OAuth, **token de usuario** o **token de usuario del sistema** (solo rol *Employee*, el de *Admin* no entra) |

## 1. Cuentas y páginas

| Herramienta | Para qué |
|---|---|
| `ads_get_ad_accounts` | Lista las cuentas publicitarias a las que tienes acceso (nombre, estado, moneda, zona horaria). Usa una con `is_ads_mcp_enabled:true` |
| `ads_get_ad_account_pages` | Las páginas que ya se usaron en anuncios de una cuenta |
| `ads_get_pages_for_business` | Las páginas que pertenecen a un portafolio |
| `ads_get_user_pages` | Todas las páginas que puedes usar para anunciar |
| `ads_get_field_context` | Metadatos de un campo (tipo, si filtra, valores posibles) **antes** de pedir métricas con `ads_get_ad_entities` |
| `ads_catalog_get_businesses` **(solo en vivo)** | Los portafolios a los que tiene acceso tu identidad (no prueba que una cuenta sea de ese portafolio) |

## 2. Campañas, conjuntos y anuncios

| Herramienta | Para qué |
|---|---|
| `ads_create_campaign` | Crea una campaña (**en pausa**) |
| `ads_create_ad_set` | Crea un conjunto con segmentación, presupuesto y objetivo de optimización (**en pausa**) |
| `ads_create_ad` | Crea un anuncio con su creativo (**en pausa**) |
| `ads_update_entity` | Edita campos de una campaña, conjunto o anuncio |
| `ads_activate_entity` | Pasa a activo. **Empieza a gastar**: solo con confirmación explícita del usuario |
| `ads_get_ad_entities` | Busca campañas, conjuntos y anuncios **y** es la herramienta principal de informes (métricas, desgloses, fechas) |
| `ads_get_errors` | Los errores que **bloquean la entrega** de una campaña, conjunto, anuncio o cuenta. Lo primero que se mira si algo "no sale" |

## 3. Creativos e imágenes

| Herramienta | Para qué |
|---|---|
| `ads_creative_upload_media` **(solo en vivo)** | ⭐ **Sube imágenes o videos** al banco de la cuenta. `upload_source:"URL"` + `media_type:"IMAGE"\|"VIDEO"` + `media_url` directa y pública, o `upload_source:"LOCAL_FILE"` (abre una mini-app para elegir el archivo; solo en clientes con "MCP Apps"). Devuelve el `image_hash` o el id del video. Drive, Dropbox o Canva con inicio de sesión **no** sirven |
| `ads_creative_upload_local_image`, `ads_finalize_local_ad_image_upload`, `ads_delete_local_ad_image` **(solo en vivo)** | Las piezas internas de la subida local que usa la mini-app. No las llames a mano: usa `ads_creative_upload_media` |
| `ads_create_creative` | Crea un creativo de imagen con enlace; también personalización por ubicación y multi-medio (disponibilidad limitada) |
| `ads_creative_update` **(solo en vivo)** | Solo **nombre, estado ACTIVE y etiquetas**. 🔴 **El contenido de un creativo publicado es INMUTABLE** (imagen, textos, enlace, `url_tags`, botón): para cambiarlo hay que crear creativo y anuncio nuevos |
| `ads_creative_delete` **(solo en vivo)** | Borra un creativo que no esté en uso (míralo antes con `ads_get_creative_ads`) |
| `ads_get_creatives`, `ads_get_creative_ads` | Lista los creativos y qué anuncios usan uno |
| `ads_get_ad_images`, `ads_get_ad_videos` | El banco de imágenes y videos de la cuenta. Un video recién subido queda en `processing`: **no lo uses hasta que diga `ready`** |
| `ads_get_ad_preview` | Vista previa de un anuncio en una ubicación (Feed, Stories, Reels…). Úsala siempre antes de dar un anuncio por bueno |

## 4. Instagram

| Herramienta | Para qué |
|---|---|
| `ads_get_ig_accounts` | Las cuentas de Instagram vinculadas a la cuenta publicitaria |
| `ads_get_ig_media` | Las publicaciones recientes que se pueden promocionar |
| `ads_boost_ig_post` | Convierte una publicación orgánica en anuncio |

## 5. Públicos personalizados

| Herramienta | Para qué |
|---|---|
| `ads_get_ad_account_custom_audiences`, `ads_get_custom_audience`, `ads_get_custom_audience_adsets` | Listar, ver tamaño y estado, y qué conjuntos usan un público |
| `ads_create_custom_audience` | Crear: lista de clientes, web, similar, app, interacción o conversiones offline |
| `ads_update_custom_audience`, `ads_update_custom_audience_users` | Editar la regla o añadir y quitar personas (datos **cifrados**) |
| `ads_delete_custom_audience` | Borrar |

## 6. Informes y diagnóstico

| Herramienta | Para qué |
|---|---|
| `ads_get_ad_entities` | Métricas (gasto, impresiones, CTR, CPC, CPM, conversiones) con filtros, desgloses y rangos |
| `ads_insights_performance_trend` | Cómo evolucionan CPC, CPM, costo por resultado, ROAS, CTR y tasa de conversión |
| `ads_insights_anomaly_signal` | Detecta desviaciones que merecen investigarse |
| `ads_insights_industry_benchmark` | Compara un conjunto con anunciantes parecidos |
| `ads_insights_performance_vs_peers` **(solo en vivo)** | Percentil de cada métrica de un conjunto frente a su grupo (más alto es mejor, también en costos). Máximo 5 conjuntos por llamada |
| `ads_insights_auction_ranking_benchmarks` | Qué anuncios ganaron la subasta y dónde invertir en puja o calidad |
| `ads_insights_advertiser_context` | Resume el negocio y el embudo para elegir el objetivo de optimización |
| `ads_get_opportunity_score` | La puntuación de optimización de la cuenta (0-100) con recomendaciones. 🔴 Son sugerencias de Meta, **no órdenes**: muchas suben el gasto o abren la segmentación |
| `ads_account_get_activity_logs` | El historial de cambios de la cuenta (quién cambió qué y cuándo), como en el Administrador |
| `ads_library_search` | La Biblioteca de anuncios pública (ver competencia). Mide lo que está **publicado**, no lo que funciona |
| `offer_report` **(solo en vivo)** | Ofrece programar el mismo análisis cada semana. No crea nada por sí sola |

## 7. Píxel, conjuntos de datos y conversiones

| Herramienta | Para qué |
|---|---|
| `ads_get_datasets` | Los píxeles o conjuntos de datos de una cuenta o portafolio |
| `ads_get_dataset_details` | Configuración, incluida la API de conversiones |
| `ads_get_dataset_stats` | Volumen de eventos (hasta 28 días), por dispositivo o por fuente (navegador/servidor) |
| `ads_get_dataset_quality` | Calidad de coincidencia (EMQ) por evento, cobertura de cada dato y frescura |
| `ads_pixel_event_read/create/update/delete` | Reglas de evento sin código (Event Setup Tool). Se crean **inactivas** |
| `ads_pixel_parameter_read/create/update/delete` | Extractores de parámetros del píxel |
| `ads_get_customconversions` | Las conversiones personalizadas de la cuenta (crearlas sigue sin poder hacerse por MCP) |

## 8. Pruebas A/B y estudios de conversion lift

| Herramienta | Para qué |
|---|---|
| `ads_experiment_list_tests`, `ads_experiment_check_eligibility` | Ver las pruebas existentes y si una cuenta o unos conjuntos pueden probarse |
| `ads_experiment_abtest_create_test/get_test/update_test` | Crear, leer resultados, editar o cancelar una prueba A/B |
| `ads_experiment_lift_create_test/get_test` | Estudios de conversion lift |

## 9. Catálogo (e-commerce)

31 herramientas `ads_catalog_*`: catálogos, productos, product sets (con filtro en lenguaje natural), feeds y
sus sesiones de subida, reglas de feed, fuentes de eventos conectadas al catálogo, diagnóstico y salud para
anuncios Advantage+ de catálogo, e integraciones con Shopify, WooCommerce y Magento
(`ads_catalog_list_partner_integrations`, solo en vivo). La tabla completa, en
`referencia-oficial-meta/en-2026-11/ads-mcp-server--ads-mcp-server-tools-catalog-creation-and-management.md`.

## 10. Ayuda y alianzas

| Herramienta | Para qué |
|---|---|
| `ads_get_help_article` | Busca en el Centro de ayuda para empresas de Meta. Úsala antes de afirmar cómo funciona algo de Meta |
| `ads_fb_partner_permission_manager` **(solo en vivo)** | Comprueba (y, solo con permiso explícito, pide) el permiso de un creador para anuncios de colaboración. Una solicitud enviada no se puede retirar |

---

## Lo que el MCP sigue sin hacer (y la salida)

- **Crear conversiones personalizadas**: por la API de Marketing o en el Administrador.
- **Editar el contenido de un creativo publicado**: no se puede por ninguna vía; se crea otro anuncio.
- **Mandar eventos a la API de conversiones** (por ejemplo, una venta o una cita que pasó fuera de la web):
  el MCP los **lee**, no los envía. Se mandan por la API de conversiones (`POST /<DATASET_ID>/events`),
  ver `PLAYBOOK-Analizar-Campanas-Meta.md`.

## Los campos que el MCP pide en cada llamada

Las herramientas piden, además de sus datos, `client_conversation_id` (un id aleatorio de 20 caracteres que
se repite en toda la conversación), `advertiser_job` (qué está intentando hacer el usuario) y
`advertiser_request` (su pedido, con sus palabras). Tu agente los rellena solo; no contienen datos tuyos
y **nunca deben llevar nombres, teléfonos ni ids de cuenta**.
