# Estándar de versiones — caja de apps

Aplica a todas las apps (CVAPP, Cifrados PRO, ABM, Mi Panel, Softr.io Apps Launcher, CompareIt).
Única excepción: Recordatorios (APK), donde el número ya va compilado en la app (versionName).
Ahí se suma la misma lógica en Kotlin (`Version.kt`): Changelog y aviso de versión nueva leyendo las
Releases, con link para descargar el instalador (.apk adjunto). Su flujo es `compilar.yml`; sin etiqueta
compila de prueba (APK en Artifacts) en lugar de solo avisar.

## Numeración (SemVer, siempre tres números)

| Tipo  | Ejemplo           | Cuándo                                   |
|-------|-------------------|------------------------------------------|
| Major | v0.9.2 → v1.0.0   | Cambios que rompen o rediseños           |
| Minor | v0.3.1 → v0.4.0   | Funcionalidad nueva compatible           |
| Patch | v0.3.0 → v0.3.1   | Correcciones y ajustes chicos            |
| Sin release | —           | El push publica pero no crea tag ni Release |

Las versiones viejas de dos números (v0.1, v0.2) quedan como están; el orden se calcula bien.
El tipo se indica en el mensaje del commit: `[major]`, `[minor]`, `[patch]` o `[skip release]`.

**Opcional, para apps cuyo flujo compila algo pesado (por ejemplo un APK):** en el `on: push:` del flujo, agregar
`paths-ignore: ['**.md']`, así un push que solo cambia documentos (DESARROLLO.md, README.md) no corre el flujo
(no compila ni puede fallar por problemas pasajeros, como el 429 de Maven). Para correrlo igual: *Run workflow*.
Caso de referencia: `ci.yml` de luchorcompartidos/notas (ANote..., v0.3.4).

## Las tres piezas

1. **Número escrito en el HTML**: `<meta name="app-version" content="vX.Y.Z">` en el `<head>`.
   Antes de cada entrega se leen los tags del repo, se calcula el mismo número que calculará
   el flujo y se escribe ahí. Se ve sin conexión, en Apps Script y con copias viejas en caché.
   El flujo `release.yml` avisa si el número del HTML no coincide con la Release creada, y si el del HTML
   es más alto que el calculado usa el del HTML (útil para la primera Release de una app con numeración propia).
2. **Fecha y Changelog desde GitHub**: al tocar el número, la app consulta las Releases
   (`api.github.com/repos/<usuario>/<repo>/releases`) y muestra la fecha del archivo más
   reciente de esa versión y el Changelog. Sin conexión usa la última copia guardada (localStorage).
3. **Aviso de versión nueva**: si GitHub tiene una versión más alta que la del archivo,
   se muestra "Hay una versión nueva (vX.Y.Z), recargá" con botón Recargar.

## Plantillas (carpeta `plantilla/`)

- `release.yml` y `reemplazar-version.yml`: flujos genéricos. Reemplazar `__APP_FILE__`, `__ADJUNTOS__`,
  `__PLANILLA_URL__` (vacío si no hay planilla), `__PLANILLA_NOMBRE__` `__CON_PLANILLA__` (true/false) y `__DISPARAR_FLUJO__` (flujo de publicación a lanzar
  tras la Release, ej. `pages.yml`, si Pages se publica solo al crear Releases; vacío = ninguno),
  y copiarlos a `.github/workflows/`.
- `version-bloque.html`: bloque autónomo (estilos + aviso + diálogo + script). Se pega antes de `</body>`,
  cambiando `__REPO__` y `__LOG_FN__` (nombre de la función de Changelog propia de la app, o vacío para
  usar el de GitHub); en el encabezado va `<button id="verBtn" class="vs-ver" type="button">v—</button>`.

- Si la app ya muestra su propio texto de versión, marcar ese elemento con `data-vs-open` (abre el diálogo sin
  cambiarle el texto). Dentro de Apps Script el aviso pide actualizar el script en lugar de recargar.
- CompareIt: después de cambiar `index.html` hay que regenerar `apps-script/App.gs` (esquema + copia del HTML en base64).

## Implementación de referencia

`index.html` de este repo (Mi Panel), bloque `Versión (estándar caja de apps)`,
y `.github/workflows/release.yml` (cálculo de versión y control del número en el HTML).
Para otra app: copiar el bloque, cambiar `REPO`, `RELS_KEY` y `APP_FILE`.
