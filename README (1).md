# CSS — Buenas Prácticas aplicadas en VetCare

Este documento explica cada práctica de CSS utilizada en el proyecto, con el fragmento de código real del archivo `styles.css` y la razón detrás de cada decisión.

---

## Índice

1. [Custom Properties — Variables CSS](#1-custom-properties--variables-css)
2. [Reset base y modelo de caja](#2-reset-base-y-modelo-de-caja)
3. [Unidades relativas — rem sobre px](#3-unidades-relativas--rem-sobre-px)
4. [Organización del archivo por secciones](#4-organización-del-archivo-por-secciones)
5. [Mobile First — Responsive Design](#5-mobile-first--responsive-design)
6. [Flexbox para layouts](#6-flexbox-para-layouts)
7. [Selectores específicos y de atributo](#7-selectores-específicos-y-de-atributo)
8. [Pseudoclases para interactividad](#8-pseudoclases-para-interactividad)
9. [Accesibilidad desde CSS](#9-accesibilidad-desde-css)
10. [Transiciones suaves](#10-transiciones-suaves)
11. [Herencia de fuente en formularios](#11-herencia-de-fuente-en-formularios)
12. [Separación de responsabilidades](#12-separación-de-responsabilidades)

---

## 1. Custom Properties — Variables CSS

### Qué son

Las Custom Properties (variables CSS) se declaran con el prefijo `--` dentro de un selector, y se consumen con la función `var()`. Al declararlas en `:root`, quedan disponibles en **todo el documento**.

### Cómo se usaron

```css
:root {
  --color-primario:   #2d6a4f;
  --color-secundario: #f4a261;
  --espaciado:        1rem;

  /* Derivadas */
  --color-fondo:  #f4f7f5;
  --color-texto:  #1c1c1c;
  --color-borde:  #c9d9d1;
  --radio:        0.5rem;
}
```

Y en los componentes:

```css
.form-section h2 {
  color: var(--color-primario);
}

.btn {
  background-color: var(--color-secundario);
  padding: 0.65rem calc(var(--espaciado) * 1.5);
}
```

### Por qué es una buena práctica

- **Consistencia visual:** un solo cambio en `:root` actualiza el color o espaciado en toda la página.
- **Mantenimiento:** si el cliente cambia el color de marca, se edita una sola línea, no 20 reglas dispersas.
- **Legibilidad:** `var(--color-primario)` comunica intención; `#2d6a4f` no dice nada por sí solo.
- **Escalabilidad:** al crecer el proyecto, las variables siguen siendo la fuente de verdad centralizada.

---

## 2. Reset base y modelo de caja

### Qué es

Por defecto, los navegadores aplican márgenes y paddings distintos a cada elemento. El reset los elimina para partir de cero y tener comportamiento predecible.

### Cómo se usó

```css
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}
```

### Explicación de `box-sizing: border-box`

Con el valor por defecto (`content-box`), si defines `width: 200px` y luego agregas `padding: 20px`, el elemento termina midiendo **240px** — el padding se suma por fuera.

Con `border-box`, el padding y el border se incluyen **dentro** del ancho declarado. El elemento siempre mide exactamente los 200px que indicaste.

```
content-box (problema):
  width: 200px + padding: 20px a cada lado = 240px reales

border-box (solución):
  width: 200px → el padding cabe dentro → 200px reales
```

### Por qué es una buena práctica

Es el estándar actual en cualquier proyecto profesional. Sin él, calcular layouts se vuelve impredecible y propenso a errores.

---

## 3. Unidades relativas — rem sobre px

### Qué es

`rem` significa "root em". Es relativo al tamaño de fuente del elemento `html`. Si `html` tiene `font-size: 16px`, entonces `1rem = 16px`.

### Cómo se usó

```css
html {
  font-size: 16px; /* base para todos los cálculos rem */
}

.site-header h1 { font-size: 1.5rem;  }  /* 24px */
.form-section h2 { font-size: 1.25rem; }  /* 20px */
.form-grupo label { font-size: 0.875rem; } /* 14px */
```

El espaciado también usa rem a través de la variable:

```css
--espaciado: 1rem; /* unidad base — todo se deriva de aquí */
```

### Por qué es una buena práctica

- **Accesibilidad:** si el usuario configura un tamaño de texto mayor en su navegador (algo común en personas con baja visión), los `rem` escalan con él. Los `px` fijos ignoran esa preferencia.
- **Proporcionalidad:** todos los tamaños guardan relación entre sí porque parten de la misma base.
- **Mantenimiento:** cambiar `font-size` en `html` reescala toda la tipografía de golpe.

---

## 4. Organización del archivo por secciones

### Qué es

Dividir el CSS en bloques comentados con un índice al inicio del archivo.

### Cómo se usó

```css
/*
   styles.css — VetCare Mini Landing
   Secciones:
     1. Custom Properties (:root)
     2. Reset base
     3. Header y Footer
     4. Sección: Producto
     5. Layout principal
     6. Sección: Formulario
     7. Sección: Panel Custom Properties
*/
```

Cada sección tiene su propio bloque:

```css
/* ================================================
   6. SECCIÓN: FORMULARIO
   Usa --color-primario, --color-secundario,
   --espaciado y --radio de :root.
================================================ */
```

### Por qué es una buena práctica

- Cualquier persona que abra el archivo sabe inmediatamente dónde encontrar cada cosa.
- Facilita el trabajo en equipo: cada persona puede trabajar en una sección sin pisar la otra.
- Reduce el tiempo de búsqueda cuando hay que hacer cambios.

---

## 5. Mobile First — Responsive Design

### Qué es

Escribir los estilos base pensando en pantallas pequeñas (móviles) y luego usar `@media (min-width: ...)` para añadir cambios en pantallas más grandes. Es lo contrario a `@media (max-width: ...)`, que parte del escritorio y "rompe" cosas para móvil.

### Cómo se usó

```css
/* MÓVIL — estilos base, sin media query */
.layout {
  display: flex;
  flex-direction: column; /* 1 columna */
  gap: var(--espaciado);
}

.producto-inner {
  display: flex;
  flex-direction: column; /* 1 columna */
  gap: calc(var(--espaciado) * 1.5);
}

/* DESKTOP — se activa solo cuando hay suficiente espacio */
@media (min-width: 768px) {
  .layout {
    flex-direction: row; /* 2 columnas */
    max-width: 900px;
    margin-inline: auto;
  }

  .producto-inner {
    flex-direction: row; /* 2 columnas */
  }
}
```

### Por qué es una buena práctica

- La mayoría del tráfico web viene de móviles. Diseñar para ellos primero garantiza que la experiencia principal esté bien resuelta.
- El CSS resultante es más limpio: se añaden estilos en lugar de sobrescribirlos.
- Obliga a priorizar el contenido esencial, ya que el espacio en móvil es limitado.

---

## 6. Flexbox para layouts

### Qué es

Flexbox es un sistema de layout de una dimensión (fila o columna) que permite distribuir y alinear elementos de forma flexible dentro de un contenedor.

### Cómo se usó

**Layout de dos columnas (formulario + panel):**

```css
.layout {
  display: flex;
  flex-direction: column; /* mobile */
  gap: var(--espaciado);
}

@media (min-width: 768px) {
  .layout { flex-direction: row; }
  .form-section  { flex: 1.4; } /* ocupa más espacio */
  .props-section { flex: 1;   } /* ocupa menos */
}
```

**Footer pegado al fondo de la página:**

```css
body {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.site-footer {
  margin-top: auto; /* empuja el footer al fondo */
}
```

**Lista de items centrados:**

```css
.props-lista li {
  display: flex;
  align-items: center; /* alineación vertical */
  gap: 0.625rem;
}
```

### Propiedades clave usadas

| Propiedad | Efecto |
|---|---|
| `display: flex` | activa el contexto flex en el contenedor |
| `flex-direction` | define si los hijos van en fila o columna |
| `gap` | espacio entre items sin necesidad de margins |
| `align-items` | alineación en el eje cruzado (vertical si es row) |
| `flex: 1` | el item crece para ocupar el espacio disponible |
| `margin-inline: auto` | centra horizontalmente un bloque con ancho fijo |

### Por qué es una buena práctica

Flexbox reemplaza hacks históricos como floats o tablas para layout. Es más legible, predecible y fácil de mantener.

---

## 7. Selectores específicos y de atributo

### Qué es

CSS ofrece distintos tipos de selectores. Usar el más específico y semántico posible evita sobrescrituras inesperadas y hace el código más claro.

### Cómo se usaron

**Selector de atributo** — apunta a elementos con un atributo concreto:

```css
/* Solo los inputs de tipo texto, no checkboxes ni radios */
.form-grupo input[type="text"],
.form-grupo input[type="email"] {
  width: 100%;
  padding: 0.6rem var(--espaciado);
  border: 1.5px solid var(--color-borde);
}
```

**Selector descendiente** — elemento dentro de un contexto:

```css
/* Solo el h2 dentro de .form-section, no todos los h2 */
.form-section h2 {
  color: var(--color-primario);
}
```

**Selector de clase modificadora** — variante de un componente:

```css
.prop-muestra--text {
  /* variante del componente .prop-muestra */
}
```

### Por qué es una buena práctica

- Los selectores de atributo son más precisos que `input` suelto: evitan aplicar estilos a inputs que no corresponden.
- Los selectores descendientes limitan el alcance de una regla y evitan colisiones con otras partes del proyecto.

---

## 8. Pseudoclases para interactividad

### Qué es

Las pseudoclases son palabras clave que se añaden a un selector y representan un estado especial del elemento.

### Cómo se usaron

**`:hover`** — cuando el cursor está encima:

```css
.btn:hover {
  background-color: #d4845a;
  transform: translateY(-2px); /* sube 2px — efecto flotante */
}
```

**`:active`** — mientras se mantiene el clic:

```css
.btn:active {
  transform: translateY(0); /* vuelve a posición original */
}
```

**`:focus`** — cuando el input está activo:

```css
.form-grupo input:focus {
  outline: none;
  border-color: var(--color-primario);
}
```

**`:placeholder-shown`** — cuando el campo está vacío (placeholder visible):

```css
.form-grupo input:placeholder-shown {
  border-style: dashed; /* indica visualmente que está vacío */
}
```

**`:focus-visible`** — solo cuando el foco viene del teclado (no del mouse):

```css
:focus-visible {
  outline: 3px solid var(--color-secundario);
  outline-offset: 2px;
}
```

### Por qué es una buena práctica

- Añaden retroalimentación visual sin necesitar JavaScript.
- Mejoran la experiencia del usuario: saben qué elemento está activo o listo para interactuar.
- `:focus-visible` en particular es importante: muestra el anillo de foco a usuarios de teclado sin mostrarlo a usuarios de mouse, que ya saben dónde están por el cursor.

---

## 9. Accesibilidad desde CSS

### Qué es

El CSS puede mejorar o dañar la accesibilidad. Estas prácticas aseguran que la página sea usable para personas con discapacidad visual o motora.

### Cómo se aplicó

**Contraste de colores:** el verde `#2d6a4f` sobre blanco y el texto `#1c1c1c` sobre fondo claro cumplen con el ratio mínimo WCAG AA (4.5:1 para texto normal).

**`line-height` generoso:**

```css
body {
  line-height: 1.6; /* ≥ 1.5 mejora lectura para dislexia */
}
```

**`font-family: inherit` en formularios:**

```css
.form-grupo input[type="text"],
.form-grupo input[type="email"] {
  font-family: inherit; /* evita que el navegador use su fuente genérica */
}
```

**Foco visible garantizado:**

```css
:focus-visible {
  outline: 3px solid var(--color-secundario);
  outline-offset: 2px;
}
```

Nunca se usó `outline: none` sin proporcionar una alternativa visible, porque eliminar el foco es una de las fallas de accesibilidad más comunes.

### Por qué es una buena práctica

La accesibilidad no es opcional — es un requisito legal en muchos países y una responsabilidad ética. Además, mejora la experiencia para todos, no solo para personas con discapacidad.

---

## 10. Transiciones suaves

### Qué es

`transition` define cómo cambia una propiedad CSS de un estado a otro, en lugar de hacerlo de forma abrupta.

### Cómo se usó

```css
/* En inputs — el borde cambia suavemente al hacer foco */
.form-grupo input[type="text"],
.form-grupo input[type="email"] {
  transition: border-color 0.2s;
}

/* En el botón — color y posición cambian al hacer hover */
.btn {
  transition: background-color 0.2s, transform 0.15s;
}
```

### Valores elegidos

`0.2s` y `0.15s` son duraciones estándar para micro-interacciones: suficientemente rápidas para no sentirse lentas, suficientemente lentas para ser perceptibles.

### Por qué es una buena práctica

Los cambios abruptos de estado se sienten toscos. Las transiciones suaves comunican calidad y cuidado en el diseño. Son uno de los detalles que diferencian un proyecto terminado de uno a medias.

---

## 11. Herencia de fuente en formularios

### Qué es

Los navegadores aplican sus propias fuentes a los elementos de formulario (`input`, `button`, `textarea`). Si no se corrige explícitamente, el formulario tendrá una fuente diferente al resto de la página.

### Cómo se usó

```css
.form-grupo input[type="text"],
.form-grupo input[type="email"] {
  font-family: inherit; /* hereda la fuente del body */
  font-size: 1rem;
}

.btn {
  font-family: inherit;
  font-size: 1rem;
}
```

### Por qué es una buena práctica

Garantiza coherencia tipográfica en toda la página. Sin esto, el botón o los inputs usarían `system-ui` o `serif` según el navegador y el sistema operativo del usuario.

---

## 12. Separación de responsabilidades

### Qué es

El HTML define la **estructura y el significado**. El CSS define la **presentación visual**. No deben mezclarse: evitar estilos inline (`style="..."`) en el HTML salvo para valores dinámicos que CSS no puede manejar.

### Cómo se aplicó

En el HTML, las muestras de color del panel necesitaban colores específicos para cada variable — algo que CSS no puede resolver sin JavaScript. En ese caso puntual se usó `style=""` de forma justificada:

```html
<span class="prop-muestra" style="background-color: #2d6a4f;" aria-hidden="true"></span>
```

Todo lo demás — layout, tipografía, colores, estados, animaciones — vive exclusivamente en `styles.css`.

### Por qué es una buena práctica

- Mantener el CSS en un archivo separado permite que el navegador lo cachee: en visitas posteriores la página carga más rápido.
- Facilita cambiar el diseño completo sin tocar el HTML.
- Hace el código más fácil de leer, depurar y mantener a largo plazo.

---

## Resumen rápido

| Práctica | Regla clave |
|---|---|
| Custom Properties | Declarar en `:root`, consumir con `var()` |
| Modelo de caja | Siempre `box-sizing: border-box` |
| Unidades | `rem` para fuentes y espaciados, nunca `px` fijo |
| Organización | Índice al inicio + bloques comentados por sección |
| Responsive | Mobile first: estilos base → `@media (min-width: ...)` |
| Layout | Flexbox: `display: flex` + `gap` en lugar de margins |
| Selectores | Usar atributos y descendientes para precisión |
| Interactividad | Pseudoclases `:hover`, `:focus`, `:active`, `:placeholder-shown` |
| Accesibilidad | Contraste, `line-height`, nunca eliminar `outline` |
| Transiciones | `0.15s–0.2s` para micro-interacciones |
| Formularios | `font-family: inherit` en inputs y botones |
| Separación | Todo el diseño en CSS, HTML solo estructura |
