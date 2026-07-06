---
name: sugeridor-productos-catalogo
description: Sugiere que productos agrupar y testear como conjuntos de productos (product sets) en campañas de catalogo / anuncios dinamicos (DPA/Advantage+ catalog) de Meta. A partir del gasto por producto de la cuenta genera hipotesis de prueba accionables. Usalo cuando el usuario tenga un catalogo/e-commerce y pregunte que productos testear, que meter en un product set, cuales estan subexpuestos o sin oportunidad, o pida ideas/hipotesis de testeo para su catalogo de Meta Ads. Es 100% de SOLO LECTURA: no crea nada, corre sobre los insights de anuncios (no toca los endpoints de catalogo).
---

# Sugeridor de productos para testear (catálogo / DPA)

Responde una sola pregunta: **¿qué productos debería agrupar y testear como conjuntos de productos?**
Analiza cómo se reparte la **inversión publicitaria** entre los productos de la cuenta y propone hipótesis
de prueba concretas. Solo aplica si el usuario corre **anuncios de catálogo / dinámicos (DPA)**.

**Es de SOLO LECTURA.** No crea conjuntos, no modifica el catálogo, no consulta endpoints de catálogo.
Corre enteramente sobre `ads_get_ad_entities` (insights de anuncios) → es estable aunque el API de
catálogo esté caído. Crear el conjunto lo decide y lo hace el usuario en Commerce Manager.

## Qué necesitas
- **`<AD_ACCOUNT_ID>`** — el ID de la cuenta publicitaria (lo único obligatorio; NO hace falta el
  `catalog_id`, el gasto por producto sale de los insights). Si no lo dio, pídelo (no lo adivines).

## Honestidad (ponlo en el reporte, no lo escondas)
- Trabaja con **gasto por producto, NO ventas.** El breakdown `product_id` de Meta no devuelve
  conversiones por producto (vuelven "Not available"). Las sugerencias salen de **dónde el algoritmo pone
  el dinero**, no de qué convierte.
- **No ve el inventario sin gasto.** El breakdown solo lista productos que recibieron presupuesto; los
  nunca anunciados no aparecen — el skill no puede sugerir "testea esto que nunca corriste".
- Lo que produce son **hipótesis para testear**, no ganadores garantizados. El test confirma; el skill
  solo dice dónde vale la pena mirar.

## Flujo
1. **Traer el gasto por producto** (una sola llamada):
   ```
   ads_get_ad_entities(<AD_ACCOUNT_ID>, level="account", date_preset="last_30d",
     fields=["amount_spent"], breakdowns=["product_id"],
     sort="amount_spent_descending", limit=50)
   ```
   Usa `last_90d` si quieres una base más amplia. `sort` + `limit=50` mantiene la respuesta chica (el resto
   es polvo de gasto). Si aún es enorme, baja `limit` a 30. El `product_id` viene como `"<FBID>, <Nombre>"`
   → separa ambos. Una fila puede venir como `product_id="unknown"`: es **gasto no atribuido**, consérvala.
2. **Normalizar el gasto a número** según la moneda de la cuenta (viene formateado: `"$ 836.793 COP"` →
   `836793`; `"2.513,67 MXN"` → `2513.67`; USD estándar).
3. **Calcular las hipótesis** (haz las sumas/porcentajes con cuidado, no "a ojo" sobre decenas de filas).

## Las 4 hipótesis
- **Cola subfinanciada** — los productos que entre TODOS suman una porción mínima del gasto (por defecto,
  la cola que acumula ≤ **15%** del gasto total, desde el de menor gasto). El algoritmo casi no los corrió;
  agrupados en su propio conjunto con presupuesto propio dejan de competir en desventaja y revelan si hay
  un ganador escondido. **Es la hipótesis más fuerte cuando la inversión está concentrada** (ej. el top 5
  se lleva > **55%** del gasto).
- **Por tipo de producto** — agrupa por el tipo (primera palabra del nombre: "camiseta", "pantalón",
  "bundle"…) y señala el tipo **subexpuesto**, candidato a un conjunto propio.
- **Variantes del mismo producto** — productos que aparecen como varias entradas (tallas/colores con FBID
  distinto). El gasto repartido esconde su fuerza real; conviene testearlos a nivel de **producto padre**.
- **Gasto sin producto identificado** — si el bucket `unknown` supera un **piso de ruido (~3%** del total),
  márcalo: hay que **entender ese gasto antes de testear nada**, porque ensucia toda lectura por producto.
  Por debajo del piso, no alertes.

Umbrales (55% concentración · 15% cola · 3% piso unknown) son **defaults sensatos** — ajústalos si el
usuario pide más o menos sensibilidad.

## Reglas
- **Gasto no es venta.** Un grupo rankeado por gasto NUNCA se llama "más vendidos".
- **Solo lectura:** nunca crees un conjunto. El skill sugiere; el usuario lo crea en Commerce Manager.
- Si el usuario aporta **ventas reales por producto** (Shopify, o el desglose del catálogo en Ads
  Manager), úsalas: con ventas, las hipótesis pasan de "dónde gasta el algoritmo" a "qué rinde de verdad".
  Pero el skill funciona sin eso.
- Presenta las hipótesis con 1–2 frases de lectura si aportan, sin inventar.
