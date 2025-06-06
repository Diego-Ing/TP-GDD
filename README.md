# Documentación del Diseño de Base de Datos - Esquema DB_SQUAD

## Introducción

Este documento explica cómo se reorganizó la tabla `gd_esquema.Maestra` dividiéndola en varias tablas más pequeñas y organizadas. El objetivo principal fue eliminar información repetida, mejorar la organización de los datos y hacer más fácil el mantenimiento de la base de datos.

## Estrategia General

La tabla original tenía mucha información duplicada. Por ejemplo, si una provincia aparecía en 100 registros, el nombre de esa provincia se guardaba 100 veces. Con el nuevo diseño, el nombre se guarda una sola vez y se referencia desde las otras tablas.

## Detalle de las Tablas Creadas

### 1. DB_SQUAD.Provincia

**¿Por qué se creó?**
Los nombres de las provincias se repetían constantemente en la tabla original cada vez que aparecía una sucursal, cliente o proveedor de esa provincia.

**Estructura:**

- `provincia_codigo`: Número único que identifica cada provincia (se genera automáticamente)
- `provincia_nombre`: Nombre de la provincia (no puede estar vacío y debe ser único)


**Decisiones importantes:**

- Se usa un número como identificador principal en lugar del nombre porque es más eficiente y evita problemas si hay errores de escritura
- No puede haber dos provincias con el mismo nombre


### 2. DB_SQUAD.Localidad

**¿Por qué se creó?**
Similar a las provincias, las localidades se repetían muchas veces y cada localidad pertenece a una provincia específica.

**Estructura:**

- `localidad_codigo`: Número único para cada localidad
- `localidad_provincia`: Referencia a qué provincia pertenece
- `localidad_nombre`: Nombre de la localidad


**Decisiones importantes:**

- Puede haber localidades con el mismo nombre siempre que estén en provincias diferentes (ej: "San Martín" puede existir en Buenos Aires y en Mendoza)
- Toda localidad debe pertenecer obligatoriamente a una provincia


### 3. DB_SQUAD.Direccion

**¿Por qué se creó?**
Las direcciones también se repetían y cada dirección pertenece a una localidad específica.

**Estructura:**

- `direccion_codigo`: Número único para cada dirección
- `direccion_localidad`: Referencia a qué localidad pertenece
- `direccion_nombre`: Nombre de la calle y número


**Decisiones importantes:**

- No puede haber direcciones duplicadas dentro de la misma localidad
- Toda dirección debe pertenecer a una localidad


### 4. DB_SQUAD.EstadoPedido

**¿Por qué se creó?**
Para tener un catálogo fijo de los posibles estados de un pedido (como "ENTREGADO", "CANCELADO", etc.) y evitar errores de escritura.

**Estructura:**

- `estado_id`: Código corto del estado (ej: 'E', 'X', 'N')
- `estado_nombre`: Nombre descriptivo del estado


### 5. DB_SQUAD.Modelo

**¿Por qué se creó?**
La información de cada modelo de sillón (descripción y precio) se repetía para cada sillón de ese modelo.

**Estructura:**

- `modelo_codigo`: Código único del modelo (se mantiene el original)
- `modelo_descripcion`: Descripción del modelo
- `modelo_precio_base`: Precio base del modelo


### 6. DB_SQUAD.Medida

**¿Por qué se creó?**
Las medidas de los sillones (alto, ancho, profundidad) y su precio se repetían para cada sillón con esas dimensiones.

**Estructura:**

- `medida_codigo`: Número único para cada conjunto de medidas
- `medida_alto`, `medida_ancho`, `medida_profundidad`: Dimensiones del sillón
- `medida_precio`: Precio asociado a esas medidas


### 7. DB_SQUAD.Material

**¿Por qué se creó?**
Los materiales y sus precios se repetían para cada sillón que usaba ese material.

**Estructura:**

- `material_codigo`: Número único para cada material
- `material_precio`: Precio del material
- `material_descripcion`: Descripción o nombre del material


### 8. DB_SQUAD.Relleno, DB_SQUAD.Madera, DB_SQUAD.Tela

**¿Por qué se crearon por separado?**
Cada tipo de material tiene características específicas:

- Los rellenos tienen densidad
- Las maderas tienen tipo y dureza
- Las telas tienen color y textura


**Estructura común:**

- Cada tabla tiene su propio identificador único
- Referencia al material general correspondiente
- Campos específicos para cada tipo


### 9. DB_SQUAD.Sillon

**¿Por qué se creó?**
Representa cada sillón individual con su modelo y medidas específicas.

**Estructura:**

- `sillon_id`: Número único interno para cada sillón
- `sillon_codigo_maestra`: Código original del sillón (para mantener la referencia)
- `sillon_modelo`: Referencia al modelo del sillón
- `sillon_medidas`: Referencia a las medidas del sillón


**Decisiones importantes:**

- La combinación de código original, modelo y medidas debe ser única


### 10. DB_SQUAD.ItemMaterial

**¿Por qué se creó?**
Un sillón puede tener varios materiales y un material puede usarse en varios sillones. Esta tabla conecta ambas entidades.

**Estructura:**

- `item_material_sillon`: Referencia al sillón
- `item_material_material`: Referencia al material


### 11. DB_SQUAD.Cliente

**¿Por qué se creó?**
Los datos de los clientes se repetían en cada pedido o factura que hacían.

**Estructura:**

- `Cliente_ID`: Número único interno para cada cliente
- `Cliente_Dni`: DNI del cliente (puede estar vacío)
- `Cliente_Nombre`, `Cliente_Apellido`: Datos personales del cliente


**Decisiones importantes:**

- Se usa un número interno como identificador principal porque el DNI puede estar vacío o tener errores
- Se maneja especialmente el caso de clientes con datos faltantes


### 12. DB_SQUAD.Proveedor

**¿Por qué se creó?**
Los datos de los proveedores se repetían en cada compra.

**Estructura:**

- `proveedor_id`: Número único para cada proveedor
- `proveedor_cuit`: CUIT del proveedor (obligatorio)
- `proveedor_razon_social`: Nombre de la empresa
- Otros datos como teléfono, dirección, localidad y provincia


**Decisiones importantes:**

- El CUIT es obligatorio por ser un identificador fiscal importante
- Se asegura que no haya proveedores duplicados comparando varios campos a la vez


### 13. DB_SQUAD.Sucursal

**¿Por qué se creó?**
Los datos de las sucursales se repetían en cada operación asociada a esa sucursal.

**Estructura:**

- `sucursal_id`: Número único para cada sucursal
- `sucursal_numero_maestra`: Número original de la sucursal
- `sucursal_direccion`: Referencia a la dirección de la sucursal
- `sucursal_mail`: Email de la sucursal


### 14. DB_SQUAD.Compra

**¿Por qué se creó?**
Los datos generales de cada compra (fecha, total, proveedor) se repetían por cada producto comprado.

**Estructura:**

- `compra_codigo`: Número único de la compra (se mantiene el original)
- `compra_fecha`: Fecha de la compra
- `compra_proveedor`: Referencia al proveedor
- `compra_total`: Total de la compra


### 15. DB_SQUAD.ItemCompra

**¿Por qué se creó?**
Cada compra puede tener varios productos. Esta tabla guarda el detalle de cada producto comprado.

**Estructura:**

- `item_compra_id`: Número único para cada línea de detalle
- `item_compra_codigo`: Referencia a la compra
- `item_compra_sillon`: Referencia al sillón (puede estar vacío por limitaciones de los datos originales)
- Cantidad, precio unitario y subtotal


### 16. DB_SQUAD.Pedido

**¿Por qué se creó?**
Los datos generales de cada pedido se repetían por cada producto pedido.

**Estructura:**

- `pedido_codigo`: Número único del pedido
- `pedido_sucursal`: Referencia a la sucursal donde se hizo el pedido
- `pedido_cliente`: Referencia al cliente que hizo el pedido
- `pedido_fecha`: Fecha del pedido (obligatorio)
- `pedido_total`: Total del pedido (obligatorio)
- `pedido_estado`: Estado actual del pedido


### 17. DB_SQUAD.ItemPedido

**¿Por qué se creó?**
Cada pedido puede tener varios productos. Esta tabla guarda el detalle de cada producto pedido.

**Estructura:**

- `item_pedido_codigo`: Referencia al pedido
- `item_pedido_sillon`: Referencia al sillón pedido
- `item_pedido_cantidad`: Cantidad pedida


**Decisiones importantes:**

- La combinación de pedido y sillón debe ser única


### 18. DB_SQUAD.Factura

**¿Por qué se creó?**
Los datos generales de cada factura se repetían por cada producto facturado.

**Estructura:**

- `factura_codigo`: Número único de la factura
- `fact_fecha`: Fecha de la factura
- `fact_total`: Total de la factura


### 19. DB_SQUAD.Envio

**¿Por qué se creó?**
Los datos de envío se asocian a una factura completa, no a productos individuales.

**Estructura:**

- `envio_id`: Número único interno para cada envío
- `envio_codigo`: Número de envío original
- `envio_factura`: Referencia a la factura (obligatorio)
- Fechas programada y real de entrega, costos de traslado y subida


### 20. DB_SQUAD.ItemFactura

**¿Por qué se creó?**
Cada factura puede tener varios productos. Esta tabla guarda el detalle de cada producto facturado.

**Estructura:**

- `item_fact_id`: Número único para cada línea de detalle
- `item_fact_codigo`: Referencia a la factura
- `item_fact_sillon`: Referencia al sillón (puede estar vacío por limitaciones de los datos originales)
- Precio, cantidad y subtotal


## Proceso de Migración

Para cada tabla se siguió un proceso similar:

1. **Identificar datos únicos**: Se usó `DISTINCT` para obtener solo los valores únicos de la tabla original
2. **Resolver referencias**: Se hicieron conexiones (`JOIN`) con las tablas ya creadas para obtener los números de referencia correctos
3. **Manejar datos faltantes**: Se permitieron valores vacíos (`NULL`) cuando los datos originales no siempre estaban completos
4. **Asegurar integridad**: Se establecieron reglas para evitar duplicados y mantener la consistencia


## Beneficios del Nuevo Diseño

1. **Menos espacio**: Se eliminó la repetición masiva de datos
2. **Mejor organización**: Cada tipo de información tiene su lugar específico
3. **Más fácil mantenimiento**: Los cambios se hacen en un solo lugar
4. **Mejor rendimiento**: Las consultas son más eficientes
5. **Mayor integridad**: Las reglas de la base de datos previenen inconsistencias
