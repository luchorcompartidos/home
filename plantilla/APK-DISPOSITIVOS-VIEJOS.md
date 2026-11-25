# APK para dispositivos Android viejos (Android 5 y 6)

> Guía para aplicar en cualquier app de la caja de apps que se distribuya como APK.
> Uso en un chat nuevo: *"Seguimos con [app]. Leé `plantilla/APK-DISPOSITIVOS-VIEJOS.md` del repo luchorcompartidos/home y `DESARROLLO.md` del repo luchorcompartidos/[repo], y aplicá la guía."*
> Casos de referencia: **DiskCompare** (Flutter; luchorcompartidos/diskcompare, v0.0.3, `.github/workflows/ci.yml`) y **ANote...** (Capacitor; luchorcompartidos/notas, v0.2.1–v0.2.2, ver sección 8).
> Al terminar en cada app: anotar la decisión en su `DESARROLLO.md` (Decisiones tomadas + Datos técnicos).
> **No aplica a todas las apps:** solo a las que tienen **Sí** en la fila "Dispositivos viejos (Android 5/6)" de la Ficha rápida de su `DESARROLLO.md` (hoy: DiskCompare y Notas). Si la fila falta o dice "Sin definir", preguntar una vez y anotarla.

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
- **Permisos de archivos:** en Android 5 el permiso de almacenamiento se da al instalar; en Android 6 se pide en la app (`Permission.storage`); en Android 11+ se usa `MANAGE_EXTERNAL_STORAGE`. El código tiene que contemplar los tres casos según `SDK_INT`.
- **Notificaciones:** `POST_NOTIFICATIONS` solo existe desde Android 13; en 5 y 6 no se pide.

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
- **Verificar en el CI** con `aapt dump badging` que el APK diga `sdkVersion:'21'`, y en la Release poner la tabla "¿Qué archivo bajar?" aunque haya un solo APK (Archivo | Arquitectura: universal | Para: Android 5.0 o más) y la nota de actualizar "Android System WebView".
- **Un solo APK** suele alcanzar: si el APK no tiene `lib/` (bibliotecas nativas), es universal y anda en armeabi-v7a y x86. No hay que separar por arquitectura como en Flutter.
- **Lo principal es la WebView** ("Android System WebView"): la de fábrica de Android 5/6 no sirve; actualizada llega a **Chromium 95** (Android 5) y **106** (Android 6). Compilar la web con un `target` que esas versiones entiendan (`es2022` pide Chromium 94).
- CSS y JS más nuevos (`color-mix`, `:has`, `structuredClone`, `Promise.withResolvers`, …): resolver con **respaldos que solo se activan donde falta la función** (`@supports not …`, detección de funciones, compilación `legacy` de la biblioteca), así la web y el celular no cambian.
- Mostrar un **aviso en ES5** en `index.html` si la WebView es demasiado vieja (en vez de pantalla en blanco), y la versión de Android y WebView en "Acerca de"/"Versión".

## 9. Historial
- 2026-11-24 — Guía creada a partir del caso DiskCompare v0.0.3 (tablets Android 6.0.1 y Bangho AERO J1216 Android 5.0).
- 2026-11-24 — Se aplica solo a las apps marcadas **Sí** en la fila "Dispositivos viejos (Android 5/6)" de la plantilla `DESARROLLO.md`.
- 2026-11-25 — Sección 8: apps Capacitor (caso ANote..., v0.2.1).
- 2026-11-25 — Sección 8: bibliotecas que piden más que API 21 y verificación en el CI (caso ANote..., v0.2.2).
