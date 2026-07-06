# Configuración — Ads CLI

> Fuente: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/setup/configuration
> Actualizado: 29 abr. 2026

Cómo la CLI de anuncios resuelve la configuración: variables de entorno, archivos `.env` y directorio de configuración.

## Variables de entorno

| Variable | Obligatorio | Descripción |
|---|---|---|
| `ACCESS_TOKEN` | Sí | Identificador de acceso de usuario del sistema (System User Token) generado en Meta Business Suite. |
| `AD_ACCOUNT_ID` | Sí (para la mayoría de comandos) | Identificador de la cuenta publicitaria de Meta. |
| `BUSINESS_ID` | No | Identificador de empresa predeterminado (para comandos de catálogo o conjunto de datos). |

## Precedencia de configuración

La CLI resuelve la configuración en este orden (mayor prioridad primero):

1. **Flags de línea de comandos** (p. ej. `--ad-account-id`)
2. **Variables de entorno**
3. **Archivo `.env` a nivel de proyecto**
4. **Configuración a nivel de usuario** (`~/.config/meta/`)

## Usar un archivo `.env`

Para no tener que fijar las variables en cada sesión, añade un `.env` en el directorio donde ejecutas la CLI:

```bash
cat > .env << 'DOTENV'
ACCESS_TOKEN='<ACCESS_TOKEN>'
AD_ACCOUNT_ID='<AD_ACCOUNT_ID>'
BUSINESS_ID='<BUSINESS_ID>'
DOTENV
```

> **Nota:** Las variables de entorno definidas en tu shell tienen prioridad sobre los valores del `.env`.

## Directorio de configuración

La CLI almacena la configuración en un directorio compatible con **XDG**:

```
~/.config/meta/
```

Para anular el directorio base, define la variable de entorno `XDG_CONFIG_HOME`.

## Consulta también

- Buscar los identificadores (cuentas, páginas, etc.): `.../ads-cli/tutorials-and-recipes#find-your-ids`
- Introducción (instalación y autenticación): `.../ads-cli/setup/get-started`
- Referencia de comandos: `.../ads-cli/command-reference`
