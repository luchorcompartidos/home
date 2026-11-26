# DESARROLLO — [Nombre de la app]

> Archivo para retomar el trabajo en un chat nuevo.
> Uso: *"Seguimos con [app], etapa [X.Y]. Leé `DESARROLLO.md` del repo luchorcompartidos/[repo]."*
> Mantenerlo corto (idealmente menos de 1 página). Lo viejo va resumido en "Historial".
> Apps con APK: si la fila "Dispositivos viejos" falta o dice "Sin definir", preguntarle al usuario una sola vez y anotar la respuesta.
> Apps con **Sí** en "Dispositivos viejos": al cerrar cada etapa, revisar si lo hecho deja algo nuevo para `plantilla/APK-DISPOSITIVOS-VIEJOS.md` del repo home (un problema, un arreglo, una versión de biblioteca, una prueba en las tablets) y, si corresponde, actualizarla (sección del tema + Historial) y subirla. No hace falta que el usuario lo pida.

---

## 1. Ficha rápida
| Campo | Valor |
|---|---|
| App | [Nombre] |
| Repo | luchorcompartidos/[repo] |
| URL publicada | [https://luchorcompartidos.github.io/[repo]/ o "—"] |
| Tipo | [Web HTML / Android APK / Web + Android / …] |
| Estado | [Idea / Prototipo / En uso] |
| Última versión (Release) | [vX.Y.Z] — [AAAA-MM-DD] |
| Se conecta con | [Google Sheets / Firebase / Softr.io / ninguno] |
| Dispositivos viejos (Android 5/6) | [Sí / No / Sin definir / No aplica (sin APK)] — Sí: aplicar `plantilla/APK-DISPOSITIVOS-VIEJOS.md` del repo home en cada cambio y, al cerrar cada etapa, revisar si lo hecho deja algo nuevo para esa guía (un problema, un arreglo, una versión de biblioteca, una prueba en las tablets) y, si corresponde, actualizarla (sección del tema + Historial) y subirla al repo home |

## 2. Etapa actual
**Etapa [X.Y] — [nombre corto de la etapa]**
- Objetivo de la etapa: [qué se busca lograr]
- Hecho hasta ahora: [1–3 viñetas]
- En curso: [lo que quedó a medio hacer, si hay algo]

## 3. Próximos pasos
1. [Paso concreto siguiente]
2. [Paso concreto]
3. [Paso concreto]

## 4. Decisiones tomadas
- [AAAA-MM-DD] [Decisión] — [motivo en pocas palabras]
- [AAAA-MM-DD] [Decisión] — [motivo]

## 5. Pendientes / dudas abiertas
- [ ] [Algo que falta definir o resolver]
- [ ] [Bug conocido o mejora pedida]

## 6. Datos técnicos clave
- Archivos principales: [index.html, app.js, …]
- Configuración / servicios: [IDs, hojas, proyectos (sin claves ni contraseñas)]
- Cómo se publica: [push a main → GitHub Pages / GitHub Actions compila APK / …]
- Cómo se prueba: [node, navegador, en el equipo…]. En el entorno de Claude no hay red para `npm install`, pero sí `git clone` desde GitHub y hay `esbuild` y `typescript` en `/opt/npm-tools` (y `bun`): se pueden bajar las bibliotecas desde su repo de GitHub y armar un paquete de prueba con el código real (ejemplo: `pruebas/tablas/` del repo notas, con ProseMirror).
- Releases: según `ESTANDAR-VERSIONES.md` del repo home (`[major]` / `[minor]` / `[patch]` / `[skip release]` en el commit; cambios que no tocan la app van sin release)

## 7. Historial (resumido)
- [vX.Y] [AAAA-MM-DD] — [qué cambió, una línea]
- [vX.Y] [AAAA-MM-DD] — [qué cambió, una línea]

---
*Al cerrar cada sesión: actualizar secciones 1, 2, 3 y 7, y subir el archivo al repo. Si "Dispositivos viejos" es **Sí**, revisar también `plantilla/APK-DISPOSITIVOS-VIEJOS.md` (repo home) y actualizarla si corresponde.*
