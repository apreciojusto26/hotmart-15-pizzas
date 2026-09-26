# Italy Pizza · Pizzas caseras fáciles

Página de ventas del ebook en Hotmart (venta directa: somos productores).

- **`/pizzas-caseras-faciles`** Página de ventas: masa universal + 15 recetas + 2 focaccias sorpresa, con botón al checkout de Hotmart.
- **`/`** Redirige a la página de ventas (`redirects` en `astro.config.mjs`).

### Páginas de agradecimiento (Hotmart → Página externa)

Plantilla común en `src/components/ThankYou.astro`; son `noindex` y no están en el sitemap.

| Campo en Hotmart                               | URL                                              |
| ---------------------------------------------- | ------------------------------------------------ |
| URL para Compras Aprobadas                     | `https://recetas.italypizza.es/gracias`          |
| URL para Compras a la espera de pago           | `https://recetas.italypizza.es/gracias/pago-pendiente` |
| URL para Compras a la Espera del Análisis de Crédito | `https://recetas.italypizza.es/gracias/en-analisis` |

### Captura desactivada (se conserva el código)

La página de captura (`src/pages/_captura.astro`) y el endpoint (`src/pages/api/_lead.ts`) **no se publican**: Astro ignora los archivos de `src/pages/` que empiezan por `_`. Para reactivar el embudo con lead magnet:

1. Renombrar `_captura.astro` → `index.astro` y `api/_lead.ts` → `api/lead.ts`.
2. Borrar la redirección de `/` en `astro.config.mjs` y volver a añadir `'/'` en `sitemap.xml.ts`.
3. Configurar las variables de Systeme.io (ver abajo).

Flujo cuando la captura está activa:

```
Captura (popup, solo email)
  → POST /api/lead
  → Systeme.io: crea/busca el contacto y le asigna la etiqueta "Lead - Masa universal"
  → Workflow de Systeme.io: envía el PDF y los emails
  → Redirección a /pizzas-caseras-faciles
  → Checkout de Hotmart
```

## Stack

Astro · React · Tailwind CSS v4 · pnpm · adaptador de Vercel (solo para `/api/lead`; el resto del sitio es estático)

## Comandos

```bash
pnpm install   # instalar dependencias
pnpm dev       # servidor local en http://localhost:4321
pnpm build     # genera el sitio estático en dist/
pnpm preview   # previsualiza el build
```

## Estructura

```
src/
  components/CapturePage.tsx        Captura + popup del formulario (sin usar mientras la captura esté desactivada)
  components/Analytics.astro        Google Analytics 4 (solo producción)
  pages/
    api/_lead.ts                    Endpoint Systeme.io (desactivado por el "_")
    _captura.astro                  Página de captura (desactivada por el "_")
    pizzas-caseras-faciles.astro    Ruta de ventas (SEO incluido)
    404.astro                       Página de error
    robots.txt.ts · sitemap.xml.ts  robots y sitemap
  styles/global.css                 Colores, fuentes y animaciones (Tailwind @theme)
public/
  assets/                           Logo
  ventas/                           Imágenes de la página de ventas (WebP) y OG
  favicon.*                         Favicon (el horno del logo)
assets/                             Originales sin optimizar (solo en local, ignorados por git)
```

## Qué editar

| Quiero cambiar…                   | Dónde                                                          |
| --------------------------------- | -------------------------------------------------------------- |
| Link de compra de Hotmart         | `CHECKOUT_URL` al inicio de `pizzas-caseras-faciles.astro`     |
| Precio                            | `PRICE_N` en el mismo archivo                                  |
| Botón de WhatsApp del banner      | `WHATSAPP_NUMBER` en el mismo archivo (vacío = sin botón)      |
| Etiqueta que arranca el workflow  | `SYSTEME_IO_LEAD_TAG_ID` en `.env` (y en el hosting)           |
| A dónde redirige tras el registro | `SALES_PAGE_URL` en `CapturePage.tsx`                          |
| Textos y SEO de ventas            | `title` y `description` al inicio de `pizzas-caseras-faciles.astro` |
| Google Analytics                  | `GA_ID` en `src/components/Analytics.astro` (vacío = sin GA)   |
| Colores de la marca               | `@theme` en `src/styles/global.css`                            |

> No hay precio ancla ni valores tachados: solo se muestra el precio real de Hotmart. Si algún día se añaden, deben ser precios de referencia reales.

## Variables de entorno

Solo hacen falta si reactivas la captura. Copia `.env.example` a `.env` y rellena:

| Variable                 | Qué es                                                                |
| ------------------------ | --------------------------------------------------------------------- |
| `SYSTEME_IO_API_KEY`     | Clave de API de Systeme.io                                            |
| `SYSTEME_IO_LEAD_TAG_ID` | ID numérico de la etiqueta "Lead - Masa universal" (arranca workflow) |
| `SITE_URL` (opcional)    | Dominio final, para canonical, `og:url` y sitemap                     |

Son secretos de servidor (`astro:env`): nunca llegan al navegador. **Añádelas también en el panel de tu hosting**, o el endpoint fallará en producción.

## SEO y dominio

Para activar canonical, `og:url`, datos estructurados y el sitemap con URLs absolutas, define el dominio al construir:

```bash
SITE_URL=https://tudominio.com pnpm build
```

## Notas

- El PDF gratuito (masa universal) se envía desde el workflow de Systeme.io (se dispara al asignar la etiqueta), no desde este proyecto.
- Si alguien se registra dos veces con el mismo email, no se repite la etiqueta y por tanto no se reenvía el PDF.
- `/api/lead` incluye validación de email, comprobación de origen, un campo honeypot anti-bots y tiempo límite de 8 s hacia Systeme.io. No tiene límite de peticiones por IP; si recibes spam, actívalo en el hosting (p. ej. Vercel Firewall).
- Textos, precio, bio del autor y FAQ salen de la página actual de Hotmart del producto.
