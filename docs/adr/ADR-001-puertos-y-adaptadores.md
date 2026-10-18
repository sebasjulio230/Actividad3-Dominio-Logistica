# ADR-001: Uso de puertos y adaptadores para el flujo de asignación

## 1. Estado

Propuesta.

Esta decisión se documenta como parte del diseño de la Actividad 3. Su implementación y validación se realizarán posteriormente.

## 2. Contexto

La plataforma logística necesita asignar un repartidor a una entrega mediante el caso de uso AssignCourierUseCase.

Durante este proceso, la aplicación debe consultar una entrega, verificar la disponibilidad de un repartidor, ejecutar las reglas del dominio y guardar el resultado.

Estas operaciones pueden requerir comunicación con bases de datos y servicios externos.

Si el caso de uso dependiera directamente de las tecnologías concretas utilizadas para estas operaciones, cualquier cambio de proveedor o mecanismo de persistencia podría obligar a modificar la lógica de la aplicación.

Además, la dependencia directa de servicios externos dificultaría la realización de pruebas unitarias aisladas.

Por estas razones, es necesario definir límites claros entre el dominio, la aplicación y la infraestructura.

## 3. Decisión

Se propone utilizar una arquitectura basada en puertos y adaptadores, aplicando el principio de inversión de dependencias.

La aplicación dependerá de contratos que representen las capacidades necesarias para completar el flujo de asignación.

Se definen los siguientes contratos:

- IDeliveryRepository: permite consultar y guardar entregas.
- ICourierAvailability: permite verificar la disponibilidad de un repartidor.
- IAssignmentResultPublisher: representa la capacidad de registrar o publicar el resultado de una asignación, pendiente de confirmar según las necesidades del negocio.

Las implementaciones concretas de estos contratos se ubicarán en la capa de infraestructura.

La entidad Delivery conservará las reglas de negocio relacionadas con la asignación.

## 4. Alternativas consideradas

### Alternativa A. Dependencia directa de la infraestructura

El caso de uso podría comunicarse directamente con una base de datos y con los servicios externos necesarios.

Ventajas:

- Menor cantidad inicial de abstracciones.
- Posible simplicidad para un prototipo pequeño.

Desventajas:

- Mayor acoplamiento tecnológico.
- Más dificultad para sustituir proveedores.
- Pruebas unitarias más dependientes de servicios externos.
- Mayor riesgo de mezclar reglas de negocio con detalles técnicos.

### Alternativa B. Puertos y adaptadores

El caso de uso utiliza contratos y la infraestructura implementa las operaciones técnicas.

Ventajas:

- Separación de responsabilidades.
- Mayor facilidad para probar el caso de uso.
- Posibilidad de cambiar implementaciones sin modificar las reglas centrales.
- Dependencias orientadas hacia los contratos de la aplicación.

Desventajas:

- Requiere definir y mantener interfaces.
- Puede introducir abstracciones adicionales que deben justificarse.
- Exige diseñar cuidadosamente los contratos y el tratamiento de errores.

## 5. Justificación

Se selecciona la alternativa B como propuesta de diseño porque permite aislar las reglas del negocio de las tecnologías externas.

Esta decisión es adecuada para el flujo de asignación porque intervienen operaciones de consulta, validación, persistencia y disponibilidad que pueden requerir implementaciones diferentes.

Los contratos también permiten sustituir las dependencias reales por implementaciones simuladas durante las pruebas.

La propuesta mantiene la entidad Delivery como responsable de proteger la regla de una única asignación activa y deja al caso de uso la coordinación de las operaciones.

## 6. Consecuencias positivas

- El dominio puede diseñarse sin depender de una base de datos específica.
- El caso de uso puede probarse con dependencias simuladas.
- Los adaptadores pueden cambiarse sin alterar necesariamente el dominio.
- Las responsabilidades de cada capa quedan documentadas.
- Los errores técnicos pueden tratarse mediante contratos definidos.

## 7. Consecuencias y riesgos

- Es necesario definir correctamente los contratos y sus resultados.
- Los adaptadores deben traducir los resultados técnicos al lenguaje de la aplicación.
- La separación arquitectónica no garantiza por sí sola la consistencia ante solicitudes simultáneas.
- Debe definirse cómo se gestionan los fallos de persistencia y los posibles reintentos.
- La publicación de resultados no debe comprometer la consistencia de una asignación ya guardada.

## 8. Riesgos pendientes

Se deberá determinar cómo impedir que dos solicitudes simultáneas generen asignaciones incompatibles para una misma entrega.

También se deberá confirmar la fuente real de disponibilidad de los repartidores y decidir si es necesario publicar eventos o si basta con registrar el resultado.

Estas decisiones dependerán de las restricciones del negocio y de la arquitectura previamente acordada.

## 9. Impacto en la implementación

Durante la Actividad 4 se deberá:

1. Definir los contratos de aplicación.
2. Implementar el caso de uso AssignCourierUseCase.
3. Mantener las reglas de asignación en Delivery.
4. Implementar o simular los adaptadores necesarios.
5. Crear pruebas para los escenarios exitosos y fallidos.
6. Verificar que las dependencias respeten los límites arquitectónicos definidos.

## 10. Resultado esperado

La decisión establece una separación explícita entre las reglas del negocio y los detalles técnicos.

Su finalidad es facilitar la evolución, la comprobación y la implementación del flujo de asignación sin introducir dependencias tecnológicas innecesarias en el dominio.