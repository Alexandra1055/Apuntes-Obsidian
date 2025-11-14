
```javascript
function(x){
(x)=> //2 opciones
//1) valor -> return valor
//2) {}
}

//ejemplo
fetch(url)
//opcion 1
.then((x) => x.json())
//opcion 2
.then((x) =>{ ... return x.json(); })

//si no pongo return no podria poner otro .then despues
```

Opcion 1: 
Si no ponemos { implica que despues de la arrow => lo que pongamos es un **return** 

Opcion 2:
En el segundo si ponemos { podemos o no hacer return 


### Botón 1 → función normal
 ```js
//Regular -> INSTANCIA
document.querySelector('#boton1').addEventListener('click', function(){
    console.log(this);
    this.style.backgroundColor='red';
})

 ```
- Usas `function(){ ... }`, una función **normal**.
    
- En los `addEventListener`, cuando usas función normal, **`this` pasa a ser el elemento que dispara el evento**.
    
- En este caso, al hacer click en `#boton1`, dentro de la función:
    
    - `this` === el botón con id `boton1`.
        
    - `console.log(this)` muestra: `<button id="boton1">Click boton 1</button>`
        
    - `this.style.backgroundColor = 'red'` → cambia el fondo de **ese botón** a rojo.
    

Todo bien, comportamiento “clásico” de `this` en manejadores de eventos.

### Botón 2 → arrow function

```js
//ARROW -> DEFINICION
document.querySelector('#boton2').addEventListener('click', ()=>{
    console.log(this);
    this.style.backgroundColor='blue';
})

```
Con las arrow functions pasa algo MUY importante:

> Una arrow **no tiene su propio `this`**.  
> Hereda el `this` del contexto donde fue definida.

En tu código, esa arrow está definida directamente dentro del `<script>` del documento, es decir, en el **ámbito global**.

- En ese contexto global, `this` suele ser `window` (o `undefined` si estás en módulo/strict, pero en un script normal es `window`).
    
- Por tanto, dentro del callback del botón 2:
    
    - `console.log(this)` muestra: `window` (no el botón).
        
    - `this.style` es `window.style`, que **no existe**.
        
    - Resultado: no cambia el color del botón y probablemente verás un error del tipo  
        `TypeError: Cannot set property 'backgroundColor' of undefined`  
        o simplemente no pasa nada visible.
        

Por eso:

- Con función normal → `this` apunta al botón → funciona.
    
- Con arrow function → `this` NO apunta al botón → NO funciona como esperas.
    

---

Una forma correcta de hacerlo con arrow sería **no usar `this`**, sino el propio `event.target`:

```js
document.querySelector('#boton2').addEventListener('click', (event)=>{
    console.log(event.target); // el botón
    event.target.style.backgroundColor = 'blue';
})

```
Así no dependes de `this` y funciona igual.
