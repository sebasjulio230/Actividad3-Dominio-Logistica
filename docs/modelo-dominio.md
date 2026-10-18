# Modelo de dominio — Plataforma logística de última milla

## 1. Propósito

El propósito de este documento es definir el modelo de dominio correspondiente al proceso de asignación de un repartidor a una entrega dentro de una plataforma logística de última milla.

El modelo identifica los conceptos principales del negocio, las entidades, los objetos de valor, las reglas de negocio y las restricciones que deben cumplirse para realizar una asignación válida.

La propuesta busca proteger las reglas del negocio y mantenerlas independientes de los detalles técnicos, como las bases de datos, las API externas y los frameworks de desarrollo.

Este diseño servirá como base para implementar y validar el flujo de asignación en la Actividad 4.

## 2. Alcance del modelo

El alcance de esta propuesta se limita al flujo de asignación de un repartidor a una entrega.

El proceso incluye las siguientes actividades:

1. Recibir el identificador de la entrega y del repartidor.
2. Consultar la entrega mediante el repositorio correspondiente.
3. Verificar que la entrega exista.
4. Comprobar que la entrega no tenga un repartidor activo asignado.
5. Consultar la disponibilidad del repartidor mediante un puerto.
6. Ejecutar la operación de asignación en la entidad Delivery.
7. Guardar los cambios mediante el repositorio.
8. Registrar o publicar el resultado de la operación mediante un adaptador.

En esta actividad no se implementará el flujo completo ni se construirán integraciones externas reales.

Quedan fuera del alcance la gestión completa de inventarios, el cálculo de rutas, las devoluciones, la gestión de incidencias y el seguimiento integral de los pedidos.

## 3. Descripción del problema de negocio

Una plataforma logística de última milla necesita coordinar la asignación de las entregas a los repartidores responsables de transportarlas hasta su destino.

Para realizar esta operación correctamente, el sistema debe comprobar que la entrega exista, que no tenga otro repartidor activo y que el repartidor seleccionado esté disponible.

Si estas condiciones no se controlan, pueden producirse asignaciones duplicadas, conflictos entre responsables y problemas en la operación logística.

Por esta razón, las reglas críticas deben protegerse en el modelo de dominio y no depender exclusivamente de las validaciones realizadas en una interfaz o en un servicio externo.

El diseño propuesto separa las reglas del negocio de los mecanismos técnicos utilizados para consultar, guardar y comunicar la información.

## 4. Actores y responsabilidades

### 4.1. Operador logístico

Es el actor que solicita la asignación de un repartidor a una entrega.

Su responsabilidad es seleccionar la entrega y el repartidor que desea asignar, iniciando el proceso correspondiente.

### 4.2. AssignCourierUseCase

Es el caso de uso encargado de coordinar el proceso de asignación.

Sus responsabilidades son:

- Recibir los datos necesarios para realizar la operación.
- Consultar la entrega.
- Solicitar la verificación de disponibilidad.
- Coordinar la asignación mediante la entidad de dominio.
- Guardar el resultado.
- Gestionar los resultados de la operación.

### 4.3. Delivery

Es la entidad que representa una entrega identificable dentro de la plataforma.

Su responsabilidad principal es proteger las reglas relacionadas con la asignación de un repartidor.

### 4.4. IDeliveryRepository

Es el contrato que define las operaciones necesarias para consultar y guardar las entregas.

El caso de uso depende de este contrato y no de una implementación concreta de base de datos.

### 4.5. Puerto de disponibilidad

Representa la capacidad de consultar si un repartidor puede recibir una asignación.

El puerto permite que la aplicación solicite esta información sin depender directamente de una API o proveedor específico.

### 4.6. Adaptadores de infraestructura

Son los componentes que implementan los contratos necesarios para comunicarse con bases de datos, servicios externos y mecanismos de registro o publicación de resultados.

## 5. Entidades del dominio

### 5.1. Delivery (Entrega)

Delivery representa una entrega que debe ser gestionada por la plataforma logística.

Atributos principales:

- Id: identificador único de la entrega.
- Address: dirección donde debe realizarse la entrega.
- CourierId: identificador del repartidor asignado. Puede no tener valor cuando la entrega todavía no está asignada.

Responsabilidades:

- Mantener la identidad de la entrega.
- Conservar los datos necesarios para identificar su destino.
- Permitir la asignación de un repartidor.
- Impedir una nueva asignación cuando ya existe un repartidor activo.
- Proteger las reglas del negocio relacionadas con su asignación.

La operación AssignCourier(courierId) será responsable de comprobar que la entrega no tenga un repartidor asignado antes de registrar la nueva asignación.

En el alcance inicial de este modelo, una entrega sin `CourierId` se considera pendiente de asignación, mientras que una entrega con `CourierId` registrado se considera asignada. Como la cancelación y la reasignación están fuera del alcance, no se contempla todavía un estado independiente para finalizar una asignación activa.

### 5.2. Courier (Repartidor)

Courier representa al repartidor que puede recibir entregas dentro de la plataforma.

El repartidor tiene una identidad propia y su disponibilidad es una condición necesaria para realizar la asignación.

En el alcance de esta actividad, la consulta de disponibilidad se realizará mediante un puerto. No se implementará todavía un modelo completo para administrar los datos personales, las rutas o el historial de trabajo de los repartidores.

### 5.3. DeliveryAssignment (Asignación de entrega)

DeliveryAssignment representa conceptualmente la relación entre una entrega y el repartidor responsable.

En esta propuesta inicial, la asignación se expresará mediante el atributo CourierId de la entidad Delivery.

No se implementará DeliveryAssignment como una entidad independiente en esta etapa. Esta posibilidad podrá evaluarse posteriormente si el negocio necesita conservar un historial de asignaciones, fechas, responsables anteriores o motivos de reasignación.

## 6. Objetos de valor

Los objetos de valor representan conceptos definidos por sus atributos y significado, sin identidad independiente dentro del modelo.

### 6.1. DeliveryId

DeliveryId representa el identificador de una entrega.

Su propósito es distinguir una entrega de cualquier otra y evitar que los identificadores se utilicen sin un significado explícito dentro del código.

### 6.2. DeliveryAddress

DeliveryAddress representa la dirección asociada a la entrega.

Este objeto permite agrupar el dato de ubicación bajo un concepto propio del negocio.

En una implementación posterior se podrán establecer reglas de validación para evitar direcciones vacías o incompletas.

### 6.3. CourierId

CourierId representa el identificador del repartidor seleccionado para la asignación.

Su propósito es distinguir al repartidor y facilitar la relación entre la entrega y el responsable asignado.

Los tipos concretos y las validaciones de estos objetos se definirán durante la implementación.

## 7. Invariantes del dominio

Las invariantes son reglas que deben mantenerse para proteger la consistencia del modelo.

### Invariante 1. Identidad válida

Toda entrega debe contar con un identificador válido que permita distinguirla dentro de la plataforma.

### Invariante 2. Existencia de la entrega

No se puede asignar un repartidor a una entrega inexistente.

El caso de uso debe comprobar la existencia de la entrega antes de ejecutar la operación de dominio.

### Invariante 3. Una única asignación activa

Una entrega no puede tener dos repartidores activos al mismo tiempo dentro del flujo definido.

Si ya existe un repartidor asignado, una nueva solicitud debe rechazarse.

### Invariante 4. Disponibilidad del repartidor

La asignación solamente debe continuar después de verificar que el repartidor está disponible.

### Invariante 5. Protección desde el dominio

La entidad Delivery debe proteger la regla que impide realizar una segunda asignación mientras exista una asignación activa.

Esta protección no debe depender exclusivamente de la interfaz de usuario o del caso de uso.

### Invariante 6. Persistencia mediante un contrato

La aplicación debe guardar la entrega mediante IDeliveryRepository, sin depender directamente de una implementación concreta de almacenamiento.

### Invariante 7. Independencia tecnológica

Las reglas del dominio no deben depender de una base de datos, una API externa o un framework específico.

### Consideración sobre concurrencia

La disponibilidad de un repartidor puede cambiar después de ser consultada. En una implementación real será necesario definir mecanismos que eviten asignaciones incompatibles cuando dos operaciones ocurran simultáneamente.

## 8. Estados y transiciones relevantes

Para este flujo se consideran los siguientes estados conceptuales:

### Pendiente de asignación

La entrega existe y todavía no tiene un repartidor asignado.

Desde este estado puede iniciarse el proceso de asignación.

### Asignada

La entrega tiene un repartidor asociado.

Una nueva asignación debe rechazarse mientras esa asignación continúe activa.

### Solicitud de asignación rechazada

Representa el resultado de una operación que no pudo completarse porque incumplió alguna regla.

Este resultado no constituye necesariamente un estado persistido de la entrega. Por ejemplo, una entrega puede continuar pendiente de asignación si el repartidor seleccionado no está disponible.

### Transiciones principales

- Pendiente de asignación → Asignada: cuando la entrega existe, el repartidor está disponible y la operación se completa correctamente.
- Pendiente de asignación → Solicitud rechazada: cuando falla alguna validación necesaria.
- Asignada → Nueva solicitud rechazada: cuando se intenta asignar otro repartidor mientras existe una asignación activa.

La cancelación de asignaciones, la reasignación y la finalización de una entrega quedan fuera del alcance de esta propuesta inicial.

## 9. Reglas de negocio

Las reglas principales que deberá respetar la solución son:

1. La entrega debe existir antes de iniciar la asignación.
2. La entrega no debe tener un repartidor activo asignado.
3. El repartidor seleccionado debe estar disponible.
4. La operación de asignación debe ejecutarse a través de la entidad Delivery.
5. El caso de uso debe coordinar las operaciones sin incorporar detalles concretos de infraestructura.
6. La entrega modificada debe guardarse mediante el repositorio.
7. Los errores deben comunicarse de forma clara para distinguir una entrega inexistente, una asignación duplicada y un repartidor no disponible.

## 10. Límites entre las capas

### Dominio

Contiene las entidades, los objetos de valor y las reglas del negocio.

En esta propuesta, Delivery protege la operación de asignación y las restricciones relacionadas con ella.

### Aplicación

Contiene el caso de uso AssignCourierUseCase y los contratos que necesita para coordinar el proceso.

Su función es organizar la secuencia de operaciones sin asumir responsabilidades propias de una base de datos o de un servicio externo.

### Infraestructura

Contiene las implementaciones concretas de los contratos, como los repositorios de almacenamiento, los clientes de servicios externos y los mecanismos de registro o publicación.

La infraestructura puede depender de la aplicación y de sus contratos, pero las capas internas no deben depender de las implementaciones externas.

## 11. Supuestos y aspectos pendientes de confirmación

Para desarrollar esta propuesta se establecen los siguientes supuestos:

- La asignación inicial no permite dos repartidores activos simultáneamente para una misma entrega.
- La disponibilidad del repartidor se consulta mediante un puerto.
- La operación de reasignación no forma parte del alcance inicial.
- El atributo CourierId permite identificar al repartidor asociado a una entrega.
- La disponibilidad puede cambiar durante el proceso, por lo que la implementación deberá considerar la concurrencia.
- El mecanismo exacto de registro o publicación del resultado se definirá al establecer los contratos de aplicación.

Estos supuestos deberán contrastarse con las decisiones arquitectónicas y las reglas del negocio definidas en las actividades anteriores.

## 12. Resultado esperado

El modelo de dominio establece los conceptos, las responsabilidades y las reglas necesarias para diseñar el flujo de asignación de repartidores.

Este documento servirá como referencia para definir el lenguaje ubicuo, los escenarios de aceptación, los contratos de aplicación, los puertos, los adaptadores y los diagramas arquitectónicos.

La propuesta no implementa todavía el flujo completo. Su finalidad es proporcionar un diseño claro, coherente y verificable que sirva como base para la Actividad 4.