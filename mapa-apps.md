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
        JOB["Job Post ATS Analizer<br/>Avisos, CV y postulaciones<br/><small>Prototipo</small>"]
        RND["Random<br/>Selecciones aleatorias con reglas y pesos<br/><small>Prototipo</small>"]
        FOTOS["FotosCompare<br/>Google Fotos vs. dispositivo: respaldos y duplicados<br/><small>Prototipo</small>"]
        DISK["DiskCompare (Android APK)<br/>Uso del almacenamiento, duplicados y comparación con la PC<br/><small>Prototipo</small>"]
        ANOTE["ANote... (Web + APK)<br/>Notas sincronizadas y compartidas<br/><small>En uso</small>"]
        PLAN["Planillas (Web + APK + Windows)<br/>Hojas tipo Excel con vistas Reporte y Ficha<br/>Importa/exporta .xlsx y .csv; copiar/pegar con Excel<br/>Cuenta con correo (la de ANote...), nube y compartir (invitación y link)<br/><small>Prototipo</small>"]
    end

    subgraph EXT["Servicios externos"]
        direction TB
        GS[("Google Sheets")]
        SOFTR(["Softr.io"])
        GAS(["Google Apps Script"])
        IA(["IA: Gemini / Claude"])
        GF(["Google Fotos (Takeout y Picker API)"])
        WIN(["PC con Windows (carpeta compartida SMB)"])
        FB[("Firebase notas-pro-2026")]
    end

    ABM --> GS
    PANEL --> GS
    SUPER --> GS
    SOFTRL --> SOFTR
    JOB -. opcional .-> GAS
    GAS --> IA
    CV -. JSON .-> JOB
    FOTOS --> GF
    DISK -. WiFi .-> WIN
    ANOTE --> FB
    PLAN -. mismas cuentas .-> FB

    classDef enuso fill:#c8e6c9,stroke:#2e7d32,color:#1b1b1b
    classDef prototipo fill:#fff3b0,stroke:#b38f00,color:#1b1b1b
    classDef idea fill:#eeeeee,stroke:#757575,color:#1b1b1b
    classDef servicio fill:#cfe3ff,stroke:#1e5aa8,color:#1b1b1b

    class CV,CIF,REC,ANOTE enuso
    class ABM,PANEL,SUPER,JOB,RND,FOTOS,DISK prototipo
    class SOFTRL,PLAN idea
    class GS,SOFTR,GAS,IA,GF,WIN,FB servicio
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
| Job Post ATS Analizer | Prototipo | IA opcional (Gemini o Claude, vía Apps Script); importa el JSON de CVAPP | [jobpost](https://github.com/luchorcompartidos/jobpost) |
| Random | Prototipo | — | [random](https://github.com/luchorcompartidos/random) |
| FotosCompare | Prototipo | Google Fotos (Takeout y Picker API, con el proyecto de Google Cloud de multigdrive) | [fotoscompare](https://github.com/luchorcompartidos/fotoscompare) |
| DiskCompare (Android APK) | Prototipo | Carpeta compartida de Windows por WiFi (SMB, solo lectura) | [diskcompare](https://github.com/luchorcompartidos/diskcompare) |
| ANote... (Web + Android APK) | En uso | Firebase (proyecto notas-pro-2026) | [notas](https://github.com/luchorcompartidos/notas) |
| Planillas (nombre a definir; Web + APK + Windows) | Prototipo (v0.0.12) | Importa/exporta Excel (.xlsx) y CSV; Firebase (notas-pro-2026, mismas cuentas que ANote...: inicio con correo desde la v0.0.9, nube desde la v0.0.10, compartir por invitación desde la v0.0.11 y por link con pedido de acceso desde la v0.0.12; reglas en un bloque propio junto a las de ANote...) | [planillas](https://github.com/luchorcompartidos/planillas) |
