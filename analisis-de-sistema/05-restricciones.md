| ID | Restricción | Descripción |
| --- | --- | --- |
| **RC01** | Protección de datos personales | El sistema debe cumplir la normativa peruana de protección de datos personales (Ley N.° 29733), dado que los datos de salud son datos sensibles. |
| **RC02** | Interoperabilidad clínica | El intercambio de información clínica debe seguir los estándares HL7/FHIR. |
| **RC03** | Canales de acceso | El sistema debe ser accesible mediante portal web, aplicativo móvil y estaciones clínicas (admisión, consultorios y triaje). |
| **RC04** | API REST | La comunicación síncrona entre clientes y servicios debe realizarse mediante API, expuesta a través de un API Gateway común. |
| **RC05** | Integraciones externas | El sistema debe integrarse con el HIS MINSA, SUSALUD, SIS/aseguradoras, el laboratorio referencial regional y una plataforma de notificaciones SMS/correo. |
| **RC06** | Portabilidad de infraestructura | El despliegue debe basarse en contenedores/Kubernetes para evitar la dependencia de un único proveedor (la selección final de proveedor cloud queda fuera de alcance). |
| **RC07** | Control de versiones | El código fuente debe gestionarse utilizando Git y mantenerse en un repositorio compartido. |
