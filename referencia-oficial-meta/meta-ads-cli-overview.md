# CLI de anuncios (Meta Ads CLI)

> Fuente: https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/ads-cli-overview
> Actualizado: 29 abr. 2026

**La CLI de anuncios es una herramienta de línea de comandos** para administrar la publicidad de Meta desde el terminal. Es una interfaz para desarrolladores sobre la **API de marketing de Meta**: permite crear campañas de principio a fin, administrar catálogos de productos y píxeles de Meta, y consultar insights de rendimiento.

Diseñada para quienes administran publicidad mediante programación:

- **Desarrolladores de integraciones de administración de anuncios** que quieren prototipar y probar procesos de la API de marketing desde el terminal antes de escribir código.
- **Equipos de operaciones** que automatizan creación, supervisión y limpieza de campañas mediante scripts y pipelines CI/CD.
- **Agentes y herramientas de IA** que interactúan con la publicidad de Meta mediante una interfaz de línea de comandos estructurada.

## Funciones clave

### Administración completa del ciclo de vida (CRUD en toda la jerarquía)

| Recurso | Operaciones |
|---|---|
| Campañas publicitarias | `list`, `create`, `get`, `update`, `delete` |
| Conjuntos de anuncios | `list`, `create` (con segmentación, píxeles, seguimiento de conversiones), `get`, `update`, `delete` |
| Anuncios | `list`, `create` (con especificaciones de seguimiento), `get`, `update`, `delete` |
| Contenido de los anuncios (creatives) | `list`, `create`, `get`, `update`, `delete` |
| Insights | `get` (con intervalos de fechas, desgloses y campos personalizados) |
| Catálogos de productos | `list`, `create`, `get`, `update`, `delete` |
| Productos | `list`, `create`, `get`, `update`, `delete` |
| Conjuntos de productos | `list`, `create`, `get`, `update`, `delete` |
| Conjuntos de datos (píxeles) | `list`, `create`, `get`, `connect`, `disconnect`, `assign-user` |
| Cuentas publicitarias | `list`, `get`, `current` |
| Páginas empresariales de Facebook | `list`, `get` |

### Insights de rendimiento

- Datos de rendimiento con intervalos de fechas flexibles, desgloses y métricas personalizadas.
- Filtrado por campaña, conjunto de anuncios o anuncio concreto.
- Ordenación y límite de resultados.

### Catálogos y seguimiento de conversiones

- Catálogos de productos para anuncios de catálogo Advantage+.
- Conjuntos de datos (píxeles) para seguimiento de conversiones.
- Conectar conjuntos de datos a cuentas publicitarias y catálogos.
- Asignar permisos de usuario en conjuntos de datos.

### Automatización lista

- Flags `--no-input` y `--force` suprimen las peticiones interactivas para funcionamiento inatendido.
- Tres formatos de salida: `table` (interactivo), `json` (para `jq` y otras herramientas), `plain` (separado por tabulaciones, para pipelines Unix).
- **Códigos de salida coherentes (0-5)** para manejo fiable de errores en scripts.
- Compatible con archivos `.env` para administrar credenciales por entorno.

## Cómo funciona

Se autentica con un **identificador de acceso de usuario del sistema (System User Token)** de Meta y llama a la API de marketing en tu nombre. La estructura de comando sigue el patrón recurso + verbo:

```
meta ads <resource> <action> [options]
```

Ejemplos:

```bash
meta ads campaign list
meta ads creative create --name "My Ad" --page-id <PAGE_ID> --image ./banner.jpg
```

Todos los comandos requieren un **identificador de cuenta publicitaria** (`AD_ACCOUNT_ID`), que se puede fijar una vez como variable de entorno o pasar por comando. Cuando es posible, la CLI resuelve los identificadores de empresa (`BUSINESS_ID`) para operaciones de catálogo y conjunto de datos a partir de la cuenta publicitaria.

> Nota de seguridad para creación: todos los recursos creados salen en estado **`PAUSED`** por defecto (ver Referencia de comandos). Nada se pone en vivo sin `--status ACTIVE` explícito.

## Próximos pasos (docs relacionadas)

- Introducción (instalación y autenticación): `.../ads-cli/setup/get-started`
- Configuración (variables de entorno y precedencia): `.../ads-cli/setup/configuration`
- Referencia de comandos: `.../ads-cli/command-reference`
- Tutoriales y recetas: `.../ads-cli/tutorials-and-recipes`
- Contenido de los anuncios (imágenes, vídeos, creatives): `.../ads-cli/ad-creatives`
- Conjuntos de datos y catálogos: `.../ads-cli/datasets-and-catalogs`
- Insights: `.../ads-cli/insights`
