# ADR-002: Control de concurrencia en la asignación de entregas

## 1. Estado

Propuesta.

Esta decisión documenta una estrategia de diseño para proteger la consistencia de las asignaciones. La implementación definitiva dependerá de la tecnología de persistencia seleccionada.

## 2. Contexto

La plataforma logística permite asignar un repartidor a una entrega mediante AssignCourierUseCase.

Una de las invariantes del dominio establece que una entrega no puede tener dos repartidores activos asignados simultáneamente.

Sin embargo, dos solicitudes podrían consultar la misma entrega cuando todavía está pendiente de asignación y continuar con sus operaciones casi al mismo tiempo.

Por ejemplo:

1. La solicitud A consulta la entrega 100 y encuentra que no tiene repartidor.
2. La solicitud B consulta la misma entrega y encuentra la misma situación.
3. La solicitud A asigna al repartidor 15.
4. La solicitud B asigna al repartidor 23.
5. Ambas solicitudes intentan guardar sus cambios.

Si no existe un mecanismo de control de concurrencia, el sistema podría aceptar resultados incompatibles o sobrescribir una asignación válida.

Por esta razón, la regla de una única asignación activa debe protegerse también durante la persistencia.

## 3. Decisión propuesta

Se propone utilizar un mecanismo de control de concurrencia en la operación de persistencia para impedir que dos solicitudes incompatibles asignen simultáneamente repartidores a la misma entrega.

Como alternativa inicial se evaluará el control de concurrencia optimista mediante una versión de la entidad o un mecanismo equivalente proporcionado por la tecnología de almacenamiento.

El repositorio deberá detectar cuando una operación intenta guardar una versión desactualizada de una entrega.

Cuando se detecte un conflicto, la operación deberá rechazarse y comunicar que la asignación no pudo completarse porque el estado de la entrega cambió.

La estrategia concreta deberá seleccionarse cuando se conozcan la base de datos y sus capacidades.

## 4. Alternativas consideradas

### Alternativa A. Validación únicamente en el caso de uso

El caso de uso consulta si la entrega está asignada y realiza la operación si encuentra que está pendiente.

Ventajas:

- Implementación inicial sencilla.
- Fácil comprensión del flujo.

Desventajas:

- No evita por sí sola las solicitudes simultáneas.
- Dos operaciones pueden superar la validación antes de que alguna guarde los cambios.
- La invariante podría incumplirse ante concurrencia.

Esta alternativa no es suficiente como único mecanismo de protección.

### Alternativa B. Control de concurrencia optimista

Cada entrega dispone de una versión o mecanismo equivalente para detectar modificaciones concurrentes.

Cuando una solicitud guarda la entrega, el almacenamiento comprueba que el estado no haya cambiado desde su lectura.

Si otra solicitud modificó previamente la entrega, la operación detecta el conflicto y no debe sobrescribir silenciosamente el cambio anterior.

Ventajas:

- Detecta conflictos sin bloquear necesariamente las lecturas.
- Puede ser apropiado para operaciones con conflictos relativamente poco frecuentes.
- Permite comunicar y gestionar explícitamente los conflictos.

Desventajas:

- Requiere soporte de la persistencia o una implementación equivalente.
- El caso de uso debe manejar los conflictos.
- La estrategia debe probarse con solicitudes simultáneas reales.

### Alternativa C. Bloqueos o transacciones con aislamiento adecuado

La operación utiliza mecanismos de bloqueo o transacciones para coordinar las modificaciones concurrentes.

Ventajas:

- Puede proteger las modificaciones incompatibles.
- Permite establecer garantías de consistencia según las capacidades del almacenamiento.

Desventajas:

- Puede incrementar la contención y reducir la concurrencia.
- Su comportamiento depende de la tecnología elegida.
- Requiere definir cuidadosamente el alcance y la duración de las transacciones.

## 5. Justificación

Se propone evaluar primero el control de concurrencia optimista porque permite detectar conflictos de actualización sin introducir bloqueos prolongados de manera predeterminada.

Esta elección es provisional. No se afirma que sea la mejor alternativa para cualquier base de datos o carga de trabajo.

La decisión definitiva deberá considerar las capacidades de persistencia, la frecuencia esperada de conflictos y los requisitos de consistencia del negocio.

Independientemente de la tecnología seleccionada, la solución deberá impedir que dos solicitudes incompatibles terminen guardando asignaciones activas diferentes para la misma entrega.

## 6. Consecuencias positivas

- Se reduce el riesgo de asignaciones incompatibles.
- Los conflictos entre solicitudes se detectan explícitamente.
- La invariante del dominio cuenta con una protección adicional durante la persistencia.
- Se pueden diseñar pruebas específicas para operaciones simultáneas.
- El caso de uso puede comunicar los conflictos sin informar un éxito inexistente.

## 7. Consecuencias y riesgos

- El repositorio debe soportar o implementar el mecanismo elegido.
- Las solicitudes concurrentes pueden terminar rechazadas y requerir una nueva consulta.
- La estrategia debe considerar posibles reintentos.
- La comprobación de disponibilidad del repartidor también puede quedar desactualizada.
- El control sobre una entrega no garantiza por sí solo que el repartidor no haya recibido otra asignación simultánea en una entrega diferente.

## 8. Alcance de la protección

La decisión protege la regla de que una misma entrega no tenga dos repartidores activos asignados simultáneamente.

No resuelve automáticamente todas las restricciones relacionadas con los repartidores.

Si el negocio exige que un repartidor no supere cierta cantidad de entregas activas o no reciba asignaciones incompatibles, deberán definirse invariantes adicionales y mecanismos de coordinación adecuados.

## 9. Impacto en los contratos

IDeliveryRepository deberá permitir que la aplicación identifique los conflictos de persistencia relevantes.

El resultado de SaveAsync tendrá que distinguir una operación guardada correctamente de una operación rechazada por conflicto.

AssignCourierUseCase deberá traducir ese resultado a una respuesta coherente para el solicitante.

La entidad Delivery continuará protegiendo las reglas de asignación a nivel de dominio, mientras que el mecanismo de persistencia proporcionará la protección necesaria frente a actualizaciones concurrentes.

## 10. Pruebas previstas

Durante la Actividad 4 se deberán realizar las siguientes pruebas:

1. Dos solicitudes intentan asignar repartidores diferentes a la misma entrega.
2. La primera solicitud guarda una asignación válida.
3. La segunda solicitud detecta el conflicto y no sobrescribe la asignación existente.
4. La respuesta de la segunda solicitud indica que la operación no pudo completarse.
5. El estado final conserva una única asignación activa.
6. Las solicitudes secuenciales siguen funcionando correctamente.
7. Los conflictos no se convierten en respuestas de éxito.

## 11. Aspectos pendientes

Antes de implementar esta decisión será necesario confirmar:

- La tecnología de persistencia.
- El soporte disponible para concurrencia optimista o transacciones.
- El mecanismo para representar las versiones de las entregas.
- La estrategia de reintentos.
- Las reglas adicionales relacionadas con la disponibilidad y la capacidad del repartidor.

## 12. Resultado esperado

La decisión establece el requisito de proteger las asignaciones frente a operaciones simultáneas.

Su finalidad es evitar que las condiciones verificadas por el caso de uso queden invalidadas por cambios concurrentes antes de guardar la entrega.

La estrategia tecnológica definitiva se documentará cuando se conozcan las restricciones reales de implementación.