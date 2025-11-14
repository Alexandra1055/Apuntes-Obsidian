### BD
Una modificacion es un conjunto de transacciones :
Ejemplo:
Hacer una transferencia bancaria es una transacción que se tiene que ejecutar si o si todo, no se puede quedar a medias
 1- manda el dinero, 2- lo resta, 3- se lo envia a la otra persona, 4- Manda los mensajes,....
 Si a mitad del camino algo falla ya no se puede seguir, porque entonces se restaría el dinero pero no lo recibiría la otra persona
Entonces esto empieza con begin, persist y commit. 

Ej en codigo;
```java
EntityManager em = ConnectionManager.getEntityManager();

try{
	em.getTransaction().begin;
	em.persist(newMovie);
	em.getTransaction().commit();
}catch (exceptions)....
```

>Otra variacion del persist es el merge pero tiene diferencias. 

Patrones DAO:
Controller <->Service <->DAO <->BD

Sirve para separar las responsabilidades, que se encarga de que los datos que manda la BD y llegan al Service se implementen segun lo que se pida

DTO (Data Transfer Object)
Esto son para los elementos que queremos que el cliente vea, y los que no pero estan en la clase, por ejemplo la ID, la fecha y hora (timestamp), que no aparezcan. Entonces esto hace de capa, para evitar que al Cliente le lleguen los campos que si necesita. 

ejemplo:
```java
public class ShowMovieDTO {  
  
    protected String title;  
	protected String description;  
	protected int year;  
  
}
```

Entonces tenemos que, el DAO controla que datos mandamos de la BD al Service, y el DTO despues filtra que datos del Service llegan al Controller.