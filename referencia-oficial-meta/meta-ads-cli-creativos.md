# Contenido de los anuncios (Ad Creatives) — CLI de anuncios de Meta

> Fuente: Documentación oficial de Meta para desarrolladores — Ads CLI.
> URL original (inglés): https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli/ad-creatives
> Actualizado: 29 abr. 2026

En esta guía se cubre cómo crear y administrar contenido de anuncios con la CLI de anuncios, incluidos los anuncios con enlace, los anuncios con vídeo, las publicaciones con foto y la optimización dinámica del contenido.

## Información sobre el contenido de los anuncios

El contenido de un anuncio es un objeto independiente que se puede reutilizar en varios anuncios. El proceso de la CLI de anuncios es el siguiente:

1. Crear contenido del anuncio con `meta ads creative create`
2. Haz referencia al contenido del anuncio cuando crees un anuncio con `meta ads ad create --creative-id <CREATIVE_ID>`.

Cada contenido del anuncio necesita una `--page-id`: la página de Facebook para empresas que actúa como identidad del anuncio. Busca los identificadores de página de empresa con:

```bash
meta ads page list
```

## Contenido de los anuncios estándar

### Anuncios con enlace

Un anuncio con enlace incluye una imagen, un título, un cuerpo de texto y una URL de destino. Este es el formato de anuncio más común.

```bash
meta ads creative create --name "Summer Sale" \
  --page-id <PAGE_ID> \
  --image ./banner.jpg \
  --body "50% off everything!" \
  --title "Shop Now" \
  --link-url https://example.com/sale \
  --description "Limited time offer" \
  --call-to-action SHOP_NOW
```

### Anuncios con vídeo

Un anuncio con vídeo incluye un archivo de vídeo, una imagen en miniatura opcional, un título y una URL de destino.

```bash
meta ads creative create --name "Video Promo" \
  --page-id <PAGE_ID> \
  --video ./promo.mp4 \
  --body "Watch our new collection" \
  --title "New Arrivals" \
  --link-url https://example.com/new
```

### Publicaciones con foto

Una publicación con foto en la página de tu empresa. Se crea cuando no se proporciona ningún `--link-url`.

```bash
meta ads creative create --name "Page Post" \
  --page-id <PAGE_ID> \
  --image ./photo.jpg \
  --body "Check out our latest product!"
```

### Cómo se determina el formato

- `--video` proporcionada → Anuncio con vídeo
- `--link-url` proporcionada (sin vídeo) → Anuncio con enlace
- Ni vídeo ni URL de enlace → Publicación con foto

### Formatos de contenido multimedia admitidos

- **Imágenes:** `.jpg`, `.jpeg`, `.png`, `.gif`, `.bmp`, `.webp`
- **Vídeos:** `.mp4`, `.mov`, `.avi`, `.mkv`, `.wmv`

La CLI de anuncios sube archivos multimedia automáticamente cuando creas o actualizas el contenido de un anuncio.

### Tipos de llamada a la acción

Se utiliza con la opción `--call-to-action`:

`APPLY_NOW`, `BOOK_TRAVEL`, `BUY_NOW`, `CONTACT_US`, `DOWNLOAD`, `GET_OFFER`, `GET_QUOTE`, `LEARN_MORE`, `NO_BUTTON`, `OPEN_LINK`, `SHOP_NOW`, `SIGN_UP`, `SUBSCRIBE`, `WATCH_MORE`

### Ubicaciones de Instagram

Para poner anuncios en circulación en Instagram, pasa el identificador de la cuenta de Instagram:

```bash
meta ads creative create --name "Insta Ad" \
  --page-id <PAGE_ID> \
  --instagram-actor-id <INSTAGRAM_ACCOUNT_ID> \
  --image ./banner.jpg \
  --body "Shop our collection" \
  --link-url https://example.com
```

## Optimización de contenido dinámico (DCO)

La optimización dinámica del contenido te permite proporcionar varias variaciones de imágenes, títulos, texto del cuerpo, descripciones y botones de llamada a la acción. Meta prueba automáticamente distintas combinaciones y optimiza la entrega para mostrar las variantes que tengan mejor rendimiento.

### Requisitos

- `--link-url` para la URL de destino
- Al menos un activo `--images` o `--videos`
- Se utilizan marcas en plural (`--images`, `--titles`, `--bodies`) en lugar de las marcas estándar de un único activo (`--image`, `--video`, `--body`, `--title`).

### Creación del contenido de un anuncio

Utiliza las marcas en plural (`--images`, `--titles`, `--bodies` y otras) y repite cada marca para las variantes adicionales:

```bash
meta ads creative create --name "DCO Test" \
  --page-id <PAGE_ID> \
  --link-url https://example.com \
  --images ./img1.jpg --images ./img2.jpg --images ./img3.jpg \
  --titles "Shop Now" --titles "Learn More" \
  --bodies "50% off everything!" --bodies "Free shipping today!" \
  --descriptions "Limited time offer" --descriptions "While supplies last" \
  --call-to-actions SHOP_NOW --call-to-actions LEARN_MORE
```

### Úsalo con vídeos

```bash
meta ads creative create --name "Video DCO" \
  --page-id <PAGE_ID> \
  --link-url https://example.com \
  --videos ./vid1.mp4 --videos ./vid2.mp4 \
  --titles "New Arrivals" --titles "Watch Now" \
  --bodies "Check out our new collection" --bodies "See what's trending"
```

### Límites

| Tipo de activo | Máximo |
|---|---|
| Imágenes | 10 |
| Vídeos | 10 |
| Títulos | 5 |
| Cuerpos | 5 |
| Descripciones | 5 |
| Llamadas a la acción | 5 |

## Actualizar contenido de anuncios

Actualiza campos específicos de contenido de un anuncio existente. La CLI de anuncios solo cambia los campos que especifiques:

```bash
# Update text
meta ads creative update <CREATIVE_ID> --body "New ad copy" --title "New Headline"

# Replace the image
meta ads creative update <CREATIVE_ID> --image ./new-banner.jpg

# Replace the video
meta ads creative update <CREATIVE_ID> --video ./new-video.mp4

# Change status
meta ads creative update <CREATIVE_ID> --status PAUSED
```

Opciones de actualización disponibles: `--name`, `--body`, `--title`, `--link-url`, `--description`, `--call-to-action`, `--image`, `--video`, `--instagram-actor-id`, `--status`

**Nota:** La API evita que se actualicen algunos campos después de la creación. Si se produce un error durante la actualización, crea contenido nuevo para el anuncio.

## Eliminar contenido de anuncios

**Nota:** No puedes eliminar contenidos de anuncios que usen los anuncios activos. Pausa o elimina los anuncios asociados primero.

```bash
meta ads creative delete <CREATIVE_ID>
meta ads creative delete <CREATIVE_ID> --force    # Skip confirmation
```

## Ejemplo de proceso completo

```bash
# 1. Find your Business Page ID
meta ads page list

# 2. Create an ad creative
meta ads creative create --name "Launch Ad" \
  --page-id <PAGE_ID> \
  --image ./launch-banner.jpg \
  --body "Our new product is here!" \
  --title "Just Launched" \
  --link-url https://example.com/launch \
  --call-to-action SHOP_NOW

# 3. Create an ad with the returned ad creative ID
meta ads ad create <AD_SET_ID> --name "Launch Ad - US" --creative-id <CREATIVE_ID>

# 4. Activate the ad with the returned ad ID
meta ads ad update <AD_ID> --status ACTIVE
```
