# Referencia de permisos de Meta · Graph API y Marketing API

> **Fuente oficial**: [developers.facebook.com/docs/permissions](https://developers.facebook.com/docs/permissions/)

Guía curada de los permisos de la Graph API y la Marketing API de Meta que importan para trabajar con anuncios, páginas, negocio y clientes potenciales. El catálogo oficial lista alrededor de 200 permisos; aquí resumimos los relevantes para publicidad y dejamos el resto agrupado al final.

Cada permiso concede a una aplicación acceso a un conjunto acotado de datos o acciones. Casi todos exigen **App Review** (revisión de la aplicación por parte de Meta) para poder usarse con cuentas de terceros; hasta que se aprueban, solo funcionan en modo desarrollo con cuentas propias o asociadas.

**Nota de alcance mínimo**: una integración de solo lectura de métricas necesita únicamente `ads_read` (estadísticas de anuncios) y, opcionalmente, `business_management` (acceso a activos de Business Manager). No requiere `ads_management` ni permisos de escritura. Pide siempre el mínimo que tu caso de uso justifique.

## Permisos de anuncios y negocio (Marketing API)

| Permiso | Para qué sirve | Requiere App Review |
|---|---|---|
| `ads_read` | Da acceso a la API de estadísticas de anuncios para obtener informes de las cuentas publicitarias que te pertenecen o a las que te concedieron acceso. También habilita la API del servidor (Conversions API) para enviar eventos web desde tu servidor a Facebook. | Sí |
| `ads_management` | Permite leer y administrar cuentas publicitarias: crear campañas, administrar anuncios y recuperar métricas mediante programación. Base para herramientas de gestión de anuncios. Depende de `pages_read_engagement` y `pages_show_list`. | Sí |
| `business_management` | Habilita operaciones de lectura y escritura con la API de Business Manager: administrar activos comerciales como cuentas publicitarias y reclamarlas. Depende de `pages_read_engagement` y `pages_show_list`. | Sí |
| `attribution_read` | Acceso a la API de atribución para obtener informes de atribución de las líneas de negocio que te pertenecen o a las que te concedieron acceso. Útil para paneles de rendimiento. | Sí |
| `catalog_management` | Crear, leer, actualizar y eliminar catálogos de productos de las empresas donde el usuario es administrador. Para soluciones de comercio electrónico, viajes, anuncios dinámicos e inventarios. Depende de `business_management`. | Sí |
| `leads_retrieval` | Recuperar y leer la información registrada mediante un formulario de anuncio para clientes potenciales (lead ad) creado en el administrador de anuncios o la Marketing API. Pensado para contactar a quienes completaron el formulario o para plataformas CRM autorizadas. Depende de `ads_management`, `ads_read`, `business_management`, `pages_manage_ads`, `pages_read_engagement` y `pages_show_list`. | Sí |

## Permisos de páginas de Facebook

| Permiso | Para qué sirve | Requiere App Review |
|---|---|---|
| `pages_show_list` | Acceder a la lista de páginas que administra una persona y verificar que la administra. Es la dependencia base de casi todos los permisos de página. Sin dependencias propias. | Sí |
| `pages_read_engagement` | Leer el contenido publicado por la página (publicaciones, fotos, vídeos, eventos), la foto de perfil, los datos de seguidores (nombre y PSID) y metadatos y estadísticas de la página. Depende de `pages_show_list`. | Sí |
| `pages_read_user_content` | Leer el contenido generado por usuarios en la página (publicaciones, comentarios, calificaciones) y eliminar comentarios de usuarios. Depende de `pages_show_list`. | Sí |
| `pages_manage_ads` | Administrar los anuncios asociados a la página: crear y gestionar anuncios de la página y anuncios de clic a mensajería (Messenger, Instagram Direct, WhatsApp). Depende de `pages_show_list`. | Sí |
| `pages_manage_metadata` | Suscribirse y recibir webhooks sobre la actividad de la página y actualizar su configuración. Depende de `pages_show_list`. | Sí |
| `pages_manage_posts` | Crear, editar o eliminar publicaciones, fotos y vídeos en la página. Depende de `pages_read_engagement` y `pages_show_list`. | Sí |
| `pages_manage_engagement` | Crear, editar y eliminar comentarios en la página, e indicar o quitar "Me gusta" en publicaciones, para moderar contenido. Depende de `pages_read_user_content` y `pages_show_list`. | Sí |
| `pages_messaging` | Acceder y administrar las conversaciones de la página en Messenger: experiencias interactivas, mensajes de soporte y confirmación de reservas, compras y pedidos. Depende de `pages_manage_metadata` y `pages_show_list`. | Sí |
| `read_insights` | Leer los datos de estadísticas de páginas, aplicaciones y dominios web propiedad de una persona, para integrarlos en herramientas de análisis propias. Depende de `pages_read_engagement` y `pages_show_list`. | Sí |

## Permisos de Instagram (cuentas profesionales)

| Permiso | Para qué sirve | Requiere App Review |
|---|---|---|
| `instagram_basic` | Leer la información y el contenido multimedia del perfil de una cuenta de Instagram: metadatos básicos como nombre de usuario e identificador de una cuenta de empresa. Depende de `pages_read_user_content` y `pages_show_list`. | Sí |
| `instagram_manage_insights` | Acceder a estadísticas de la cuenta de Instagram vinculada a una página: metadatos, estadísticas de datos y estadísticas de historias de una cuenta de empresa. Depende de `instagram_basic`, `pages_read_engagement` y `pages_show_list`. | Sí |
| `instagram_content_publish` | Crear publicaciones orgánicas de foto o vídeo en el feed en nombre de una cuenta de empresa. Depende de `instagram_basic`, `pages_read_engagement` y `pages_show_list`. | Sí |
| `instagram_manage_comments` | Crear, eliminar y ocultar comentarios de la cuenta de Instagram vinculada a una página, y leer menciones y etiquetas. Depende de `instagram_basic`, `pages_read_engagement` y `pages_show_list`. | Sí |
| `instagram_manage_messages` | Leer y responder mensajes de Instagram Direct: recuperar conversaciones y gestionar mensajes con clientes, incluido vía CRM de terceros. Depende de `instagram_basic`, `pages_read_engagement` y `pages_show_list`. | Sí |

## Permisos de inicio de sesión (perfil de usuario)

| Permiso | Para qué sirve | Requiere App Review |
|---|---|---|
| `public_profile` | Leer los campos del perfil público predeterminado del nodo Usuario, para autenticar y personalizar la experiencia. **Se concede automáticamente a todas las aplicaciones.** Sin dependencias. | No (automático) |
| `email` | Leer la dirección de correo principal de una persona, para comunicarte con ella o permitir el inicio de sesión con la cuenta de Facebook. | Sí |

## Otros grupos de permisos (no relevantes para publicidad)

El catálogo oficial incluye muchas más familias de permisos que este panel no necesita, pero conviene reconocerlas:

- **Commerce / tienda**: `commerce_account_manage_orders`, `commerce_account_read_orders`, `commerce_account_read_reports`, `commerce_account_read_settings`, `commerce_manage_accounts` — gestión de pedidos, informes financieros y cuentas comerciales.
- **Contenido de marca (branded content)**: `facebook_branded_content_ads_brand`, `instagram_branded_content_brand`, `instagram_branded_content_creator`, `instagram_branded_content_ads_brand` — anuncios de socio (Partnership Ads) y contenido patrocinado.
- **Instagram avanzado**: `instagram_manage_engagement`, `instagram_manage_events`, `instagram_shopping_tag_products`, `instagram_manage_upcoming_events`, `instagram_manage_contents`.
- **WhatsApp Business**: `whatsapp_business_management`, `whatsapp_business_manage_events` — gestión de cuentas y registro de eventos.
- **Threads**: `threads_basic`, `threads_read_replies`, `threads_share_to_instagram`.
- **Páginas (mensajería y utilidades)**: `pages_manage_cta`, `pages_manage_instant_articles`, `pages_events`, `pages_utility_messages`, `pages_user_gender`, `pages_user_locale`, `pages_user_timezone`, `paid_marketing_messages`.
- **Otros**: `read_audience_network_insights`, `manage_app_solution`, `manage_fundraisers`, `publish_video`, más los permisos de gaming.

## Conceptos clave

- **Dependencias**: muchos permisos exigen que primero se conceda otro. Casi toda la familia de páginas depende de `pages_show_list`; `leads_retrieval` arrastra el conjunto completo de anuncios y páginas.
- **App Review y Advanced Access**: para usar un permiso con cuentas de terceros hay que pasar la revisión de Meta y contar con Advanced Access. En modo desarrollo (Development) los permisos solo operan sobre cuentas propias o asociadas vía Partnership.
- **Solicita solo lo mínimo**: pide únicamente los permisos que tu caso de uso justifica. Solicitar de más complica la revisión y aumenta el riesgo de rechazo.

## Recursos oficiales

- [Referencia de permisos (Graph API)](https://developers.facebook.com/docs/permissions/)
- [Documentación de App Review](https://developers.facebook.com/docs/app-review)
- [Marketing API](https://developers.facebook.com/docs/marketing-apis)
