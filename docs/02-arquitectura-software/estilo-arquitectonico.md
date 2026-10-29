# Estilo arquitectónico

El sistema adopta un estilo de **microservicios desplegados en múltiples zonas de disponibilidad activas**, con un sitio adicional de recuperación ante desastres (DR). Responde a ADR-001, ADR-002, ADR-003 y ADR-004.

```mermaid
flowchart TD
    Clientes["Clientes: portal web, app móvil, estaciones clínicas, triaje"]
    DNS["DNS / Balanceador global con health-check automático"]

    subgraph ZA["Zona de disponibilidad A - Activa"]
        AppA["Clúster de aplicación, 3+ réplicas"]
        TriajeA["Clúster Triaje / Emergencia"]
        BDA["BD primaria de HCE: escrituras y réplica síncrona"]
    end

    subgraph ZB["Zona de disponibilidad B - Activa"]
        AppB["Clúster de aplicación, 3+ réplicas"]
        TriajeB["Clúster Triaje / Emergencia"]
        BDB["BD réplica local: lecturas, failover automático menor a 30 s"]
    end

    subgraph DR["Sitio de recuperación ante desastres - warm standby"]
        Espera["Réplica en espera"]
        Backups["Backups cifrados, retención 7 años"]
    end

    Clientes --> DNS
    DNS --> ZA
    DNS --> ZB
    BDA <-->|"replicación síncrona"| BDB
    BDA -.->|"replicación al DR"| DR

    style ZA fill:#222,stroke:#fff,stroke-width:2px,color:#fff
    style ZB fill:#222,stroke:#fff,stroke-width:2px,color:#fff
    style DR fill:#4a1a1a,stroke:#fff,stroke-width:2px,color:#fff
```

## Metas de recuperación

| Escenario | RTO | RPO |
| --- | --- | --- |
| Falla de una zona | < 5 minutos | ≈ 0 |
| Pérdida de ambas zonas (sitio DR) | < 2 horas | < 15 minutos |

## Justificación

- **Microservicios:** permiten escalar y proteger de forma independiente al módulo más crítico (Triaje/HCE) sin sobredimensionar los demás.
- **Multi-zona activo-activo:** la pérdida de una zona completa no interrumpe el servicio.
- **Sitio DR:** cubre el escenario de baja probabilidad pero alto impacto de falla simultánea de ambas zonas.
- **Portabilidad:** el uso de contenedores/Kubernetes reduce la dependencia de un único proveedor de infraestructura.
