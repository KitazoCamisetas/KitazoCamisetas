# Mobile Product Cards — Rediseño

**Fecha:** 2026-05-29
**Scope:** Solo mobile (≤640px). Desktop permanece igual.

## Contexto

El catálogo actual en mobile usa 2 columnas con cards verticales (imagen 3:4 + body con texto + footer precio/botón). El diseño actual hace que el texto del footer quede apretado y los botones sean pequeños. El cliente quiere que las fotos sean el elemento dominante.

## Decisiones de diseño

| Decisión | Elección |
|---|---|
| Estructura de card | Full-bleed: imagen ocupa toda la card, overlay en borde inferior |
| Grilla mobile | 2 columnas, 8px de gap |
| Aspect ratio | 3:4 (sin cambio) |
| Overlay contenido | Badge + dots + nombre + subtítulo + precio + botón "+" |
| CTA primario en card | Botón "+" circular limegreen (abre bottom sheet de talle) |
| Indicador de fotos | Dots verticales top-right (reemplazan flechas hover-only) |
| Touch | Swipe horizontal en imagen para navegar fotos |

## Especificación del componente

### Estructura HTML (sin cambios)

La card ya se genera desde JS ofuscado. Los cambios son **solo CSS** dentro del breakpoint `@media (max-width: 640px)`.

### Overlay

```
[ Badge top-left ]           [ Dots top-right ]
                                              ·
  ─ gradiente negro de abajo hacia arriba ─  ·
                                              ○  ← dot activo
  ARGENTINA          ← nombre: Barlow 900 15px
  Home · 2026        ← subtítulo: DM Sans 9px, opacity 45%
  $2490    [+]       ← precio 14px azul + botón "+" limegreen 26px
```

### Tokens CSS nuevos (mobile only)

```css
@media (max-width: 640px) {
  /* Grid */
  .product-grid { gap: 8px; }

  /* Card full-bleed */
  .product-card { border-radius: 14px; }

  /* Imagen ocupa todo */
  .card-img { aspect-ratio: 3/4; }

  /* .card-body se convierte en el overlay — posicionado sobre la imagen */
  .card-body {
    position: absolute;
    bottom: 0; left: 0; right: 0;
    background: linear-gradient(to top, rgba(0,0,0,0.97) 0%, rgba(0,0,0,0.75) 55%, transparent 100%);
    padding: 24px 9px 9px;
    z-index: 5;
  }

  /* Nombre en overlay */
  .card-nombre {
    font-size: 15px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  /* Subtítulo en overlay */
  .card-sub { font-size: 9px; color: rgba(240,237,232,0.45); margin-bottom: 6px; }

  /* Footer overlay */
  .card-footer {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  /* Precio */
  .card-price { font-size: 14px; }

  /* Botón "+" — reemplaza el botón rectangular actual */
  .card-add-btn {
    width: 26px; height: 26px;
    border-radius: 50%;
    padding: 0;
    font-size: 18px;
    justify-content: center;
    box-shadow: 0 2px 8px rgba(198,255,0,0.3);
  }

  /* Dots verticales (indicador de fotos) — reemplazan flechas */
  .slider-dots {
    position: absolute;
    top: 8px; right: 8px;
    bottom: auto;
    left: auto;
    transform: none;
    flex-direction: column;
    gap: 3px;
  }
  .slider-dot { width: 3px; height: 3px; }

  /* Ocultar flechas en mobile (no funcionan con touch) */
  .slider-arrow { display: none; }

  /* Botón WA oculto en la card (solo en el bottom sheet) */
  .card-wa-btn { display: none; }
}
```

### Touch — swipe en imagen

El slider actual usa flechas (`.slider-arrow`) que son hover-only → inútiles en touch. Se agrega un listener `touchstart`/`touchend` para detectar swipe horizontal y avanzar/retroceder slides.

Threshold: 40px de desplazamiento horizontal para activar el cambio.

### Qué NO cambia

- Bottom sheet selector de talle (`.selector-modal`) — sin cambios
- Lightbox al tocar la imagen — sin cambios
- Carrito flotante — sin cambios
- Todas las cards en desktop (≥641px) — sin cambios

## Archivos a modificar

| Archivo | Tipo de cambio |
|---|---|
| `index.html` | CSS dentro de `@media (max-width: 640px)` |
| `index.html` | JS: agregar touch listeners al slider (función existente) |
| `index.html` | JS: inyectar `.card-overlay-mobile` con nombre+sub+footer en cada card generada (requiere leer la función de render, que está ofuscada) |

## Riesgo: JS ofuscado

El HTML de las cards se genera desde JS ofuscado (línea 2489). Si el overlay mobile necesita elementos HTML nuevos (`.card-overlay-mobile`), hay dos opciones:

**Enfoque adoptado (solo CSS):** `.card-body` pasa a `position: absolute; bottom: 0` con gradiente como fondo. `.card-season` se oculta (su info ya está en `.card-sub`). No se toca el JS ofuscado.
