# Mapa de mi caja de apps

```mermaid
flowchart LR
    subgraph CAJA["Caja de apps"]
        direction TB
        CV["CVAPP<br/><small>En uso</small>"]
        CIF["Cifrados PRO<br/><small>En uso</small>"]
        REC["Recordatorios (Android APK)<br/><small>En uso</small>"]
        ABM["ABM Registros<br/><small>Prototipo</small>"]
        PANEL["Mi Panel<br/>Portal de apps y accesos<br/><small>Prototipo</small>"]
        SOFTRL["Softr.io Apps Launcher<br/><small>Idea</small>"]
        SUPER["Super (Góndola)<br/>Comparador de precios<br/><small>Prototipo</small>"]
    end

    subgraph EXT["Servicios externos"]
        direction TB
        GS[("Google Sheets")]
        SOFTR(["Softr.io"])
    end

    ABM --> GS
    PANEL --> GS
    SUPER --> GS
    SOFTRL --> SOFTR

    classDef enuso fill:#c8e6c9,stroke:#2e7d32,color:#1b1b1b
    classDef prototipo fill:#fff3b0,stroke:#b38f00,color:#1b1b1b
    classDef idea fill:#eeeeee,stroke:#757575,color:#1b1b1b
    classDef servicio fill:#cfe3ff,stroke:#1e5aa8,color:#1b1b1b

    class CV,CIF,REC enuso
    class ABM,PANEL,SUPER prototipo
    class SOFTRL idea
    class GS,SOFTR servicio
```

**Referencias:** verde = En uso · amarillo = Prototipo · gris = Idea · azul = servicio externo

| App | Estado | Se conecta con | Repo |
|---|---|---|---|
| CVAPP | En uso | — | [cvapp](https://github.com/luchorcompartidos/cvapp) |
| Cifrados PRO | En uso | — | [cifrados](https://github.com/luchorcompartidos/cifrados) |
| Recordatorios (Android APK) | En uso | — | [recordatorios](https://github.com/luchorcompartidos/recordatorios) |
| ABM Registros | Prototipo | Google Sheets | [abm](https://github.com/luchorcompartidos/abm) |
| Mi Panel (Portal de apps y accesos) | Prototipo | Google Sheets | [home](https://github.com/luchorcompartidos/home) |
| Softr.io Apps Launcher | Idea | Softr.io | [softrapps](https://github.com/luchorcompartidos/softrapps) |
| Super (Góndola) | Prototipo | Google Sheets | [super](https://github.com/luchorcompartidos/super) |
