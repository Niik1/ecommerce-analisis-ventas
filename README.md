# 🛒·Análisis de E-commerce de punta a punta

> **Resumen en una frase:** Analicé [X] pedidos de [empresa/dataset] entre [año] y [año] para identificar [problema] y recomendar [acción], lo que podría [impacto estimado, ej. reducir 12% los retrasos de entrega].

- Ejemplo: "Analicé 100 mil pedidos de Olist (Brasil, 2016-2018) para entender por qué cae la satisfacción del cliente y recomendar mejoras logísticas que podrían reducir 15% las malas reseñas." 

## 🔗 Enlaces rápidos

| 📊 Dashboard | 🎥 Video (2-3 min) |
|---|---|
| [Ver Dashboard Interactivo](URL) | [Ver Video Resumen](URL) |

![Vista previa del dashboard](dashboard/captura_principal.png)
<!-- Un GIF del dashboard en uso también funciona muy bien aquí. -->

---

## 1. Problema de negocio

**Contexto:** [2-3 líneas: quién es la empresa, qué vende, qué situación enfrenta.]

**Preguntas que quería responder:**
1. ¿[Pregunta 1, ej. cómo evolucionan las ventas mes a mes y qué categorías las impulsan]?
2. ¿[Pregunta 2, ej. qué regiones tienen más retrasos en entrega]?
3. ¿[Pregunta 3, ej. cuál es la relación entre tiempo de entrega y satisfacción]?
4. ¿[Pregunta 4, ej. qué segmento de clientes es más valioso]?

---

## 2. Hallazgos principales

<!-- Llena esta sección AL FINAL, pero déjala arriba: es lo que más lee el reclutador. Siempre con números. -->

- 📈 **[Hallazgo 1]:** [dato concreto, ej. las ventas crecieron 45% en Q4, impulsadas por 3 categorías].
- 🚚 **[Hallazgo 2]:** [dato concreto, ej. los pedidos con más de 10 días de entrega reciben 2.3x más reseñas negativas].
- 👥 **[Hallazgo 3]:** [dato concreto, ej. solo 3% de clientes recompra, hay oportunidad de retención].

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

## 4. Datos

- **Fuente:** [nombre y enlace, ej. Brazilian E-Commerce Public Dataset by Olist, Kaggle]
- **Periodo:** [fechas]
- **Tamaño:** [X filas, Y tablas]
- **Tablas principales:** `orders`, `order_items`, `customers`, `products`, `payments`, `reviews`, [...]

> ⚠️ Si el dataset es muy pesado, sube solo una muestra en `/data` y deja el enlace al original.

---

## 5. Proceso paso a paso
### 5.1 Limpieza con Power Query


| Problema | Solución (paso aplicado) |
|---|---|
| Fechas como texto | Cambiar tipo a fecha con configuración regional |
| Nulos en [columna] | Reemplazar valores / Quitar filas |
| Nombres inconsistentes | Recortar, minúsculas y reemplazar valores |
| Varios archivos separados | Combinar archivos de una carpeta |


### 5.2 Modelo estrella ⭐

![Modelo estrella](model/modelo_estrella.jog)

| Tipo | Tabla | Descripción | Clave |
|---|---|---|---|
| **Hechos** | `FACT_VENTAS` | Una fila por producto vendido en cada pedido | `ID_PEDIDO`, `product_id` |
| Dimensión | `DIM_CLIENTES` | Datos del cliente | `DNI` |
| Dimensión | `DIM_PRODUCTOS` | Producto, categoria y subcategoria | `SKU` |
| Dimensión | `DIM_METODO_PAGO` | Datos del metodo pago | `ID_METODO_PAGO` |
| Dimensión | `DIM_SUCURSAL` | Datos de la sucursal | `ID_SUCURSAL` |
| Dimensión | `DIM_CALENDARIO` | Fechas, mes, trimestre, año | `DATE` |

**Relaciones:** todas de uno a muchos (1:*), con filtro en una sola dirección desde las dimensiones hacia la tabla de hechos.

**Decisiones de modelado:** creé una tabla calendario propia para usar funciones de inteligencia de tiempo.

### 5.3 Medidas DAX 🧮

Documentación completa en [`docs/medidas_dax.md`](docs/medidas_dax.md).

| Medida | Fórmula | Qué mide |
|---|---|---|
| Ventas Totales | `SUM(fact_ventas[precio])` | Ingresos totales |
| Pedidos | `DISTINCTCOUNT(fact_ventas[order_id])` | Número de pedidos únicos |
| Ticket Promedio | `DIVIDE([Ventas Totales], [Pedidos])` | Gasto promedio por pedido |
| Ventas Año Anterior | `CALCULATE([Ventas Totales], SAMEPERIODLASTYEAR(dim_calendario[fecha]))` | Ventas del mismo periodo del año pasado |
| % Crecimiento YoY | `DIVIDE([Ventas Totales] - [Ventas Año Anterior], [Ventas Año Anterior])` | Variación anual |
| [Tu medida] | `[fórmula]` | [descripción] |

### 5.4 Dashboard 📊

**Página 1: Resumen ejecutivo** · KPIs, tendencia de ventas, top categorías
![Página 1](dashboard/pagina1.png)

**Página 2: [Logística / Clientes / Producto]** · [qué muestra]
![Página 2](dashboard/pagina2.png)

**Interactividad:** [segmentadores por fecha y región, drill-through, tooltips personalizados, etc.]

---

## 6. Recomendaciones de negocio

1. **[Recomendación 1]:** [acción concreta + impacto esperado].
2. **[Recomendación 2]:** [acción concreta + impacto esperado].
3. **[Recomendación 3]:** [acción concreta + impacto esperado].

---

## 7. Limitaciones y próximos pasos

- [Limitación, ej. los datos no incluyen costos, así que no se puede calcular margen.]
- [Próximo paso, ej. automatizar la actualización con un pipeline en Python.]
- [Próximo paso, ej. agregar un modelo de predicción de ventas.]

---
