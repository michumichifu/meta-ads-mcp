# Uso de la API · Enviar eventos con la API de conversiones (doc oficial de Meta)

> **Fuente oficial**: [developers.facebook.com/docs/marketing-api/conversions-api/using-the-api](https://developers.facebook.com/docs/marketing-api/conversions-api/using-the-api)
> **Última actualización del documento**: 6 de febrero de 2026
> **Nota**: el contenido original de Meta fue traducido al español con IA; puede contener imprecisiones. Este artículo es un resumen curado y fiel de esa página oficial.

Esta es la página "Uso de la API" de la **API de conversiones**. Explica cómo enviar eventos del servidor, cómo probarlos antes de producción y qué límites aplican. La API de conversiones se apoya en la API de marketing de Facebook, que a su vez se apoya en la API Graph, y sigue el ciclo de versiones de la API Graph (cada versión se admite al menos dos años).

## Enviar solicitudes

Para enviar eventos nuevos se hace una solicitud **POST** al perímetro `/events` del píxel:

```
https://graph.facebook.com/{API_VERSION}/{PIXEL_ID}/events?access_token={TOKEN}
```

Al publicar en este perímetro, Meta crea nuevos eventos del servidor. El identificador de acceso se adjunta mediante el parámetro de consulta `access_token`. También se puede usar el **explorador de la API Graph** para hacer la solicitud POST a `/<pixel_id>/events`.

| Parte de la solicitud | Valor |
|---|---|
| Método | `POST` |
| Perímetro | `/{PIXEL_ID}/events` |
| Host | `https://graph.facebook.com/{API_VERSION}` |
| Autenticación | parámetro `access_token` |
| Cuerpo | objeto `data` con uno o varios eventos |

Un cuerpo mínimo incluye `event_name`, `event_time`, `event_id`, `event_source_url`, `action_source` y `user_data` (con datos del cliente ya cifrados con hash cuando corresponde), más un `custom_data` opcional (`value`, `currency`, `content_ids`, etc.).

## Hora de subida frente a hora de transacción

El campo `event_time` es la **hora de la transacción** del evento, no la hora de envío. Se manda como marca de tiempo Unix en segundos e indica cuándo ocurrió el evento real.

| Regla | Detalle |
|---|---|
| Formato de `event_time` | marca de tiempo Unix en segundos |
| Puede ser anterior al envío | Sí — habilita optimización y procesamiento por lotes |
| Antigüedad máxima (web/app) | hasta **7 días** antes del envío |
| Si algún evento supera 7 días | se devuelve error para **toda la solicitud** y no se procesa ningún evento |
| Eventos offline / tienda física (`action_source: physical_store`) | subir dentro de los **62 días** posteriores a la conversión |

## Solicitudes por lotes

- Se puede enviar un **máximo de 1000 eventos** en `data`.
- Para un rendimiento óptimo, Meta recomienda enviar los eventos en cuanto ocurren, idealmente dentro de **una hora**.
- Si **alguno** de los eventos del lote no es válido, **se rechaza todo el lote**.

## Algoritmos hash

Algunos parámetros de `user_data` deben cifrarse con hash antes de enviarse a Meta (ver la página de parámetros de información del cliente). Si usas uno de los **SDK para empresas** (Business SDK), el propio SDK aplica el hash automáticamente.

## Funciones del SDK para empresas

El Business SDK ofrece tres funciones pensadas para la API de conversiones: **solicitudes asíncronas**, **lotes simultáneos** e **interfaz de servicio HTTP**. Versión mínima del lenguaje para usarlas:

| Lenguaje | Versión mínima |
|---|---|
| PHP | >= 7.2 |
| Node.js | >= 7.6.0 |
| Java | >= 8 |
| Python | >= 2.7 |
| Ruby | >= 2 |

(La compatibilidad del Business SDK con PHP 5 se retiró en enero de 2019.)

## Verificación de eventos

Tras enviar eventos, se confirma su recepción en el **Administrador de eventos**:

1. En **Orígenes de datos**, abre el píxel correspondiente al `PIXEL_ID` de la solicitud POST.
2. Entra en **Información general**: ahí ves el número de eventos **sin procesar, coincidentes y atribuidos** recibidos.
3. En **Método de conexión** ves por qué canal llegó el evento.

Al empezar a enviar, los eventos deberían verse en un plazo de **20 minutos**.

## Herramienta "Probar eventos"

Sirve para verificar que Meta recibe correctamente los eventos del servidor. Ruta: **Events Manager > Data Sources > Your Pixel > Test Events**. La herramienta genera un identificador de prueba que se envía como parámetro `test_event_code`; a partir de ahí la actividad aparece en la ventana "Probar eventos".

> **Importante**: `test_event_code` es solo para pruebas y debe **eliminarse** en la carga útil de producción. Los eventos enviados con `test_event_code` **no se omiten**: llegan al Administrador de eventos y se usan para segmentación y medición de anuncios.

## Opciones de tratamiento de datos (usuarios de EE. UU.)

Para el uso limitado de datos (LDU) se añaden a cada evento los campos `data_processing_options`, `data_processing_options_country` y `data_processing_options_state`.

| Objetivo | Configuración |
|---|---|
| No activar LDU | `data_processing_options: []` (matriz vacía) o quitar el campo |
| Activar LDU con geolocalización de Meta | `["LDU"]`, `country: 0`, `state: 0` |
| Activar LDU con ubicación manual (ej. California) | `["LDU"]`, `country: 1`, `state: 1000` |

La API de conversiones offline también permite subir eventos manualmente desde un archivo `.csv`; en ese caso las opciones de tratamiento de datos (opción, país y estado) se añaden como columnas del archivo.

> Nota de Meta: los eventos de la aplicación y las API de conversiones offline ya no se recomiendan para integraciones nuevas. En su lugar se recomienda la **API de conversiones**, que hoy admite eventos web, de app y offline.

## Límites de la API

Este es el único apartado de la página que toca limitación de frecuencia (rate limiting):

- La **API de marketing** tiene su propia lógica de limitación de frecuencia, **independiente** de la de la API Graph. Las llamadas a la API de marketing **no cuentan** para la restricción de la API Graph.
- **No hay un límite de frecuencia específico para la API de conversiones.** Sus llamadas se contabilizan como llamadas a la **API de marketing**.
- La única restricción propia de la API de conversiones es el máximo de **1000 eventos por solicitud**.

Para el detalle de puntos, cuotas y encabezados de uso (por app, por usuario y Business Use Case) consulta el resumen interno "Rate limits de la Marketing API" del Centro de Conocimiento y la limitación de frecuencia de la API de marketing en la documentación oficial.

## Deduplicación (puerta de enlace)

Si trabajas con la puerta de enlace de la API de conversiones (CAPI Gateway) mediante el Business SDK, pasa el valor `eventId` en la solicitud para **eliminar eventos duplicados** cuando la publicación por la API de conversiones está activada. El SDK expone el objeto `CAPIGatewayIngressRequest` con opciones como `setSendToDestinationOnly` (enviar solo al extremo seleccionado, predeterminado `False`) y `setFilter` (función que decide, evento a evento, si se envía o se omite).
