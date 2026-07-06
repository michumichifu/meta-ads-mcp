# Parámetros de eventos del servidor (API de conversiones / CAPI)

> **Fuente oficial**: [Parámetros de eventos del servidor — Documentación para desarrolladores de Meta](https://developers.facebook.com/docs/marketing-api/conversions-api/parameters/server-event/)
> **Actualizado por Meta**: 12 feb. 2026
> Esta guía resume la referencia oficial de los parámetros de nivel superior que se envían en cada evento de la API de conversiones (Conversions API / CAPI). El contenido detallado de `user_data` y `custom_data` vive en páginas propias de Meta, enlazadas más abajo.

## ¿Qué es la API de conversiones?

La API de conversiones (CAPI) permite enviar eventos directamente desde tu servidor a Meta, en paralelo al píxel de Meta del navegador o al SDK de la aplicación. Cada evento del servidor viaja como un objeto JSON dentro del arreglo `data` de la solicitud. Los parámetros de esta tabla definen la estructura de ese objeto.

Un punto clave es la **eliminación de eventos duplicados**: cuando el mismo evento llega por el navegador (píxel) y por el servidor (CAPI), Meta los deduplica usando la pareja `event_name` + `event_id`. Sin esa coincidencia se contarían compras o leads dobles.

## Tabla de parámetros del evento del servidor

| Parámetro | Tipo | ¿Obligatorio? | Descripción |
|---|---|---|---|
| `event_name` | Cadena | Obligatorio | Nombre de un evento estándar o personalizado (p. ej. `Purchase`, `Lead`). Junto con `event_id` se usa para deduplicar eventos del navegador/app y del servidor. Para la misma acción del cliente, el `event` del navegador debe coincidir con el `event_name` del servidor. Si dos eventos coinciden dentro de 48 horas, solo cuenta el primero; si llegan casi a la vez (≤5 min de diferencia), se conserva el del navegador/app. |
| `event_time` | Entero | Obligatorio | Marca de tiempo Unix en segundos (zona horaria GMT) de cuándo ocurrió el evento real. Puede ser anterior al momento de envío. **Límite: hasta 7 días de antigüedad.** Si algún `event_time` del arreglo `data` supera los 7 días, se rechaza toda la solicitud y no se procesa ningún evento. |
| `user_data` | Objeto | Obligatorio | Mapa con la información del cliente (correo, teléfono, IP, `fbc`, `fbp`, etc.). Ver "Parámetros de información del cliente" (abajo). Estos campos se hashean con SHA-256 antes de enviarse. |
| `custom_data` | Objeto | Opcional | Mapa con datos comerciales adicionales del evento (`currency`, `value`, `content_ids`, etc.). Ver "Parámetros de datos personalizados". |
| `event_source_url` | Cadena | Opcional* | URL del navegador donde ocurrió el evento; debe coincidir con el dominio verificado. *Es **obligatorio** para eventos de sitio web compartidos por CAPI. |
| `opt_out` | Booleano | Opcional | Si es `true`, el evento no se usa para optimizar la entrega de anuncios; solo se usa para atribución. |
| `event_id` | Cadena | Opcional (recomendado) | Identificador único elegido por el anunciante. Para deduplicar, el `eventID` del navegador/app debe coincidir con el `event_id` del servidor. Sirve un número de pedido, un ID de transacción, o un número aleatorio (siempre que sea el mismo en ambos lados). |
| `action_source` | Cadena | Obligatorio | Dónde se produjo la conversión (ver valores abajo). Al usar CAPI aceptas que este valor es correcto hasta donde sabes. |
| `data_processing_options` | Arreglo | Opcional | Opciones de tratamiento a activar por evento. El valor aceptado es `LDU` (uso limitado de datos). Un arreglo vacío indica explícitamente que el evento no debe procesarse con restricciones LDU. |
| `data_processing_options_country` | Entero | Obligatorio si envías `LDU` | País asociado al tratamiento: `1` = Estados Unidos, `0` = solicitar geolocalización del evento. |
| `data_processing_options_state` | Entero | Obligatorio en ciertos casos | Estado asociado: `1000` = California, `0` = solicitar geolocalización. Si defines país debes definir estado. Es obligatorio si envías `LDU` sin dirección IP. |
| `app_data` | Objeto | Obligatorio en eventos de app | Datos de la aplicación e información del dispositivo. `extinfo` es su subparámetro. |
| `extinfo` | Objeto/Arreglo | Obligatorio en eventos de app | Información ampliada del dispositivo (arreglo de valores separados por comas, en orden indexado; ver abajo). Todos los valores son requeridos; usa cadena vacía como marcador de posición si falta alguno. |
| `referrer_url` | Cadena | Opcional | Encabezado HTTP `referrer` visto por la página que activó el evento. Suele ser la página anterior del navegador. |
| `original_event_data` | Objeto | Opcional | Metadatos para asociar un evento "retrasado" a un evento de captación pasado. Recomendado cuando hay demora entre el envío y el evento original. |
| `customer_segmentation` | Enumeración | Opcional | Segmento del usuario que realiza el evento (ver valores abajo). |

## Valores de `action_source`

Indican dónde ocurrió la conversión. Todos los valores activan las funciones de medición, creación de audiencias personalizadas y optimización de anuncios.

- `email` — la conversión se produjo por correo electrónico.
- `website` — en tu sitio web.
- `app` — en tu aplicación móvil.
- `phone_call` — por teléfono.
- `chat` — por mensajería (app de mensajes, SMS o chat en internet).
- `physical_store` — en persona, en tu tienda física.
- `system_generated` — automáticamente (p. ej. renovación de suscripción con pago automático).
- `business_messaging` — desde anuncios de clic a Messenger, Instagram o WhatsApp.
- `other` — de una manera no listada.

## `user_data` y hashing — dónde vive el detalle

Esta página de referencia trata los parámetros de nivel superior. El detalle de los campos de `user_data` (correo `em`, teléfono `ph`, nombre `fn`, apellido `ln`, ciudad, código postal, `client_ip_address`, `client_user_agent`, `fbc`, `fbp`, etc.) está en la página **"Parámetros de información del cliente"** de Meta. Reglas generales que aplican a esos campos:

- Los datos personales identificables (correo, teléfono, nombre, apellido, ciudad, estado, código postal, país, fecha de nacimiento, género) deben enviarse **hasheados con SHA-256** en minúsculas y sin espacios sobrantes.
- Excepciones que **no** se hashean: `client_ip_address`, `client_user_agent`, `fbc` (cookie `_fbc` / clic) y `fbp` (cookie `_fbp` del navegador). Se envían en texto plano.
- Para equivalencias con los datos enviados por el píxel de Meta, consulta también "Coincidencias avanzadas".

## `extinfo` (eventos de aplicación) — orden indexado

Arreglo de valores en orden fijo. `version` debe ser `a2` para Android e `i2` para iOS.

| Índice | Valor | Ejemplo |
|---|---|---|
| 0 | Versión de `extinfo` (obligatorio) | `i2` |
| 1 | Nombre del paquete de la app | `com.facebook.sdk.samples.hellofacebook` |
| 2 | Versión corta | `1.0` |
| 3 | Versión larga | `1.0 long` |
| 4 | Versión del SO (obligatorio) | `13.4.1` |
| 5 | Nombre del modelo de dispositivo | `iPhone5,1` |
| 6 | Idioma | `En_US` |
| 7 | Abreviatura de zona horaria | `PDT` |
| 8 | Operador | `AT&T` |
| 9 | Ancho de pantalla | `320` |
| 10 | Alto de pantalla | `568` |
| 11 | Densidad de pantalla | `2` |
| 12 | Núcleos de CPU | `2` |
| 13 | Almacenamiento externo (GB) | `13` |
| 14 | Espacio libre externo (GB) | `8` |
| 15 | Zona horaria del dispositivo | `USA/New York` |

## Valores de `customer_segmentation`

Contexto sobre la relación del usuario con la empresa:

- `new_customer_to_business` — nuevo cliente de la empresa.
- `new_customer_to_business_line` — nuevo cliente de una línea de negocio (producto o servicio).
- `new_customer_to_product_area` — nuevo cliente de un área de producto (comercio electrónico, finanzas).
- `new_customer_to_medium` — nuevo cliente de un medio de marketing (redes sociales, correo).
- `existing_customer_to_business` — cliente actual de la empresa.
- `existing_customer_to_business_line` — cliente actual de una línea de negocio.
- `existing_customer_to_product_area` — cliente actual de un área de producto.
- `existing_customer_to_medium` — cliente actual de un medio de marketing.
- `customer_in_loyalty_program` — forma parte de un programa de fidelidad.

## Ejemplo de carga útil JSON

```json
{
  "event_name": "Purchase",
  "event_time": 1643723400,
  "user_data": {
    "em": "user@example.com"
  },
  "custom_data": {
    "currency": "USD",
    "value": 100.00,
    "customer_segmentation": "new_customer_to_business"
  }
}
```

> En una implementación real, `em` (y los demás datos personales de `user_data`) deben enviarse hasheados con SHA-256, no en texto plano como en este ejemplo simplificado de la documentación.

## Notas de implementación

- La deduplicación píxel + servidor depende de enviar el mismo par `event_name` + `event_id` por ambos canales; conviene reutilizar el número de lead o de pedido como `event_id`.
- `event_source_url` es obligatorio para eventos web por CAPI y debe usar un dominio verificado.
- Respeta el límite de 7 días de `event_time`: un solo evento fuera de rango rechaza toda la solicitud.
- Para campañas de leads, el `action_source` típico será `website` (formularios) o `chat`/`business_messaging` (WhatsApp/Messenger).
