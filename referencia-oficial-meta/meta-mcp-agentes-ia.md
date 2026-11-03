# Conectar agentes de IA a Meta con MCP

> ⚠️ **DESACTUALIZADA (nota del 3 nov 2026):** esta copia es del 12 jun 2026 y dice que no hay servidor MCP de anuncios. Hoy sí lo hay (`https://mcp.facebook.com/ads`), con documentación propia: ver `en-2026-11/ads-mcp-server--*.md` y `REFERENCIA - Herramientas del MCP de Meta.md`.

> Fuente: https://developers.facebook.com/documentation/mcp
> Actualizado: 12 jun. 2026 · Versión Markdown oficial: https://developers.facebook.com/documentation/mcp.md

Servidores MCP remotos **oficiales** para integrar las funciones de anuncios de Facebook, Instagram, WhatsApp y Meta en los flujos de trabajo de tus agentes de IA.

## Prestaciones

- **Un protocolo, diseñado para flujos de trabajo de agentes.** El agente se conecta a un servidor MCP de Meta y descubre las herramientas disponibles en tiempo de ejecución. Las descripciones de herramientas, los esquemas de parámetros y las respuestas de error están diseñados para que los LLM los interpreten correctamente, minimizando alucinaciones y llamadas erróneas.
- **Autenticación estándar.** Autenticación basada en **OAuth** con los ámbitos de lectura y administración que controlas desde la configuración de tu cuenta de Meta. Puedes revocar el acceso en cualquier momento.

## Servidores MCP disponibles

| Servidor | Descripción | Tipo |
|---|---|---|
| **MCP de herramientas para desarrolladores de Meta** (devtools-mcp) | Administra aplicaciones de Meta, configura webhooks, comprueba el cumplimiento, revisa el estado de la aplicación y busca documentación para desarrolladores. | MCP remoto |

> Referencia del servidor devtools: https://developers.facebook.com/documentation/mcp/devtools-mcp
> Nota: por ahora este es el único servidor MCP publicado. No hay (todavía) un servidor MCP dedicado de gestión de campañas/anuncios; para eso está la **Ads CLI**.

## Clientes compatibles (validados por Meta, disponibilidad general)

- Claude Desktop
- **Claude Code**
- ChatGPT (web)
- App de Codex
- CLI de Codex
- App de Cursor
- CLI de Cursor

Cada cliente necesita compatibilidad explícita; los clientes no listados aún no son compatibles. Habrá más próximamente.

## Cómo empezar (patrón común a todos los servidores MCP de Meta)

El patrón de configuración es el mismo para todos los servidores MCP de Meta. Los fragmentos de configuración **por cliente** están en la página de referencia de cada servidor (p. ej. la del devtools-mcp).

1. **Añade la URL del servidor al cliente.** La mayoría de clientes tienen una ventana de configuración de conector o MCP.
2. **Inicia sesión con tu cuenta de Meta** cuando se te solicite.
3. **Selecciona las aplicaciones** a las que quieras conceder acceso en la pantalla de consentimiento. Puedes ajustar el alcance (lectura o administración) de cada aplicación después desde la configuración de **integraciones empresariales** de tu cuenta de Facebook.
4. **Verifica.** Pregúntale a tu agente qué herramientas tiene; deberías ver la lista de herramientas del servidor.

> Importante: el transporte es **MCP remoto** con OAuth (no stdio local). La conexión es por **URL del servidor** + inicio de sesión OAuth con la cuenta de Meta. La documentación general no incluye un snippet literal `claude mcp add`; ese snippet exacto vive en la página de referencia del servidor concreto (devtools-mcp).

## Seguridad y prácticas recomendadas

Al usar un servidor MCP de Meta, tú y el agente que actúa en tu nombre aceptáis las Condiciones de la plataforma de Meta, que rigen cómo se almacenan, tratan, comparten y muestran los datos de las API de Meta (límites de retención, usos prohibidos, transparencia con usuarios finales).

**Importante — inyección de instrucciones:** los agentes de IA actúan en tu nombre. Todo lo que un agente pueda llamar según los ámbitos concedidos, puede llamarlo directamente, incluso en respuesta a instrucciones maliciosas ocultas en los resultados de las herramientas (cargas de webhooks, documentos, páginas web). Estas instrucciones vienen de contenido no fiable que procesa el agente, no del servidor MCP. Trata los servidores MCP como cualquier otra integración de terceros.

### Prácticas recomendadas

- **Usa solo el ámbito que necesites.** Mantén cada aplicación en ámbito de lectura salvo que un agente necesite hacer cambios; define el ámbito por aplicación desde integraciones empresariales.
- **Usa aplicaciones separadas para desarrollo y producción.** No des ámbitos de producción a agentes experimentales.
- **Audita periódicamente.** Revisa los servidores MCP conectados en facebook.com > **Configuración** > **Integraciones empresariales** cada pocos meses. Revoca los que ya no uses.
- **Protégete de la inyección de instrucciones.** No concedas permisos de escritura/administración a agentes que procesan entradas no confiables.

## Más recursos

- Condiciones de la plataforma de Meta: https://developers.facebook.com/terms
- Iniciativas para una plataforma responsable: https://developers.facebook.com/docs/resp-plat-initiatives
- Integración empresarial (revocar accesos): https://www.facebook.com/settings/?tab=business_tools
- Especificación del Model Context Protocol: https://modelcontextprotocol.io/
- llms.txt: https://developers.facebook.com/llms.txt
