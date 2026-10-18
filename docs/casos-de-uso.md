# Casos de uso — Plataforma logística de última milla

## 1. Propósito

Este documento define los casos de uso relacionados con el flujo de asignación de un repartidor a una entrega.

El diseño se centra en AssignCourierUseCase, cuyo objetivo es coordinar la asignación de un repartidor a una entrega existente, respetando las invariantes del dominio y manteniendo la aplicación independiente de los detalles de infraestructura.

## 2. Caso de uso principal: AssignCourierUseCase

### 2.1. Identificación

- Nombre: AssignCourierUseCase.
- Objetivo: asignar un repartidor disponible a una entrega que no tenga una asignación activa.
- Actor principal: operador logístico.
- Entidad principal: Delivery.
- Resultado esperado: entrega guardada con el identificador del repartidor asignado.

### 2.2. Precondiciones

Antes de completar la operación deben cumplirse las siguientes condiciones:

1. La solicitud contiene identificadores válidos.
2. La entrega existe.
3. La entrega no tiene un repartidor activo asignado.
4. El repartidor seleccionado está disponible.
5. La operación puede guardarse correctamente.

Las precondiciones se verifican mediante la coordinación del caso de uso y las reglas protegidas por la entidad de dominio.

### 2.3. Datos de entrada

El caso de uso recibe los siguientes datos:

- DeliveryId: identificador de la entrega.
- CourierId: identificador del repartidor que se desea asignar.

Los identificadores se utilizarán para consultar la entrega y solicitar la verificación de disponibilidad del repartidor.

### 2.4. Flujo principal

1. El operador solicita asignar un repartidor a una entrega.
2. La aplicación recibe DeliveryId y CourierId.
3. AssignCourierUseCase consulta la entrega mediante IDeliveryRepository.
4. Si la entrega no existe, la operación termina con un resultado de error.
5. El caso de uso verifica que la entrega no tenga una asignación activa.
6. El caso de uso consulta la disponibilidad mediante el puerto correspondiente.
7. Si el repartidor está disponible, el caso de uso ejecuta Delivery.AssignCourier(courierId).
8. La entidad protege la invariante e incorpora la asignación.
9. El caso de uso guarda la entrega mediante IDeliveryRepository.
10. La aplicación registra o publica el resultado mediante el mecanismo definido para ese propósito.
11. La operación finaliza informando que la asignación se completó.

### 2.5. Flujos alternativos

#### A. Entrega inexistente

Si el repositorio no encuentra la entrega, el caso de uso debe detener el proceso e informar que no existe.

No debe intentar ejecutar la asignación ni guardar una entidad inexistente.

#### B. Entrega ya asignada

Si la entrega ya tiene un repartidor activo, el caso de uso debe rechazar la solicitud.

La asignación existente debe conservarse.

#### C. Repartidor no disponible

Si la consulta de disponibilidad indica que el repartidor no puede recibir la entrega, el caso de uso debe rechazar la operación.

No debe completarse una nueva asignación.

#### D. Error de persistencia

Si el repositorio falla al guardar la entrega, la aplicación debe comunicar que la operación no pudo completarse.

El diseño de implementación deberá definir cómo manejar la consistencia de los cambios y los posibles reintentos.

#### E. Error en la consulta de disponibilidad

Si no se puede verificar la disponibilidad, el caso de uso debe impedir que la operación se complete como si la validación hubiera sido exitosa.

El error debe comunicarse de forma controlada.

### 2.6. Postcondiciones

Cuando la operación finaliza correctamente:

- La entrega conserva su identificador.
- La entrega tiene asociado el identificador del repartidor seleccionado.
- La asignación ha sido guardada mediante el repositorio.
- Se comunica el resultado de la operación.

Cuando la operación falla:

- No debe informarse que la asignación se completó correctamente.
- Debe comunicarse el motivo del rechazo o del fallo.
- La solución debe preservar las invariantes del dominio.

La garantía exacta de persistencia y recuperación ante fallos deberá concretarse durante la implementación.

## 3. Contrato conceptual del caso de uso

### Solicitud

AssignCourierRequest:

- DeliveryId: identificador de la entrega.
- CourierId: identificador del repartidor.

### Resultado

AssignCourierResult:

- Success: indica si la operación finalizó correctamente.
- ErrorCode: código que identifica el motivo del fallo, cuando corresponda.
- Message: descripción del resultado para la capa de aplicación o presentación.

Estos contratos son propuestas de diseño. Sus tipos concretos y su implementación se definirán en la Actividad 4.

## 4. Responsabilidades y separación de capas

### Dominio

La entidad Delivery protege la regla que impide una segunda asignación mientras exista un repartidor activo.

### Aplicación

AssignCourierUseCase coordina las consultas, las validaciones, la asignación y el almacenamiento.

### Infraestructura

Los adaptadores implementan los contratos de acceso a datos, consulta de disponibilidad y registro o publicación de resultados.

El caso de uso no debe crear directamente conexiones a bases de datos ni depender de un proveedor externo concreto.

## 5. Criterios de finalización

El caso de uso se considerará correctamente diseñado cuando:

1. Sus entradas estén definidas.
2. Sus dependencias estén expresadas mediante contratos.
3. El flujo principal y los flujos alternativos estén documentados.
4. Las reglas críticas permanezcan protegidas en el dominio.
5. Los errores puedan comunicarse sin confundir una operación fallida con una exitosa.
6. El diseño pueda transformarse en pruebas durante la Actividad 4.

## 6. Resultado esperado

AssignCourierUseCase establece una coordinación clara para asignar un repartidor a una entrega.

Su diseño permite comprobar las reglas del negocio y definir las responsabilidades de las capas sin implementar todavía las integraciones externas ni la persistencia real.