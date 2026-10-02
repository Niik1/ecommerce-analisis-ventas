# 🛒·Análisis de Ventas de un E-commerce 

> **Resumen en una frase:** Analicé [X] pedidos de [empresa/dataset] entre [año] y [año] para identificar [problema] y recomendar [acción], lo que podría [impacto estimado, ej. reducir 12% los retrasos de entrega].

- Ejemplo: "Analicé 100 mil pedidos de Olist (Brasil, 2016-2018) para entender por qué cae la satisfacción del cliente y recomendar mejoras logísticas que podrían reducir 15% las malas reseñas." 
> ⚠️ **Nota sobre los datos:** Este proyecto está inspirado en una experiencia laboral real, 
> pero todos los datos (precios, nombres de clientes, cifras de ventas) son sintéticos 
> y fueron generados para fines de portafolio, sin vulnerar ninguna información 
> confidencial de la empresa original.

## 🔗 Enlaces rápidos

| 📊 Dashboard | 🎥 Video (2-3 min) |
|---|---|
| [Ver Dashboard Interactivo](URL) | [Ver Video Resumen](URL) |

![Vista previa del dashboard](dashboard/captura_principal.png)

---

## 1. Problema de negocio

**Contexto:**  Este proyecto está inspirado en mi experiencia trabajando para una empresa retail/e-commerce peruana dedicada a la venta de productos gaming con presencia en varios puntos de venta. Por motivos de confidencialidad, los datos reales de la empresa no pueden ser utilizados ni publicados: el dataset que se presenta en este repositorio es sintético, generado a partir de una estructura en consolidado, no todos los archivos reales, categorías de productos, pero con cifras, precios y datos de clientes ficticios. 

**Preguntas que quería responder:**
1. ¿Qué sucursal genera más ventas totales y cuál menos?
2. ¿La diferencia es marginal o hay una brecha grande?
3. ¿Coincide la sucursal líder en ventas con la que tiene más órdenes, o vende menos órdenes pero de mayor valor?
4. ¿Qué categoría tiene el ticket promedio más alto por línea?}
5. ¿Qué categoría mueve más volumen pero factura menos?
6. ¿Qué % de clientes compró más de una vez?
7. ¿Ese grupo de clientes recurrentes representa una porción desproporcionada de las ventas?
8. ¿Qué % total de pedidos se cancela?
9. ¿La tasa de cancelación es pareja entre sucursales, o alguna se aleja del resto?
10. ¿Las cancelaciones se concentran en alguna categoría en particular?

---

## 2. Hallazgos principales


- **La Caída oculta en Mall Santa Anita** aunque el acumulado del 2024 y 2025 la muestra es solo de 4 puntos por debajo de la líder, al desagregar por año se identificó una caída sostenida durante 2025 de S/ 47 mil en diciembre del 2024 a un mínimo de S/ 9 mil en septiembre del 2025, causada por un incremento en cancelaciones de 5.6% → 9.4% concentrado en la categoría Consolas con el 18.8% de cancelación, mientras el resto de la red mejoraba su tasa en el mismo periodo.
- **Las Consolas concentran el valor, mientras que Accesorios concentra el volumen.** Consolas tiene el ticket promedio más alto de S/ 2,098, es 6 veces mas que el de Videojuegos y genera 46% de la facturación total, mientras Accesorios es la categoría de mayor rotación de 3,527 unidades con el ticket más bajo, confirmando que el negocio depende fuertemente de una sola categoría para sus ingresos.
- **El 36% de los clientes genera el 63.5% de las ventas.** los clientes recurrentes que realizaron 2+ compras representan poco más de un tercio de la base, pero casi dos tercios de la facturación, evidencia de que la fidelización tiene un retorno desproporcionado frente a la adquisición de nuevos clientes.

---

## 3. Flujo del proyecto

```mermaid
flowchart LR
    A[Datos crudos<br/>CSV] --> B[Extracción y Limpieza<br/>Power Query]
    B --> C[Modelo estrella<br/>Power BI]
    C --> D[Medidas DAX]
    D --> E[Dashboard]
    E --> F[Insights y<br/>recomendaciones]
```

---


## 4. Proceso paso a paso

### 4.1 Limpieza con Power Query

| Problema | Solución aplicada |
|---|---|
| Fechas en 4 formatos distintos como texto | Cambio de tipo a fecha con configuración regional |
| Precios con texto (`S/`, `S.`) mezclado con números | Columna personalizada: `Text.Remove` de letras/símbolos + `Text.Trim` |
| Método de pago con variantes (`YAPE`, `yap`, `Tarje credito`, `Efectivoo`) | Columna condicional con `Text.Contains` para normalizar a 5 valores estándar |
| Sucursal con nombres parciales (`anita`, `civico`, `Polvos Azu`) | Columna condicional con `Text.Contains` para normalizar a 4 sucursales |
| Columnas originales sucias | Eliminadas y reemplazadas por sus versiones limpias, luego renombradas |
| Tipos de datos finales | `Precio_Unitario` y `Cantidad` a entero, resto de columnas verificadas |

<details>
<summary>Ver código M completo (Power Query)</summary>

```m
let
    Origen = Csv.Document(File.Contents("ventas_ecommerce.csv"),[Delimiter=",", Columns=14, Encoding=1252, QuoteStyle=QuoteStyle.None]),
    #"Encabezados promovidos" = Table.PromoteHeaders(Origen, [PromoteAllScalars=true]),
    #"Tipo cambiado" = Table.TransformColumnTypes(#"Encabezados promovidos",{{"Fecha_Pedido", type date}}),
    #"Filas ordenadas" = Table.Sort(#"Tipo cambiado",{{"Fecha_Pedido", Order.Ascending}}),
    #"Texto en mayúsculas" = Table.TransformColumns(#"Filas ordenadas",{{"Estado_Pedido", Text.Upper, type text}, {"Sucursal", Text.Upper, type text}, {"Metodo_Pago", Text.Upper, type text}, {"Producto", Text.Upper, type text}, {"Subcategoria", Text.Upper, type text}, {"Categoria", Text.Upper, type text}, {"Nombres_Cliente", Text.Upper, type text}}),
    #"Personalizada agregada" = Table.AddColumn(#"Texto en mayúsculas", "Precio_Limpio", each let
        limpieza_base = Text.Remove([Precio_Unitario], {"a".."z", "A".."Z", "/", "."}),
        limpieza_espacios = Text.Trim(limpieza_base)
    in
        limpieza_espacios),
    #"Personalizada agregada1" = Table.AddColumn(#"Personalizada agregada", "Metodo_Pago_Limpio", each if Text.Contains([Metodo_Pago], "EFECT") or Text.Contains([Metodo_Pago], "EFECTIVOO") then "EFECTIVO"
        else if Text.Contains([Metodo_Pago], "YAP") then "YAPE"
        else if Text.Contains([Metodo_Pago], "TARJE CREDITO") or Text.Contains([Metodo_Pago], "TARJETA CREDITO") or Text.Contains([Metodo_Pago], "CREDITO") then "TARJETA DE CREDITO"
        else if Text.Contains([Metodo_Pago], "DEBITO") then "TARJETA DE DEBITO"
        else if Text.Contains([Metodo_Pago], "PLIN") then "PLIN"
        else "REVISAR METODO"),
    #"Personalizada agregada2" = Table.AddColumn(#"Personalizada agregada1", "Sucursal_Limpia", each if Text.Contains([Sucursal], "ROSADO") then "POLVOS ROSADOS"
        else if Text.Contains([Sucursal], "AZULES") or Text.Contains([Sucursal], "AZU") then "POLVOS AZULES"
        else if Text.Contains([Sucursal], "ANITA") or Text.Contains([Sucursal], "SANTA ANITA") or Text.Contains([Sucursal], "MALL") then "MALL SANTA ANITA"
        else if Text.Contains([Sucursal], "CIVICO") or Text.Contains([Sucursal], "REALPLAZA") then "REAL PLAZA CENTRO CIVICO"
        else "REVISAR SUCURSAL"),
    #"Columnas quitadas" = Table.RemoveColumns(#"Personalizada agregada2",{"Precio_Unitario", "Metodo_Pago", "Sucursal"}),
    #"Columnas con nombre cambiado" = Table.RenameColumns(#"Columnas quitadas",{{"Producto", "Nombre_Producto"}, {"Precio_Limpio", "Precio_Unitario"}, {"Metodo_Pago_Limpio", "Metodo_Pago"}, {"Sucursal_Limpia", "Sucursal"}}),
    #"Tipo cambiado1" = Table.TransformColumnTypes(#"Columnas con nombre cambiado",{{"Sucursal", type text}, {"Metodo_Pago", type text}, {"Precio_Unitario", Int64.Type}, {"Cantidad", Int64.Type}})
in
    #"Tipo cambiado1"
```

</details>

### 4.2 Modelo estrella

![Modelo estrella](model/modelo_estrella.jpg)

| Tipo | Tabla | Descripción | Clave |
|---|---|---|---|
| **Hechos** | `FACT_VENTAS` | Una fila por producto vendido en cada pedido | `ID_PEDIDO`, `product_id` |
| Dimensión | `DIM_CLIENTES` | Datos del cliente | `DNI` |
| Dimensión | `DIM_PRODUCTOS` | Producto, categoria y subcategoria | `SKU` |
| Dimensión | `DIM_METODO_PAGO` | Datos del metodo pago | `ID_METODO_PAGO` |
| Dimensión | `DIM_SUCURSAL` | Datos de la sucursal | `ID_SUCURSAL` |
| Dimensión | `DIM_CALENDARIO` | Fechas, mes, trimestre, año | `DATE` |

**Relaciones:** todas de uno a muchos (1:*), con filtro en una sola dirección desde las dimensiones hacia la tabla de hechos.

**Decisiones de modelado:** creé una tabla calendario propia para usar funciones de inteligencia de tiempo y una tabla medidas DAX para ser organizado y no tener sueltas las medidas por todas las tablas.

### 4.3 Medidas DAX 

Documentación completa en [`docs/medidas_dax.md`](docs/medidas_dax.md).

| Medida | Fórmula | Qué mide |
|---|---|---|
| Ventas Totales | `SUM(fact_ventas[precio])` | Ingresos totales |
| Pedidos | `DISTINCTCOUNT(fact_ventas[order_id])` | Número de pedidos únicos |
| Ticket Promedio | `DIVIDE([Ventas Totales], [Pedidos])` | Gasto promedio por pedido |
| Ventas Año Anterior | `CALCULATE([Ventas Totales], SAMEPERIODLASTYEAR(dim_calendario[fecha]))` | Ventas del mismo periodo del año pasado |
| % Crecimiento YoY | `DIVIDE([Ventas Totales] - [Ventas Año Anterior], [Ventas Año Anterior])` | Variación anual |
| [Tu medida] | `[fórmula]` | [descripción] |

### 4.4 Dashboard 

**Página 1: Resumen ejecutivo** · KPIs, tendencia de ventas, top categorías
![Página 1](dashboard/pagina1.png)

**Página 2: [Logística / Clientes / Producto]** · [qué muestra]
![Página 2](dashboard/pagina2.png)

**Interactividad:** [segmentadores por fecha y región, drill-through, tooltips personalizados, etc.]

---

## 5. Recomendaciones de negocio


1. Auditar el proceso de gestión de inventario de Consolas en Mall Santa Anita. La tasa de cancelación de Consolas en esta sucursal es 18.8% casi duplica el promedio de la red, y coincide con una caída sostenida de ventas durante 2025 de S/ 47 mil en diciembre 2024 a un mínimo de S/ 9 mil en septiembre 2025. Un patrón común en negocios con venta online y recojo en tienda es que el catálogo no se desactiva en tiempo real cuando el stock físico llega a cero, por lo que se aceptan y cobran pedidos que luego no se pueden cumplir dentro de la ventana de despacho, terminando en cancelación y devolución. Se recomienda auditar la sincronización entre inventario físico y catálogo online para esa sucursal, desactivando automáticamente productos sin stock disponible. Corregirlo podría recuperar hasta S/ 106,946 en ventas hoy perdidas por cancelación y revertir la caída sostenida que arrastra la sucursal desde inicios de 2025.

2. Lanzar una campaña de reactivación con cupón de segunda compra. Los clientes de compra única representan el 64% de la base, pero solo el 36% de las ventas, menos de la mitad del valor promedio de un cliente recurrente. Se recomienda enviarles por correo o mensaje de WhatsApp, usando los datos de contacto ya capturados en su primera compra. Un cupón de 10% o 15% de descuento en Accesorios o Videojuegos, válido por 30 días desde el envío. Elegir categorías de ticket bajo reduce el costo del incentivo por cliente, mientras que el plazo corto genera urgencia para convertir la segunda compra antes de que el cliente pierda interés.

3. Ofrecer descuento cruzado en accesorios al comprar Consolas o Sillas. Accesorios es la categoría de mayor rotación con 3,527 unidades vendidas pero con el ticket promedio más bajo de S/ 341, mientras que Consolas y Sillas concentran el mayor valor por transacción. Se recomienda que, al comprar una Consola o Silla, el cliente reciba automáticamente un 15% de descuento en un accesorio asociado, por ejemplo un control adicional al comprar una consola, o un set de parlantes al comprar una silla gamer, aplicado en el mismo carrito de compra. Esto aumenta el ticket promedio de la transacción principal sin necesidad de atraer tráfico nuevo, aprovechando una compra que el cliente ya decidió hacer.


---

