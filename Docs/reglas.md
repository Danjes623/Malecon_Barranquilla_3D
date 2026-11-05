# Reglas de trabajo

> Normas de trabajo para Daniel Mejía y Meriyein Manosalva. Se cambian por PR aprobado por ambos.

## 1. Organización

- Toda tarea existe como **issue** en GitHub con su ID de `fases-tareas.md` (`[F3-04] Modelar Luna del Río`).
- Cada issue tiene: responsable, etiqueta de fase (`fase-3`), etiqueta de área (`3d`, `web`, `contenido`, `infra`, `legal`, `security`) y criterio de aceptación.
- El tablero (GitHub Project) tiene columnas: **Backlog → Por hacer → En curso → En revisión → Hecho**.
- Máximo **2 tareas "En curso"** por persona.
- Reunión corta de sincronización 1–2 veces por semana: qué hice, qué haré, qué me bloquea.
- Una tarea bloqueada más de 2 días se comenta en el issue y se habla en la siguiente sincronización.

## 2. Ramas

- `main`: siempre desplegable. Protegida.
- Ramas de trabajo desde `main`, de vida corta (idealmente < 1 semana):

| Prefijo | Uso | Ejemplo |
|---|---|---|
| `feat/` | Funcionalidad web | `feat/F5-02-seleccion-click` |
| `model/` | Trabajo en Blender / GLB | `model/F3-04-luna-del-rio` |
| `content/` | Textos, fotos, JSON de zonas | `content/F3-16-textos-mvp` |
| `fix/` | Corrección | `fix/panel-esc-movil` |
| `docs/` | Documentación | `docs/actualizar-design` |
| `chore/` | Configuración, dependencias | `chore/F1-06-ci` |

- Actualizar la rama con `git pull --rebase origin main` antes de abrir el PR.

## 3. Commits

Formato **Conventional Commits**, en español, en imperativo:

```
<tipo>(<ámbito>): <descripción corta>

[cuerpo opcional: por qué]

Refs #<issue>
```

- Tipos: `feat`, `fix`, `model`, `content`, `docs`, `test`, `chore`, `refactor`, `perf`.
- Ejemplos:
  - `model(luna): modelar rueda low-poly y hitbox`
  - `feat(escena): sincronizar zona seleccionada con ?zona=`
  - `content(caiman): textos ES/EN con fuentes`
- Commits pequeños y con un solo propósito.

## 4. Pull Requests

- Un PR = una tarea (o pocas muy relacionadas). Título con el ID: `[F5-02] Selección por clic`.
- Plantilla del PR:
  ```
  ## Qué cambia
  ## Cómo probarlo (enlace al preview de Vercel)
  ## Capturas / GIF (obligatorio si es visual o 3D)
  ## Checklist
  - [ ] CI en verde
  - [ ] Sin secretos ni archivos prohibidos
  - [ ] CREDITS.md actualizado (si hay assets nuevos)
  - [ ] Textos ES y EN (si hay contenido)
  - [ ] Docs actualizados (si cambia diseño o reglas)
  Cierra #<issue>
  ```
- **Revisión cruzada obligatoria**: Daniel revisa los PR de Mery y Mery los de Daniel.
- La revisión se hace en menos de 48 h. Comentarios concretos y amables; se aprueba o se piden cambios, no se deja en silencio.
- Fusión con **Squash and merge**. Borrar la rama al fusionar.

## 5. Reglas de modelado 3D (Daniel)

- Un único `.blend` principal en `blender/` (Git LFS). Avisar en el issue antes de editarlo para evitar conflictos (los binarios no se fusionan).
- Seguir las convenciones de nombres de `design.md` §7.4 y poner Custom Properties en todo objeto clickeable.
- Un solo material con textura-paleta. Si se necesita un color nuevo, se añade a la paleta.
- Cada hito se modela como pieza independiente sobre su losa.
- Antes de abrir PR de modelo: ejecutar `npm run model`, revisar `gltf-transform inspect` (nombres intactos, draw calls, triángulos, tamaño) y adjuntar captura del render.
- Las capas `ref_*` no se exportan.
- Cada asset externo se registra en `CREDITS.md` en el mismo PR.

## 6. Reglas de código (Mery y Daniel)

- TypeScript `strict`. Sin `any` salvo justificación en comentario.
- ESLint + Prettier sin errores; `npm run lint`, `typecheck` y `test` antes de subir.
- Componentes 3D solo en `src/components/scene/` y siempre client-only.
- Textos de interfaz en `src/messages/{es,en}.json`, nunca escritos en el componente.
- Contenido de zonas en `content/zonas/*.json`, validado con Zod.
- Todo cambio funcional lleva prueba (unidad o E2E) cuando sea razonable.
- `Malecon.tsx` es generado: no se edita a mano; se regenera con `npm run model`.

## 7. Reglas de contenido

- Cada zona tiene `es` y `en` completos.
- Todo dato verificable tiene enlace en `sources`.
- Datos dudosos se marcan como no verificados o no se publican.
- Coordenadas con `verified:false` hasta contrastarlas.
- Imágenes con `alt` en ES y EN y `credit`.

## 8. Versionado y entregas

- Etiquetas SemVer al cerrar cada fase (`v0.1`, `v0.2`, `v0.5`, `v1.0.0`; ver `fases-tareas.md`).
- Cada etiqueta tiene *release notes* en GitHub con lo que entró.
- Al cerrar una fase: revisión conjunta, actualizar `design.md` si algo cambió y mover las incertidumbres resueltas.

## 9. Documentación

- `Docs/` es la fuente de verdad del proyecto. Si el código y los docs no coinciden, se corrige uno de los dos en el mismo PR.
- Decisiones importantes (cambio de stack, río ocre vs azul, licencia) se anotan en `design.md` con fecha y responsables.

## 10. Comunicación

- Las decisiones se registran en el issue o PR, no solo en chat.
- Si alguien no puede cumplir una fecha, lo avisa en el issue en cuanto lo sepa.
- Respeto y crítica constructiva en revisiones: se comenta el trabajo, no la persona.
