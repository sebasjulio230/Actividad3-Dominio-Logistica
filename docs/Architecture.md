# Arquitectura de la plataforma logística de última milla

## 1. Objetivo

Este documento describe la arquitectura propuesta para el flujo de asignación de un repartidor a una entrega.

La arquitectura busca separar las reglas del negocio de los detalles técnicos, facilitando la comprensión, las pruebas y la evolución de la solución.

La propuesta constituye un diseño para la Actividad 3. No representa una implementación ya desplegada.

## 2. Estilo arquitectónico

Se propone una arquitectura basada en separación de responsabilidades, inversión de dependencias y puertos y adaptadores.

El diseño distingue tres áreas principales:

- Dominio: contiene las entidades y las reglas del negocio.
- Aplicación: coordina los casos de uso y define los contratos que necesita.
- Infraestructura: implementa los mecanismos técnicos de persistencia y comunicación externa.

Esta organización permite que las reglas de asignación no dependan directamente de una tecnología concreta.

## 3. Componentes principales

### 3.1. Dominio

La entidad Delivery representa una entrega y protege la regla que impide una segunda asignación mientras exista un repartidor activo.

Los objetos de valor DeliveryId y DeliveryAddress representan conceptos propios del dominio.

### 3.2. Aplicación

AssignCourierUseCase coordina el proceso de asignación.

Utiliza el repositorio para consultar y guardar entregas, y el puerto de disponibilidad para verificar si el repartidor puede recibir una asignación.

### 3.3. Infraestructura

La infraestructura implementará los contratos necesarios mediante adaptadores.

Se proponen los siguientes:

- DeliveryRepositoryAdapter.
- CourierAvailabilityAdapter.
- AssignmentResultPublisherAdapter.

Las tecnologías concretas y la necesidad de publicar eventos deberán confirmarse antes de la implementación.

## 4. Flujo arquitectónico

1. El operador solicita la asignación de un repartidor.
2. AssignCourierUseCase recibe los identificadores de la entrega y del repartidor.
3. El caso de uso consulta la entrega mediante IDeliveryRepository.
4. Verifica que la entrega no tenga una asignación activa.
5. Consulta la disponibilidad mediante ICourierAvailability.
6. Solicita a Delivery que ejecute la asignación.
7. Guarda la entrega mediante el repositorio.
8. Comunica el resultado de la operación según el contrato definido.

Si una validación falla, la operación debe detenerse y comunicar el resultado correspondiente.

## 5. Principios de diseño

### Separación de responsabilidades

Cada componente tiene una responsabilidad delimitada. La entidad protege las reglas del negocio, el caso de uso coordina el proceso y los adaptadores manejan la comunicación técnica.

### Inversión de dependencias

La aplicación depende de contratos y no de implementaciones concretas.

### Encapsulamiento de las reglas

La invariante de una única asignación activa se protege en el dominio y no exclusivamente en la interfaz o en el caso de uso.

### Testabilidad

Los contratos permiten sustituir las dependencias externas por implementaciones simuladas durante las pruebas.

### Independencia tecnológica

Las reglas centrales no dependen directamente de una base de datos, un proveedor de mensajería o una API externa.

## 6. Consistencia y concurrencia

Dos solicitudes simultáneas podrían intentar asignar diferentes repartidores a una misma entrega.

Por ello, la implementación deberá incluir un mecanismo que proteja la invariante incluso ante concurrencia. Entre las alternativas que se podrán evaluar se encuentran el control de concurrencia optimista, las transacciones y las restricciones de persistencia apropiadas.

La selección definitiva dependerá de la tecnología de almacenamiento y de las decisiones arquitectónicas previas.

## 7. Manejo de errores

El diseño distingue entre errores de negocio y errores técnicos.

Los errores de negocio incluyen una entrega inexistente, una entrega ya asignada y un repartidor no disponible.

Los errores técnicos incluyen fallos al consultar o guardar datos y problemas al verificar la disponibilidad.

La aplicación no debe comunicar un resultado exitoso cuando no se haya confirmado la persistencia de la asignación.

## 8. Alcance de la propuesta

Esta arquitectura se limita al diseño del flujo de asignación.

No incluye la implementación de bases de datos, servicios externos, interfaces de usuario, cálculo de rutas ni gestión completa de repartidores.

La implementación y validación del flujo se desarrollarán posteriormente.

## 9. Diagramas relacionados

La arquitectura se complementa con los siguientes archivos:

- diagramas/modelo-entidades.mmd
- diagramas/componentes.mmd
- diagramas/arquitectura-c4.mmd

Estos diagramas representan las entidades principales, las relaciones entre componentes y la organización lógica de la solución.

## 10. Resultado esperado

La propuesta establece límites claros entre el dominio, la aplicación y la infraestructura.

Su propósito es servir como base verificable para implementar el caso de uso AssignCourierUseCase y validar sus reglas mediante las pruebas de la Actividad 4.