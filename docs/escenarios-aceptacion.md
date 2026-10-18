# Escenarios de aceptación — Asignación de repartidor

## 1. Propósito

Este documento define los escenarios de aceptación del flujo de asignación de un repartidor a una entrega en una plataforma logística de última milla.

Los escenarios utilizan el formato Dado-Cuando-Entonces para describir las condiciones iniciales, la acción ejecutada y el resultado esperado.

Su finalidad es establecer criterios verificables que sirvan como base para las pruebas de la Actividad 4.

## 2. Escenario 1: asignación exitosa

**Dado** que existe una entrega pendiente de asignación y el repartidor seleccionado está disponible.

**Cuando** el operador solicita asignar ese repartidor a la entrega.

**Entonces** el sistema debe asociar el identificador del repartidor a la entrega, guardar el resultado y comunicar que la operación fue exitosa.

## 3. Escenario 2: entrega inexistente

**Dado** que el identificador proporcionado no corresponde a una entrega existente.

**Cuando** el operador solicita asignar un repartidor a esa entrega.

**Entonces** el sistema debe rechazar la operación, informar que la entrega no existe y evitar guardar una asignación.

## 4. Escenario 3: entrega ya asignada

**Dado** que existe una entrega que ya tiene un repartidor activo asignado.

**Cuando** el operador intenta asignarle otro repartidor.

**Entonces** el sistema debe rechazar la solicitud, conservar la asignación existente y comunicar que la entrega ya está asignada.

## 5. Escenario 4: repartidor no disponible

**Dado** que existe una entrega pendiente de asignación y el repartidor seleccionado no está disponible.

**Cuando** el operador solicita realizar la asignación.

**Entonces** el sistema debe rechazar la operación, informar que el repartidor no está disponible y mantener la entrega sin una nueva asignación.

## 6. Escenario 5: identificador de repartidor inválido

**Dado** que el identificador del repartidor recibido no cumple el formato o las condiciones de validación establecidas.

**Cuando** se solicita ejecutar el caso de uso de asignación.

**Entonces** el sistema debe rechazar la solicitud antes de completar la asignación y comunicar un error de validación.

La validación concreta dependerá del tipo de identificador y de las reglas que se establezcan durante la implementación.

## 7. Escenario 6: intento de asignación duplicada

**Dado** que una entrega ya tiene un repartidor activo asociado.

**Cuando** se ejecuta nuevamente la operación de asignación sobre esa misma entrega.

**Entonces** la entidad Delivery debe impedir la segunda asignación y proteger la regla de una única asignación activa.

## 8. Escenario 7: fallo al guardar la entrega

**Dado** que la entrega existe, el repartidor está disponible y la operación de asignación ha superado las validaciones.

**Cuando** ocurre un error al guardar los cambios mediante el repositorio.

**Entonces** el caso de uso debe comunicar que la operación no pudo completarse y no debe informar que la asignación quedó guardada correctamente.

La implementación deberá definir cómo se garantiza la consistencia de los cambios ante fallos de persistencia.

## 9. Escenario 8: disponibilidad no verificable

**Dado** que existe una entrega pendiente de asignación.

**Cuando** el servicio encargado de consultar la disponibilidad no puede proporcionar una respuesta válida.

**Entonces** el sistema debe impedir que la asignación se complete sin verificar la disponibilidad y comunicar que la operación no pudo validarse.

## 10. Escenario 9: dos solicitudes simultáneas

**Dado** que una entrega se encuentra pendiente de asignación y dos solicitudes intentan asignar repartidores diferentes.

**Cuando** ambas solicitudes se procesan de forma concurrente.

**Entonces** la solución debe impedir que se guarden dos asignaciones activas incompatibles para la misma entrega.

El mecanismo técnico para garantizar esta condición deberá definirse durante la implementación, teniendo en cuenta las capacidades del repositorio y del sistema de persistencia.

## 11. Relación con las invariantes

| Invariante | Escenarios relacionados |
|---|---|
| La entrega debe existir | Escenario 2 |
| Una entrega no puede tener dos repartidores activos | Escenarios 3, 6 y 9 |
| El repartidor debe estar disponible | Escenarios 1, 4 y 8 |
| Los identificadores deben validarse | Escenario 5 |
| El resultado debe guardarse correctamente | Escenarios 1 y 7 |
| Las reglas deben protegerse frente a solicitudes simultáneas | Escenario 9 |

## 12. Criterios generales de aceptación

El flujo se considerará correctamente diseñado cuando:

1. La asignación exitosa produzca una relación válida entre la entrega y el repartidor.
2. Las solicitudes que incumplan las reglas sean rechazadas.
3. La entidad Delivery proteja la invariante de asignación.
4. El caso de uso coordine la operación a través de contratos.
5. Los errores de disponibilidad y persistencia se comuniquen sin presentar la operación como exitosa.
6. El diseño contemple la consistencia ante solicitudes simultáneas.
7. Los escenarios puedan convertirse en pruebas automatizadas durante la Actividad 4.

## 13. Resultado esperado

Estos escenarios describen el comportamiento esperado del flujo de asignación y proporcionan criterios verificables para validar el modelo de dominio y el caso de uso.

Los escenarios constituyen una especificación de comportamiento y no implican que las pruebas o las integraciones ya estén implementadas.