# Motos-HTML-CSS-SCSS

Proyecto de maquetación de la página "Todas las motos" en HTML + SCSS, a partir de un diseño en Figma con 3 vistas fijas de referencia (Desktop 1298px, Tablet 842px, Mobile 468px). He implementado el proyecto para que además se comporte de forma fluida en anchos intermedios, no solo en esas 3 medidas exactas.

## Índice

- [Estructura del proyecto](#estructura-del-proyecto)
- [HTML semántico](#html-semántico)
- [Metodología BEM](#metodología-bem)
- [Custom properties (variables CSS)](#custom-properties-variables-css)
- [Arquitectura SCSS](#arquitectura-scss)
- [Flexbox y Grid](#flexbox-y-grid)
- [Breakpoints y enfoque responsive](#breakpoints-y-enfoque-responsive)
- [Decisiones de accesibilidad](#decisiones-de-accesibilidad)
- [Resultado de WAVE](#resultado-de-wave)
- [Elementos descartados y por qué](#elementos-descartados-y-por-qué)

## Estructura del proyecto

```
src/
  assets/
    icons/
    images/
  scss/
    01-external-reset/
    02-tools/
    03-tokens/
    04-atoms/
    05-molecules/
    06-organisms/
    07-pages/
    08-utils/
index.html
```

He organizado el SCSS siguiendo un patrón por capas (inspirado en ITCSS + Atomic Design), donde cada capa solo puede depender de las anteriores:

1. **01-external-reset**: mi reset de CSS propio (margin/padding, box-sizing, listas, enlaces, imágenes, inputs).
2. **02-tools**: funciones y mixins de Sass reutilizables que he creado (`pixel-to-rem()`, `breakpoint()`). No genera CSS por sí solo.
3. **03-tokens**: custom properties CSS (colores, tipografía, espaciados, tamaños, bordes), que agrupo en un único `:root` mediante mixins.
4. **04-atoms**: componentes mínimos reutilizables que he definido (botón, chip, enlace, input, título).
5. **05-molecules**: agrupaciones pequeñas con sentido propio (card de moto, campo de formulario label+input).
6. **06-organisms**: piezas grandes y reutilizables compuestas por átomos/moléculas (header, footer con newsletter, catálogo con filtros).
7. **07-pages**: composición final específica de esta página concreta (contenedor `.main`).

He añadido un `_index.scss` en cada carpeta que reenvía (`@forward`) sus ficheros internos, de forma que en `app.scss` solo necesito una línea `@use` por capa.

## HTML semántico

He justificado las siguientes decisiones de etiquetado:

- **`<header>` / `<main>` / `<footer>`**: estructura general del documento.
- **`<nav>`** dentro del header, envolviendo el enlace "Ver todas las motos": lo he separado del icono de usuario porque es navegación del sitio, no una acción de usuario.
- **`<nav>`** en el footer, envolviendo la lista de enlaces legales: aplico el mismo criterio.
- **`<article class="card">`** para cada moto: lo he elegido porque es contenido autocontenido y repetible, que tendría sentido incluso distribuido de forma independiente.
- **Jerarquía de headings sin saltos**: `h1` (título de página) → `h2` (título de cada card y título del newsletter). Corregí un salto inicial de `h1` a `h3` que tenía al principio.
- **`alt=""`** en las imágenes de las motos: las he dejado vacías porque son redundantes con el texto "Moto" que ya aparece como `h2` justo debajo, para no generar ruido a un lector de pantalla.
- **`alt` descriptivo** en el resto de iconos (logo, usuario, menú hamburguesa, descuento), ya que aportan información que no está repetida en el texto adyacente.
- **`<section class="newsletter">`** dentro de `<footer>`: decidí no dejar la sección suelta fuera de un contenedor jerárquico; considero el newsletter contenido de pie de página, y cuenta con su propio encabezado (`h2`).
- **La línea divisoria entre newsletter y footer** no la he implementado como `<hr>`: al revisar el diseño en Figma confirmé que en realidad es el `border-top` visual del propio `.footer`, y un `<hr>` representa un cambio temático de contenido, no una línea puramente decorativa.

## Metodología BEM

Aplico BEM (`bloque__elemento--modificador`) en todos los componentes. Además, combino este patrón con "átomo + contexto" para elementos de apariencia reutilizable:

```html
<a href="#" class="link card__link">Ver detalles</a>
<h1 class="title main__title">Todas las motos</h1>
```

La clase de átomo (`.link`, `.title`) resuelve la apariencia visual reutilizable; la clase BEM del bloque (`.card__link`, `.main__title`) la dejo disponible para ajustes de posición/contexto específicos de ese bloque, sin duplicar estilos.

## Custom properties (variables CSS)

He usado custom properties (`--variable`) en lugar de variables Sass para todo lo que representa un valor de diseño (color, tipografía, espaciado, tamaño, borde), porque:

- Sass resuelve sus variables en tiempo de compilación; las custom properties se resuelven en el navegador, así que están disponibles para JavaScript o para un futuro cambio de tema sin recompilar.
- Me permiten centralizar el sistema de diseño en un único lugar (`03-tokens`).

Agrupo todas las custom properties en un único bloque `:root` (mediante mixins de Sass que incluyo en `03-tokens/_index.scss`), en lugar de generar un `:root` distinto por fichero, para que el CSS compilado quede más limpio.

Reservo las variables Sass (`$variable`) solo para lo que necesita el compilador: el mapa de breakpoints y el tamaño base de conversión a rem, en `02-tools`.

## Arquitectura SCSS

- Paso todas las medidas de tipografía, espaciado y tamaño de icono por la función `pixel-to-rem()` (`02-tools/_px-to-rem.scss`), que convierte un valor en píxeles a `rem` en base a 16px, para que la interfaz escale correctamente si el usuario cambia el tamaño de fuente de su navegador.
- **Excepción intencionada**: mantengo los valores de `border-radius`, `border-width` y las alturas/anchos fijos de imágenes en unidades no relativas al tamaño de fuente, ya que corresponden a decisiones puramente visuales (radio de esquina, grosor de borde) que no deben depender de la configuración tipográfica del usuario.
- Uso una escala de espaciados en múltiplos de 8px (`--spacing-8px` a `--spacing-64px`), en lugar de nombres tipo small/medium/large, porque necesitaba más pasos intermedios de los que una escala nominal puede cubrir con claridad. También descarté la numeración secuencial (`--spacing-1`, `--spacing-2`...) a favor de nombrar cada variable directamente con su valor en píxeles, para que el nombre sea autoexplicativo sin tener que abrir el fichero de tokens para saber a cuántos píxeles equivale cada paso.
- Nombro los tamaños de icono de forma descriptiva por rol (`--size-icon-logo`, `--size-icon-user`...) en lugar de numérica, ya que cada valor está atado a un elemento concreto y no forma parte de una escala continua.

## Flexbox y Grid

- Uso **Flexbox** para composiciones en una dimensión: filas de chips, fila del header, fila de precio + enlace, formulario del newsletter.
- Uso **Grid** para el catálogo de cards (`.main__catalog`), porque necesito control bidimensional (filas y columnas) con distinto número de columnas por breakpoint: 1 columna en mobile, 2 en tablet, 3 en desktop, con `repeat(n, 1fr)`.
- Nunca fijo el ancho de las cards manualmente: dejo que lo gestione el propio Grid mediante `1fr`, para no tener que recalcular anchos al cambiar de breakpoint.

## Breakpoints y enfoque responsive

No me he limitado a que el proyecto se muestre correctamente solo en los 3 anchos exactos del diseño (468 / 842 / 1298px); he buscado que se comporte de forma fluida en cualquier ancho intermedio o superior:

- Mi mixin `breakpoint()` (`02-tools/_breakpoint.scss`) usa exclusivamente `min-width`, siguiendo un enfoque **mobile-first**: defino primero el estilo base (mobile) y lo sobrescribo progresivamente a partir de cada punto de corte, en lugar de definir rangos cerrados con `min-width` + `max-width`. Cambié a este enfoque porque el de rangos cerrados me generaba huecos y solapamientos entre breakpoints, además de ser el que recomiendan en proyectos profesionales.
- Para los elementos que en Figma tienen el ancho configurado como "Fill" (por ejemplo, el input del newsletter o el propio formulario), los he implementado con `width: 100%` combinado con `max-width` como tope, en lugar de un ancho fijo en píxeles por breakpoint, para que se adapten de forma continua.
- La imagen de cada card la he resuelto con `aspect-ratio` en lugar de una altura fija en píxeles, para que mantenga siempre la proporción real de la fotografía (385:274) sea cual sea el ancho final de la columna, evitando recortes excesivos con `object-fit: cover` en anchos no contemplados exactamente en el diseño.

## Decisiones de accesibilidad

- Mantengo el `outline` de foco por defecto del navegador en los campos de formulario, en lugar de personalizarlo, para garantizar un contraste suficiente sin tener que hacer validaciones adicionales de color.
- Asocio el `<label>` del campo de email al `<input>` mediante `for`/`id`.
- Uso elementos `<button>` para los chips de filtro, para que sean accesibles por teclado.
- He seleccionado los colores de texto sobre fondo (gris secundario, rojo de descuento, azul de enlace) cuidando el contraste, que después he verificado con WAVE.

## Resultado de WAVE

Lo he ejecutado sobre la página con los estilos activados:

- **0 errores**
- **0 errores de contraste**
- **1 alerta**: "Possible heading" sobre `.main__count` (el contador "8 resultados de motos...").
- **AIM Score: 10/10**
  
  <img width="472" height="653" alt="image" src="https://github.com/user-attachments/assets/8f360883-0423-443f-ad15-59c7ccc4f31d" />


### Justificación de la alerta

WAVE marca `.main__count` como posible heading por su peso visual (negrita, tamaño mayor al texto normal), pero he decidido mantenerlo como elemento `<p>` en lugar de convertirlo en un heading semántico. Su función es mostrar un contador de resultados dinámico (un texto de estado que cambiaría según los filtros aplicados), no un título de sección que organice el contenido del documento para la navegación por encabezados. Al tratarse de una alerta (una revisión sugerida) y no de un error, y teniendo una justificación semántica clara, he optado por mantener el marcado y el estilo visual originales.

## Elementos descartados y por qué

- **`04-atoms/_hr.scss`**: no uso ningún `<hr>` en el HTML; la única línea divisoria del diseño es en realidad el `border-top` del footer.
- **`04-atoms/_icons.scss`**: cada icono vive siempre dentro de un contenedor BEM específico (header o card) sin repetirse en contextos desconectados entre sí, así que aplico los tamaños directamente en el organismo/molécula correspondiente en lugar de crear un átomo genérico.
- **`03-tokens/_zindexs.scss`**: no tengo elementos superpuestos en el diseño (modales, dropdowns, headers con overlap), así que no he necesitado gestionar `z-index`.
- **`03-tokens/_breakpoints.scss`** (como custom properties): las custom properties CSS no son válidas dentro de una `@media`, así que mantengo los valores de los breakpoints únicamente como variables Sass en `02-tools/_breakpoint.scss`.
