# Mobile Product Cards Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rediseñar las product cards en mobile (≤640px) para un look full-bleed con overlay inferior compacto, dejando la foto como elemento dominante.

**Architecture:** Todo son cambios CSS-only dentro del breakpoint `@media (max-width: 640px)` existente, más un script de touch swipe vanilla JS agregado antes de `</body>`. El JS ofuscado no se toca. El `.card-body` se reposiciona a `absolute` para superponerse sobre la imagen — no se cambia el HTML generado.

**Tech Stack:** CSS puro (variables CSS ya definidas en `:root`), vanilla JS (touch events). Archivo único: `index.html`.

---

## Archivos

| Archivo | Cambio |
|---|---|
| `index.html` líneas 1587-1596 | Reemplazar bloque `@media (max-width: 640px)` completo |
| `index.html` línea 2490 | Insertar `<script>` con touch swipe antes de `</body>` |

---

## Task 1: CSS mobile — card full-bleed + overlay

**Archivos:**
- Modify: `index.html:1587-1596` — reemplazar el bloque `@media (max-width: 640px)` actual

- [ ] **Step 1: Verificar el bloque actual**

Abrir `index.html` y confirmar que líneas 1587–1596 contienen exactamente:

```css
@media (max-width: 640px) {
  .tienda-inner { padding: 80px 20px; }
  .product-grid { grid-template-columns: repeat(2, 1fr); gap: 12px; }
  .card-nombre { font-size: 22px; }
  .card-price { font-size: 26px; }
  .card-flag { font-size: 56px; }
  .card-wa-btn { padding: 8px 12px; font-size: 9px; }
  .search-sort-row { flex-direction: column; align-items: stretch; }
  .sort-select { width: 100%; }
}
```

- [ ] **Step 2: Reemplazar el bloque completo**

Reemplazar esas 10 líneas con el nuevo bloque:

```css
@media (max-width: 640px) {
  /* Layout general */
  .tienda-inner { padding: 80px 16px; }
  .search-sort-row { flex-direction: column; align-items: stretch; }
  .sort-select { width: 100%; }

  /* Grilla más compacta */
  .product-grid { grid-template-columns: repeat(2, 1fr); gap: 8px; }

  /* Card: sin borde visible, bordes más pequeños */
  .product-card { border-radius: 14px; }

  /* La imagen ya ocupa aspect-ratio 3/4 — sin cambios en .card-img */

  /* card-body se convierte en overlay absoluto sobre la imagen */
  .card-body {
    position: absolute;
    bottom: 0; left: 0; right: 0;
    background: linear-gradient(
      to top,
      rgba(0,0,0,0.97) 0%,
      rgba(0,0,0,0.72) 55%,
      transparent 100%
    );
    padding: 28px 9px 9px;
    z-index: 5;
    flex: none;
  }

  /* Nombre: Barlow Condensed compacto, sin overflow */
  .card-nombre {
    font-size: 15px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    margin-bottom: 1px;
  }

  /* Subtítulo: muy pequeño, muy muted */
  .card-sub {
    font-size: 9px;
    color: rgba(240,237,232,0.42);
    margin-bottom: 5px;
  }

  /* Temporada: ocultar (ya está en card-sub) */
  .card-season { display: none; }

  /* Footer: precio a la izq, botón "+" a la der */
  .card-footer {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 6px;
    margin-top: 0;
  }

  /* Precio: más chico, sigue con --accent */
  .card-price { font-size: 14px; line-height: 1; }

  /* Label de precio: ocultar */
  .card-price-label { display: none; }

  /* Botón WA: ocultar de la card (queda solo en el bottom sheet) */
  .card-wa-btn { display: none; }

  /* Botón "+": circular, compacto, solo ícono */
  .card-add-btn {
    width: 28px;
    height: 28px;
    border-radius: 50%;
    padding: 0;
    gap: 0;
    justify-content: center;
    font-size: 0;       /* oculta el texto, mantiene el SVG */
    box-shadow: 0 2px 10px rgba(198,255,0,0.3);
    flex-shrink: 0;
  }
  .card-add-btn svg {
    width: 14px;
    height: 14px;
  }

  /* Dots: mover de bottom-center a top-right, orientación vertical */
  .slider-dots {
    position: absolute;
    top: 8px; right: 8px;
    bottom: auto;
    left: auto;
    transform: none;
    flex-direction: column;
    gap: 3px;
    z-index: 6;
  }
  .slider-dot { width: 3px; height: 3px; }

  /* Flechas: ocultar en mobile (no funcionan con touch) */
  .slider-arrow { display: none !important; }

  /* Flag emoji: ajustar tamaño */
  .card-flag { font-size: 48px; }
}
```

- [ ] **Step 3: Verificar en DevTools — Mobile**

1. Abrir `index.html` en Chrome
2. DevTools → Toggle Device Toolbar → seleccionar "iPhone 14 Pro" (393px ancho) o "iPhone SE" (375px)
3. Navegar al catálogo de productos
4. Confirmar que:
   - Las cards muestran la foto full-bleed
   - El overlay aparece en la parte inferior con nombre, subtítulo, precio y botón "+"
   - El botón "+" es un círculo limegreen
   - No aparece el botón WA en la card
   - Los dots están arriba a la derecha en vertical
   - No hay flechas de slider

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Rediseña cards mobile: full-bleed overlay, btn+ circular, dots verticales"
```

---

## Task 2: JS — Touch swipe en el slider

**Archivos:**
- Modify: `index.html:2490` — insertar `<script>` antes de `</body>`

- [ ] **Step 1: Verificar el punto de inserción**

Confirmar que la línea 2490 del archivo es una línea en blanco seguida de `</body>` en la línea 2491. El nuevo script va entre el cierre del `</script>` obfuscado (línea 2489) y `</body>` (línea 2491).

- [ ] **Step 2: Insertar el script de swipe**

Añadir justo antes de `</body>` el siguiente bloque:

```html
<script>
(function() {
  var SWIPE_THRESHOLD = 40;

  function attachSwipe(slider) {
    var startX = 0;
    slider.addEventListener('touchstart', function(e) {
      startX = e.touches[0].clientX;
    }, { passive: true });
    slider.addEventListener('touchend', function(e) {
      var dx = e.changedTouches[0].clientX - startX;
      if (Math.abs(dx) < SWIPE_THRESHOLD) return;
      var card = slider.closest('.product-card');
      if (!card) return;
      var btn = dx < 0
        ? card.querySelector('.slider-arrow.next')
        : card.querySelector('.slider-arrow.prev');
      if (btn) btn.click();
    }, { passive: true });
  }

  function initSwipe() {
    document.querySelectorAll('.card-slider').forEach(attachSwipe);
  }

  // Esperar a que el JS ofuscado genere las cards
  var grid = document.getElementById('productGrid');
  if (grid) {
    var observer = new MutationObserver(function() {
      document.querySelectorAll('.card-slider:not([data-swipe])').forEach(function(s) {
        s.setAttribute('data-swipe', '1');
        attachSwipe(s);
      });
    });
    observer.observe(grid, { childList: true, subtree: true });
  }

  // También aplicar a cualquier slider ya presente al cargar
  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', initSwipe);
  } else {
    initSwipe();
  }
})();
</script>
```

- [ ] **Step 3: Verificar swipe en DevTools**

1. En DevTools mobile (iPhone), ir al catálogo
2. Abrir una card que tenga múltiples fotos (dots > 1)
3. Simular swipe: Click + drag horizontal sobre la imagen de la card
4. Confirmar que la imagen cambia al slide siguiente/anterior
5. Confirmar que el dot activo cambia en consecuencia

- [ ] **Step 4: Verificar que el botón "+" sigue funcionando**

1. Tap en el botón "+" circular de cualquier card
2. Confirmar que se abre el bottom sheet de selección de talle
3. Seleccionar talle, confirmar, cerrar
4. Verificar que el ícono del carrito actualiza el contador

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Agrega swipe touch al slider de cards en mobile"
```

---

## Task 3: QA final

- [ ] **Step 1: Verificar desktop intacto**

1. En DevTools, cambiar a vista desktop (1280px+)
2. Confirmar que las cards se ven exactamente igual que antes del cambio
3. Las flechas del slider deben aparecer en hover
4. El botón WA debe aparecer en la card
5. El card-body no debe verse como overlay — debe estar debajo de la imagen

- [ ] **Step 2: Verificar casos edge en mobile**

Probar en mobile (375px):
- Card con 1 sola foto: no deben aparecer dots, no debe haber swipe
- Card con nombre muy largo (ej: "MANCHESTER UNITED"): debe truncar con `…`
- Card con bandera emoji (sin imagen real): el emoji debe centrarse en la foto
- Tap en la foto (no en el "+"): debe abrir el lightbox

- [ ] **Step 3: Commit final de QA**

Si no hubo cambios, no hay nada que commitear. Si se corrigió algo durante el QA:

```bash
git add index.html
git commit -m "Fix QA: ajustes menores post-rediseño cards mobile"
```
