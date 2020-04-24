---
layout: post
title:  "Diff Util "
categories: [Android, Kotlin]
tags: [Android, Kotlin,Diff]
---

`Es una clase de utlidad usada para calcular la diferencia entre dos listas y generar una nueva lista con las operaciones de actualización para convertir la primer lista en la segunda.`

Cuando estamos contruyendo una aplicación generalmente usamos un `RecyclerView` para mostrar una lista de elementos. Cuando debemos actualizar esta lista nosotros mismos escribimos el codigo para calcular la diferencia entre los elementos que se encuentran en el `RecyclerView` y los que no; estas operaciones puede ser costosas por no mencionar que ademas notificar los cambios en el adaptador usando `notifyDataSetChanged()` es mucho mas costoso. 

DiffUtil es una herramienta que soluciona este problema y nos permite ademas de calcular la diferencia entre las dos listas, notificar los cambios al  `RecyclerView` de una manera optima. 

Si queremos usar esta utilidad necesitaremos crear una clase que extienda `DiffUtil.Callback`

## Metodos 

`DiffUtil.Callback` como ya mencionamos calculara la diferencia entre dos listas, por lo cual se espera que las clases que hereden de esta misma, tenga minimamente dos conjuntos de datos, uno el cual se conocera como *newList* y otro como *oldList*

*oldList*: Esta lista es la lista actual de nuestro `RecyclerView` o la lista original de elementos

*newList*: Esta es la nueva lista o la lista con los elementos actualizados

`DiffUtil.Callback` tiene 5 metodos que debemos sobreescribir: 

1. **getOldListSize**: este metodo retorna el tamaño de la lista conocida como *oldList*

2. **getNewListSize**: Este metodo retorna el tamaño de la lista nueva o la que contiene los elementos actualizados

3. **areItemsTheSame**: Este metodo retorna un boleano que indicara si dos elementos son el mismo elemento, cabe resaltar que no estamos hablando de su contenido sino de la igualdad en cuestion de identificador de elementos. 

4. **areContentsTheSame**: Este metodo retorna un boleano que indicara si dos elementos con el mismo identificador tienen el mismo contenido o no. 

5. **getChangePayload**: Cuando dos elementos tienen el mismo id pero sus contenidos difieren, es decir si *areItemsTheSame* retorna verdadero pero *areContentsTheSame* retorna falso, este metodo se llamara con el fin que devuelve cualquier tipo de objeto como extras con el fin de obtener las diferencias entre los elementos y poder ser utilizadas luego dentro del adaptador del `RecyclerView` al cual le llegara dicho objeto. 

## Muestrame el codigo

Supongamos que tenemos la siguiente clase:

<script src="https://gist.github.com/oscarg798/b3e37a9b389aa2209d72ad0df417514c.js"></script>

Esta clase representa un producto, que se mostrara en una lista de productos, a los cuales se los podra actualizar su cantidad. 

La implementacion de `DiffUtil.Callback` para una lista de `ViewProduct` seria: 

	
<script src="https://gist.github.com/oscarg798/39c7e405138f2479c18ab37304619e0f.js"></script>

Los primeros 4 metodos son realmente simples, pero prestemos atencion en particual a `getChangePayload`

<script src="https://gist.github.com/oscarg798/7f867091d18e4841f137ce5fae774923.js"></script>

Llegados a este punto cuando este metodo se llame significara que tenemos dos elementos que son el mismo, pero su contenido difiere; en nuestro ejemplo solo nos preocupamos por la cantidad de los productos, que finalmente sera la propiedad que queremos actualizar en la vista si esta misma cambia. Si esta cambio simplemente devolvemos un `entero` de lo controlario `null`. 

Luego que tenemos nuestro `DiffUtil.Callback`, ya podemos usarlo en nuestro adaptor, el cual tendra unos pequeños cambios a lo que usalmente acostumbramos a escribir. 

<script src="https://gist.github.com/oscarg798/9d617fbaa9a591ac20a4bd0d37afd47b.js"></script>

La principal diferencia de este adaptador es que como podemos observar, ahora el metodo **onBindViewHolder** recibira un parametro extra *payloads* que sera la información o el payload que utilizamos como propiedad de retorno en el metodo **getChangePayload** de nuestro DiffUtil.Callback. El resto sera lo msimo a lo que ya estamos acostumbrados al escribir un adaptador para un **RecyclerView** 

<script src="https://gist.github.com/oscarg798/b2c85f73cb139b521e72d4041f236709.js"></script>


