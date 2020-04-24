# Diff Util 

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

Supongamos que tenemos la siguiente clase

```kotlin

data class ViewProduct(
    val id: String,
    val name: String,
    val description: String,
    val quantity: Int
)
```

Esta clase representa un producto, que se mostrara en una lista de productos, a los cuales se los podra actualizar su cantidad. 

La implementacion de `DiffUtil.Callback` para una lista de `ViewProduct` seria: 

```kotlin
class ProductDiffUtilCallback(
    private val oldProducts: List<ViewProduct>,
    private val newProducts: List<ViewProduct>
) : DiffUtil.Callback() {

    override fun areItemsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean =
        oldProducts[oldItemPosition].id == newProducts[newItemPosition].id

    override fun getOldListSize(): Int = oldProducts.size

    override fun getNewListSize(): Int = newProducts.size

    override fun areContentsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean =
        oldProducts[oldItemPosition] == newProducts[newItemPosition]

    override fun getChangePayload(oldItemPosition: Int, newItemPosition: Int): Any? {
        val oldItem = oldProducts[oldItemPosition]
        val newItem = newProducts[newItemPosition]

        if (oldItem.quantityInCart != newItem.quantityInCart) {
            return UPDATE_QUANTITY
        }

        return null
    }
}

private const val UPDATE_QUANTITY = 1
```

Los primeros 4 metodos son realmente simples, pero prestemos atencion en particual a `getChangePayload`

```kotlin
override fun getChangePayload(oldItemPosition: Int, newItemPosition: Int): Any? {
        val oldItem = oldProducts[oldItemPosition]
        val newItem = newProducts[newItemPosition]

        if (oldItem.quantityInCart != newItem.quantityInCart) {
            return UPDATE_QUANTITY
        }

        return null
    }
```

Llegados a este punto cuando este metodo se llame significara que tenemos dos elementos que son el mismo, pero su contenido difiere; en nuestro ejemplo solo nos preocupamos por la cantidad de los productos, que finalmente sera la propiedad que queremos actualizar en la vista si esta misma cambia. Si esta cambio simplemente devolvemos un `entero` de lo controlario `null`. 

Luego que tenemos nuestro `DiffUtil.Callback`, ya podemos usarlo en nuestro adaptor, el cual tendra unos pequeños cambios a lo que usalmente acostumbramos a escribir. 

```kotlin
class ProductAdapter(
    private val products: ArrayList<ViewProduct>
) : RecyclerView.Adapter<ProductViewHolder>() {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ProductViewHolder {
        return ProductViewHolder(
            LayoutInflater.from(parent.context).inflate(R.layout.product_item, parent, false)
        )
    }

    override fun getItemCount(): Int = products.size

    override fun onBindViewHolder(holder: ProductViewHolder, position: Int) {
        holder.bind(products[position])
    }

    override fun onBindViewHolder(
        holder: ProductViewHolder,
        position: Int,
        payloads: MutableList<Any>
    ) {
        if (payloads.isEmpty()) {
            onBindViewHolder(holder, position)
        } else {
            for (data in payloads) {
                when (data as Int) {
                    INVALID_PRODUCT_POSITION -> holder.updateProductQuantity(products[position].quantityInCart)
                }
            }
        }
    }

    fun updateProducts(products: List<ViewProduct>) {
        val result = DiffUtil.calculateDiff(ProductDiffUtilCallback(this.products, products))
        result.dispatchUpdatesTo(this)
        this.products.clear()
        this.products.addAll(products)
    }
    
    fun removeProduct(product: ViewProduct) {
        val index = products.indexOf(product)

        if (index == INVALID_PRODUCT_POSITION) {
            return
        }

        products.removeAt(index)
        notifyItemRemoved(index)
    }

    companion object {
        private const val INVALID_PRODUCT_POSITION = 1
    }
}
```

La principal diferencia de este adaptador es que como podemos observar, ahora el metodo **onBindViewHolder** recibira un parametro extra *payloads* que sera la información o el payload que utilizamos como propiedad de retorno en el metodo **getChangePayload** de nuestro DiffUtil.Callback. El resto sera lo msimo a lo que ya estamos acostumbrados al escribir un adaptador para un **RecyclerView** 

```kotlin
   override fun onBindViewHolder(
        holder: ProductViewHolder,
        position: Int,
        payloads: MutableList<Any>
    ) {
        if (payloads.isEmpty()) {
            onBindViewHolder(holder, position)
        } else {
            for (data in payloads) {
                when (data as Int) {
                    INVALID_PRODUCT_POSITION -> holder.updateProductQuantity(products[position].quantityInCart)
                }
            }
        }
    }
```


