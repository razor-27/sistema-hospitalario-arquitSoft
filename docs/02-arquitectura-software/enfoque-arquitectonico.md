# Enfoque arquitectónico

Dentro de cada microservicio se aplica un enfoque **por capas**, con separación de responsabilidades. Como ejemplo, se muestra el microservicio de **Historia Clínica Electrónica (HCE)**, por ser el de mayor criticidad (ADR-008).

```mermaid
flowchart TD
    GW["Desde API Gateway: /api/hce/*"]

    subgraph API["CAPA API - controllers REST"]
        PC["PacienteController"]
        CC["ConsultaController"]
        DC["DiagnosticoController"]
        AC["AuditoriaController"]
    end

    subgraph APP["CAPA DE APLICACIÓN Y DOMINIO"]
        HS["HistoriaClinicaService: orquesta casos de uso"]
        VS["ValidacionClinicaService: reglas de negocio"]
        AS["AuditoriaService: registro inmutable de accesos"]
        ENT["Entidades: Paciente, Consulta, Diagnóstico, Prescripción"]
    end

    subgraph DATA["CAPA DE ACCESO A DATOS"]
        PR["PacienteRepository"]
        CR["ConsultaRepository"]
    end

    subgraph INFRA["INFRAESTRUCTURA DE SOPORTE"]
        CacheC["Cliente de caché Redis"]
        Pub["Publicador de eventos"]
    end

    subgraph PERS["PERSISTENCIA"]
        DB["BD esquema HCE: pacientes, consultas, diagnósticos, prescripciones, resultados de laboratorio. Réplica síncrona multi-zona"]
        Redis["Redis Cluster: sesión clínica activa, disponibilidad de citas, TTL corto"]
        Bus["Bus de eventos: notifica a Laboratorio, Farmacia y Notificaciones"]
    end

    GW --> API
    API --> APP
    APP --> DATA
    APP --> INFRA
    PR --> DB
    CR --> DB
    CacheC --> Redis
    Pub --> Bus

    style API fill:#222,stroke:#fff,stroke-width:2px,color:#fff
    style APP fill:#222,stroke:#fff,stroke-width:2px,color:#fff
    style DATA fill:#222,stroke:#fff,stroke-width:2px,color:#fff
    style INFRA fill:#222,stroke:#fff,stroke-width:2px,color:#fff
    style PERS fill:#222,stroke:#fff,stroke-width:2px,color:#fff
```

## Reglas del enfoque

1. Cada capa solo invoca a la capa inmediatamente inferior.
2. Los controladores REST no contienen reglas de negocio.
3. Las reglas de validación clínica viven en la capa de dominio, por lo que se pueden añadir sin modificar los controladores.
4. El motor de caché o el bus de mensajería pueden sustituirse sin afectar la capa de dominio.
5. Los mecanismos de resiliencia (reintentos, timeouts, circuit breakers) se aplican a nivel de código en esta estructura.
