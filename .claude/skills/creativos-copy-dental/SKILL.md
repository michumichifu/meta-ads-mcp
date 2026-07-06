---
name: creativos-copy-dental
description: Escribe copy (primary text, titulares, descripciones, voz en off) y super prompts de imagen para anuncios de Meta Ads del sector dental. Usalo cuando el usuario pida copys, super prompts, creativos, anuncios, guiones o voz en off para una clinica dental (implantes, brackets, coronas, protesis, blanqueamiento). Base generalizada a personalizar por cliente (paleta, marca, servicios, precios).
---

# Creativos y copy para Meta Ads dental (BASE de ejemplo)

> **Qué es esto:** una **base generalizada y ejemplificada** para producir los anuncios (imágenes + copy)
> de una clínica dental en Meta Ads. **Es un punto de partida — personalízalo por cliente** (cada clínica
> tiene su paleta, marca, servicios, precios, ubicación y línea visual propias). Los `<placeholders>` los
> reemplazas con los datos reales del cliente.
>
> **Cómo se usa:** pídele a Claude (con este playbook) que te **escriba o regenere** el súper prompt de
> imagen y el copy. **Configúralo primero** con las especificaciones del cliente y de la campaña — la
> estética, la paleta, la marca, los servicios, el destino y lo que quieres lograr. Sin esas specs el
> súper prompt sale genérico; con ellas, Claude lo adapta. Luego llevas ese súper prompt a la **IA de
> imagen que uses** (ej. Gemini / Nano Banana, ChatGPT, Flux) para generar la pieza.
>
> **Alcance:** esto es **producción de creativos**, NO es lo que hace el MCP de Meta — el MCP solo
> **monta** en campañas los creativos ya hechos. Es un paso **opcional**: si ya produces tus creativos por
> otra vía, sáltatelo; esta base es para quien parte de cero.

---

## 1. Estrategia (antes de producir)

**Niveles de conciencia del paciente** (define el tono de cada pieza):
- **N1 Inconsciente:** convive con el problema y lo normalizó.
- **N2 Consciente del problema:** sabe que algo está mal pero no busca solución.
- **N3 Consciente de la solución:** sabe que el tratamiento resuelve, pero no se decide.
- **N4 Consciente del producto:** compara clínicas, precios, materiales.
- **N5 Lo más consciente:** casi decidido, necesita el empujón final.

**Ángulos de venta** (qué comunica la pieza): precio/descuento · función (masticar, mordida) ·
estético-emocional (confianza al sonreír) · proceso (rápido, cómodo) · durabilidad/calidad · bajo riesgo
(evaluación, primer paso) · autoridad (especialista, testimonios).
**Regla:** mínimo **3 ángulos** por campaña (ideal 5-6). Cada creativo = **1 ángulo × 1 nivel de
conciencia**; nunca mezclar ángulos en una pieza.

**Destino (elige por la habilidad de venta de quien atiende el chat):** WhatsApp exige mucha destreza de
cierre → conviene a clínicas que cierran bien. Si el equipo no vende bien, **landing con embudo**
(pre-filtra y la clínica contacta). Detalle de costos en el skill `analizar-campanas-meta`.

---

## 1b. Aprender de anuncios ganadores (ingeniería inversa)
Antes de producir, deconstruye 1-2 anuncios que funcionen (tuyos o de la competencia) para extraer su
**fórmula** y adaptarla. Lente rápido: **nivel de consciencia** (Schwartz), cuál de las **8 fuerzas de
vida (LF8)** activa, **NESB** (Nuevo/Fácil/Seguro/Grande), el **hook** (qué detiene el scroll) y la
**jerarquía visual** (qué se lee primero → último). Luego replica esa secuencia con TU oferta y marca.
Para un análisis completo, usa el skill `analizar-anuncio`.

## 2. Copy del anuncio

### Tono (mercado dominicano)
Natural, directo, conversacional, "aplatanao" pero profesional — como el consejo de un buen amigo, no
locutor de infomercial.
- **Evitar:** "postergas", "adquirir", "exonerado" (rebuscadas); "tumbamos los precios", "regalado",
  "barato" (abaratan la marca); "aproveche esta increíble oportunidad" (infomercial).
- **Preferir:** "hacemos la salud dental accesible", "ajustamos los precios", "te queda en RD$<PRECIO>",
  "aquí en <CLINICA>, en <CIUDAD>", "préstame 1 minuto", "sin enredos", "sin letra chiquita".

### Formatos y cantidades exactas
- **Primary text (texto principal):** 200-400 palabras, escaneable. 3 variantes por ángulo.
- **Titulares (headlines):** **exactamente 5**, máximo **40 caracteres** cada uno, con emojis.
- **Descripciones:** **exactamente 5**, 1 línea, con emojis.
- **Voz en off (solo video):** 15-20 seg (35-45 palabras).

### Estructura del primary text
1. Hook con emoji (1 línea que detiene el scroll).
2. Empatía o gancho narrativo.
3. Oferta en bullets con emojis (✅ 🦷 🛡️ 🎁).
4. Comparación Antes/Ahora si el descuento es ≥30%.
5. Disclaimer/condiciones.
6. Ubicación con referencia local ("<CIUDAD>, <SECTOR>").
7. CTA + WhatsApp visible (si el destino es WhatsApp).

**Emojis:** funcionales, no decorativos (al inicio de líneas de beneficio, en el hook, en el CTA) — el
humano **escanea**, no lee. Excepción: si el cliente pide copy sin emojis.
**CTA agnóstico al botón:** "Toca el botón aquí abajo", "Escríbenos por WhatsApp", "Agenda tu
evaluación". **Nunca** "haz clic en el enlace".

### Voz en off (para video)
- **Sin emojis** (la IA de voz los lee como "carita feliz"). Sin URLs ni teléfonos recitados.
- **Deletrear las cifras:** "cincuenta mil pesos", no "RD$50,000"; "cuarenta por ciento", no "40%".
- Salto de línea después de cada oración o coma → fuerza pausas naturales.

---

## 3. Prompts de imagen (crear el creativo con IA)

### Regla #1 (el error más común)
**Escribe el TEXTO EXACTO que debe aparecer en la imagen, nunca placeholders.** Si el prompt dice
`[TITULAR]` o "rotular en Canva", la IA lo escribe literal. Da el texto final real:
`Titular: "Tu <servicio> en una sola visita"`.

### Plantilla del prompt
```
Diseña una imagen publicitaria vertical 4:5 (1080x1350px) para Meta Ads de <SERVICIO>. Fotorrealista,
premium, legible en móvil.
ESTILO VISUAL: <tratamiento distinto por pieza — ver variedad>.
PALETA: <HEX con el rol de cada color> + tipografía sans-serif gruesa y legible.
ESCENA: <demografía exacta del modelo, expresión, contexto — o el producto físico>.
TEXTO EXACTO (respeta acentos y ñ, sin errores):
- <cada línea con su texto final real, en español>
NO inventes logos ni marcas. NO agregues texto adicional. Todo en español.
<restricción del producto — ej. implante = SOLO el poste de titanio, SIN corona>.
```

### Anatomía de 5 zonas (reorganízalas para dar variedad)
1. **Localizador:** ciudad/sector, pequeño, arriba (ej. "<CIUDAD>, <SECTOR>").
2. **Titular/gancho** según el ángulo (grande, dominante), con una palabra clave resaltada en color.
3. **Bloque de precio** cuando aplique (caja/pill/badge): "Antes RD$<X>" tachado en gris + grande
   "Ahora RD$<Y>".
4. **Imagen humana o producto** apropiada al ángulo.
5. **CTA:** botón (verde con icono de WhatsApp si el destino es WhatsApp).

### Variedad visual (NO un solo layout)
Sobre la paleta fija del cliente, varía el estilo: clean medical premium (fondo claro, alto contraste) ·
editorial magazine (bloques de color) · oscuro premium spotlight · producto hero (macro) · infográfico
(reloj/escudo + dato) · split/comparación (antes/después). **Cada pieza del set se ve distinta** — Meta
premia la variedad creativa.

### Reglas de contenido en la imagen
- **Precio = prefiltro de leads (insight central):** en campañas a WhatsApp/landing, mostrar el
  precio/descuento en el creativo **precalifica** al lead (llega sabiendo cuánto cuesta → menos "¿y
  cuánto es?"). Va **visible en la mayoría de las piezas**, formato flexible (precio directo, "mitad de
  precio", "-50%", "desde RD$<X>"). Solo piezas puramente aspiracionales (N1, a veces N2 emocional)
  pueden omitirlo. **La evaluación nunca es el titular de precio** (el precio del tratamiento manda).
- **Mostrar el PRODUCTO físico (1-2 piezas mínimo):** no basta cara + precio + texto; muestra algo
  tangible (el producto, el objeto, o el resultado antes/después). El paciente no siempre sabe qué
  necesita; verlo educa y vende.
- **Modelos:** demografía del público del ángulo (ej. implantes 35-60, brackets 18-35). **Varía los
  tonos de piel** — no todas de piel morena/canela; incluye piel clara **tipo hispano/mediterráneo, no
  nórdico** (y pídelo explícito: "tez blanca hispana, no morena ni canela"). Sonrisa sana/restaurada;
  **nada quirúrgico, sangre ni encías en primer plano**.
- **No sobre-premiumizar:** que NO todas se vean de lujo (transmite "muy caro"). Evita exceso de
  difuminado/bokeh, mucho blanco y fondos de lujo perfectos en todas; varía con estética cotidiana/real.
- **Anti-logos:** instruye "NO inventes logos ni marcas" (los generadores tienden a fabricar logos que
  comen espacio). El logo real se añade después en Canva.
- **Teléfono opcional** (no en todas); el CTA verbal siempre.
- **Paleta del cliente, FIJA:** nunca heredar colores de la agencia ni de otro cliente.

### Producción
1. Genera la base con el prompt. 2. **Revisa el texto** (la IA deforma palabras largas y acentos;
verifica precios exactos: que diga RD$<X> correcto, no una cifra parecida). 3. Retoca en Canva solo el
detalle puntual. 4. Si el producto sale mal, sobrepón una imagen stock real.

---

## 4. Cumplimiento por tipo de oferta (reglas duras, no romper)

- **Contado vs cuotas: NUNCA mezclar** en una pieza. Contado con descuento → reforzar "por pago de
  contado"; a plazos → "cuotas cómodas", "inicial accesible".
- **Evaluación con costo:** nunca "gratis" si tiene costo; "depósito de RD$<X> reembolsable al iniciar el
  tratamiento". El costo filtra curiosos; el reembolso es argumento de cierre en el chat, no gancho.
- **Inicial dividida:** aclarar que dividir en partes es facilidad de pago, no el precio total (ej.
  "RD$<X> x 2" es facilidad; la inicial completa es RD$<2X>).
- **ARS / seguros:** nunca "100% gratis" ni "cero diferencia" absoluto → condicional "según tu plan/
  cobertura", con disclaimer: "el pago de diferencia dependerá de las políticas y fondos de tu plan".
- **Promesas clínicas:** nunca "sin dolor" absoluto → "mínimas molestias con anestesia local",
  "procedimiento cómodo".
- **Componente aparte:** si el precio no incluye algo (ej. la corona en un implante), línea fina:
  "Precio del <componente>; el/la <otro> se cotiza aparte" — mucha gente no diferencia uno de otro y
  creería que todo cuesta ese precio.
- **Terminología que el público entiende:** usa el término que reconocen al instante (ej. en RD, para
  ortodoncia di **"brackets"**, no "ortodoncia"). El objetivo es que entiendan la oferta de un vistazo.
- **Nombres reales:** verifica la ortografía exacta de doctores/personas (la IA los deforma); refuerza
  la ortografía literal en el prompt.

---

## 5. Coherencia (regla de oro)
Lo que promete el anuncio debe coincidir con la **imagen** y con la **landing/WhatsApp**. Si la imagen
dice "sin costo" pero la landing cobra, hay que alinearlo. Anuncio ↔ imagen ↔ destino, siempre coherentes.

---

> **Personalización obligatoria por cliente:** paleta (HEX reales), tipografía, logo, servicios, precios,
> ubicación y ángulos que le funcionan. Esta base es el "cómo"; el "qué" concreto lo pone cada clínica.
