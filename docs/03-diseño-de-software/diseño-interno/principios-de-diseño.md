# Principios de diseño (SOLID)

## Objetivo

Documentar cómo se aplican los principios SOLID en los servicios del sistema hospitalario, para que cada servicio se programe de forma uniforme y las reglas clínicas no dependan de la tecnología.

## Contexto

- **Estilo:** microservicios (ADR-001).
- **Enfoque interno:** capas con inversión de dependencias (ADR-008).
- **Servicio de referencia:** Historia Clínica Electrónica (HCE), por ser el de mayor criticidad. Integra BD, caché, bus de eventos y laboratorio.

## SOLID en el servicio HCE

| Principio | Dónde se aplica (capa) | Cómo se cumple en el sistema | Archivos |
|---|---|---|---|
| **S** · Responsabilidad única | Todas las capas | El controlador recibe la petición HTTP, el caso de uso coordina, la entidad aplica las reglas clínicas y el repositorio guarda en la BD | `consulta.controller.ts`, `registrar-consulta.use-case.ts`, `consulta.ts`, `postgres-consulta-repository.ts` |
| **O** · Abierto/cerrado | Infraestructura | Para cambiar de Kafka a RabbitMQ o agregar otro destino de eventos se crea un nuevo adaptador; el caso de uso no se modifica | `broker-event-publisher.ts` y nuevos adaptadores |
| **L** · Sustitución de Liskov | Infraestructura | `PostgresPacienteRepository` y `CachedPacienteRepository` cumplen el mismo contrato; el caso de uso funciona igual con cualquiera | `postgres-paciente-repository.ts`, `cached-paciente-repository.ts` |
| **I** · Segregación de interfaces | Dominio | Cada interfaz tiene solo lo que HCE necesita: `AuditoriaPort` solo registra, `EventPublisher` solo publica, `ConsultaRepository` solo guarda y busca consultas | `auditoria-port.ts`, `event-publisher.ts`, `consulta-repository.ts` |
| **D** · Inversión de dependencias | Aplicación | El caso de uso recibe interfaces por el constructor y no conoce PostgreSQL, Redis ni el bus; las implementaciones se conectan en `hce.module.ts` | `registrar-consulta.use-case.ts`, `hce.module.ts` |

## Qué pasaría sin SOLID

| Situación real | Sin SOLID | Con SOLID |
|---|---|---|
| Se cambia el bus de eventos (Kafka → RabbitMQ) | Hay que modificar el caso de uso | Solo se crea o cambia el adaptador |
| Se cambia el motor de caché | Se modifica la lógica de consulta del paciente | Solo se reemplaza el decorator |
| Se quiere probar el registro de consultas | Se necesita BD, Redis y bus reales | Se prueba con implementaciones en memoria |
| Cambia una regla clínica | Hay que buscarla en controladores y servicios | Está solo en la entidad `Consulta` |
| Se agrega auditoría a otra operación | Se copia código en cada controlador | Se reutiliza `AuditoriaPort` |

## Relación con la arquitectura y los patrones

| Principio | Lo cumple en el proyecto |
|---|---|
| Responsabilidad única | La separación en capas (ADR-008) |
| Abierto/cerrado y Liskov | Los patrones Adapter y Decorator |
| Segregación de interfaces | Los puertos del dominio (`AuditoriaPort`, `EventPublisher`, repositorios) |
| Inversión de dependencias | El patrón Repository y la regla de dependencia: las capas externas dependen del dominio |
