# Landing 1.0 — Chiva77

Landing page estática optimizada para redirección instantánea a WhatsApp. Sin backend, sin APIs, sin base de datos.

---

## Arquitectura

```
Usuario abre la landing
  → Pixel PageView (diferido, no bloquea)
  → Click botón
    → Número seleccionado (round-robin local)
    → Promo code generado (UUID único)
    → Pixel Contact enriquecido (no bloquea)
    → Redirect instantáneo a wa.me
```

## Archivos

| Archivo | Descripción |
|---------|-------------|
| `index.html` | HTML + CSS crítico inline + Meta Pixel diferido + JavaScript inline (toda la lógica) |
| `styles.css` | Estilos completos, cargado de forma asíncrona (`media="print" onload`) |
| `imagenes/` | `fondo.avif`, `logo.png`, `whatsapp.png`, `favicon.ico` |

## Funcionalidades

### Meta Pixel (diferido)

- Se crea un stub mínimo de `fbq()` en el `<head>` que solo encola llamadas.
- El script real (`fbevents.js`) se carga en `setTimeout(0)` — después del primer pintado.
- Dispara `PageView` al cargar y `Contact` al hacer click.
- No bloquea la carga visual ni la interacción.

### Advanced Matching (Pixel)

Tanto en `init` como en `Contact`, se envían datos enriquecidos si están disponibles vía query params:

| Parámetro | Fuente | Normalización |
|-----------|--------|---------------|
| `em` | `?em=` o `?email=` | `trim().toLowerCase()` |
| `ph` | `?ph=` o `?phone=` | Solo dígitos, prefijo `54` si tiene 10 dígitos |
| `fn` | `?fn=` | `trim()` |
| `ln` | `?ln=` | `trim()` |
| `external_id` | `localStorage` | UUID generado en la primera visita, persiste |
| `country` | Hardcodeado | `"AR"` |

### Round-Robin de números

- Los números están hardcodeados en el array `FIXED_PHONES`.
- Se rotan usando un índice guardado en `localStorage` (key: `wa_rr_index_fixed`).
- Funciona con 1 número o con varios.
- La selección es síncrona e instantánea.

### Promo Code

- Formato: `{LANDING_TAG}-{uuid12}` (ej: `CH1.0-3a8f2bc91d4e`).
- Se genera al momento del click.
- Se incluye al final del mensaje de WhatsApp.
- Permite rastrear cada click individual.

### Mensaje aleatorio

- 20 variantes de mensaje pre-definidas.
- Se selecciona una al azar en cada click.
- El promo code se concatena al final.

### Anti doble-click

- Flag `window.__waInFlight` que se activa al hacer click.
- Se resetea al volver a la página (`pageshow`) o al cambiar la visibilidad (`visibilitychange`).

### Optimizaciones de carga

| Técnica | Implementación |
|---------|----------------|
| Critical CSS inline | Estilos del primer pintado en `<style>` del `<head>` |
| CSS async | `styles.css` con `media="print" onload="this.media='all'"` |
| Preconnect | `connect.facebook.net` (preconnect + dns-prefetch) |
| Preload imagen | `fondo.avif` con `fetchpriority="high"` |
| Pixel diferido | `setTimeout(0)` para cargar `fbevents.js` |

### Accesibilidad y UX

- `aria-label` en el botón.
- `:active` feedback táctil (`transform: scale(.97)`).
- `:focus-visible` para navegación por teclado.
- `prefers-reduced-motion` para desactivar animaciones.
- `text-wrap: balance` en el título.
- `text-rendering: optimizeLegibility` en el body.

## Configuración (editar en `index.html`)

### Datos del cliente

```javascript
const LANDING_CONFIG = {
  BRAND_NAME: "DON SOCIAL",        // Nombre de la marca
  MODE: "ads",                      // "ads" o "normal"
  PROMO: {
    ENABLED: true,                  // Habilitar promo codes
    LANDING_TAG: "CH1.0"            // Prefijo del promo code
  }
};
```

### Números de WhatsApp

```javascript
const FIXED_PHONES = [
  "5493516768842"       // Agregar más números para round-robin
];
```

### Pixel ID

Buscar y reemplazar en dos lugares:
1. `fbq("init", "1459000075661483", {...})` — en el `<script>` del `<head>`.
2. `<noscript><img ... src="https://www.facebook.com/tr?id=1459000075661483..."/>` — fallback.

### Textos e imágenes

- `<title>` — título del tab del navegador.
- `.title`, `.subtitle`, `.description` — textos visibles.
- `imagenes/` — fondo, logo, ícono de WhatsApp, favicon.

## Deploy

Vercel (estático). No requiere variables de entorno ni configuración de servidor.

```bash
git add -A && git commit -m "cambios" && git push
```

El deploy se actualiza automáticamente.

## Qué NO hace esta landing

- No envía datos a Google Sheets.
- No envía eventos server-side (CAPI).
- No detecta geolocalización.
- No tiene backend ni APIs serverless.
- No necesita Redis, cron jobs ni variables de entorno.
