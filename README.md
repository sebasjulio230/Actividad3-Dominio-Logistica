# Actividad 3 — Modelo de dominio de una plataforma logística

## 1. Descripción del proyecto

Este proyecto documenta el diseño del modelo de dominio de una plataforma logística de última milla, centrándose en el proceso de asignación de un repartidor a una entrega.

El propósito es establecer una base arquitectónica clara para desarrollar y validar el flujo en la Actividad 4.

La propuesta identifica los conceptos principales del negocio, las entidades, los objetos de valor, las invariantes, los casos de uso, los contratos y los límites entre los componentes.

## 2. Objetivo

Diseñar un modelo de dominio que permita asignar un repartidor a una entrega respetando las reglas del negocio y manteniendo la lógica central independiente de los detalles técnicos de infraestructura.

El diseño busca facilitar la implementación posterior, la validación mediante escenarios de aceptación y la evolución de la solución.

## 3. Flujo de negocio seleccionado

El flujo seleccionado es la asignación de un repartidor a una entrega mediante el caso de uso `AssignCourierUseCase`.

El proceso propuesto comprende:

1. Recibir los identificadores de la entrega y del repartidor.
2. Consultar la entrega.
3. Comprobar que la entrega exista y no tenga una asignación activa.
4. Consultar la disponibilidad del repartidor.
5. Ejecutar la asignación mediante la entidad `Delivery`.
6. Guardar los cambios mediante el contrato del repositorio.
7. Comunicar el resultado de la operación.

La propuesta contempla tanto los escenarios exitosos como los errores de negocio, los fallos técnicos y los posibles conflictos de concurrencia.

## 4. Arquitectura propuesta

El diseño separa las responsabilidades en tres áreas principales.

### Dominio

Contiene la entidad `Delivery`, los objetos de valor y las reglas que protegen la consistencia de la asignación.

### Aplicación

Contiene `AssignCourierUseCase` y los contratos que necesita para coordinar el proceso.

### Infraestructura

Contiene las implementaciones técnicas de los contratos de persistencia, consulta de disponibilidad y, si se confirma su necesidad, publicación o registro de resultados.

Se propone utilizar puertos y adaptadores para mantener la aplicación independiente de las tecnologías concretas.

## 5. Estructura del proyecto

La documentación está organizada de la siguiente manera:

- `docs/modelo-dominio.md`: entidades, objetos de valor, invariantes y estados.
- `docs/lenguaje-ubicuo.md`: vocabulario común y términos ambiguos.
- `docs/escenarios-aceptacion.md`: escenarios de aceptación en formato Dado-Cuando-Entonces.
- `docs/casos-de-uso.md`: flujo principal, alternativas, entradas y resultados.
- `docs/contratos-y-puertos.md`: contratos de aplicación y adaptadores propuestos.
- `docs/Architecture.md`: descripción de la arquitectura y separación de responsabilidades.
- `docs/diagramas/modelo-entidades.mmd`: representación del modelo de entidades.
- `docs/diagramas/componentes.mmd`: relaciones entre las capas y los componentes.
- `docs/diagramas/arquitectura-c4.mmd`: representación de la arquitectura lógica.
- `docs/adr/ADR-001-puertos-y-adaptadores.md`: decisión sobre la separación entre contratos e infraestructura.
- `docs/adr/ADR-002-control-concurrencia.md`: decisión propuesta para proteger las asignaciones frente a solicitudes simultáneas.
- `evidencias/`: carpeta destinada a las capturas y los resultados visuales que se recopilen durante la validación.

## 6. Decisiones arquitectónicas

### ADR-001: Puertos y adaptadores

Se propone que el caso de uso dependa de contratos, en lugar de depender directamente de una base de datos o de un servicio externo.

Esta decisión favorece la separación de responsabilidades y facilita las pruebas mediante dependencias simuladas.

### ADR-002: Control de concurrencia

Se establece la necesidad de impedir que dos solicitudes incompatibles generen asignaciones activas diferentes para una misma entrega.

Se propone evaluar el control de concurrencia optimista, aunque la estrategia definitiva dependerá de la tecnología de persistencia seleccionada.

## 7. Reglas principales del dominio

Las reglas centrales del flujo son:

1. La entrega debe existir antes de asignar un repartidor.
2. La entrega no puede tener dos repartidores activos simultáneamente.
3. El repartidor debe estar disponible para recibir la asignación.
4. La entidad `Delivery` debe proteger las reglas de asignación.
5. La aplicación debe utilizar contratos para consultar y guardar las entregas.
6. Los fallos de validación, disponibilidad o persistencia no deben presentarse como operaciones exitosas.
7. La implementación debe contemplar los conflictos derivados de solicitudes simultáneas.

## 8. Escenarios de aceptación

La documentación contempla los siguientes casos:

- Asignación exitosa.
- Entrega inexistente.
- Entrega ya asignada.
- Repartidor no disponible.
- Identificador de repartidor inválido.
- Intento de asignación duplicada.
- Fallo al guardar la entrega.
- Imposibilidad de verificar la disponibilidad.
- Dos solicitudes simultáneas sobre una misma entrega.

Estos escenarios servirán como base para las pruebas que se desarrollen en la Actividad 4.

## 9. Diagramas

Los diagramas se encuentran en `docs/diagramas`.

Los archivos `.mmd` contienen el código fuente Mermaid. Pueden visualizarse mediante un editor compatible con Mermaid, como [Mermaid Live Editor](https://mermaid.live/).

Las imágenes exportadas se podrán incorporar a la carpeta `evidencias` cuando se hayan generado y verificado.

## 10. Alcance y limitaciones

Este repositorio contiene una propuesta de diseño y documentación arquitectónica.

No incluye todavía una implementación funcional del caso de uso, una base de datos configurada, integraciones reales con servicios externos ni pruebas automatizadas ejecutadas.

Los mecanismos de persistencia, disponibilidad y control de concurrencia deberán concretarse durante la implementación.

## 11. Validación prevista

Durante la Actividad 4 se espera:

1. Implementar los contratos y el caso de uso.
2. Aplicar las reglas de dominio.
3. Implementar o simular los adaptadores.
4. Ejecutar pruebas sobre los escenarios de aceptación.
5. Validar el comportamiento frente a errores y solicitudes concurrentes.
6. Verificar que las dependencias respeten los límites arquitectónicos propuestos.

## 12. Resultado esperado

El proyecto proporciona una propuesta documentada del modelo de dominio y de la arquitectura del flujo de asignación de repartidores.

Su finalidad es facilitar la construcción y validación posterior de la solución, manteniendo claras las reglas del negocio, las responsabilidades de los componentes y las decisiones arquitectónicas.