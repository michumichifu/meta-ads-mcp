# Empezar — Instalación y autenticación de la Ads CLI

> Fuente: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/setup/get-started
> Actualizado: 29 abr. 2026

Guía para instalar la CLI de anuncios, autenticarse con un **identificador de acceso de usuario del sistema** y ejecutar los primeros comandos.

## Requisitos

- **Python 3.12 o posterior**
- Un entorno virtual configurado.
- Administradores de paquetes **pip** y **uv** (https://docs.astral.sh/uv/).
- Un **identificador de acceso de usuario del sistema (System User Token)** de Meta con los ámbitos adecuados (ver Autenticación).
- Una cuenta publicitaria con los activos que quieres administrar.

## Instalación

### Paso 1: Instalar el paquete

El paquete `meta-ads` está en PyPI (https://pypi.org/project/meta-ads/):

```bash
pip install meta-ads
```

### Paso 2: Sincronizar dependencias y configurar el entorno virtual

```bash
uv sync
```

Después de configurar, ejecuta comandos con `uv run meta` o activa el entorno virtual para usar la CLI directamente.

## Autenticación

> **Importante:** La CLI necesita un **identificador de acceso de usuario del sistema** para autenticar el acceso programático.

### Paso 1: Crear un usuario del sistema

1. Ve a **Meta Business Suite** (https://business.facebook.com/) > **Configuración** > **Usuarios** > **Usuarios del sistema**.
2. Haz clic en **Añadir** para crear un nuevo usuario del sistema.
3. Establece el rol en **Administrador**.
4. Asígnale un nombre descriptivo (p. ej. "CLI de anuncios").

### Paso 2: Asignar activos al usuario del sistema

Haz clic en el usuario del sistema y selecciona **Asignar activos**. Concede acceso a:

- Los conjuntos de datos (píxeles de Meta) para seguimiento de conversiones.
- Las cuentas publicitarias que quieres administrar.
- Las páginas de empresa que usarás para el contenido de los anuncios.
- Los catálogos de productos, si usas anuncios de catálogo.

### Paso 3: Añadir el usuario del sistema como administrador de la aplicación

1. Ve a tu app en **Meta for Developers** (https://developers.facebook.com/apps) > **Configuración de la aplicación** > **Roles** > **Roles**.
2. Añade el usuario del sistema como **administrador de la aplicación**.

### Paso 4: Generar un identificador de acceso

1. En Meta Business Suite > **Configuración** > **Usuarios** > **Usuarios del sistema**, selecciona el usuario del sistema.
2. Haz clic en **Generar nuevo identificador**.
3. Selecciona la aplicación.
4. Concede los siguientes ámbitos:
   - `business_management`
   - `ads_management`
   - `pages_show_list`
   - `pages_read_engagement`
   - `pages_manage_ads`
   - `catalog_management`
   - `read_insights`

   Añade cualquier ámbito adicional que necesites para tu caso de uso.
5. Haz clic en **Generar identificador** y copia el identificador generado.

> ADVERTENCIA: estos ámbitos son de **escritura/administración** (`ads_management`, `pages_manage_ads`, etc.), muy por encima del `ads_read` de solo lectura. Solo la creación de campañas por CLI los necesita; para lectura de métricas basta `ads_read` + `read_insights`.

### Guardar el identificador

**Opción A — archivo `.env`:** crea un `.env` en el directorio del proyecto:

```
ACCESS_TOKEN=<ACCESS_TOKEN>
```

**Opción B — variable de entorno (CI o scripts):**

```bash
export ACCESS_TOKEN=<ACCESS_TOKEN>
```

### Comprobar el estado de la autorización

```bash
meta auth status
```

## Configurar el identificador de la cuenta publicitaria

La mayoría de comandos requieren un `AD_ACCOUNT_ID`. Configúralo una vez y aplica a todos los comandos:

```bash
# Via .env (recomendado)
AD_ACCOUNT_ID=<AD_ACCOUNT_ID>

# Via variable de entorno
export AD_ACCOUNT_ID=<AD_ACCOUNT_ID>

# Via flag (por comando)
meta ads --ad-account-id <AD_ACCOUNT_ID> campaign list
```

Para recuperar los identificadores de cuentas publicitarias:

```bash
meta ads adaccount list
```

## Ejecutar los primeros comandos

```bash
# Listar tus campañas
meta ads campaign list

# Rendimiento (por defecto últimos 30 días)
meta ads insights get --fields spend,impressions,ctr,cpc

# Listar tus páginas de empresa (necesarias para crear creatives)
meta ads page list
```
