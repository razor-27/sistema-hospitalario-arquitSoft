# Diseño interno de módulos

> **Proyecto:** Sistema de Gestión Hospitalaria · **Servicio de referencia:** Historia Clínica Electrónica (HCE)  
> **Enfoque:** arquitectura por capas con inversión de dependencias dentro de cada microservicio (ADR-008)  
> **Nivel C4:** 4 – Código

---

## 1. Propósito

Describe **cómo se organiza el código dentro de un servicio**. Tiene tres objetivos:

- que todos los servicios se construyan con la misma estructura;
- que las reglas clínicas queden aisladas de la tecnología;
- que cualquier integrante sepa dónde va cada archivo y de qué puede depender.

---

## 2. Estructura de carpetas

```text
hce-service/
└── src/
    ├── dominio/
    │   ├── paciente.ts                      Entidad Paciente
    │   ├── consulta.ts                      Entidad Consulta y sus reglas
    │   ├── diagnostico.ts                   Objeto de valor Diagnostico
    │   ├── estado-consulta.ts               Enumeración EstadoConsulta
    │   ├── paciente-repository.ts           Puerto de persistencia de pacientes
    │   ├── consulta-repository.ts           Puerto de persistencia de consultas
    │   ├── auditoria-port.ts                Puerto de auditoría
    │   └── event-publisher.ts               Puerto de publicación de eventos
    ├── aplicacion/
    │   ├── registrar-consulta.use-case.ts   Caso de uso RegistrarConsulta
    │   └── registrar-consulta.command.ts    Datos de entrada
    ├── infraestructura/
    │   ├── postgres-consulta-repository.ts  Implementación con la BD de HCE
    │   ├── postgres-paciente-repository.ts  Implementación con la BD de HCE
    │   ├── cached-paciente-repository.ts    Decorator con Redis
    │   ├── auditoria-postgres.adapter.ts    Registro inmutable de auditoría
    │   └── broker-event-publisher.ts        Publicador al bus (Kafka o RabbitMQ)
    ├── presentacion/
    │   ├── consulta.controller.ts           Traduce HTTP ⇄ caso de uso
    │   └── hce.routes.ts                    Rutas /api/hce/*
    ├── hce.module.ts                        Raíz de composición
    └── index.ts                             Arranque del servicio
```

| Archivo | Capa | Responsabilidad | Importa de |
|---|---|---|---|
| `consulta.ts` | Dominio | Crear consultas válidas, agregar diagnósticos, cerrar | Solo dominio |
| `diagnostico.ts` | Dominio | Código CIE-10 y descripción | Nada |
| `estado-consulta.ts` | Dominio | Estados posibles de la consulta | Nada |
| `paciente-repository.ts` | Dominio | Declarar **qué** necesita para leer pacientes | `paciente.ts` |
| `consulta-repository.ts` | Dominio | Declarar **qué** necesita para guardar consultas | `consulta.ts` |
| `auditoria-port.ts` | Dominio | Declarar **qué** necesita para auditar | Nada |
| `event-publisher.ts` | Dominio | Declarar **qué** necesita para publicar eventos | Nada |
| `registrar-consulta.use-case.ts` | Aplicación | Coordinar los pasos | Dominio |
| `postgres-*-repository.ts` | Infraestructura | Guardar y leer en la BD de HCE | Dominio + driver de BD |
| `cached-paciente-repository.ts` | Infraestructura | Leer primero de Redis | Dominio + cliente Redis |
| `auditoria-postgres.adapter.ts` | Infraestructura | Guardar la auditoría | Dominio + driver de BD |
| `broker-event-publisher.ts` | Infraestructura | Publicar en el bus | Dominio + cliente del bus |
| `consulta.controller.ts` | Presentación | Leer la petición HTTP e invocar el caso de uso | Aplicación |
| `hce.module.ts` | Composición | Crear instancias concretas e inyectarlas | Todas las capas |

---

## 3. Capas y regla de dependencia

**Regla de dependencia:** el código solo puede importar hacia el dominio. El dominio no conoce el framework web, la base de datos, Redis ni el bus.

| Capa | Contiene | Responsabilidad | Puede depender de | **No** puede depender de |
|---|---|---|---|---|
| **Dominio** | Entidades, objetos de valor, interfaces | Reglas clínicas válidas con cualquier tecnología | Nada externo | Aplicación, infraestructura, presentación, framework, BD, Redis |
| **Aplicación** | Casos de uso, comandos | Orquestar el flujo y decidir qué hacer si falla | Dominio | Infraestructura, presentación |
| **Infraestructura** | Repositorios, caché, adaptadores | Implementar las interfaces del dominio | Dominio, librerías externas | Aplicación, presentación |
| **Presentación** | Controladores, rutas | Traducir HTTP a llamadas del caso de uso | Aplicación | Infraestructura, BD |
| **Composición** | `hce.module.ts` | Elegir implementaciones y conectarlas | Todas | — |


---

## 4. Reglas de negocio de ejemplo

| Código | Regla |
|---|---|
| RN-01 | La consulta debe tener paciente y médico. |
| RN-02 | La consulta debe tener motivo de consulta. |
| RN-03 | Todo diagnóstico debe tener un código CIE-10. |
| RN-04 | Una consulta cerrada no se modifica; cualquier corrección se registra como nueva entrada con trazabilidad (RF05). |

---

## 5. Flujo de ejecución: registrar consulta

El diagrama de secuencia está en [`Nivel4-DiagramadeCodigo.md`](/docs/04-modelo-c4/Nivel4-DiagramadeCodigo.md#3-diagrama-de-secuencia-registrar-consulta)

| Paso | Origen → destino | Acción | Capa |
|---|---|---|---|
| 1 | Cliente → `ConsultaController` | `POST /api/hce/consultas` | Presentación |
| 2 | `ConsultaController` → `RegistrarConsultaUseCase` | `ejecutar(cmd)` | Presentación → Aplicación |
| 3 | `RegistrarConsultaUseCase` → `PacienteRepository` | `buscarPorId` (con caché) | Aplicación → Dominio (interfaz) |
| 4 | `RegistrarConsultaUseCase` → `Consulta` | `registrar(...)`; valida RN-01 a RN-04 | Aplicación → Dominio |
| 5 | `RegistrarConsultaUseCase` → `ConsultaRepository` | `guardar(consulta)` | Aplicación → Dominio (interfaz) |
| 6 | `RegistrarConsultaUseCase` → `AuditoriaPort` | `registrar(...)` | Aplicación → Dominio (interfaz) |
| 7 | `RegistrarConsultaUseCase` → `EventPublisher` | `publicar(ConsultaRegistrada)` | Aplicación → Dominio (interfaz) |
| 8 | `ConsultaController` → Cliente | `201 Created` con `consultaId` | Presentación |
| 9 | Alternativa | Si no se cumple una regla, no se guarda y se responde `422` | Aplicación → Presentación |

---

## 6. Código de referencia

### 6.1 Dominio · entidad e interfaces

```ts
// dominio/consulta.ts
import { Diagnostico } from './diagnostico';
import { EstadoConsulta } from './estado-consulta';
import { ReglaClinicaError } from '../../shared/errores';

export class Consulta {
  private constructor(
    readonly id: string,
    readonly pacienteId: string,
    readonly medicoId: string,
    readonly motivo: string,
    private readonly diagnosticos: Diagnostico[],
    private estado: EstadoConsulta,
  ) {}

  static registrar(pacienteId: string, medicoId: string, motivo: string,
                   diagnosticos: Diagnostico[]): Consulta {
    if (!pacienteId || !medicoId) throw new ReglaClinicaError('RN-01: falta paciente o médico');
    if (!motivo?.trim())          throw new ReglaClinicaError('RN-02: falta el motivo de consulta');
    diagnosticos.forEach((d) => {
      if (!d.codigoCie10) throw new ReglaClinicaError('RN-03: el diagnóstico requiere código CIE-10');
    });
    return new Consulta(crypto.randomUUID(), pacienteId, medicoId, motivo,
                        diagnosticos, EstadoConsulta.ABIERTA);
  }

  agregarDiagnostico(d: Diagnostico): void {
    if (this.estado === EstadoConsulta.CERRADA) {
      throw new ReglaClinicaError('RN-04: una consulta cerrada no se modifica');
    }
    if (!d.codigoCie10) throw new ReglaClinicaError('RN-03: el diagnóstico requiere código CIE-10');
    this.diagnosticos.push(d);
  }

  cerrar(): void {
    this.estado = EstadoConsulta.CERRADA;
  }
}
```

```ts
// dominio/consulta-repository.ts
import { Consulta } from './consulta';

export interface ConsultaRepository {
  guardar(consulta: Consulta): Promise<void>;
  buscarPorId(id: string): Promise<Consulta | null>;
}
```

```ts
// dominio/auditoria-port.ts
export interface AuditoriaPort {
  registrar(evento: { usuarioId: string; accion: string; recurso: string; fecha: Date }): Promise<void>;
}
```

```ts
// dominio/event-publisher.ts
export interface EventPublisher {
  publicar(tipo: string, payload: unknown): Promise<void>;
}
```

### 6.2 Aplicación · caso de uso

```ts
// aplicacion/registrar-consulta.use-case.ts
import { Consulta } from '../dominio/consulta';
import { PacienteRepository } from '../dominio/paciente-repository';
import { ConsultaRepository } from '../dominio/consulta-repository';
import { AuditoriaPort } from '../dominio/auditoria-port';
import { EventPublisher } from '../dominio/event-publisher';
import { RegistrarConsultaCommand } from './registrar-consulta.command';

export class RegistrarConsultaUseCase {
  constructor(
    private readonly pacienteRepository: PacienteRepository,
    private readonly consultaRepository: ConsultaRepository,
    private readonly auditoria: AuditoriaPort,
    private readonly eventos: EventPublisher, 
  ) {}

  async ejecutar(cmd: RegistrarConsultaCommand): Promise<string> {
    const paciente = await this.pacienteRepository.buscarPorId(cmd.pacienteId);
    if (!paciente) throw new Error('Paciente no encontrado');

    const consulta = Consulta.registrar(paciente.id, cmd.medicoId, cmd.motivo, cmd.diagnosticos);
    await this.consultaRepository.guardar(consulta);

    await this.auditoria.registrar({
      usuarioId: cmd.medicoId, accion: 'CREAR_CONSULTA',
      recurso: `consulta/${consulta.id}`, fecha: new Date(),
    });
    await this.eventos.publicar('ConsultaRegistrada', { consultaId: consulta.id, pacienteId: paciente.id });
    return consulta.id;
  }
}
```

### 6.3 Infraestructura · decorator de caché

```ts
// infraestructura/cached-paciente-repository.ts
import { PacienteRepository } from '../dominio/paciente-repository';
import { Paciente } from '../dominio/paciente';

export class CachedPacienteRepository implements PacienteRepository {
  constructor(
    private readonly origen: PacienteRepository,
    private readonly cache: CacheClient,
    private readonly ttlSegundos = 60,
  ) {}

  async buscarPorId(id: string): Promise<Paciente | null> {
    const enCache = await this.cache.get(`paciente:${id}`);
    if (enCache) return JSON.parse(enCache);

    const paciente = await this.origen.buscarPorId(id);
    if (paciente) await this.cache.set(`paciente:${id}`, JSON.stringify(paciente), this.ttlSegundos);
    return paciente;
  }
}
```

### 6.4 Presentación · controlador

```ts
// presentacion/consulta.controller.ts
import { Request, Response } from 'express';
import { RegistrarConsultaUseCase } from '../aplicacion/registrar-consulta.use-case';

export class ConsultaController {
  constructor(private readonly registrarConsulta: RegistrarConsultaUseCase) {}

  registrar = async (req: Request, res: Response): Promise<void> => {
    const consultaId = await this.registrarConsulta.ejecutar({
      medicoId: req.user.id,
      pacienteId: req.body.pacienteId,
      motivo: req.body.motivo,
      diagnosticos: req.body.diagnosticos,
    });
    res.status(201).json({ consultaId });
  };
}
```

### 6.5 Raíz de composición

```ts
// hce.module.ts
export function crearModuloHce(db: Pool, redis: CacheClient, bus: BusClient) {
  const pacienteRepository = new CachedPacienteRepository(new PostgresPacienteRepository(db), redis);
  const consultaRepository = new PostgresConsultaRepository(db);
  const auditoria = new AuditoriaPostgresAdapter(db);
  const eventos = new BrokerEventPublisher(bus);
  const registrar = new RegistrarConsultaUseCase(pacienteRepository, consultaRepository, auditoria, eventos);
  return { controller: new ConsultaController(registrar) };
}
```
