Hola grupo ,para terminar la Práctica 2  vamos a dividir el trabajo así. Mi parte, Persona 1, ya deja listo el flujo inicial:

LlegadaClientes → NodoColaUnica → MostradorAtencion → NodoOrdenOK

Desde `NodoOrdenOK` continúan las demás partes del modelo.

## Persona 2: Productos, tablas, inventario, reposición y faltantes

Tu parte empieza desde `NodoOrdenOK`, donde ya sale un cliente con pedido confirmado.

Tenés que encargarte de importar los datos de Excel a tablas internas de Simio y asignar a cada pedido un producto, cantidad, inventario, precio y datos relacionados.

### Componentes o elementos a usar

* Tablas internas de Simio:

  * `TablaProductos`
  * `TablaPrecios`
  * `TablaCostos`
* Estados o variables:

  * `InventarioBujias`
  * `InventarioLubricantes`
  * `InventarioFiltros`
  * `InventarioPastillas`
  * `FaltantesTotales`
  * `OrdenesConEscasez`
* Add-On Processes o procesos:

  * `AsignarProductoPedido`
  * `DescontarInventario`
  * `RevisarReposicion`
  * `RegistrarFaltante`
* Labels:

  * Status Label para mostrar el tipo de producto de la entidad cuando sea necesario.

### Qué debe hacer esta parte

1. Importar el archivo `ProbabilidadesProductos.xlsx` a una tabla interna.
2. Usar esa tabla para seleccionar el tipo de producto que solicita cada cliente:

   * Bujías.
   * Lubricantes.
   * Filtros.
   * Pastillas de freno.
3. Asignar también la cantidad pedida por el cliente según la tabla.
4. Guardar en atributos de la entidad:

   * `TipoProducto`
   * `CantidadPedida`
   * `CantidadDisponible`
   * `CantidadFaltante`
   * `PrecioUnitario`
5. Descontar inventario cuando se confirme el pedido.
6. Si no hay suficiente inventario, el pedido debe continuar con lo disponible, pero se debe registrar:

   * Cuántas unidades quedaron pendientes.
   * Cuántas órdenes tuvieron escasez.
7. Cuando el inventario de un producto llegue a la mitad de su inventario inicial, debe activarse una reposición.
8. La reposición debe tardar un tiempo uniforme entre 2 y 5 minutos.
9. Cada lote de reposición debe ser equivalente al 25% del inventario inicial del producto.

### Salida de tu parte

Después de asignar producto, cantidad e inventario, el pedido debe pasar al área de almacén automático de la Persona 3.

---

## Persona 3: Almacén, despacho automático y control de calidad

Tu parte empieza después de que la Persona 2 ya asignó producto y cantidad al pedido.

Tenés que modelar el almacén automático, la salida unidad por unidad, las bandas transportadoras y las dos inspecciones de calidad.

### Componentes a colocar

* Server:

  * `AlmacenAutomatico`
* Separator o lógica equivalente:

  * `SeparadorUnidades`
* Path o TimePath:

  * `BandaAlmacenCalidad`
* Servers:

  * `InspeccionDefectos`
  * `InspeccionPeso`
* Path o TimePath:

  * `BandaEntreInspecciones`
  * `BandaAReciclaje`
  * `BandaAEmpaque`
* Sink temporal o final de desecho:

  * `ContenedorReciclaje`

### Configuraciones importantes

1. El almacén debe liberar las unidades una por una.
2. El tiempo de salida por unidad debe ser:

   * `Random.Triangular(1.5, 2.0, 2.5)` segundos.
3. La cinta principal del almacén a control de calidad debe tener:

   * Longitud: 12 metros.
   * Velocidad: 170 cm/s.
4. El almacén debe terminar todo el pedido actual antes de empezar el siguiente.
5. Las inspecciones son dos estaciones consecutivas:

   * Primera inspección: desperfectos de fábrica.
   * Segunda inspección: revisión de peso/empaque individual.
6. La banda entre inspecciones debe tener:

   * Longitud: 5 metros.
   * Velocidad: 50 cm/s.
7. Las probabilidades de aprobación y tiempos de inspección deben venir de `ProbabilidadesInspeccion.xlsx`.
8. Si una unidad falla cualquiera de las dos inspecciones, debe desviarse a reciclaje.
9. La banda hacia reciclaje debe tener:

   * Longitud: 15 metros.
   * Velocidad: 50 cm/s.
10. Las unidades que aprueben ambas inspecciones deben ir a empaque.
11. La banda hacia empaque debe tener:

* Longitud: 14 metros.
* Velocidad: 130 cm/s.

### Salida de tu parte

Las unidades aprobadas deben llegar al área de empaque de la Persona 4. Las unidades rechazadas deben salir por `ContenedorReciclaje`.

---

## Persona 4: Empaque, cajas y entrega final FIFO

Tu parte empieza cuando llegan unidades aprobadas desde control de calidad.

Tenés que agrupar las unidades en cajas de 10, agregar cajas vacías, sellar y etiquetar, y entregar los pedidos respetando FIFO.

### Componentes a colocar

* Source:

  * `DispensadorCajas`
* Combiner o lógica equivalente:

  * `ArmadoCaja`
* Server:

  * `SelladoEtiquetado`
* Path o TimePath:

  * `BandaCajasVacias`
  * `BandaCajaEntrega`
* Server o nodo de entrega:

  * `EntregaFinal`
* Sink:

  * `SalidaClientes`

### Configuraciones importantes

1. Cada caja debe contener exactamente 10 unidades.
2. No importa el tipo de producto: cada caja agrupa 10 unidades aprobadas.
3. Las cajas vacías vienen desde un dispensador automático.
4. El dispensador de cajas vacías está a 7 metros del área de empaque.
5. La banda de cajas vacías debe avanzar a 120 cm/s.
6. El proceso de sellado y etiquetado debe tener:

   * `Random.Triangular(6, 8, 11)` segundos.
7. Las cajas terminadas deben pasar al área de entrega final.
8. La entrega al cliente debe respetar FIFO, es decir, el primer pedido que llegó debe ser el primero en entregarse.
9. Se debe cuidar que las cajas correspondan al orden del pedido original del cliente.

### Salida de tu parte

El pedido final debe salir por `SalidaClientes`, que representará los pedidos completados y entregados.

---

## Persona 5: Finanzas, gráficas, labels, estética y documentación

Tu parte es integrar los indicadores económicos, la visualización del modelo y la documentación final.

### Elementos a configurar

* Tablas:

  * `TablaPrecios`
  * `TablaCostos`
* Estados o variables:

  * `IngresosTotales`
  * `CostosTotales`
  * `GananciaTotal`
  * `PedidosCompletados`
  * `PedidosConFaltante`
  * `UnidadesDesechadas`
* Floor Labels:

  * `LblIngresosTotales`
  * `LblCostosTotales`
  * `LblGananciaTotal`
* Status Labels:

  * Cantidad de pedidos.
  * Tipo de entidad/producto cuando sea necesario.
* Gráfica:

  * Utilización de servidores en gráfica de líneas.
* Documentación:

  * Archivo `[MyS1]Documentación_G3.md`.

### Qué debe hacer esta parte

1. Importar o revisar los datos de `Precios.xlsx`.
2. Importar o revisar los datos de `Costos.xlsx`.
3. Calcular ingresos según producto vendido y cantidad entregada.
4. Calcular costos según uso de servidores y costos definidos en la tabla.
5. Calcular utilidad:

   * `GananciaTotal = IngresosTotales - CostosTotales`
6. Mostrar en todo momento:

   * Ingresos totales.
   * Costos totales.
   * Ganancias totales.
7. Crear una gráfica de líneas con el factor de utilización de los servidores.
8. Revisar que el modelo tenga estética y nombres claros.
9. Armar la documentación final explicando:

   * Diseño del modelo.
   * Componentes usados.
   * Distribuciones usadas.
   * Tablas importadas.
   * Estados y atributos.
   * Lógica de inventario.
   * Lógica de inspección.
   * Lógica de empaque.
   * Lógica FIFO.
   * Finanzas.
   * Resultados.
   * Conclusiones.
10. Preparar capturas del modelo, resultados y gráficas.

---

## Integración final

Cuando todos terminen, el flujo completo debe quedar así:

LlegadaClientes
→ NodoColaUnica
→ MostradorAtencion
→ NodoOrdenOK
→ Selección de Producto e Inventario
→ Almacén Automático
→ Separación de Unidades
→ Banda a Control de Calidad
→ Inspección 1
→ Inspección 2
→ Empaque
→ Armado de Cajas de 10 unidades
→ Sellado y Etiquetado
→ Entrega Final FIFO
→ SalidaClientes

Los productos rechazados deben ir a:

Inspección fallida
→ BandaAReciclaje
→ ContenedorReciclaje

Y si hay faltantes de inventario, el pedido no se detiene: debe continuar con lo disponible, registrando unidades pendientes y órdenes con escasez.

Cada persona debe poder explicar su parte en la defensa, porque van a hacer preguntas para validar que todos trabajaron.
