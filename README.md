# 🎓 Guía Académica — Ejercicio Largo en Astro 7.3

En este repositorio se encuentra el desarrollo completo de los tres retos del **Ejercicio Largo** del curso de Astro 7.3 con JavaScript.

En lugar de armar ejemplos genéricos con datos de prueba que luego no iba a volver a usar, decidí enfocar todo el trabajo directamente en mi emprendimiento real: **Guía Académica** (un portal de asesoría especializada en tesis de pregrado y maestría, artículos científicos Q1-Q3, normas APA 7, tutorías universitarias de psicología/humanidades y análisis psicométrico y estadístico con RStudio, Jamovi y SPSS). De esta manera, todo lo construido aquí ya funciona como la base principal para mi proyecto final del curso.

---

## 📁 Estructura del Proyecto

```text
/
├── public/
│   └── favicon.svg
├── src/
│   ├── components/
│   │   ├── Header.astro      # Barra superior con navegación dinámica
│   │   ├── Nav.astro         # Menú generado con .map()
│   │   ├── Hero.astro        # Portada principal con banner condicional
│   │   ├── Card.astro        # Tarjeta reutilizable para cada servicio
│   │   ├── Footer.astro      # Pie de página con año dinámico y redes
│   │   └── Menu.astro        # Componente corregido del Reto 3
│   ├── data/
│   │   └── productos.json    # Base de datos local con los servicios y módulos
│   ├── layouts/
│   │   └── Layout.astro      # Molde general con estilos globales (is:global)
│   └── pages/
│       ├── index.astro       # Reto 1: Landing page del emprendimiento
│       └── catalogo.astro    # Reto 2: Catálogo detallado de cursos y sílabos
└── package.json
```

---

## 🚀 Reto 1: Landing Page del Emprendimiento (`index.astro`)

El objetivo de esta primera parte fue dejar de tener todo el código aplastado en un solo archivo y dividir la interfaz en componentes reutilizables dentro de `src/components/`, apoyándonos en una plantilla base en `src/layouts/Layout.astro`.

### ¿Qué se implementó y por qué?

1. **Molde base (`Layout.astro`) y estilos globales:**
   - Creé el archivo `Layout.astro` usando la etiqueta `<slot />`, que funciona como el "hueco" donde se inyecta el contenido de cada página para no repetir el `<head>`, el menú ni el pie de página.
   - Ahí mismo coloqué el bloque `<style is:global>` para asegurar que la fuente y la paleta de colores institucional (azul marino `#013366`, verde menta `#71e7b6` y fondo `#f4f4f4`) se mantengan uniformes al pasar de una página a otra.

2. **Componentes dinámicos con `.map()` (`Header.astro`, `Nav.astro` y `Footer.astro`):**
   - En lugar de escribir cada enlace `<li><a>...</a></li>` a mano, declaré arreglos de objetos en el _Frontmatter_ (entre las tres rayitas `---` que se ejecutan en el servidor) tanto para el menú de navegación como para las redes sociales del pie de página, recorriéndolos con `.map()`.
   - En `Footer.astro` utilicé `new Date().getFullYear()` para calcular el año actual directamente en el servidor al momento de compilar.
   - Cada componente incluye su respectivo comentario privado `{/* ... */}` para documentar el código internamente sin que aparezca en el HTML final del navegador.

3. **Filtrado de servicios disponibles desde JSON (`src/data/productos.json` y `Card.astro`):**
   - Armé el archivo `productos.json` con los servicios reales de **Guía Académica**. Guardé los precios como números puros para poder formatearlos con `.toFixed(2)` dentro de `Card.astro`.
   - Para cumplir con el requisito de **mostrar únicamente los servicios disponibles**, registré 7 servicios activos con `"disponible": true` y un 8vo servicio (un seminario grupal con cupos agotados) con `"disponible": false`.
   - En `index.astro` apliqué `productos.filter((p) => p.disponible)` antes del `.map()`, logrando que la página filtre la información en el servidor y renderice solo los 7 servicios activos.
   - _Nota técnica:_ Como estamos trabajando con JavaScript puro, agregué `// @ts-nocheck` al inicio del _Frontmatter_ para evitar advertencias estrictas de tipos de TypeScript al leer el JSON.

4. **Renderizado condicional con `&&` y 0 KB de JavaScript en `dist/`:**
   - Tanto para el aviso promocional en `Hero.astro` como para la insignia de _"Especialidad / Nuevo"_ en `Card.astro`, utilicé el operador lógico de cortocircuito `&&` (`{nuevo && <span class="insignia">Especialidad</span>}`). Como dentro de las llaves `{}` de Astro solo pueden ir expresiones y no un `if` tradicional, el operador `&&` evalúa la condición y solo dibuja la etiqueta si el valor es `true`.
   - Toda la navegación de la landing se resolvió con enlaces de ancla HTML y CSS (`scroll-behavior: smooth`), logrando que al ejecutar `pnpm astro build` la carpeta `dist/` tenga **0 archivos `.js`**.

---

## 📚 Reto 2: Catálogo de Cursos (`src/pages/catalogo.astro`)

Para el segundo reto, en lugar de inventar datos sueltos desde cero, reutilicé y enriquecí el mismo archivo `src/data/productos.json` agregándole a cada servicio su `nivel`, cantidad de `horas`, la propiedad booleana `destacado`, una `descripcion` con etiquetas HTML y un arreglo anidado de `modulos` con sus respectivos `temas`.

### Puntos clave desarrollados en `catalogo.astro`:

1. **Validación de arreglo vacío:**
   - Se implementó un operador ternario que verifica `cursos.length === 0`. Si no hay cursos disponibles, muestra automáticamente el mensaje `"Pronto más cursos"`.

2. **Etiquetas dinámicas (`<h2>` vs `<h3>`):**
   - Para cumplir la regla de que los cursos destacados usen `<h2>` y los demás `<h3>`, asigné la etiqueta a una variable con inicial mayúscula:
     ```javascript
     const EtiquetaTitulo = curso.destacado ? "h2" : "h3";
     ```
   - Luego la invoqué en la plantilla como `<EtiquetaTitulo>{curso.nombre}</EtiquetaTitulo>`.

3. **Listas anidadas con doble `.map()`:**
   - Cada programa académico tiene varios módulos y cada módulo tiene varios temas específicos. Utilicé un primer `.map()` para recorrer `curso.modulos` dentro de una lista ordenada `<ol>`, y dentro de cada módulo anidamos un segundo `.map()` para recorrer `mod.temas` en una lista `<ul>`.

4. **Ficha técnica con `<dl>` y Fragmentos invisibles (`<>...</>`):**
   - Para mostrar el Nivel, las Horas y el total de Módulos usé una lista de definición `<dl>`. Como el estándar HTML no permite meter etiquetas `<div>` sueltas dentro de un `<dl>` para agrupar elementos en un bucle o bloque, envolví cada par `<dt>` y `<dd>` dentro de fragmentos invisibles (`<>...</>`). Así Astro agrupa los elementos en el código, pero en el navegador el envoltorio desaparece dejando un HTML limpio.

5. **Inyección de HTML seguro con `<Fragment set:html={...} />`:**
   - Como las descripciones del JSON incluyen etiquetas como `<p>`, `<strong>` y `<em>` (para resaltar términos como _Scopus_, _Alfa de Cronbach_ o _APA 7_), por seguridad Astro las imprimiría como texto plano si usáramos llaves normales. Al usar `<Fragment set:html={curso.descripcion} />`, le indicamos a Astro que interprete ese contenido como HTML real sin añadir etiquetas contenedoras extra.

---

## 🔍 Reto 3: Cazador de Errores (`src/components/Menu.astro`)

En la tercera parte analicé el componente `Menu.astro` que venía con errores intencionales por mezclar costumbres de React (JSX) con malas prácticas de HTML, impidiendo que el compilador en Rust de Astro 7 construyera el proyecto.

### Tabla de los 7 errores encontrados y corregidos

|   #   | Código original con error                               | ¿Por qué falla en Astro 7?                                                                                                                        | Corrección aplicada                                                                                                        |
| :---: | :------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------- |
| **1** | `<section className="menu">`                            | Astro utiliza atributos estándar de HTML puro, no `className` de React (JSX).                                                                     | Se cambió a `<section class="menu">`.                                                                                      |
| **2** | `<h2>Menú`                                              | **Regla 1 del compilador:** Toda etiqueta abierta debe cerrarse obligatoriamente o el `build` se detiene.                                         | Se cerró correctamente con `<h2>Menú</h2>`.                                                                                |
| **3** | `{ if (abierto) { <p>Abierto</p> } }`                   | Dentro de las llaves `{}` de la plantilla solo se admiten **expresiones**, nunca sentencias como `if`.                                            | Se reemplazó por evaluación de cortocircuito: `{abierto && <p>Abierto</p>}`.                                               |
| **4** | `<p><ul>...</ul></p>` y dos `<li>` sueltos en `.map()`  | **Regla 2 (Anidamiento válido):** Un párrafo `<p>` no puede contener una lista `<ul>`. Además, el `.map()` devolvía dos elementos sin raíz común. | Se eliminó el `<p>` envolvente y se agruparon los dos `<li>` dentro de `<Fragment>...</Fragment>`.                         |
| **5** | Salto de línea entre `<span>Total:</span>` y `<strong>` | **Regla 3 (Espacios estilo JSX):** Astro comprime los saltos de línea entre etiquetas contiguas, pegando el texto como `"Total:2"`.               | Se inyectó un espacio explícito usando `{" "}` entre ambas etiquetas.                                                      |
| **6** | `<button onClick={pedir}>`                              | La función `pedir()` estaba en el servidor (`---`) y nunca viaja al navegador; además `onClick` es sintaxis de React.                             | Se asignó `id="btn-pedir"` al botón y se manejó el clic desde el cliente con una etiqueta `<script>` y `addEventListener`. |
| **7** | `<img src="/menu.jpg">`                                 | A la etiqueta de imagen le faltaba el atributo obligatorio de accesibilidad `alt` y el autocierre limpio.                                         | Se corrigió a `<img src="/favicon.svg" alt="Menú" />`.                                                                     |

---

## 🛠️ Comandos para ejecutar el proyecto

En este proyecto utilizamos **`pnpm`** como gestor de paquetes:

| Comando              | Descripción                                                             |
| :------------------- | :---------------------------------------------------------------------- |
| `pnpm install`       | Instala todas las dependencias del proyecto.                            |
| `pnpm astro dev`     | Levanta el servidor local de desarrollo en `http://localhost:4321`.     |
| `pnpm astro build`   | Compila la versión estática de producción dentro de la carpeta `dist/`. |
| `pnpm astro preview` | Permite previsualizar localmente el sitio ya compilado en `dist/`.      |
