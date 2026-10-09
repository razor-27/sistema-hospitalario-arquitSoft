## Atributos de calidad

| ID | Atributo de calidad | Escenario de calidad |
| --- | --- | --- |
| **AC01** | Disponibilidad | Los módulos críticos (HCE, Triaje/Emergencia, Admisión) deben operar 24/7 con una disponibilidad de 99.99% anual (≈ 52 min/año), incluso ante la caída de una zona completa. Los módulos no críticos (reportes/BI, notificaciones) deben alcanzar 99.9% anual. |
| **AC02** | Rendimiento | La consulta de HCE debe responder en menos de 2 s (percentil 95) y el registro de triaje en menos de 1 s (percentil 95), incluso bajo carga pico (~14,000 usuarios concurrentes). |
| **AC03** | Escalabilidad | El sistema debe soportar el crecimiento de pacientes registrados y picos puntuales (hasta ~14,000 concurrentes en emergencia masiva) mediante autoescalado horizontal, con un mínimo de 3 réplicas por servicio crítico y zona. |
| **AC04** | Seguridad y privacidad | Los datos de salud deben protegerse en tránsito (TLS 1.3) y en reposo (AES-256), con autenticación multifactor para el personal clínico, control de acceso por roles (RBAC), auditoría inmutable y cumplimiento de la Ley N.° 29733. |
| **AC05** | Recuperación ante desastres | Ante la falla de una zona, el sistema debe recuperarse en menos de 5 min (RTO) con RPO ≈ 0. Ante la pérdida de ambas zonas, el sitio DR debe recuperarse en menos de 2 h (RTO) con RPO menor a 15 min. |
| **AC06** | Mantenibilidad / Modificabilidad | El sistema debe estar organizado en servicios desacoplados que permitan incorporar nuevos servicios (laboratorio, farmacia, telemedicina) y cambiar reglas o componentes de infraestructura sin rediseñar el núcleo. |
| **AC07** | Interoperabilidad | El sistema debe intercambiar información clínica con laboratorio, farmacia y entidades del sistema de salud siguiendo el estándar HL7/FHIR. |

## Estimación de carga y usuarios

### Población de usuarios

| Segmento de usuario | Cantidad | % del total | Observación |
| --- | --- | --- | --- |
| Médicos | 450 | 2.3% | Acceso prioritario a HCE y Triaje |
| Enfermería | 750 | 3.7% | Uso intensivo en turnos de 12 h |
| Técnicos (laboratorio, imágenes, farmacia) | 400 | 2.0% | Integración vía bus de eventos |
| Personal administrativo / Admisión | 350 | 1.8% | Módulo de Citas y Admisión |
| Soporte / otros | 350 | 1.8% | Reportes y back-office |
| Pacientes con cuenta activa (portal/app) | 17,700 | 88.5% | Agendamiento y consulta de resultados |
| **TOTAL DE USUARIOS REGISTRADOS** | **20,000** | **100%** | Base para el dimensionamiento |

### Escenarios de concurrencia

| Escenario | Usuarios concurrentes | % del total registrado | Contexto típico |
| --- | --- | --- | --- |
| Operación normal (fuera de turno pico) | ~1,200 | 6% | Madrugada / fin de semana |
| Turno de mayor actividad (7:00–13:00 h) | ~3,000 | 15% | Consulta externa habitual |
| Pico diario (apertura de agenda 7:00 a.m.) | ~5,000 | 25% | Ráfaga de 10–15 min |
| **Pico extraordinario** (emergencia masiva, campaña de vacunación, contingencia sanitaria) | **hasta ~14,000** | ~90% del personal en turno + oleada de pacientes | **Escenario de diseño (peor caso)** |
