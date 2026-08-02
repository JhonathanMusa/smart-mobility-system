# Fase 3: Construcción de Vistas Arquitectónicas

Especificación y modelos visuales para la representación del **Smart Mobility System (SMS)**.

---

## 1. Vista de Casos de Uso (UML)

La Vista de Casos de Uso define el comportamiento del sistema desde el punto de vista de los actores involucrados.

```mermaid
graph TD
    U[Usuario Móvil] -->|Consultar Ruta Óptima| CU1(CU01. Consultar Ruta)
    U -->|Recibir Alertas| CU2(CU02. Recibir Alerta)
    S[Sistema Sensores] -->|Transmitir Telemetría| CU3(CU03. Transmitir Datos)
    SEM[Semáforo Inteligente] -->|Recibir Ajustes| CU4(CU04. Ajustar Ciclos)
    OP[Operador Centro Control] -->|Control Manual| CU6(CU06. Intervención Manual)
    CU6 -->|Disparar Alerta| CU7(CU07. Notificar Incidente)
    CU7 -->|Notificar| EM[Servicios Emergencia]

    CU1 -.->|includes| CU5(CU05. Validar Sesión)
    CU6 -.->|includes| CU5

    classDef actorUser fill:#F0997B,stroke:#993C1D,stroke-width:1.5px,color:#4A1B0C;
    classDef actorCore fill:#5DCAA5,stroke:#0F6E56,stroke-width:1.5px,color:#04342C;
    classDef actorAdmin fill:#AFA9EC,stroke:#534AB7,stroke-width:1.5px,color:#26215C;
    classDef actorExt fill:#F4C0D1,stroke:#993556,stroke-width:1.5px,color:#4B1528;
    classDef useCase fill:#E1F5EE,stroke:#0F6E56,stroke-width:1px,color:#04342C;
    classDef shared fill:#D3D1C7,stroke:#5F5E5A,stroke-width:1px,color:#2C2C2A;

    class U actorUser
    class S actorCore
    class SEM actorCore
    class OP actorAdmin
    class EM actorExt
    class CU1 useCase
    class CU2 useCase
    class CU3 useCase
    class CU4 useCase
    class CU6 useCase
    class CU7 useCase
    class CU5 shared
```

### Especificación de Casos de Uso Clave

* **CU01 - Consultar Ruta Óptima:**
  - *Actor:* Usuario Móvil.
  - *Propósito:* Obtener la ruta con menor tiempo estimado de llegada (ETA) considerando congestiones e imprevistos.
  - *Inclusiones:* `CU05. Validar Sesión`.

* **CU03 - Transmitir Datos de Telemetría:**
  - *Actor:* Sistema de Sensores IoT.
  - *Propósito:* Enviar ráfagas de datos con conteo de vehículos, velocidad promedio y ocupación de carril.

* **CU04 - Ajustar Ciclos de Semáforos:**
  - *Actor:* Semáforo Inteligente / Módulo de Gestión.
  - *Propósito:* Recibir y ejecutar las instrucciones de modificación de tiempo de luz verde emitidas por el sistema central.

* **CU06 - Intervenir Control Semafórico Manual:**
  - *Actor:* Operador del Centro de Control.
  - *Propósito:* Forzar el estado de un semáforo o crear un corredor verde prioritario ante emergencias.

---

## 2. Vista Lógica (Diagrama de Componentes)

Estructura estática organizada por capas y módulos desacoplados mediante interfaces de integración.

```mermaid
graph TB
    subgraph Presentacion ["Capa de Presentación"]
        App[App Móvil iOS/Android]
        Dash[Dashboard Operador]
    end

    subgraph Gateway ["Capa de Integración"]
        GW[API Gateway / OAuth2]
    end

    subgraph Microservicios ["Capa de Negocio (Microservicios)"]
        MGS[Procesamiento Sensores]
        MGT[Gestión Tráfico]
        MGSem[Gestión Semáforos]
        MGR[Gestión Rutas]
        MGA[Gestión Alertas]
        ML[Análisis Predictivo]
    end

    subgraph Persistencia ["Capa de Datos"]
        DB1[(TimescaleDB - Telemetría)]
        DB2[(PostGIS / Neo4j - Rutas)]
        Kafka{{Bus de Eventos Kafka}}
    end

    App --> GW
    Dash --> GW
    GW --> Microservicios
    MGS --> Kafka
    Kafka --> MGT
    MGT --> MGSem
    MGT --> MGA
    MGR --> DB2
    MGS --> DB1

    style Presentacion fill:#FAECE7,stroke:#993C1D,stroke-width:1.5px,color:#4A1B0C
    style Gateway fill:#F1EFE8,stroke:#5F5E5A,stroke-width:1.5px,color:#2C2C2A
    style Microservicios fill:#E1F5EE,stroke:#0F6E56,stroke-width:1.5px,color:#04342C
    style Persistencia fill:#EEEDFE,stroke:#534AB7,stroke-width:1.5px,color:#26215C

    classDef app fill:#F0997B,stroke:#993C1D,color:#4A1B0C;
    classDef gw fill:#B4B2A9,stroke:#5F5E5A,color:#2C2C2A;
    classDef svc fill:#5DCAA5,stroke:#0F6E56,color:#04342C;
    classDef data fill:#AFA9EC,stroke:#534AB7,color:#26215C;

    class App app
    class Dash app
    class GW gw
    class MGS svc
    class MGT svc
    class MGSem svc
    class MGR svc
    class MGA svc
    class ML svc
    class DB1 data
    class DB2 data
    class Kafka data
```

---

## 3. Vista de Procesos (Diagrama de Secuencia)

Flujo dinámico e interacciones asíncronas para el escenario: **Detección de Congestión y Generación Automática de Alertas con Ajuste Semafórico**.

```mermaid
%%{init: {'theme':'base', 'themeVariables': {
  'primaryColor': '#E1F5EE',
  'primaryTextColor': '#04342C',
  'primaryBorderColor': '#0F6E56',
  'lineColor': '#5F5E5A',
  'actorBkg': '#EEEDFE',
  'actorBorder': '#534AB7',
  'actorTextColor': '#26215C',
  'signalColor': '#2C2C2A',
  'signalTextColor': '#2C2C2A',
  'noteBkgColor': '#FAECE7',
  'noteBorderColor': '#993C1D',
  'noteTextColor': '#4A1B0C'
}}}%%
sequenceDiagram
    autonumber
    actor Sensor as Sensor IoT
    participant Ingestion as Ingesta IoT
    participant Trafico as Gestión Tráfico
    participant Semaforo as Gestión Semáforos
    participant Alertas as Gestión Alertas
    actor App as App Móvil

    Sensor->>Ingestion: Telemetría Tráfico Alto
    Ingestion->>Trafico: Evento TelemetriaRecibida
    Trafico->>Trafico: Evaluar Umbral Congestión
    Trafico->>Semaforo: Notificar Congestión Detectada
    Semaforo->>Semaforo: Calcular Nueva Fase Verde
    Semaforo->>Sensor: Comando AmpliarFaseVerde()
    Trafico->>Alertas: Emitir Alerta Congestión
    Alertas->>App: Push Notification Geolocalizada
```

---

## 4. Vista Conceptual (Diagrama Macro)

La "gran fotografía" del ecosistema **Smart Mobility System** relacionando dominios físicos y lógicos.

```mermaid
flowchart LR
    subgraph Edge ["1. CAPTURA Y EDGE"]
        E1[Sensores Viales]
        E2[Semáforos]
    end

    subgraph Core ["2. SISTEMA CENTRAL (CLOUD)"]
        C1[Ingestor IoT]
        C2[Event Broker - Kafka]
        C3[Motor Analytics & Routing]
        C4[Bases de Datos]
    end

    subgraph External ["3. CONSUMIDORES & EXTERNO"]
        X1[App Ciudadanos]
        X2[Centro Control]
        X3[Servicios Emergencia / Mapas]
    end

    E1 -->|MQTT / gRPC| C1
    C1 --> C2
    C2 --> C3
    C3 <--> C4
    C3 -->|REST / WebSockets| X1
    C3 -->|REST / WebSockets| X2
    C3 <-->|APIs| X3
    C3 -->|Comandos| E2

    style Edge fill:#E1F5EE,stroke:#0F6E56,stroke-width:1.5px,color:#04342C
    style Core fill:#EEEDFE,stroke:#534AB7,stroke-width:1.5px,color:#26215C
    style External fill:#FAECE7,stroke:#993C1D,stroke-width:1.5px,color:#4A1B0C

    classDef edge fill:#5DCAA5,stroke:#0F6E56,color:#04342C;
    classDef core fill:#7F77DD,stroke:#534AB7,color:#FFFFFF;
    classDef ext fill:#F0997B,stroke:#993C1D,color:#4A1B0C;

    class E1 edge
    class E2 edge
    class C1 core
    class C2 core
    class C3 core
    class C4 core
    class X1 ext
    class X2 ext
    class X3 ext
```