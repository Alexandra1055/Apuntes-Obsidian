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

#### Import/exports
- **Clasico**: no esta obsoleto, aun se utiliza, pero hay formas mas modernas de hacerlo
	<script src=".../x.js"></script>
	Esto conlleva 3 problemas importantes:
	- El Orden si importa, dependiendo de como lo coloques afecta a su uso
	- Se vuelve Global, es decir si declaro una variable en una carpeta (ej: views.js) después se podría usar o llamar en el html (esto es un problema) porque el orden haria que si esto esta declarado en otro script de forma distinta cambiaria
	- Sobreescritura: esto quiere decir que se pueden reescribir valores en otros archivos. Ej: pongo en un script const x= 5, y despues en otro script no podria decir const x= 8, porque ya se declaro en otro sitio, y al reves, si son let, cambiare los valores. 
- Moderno: los Scripts se vuelven Modulos, esto hace que ya no sean globales sino locales. Se declaran:
	type = "module"
  Normas de uso:
  - Solo podemos importar los elementos marcados como export
	```js
	import {Persona} from '../model/models.js';
	export function getPersona() {
    return fetch("https://randomuser.me/api/")
        .then(function (respuesta) {
            return respuesta.json();
        })
        .then(function (result) {
  //respuesta = JSON de randomuser -> Persona
                return new Persona(
			        result.results[0].name.first,
			        result.results[0].name.last,
			        result.results[0].email,
			        result.results[0].gender,
			        'NO DNI'
                );
            }
        )
}
	```
  - Solo podemos usar Import en los modulos
Ejemplo:
```js
<script type="module">
        import {getPersona} from '../ex2-import-export/service/services.js';
        import {paintResult} from 'src="../ex2-import-export/view/views.js';
        function init() {
          getPersona().then(function(persona){
                paintResult(persona);
           });
        }
        init();
    </script>
```


HTTP
Cliente ->Servidor
url -> politecnicllevant.cat/user
header
```js
const headers = new Headers({
  "Content-Type": "application/json",
  "x-api-key": "DEMO-API-KEY"
});

var requestOptions = {
  method: 'GET',
  headers: headers,
  redirect: 'follow'
};

fetch("https://api.thecatapi.com/v1/images/search?size=med&mime_types=jpg&format=json&has_breeds=true&order=RANDOM&page=0&limit=1", requestOptions)
  .then(response => response.text())
  .then(result => console.log(result))
  .catch(error => console.log('error', error)); 
```
verbos: GET, POST, PUT, PATCH, DELETE,....

CRUD: 
- C -> POST
- R -> GET
- U -> PUT PATCH
- D -> DELETE

Parametros: GET => url ?x=1&y=2