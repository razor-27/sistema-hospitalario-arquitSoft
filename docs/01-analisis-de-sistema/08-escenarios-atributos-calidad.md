# Escenarios de atributos de calidad


## EQ-01. Disponibilidad: continuidad de los módulos críticos (AC01)

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Disponibilidad |
| **Fuente** | Médico, enfermería o personal de admisión |
| **Estímulo** | Intenta consultar la HCE, registrar un triaje o admitir un paciente mientras falla una instancia, un nodo o una zona completa. |
| **Condición** | Sistema en producción, desplegado en las zonas A y B, con monitoreo y health-check activos. |
| **Respuesta esperada** | El balanceador redirige el tráfico a las instancias o a la zona sanas; la operación se completa sin intervención manual. Los módulos no críticos pueden degradarse sin afectar a los críticos. |
| **Medida** | Módulos críticos (HCE, Triaje/Emergencia, Admisión): **99.99 % anual** (≈ 52 min/año). Módulos no críticos (reportes/BI, notificaciones): **99.9 % anual** (≈ 8.8 h/año). |
| **Verificación** | Revisar los registros de monitoreo, calcular los periodos de indisponibilidad y ejecutar pruebas de falla de instancia y de zona. |
| **Estado** | Pendiente de definir las condiciones de mantenimiento programado y el entorno de pruebas multi-zona. |

## EQ-02. Rendimiento: consulta de HCE y registro de triaje (AC02)

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Rendimiento |
| **Fuente** | Médico o enfermería |
| **Estímulo** | Consulta la historia clínica de un paciente o registra un triaje. |
| **Condición** | Carga pico de diseño (~14,000 usuarios concurrentes) con datos representativos. |
| **Respuesta esperada** | El sistema devuelve el historial o confirma el triaje sin errores inesperados, priorizando el tráfico de Triaje/Emergencia sobre el de módulos no críticos. |
| **Medida** | Consulta de HCE: **< 2 s (p95)**. Registro de triaje: **< 1 s (p95)**. |
| **Verificación** | Pruebas de rendimiento con la carga del escenario pico; registrar tiempos de respuesta por percentil, errores y uso de recursos. |
| **Estado** | Pendiente de definir el volumen de datos de prueba y la herramienta de carga. Falta fijar el porcentaje máximo de errores técnicos aceptado. |

## EQ-03. Escalabilidad: pico extraordinario de usuarios (AC03)

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Escalabilidad |
| **Fuente** | Pacientes y personal en turno |
| **Estímulo** | Emergencia masiva, campaña de vacunación o apertura de agenda (7:00 a. m.) que eleva la concurrencia de ~3,000 a ~5,000 (pico diario) o hasta ~14,000 usuarios (pico extraordinario). |
| **Condición** | Autoescalado habilitado, con mínimo 3 réplicas por servicio crítico y por zona. |
| **Respuesta esperada** | El orquestador agrega instancias (primero de Triaje/Emergencia) y las colas priorizadas por criticidad clínica mantienen el servicio sin degradar los tiempos de EQ-02. |
| **Medida** | Soportar **5,000 concurrentes (pico diario)** y **14,000 (pico extraordinario)** cumpliendo las medidas de EQ-02. |
| **Verificación** | Aumentar progresivamente la carga hasta el escenario de diseño; comparar tiempos de respuesta, errores, número de réplicas y uso de CPU y memoria. |
| **Estado** | Pendiente de medir el tiempo real de incorporación de nuevas instancias (la propuesta indica "segundos") y de definir las reglas de escalado. |

## EQ-04. Seguridad: acceso a datos de salud (AC04)

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Seguridad y privacidad |
| **Fuente** | Usuario sin sesión, con credenciales inválidas o sin permisos para ese paciente o función |
| **Estímulo** | Intenta consultar o modificar una historia clínica, o acceder a una función protegida. |
| **Condición** | Pruebas sobre las funciones protegidas de HCE, Triaje y Admisión. |
| **Respuesta esperada** | El sistema rechaza la operación, no ejecuta el cambio, no expone datos clínicos y registra el intento. Todo acceso permitido queda en la auditoría. |
| **Medida** | **100 %** de los intentos no autorizados de las pruebas definidas rechazados; **100 %** de los accesos a HCE con registro de auditoría inmutable; cifrado TLS 1.3 en tránsito y AES-256 en reposo; MFA para el personal clínico. |
| **Verificación** | Pruebas de acceso autorizado y no autorizado por rol (RBAC), revisión de respuestas, revisión del registro de auditoría y verificación de la configuración de cifrado. |
| **Estado** | Pendiente de definir la matriz de permisos por rol y las reglas de acceso de emergencia ("romper el vidrio"). |

## EQ-05. Recuperación ante desastres (AC05)

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Recuperación ante desastres |
| **Fuente** | Falla de infraestructura |
| **Estímulo** | Se pierde una zona de disponibilidad completa, o ambas zonas al mismo tiempo. |
| **Condición** | Replicación síncrona de la BD de HCE entre zonas y sitio DR en warm standby operativos. |
| **Respuesta esperada** | Si cae una zona, la otra continúa sin pérdida de datos clínicos. Si caen ambas, el sitio DR asume el servicio. |
| **Medida** | Falla de una zona: **RTO < 5 min, RPO ≈ 0**. Pérdida de ambas zonas: **RTO < 2 h, RPO < 15 min**. |
| **Verificación** | Simulacros de falla de zona y de activación del sitio DR; medir el tiempo de recuperación y comprobar la integridad de los datos después del failover. |
| **Estado** | Pendiente de planificar la frecuencia de los simulacros y de validar la restauración de los backups cifrados (retención de 7 años). |

## EQ-06. Mantenibilidad: cambios sin rediseñar el núcleo (AC06)

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Mantenibilidad / Modificabilidad |
| **Fuente** | Equipo de desarrollo |
| **Estímulo** | Cambia una regla de validación clínica, se reemplaza el motor de caché o el bus de mensajería, o se agrega un servicio nuevo (laboratorio, farmacia, telemedicina). |
| **Condición** | Servicios desacoplados, organizados por capas y con pruebas para las funciones afectadas. |
| **Respuesta esperada** | El cambio se concentra en la capa o el servicio responsable, sin modificar los controladores REST ni los demás servicios. |
| **Medida** | Las pruebas del servicio modificado y las de regresión acordadas deben aprobarse; las capas de dominio y aplicación **no importan** librerías de infraestructura (regla de dependencia). |
| **Verificación** | Revisar los cambios de código, ejecutar las pruebas e inspeccionar las importaciones de `dominio/` y `aplicacion/`. |
| **Estado** | Pendiente de definir las pruebas obligatorias y la herramienta que verifique la regla de dependencia en la integración continua. |

## EQ-07. Interoperabilidad: intercambio de información clínica (AC07)

| Elemento | Descripción |
|---|---|
| **Atributo de calidad** | Interoperabilidad |
| **Fuente** | Laboratorio referencial, SIS/aseguradoras, HIS MINSA o SUSALUD |
| **Estímulo** | El sistema envía o recibe información clínica (órdenes y resultados de laboratorio, cobertura, reporte de atenciones). |
| **Condición** | Integraciones configuradas con cada entidad externa. |
| **Respuesta esperada** | El intercambio usa mensajes HL7/FHIR válidos; si el sistema externo no responde, el servicio reintenta y no bloquea la atención. |
| **Medida** | **100 %** de los mensajes de las pruebas conformes con los perfiles HL7/FHIR acordados; la indisponibilidad de un sistema externo no impide registrar la atención. |
| **Verificación** | Validar los mensajes contra los perfiles FHIR y simular la caída de cada sistema externo. |
| **Estado** | Pendiente de confirmar con cada entidad los perfiles FHIR y el protocolo de integración que realmente ofrece. |

## Resumen de escenarios

| Código | Atributo | Qué se busca comprobar | Driver |
|---|---|---|---|
| EQ-01 | Disponibilidad | Que los módulos críticos operen 24/7 aun con fallas. | DA01 |
| EQ-02 | Rendimiento | Que HCE y triaje respondan en 2 s y 1 s (p95) bajo carga pico. | DA03 |
| EQ-03 | Escalabilidad | Que el sistema absorba hasta ~14,000 concurrentes. | DA02 |
| EQ-04 | Seguridad | Que los datos de salud solo los acceda quien tiene permiso y quede auditado. | DA04 |
| EQ-05 | Recuperación ante desastres | Que no se pierdan datos clínicos al caer una zona o el centro de datos. | DA05 |
| EQ-06 | Mantenibilidad | Que los cambios no obliguen a rediseñar el núcleo. | DA06 |
| EQ-07 | Interoperabilidad | Que el intercambio con terceros siga HL7/FHIR. | DA07 |
