# APK para dispositivos Android viejos (Android 5 y 6)

> Guía para aplicar en cualquier app de la caja de apps que se distribuya como APK.
> Uso en un chat nuevo: *"Seguimos con [app]. Leé `plantilla/APK-DISPOSITIVOS-VIEJOS.md` del repo luchorcompartidos/home y `DESARROLLO.md` del repo luchorcompartidos/[repo], y aplicá la guía."*
> Casos de referencia: **DiskCompare** (Flutter; luchorcompartidos/diskcompare, v0.0.3, `.github/workflows/ci.yml`) y **ANote...** (Capacitor; luchorcompartidos/notas, v0.2.1–v0.3.1, probada en Android 5.0 y 6.0.1; ver sección 8) **Planillas** (Capacitor, luchorcompartidos/planillas) nació con el esqueleto de ANote... ya adaptado.
> Al terminar en cada app: anotar la decisión en su `DESARROLLO.md` (Decisiones tomadas + Datos técnicos).
> **Mantener esta guía al día:** al cerrar cada etapa de una app con **Sí** en "Dispositivos viejos", revisar si lo hecho deja algo nuevo para esta guía (un problema, un arreglo, una versión de biblioteca, una prueba en las tablets) y, si corresponde, agregarlo en la sección del tema y en el Historial, y subirla. Sin esperar a que el usuario lo pida. La misma regla está en `plantilla/DESARROLLO.md` y en el `DESARROLLO.md` de cada app con **Sí**.
> **No aplica a todas las apps:** solo a las que tienen **Sí** en la fila "Dispositivos viejos (Android 5/6)" de la Ficha rápida de su `DESARROLLO.md` (hoy: DiskCompare, Notas y Planillas). Si la fila falta o dice "Sin definir", preguntar una vez y anotarla.

---

## 1. El problema
- Síntoma al abrir el .apk en un equipo viejo: **"Error de análisis — Se produjo un error durante el análisis del paquete"**.
- Causa (apps **Flutter**): desde **Flutter 3.35** la versión mínima es **Android 7.0 (API 24)**. GitHub Actions compila con el último Flutter, así que el APK queda "solo Android 7+" y Android 5/6 lo rechaza antes de instalar.
- **No es por la arquitectura**: si faltara la arquitectura, el mensaje suele ser "App no instalada".
- Flutter **3.32** es la última versión que soporta **Android 5.0 (API 21)** y 6.0.
- Por debajo de Android 5.0 no hay solución con Flutter.

## 2. Equipos de prueba
| Equipo | Android | Conjuntos de instrucciones (DevCheck) | Qué APK usa |
|---|---|---|---|
| Celular personal | 7+ (64 bits) | arm64-v8a … | normal **arm64-v8a** |
| Tablet Android 6.0.1 | 6.0.1 (API 23), 32 bits | armeabi-v7a | **android5-6-armeabi-v7a** |
| Tablet Bangho AERO J1216 (Intel Atom Z3735F) | 5.0 (API 21) | `x86 armeabi-v7a armeabi` | **android5-6-armeabi-v7a**, con traductor Intel (Houdini) |

- **Cómo saber la arquitectura real:** en DevCheck (o CPU-Z) mirar **"Conjuntos de instrucciones" / "Supported ABIs"**, no "Arquitectura" ni "Kernel". La Bangho muestra kernel *x86_64*, pero su Android es de 32 bits: no tiene x86_64 en la lista.
- Con el traductor de Intel la app puede ir más lenta. En algunas tablets Intel viejas las apps Flutter se cierran al abrir: solo se sabe probando. Si pasa, la alternativa es una versión nativa Kotlin (costo mayor).

## 3. Esquema: dos APK por Release (misma app, misma firma)
| Archivo | Arquitectura | Flutter | Para |
|---|---|---|---|
| `[app]-vX.Y.Z-arm64-v8a.apk` | **arm64-v8a** | último stable | Celulares y tablets con Android 7 o más (64 bits) |
| `[app]-vX.Y.Z-android5-6-armeabi-v7a.apk` | **armeabi-v7a** | 3.32.x | Android 5 y 6 (y Android de 32 bits) |

- Ventaja extra: el APK normal baja mucho de peso. En DiskCompare pasó de 46 MB (universal, 3 arquitecturas) a 16 MB, y el de tablets pesa 6,6 MB.
- Sin **x86_64**: no hay equipos que lo usen. No afecta a una versión Windows (.exe), que es otra compilación.
- El nombre de cada APK y el texto de cada Release llevan el **nombre literal de la arquitectura**.
- Instalar el APK que no corresponde no rompe nada: Android no lo deja instalar.

## 4. Cómo aplicarlo en una app Flutter (pasos)
1. **Revisar dependencias**: cada paquete del `pubspec.yaml` tiene que poder resolver con Flutter 3.32 (Dart 3.8). `flutter pub get` elige solo versiones compatibles; si alguno exige un Flutter más nuevo, fijar una versión anterior o dejar esa función fuera del APK de tablets.
2. **Revisar el código**: no usar APIs de Flutter más nuevas que la 3.32 (o revisarlo antes en el APK de tablets).
3. **Revisar la versión mínima de Android de cada servicio** (ver sección 6, sobre todo Firebase).
4. **CI**: convertir el job `android` en una matriz (ver sección 5). Mantener el `version` y el `release` del estándar, con descarga de los dos artifacts.
5. **Mostrar la arquitectura en la app** con `--dart-define=APP_ABI=...` (en "Acerca de" y en el pie), para saber cuál quedó instalado.
6. **Release** según `ESTANDAR-VERSIONES.md` (normalmente Patch). Probar en el celular y en las 2 tablets.
7. **DESARROLLO.md** de la app: anotar la decisión, los nombres de archivo y la regla "compila con Flutter 3.32".

## 5. Fragmentos del CI (copiados de DiskCompare)
```yaml
  android:
    needs: version
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        include:
          - kind: normal
            flutter: ""            # último stable
            platform: android-arm64
            abi: arm64-v8a
            suffix: arm64-v8a
          - kind: android5-6
            flutter: "3.32.x"      # último que soporta Android 5/6
            platform: android-arm
            abi: armeabi-v7a
            suffix: android5-6-armeabi-v7a
    steps:
      # ... checkout, setup-java 17 ...
      - uses: subosito/flutter-action@v2
        with:
          channel: stable
          flutter-version: ${{ matrix.flutter }}
          cache: true
      # ... flutter create, parches, firma fija ...
      - name: Compilar APK (${{ matrix.abi }})
        run: |
          flutter build apk --release \
            --target-platform ${{ matrix.platform }} \
            --build-name=${{ needs.version.outputs.number }} \
            --build-number=${{ needs.version.outputs.code }} \
            --dart-define=APP_VERSION=$VERSION \
            --dart-define=APP_ABI=${{ matrix.abi }}
          mkdir -p out
          cp build/app/outputs/flutter-apk/app-release.apk "out/[app]-v${VERSION}-${{ matrix.suffix }}.apk"
      - uses: actions/upload-artifact@v4
        with:
          name: android-${{ matrix.kind }}
          path: out/*.apk
```
En el job `release`:
```yaml
      - uses: actions/download-artifact@v4
        with:
          path: out
          pattern: android-*
          merge-multiple: true
```
Y en el texto de la Release, una tabla **"¿Qué archivo bajar?"** con *Archivo | Arquitectura | Para* (ver `ci.yml` de diskcompare, paso "Notas de la versión").

## 6. Puntos a verificar según la app
- **Firebase (por ejemplo, Notas):** las versiones recientes del SDK de Firebase para Android podrían pedir un mínimo mayor que Android 5.0, quizá Android 6.0 (API 23). **Verificarlo antes de prometer nada.** Posibles resultados: la tablet Android 6 anda y la de Android 5 no, salvo que se fijen versiones más viejas de los paquetes de Firebase en el APK de tablets.
- **Inicio de sesión de Google / OAuth / WebView:** en Android 5/6, el navegador o el WebView del sistema puede ser muy viejo. Probar el inicio de sesión en las tablets.
- **Alternativa sin Google: correo y contraseña de Firebase** (caso ANote..., v0.3.0): con el SDK web de Firebase dentro de la WebView no hace falta ningún plugin nativo, no sube el mínimo de Android y no depende de Play services (que en Android 5 no se actualiza desde 2024). Sirve si el inicio con Google falla en una tablet vieja. *A confirmar en la Bangho (Android 5.0).*
- **Permisos de archivos:** en Android 5 el permiso de almacenamiento se da al instalar; en Android 6 se pide en la app (`Permission.storage`); en Android 11+ se usa `MANAGE_EXTERNAL_STORAGE`. El código tiene que contemplar los tres casos según `SDK_INT`.
- **Notificaciones:** `POST_NOTIFICATIONS` solo existe desde Android 13; en 5 y 6 no se pide.
- **Leer el contenido de muchos archivos** (huellas, duplicados; caso DiskCompare v0.0.7): en Dart puro, sin el paquete `crypto` (no suma dependencias que puedan pedir un Flutter más nuevo). Hacerlo en un isolate aparte, en bloques de 1 MB, comparando primero el principio y el final de cada archivo y el archivo entero solo si coinciden; con barra de progreso y **Cancelar**. En armeabi-v7a la aritmética de 64 bits es emulada y en la Bangho además pasa por el traductor de Intel: puede ir bastante más lenta que en el celular. *A medir en las tablets.*

## 7. Apps nativas Kotlin (Recordatorios, Mensajes programados, …)
- No tienen el límite de Flutter: en general alcanza con **un solo APK**, bajando `minSdk` a 21 (Android 5.0) en `build.gradle(.kts)`.
- Revisar las librerías (AndroidX, Compose, WorkManager): cada una declara su propio `minSdk`; el más alto manda.
- Las funciones que no existen en Android 5/6 se protegen con `if (Build.VERSION.SDK_INT >= ...)`.
- Si el APK es universal y pesa poco, se puede dejar así. Si pesa mucho, separar por arquitectura igual que en Flutter (`splits { abi { … } }`).

## 8. Apps Capacitor (web dentro de un APK: Notas / ANote...)
- **Instalar:** Capacitor 7 y sus plugins toman `minSdkVersion` de `android/variables.gradle` (por defecto 23 = Android 6). Para Android 5 se baja a 21 ahí; revisar los `SDK_INT` del código nativo y de los plugins (en Notas, solo los recordatorios usaban una función de Android 6 sin control).
- **Bibliotecas que piden más que API 21:** el *manifest merger* corta la compilación y las muestra **de a una**. Conviene agregar al CI un paso `if: failure()` que liste todas las AAR del caché de Gradle con `minSdkVersion` mayor a 21 (ver `ci.yml` de notas). En ANote... (v0.2.2) aparecieron tres:
  - `com.google.firebase:firebase-auth` 23.x (API 23) → fijar `firebaseAuthVersion = "22.3.1"` en `variables.gradle` (para `@capacitor-firebase/authentication` 7.x).
  - `@capacitor/filesystem` 7.1.x, que trae `io.ionic.libs:ionfilesystem-android` (API 23) → fijar `"@capacitor/filesystem": "7.0.1"` en `package.json`.
  - `org.apache.cordova:framework` 10.1.1 (API 22 = Android 5.1), que Capacitor incluye siempre → `<uses-sdk tools:overrideLibrary="org.apache.cordova" />` en el manifiesto (con `xmlns:tools`), si la app no usa plugins de Cordova. Confirmar probando en un Android 5.0.
- **Que compile no alcanza:** con el mínimo bajado a 21, una biblioteca puede llamar a una función de Android 6+ sin control y la app **se instala pero se cierra al abrir** ("Se ha detenido la aplicación"). En la tablet Android 6 anda, en la de Android 5.0 no.
  - **Ver el error en el equipo:** ANote... (v0.2.5) tiene un **atrapador de cierres**: un `Application` propio (`AnoteApp`) con `UncaughtExceptionHandler` que abre una pantalla en proceso aparte (`CierreActivity`, `:cierre`, solo vistas de Android) con el error y botones Copiar / Enviar…. Copiar los dos `.java` de `notas/android-extra/java/` y el bloque del manifiesto de `patch-android.py`. Sirve para cualquier app nativa (Capacitor o Kotlin). No atrapa cierres nativos (de la WebView): para eso, `adb logcat` desde la PC.
  - **Caso ANote... (v0.2.6):** `@capacitor/status-bar` 7.0.6 llama a `View.getRootWindowInsets()` (API 23) al cargarse → `NoSuchMethodError` en Android 5.0. Se corrige el `.java` del plugin en `node_modules` desde `patch-android.py` (Gradle compila los plugins desde ahí), con `if (SDK_INT < 23)` y el alto del recurso `status_bar_height`. Con un `assert`, para que el CI corte si el plugin cambia.
  - **Resultado ANote... (v0.2.6, 2026-11-26):** con el arreglo de `@capacitor/status-bar` la app abre y anda en la Bangho (Android 5.0, WebView 95) y en la tablet Android 6.0.1. El puente de Cordova con `tools:overrideLibrary` no dio problemas. Un solo APK universal para los tres equipos.
- **Portapapeles:** la WebView no deja leer el portapapeles desde JavaScript (`navigator.clipboard.readText` falla sin permiso). ANote... (v0.2.7) usa un plugin propio chico (`PortapapelesPlugin.java`, con `ClipboardManager`, que existe desde Android 3) registrado en `MainActivity`; no suma dependencias ni sube el mínimo de Android. Planillas (v0.0.8) lo reutiliza tal cual (cambiando el paquete): una `MainActivity.java` propia en `android-extra/java/` reemplaza la que genera `npx cap add android` (el `patch-android.py` copia la carpeta encima y verifica con un `assert` que registre el plugin). Compiló a la primera con `sdkVersion:'21'`. En la web, para Ctrl+C/X/V conviene usar los eventos `copy`/`cut`/`paste` (`clipboardData`), que no piden permiso.
- **Verificar en el CI** con `aapt dump badging` que el APK diga `sdkVersion:'21'`, y en la Release poner la tabla "¿Qué archivo bajar?" aunque haya un solo APK (Archivo | Arquitectura: universal | Para: Android 5.0 o más) y la nota de actualizar "Android System WebView".
- **Un solo APK** suele alcanzar: si el APK no tiene `lib/` (bibliotecas nativas), es universal y anda en armeabi-v7a y x86. No hay que separar por arquitectura como en Flutter.
- **Lo principal es la WebView** ("Android System WebView"): la de fábrica de Android 5/6 no sirve; actualizada llega a **Chromium 95** (Android 5) y **106** (Android 6). Compilar la web con un `target` que esas versiones entiendan (`es2022` pide Chromium 94).
- CSS y JS más nuevos (`color-mix`, `:has`, `structuredClone`, `Promise.withResolvers`, …): resolver con **respaldos que solo se activan donde falta la función** (`@supports not …`, detección de funciones, compilación `legacy` de la biblioteca), así la web y el celular no cambian.
- **Revisar el APK con `grep`** (en `assets/public/assets/*.js`) que no aparezcan esas funciones sin control. Ojo con las **falsas alarmas**: en ANote... `structuredClone` y `Promise.withResolvers` aparecen una vez cada una desde la v0.2.x, pero dentro de una detección (`typeof Promise.withResolvers=="function"&&typeof structuredClone=="function"…`) que elige la versión de pdfjs. Para saber si es un uso real, mirar el contexto: `grep -o ".\{70\}structuredClone.\{40\}"`, y comparar con el APK de la Release anterior. `AbortSignal.any` también aparece una vez dentro de esa misma detección de pdfjs (falsa alarma; visto en ANote... v0.3.1 a v0.3.3). `Object.hasOwn` aparece una vez en el `dist` de ExcelJS, como `Object.hasOwn||function…` (respaldo propio: falsa alarma; visto en Planillas v0.0.5). De todos modos anda desde Chromium 93.
- **Paneles con campos de texto** (ANote..., v0.3.1): en el APK, si la WebView no se achica al abrir el teclado, el teclado tapa los paneles inferiores. Se resuelve con la variable CSS `--kb` (lo que tapa el teclado) también en el panel (`.kb-open #sheet { bottom: var(--kb) }`) y centrando el campo enfocado (`scrollIntoView`). No es propio de Android viejo, pero conviene probarlo en las tablets.
- **App Capacitor nueva desde cero** (caso Planillas v0.0.1, 2026-11-27): copiar de `notas` el `ci.yml` (versión por marca, mínimo 21, `overrideLibrary` de Cordova, firma fija, `aapt`, paso `if: failure()` de bibliotecas, tabla "¿Qué archivo bajar?"), el atrapador de cierres (los dos `.java`, cambiando paquete y textos), el arreglo de `@capacitor/status-bar` de `patch-android.py`, el aviso ES5 de `index.html` y `target: es2022` en `vite.config.js`. Sumar plugins de a uno, en la etapa que los use, y fijar los que piden más que API 21 (`firebaseAuthVersion` 22.3.1 solo si se usa `@capacitor-firebase/authentication`; `@capacitor/filesystem` 7.0.1). Con `app`, `keyboard` y `status-bar` el APK compiló a la primera con `sdkVersion:'21'`. Falta probarlo en las tablets. En la v0.0.4 se sumaron `@capacitor/filesystem` **7.0.1** y `@capacitor/share` 7.0.4 (exportar con "Guardar o enviar", como ANote...): también compiló a la primera con `sdkVersion:'21'`.
- **Cuenta con correo y contraseña de Firebase sin plugin** (caso Planillas v0.0.9, como ANote... v0.3.0): solo el paquete `firebase` (SDK web, `firebase/auth`) dentro de la WebView; no suma nada nativo ni `google-services.json`, así que el APK sigue con `sdkVersion:'21'` y no depende de Play services. Conviene `initializeAuth(app, { persistence: [indexedDBLocalPersistence, browserLocalPersistence] })` **sin** resolver de ventanas emergentes (no carga el iframe de Firebase, que en el WebView no hace falta para correo) y cargarlo con `import()` dinámico solo al usarlo (en Planillas, ~141 KB aparte). El `grep` del APK no encontró ninguna función nueva. *A confirmar en las tablets.*
- **Firestore sin plugin** (caso Planillas v0.0.10, como ANote...): el SDK web (`firebase/firestore`) dentro de la WebView, sin nada nativo; APK universal con `sdkVersion:'21'`. Si la app ya guarda su copia de trabajo en IndexedDB, alcanza `initializeFirestore(app, { localCache: memoryLocalCache() })` (sin caché persistente: menos trabajo para la WebView vieja); ANote... usa `persistentLocalCache`. Cargarlo con `import()` dinámico solo con la sesión iniciada (en Planillas, ~321 KB aparte). El `grep` del APK no encontró ninguna función nueva. *A confirmar en las tablets.*
- **Bibliotecas grandes de JavaScript** (caso Planillas v0.0.4, ExcelJS ~940 KB): cargarlas con `import()` dinámico (solo al usarlas) y preferir las que traen un `dist` ya pasado a ES5 con sus polyfills (el de ExcelJS lo es): no suben el mínimo de la WebView. El `grep` del APK no encontró ninguna función nueva. Para leer CSV viejos de Excel en Windows, `TextDecoder('windows-1252')` anda en Chromium 94.
- **Importar archivos con `<input type="file">`** (ANote..., Planillas): Capacitor pasa `accept` al selector del sistema convirtiendo extensiones a tipos MIME. Algunos administradores de archivos viejos marcan el CSV como `text/comma-separated-values`: conviene ponerlo en `accept` junto a `text/csv` y `.csv`. *A confirmar en las tablets.*
- **App Links** (ANote... v0.2.8, Planillas v0.0.13): `intent-filter` con `android:autoVerify="true"` (https, host y `pathPrefix` de la app) en la actividad principal, desde `patch-android.py`, y `.well-known/assetlinks.json` en el repo `luchorcompartidos.github.io` (un bloque `target` por app, con la SHA-256 de la firma: `keytool -printcert -jarfile app.apk`). **Android 5 no verifica App Links:** el link muestra «Abrir con…» (Planillas o el navegador); es normal y no hay que arreglar nada. Desde Android 6 abre la app directo (si no, Información de la app → Abrir de forma predeterminada). Para recibir el link alcanza `@capacitor/app` (`getLaunchUrl()` al abrir y `appUrlOpen` con la app abierta; la actividad de Capacitor es `singleTask`): no suma plugins ni sube el mínimo de Android. ANote... usa su plugin propio porque además recibe archivos. *A confirmar en las tablets (Planillas).*
- **WebAssembly pesado (caso ANote... v0.5.0, leer el texto de una imagen con Tesseract.js 7):** anda dentro de la WebView sin nada nativo; el APK sigue universal con `sdkVersion:'21'`. Usar el núcleo **con SIMD** (`tesseract-core-simd-lstm.wasm.js`, desde Chromium 91) y pasar `corePath` como **archivo**, no como carpeta (con carpeta, en los Chrome nuevos pide la variante `relaxedsimd`). Detectar SIMD con `WebAssembly.validate` y, si falta, deshabilitar la función con el motivo. Cargar la biblioteca con `import()` dinámico, el worker del mismo origen (`workerBlobURL: false`) y cerrarlo tras unos minutos sin uso (memoria). Achicar las fotos antes (2400 px). El `grep` de los archivos del lector solo encontró `.at(` (Chromium 92). *A medir el tiempo en las tablets.*
- **Assets `.gz` en el APK:** al armar el APK, Android **saca la extensión `.gz`** de los archivos de `assets/` y los guarda descomprimidos (`x.traineddata.gz` → `x.traineddata`). Si la app pide el `.gz`, en el APK da 404 aunque en la web ande. Revisarlo con `unzip -l` del APK y pedir el nombre sin `.gz` cuando `isNative` (caso ANote... v0.5.1).
- Mostrar un **aviso en ES5** en `index.html` si la WebView es demasiado vieja (en vez de pantalla en blanco), y la versión de Android y WebView en "Acerca de"/"Versión".

## 9. Historial
- 2026-11-24 — Guía creada a partir del caso DiskCompare v0.0.3 (tablets Android 6.0.1 y Bangho AERO J1216 Android 5.0).
- 2026-11-24 — Se aplica solo a las apps marcadas **Sí** en la fila "Dispositivos viejos (Android 5/6)" de la plantilla `DESARROLLO.md`.
- 2026-11-25 — Sección 8: apps Capacitor (caso ANote..., v0.2.1).
- 2026-11-25 — Sección 8: bibliotecas que piden más que API 21 y verificación en el CI (caso ANote..., v0.2.2).
- 2026-11-25 — Sección 8: cierre al abrir en Android 5.0, atrapador de cierres y arreglo de `@capacitor/status-bar` (caso ANote..., v0.2.5–v0.2.6).
- 2026-11-26 — Sección 8: ANote... v0.2.6 probada con éxito en Android 5.0 y 6.0.1 (cierre de la Parte 3); portapapeles con plugin propio (v0.2.7).
- 2026-11-26 — Secciones 6 y 8: inicio con correo y contraseña de Firebase como alternativa sin Play services (ANote... v0.3.0), falsas alarmas al revisar el APK con `grep` y paneles con teclado (v0.3.1). Regla nueva: revisar y actualizar esta guía al cerrar cada etapa de una app con **Sí**.
- 2026-11-26 — Sección 8: `AbortSignal.any` también es falsa alarma dentro de la detección de pdfjs (ANote... v0.3.2). Ocultar/mostrar por reglas (v0.3.2–v0.3.3) no sumó nada nativo ni CSS/JS nuevo.
- 2026-11-26 — ANote... v0.3.5–v0.3.6 (tablas: anchos al exportar y arrastrar para seleccionar) no sumaron nada nativo ni CSS/JS nuevo: el `grep` del APK da las mismas cuentas que en la v0.3.4. Un `touchmove` no pasivo (para arrastrar sin desplazar) anda en Chromium 94.
- 2026-11-26 — ANote... v0.3.7–v0.3.8 (tablas: combinar y separar celdas, y operaciones con celdas combinadas) no sumaron nada nativo ni CSS/JS nuevo: el `grep` del APK da las mismas cuentas que en la v0.3.4.
- 2026-11-26 — ANote... v0.3.9 (tablas: exportar celdas combinadas a PDF, docx, md y txt) no sumó nada nativo ni CSS/JS nuevo: el `grep` del APK da las mismas cuentas que en la v0.3.8 (y la v0.3.4). Cierra combinar celdas.
- 2026-11-26 — ANote... v0.4.0 (administración de usuarios: listado, cerrar sesión en todos los dispositivos, pedir contraseña propia) no sumó nada nativo ni CSS/JS nuevo: el `grep` del APK da las mismas cuentas que en la v0.3.9 (falsas alarmas de pdfjs). Sin cambios en las secciones.
- 2026-11-27 — Sección 8: app Capacitor nueva desde cero con el esqueleto de ANote... (caso Planillas v0.0.1: compiló a la primera con mínimo Android 5.0). Planillas se suma a las apps con **Sí**.
- 2026-11-27 — ANote... v0.4.1 (solo CI: reintento de `deploy-pages`) no cambió la app: nada nuevo para esta guía.
- 2026-11-27 — Planillas v0.0.3 (Etapa 1b: lista de archivos en IndexedDB, vista Ficha, insertar/borrar filas y columnas, menú de celda con mantener apretado = evento `contextmenu`) no sumó nada nativo ni plugins. Lo usado anda desde Chromium 94 (`IDBObjectStore.getAll` desde el 48; `??` y `?.` desde el 80). Al salir de la app se guarda además una copia sincrónica en `localStorage` por si IndexedDB no llega a escribir. CI: reintento automático de `./gradlew` por el 429 de Maven (ver `ESTANDAR-VERSIONES.md`). Sin cambios en las secciones.
- 2026-11-27 — Sección 8: Planillas v0.0.4 (Etapa 1c: importar/exportar `.xlsx` y `.csv`) suma `@capacitor/filesystem` 7.0.1 y `@capacitor/share` con mínimo Android 5.0 sin problemas; bibliotecas grandes con `import()` dinámico y `dist` en ES5 (ExcelJS); `accept` del selector de archivos con `text/comma-separated-values`. El `grep` del APK da 0 en todas las funciones nuevas.
- 2026-11-27 — Planillas v0.0.5 (Etapa 1d-1a: vista Reporte con tema oscuro/claro por variables CSS y clase, franjas con `:nth-child`, títulos y primera columna con `position: sticky`, tema "Sistema" con `matchMedia('(prefers-color-scheme: dark)')`) no sumó nada nativo ni plugins; todo anda desde Chromium 94. APK universal con `sdkVersion:'21'`. Sección 8: `Object.hasOwn` de ExcelJS es falsa alarma en el `grep`.
- 2026-11-28 — Planillas v0.0.6 (Etapa 1d-1b: etiquetas de color con paleta fija por tema, buscador `type="search"`, chips deslizables, orden con `Intl.Collator('es', { numeric: true })` y búsqueda sin acentos con `normalize('NFD')`) no sumó nada nativo ni plugins; todo anda desde Chromium 94. APK universal con `sdkVersion:'21'`; el `grep` del APK da las mismas cuentas que en la v0.0.5 (solo `Object.hasOwn` de ExcelJS, falsa alarma). Sin cambios en las secciones.
- 2026-11-28 — Planillas v0.0.7 (Etapa 1d-2: agrupar con subtotales y grupos plegables, "Configurar vista" como hoja inferior con `select` y `checkbox` nativos, acentos por clase con colores fijos por tema, fila de grupo con `position: sticky` dentro de una celda con `colspan`) no sumó nada nativo ni plugins. Lo único nuevo de CSS es `accent-color` en las casillas (Chromium 93; donde falte, quedan con el color de fábrica). APK universal con `sdkVersion:'21'`; el `grep` del APK da las mismas cuentas que en la v0.0.6 (solo `Object.hasOwn` de ExcelJS, falsa alarma). Sin cambios en las secciones.
- 2026-11-28 — Sección 8: Planillas v0.0.8 (Etapa 1e: escribir en la celda, anchos/altos arrastrando con Pointer Events + `setPointerCapture` y `touch-action: none` en el borde, rangos con mantener y arrastrar —`touchmove` no pasivo, como ANote...—, copiar/pegar) suma el plugin propio `Portapapeles` de ANote... con una `MainActivity` propia; compiló a la primera. Lo demás (`canvas.measureText`, `datalist`, `DataTransfer`/`ClipboardEvent`) anda desde Chromium 94. APK universal con `sdkVersion:'21'`; el `grep` del APK da las mismas cuentas que en la v0.0.7 (solo `Object.hasOwn` de ExcelJS, falsa alarma).
- 2026-11-28 — Sección 8: Planillas v0.0.9 (Etapa 2a: cuenta con correo y contraseña) suma el SDK web de Firebase Auth sin plugin nativo, con `initializeAuth` sin resolver de ventanas y `import()` dinámico. APK universal con `sdkVersion:'21'`; el `grep` del APK da las mismas cuentas que en la v0.0.8 (solo `Object.hasOwn` de ExcelJS, falsa alarma).
- 2026-11-28 — Sección 8: Planillas v0.0.10 (Etapa 2b-1: nube con Firestore) suma el SDK web de Firestore sin plugin, con `memoryLocalCache` e `import()` dinámico. APK universal con `sdkVersion:'21'`; el `grep` del APK da las mismas cuentas que en la v0.0.9 (solo `Object.hasOwn` de ExcelJS, falsa alarma).
- 2026-11-28 — Planillas v0.0.11 (Etapa 2b-2a: compartir planillas por invitación; solo lectura con `readonly` y una clase en `body`) no sumó nada nativo ni plugins ni funciones nuevas: usa el mismo SDK web de Firestore (más `updateDoc`, `arrayUnion`, `deleteField` y escuchas). APK universal (sin `lib/`) con `sdkVersion:'21'`; el `grep` del APK da las mismas cuentas que en la v0.0.10 (solo `Object.hasOwn` de ExcelJS, falsa alarma). Sin cambios en las secciones.
- 2026-11-28 — Planillas v0.0.12 (Etapa 2b-2b-1: link con pedido de acceso; link con `#/planilla/{id}?o=…` leído al abrir y limpiado con `history.replaceState`, «Enviar link» con `@capacitor/share` ya instalado o `navigator.share`) no sumó nada nativo ni plugins ni funciones nuevas (`CSS.escape` existe desde Chromium 46). APK universal (sin `lib/`) con `sdkVersion:'21'`; el `grep` del APK da las mismas cuentas que en la v0.0.11 (solo `Object.hasOwn` de ExcelJS, falsa alarma). Abrir el link en el APK queda para la 2b-2b-2 (App Links). Sin cambios en las secciones.
- 2026-11-28 — Sección 6: DiskCompare v0.0.7 (pestaña Duplicados) lee el contenido de los archivos en Dart puro (`dart:io` + `Uint64List`, sin `crypto`), en isolate y por bloques; sigue compilando con Flutter 3.32 para el APK android5-6. *A medir en las tablets.*
- 2026-11-28 — Sección 8: App Links en apps Capacitor (Android 5 muestra «Abrir con…»; `@capacitor/app` alcanza para recibir el link) (caso Planillas v0.0.13, como ANote... v0.2.8).
- 2026-11-28 — Sección 8: ANote... v0.5.0–v0.5.1 (leer el texto de una imagen con Tesseract.js): WebAssembly con SIMD sin nada nativo (APK universal con `sdkVersion:'21'`, de 6,5 a 13 MB) y Android saca la extensión `.gz` de los assets del APK (v0.5.1). *Falta medir en las tablets.*
