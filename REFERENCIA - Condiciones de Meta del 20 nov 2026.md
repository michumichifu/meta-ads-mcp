# REFERENCIA — Las condiciones de anuncios de Meta que entran el 20 de noviembre de 2026

> **Qué es:** una lectura operativa (no asesoría legal) de las Condiciones comerciales y las Condiciones de los
> anuncios de autoservicio nuevas de Meta. Se anunciaron para el 30 de octubre de 2026 y **Meta las aplazó al
> 20 de noviembre de 2026** (comprobado en las páginas en vivo el 31 oct). Ante cualquier duda manda el texto de
> Meta, y la redacción ha cambiado entre versiones: **antes de citar, copia de la página**.
>
> | Texto | URL |
> |---|---|
> | Condiciones comerciales, nuevas | `https://www.facebook.com/legal/commercial_terms_preview` |
> | Condiciones de autoservicio, nuevas | `https://www.facebook.com/legal/self_service_ads_terms_preview` |

## Por qué importa si usas el MCP, el CLI o cualquier agente de IA

**No hay nada que firmar:** seguir anunciando después de la fecha es aceptarlas. Es sobre todo una
reorganización (cuatro documentos pasan a dos: las condiciones del catálogo entran en las comerciales, y las de la
IA generativa de anuncios, en las de autoservicio), pero amplía cinco cosas:

1. **El software de terceros corre por tu cuenta.** Texto nuevo: *"Eres el único responsable de los actos u
   omisiones de cualquier software o servicio de terceros que uses o autorices"*. Si un agente, un MCP, un CLI o un
   script crea mal un anuncio, lo publica cuando no debía o gasta de más, Meta no entra a discutir quién fue. Es la
   razón de fondo de la regla de este kit: **todo se crea en pausa y nada se activa sin confirmación humana**.
2. **Las funciones automáticas y de IA dejan de llamarse "opcionales".** Desaparece la frase de que se pueden
   activar o desactivar, y el anunciante pasa a ser responsable de *"revisarlos, aprobarlos o suprimirlos, incluso
   cuando utilices nuestras funciones automatizadas"*, incluido el contenido que genere la IA de Meta. El texto no
   dice que los interruptores vayan a desaparecer de la API; lo que desaparece es el compromiso escrito.
3. **Meta puede rastrear las webs asociadas a tus anuncios** con bots y *"credenciales de prueba"*, y usar ese
   contenido para *"entrenar"* sus productos. (Esos rastreadores llegan a tu servidor con user-agent de navegador
   normal: ver el § 1.2a de `PLAYBOOK-Analizar-Campanas-Meta.md`.)
4. **Meta no garantiza resultados ni la exactitud de sus propias métricas:** *"No garantizamos la exactitud,
   integridad ni fiabilidad de las métricas, los análisis, la segmentación, los informes u otros datos"*. Otra razón
   para contar los resultados en tu propia fuente.
5. **El saldo prepagado caduca a los tres años**, y quien paga por factura tiene 30 días.

## Lo que no cambia y sigue mandando

- **Prohibido enviarle a Meta datos de salud**, financieros, biométricos u otros especialmente protegidos, y datos
  de menores de 13 años. Ahora las condiciones del píxel y la API de conversiones quedan **incorporadas por
  referencia** a las comerciales: es un solo contrato. Importa mucho en salud (clínicas): ni en los eventos, ni en
  los parámetros, ni en las URL que mandas.
- **Quien anuncia por un tercero responde por él:** Meta *"podrá hacerte responsable"* de las infracciones del
  anunciante al que representas.
- Meta puede rechazar o retirar cualquier anuncio "por cualquier motivo", y un anuncio publicado es información
  pública.

## Qué revisar después del 20 de noviembre

- Que el texto en vigor coincida con el de la vista previa.
- Crear un anuncio de prueba (en pausa) y **leer su `creative_features_spec`**: si alguna mejora automática nace
  encendida que antes no (ver el § 0 punto 5 de `PLAYBOOK-Crear-Campanas-Meta.md`).
- El desglose por emplazamiento de las campañas activas: **WhatsApp y Threads** entran como ubicaciones.
- Si las imágenes que generas con una IA de fuera conservan sus metadatos: el texto nuevo prohíbe quitar marcas de
  agua o metadatos de "contenido generado por IA" y no aclara si alcanza a lo generado fuera de Meta.
