# Lenguaje ubicuo — Plataforma logística de última milla

## 1. Propósito

El propósito de este documento es establecer un vocabulario común para describir el proceso de asignación de un repartidor a una entrega.

El lenguaje ubicuo permite que las personas relacionadas con el negocio y el equipo técnico utilicen los mismos términos y comprendan de manera consistente las reglas y responsabilidades del sistema.

Los términos definidos en este documento se utilizarán en el modelo de dominio, los casos de uso, los escenarios de aceptación y los diagramas arquitectónicos.

## 2. Glosario del negocio

### Entrega (Delivery)

Unidad logística que debe ser transportada hasta una dirección de destino. Posee un identificador único y puede tener un repartidor asignado.

### Repartidor (Courier)

Persona responsable de transportar una entrega hasta su destino. Para recibir una asignación debe cumplir las condiciones de disponibilidad definidas por el negocio.

### Asignación (Assignment)

Operación mediante la cual se relaciona una entrega con un repartidor responsable.

### Repartidor asignado

Repartidor asociado a una entrega mediante una asignación válida.

### Disponibilidad

Condición que determina si un repartidor puede recibir una nueva asignación en el momento de la consulta.

La disponibilidad se consultará mediante un puerto para evitar que el caso de uso dependa directamente de un servicio externo.

### Entrega pendiente de asignación

Entrega que existe en el sistema y que todavía no tiene un repartidor asignado.

### Entrega asignada

Entrega que tiene un repartidor asociado y cuya asignación continúa activa.

### Operador logístico

Actor que solicita la asignación de un repartidor a una entrega.

### Caso de uso (Use Case)

Unidad de la aplicación que coordina una operación del negocio. En este flujo, AssignCourierUseCase coordina la asignación de un repartidor.

### Entidad

Objeto del dominio que posee identidad propia y representa un concepto del negocio. Delivery es la entidad principal del flujo modelado.

### Objeto de valor

Objeto definido por su significado y sus atributos, sin identidad independiente. DeliveryId y DeliveryAddress son ejemplos del modelo propuesto.

### Invariante

Regla que debe cumplirse para conservar la validez del dominio. Por ejemplo, una entrega no puede tener dos repartidores activos simultáneamente dentro del alcance definido.

### Puerto

Contrato que define una capacidad que necesita la aplicación sin especificar cómo se implementa técnicamente.

### Adaptador

Componente que conecta un contrato con una tecnología concreta, como una base de datos o un servicio externo.

### Repositorio

Contrato utilizado para consultar y guardar entidades del dominio. En este diseño, IDeliveryRepository representa las operaciones de acceso a las entregas.

### Infraestructura

Capa que contiene las implementaciones técnicas de los contratos, las conexiones externas y los mecanismos de persistencia.

## 3. Términos técnicos del modelo

| Término | Significado |
|---|---|
| DeliveryId | Identificador de una entrega. |
| CourierId | Identificador de un repartidor. |
| DeliveryAddress | Dirección asociada a la entrega. |
| AssignCourierUseCase | Caso de uso que coordina la asignación. |
| IDeliveryRepository | Contrato para consultar y guardar entregas. |
| Puerto de disponibilidad | Contrato para consultar si un repartidor puede recibir una asignación. |

## 4. Términos ambiguos que deben confirmarse

### 4.1. Entrega activa

Debe confirmarse qué condiciones determinan que una entrega esté activa. En esta propuesta, el concepto se limita a una entrega que todavía admite el proceso de asignación.

### 4.2. Repartidor disponible

Debe confirmarse si la disponibilidad depende únicamente de que el repartidor no tenga otra entrega asignada o si también depende de su jornada laboral, capacidad, ubicación y restricciones de ruta.

### 4.3. Asignación activa

Debe confirmarse cuándo una asignación deja de estar activa. Por ejemplo, podría terminar al cancelar la asignación o al completar la entrega, pero esos comportamientos quedan fuera del alcance actual.

### 4.4. Reasignación

Debe confirmarse si una entrega puede cambiar de repartidor y qué autorizaciones, motivos e historial serían necesarios. La reasignación no se contempla en el flujo inicial.

### 4.5. Confirmación de la asignación

Debe definirse si una asignación se considera confirmada inmediatamente después de guardar la entrega o si requiere una confirmación adicional del repartidor.

### 4.6. Capacidad del repartidor

Debe confirmarse si el repartidor puede recibir varias entregas simultáneamente y qué restricciones de capacidad o ruta deben aplicarse.

## 5. Reglas terminológicas

Para mantener la consistencia de la documentación se aplicarán las siguientes reglas:

1. Se utilizará el término entrega para referirse a Delivery.
2. Se utilizará repartidor para referirse a Courier.
3. Se utilizará asignación para describir la relación operativa entre una entrega y su repartidor.
4. Se utilizará disponibilidad para expresar la posibilidad de recibir una nueva asignación.
5. Se utilizará caso de uso para describir la coordinación de una operación de la aplicación.
6. Se utilizará puerto para los contratos que definen capacidades requeridas por la aplicación.
7. Se utilizará adaptador para las implementaciones técnicas que conectan esos contratos con sistemas externos.

## 6. Resultado esperado

Este glosario establece un vocabulario común para el diseño del proceso de asignación.

Los términos ambiguos quedan identificados como supuestos pendientes de confirmación y no se presentan como reglas definitivas del negocio.

El vocabulario deberá mantenerse consistente con el modelo de dominio, los contratos, los escenarios de aceptación y los diagramas de arquitectura.