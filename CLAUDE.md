# Geriatría UC · Guía del rotante — handoff para Claude Code

Documento de traspaso escrito por Marshall (asistente Hermes de Hugo) el 22-10-2026 para que Claude Code
continúe el desarrollo. Léelo completo antes de tocar nada. Está en español porque el usuario y las apps lo son.

---

## 1. Qué es esto (contexto humano)

- **Dueño**: Dr. Hugo González (internista + geriatra, UC CHRISTUS, Santiago de Chile). Coordinador docente de
  rotantes de Geriatría UC y, desde 17-08-2026, jefe de la UGA (Unidad Geriátrica de Agudos) de la Clínica San
  Carlos de Apoquindo. **Ojo**: UGA = San Carlos; Hospital Clínico UC = Marcoleta = Sala UC. No mezclarlos.
- **Origen de la idea**: Homero Gac (jefe de Geriatría UC, Hugo le dice "Profe") propuso una app para que becados y
  rotantes sepan cómo funciona la rotación: dónde ir, con quién, qué se espera, plantillas, calculadoras.
- **Usuarios finales**: becados de Geriatría, rotantes de Medicina Interna / Medicina Familiar / internos, en celular,
  muchas veces sin señal dentro del hospital. Por eso: **mobile-first, un solo HTML autocontenido, funciona offline
  y desde `file://`**.
- **Estado**: prototipo. Hugo lo mostró y le gustó ("Eso. Excelente"). Los contenidos vienen de material real
  (Drive de docencia, grupo WhatsApp "Geriatría UC ampliado", skills internas de Hugo), pero muchos datos los
  tiene que cerrar Hugo (ver §6).

## 2. Dos apps, no una

| App | Archivo maestro (en este repo) | URL pública |
|---|---|---|
| **General** "Geriatría UC · Guía del rotante" v0.3 | `index.html` (162 KB) | https://hugonzalez10.github.io/geriatria-uc-app/ |
| **San Carlos** "Geriatría UC · San Carlos de Apoquindo" v0.2 | `san-carlos/index.html` (143 KB) | https://hugonzalez10.github.io/geriatria-uc-app/san-carlos/ |

El repo **es** la fuente de verdad: se edita directo aquí. No hay copias maestras fuera del repo.

- Hugo pidió explícitamente **dos archivos separados**: "Generame una aparte para san carlos. Manten esta como
  geriatria uc". La general cubre toda la rotación (ambulatorio, sedes, poli, docencia); la de San Carlos es solo
  la UGA de la Clínica San Carlos (turnos, logística). Sala UC (Marcoleta) va solo en la general.
- Versiones previas: viven en el historial de git (ver commits en §3), no como archivos `.vX.Y.html`. Para
  recuperar una: `git show <commit>:index.html`.
- San Carlos v0.1 se generó recortando la general v0.2; desde v0.2 se edita a mano. Edita
  `san-carlos/index.html` directamente.

## 3. Repo y publicación

- Repo público: `hugonzalez10/geriatria-uc-app` (GitHub Pages, rama `main`, raíz `/`). Clon local de trabajo:
  `~/Documents/geriatria-uc-app` (este directorio). Commits de versiones: `a842585` v0.2, `bc46b0d` SC v0.1,
  `1ce21d6` general v0.3 + SC v0.2.
- Flujo de actualización:
  1. Editar `index.html` y/o `san-carlos/index.html` directo en este repo.
  2. Validar: `node --check` no sirve directo sobre HTML; extraer el `<script>` o abrir en navegador y mirar consola (0 errores exigido).
  3. Correr el grep de teléfonos/correos de §5.2.
  4. `git commit -am "…"` (el git local ya está configurado como Hugo González).
  5. `git push` (si pide usuario: `gh auth setup-git` una vez; `gh` ya está autenticado como hugonzalez10).
  6. Pages tarda ~2 min: `gh api repos/hugonzalez10/geriatria-uc-app/pages/builds/latest --jq .status` → `built`.
- Telegram no abre HTML en el celular: a Hugo siempre se le manda la **URL**, no el archivo.

## 4. Arquitectura del HTML (igual en ambas apps)

Un archivo: `<style>` (línea ~38–226) + markup mínimo + un único `<script>` dividido en bloques comentados:

```
[CSS]      variables (--uc azul UC), layout mobile, cards, tablas, modal, toast, nav inferior
[JS-0]     utilidades: esc(), $(), show(view), modal(title, html), toast(), storage helpers
[JS-1]     Rotación: bienvenida, organización, semana tipo, reglas por tipo de rotante, poli SJ, turnos FDS, sedes, expectativas, evaluación, checklist 1er día, contacto
[JS-2]     Calculadoras: motor genérico → const CALCS = []; CALCS.push({id, abbr, name, desc, type:'items'|'custom', ...})
[JS-3]     Directorio: const DIR_SECTIONS; contactos editables, import/export JSON
[JS-4]     Recursos: referencias + enlaces propios (editable)
[JS-6]     Plantillas: const TPLS (VGI HGG 12 secciones, entrega de turno 8 col/4 ejes, interconsulta, evolución; SC agrega traslado/epicrisis). Borradores SOLO en sessionStorage
[JS-7]     Más: CHECKLISTS, GLOSARIO, PAUTA_DEFAULT (borrador evaluación 10 criterios 1–7), calendario, "Mi semana"
[JS-8]     v0.3: AVISO_DEFAULT (banner contingencia editable), SUPERVISION_SJ, TURNOS_FDS, EQUIPO (organigrama + "a quién acudir"), deep-link #equipo
[JS-5]     Arranque: render inicial, hash routing
```

- 5 pestañas (nav inferior): **Rotación · Plantillas · Calculadoras · Directorio · Más**. Vistas `#view-rotacion`,
  `#view-plant`, `#view-calc`, `#view-dir`, `#view-mas`. Modal `#modal`, toast `#toast`, banner `#banner`.
- Persistencia: `localStorage` para directorio, enlaces, pauta, mi semana, aviso editado. En San Carlos todas las
  claves van con prefijo **`sca:`** (`SK(k)`) para no colisionar con la general si ambas se abren en el mismo
  navegador. Plantillas clínicas → `sessionStorage` (se borran al cerrar: privacidad).
- Calculadoras actuales. General (30): CFS, Barthel, Katz, CAM, Mini-Cog, Charlson, Cockcroft-Gault, CHA2DS2-VASc/
  HAS-BLED (FA), TEV/Caprini, FRAX (enlace externo, no cálculo propio), TUG, IMC, FAST, Lawton, Yesavage, MNA-SF,
  Braden, FRAIL, ACB (~85 fármacos), 4AT, CKD-EPI 2021, SPPB, SARC-F, EAT-10, CURB-65, Zarit, + accesos a plantillas
  HGG/ET/IC/EVO. San Carlos (26): subconjunto hospitalario + Bush-Francis (catatonía), Glasgow, ATS/IDSA NAC grave,
  plantilla traslado (TR).
- Datos de sedes (general, tabla "Sedes UC CHRISTUS"): Santa Lucía = Capitán Orellana 2467 (Santiago); Vitacura =
  Av. Vitacura 5670; El Alba = Camino El Alba 9500, piso 4, torre A (Las Condes); Escuela Militar = Av. Américo
  Vespucio 100; Irarrázaval 4949 (Ñuñoa); Bilbao = Francisco Bilbao 4144; Alcántara 3990 (Las Condes); San Joaquín =
  Av. Vicuña Mackenna 4686 (Macul) — corregido por Hugo; Edificio Académico = Diagonal Paraguay `[n°]` (Santiago);
  Hospital San Carlos = Camino El Alba 12351 (Las Condes); Hospital Clínico = Marcoleta 367. HLF / San Jorge / Lira
  siguen `[completar]`.

## 5. Reglas duras (no negociables, vienen de Hugo)

1. **Nunca datos de pacientes** (ni nombres, ni RUT, ni camas con nombre). Las plantillas piden edad/cama, nunca identidad.
2. **Nunca teléfonos ni correos en el HTML** (el repo es público). Se verificó con regex antes de cada publicación;
   repítelo: `grep -En '\+?56 ?9|\b9[0-9]{8}\b|[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[a-z]{2,}' index.html san-carlos/index.html`
   debe devolver nada (salvo URLs legítimas).
3. **No inventar**: nombres, horarios, teléfonos, protocolos, fechas. Si falta, placeholder `[completar]` /
   `[completar: qué]` / `[pendiente …]`. Hugo revisa y rellena. Mejor placeholder que dato plausible.
4. Español de Chile, tono clínico sobrio, sin anglicismos innecesarios. Pie obligatorio: "No reemplaza el juicio clínico".
5. Un solo HTML por app, sin dependencias externas (nada de CDN, fuentes web, frameworks). Vanilla JS. Debe abrir
   desde `file://` sin consola roja.
6. Mobile-first (iPhone/Android), pero usable en desktop. Ligero: hoy 143–162 KB, no inflar.
7. Mantener las dos apps **separadas** y con el mismo CSS/JS base para que un fix se pueda portar.
8. Antes de publicar algo que cambie contenido clínico u organizacional, mostrarle a Hugo qué cambia (él valida).

## 6. Placeholders que solo Hugo puede cerrar (preguntarle, no adivinar)

General: a quién llamar cuando no hay SOS de geriatría (no existe SOS UCI/Intermedio), fechas Core/seminario
delirium 2026, tope de rotantes simultáneos, fecha reapertura San Carlos y El Alba tras el aluvión (08-10-2026),
links reales de Sheet de ausencias / planilla / Saturno, apellidos de "Isi" y "Mapi", direcciones HLF / San Jorge /
Lira, número de Diagonal Paraguay, horas del día tipo, hora y lugar de entrega de turno y visita, quién gestiona
perfiles TrakCare por centro, protocolos locales.

San Carlos: puerta de acceso y casilleros, hora y lugar de la entrega de turno UGA, tratante de la semana,
contingencia post-aluvión (`[pendiente reapertura]`), cómo ubicar a cada miembro del equipo, a quién llamar sin SOS.

## 7. Contexto organizacional que ya está cargado (para no re-preguntar)

- Organigrama: Carrasco (jefa Programa) → Hoyl (programa/MedFam), González (UGA SC + calendario semanal de rotantes,
  se arma domingo, imagen+correo, Danke revisa), Born (Sala UC Marcoleta), Danke, Gac (domicilio / UC en Casa,
  ex-jefe UGA), Browne (investigación), Bórquez (calendarios maestro "CALENDARIOS 2026"), Rodríguez (seminario
  delirium), Meza/Cáceres (HLF), Silva, Piquer.
- Poli becados San Joaquín: mar/mié 14:00, sala espejo "en el 36, por atrás"; supervisión rotativa los miércoles
  jul–dic 2026 (tabla SUPERVISION_SJ). Regla Born 02-10: mar/mié PM nadie en Sala UC.
- Turnos fin de semana: visita sáb-dom por 8 staff, ~1 cada 7–8 semanas, jul-2026→ene-2027 (TURNOS_FDS); traspaso
  el viernes; **no existe SOS de geriatría** en UCI/Intermedio.
- Fijos: Zoom martes 08:00, reunión 5A jueves 14:00, Core jueves 15:00, UGA visita desde 09:00, reunión multidisciplinaria UGA miércoles 08:30 (gimnasio).
- Contingencia: aluvión San Carlos 08-10-2026 → clínica cerrada, agendas bloqueadas oct–nov, Gac/González/Marín
  reubicados, rotantes UGA → Marcoleta / HLF. Con San Carlos cerrado Hugo pasa visita mié/jue en Marcoleta.
- Formato UC: VGI "formato HGG" 12 secciones; entrega de turno 8 columnas / 4 ejes; checklists delirium, caídas,
  UPP, IDDSI, tromboprofilaxis, desprescripción; alta segura; UC en Casa.
- Pauta de evaluación de rotantes: **no existe oficialmente**; la app trae un borrador de 10 criterios (1–7) "a
  definir con Gac/Hoyl".

## 8. Fuentes de donde salió el contenido (en el iMac de Hugo, usuario `hugonzalez10`)

- `~/.hermes/data/app_geriatria_analisis_drive_22-10.log` — análisis del Drive de docencia (`gdrive:Docencia`,
  `Rotaciones_Geriatria.xlsx`; `gdrive:Hospital` con formato HGG). rclone remote `gdrive:`.
- `~/.hermes/data/app_geriatria_analisis_grupo_22-10.md` — lectura del grupo WhatsApp "Geriatría UC ampliado"
  (29-06→22-10-2026), secciones A–F: supervisión SJ, turnos FDS, organigrama, formato calendario, temas, sugerencias.
- Skills Hermes con el saber hacer: `~/.hermes/profiles/rotantes/skills/openclaw-imports/calendario-rotantes*/`
  (incl. `references/personas-y-sitios.md` con direcciones) y `.../turno-medicina-uc/`.
- Los PNG de los calendarios del grupo no se pudieron bajar; si Hugo los pasa, sirven para las tablas.

## 9. Ideas pendientes / backlog sugerido (no comprometido con Hugo)

- Exportar plantillas a texto/clipboard con un toque (ya hay copiar; falta "compartir" nativo).
- Modo "mi rotación": elegir tipo de rotante y que la app filtre reglas/semana tipo.
- Importar las tablas (supervisión, FDS) desde un JSON/CSV editable en vez de constantes en JS.
- PWA mínima (manifest + service worker) para "Agregar a pantalla de inicio" con ícono; cuidado con cachés viejas.
- Versionado visible en el pie y un "qué hay de nuevo".
- Test rápido en Node que extraiga el `<script>` y haga `new Function()` para detectar errores de sintaxis en CI.

## 10. Cómo trabajar con Hugo

- Respuestas cortas, en español, con negritas y emojis de estado; un renglón vacío entre ítems. Nada de prosa larga.
- Antes de ejecutar algo con efecto externo (publicar, borrar, cambiar contenido clínico): mostrar el plan y esperar
  el "dale".
- Si un dato falta, decirlo y parar; no rellenar con algo verosímil.
