---
layout: post
title:  "Diff Util "
categories: [Android, Kotlin]
tags: [Android, Kotlin,Diff]
---

`Es una clase de utilidad usada para calcular la diferencia entre dos listas y generar una nueva lista con las operaciones de actualización para convertir la primer lista en la segunda.`

Cuando estamos construyendo una aplicación generalmente usamos un `RecyclerView` para mostrar una lista de elementos. Cuando debemos actualizar ésta lista, nosotros mismos escribimos el código para calcular la diferencia entre los elementos que se encuentran en el `RecyclerView` y los que no; estas operaciones pueden ser costosas por no mencionar que además notificar los cambios en el adaptador usando `notifyDataSetChanged()` es mucho más costoso. 

DiffUtil es una herramienta que soluciona este problema y nos permite además de calcular la diferencia entre las dos listas, notificar los cambios al `RecyclerView` de una manera optima. 

Si queremos usar esta utilidad necesitaremos crear una clase que extienda `DiffUtil.Callback`

## Metodos 

`DiffUtil.Callback` como ya mencionamos calculará la diferencia entre dos listas, por lo cual se espera que las clases que hereden de esta, tengan mínimamente dos conjuntos de datos, uno que se conocerá como *newList* y otro como *oldList*

*oldList*: ésta lista es la lista actual de nuestro `RecyclerView` o la lista original de elementos

*newList*: ésta es la nueva lista o la lista con los elementos actualizados

`DiffUtil.Callback` tiene 5 métodos que debemos sobreescribir: 

1. **getOldListSize**: éste método retorna el tamaño de la lista conocida como *oldList*

2. **getNewListSize**: éste método retorna el tamaño de la lista nueva o la que contiene los elementos actualizados

3. **areItemsTheSame**: éste método retorna un booleano que indicará si dos elementos son el mismo elemento, cabe resaltar que no estamos hablando de su contenido sino de la igualdad en cuestión de identificador de elementos. 

4. **areContentsTheSame**: éste método retorna un booleano que indicará si dos elementos con el mismo identificador tienen el mismo contenido o no. 

5. **getChangePayload**: cuando dos elementos tienen el mismo id pero sus contenidos difieren, es decir, si *areItemsTheSame* retorna verdadero pero *areContentsTheSame* retorna falso, éste método se llamará, con el fin de devolver cualquier tipo de objeto como extras para obtener las diferencias entre los elementos y poder ser utilizadas luego dentro del adaptador del `RecyclerView` al cual le llegará dicho objeto.

## Muestrame el codigo

Supongamos que tenemos la siguiente clase:

<script src="https://gist.github.com/oscarg798/b3e37a9b389aa2209d72ad0df417514c.js"></script>

Esta clase representa un producto, que se mostrará en una lista de productos, a los cuales se les podrá actualizar su cantidad.

La implementación de `DiffUtil.Callback` para una lista de `ViewProduct` seria: 

    
<script src="https://gist.github.com/oscarg798/39c7e405138f2479c18ab37304619e0f.js"></script>

Los primeros 4 métodos son realmente simples, pero prestemos atención en partícular a `getChangePayload`

<script src="https://gist.github.com/oscarg798/7f867091d18e4841f137ce5fae774923.js"></script>

Al llegar a este punto cuando este método se llame significará que tenemos dos elementos que son el mismo, pero su contenido es diferente; en nuestro ejemplo solo nos preocuparemos por la cantidad de los productos, que finalmente será la propiedad que queremos actualizar en la vista. Si detectamos un cambio simplemente devolvemos un `entero` de lo contrario `null`. 

Luego que tenemos nuestro`DiffUtil.Callback`, ya podemos usarlo en nuestro adapter, el cual tendrá unas pequeñas diferencias con lo que usualmente estamos acostumbramos a escribir.
<script src="https://gist.github.com/oscarg798/9d617fbaa9a591ac20a4bd0d37afd47b.js"></script>

La principal diferencia de este adaptador es que como podemos observar, ahora el método **onBindViewHolder** recibirá un parámetro extra *payloads* que sera la información o el payload que utilizamos como propiedad de retorno en el método **getChangePayload** de nuestro DiffUtil.Callback, el resto será lo mismo a lo que ya estamos acostumbrados a escribir en un adaptador para un **RecyclerView**
<script src="https://gist.github.com/oscarg798/b2c85f73cb139b521e72d4041f236709.js"></script>
