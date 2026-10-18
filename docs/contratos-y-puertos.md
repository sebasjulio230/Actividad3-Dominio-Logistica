# Contratos y puertos — Plataforma logística de última milla

## 1. Propósito

Este documento define los contratos y puertos necesarios para el caso de uso AssignCourierUseCase, encargado de asignar un repartidor a una entrega.

Los contratos establecen las operaciones que necesita la aplicación para consultar entregas, verificar la disponibilidad de un repartidor, guardar los cambios y registrar el resultado de la operación.

El objetivo es mantener la lógica del negocio independiente de las implementaciones técnicas de persistencia, comunicación externa y registro de eventos.

## 2. Principio de diseño

La solución utilizará el principio de inversión de dependencias.

El caso de uso no dependerá directamente de una base de datos, una API externa o un proveedor de mensajería. En su lugar, dependerá de contratos que expresen las capacidades necesarias para completar la asignación.

Las implementaciones concretas de esos contratos estarán en la capa de infraestructura.

De esta manera, será posible cambiar la tecnología utilizada para guardar entregas o consultar disponibilidad sin modificar las reglas centrales del dominio.

## 3. Puerto de entrada

### 3.1. AssignCourierUseCase

El puerto de entrada representa la operación que permite solicitar la asignación de un repartidor a una entrega.

Su responsabilidad es coordinar el proceso de asignación a partir de los identificadores recibidos.

#### Datos de entrada

- DeliveryId: identificador de la entrega.
- CourierId: identificador del repartidor seleccionado.

#### Resultado esperado

Un resultado que indique si la asignación se completó o si fue rechazada o no pudo finalizarse.

#### Responsabilidades

- Recibir la solicitud.
- Consultar la entrega.
- Verificar las condiciones necesarias.
- Solicitar la consulta de disponibilidad.
- Ejecutar la operación de dominio.
- Guardar los cambios.
- Comunicar el resultado.

El caso de uso no debe contener detalles específicos de conexiones a bases de datos ni de clientes HTTP.

## 4. Puerto de repositorio de entregas

### 4.1. IDeliveryRepository

IDeliveryRepository es el contrato que necesita la aplicación para consultar y guardar las entidades Delivery.

#### Operación GetAsync

Recibe un DeliveryId y devuelve la entrega correspondiente si existe.

Si no se encuentra, devuelve un resultado que permita identificar que la entrega no existe.

#### Operación SaveAsync

Recibe una entidad Delivery y solicita guardar su estado actual.

Si ocurre un error durante el almacenamiento, la aplicación debe poder reconocer que la operación no finalizó correctamente.

#### Responsabilidades del puerto

- Definir las operaciones de consulta y almacenamiento.
- Evitar que el caso de uso dependa de una base de datos específica.
- Facilitar la sustitución de la implementación de persistencia.
- Permitir que el flujo sea probado utilizando implementaciones simuladas.

La implementación concreta del repositorio queda fuera del alcance de esta actividad.

## 5. Puerto de disponibilidad del repartidor

### 5.1. ICourierAvailability

Este puerto representa la capacidad de consultar si un repartidor puede recibir una nueva asignación.

#### Operación IsAvailableAsync

Recibe un CourierId y devuelve el resultado de la consulta de disponibilidad.

El resultado debe permitir distinguir entre las siguientes situaciones:

- El repartidor está disponible.
- El repartidor no está disponible.
- No fue posible verificar su disponibilidad.

El tercer resultado es importante porque un fallo de comunicación no debe interpretarse como una confirmación de disponibilidad.

#### Responsabilidades del puerto

- Definir el contrato de consulta.
- Evitar la dependencia directa de una API externa.
- Permitir que la aplicación maneje errores de disponibilidad.
- Facilitar las pruebas del caso de uso.

La fuente real de información sobre disponibilidad deberá definirse antes de la implementación.

## 6. Puerto de registro o publicación del resultado

### 6.1. IAssignmentResultPublisher

Este puerto representa la capacidad de registrar o publicar el resultado de una asignación completada.

Su finalidad es permitir que otros componentes conozcan el resultado de la operación cuando el negocio lo requiera.

#### Información conceptual

- Identificador de la entrega.
- Identificador del repartidor asignado.
- Tipo de resultado.
- Fecha y hora del registro, si corresponde.

#### Responsabilidades

- Definir la información que se comunicará.
- Evitar que el caso de uso dependa directamente de un sistema de mensajería.
- Permitir distintas implementaciones de publicación o registro.

Este puerto es una propuesta de diseño. Su necesidad, su mecanismo de ejecución y su comportamiento ante fallos deberán confirmarse con las decisiones arquitectónicas anteriores.

## 7. Adaptadores de infraestructura propuestos

Los adaptadores implementarán los contratos definidos por la aplicación.

### 7.1. DeliveryRepositoryAdapter

Implementará IDeliveryRepository utilizando la tecnología de persistencia seleccionada.

Sus responsabilidades serán consultar las entregas, guardar los cambios y traducir los resultados técnicos al contrato definido por la aplicación.

### 7.2. CourierAvailabilityAdapter

Implementará ICourierAvailability y consultará la fuente que proporcione información sobre la disponibilidad del repartidor.

Podría utilizar una API interna, otro servicio o una fuente de datos autorizada por la arquitectura.

La tecnología concreta todavía no está definida.

### 7.3. AssignmentResultPublisherAdapter

Implementará IAssignmentResultPublisher para registrar o publicar el resultado de la asignación.

Podría utilizar un mecanismo de eventos, una cola de mensajes o un sistema de registro, siempre que la alternativa seleccionada sea coherente con las decisiones arquitectónicas del proyecto.

No se implementará ninguna integración real en esta actividad.

## 8. Contrato conceptual de solicitud

### AssignCourierRequest

Representa la solicitud de asignación.

Campos:

- DeliveryId: identificador de la entrega.
- CourierId: identificador del repartidor.

La solicitud deberá validarse antes de iniciar el proceso.

## 9. Contrato conceptual de respuesta

### AssignCourierResult

Representa el resultado del caso de uso.

Campos propuestos:

- Success: indica si la operación se completó correctamente.
- ErrorCode: identifica el motivo del fallo, cuando corresponda.
- Message: proporciona una descripción del resultado.

Códigos de error conceptuales:

- DELIVERY_NOT_FOUND: la entrega no existe.
- DELIVERY_ALREADY_ASSIGNED: la entrega ya tiene un repartidor activo.
- COURIER_NOT_AVAILABLE: el repartidor no está disponible.
- INVALID_REQUEST: los datos de entrada no son válidos.
- AVAILABILITY_CHECK_FAILED: no fue posible verificar la disponibilidad.
- PERSISTENCE_FAILED: no fue posible guardar la entrega.

Estos códigos son propuestas para documentar el contrato y no corresponden todavía a excepciones o clases implementadas.

## 10. Dirección de las dependencias

La dirección de las dependencias seguirá este principio:

1. El caso de uso depende de los contratos que necesita.
2. Los contratos no dependen de implementaciones concretas de infraestructura.
3. Los adaptadores implementan los contratos.
4. El dominio protege las reglas de negocio.
5. Los detalles técnicos permanecen fuera de las entidades del dominio.

El flujo conceptual es el siguiente:

Operador logístico → AssignCourierUseCase → Contratos de aplicación → Adaptadores de infraestructura.

Durante el proceso, AssignCourierUseCase utiliza la entidad Delivery para ejecutar la regla de asignación.

La entidad Delivery no necesita conocer las API externas, los repositorios concretos ni los mecanismos de mensajería.

## 11. Tratamiento de errores

El diseño debe distinguir los errores de negocio de los errores técnicos.

### Errores de negocio

- Entrega inexistente.
- Entrega ya asignada.
- Repartidor no disponible.
- Solicitud inválida.

Estos resultados indican que la operación no puede continuar bajo las condiciones actuales.

### Errores técnicos

- Fallo al consultar el repositorio.
- Error al consultar la disponibilidad.
- Fallo al guardar los cambios.
- Error al registrar o publicar el resultado.

Los errores técnicos deben gestionarse de forma controlada y no deben presentarse como una asignación exitosa.

El tratamiento de la consistencia, los reintentos y la recuperación ante fallos se definirá durante la implementación.

## 12. Pruebas previstas

Los contratos facilitarán la validación del flujo mediante pruebas unitarias y pruebas de integración.

Se proponen las siguientes pruebas:

1. Asignación exitosa con un repositorio simulado y un repartidor disponible.
2. Rechazo cuando la entrega no existe.
3. Rechazo cuando la entrega ya está asignada.
4. Rechazo cuando el repartidor no está disponible.
5. Manejo del fallo al consultar disponibilidad.
6. Manejo del fallo de persistencia.
7. Verificación de que la entidad protege la invariante de asignación.
8. Verificación de que los errores no producen una respuesta de éxito.

Estas pruebas se implementarán en la Actividad 4.

## 13. Decisiones pendientes

Antes de implementar los contratos será necesario confirmar:

- La tecnología y las condiciones de persistencia.
- La fuente real de disponibilidad de los repartidores.
- La necesidad de publicar eventos o registrar únicamente el resultado.
- El mecanismo para mantener la consistencia ante solicitudes simultáneas.
- La estrategia de recuperación ante fallos de almacenamiento y publicación.

## 14. Resultado esperado

Los contratos y puertos definidos proporcionan una estructura para desarrollar AssignCourierUseCase sin acoplarlo a tecnologías concretas.

La propuesta separa las responsabilidades del dominio, la aplicación y la infraestructura, y permite implementar y probar el flujo en una actividad posterior.