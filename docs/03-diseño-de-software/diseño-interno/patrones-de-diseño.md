# Patrones de diseño

## Objetivo

Documentar los patrones aplicados en los servicios del sistema hospitalario, dónde se usan, qué problema resuelven y con qué se implementan, para que el equipo programe de forma uniforme.

## Contexto

- **Estilo:** microservicios en múltiples zonas activas con sitio DR (ADR-001, ADR-002).
- **Enfoque interno:** capas con inversión de dependencias (ADR-008).
- **Base:** diagrama de componentes (C4 nivel 3).

## Patrones aplicados

| Patrón | Dónde se usa | Problema que resuelve | Se implementa con | Elementos del diseño |
|---|---|---|---|---|
| Adapter | Citas y Admisión → SIS / Aseguradoras | Cada aseguradora tiene su propia interfaz | API del proveedor (HTTPS) | `CoberturaPort`, `SisAdapter` |
| Adapter | HCE → Laboratorio referencial | El laboratorio tiene su propio formato | HL7/FHIR sobre HTTPS | `LaboratorioPort`, `LaboratorioFhirAdapter` |
| Adapter | Soporte → HIS MINSA, SUSALUD | Cada entidad exige su propio formato de reporte | API de cada entidad | `ReporteSanitarioPort` y un adaptador por entidad |
| Adapter | Soporte → Plataforma SMS / Correo | El proveedor de mensajería puede cambiar | API del proveedor | `NotificacionPort`, `SmsAdapter` |
| Repository | Todos los servicios → Base de datos | El dominio no debe conocer SQL | Driver de BD u ORM | `ConsultaRepository`, `PostgresConsultaRepository` |
| Decorator | HCE → Caché | Consultar más rápido sin tocar el repositorio | Redis | `CachedPacienteRepository` |
| Publish/Subscribe | Servicios → Bus de eventos | Evitar acoplamiento con laboratorio, farmacia y notificaciones | Kafka o RabbitMQ (por definir) | `EventPublisher`, `BrokerEventPublisher` |
| Circuit Breaker | Adaptadores de sistemas externos | La caída de un tercero no debe bloquear la atención | Librería de resiliencia | Dentro de cada adaptador |
| API Gateway | Entrada a todos los servicios | Autenticación y acceso centralizados | Gateway con SSO | Nivel 2 (ADR-007) |

## Cómo funciona cada patrón

| Patrón | En pocas palabras |
|---|---|
| Adapter | El sistema dice `verificarCobertura(paciente)` y el adaptador lo convierte en la llamada propia del SIS o de la aseguradora. |
| Repository | El caso de uso dice `guardar(consulta)`; el repositorio escribe el SQL. |
| Decorator | Busca primero en Redis; si no está, consulta la BD y lo guarda en Redis para la próxima vez. |
| Publish/Subscribe | HCE publica `ConsultaRegistrada` sin saber quién la consume; Laboratorio, Farmacia y Notificaciones se suscriben. |
| Circuit Breaker | Si un sistema externo falla repetidamente, se deja de llamarlo por un tiempo y se responde de forma controlada. |

## Beneficio

- Cambiar el laboratorio, el proveedor de SMS o la BD solo afecta al adaptador o al repositorio.
- La caché se puede activar o retirar sin tocar las reglas clínicas.
- Un servicio nuevo (farmacia, telemedicina) se suscribe a los eventos sin modificar a los demás.

## Patrones previstos para siguientes iteraciones

| Patrón | Dónde se aplicaría | Cuándo |
|---|---|---|
| Outbox | HCE → Bus de eventos | Para no perder eventos si falla la publicación después de guardar |
| Saga | Admisión → verificación de cobertura → alta | Si un proceso abarca varios servicios y necesita compensaciones |
| CQRS | Reportes / Data Warehouse | Si las consultas de reportes afectan el rendimiento de los servicios clínicos |
