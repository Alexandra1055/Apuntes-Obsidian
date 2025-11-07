---
title: Repaso examen JavaScript
tags: [javascript, examen, fundamentos, dom, eventos]
---

# Repaso examen JavaScript

> [!summary] Idea general
> JavaScript es **débilmente tipado**: las variables no fijan su tipo; el tipo puede cambiar dinámicamente.

## Variables y tipos

Tipos principales:

- **String**
- **Number** / **BigInt**
- **Object** / **Symbol**  
  *(Los arrays son objetos, y los objetos pueden contener arrays.)*
- **Boolean**
- **Null** → `const x = null;` *(tiene valor `null`)*
- **Undefined** → `let x;` *(no tiene valor, ni está definido)*

```javascript
var x;        // undefined
var x = null; // null
```

### Comparaciones

- `==` → compara **valores** (conversión de tipo)
- `===` → compara **valor y tipo**

```javascript
3 == "3"   // true
3 === "3"  // false
```

### Extra útil sobre variables

- **var**: ámbito global o de función, se puede **redeclara** y **reasignar**. *Obsoleto en la práctica.*
- **let**: ámbito de **bloque** `{}`, se puede **reasignar** pero no **redeclara** en el mismo bloque.
- **const**: ámbito de **bloque** `{}`, **no** se puede **reasignar**.  
  Si es un **objeto o array**, **sí** puedes cambiar su **contenido interno**.

---

## Operador `%` (módulo)

El operador `%` devuelve el **resto de la división entera**.

```javascript
10 % 3  // 1  -> 10 dividido entre 3 es 3 con resto 1
8 % 2   // 0  -> 8 es múltiplo de 2
5 % 2   // 1
```

**Usos típicos:**

### Saber si un número es par o impar

```javascript
function esPar(n) {
  return n % 2 === 0;
}
console.log(esPar(8)); // true
console.log(esPar(5)); // false
```

### Repartir turnos / tareas cíclicas

```javascript
const personas = ["Ana", "Luis", "Marta"];

for (let dia = 0; dia < 7; dia++) {
  const persona = personas[dia % personas.length];
  console.log("Día " + dia + " le toca a " + persona);
}
```

> [!hint] Explicación
> `dia % personas.length` produce `0,1,2,0,1,2,...`.  
> Es el patrón típico para **tareas cíclicas**.

### Mantener un índice en un rango

```javascript
let index = 0;
const lista = ["rojo", "verde", "azul"];

function siguiente() {
  index = (index + 1) % lista.length;
  console.log(lista[index]);
}

siguiente(); // verde
siguiente(); // azul
siguiente(); // rojo (vuelve al inicio gracias al %)
```

### Repetir acción cada X pasos

```javascript
for (let i = 1; i <= 12; i++) {
  if (i % 4 === 0) {
    console.log("Mensaje especial cada 4 pasos, paso:", i);
  }
}
```

---

## Objetos (básico)

Los objetos en JS son **prototípicos** (basados en `prototype`).  
Para acceder a sus propiedades usamos el **punto** `.`:

```javascript
const persona = {
  nombre: "Alex",
  edad: 25
};

console.log(persona.nombre);
```

---

## Arrays (básico)

Los arrays pueden contener **distintos tipos** (aunque no es recomendable).

```javascript
const i = ["Rosa", "Josep", "Cristian"];
```

**Propiedades y métodos:**

```javascript
i.length;        // longitud del array
const j = i[2];  // acceder al valor de una posición

i.push("Sergio"); // añade elemento al final
i.pop();          // elimina el último elemento
```

### Extra útil sobre arrays

- `shift()` → quita el primer elemento.
- `unshift(valor)` → mete un elemento al principio.
- `splice(pos, cuantos)` → quita o inserta en medio.
- `slice(inicio, fin)` → copia una parte **sin** modificar el original.
- `indexOf(valor)` → posición del valor (o `-1` si no está).

```javascript
const nombres = ["Ana","Luis","Marta"];
console.log(nombres.indexOf("Luis")); // 1
console.log(nombres.indexOf("Pepe")); // -1
```

- `includes(valor)` → `true` si el array contiene ese valor.

```javascript
const permitidos = ["rojo","verde","azul"];

console.log(permitidos.includes("verde")); // true
console.log(permitidos.includes("negro")); // false

if (!permitidos.includes("negro")) {
  console.log("Color no válido");
}
```

### Ordenar e invertir

```javascript
const numeros = [30, 4, 100, 7];
numeros.sort(); 
console.log(numeros); // [100, 30, 4, 7]  (OJO: ordena como texto)

// Para orden numérico real:
numeros.sort((a, b) => a - b);
console.log(numeros); // [4, 7, 30, 100]
```

```javascript
const letras = ["a","b","c"];
letras.reverse();
console.log(letras); // ["c","b","a"]
```

---

## Funciones

Las funciones son **valores**; se pueden **guardar en variables**.

```javascript
const j = function() {
  return "Hola, ¿qué tal?";
};

console.log(j, j()); // j -> función | j() -> resultado
```

**Tipos:**

- Con nombre:

```javascript
function saludar() {
  return "Hola";
}
```

- Anónima:

```javascript
const anonima = function() {
  return "Hola";
};
```

---

## Callbacks

Una **callback** es una función que se pasa como **parámetro** a otra función, para ejecutarse después.

```javascript
function saludar(nombre) {
  console.log("Hola " + nombre);
}

function procesarUsuario(callback) {
  const nombre = "Alex";
  callback(nombre);
}

procesarUsuario(saludar);
```

---

## `setTimeout()` y `setInterval()`

### `setTimeout`
Ejecuta una función **una sola vez** después del tiempo indicado (ms).

```javascript
setTimeout(() => {
  console.log("Han pasado 2 segundos");
}, 2000);
```

### `setInterval`
Ejecuta una función **repetidamente** cada cierto intervalo (ms).

```javascript
const id = setInterval(() => {
  console.log("Esto se repite cada segundo");
}, 1000);

// Para detenerlo:
// clearInterval(id);
```

> [!warning] Importante
> - Si hay un **while infinito**, el navegador no “respira”; los `setTimeout` **no** se ejecutan.
> - Si haces `clearInterval` **antes** de que pase el primer intervalo, **no** se imprime nada.

```javascript
// Ejemplo while infinito: nunca llega a dispararse setTimeout
var x = 0;
while (x === 0) { // true, entra y nunca sale
  setTimeout(function () {
    console.log('Examen');
    x = 1;
  }, 1000);
}
```

```javascript
// clearInterval antes de tiempo: no imprime nada
var x = setInterval(function () {
  console.log('Examen');
}, 1000);
clearInterval(x);
```

> [!note]
> Varios `setTimeout` con el **mismo** tiempo se ejecutan **todos juntos** tras el retraso.

---

## DOM (Document Object Model)

El DOM es un **objeto** (con propiedades y funciones) integrado en los navegadores.  
Permite manipular el **árbol de nodos HTML**.  
**No** forma parte de **Node.js**.

**Estructura básica:**
- El árbol del DOM comienza en `<html>`
  - Tiene dos ramas principales: `<head>` y `<body>`
  - Dentro de `<body>` puede haber `<main>`, `<div>`, `<p>`, etc.
  - Los nodos hoja son los textos o elementos internos.

**Acceso principal:**
- `document` es la **instancia** del DOM.

```javascript
document.querySelector("div");
```

**Métodos antiguos (aún útiles):**
- `getElementById()`
- `getElementsByClassName()` *(HTMLCollection)*

### Modificar contenido

- **`innerHTML`**
  - **get**: devuelve el contenido HTML
  - **set**: reemplaza los nodos

- **Navegación de nodos**: `children` | `parent` | `sibling`

### Modificar estilos desde JS

```javascript
const caja2 = document.querySelector("#caja2");

// Cambios inline:
caja2.style.color = "red";
caja2.style.backgroundColor = "black";
caja2.style.fontSize = "20px";

// Usar en eventos:
const boton = document.querySelector("button");
boton.addEventListener("click", () => {
  caja2.style.border = "2px solid green";
  caja2.style.padding = "10px";
});
```

**Clases CSS con `classList`:**

```javascript
caja2.classList.add("activa");     // añade una clase
caja2.classList.remove("oculta");  // quita una clase
caja2.classList.toggle("roja");    // alterna la clase
```

---

## `addEventListener()`

### Click (`click`)

**Qué dispara `click`:**
- Pulsar y soltar el botón principal del ratón sobre el elemento.
- Tocar con el dedo (táctiles).
- Teclas como **Enter** o **Espacio** en elementos activables (p.ej. `<button>`).

**Dónde se usa:** botones, enlaces, iconos, `div` usados como botón, etc.

```javascript
const btn = document.querySelector("#comprar");

btn.addEventListener("click", (event) => {
  console.log("Has hecho click en el botón");
  // event.target es el elemento donde se hizo click
});
```

### Doble click (`dblclick`)

```javascript
const img = document.querySelector("#foto");

img.addEventListener("dblclick", (event) => {
  console.log("Doble click en la imagen para hacer zoom");
});
```

### Teclado (`keydown` y `keyup`)

- `keydown`: cuando una tecla se **presiona** (se repite si la mantienes).
- `keyup`: cuando **sueltes** la tecla.

> [!tip]
> Para que un `div` reciba teclado debe poder tener **foco** (por ejemplo `tabindex="0"`).

**Propiedades útiles de `KeyboardEvent`:**

- `event.key` → nombre lógico de la tecla (`"a"`, `"Enter"`, `"ArrowUp"`, …)
- `event.code` → **posición física** (`"KeyA"`, `"ArrowUp"`, …)

```javascript
document.addEventListener("keydown", (event) => {
  console.log("Tecla pulsada:", event.key);

  if (event.key === "Escape") {
    console.log("Cerrar modal");
  }

  if (event.key === "ArrowRight") {
    console.log("Pasar a la siguiente imagen");
  }
});

document.addEventListener("keyup", (event) => {
  console.log("Has soltado:", event.key);
});
```

### Escritura en inputs (`input`)

Se dispara **cada vez** que cambia el valor de `<input>`, `<textarea>` o `<select>` por acción del usuario.

```javascript
const nombre = document.querySelector("#nombre");

nombre.addEventListener("input", (event) => {
  console.log("Valor actual:", event.target.value);
});
```

### Cambio confirmado (`change`)

- En `<select>`, cuando eliges otra opción.
- En `<input type="text">`/`<textarea>`, al **perder el foco** tras cambiar.

```javascript
const pais = document.querySelector("#pais");

pais.addEventListener("change", (event) => {
  console.log("País elegido:", event.target.value);
});
```

### Enviar formulario (`submit`)

Se dispara en el **`<form>`** al intentar enviarse.

```javascript
const form = document.querySelector("#loginForm");

form.addEventListener("submit", (event) => {
  event.preventDefault(); // evita recarga

  const user = document.querySelector("#user").value;
  const pass = document.querySelector("#pass").value;

  console.log("Voy a enviar esto por fetch:", user, pass);
  // aquí harías fetch(...) en vez de submit tradicional
});
```

---

## Objetos (forma 2)

```javascript
const x = {
  nom: "Joan",
  cognom: "Galmes",
  nomComplet: function() { // función anónima dentro del objeto
    return this.nom + " " + this.cognom;
  }
};
```

**Funciones constructoras:**

```javascript
function Persona(nom, cognom) {
  this.nom = nom;
  this.cognom = cognom;
}

const joan = new Persona("Joan", "Galmes");
```

> [!note] Extra
> Con `new Persona(...)`, JS crea un objeto que hereda de `Persona.prototype`.  
> Así todas las instancias comparten métodos sin duplicarlos en memoria.

---

## BOM (Browser Object Model)

El BOM son las funcionalidades del **navegador** (no todos lo implementan igual).

- **DOM** → `document`
- **BOM** → `window` *(a veces implícito)*

```javascript
window.alert("Hola");
// equivalente a
alert("Hola");
```

> [!warning] Recomendación
> Evita `alert()`. Mejor un `<div>` con estilos.

**Funcionalidades típicas del BOM:**

- Abrir/cerrar ventanas: `window.open()`
- Temporizadores: `setTimeout()`, `setInterval()`
- Historial de navegación
- Recargar páginas
- Control de tiempo, etc.

### `window.open` y `window.close`

```javascript
// Abrir nueva pestaña/ventana:
const nuevaVentana = window.open("https://google.com", "_blank");

// Popup (si el navegador lo permite):
const popup = window.open(
  "https://example.com",
  "ventanaPequena",
  "width=400,height=300"
);

// Cerrar una ventana abierta por tu script:
popup.close();
```

> [!important] Notas
> - `window.open` devuelve una **referencia** a la nueva ventana/pestaña.  
> - Solo puedes cerrar **ventanas abiertas por tu script**.  
> - Muchos navegadores **bloquean popups** si no hay interacción previa del usuario.

---

## Style con JS (mini-ejemplos)

### Tabla con filas pares e impares (todo inline)

```javascript
// 1) Creo la tabla
const tabla = document.createElement("table");

// estilo base
tabla.style.borderCollapse = "collapse";
tabla.style.fontFamily = "sans-serif";
tabla.style.margin = "20px 0";
tabla.style.minWidth = "300px";
tabla.style.border = "1px solid #999";

// 2) Relleno 10 filas x 2 columnas
for (let i = 1; i <= 10; i++) {
  const tr = document.createElement("tr");

  // alterno color de fondo
  tr.style.backgroundColor = (i % 2 === 0) ? "#f0f0f0" : "#ffffff";

  for (let j = 1; j <= 2; j++) {
    const td = document.createElement("td");
    td.textContent = `Fila ${i}, Col ${j}`;

    // estilo de celda
    td.style.border = "1px solid #999";
    td.style.padding = "8px";

    tr.appendChild(td);
  }

  tabla.appendChild(tr);
}

// 3) Inserto en el body
document.body.appendChild(tabla);
```

### Lista `<ul>` / `<li>` con colores alternos

```javascript
// Creo la lista
const ul = document.createElement("ul");

// estilo base
ul.style.listStyle = "none";
ul.style.padding = "0";
ul.style.margin = "20px 0";
ul.style.fontFamily = "sans-serif";

// Datos
const items = ["Manzana", "Pera", "Plátano", "Melocotón", "Uva"];

items.forEach((texto, index) => {
  const li = document.createElement("li");
  li.textContent = texto;

  li.style.padding = "6px 10px";

  // alterno color según par/impar
  if (index % 2 === 0) {
    li.style.backgroundColor = "#222";
    li.style.color = "white";
  } else {
    li.style.backgroundColor = "#ddd";
    li.style.color = "#000";
  }

  ul.appendChild(li);
});

// Inserto en el body
document.body.appendChild(ul);
```

### Cambiar estilo por defecto de un enlace

```javascript
const link = document.createElement("a");
link.href = "https://ejemplo.com";
link.textContent = "Ir a ejemplo";

// estilos
link.style.textDecoration = "none"; // quita subrayado
link.style.color = "black";         // evita azul
link.style.fontWeight = "bold";
link.style.fontFamily = "sans-serif";

document.body.appendChild(link);
```

### Hover sobre imagen (zoom con eventos)

```javascript
const imgZoom = document.createElement("img");
imgZoom.src = "foto1.jpg";
imgZoom.alt = "Producto";
imgZoom.style.width = "200px";
imgZoom.style.transition = "transform 0.2s ease";

imgZoom.addEventListener("mouseover", () => {
  imgZoom.style.transform = "scale(1.1)";
});

imgZoom.addEventListener("mouseout", () => {
  imgZoom.style.transform = "scale(1)";
});

document.body.appendChild(imgZoom);
```

### Hover: cambiar imagen al pasar el ratón

```javascript
const imgSwap = document.createElement("img");
imgSwap.src = "foto_frontal.jpg";
imgSwap.alt = "Producto frontal";
imgSwap.style.width = "200px";
imgSwap.style.transition = "transform 0.2s ease";

// Rutas
const normalSrc = "foto_frontal.jpg";
const hoverSrc  = "foto_trasera.jpg";

imgSwap.addEventListener("mouseover", () => {
  imgSwap.src = hoverSrc;                 // otra imagen
  imgSwap.style.transform = "scale(1.05)";
});

imgSwap.addEventListener("mouseout", () => {
  imgSwap.src = normalSrc;
  imgSwap.style.transform = "scale(1)";
});

document.body.appendChild(imgSwap);
```

---

> [!tip] Notas rápidas examen
> - Un **while infinito** bloquea el hilo; `setTimeout` no se ejecuta.  
> - `setInterval` repite hasta `clearInterval`.  
> - Varios `setTimeout` con el **mismo** tiempo → se disparan juntos tras el retraso.
