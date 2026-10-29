## Requisitos funcionales

| ID | Requisito funcional |
| --- | --- |
| **RF01** | El sistema debe permitir consultar la disponibilidad de citas por especialidad. |
| **RF02** | El sistema debe permitir registrar, confirmar, reprogramar y cancelar citas médicas mediante portal web, aplicativo móvil y ventanilla de admisión. |
| **RF03** | El sistema debe permitir realizar el check-in del paciente en el momento de la atención. |
| **RF04** | El sistema debe permitir crear y consultar la historia clínica electrónica (HCE) de cada paciente (antecedentes, diagnósticos, tratamientos, resultados de laboratorio e imágenes). |
| **RF05** | El sistema debe permitir actualizar la HCE registrando la trazabilidad de cada cambio. |
| **RF06** | El sistema debe permitir registrar el triaje y la priorización clínica en el módulo de Emergencia, con acceso inmediato al historial del paciente. |
| **RF07** | El sistema debe permitir gestionar la admisión y el alta de pacientes. |
| **RF08** | El sistema debe permitir verificar el seguro/cobertura del paciente con SIS y aseguradoras. |
| **RF09** | El sistema debe enviar notificaciones automáticas a pacientes y personal (recordatorios de cita, resultados de laboratorio disponibles). |
| **RF10** | El sistema debe permitir recibir y consultar resultados de laboratorio e imágenes del paciente. |
| **RF11** | El sistema debe registrar y permitir consultar la auditoría de accesos a la historia clínica. |


## Relación entre HU y requisitos funcionales

| Historia de usuario | Requisitos funcionales relacionados |
| --- | --- |
| **HU01** Agendar cita | RF01, RF02 |
| **HU02** Gestionar mis citas | RF02 |
| **HU03** Citas, check-in y cobertura en ventanilla | RF02, RF03, RF08 |
| **HU04** Admisión y alta | RF07, RF08 |
| **HU05** Consultar y actualizar HCE | RF04, RF05, RF10 |
| **HU06** Triaje y acceso inmediato al historial | RF06, RF04 |
| **HU07** Notificaciones | RF09, RF10 |
| **HU08** Auditar accesos | RF11 |
