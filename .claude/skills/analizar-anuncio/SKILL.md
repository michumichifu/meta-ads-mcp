---
name: analizar-anuncio
description: Deconstruye la imagen de un anuncio (ingenieria inversa) para entender por que funciona y extraer su formula replicable. Usalo cuando el usuario suba o pegue una imagen de un anuncio/ad/creativo y pida analizarlo, deconstruirlo, entender por que funciona, hacer ingenieria inversa o extraer aprendizajes para replicar. Marcos: consciencia y sofisticacion de Schwartz, LF8, NESB, hook, jerarquia visual y la formula del anuncio.
---

# Análisis inverso de anuncios (ingeniería inversa de un creativo)

> **Qué es:** cómo deconstruir un anuncio estático de alto rendimiento (tuyo o de la competencia) para
> extraer su **fórmula** y poder replicarla. Le pasas a Claude la **imagen de un anuncio** y este produce
> un reporte que explica *por qué funciona* y termina con una **recreación de la fórmula** aplicada a otro
> producto. Salida: un archivo Markdown (`analisis-ad-AAAA-MM-DD.md`).
>
> Sirve para **cualquier rubro/plataforma** (Meta, Instagram, Google, etc.). Está basado en marcos
> clásicos de marketing de respuesta directa (de dominio público): la **consciencia y sofisticación de
> mercado** de Eugene Schwartz, las **8 fuerzas de vida (LF8)** de Drew Whitman, el marco **NESB** y los
> conceptos de **USP / mecanismo único**. No inventes datos: analiza solo lo que la imagen muestra.

---

## Cómo usarlo
1. Pega la imagen del anuncio a analizar (si no hay imagen, pídela con una línea: *"Pégame la imagen del
   anuncio que quieres deconstruir"*).
2. Claude deconstruye la imagen aplicando los marcos de abajo y **cubre todas las secciones** (si algo no
   aplica, lo dice en una línea con el porqué).
3. Guarda el reporte en un `.md` en el directorio actual. En el chat, solo confirma ("Listo, guardado").

---

## Marcos base (el lente del análisis)

- **Consciencia de mercado (Schwartz):** inconsciente → consciente del problema → consciente de la
  solución → consciente del producto → el más consciente. *¿A qué nivel le habla el anuncio?*
- **Sofisticación de mercado (Schwartz, niveles 1–5):** qué tan "gastado" está el mercado y cómo el
  anuncio se diferencia para no verse como todos los demás (nivel 1 = claim directo; nivel 5 =
  identificación/mecanismo, porque los claims ya no bastan).
- **LF8 — 8 fuerzas de vida (deseos biológicos):** (1) supervivencia/salud/vida larga · (2) disfrute de
  comida y bebida · (3) libertad del miedo, dolor y peligro · (4) compañía sexual · (5) condiciones de
  vida cómodas · (6) ser superior / ganar / no quedar atrás · (7) cuidar y proteger a los seres queridos ·
  (8) aprobación social. *¿Cuál activa principalmente?*
- **NESB:** ¿cómo se enmarca como **N**uevo, **F**ácil (Easy), **S**eguro (Safe) y **G**rande (Big)?
- **USP / mecanismo único (UM) / mecanismo único de prueba (UMP):** ¿cuál es la propuesta única, el
  "cómo" propietario, y la prueba que lo valida (captura, estadística, antes/después, testimonio, badge)?
- **Valencia e intensidad emocional:** la emoción dominante (miedo, esperanza, aspiración, curiosidad,
  alivio, orgullo, FOMO…), su dirección (hacia lo deseado / lejos de lo temido) y su intensidad (1–10),
  con evidencia.
- **Jerarquía visual y flujo ocular:** en qué aterriza el ojo primero, y su recorrido (patrón Z, F, punto
  focal único); qué técnicas fuerzan ese orden (contraste de tamaño/color, espacio en blanco, señales
  direccionales, la mirada de las personas). Principios Gestalt (proximidad, similitud, cierre,
  figura-fondo). *¿El orden de lectura coincide con la secuencia de persuasión?*

---

## Secciones del reporte (cúbrelas todas; sé específico, cita elementos concretos del anuncio)

1. **Transcripción / inventario:** todo el texto (titular, subtítulo, cuerpo, CTA, letra fina, badges,
   watermarks), los elementos visuales (producto, lifestyle, íconos, logo, fondo) y el layout (posición
   relativa de cada elemento). Contexto de plataforma visible.
2. **Hook (detención de scroll):** qué detiene el ojo en <1 s (titular, imagen, patrón interrumpido);
   qué elemento captura primero y por qué; a qué nivel de consciencia apunta; cómo trabajan juntos el
   hook del titular y el hook visual.
3. **Jerarquía visual y flujo ocular:** la secuencia de lectura y las técnicas que la fuerzan (ver marcos).
4. **Estrategia de imagen:** tipo (producto, lifestyle, UGC, antes/después, captura, solo texto, IA…);
   qué comunica sin el texto; ¿estado del problema, de la solución, o transformación?; ¿identificación o
   aspiración?; prominencia del producto; señales de autenticidad (pulido vs crudo/orgánico) — ¿correcto
   para la audiencia?
5. **Tipografía y copy en imagen:** fuentes y qué señalan (autoridad, casual, urgencia, premium);
   jerarquía de tamaños (titular/subtítulo/cuerpo/CTA); color y contraste; tratamientos (destacados,
   tachados, manuscrito, emojis); ratio texto-imagen.
6. **Color:** dominantes y sus asociaciones; contraste entre elementos clave; ¿colores de marca vs
   colores de captar atención — cuál prioriza?; ¿choca o se mimetiza con la UI de la plataforma?
7. **Consciencia y sofisticación:** dónde está la audiencia y cómo el anuncio lo aborda en un solo frame;
   cómo se diferencia del panorama competitivo.
8. **La gran idea:** el concepto central que hace distinta la oferta, en un frame.
9. **Deseos y LF8:** deseo principal + secundarios; cuál de las LF8 activa (principal y secundaria).
10. **Emociones, comportamientos, experiencias, demografía y eventos detonadores del avatar:** qué siente,
    qué hace hoy, qué intentó antes, a quién le habla, y qué momento de vida crea urgencia/receptividad.
11. **Transferencia de creencias y manejo de objeciones:** cómo desplaza la creencia del espectador
    (implicación, yuxtaposición, prueba, reencuadre); qué objeciones desactiva y dónde (una línea, un
    badge, un elemento visual de prueba); descalificación de alternativas (aunque sea implícita).
12. **Mecanismo único (UM) y su prueba (UMP):** el "cómo" propietario y la evidencia que lo valida.
13. **NESB:** cómo se presenta Nuevo / Fácil / Seguro / Grande.
14. **Apilamiento de beneficios:** cómo apila varios beneficios en poco espacio (subtexto, implicación
    visual, copy comprimido).
15. **CTA:** acción pedida, urgencia, reducción de fricción, prominencia; ¿CTA en el creativo o depende
    del botón nativo? ¿cómo trabajan juntos?
16. **Prueba social:** calificaciones, reseñas, testimonios, "visto en", conteos, antes/después,
    certificaciones, influencers.
17. **Lenguaje conversacional y contenido nativo:** coloquialismos, preguntas retóricas, contracciones,
    1ª/2ª persona, imperfecciones deliberadas; ¿se parece a contenido orgánico/meme/captura o es
    obviamente branded?; formato (cuadrado/vertical/story) y su propósito; ¿un usuario en scroll rápido
    pausaría creyéndolo orgánico?
18. **Posicionamiento:** ¿Amigo/par, Autoridad, o Híbrido? Con evidencia del copy y los visuales.
19. **Fórmula del anuncio:** deconstrúyelo como una **secuencia espacial** de persuasión (el argumento se
    entrega en el espacio, no en el tiempo). Ej.: *"Interrupción visual → brecha de curiosidad en el
    titular → prueba/identificación en la imagen → mecanismo en el subtítulo → apilamiento de beneficios →
    CTA con badge de urgencia".* Nómbrala si coincide con un arquetipo (Problema-Agita-Solución, captura
    de testimonio, Nosotros vs Ellos, listicle de beneficios, repost UGC, antes/después, meme-hijack,
    tarjeta editorial nativa).
20. **Recreación de la fórmula (producto ficticio):** aplica la fórmula identificada a un producto
    **completamente distinto**, como un brief listo: titular + todo el copy, dirección de imagen,
    layout (dónde va cada elemento), color y tipografía, plataforma y formato. Esto prueba que
    entendiste la fórmula y te da una plantilla replicable.

---

## Reglas de calidad
- **No omitas secciones:** si un elemento no está, dilo y explica qué implica estratégicamente.
- **Sé específico:** cita elementos concretos del anuncio como evidencia, no frases genéricas.
- **Nombra los patrones** con su término técnico.
- **Cada sección cierra con un insight accionable** que se pueda aplicar a tus creativos.
- Idioma: español neutro, sin HTML, acentos y ñ correctos.
- Para **producir** tu propio creativo a partir de lo aprendido, ver el skill `creativos-copy-dental`.
