Las promesas se hacen en un tiempo determinado, NO son SINCRONAS, pueden cumplirse o no, por lo que se usan comandos como:
-  .then : resolve (SCOPE) /reject 
- .catch

![[Pasted image 20251107123053.png]]

Ejemplo muy rapido: (nunca se usara asi)

```javascript
 let global = 10;
        function getRndInteger(min, max) {
            return Math.floor(Math.random() * (max - min)) + min;
        }
        function prueba(time) {
            return new Promise(function (resolve, reject) {
                setTimeout(function () {
                    const r = getRndInteger(1, 10);
                    global = 0;
                    if (r % 2 === 0) {
                        resolve("Hola");
                    } else {
                        reject("Adios");
                    }
                }, time)
            }).then(function (resultat) {
                console.log("Resultat OK: ", resultat);
            }).catch(function (resultat) {
                console.log("Resultat KO: ", resultat);
            });
        }
        prueba(5000);
        prueba(1000);
        prueba(7000);
        prueba(2000);
        console.log(global);
```

Las promesas siempre se deben resolver

## API -> FETCH

#### fetch("url")
A los fetch les pasamos una url, esta nos devolverá una Promesa, y su resultado, que tambien sera una promesa, sera en formato JSON() (o text(), o blob()), se conocen como RAW. 

```javascript
function init(){
	const peticion = fetch("https://randomuser.me/api/")
	    .then(function(respuesta){
             return respuesta.json();
         })
        .then(function(respuesta){
             console.log(respuesta);
         })
   }
  init();
```
