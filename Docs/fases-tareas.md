# Fases y tareas

> Responsables: **D** = Daniel Mejía (modelado 3D) · **M** = Meriyein Manosalva (web, contenido, pruebas, despliegue) · **D+M** = conjunta.
> Cada tarea se convierte en un *issue* de GitHub con el ID como prefijo (ej. `[F3-04] Modelar Luna del Río`).
> Estimación total heurística: ~60–125 h de trabajo, repartidas en dos líneas paralelas.

## Cómo se trabaja en paralelo

```
Daniel:  F0 ── F2 (layout) ── F3 (modelado) ─────── F4 (pipeline) ── F7 (recorte)
Mery:    F0 ── F1 (esqueleto) ── F5 (interacción, con GLB provisional) ── F6 ── F7 ── F8 ── (F9)
```

- Mery trabaja con un **GLB provisional** (losas en caja de F2-04) hasta que llegue el modelo final.
- El **contrato** entre ambas líneas es: nombres de objetos + Custom Properties (`design.md` §7.4) y el esquema `Zone` (`design.md` §6).
- Cada fase termina con una revisión conjunta y una etiqueta (`v0.1`, `v0.2`, …).

## Definición de Hecho (DoD) común

Una tarea está hecha cuando: el PR está fusionado en `main`, pasa CI, fue revisado por la otra persona, y el *preview* de Vercel funciona (si aplica).

---

## Fase 0 — Organización, decisiones y referencia (≈1 semana)

| ID | Tarea | Resp. | Entregable |
|---|---|---|---|
| F0-01 | Crear repo GitHub, subir `Docs/`, activar protección de `main`, Dependabot, secret scanning, 2FA de ambos | D | Repo configurado (`plan-seguridad.md`) |
| F0-02 | Crear GitHub Project (tablero Kanban), etiquetas y plantillas de issue/PR | M | Tablero con issues F0–F8 |
| F0-03 | Configurar Git LFS para `*.blend`, `*.glb` fuente y `refs/*` | D | `.gitattributes` |
| F0-04 | Buscar la URL de "Zaragoza cabe en tu mano" e inspeccionar su stack con DevTools | M | Nota en `Docs/` (opcional) |
| F0-05 | Congelar inventario: 9–12 hitos héroe y lista de exclusiones | D+M | Tabla final en `design.md` §7.3 |
| F0-06 | Tomar a mano coordenadas de 8 puntos ancla en openstreetmap.org | M | `content/geo.json` con `verified:false` |
| F0-07 | Reunir referencias visuales propias y de Mapillary/KartaView (nunca Google 3D Tiles) | D | `blender/refs/` + fuentes anotadas |
| F0-08 | Definir paleta (16–32 colores) y decidir río ocre vs azul | D | `blender/palette.png` + decisión en `design.md` |
| F0-09 | Elegir licencia del código y del modelo | D+M | `LICENSE` (ver `plan-legal.md`) |

## Fase 1 — Esqueleto técnico (≈3–5 días)

| ID | Tarea | Resp. | Entregable |
|---|---|---|---|
| F1-01 | `create-next-app` (Next 16.4, TS, Tailwind, ESLint) + `three@0.187`, `@react-three/fiber@9.8`, `@react-three/drei@10.7`, `zustand` | M | App base |
| F1-02 | Escena mínima (cubo clickeable) en client wrapper `dynamic(..., {ssr:false})`; confirmar R3F + React 19.3 | M | Escena sin errores; versiones fijadas |
| F1-03 | Configurar next-intl (`proxy.ts`, `app/[locale]/`) con `es` y `en` | M | Rutas `/es`, `/en` |
| F1-04 | Esquema Zod `Zone` + 3 zonas de prueba en `content/zonas/` | M | `src/lib/schema.ts` |
| F1-05 | Vitest: cada zona tiene ES y EN; `slug` único; `order` único | M | Tests en CI |
| F1-06 | CI en GitHub Actions: lint, typecheck, test, build | M | `.github/workflows/ci.yml` |
| F1-07 | Conectar repo a Vercel (previews por PR) | M | URL de preview (`plan-sitio.md`) |

## Fase 2 — Datos geográficos y layout (≈1 semana)

| ID | Tarea | Resp. | Entregable |
|---|---|---|---|
| F2-01 | Instalar BlenderGIS (Blender 5.x) o Blosm (4.5 LTS); importar orilla, río y Vía 40/Av. del Río | D | Capa `ref_*` en `.blend` |
| F2-02 | Trazar y simplificar la curva de orilla | D | Curva `ref_orilla` |
| F2-03 | Hoja de escalas: longitud de maqueta por sector según importancia | D+M | `blender/escalas.csv` |
| F2-04 | Colocar losas vacías con nombre a lo largo de la curva y validar desde la cámara final | D | **GLB provisional** para Mery |

## Fase 3 — Modelado low-poly (≈2–3 semanas)

| ID | Tarea | Resp. | Entregable |
|---|---|---|---|
| F3-01 | Material único con textura-paleta (interpolación "Closest") | D | Material `mat_palette` |
| F3-02 | Base y río de cada losa (borde de tierra cortada, agua plana) | D | Losas con base |
| F3-03 | Importar y recolorear props CC0 (Kenney/Quaternius) como *linked duplicates* | D | Palmeras, faroles, bancas, canchas |
| F3-04 | Hito: Luna del Río | D | `mon_luna_del_rio` |
| F3-05 | Hito: Ventana de Campeones | D | `mon_ventana_de_campeones` |
| F3-06 | Hito: Pabellón de Cristal | D | `mon_pabellon_de_cristal` |
| F3-07 | Hito: Centro de Eventos Puerta de Oro | D | `mon_puerta_de_oro` |
| F3-08 | Hito: Caimán del Río | D | `mon_caiman_del_rio` |
| F3-09 | Hito: El Ancla / Plaza de los Marinos | D | `mon_el_ancla` |
| F3-10 | Estatuas (siluetas simplificadas, ver `plan-legal.md`) | D | `mon_estatua_*` |
| F3-11 | Hito: Carrusel del Río | D | `mon_carrusel_del_rio` |
| F3-12 | Exagerar hitos 1,5–3× y crear hitbox `*_hit` por hito | D | Hitboxes |
| F3-13 | Nombres `mon_*`/`zone_*` + Custom Properties (`zone_id`, `order`, `clickable`) | D | Contrato cumplido |
| F3-14 | Hornear AO (Cycles) en vertex colors o lightmap | D | AO horneado |
| F3-15 | Registrar cada asset de terceros en `CREDITS.md` | D | `CREDITS.md` |
| F3-16 | Redactar textos ES/EN de cada zona con fuentes | M | `content/zonas/*.json` completos |
| F3-17 | Conseguir/tomar fotos con licencia para cada panel | M | Imágenes + créditos |

## Fase 4 — Pipeline de exportación (≈2 días)

| ID | Tarea | Resp. | Entregable |
|---|---|---|---|
| F4-01 | Exportar `.glb` (+Y Up, Apply Modifiers, Custom Properties, sin Draco) | D | `malecon.raw.glb` |
| F4-02 | `gltf-transform optimize` (meshopt + webp, `--join false --flatten false`) + `inspect` | D | `public/models/malecon.glb` |
| F4-03 | `gltfjsx --types --keepnames --keepgroups --instance` → `Malecon.tsx` | D | Componente tipado |
| F4-04 | Script `npm run model` que automatiza F4-01..03 | D+M | `scripts/model.mjs` |
| F4-05 | Test: todo `meshName` de `content/zonas` existe en el GLB | M | Test en CI |

## Fase 5 — Interacción 3D (≈1–2 semanas)

| ID | Tarea | Resp. | Entregable |
|---|---|---|---|
| F5-01 | `<Canvas frameloop="demand">` + `CameraControls` restringido; `invalidate()` en `change` | M | Navegación limitada |
| F5-02 | `onClick` → `stopPropagation` → `select(zone_id)`; hover con cursor y resaltado | M | Selección |
| F5-03 | Sincronizar store con `?zona=slug` | M | Deep links |
| F5-04 | Vuelo de cámara `setLookAt(...preset, true)`; panel en evento `rest` | M | Vuelo |
| F5-05 | Ajustar presets de cámara por zona desde Blender | D | Presets en JSON |
| F5-06 | Etiquetas `<Html>` solo para hitos héroe | M | Etiquetas |
| F5-07 | "Recorrer la ruta" con Pausa y Siguiente | M | Tour |
| F5-08 | Agua estilizada (UV animada) y barca sobre curva | D | Efectos |
| F5-09 | `TiltShift2` desactivable | M | Post-proceso |

## Fase 6 — Interfaz, accesibilidad y SEO (≈1 semana)

| ID | Tarea | Resp. | Entregable |
|---|---|---|---|
| F6-01 | Panel de info: Sheet (escritorio) / Drawer (móvil), fotos, horario, enlaces, Esc | M | Panel |
| F6-02 | `<nav aria-label="Monumentos">` con `focusLandmark(id)` | M | Lista accesible |
| F6-03 | `prefers-reduced-motion` + fallback sin WebGL (render pre-hecho) | M | Fallback |
| F6-04 | Render estático de la maqueta para fallback y OG image | D | Imágenes |
| F6-05 | Selector ES/EN, modo oscuro (escena incluida), panel "(i) Créditos" con OSM | M | UI |
| F6-06 | Páginas `/[locale]/zona/[slug]` con `generateStaticParams`, `generateMetadata`, `ensureStatic` | M | SEO |

## Fase 7 — Rendimiento y pruebas (≈1 semana)

| ID | Tarea | Resp. | Entregable |
|---|---|---|---|
| F7-01 | Medir con `r3f-perf`; `PerformanceMonitor` + `AdaptiveDpr` (DPR ≤ 1,5) | M | Informe |
| F7-02 | Probar en Android de gama media real | D+M | Resultado anotado |
| F7-03 | Recortar geometría hasta cumplir presupuesto (`design.md` §8) | D | GLB dentro de límites |
| F7-04 | E2E Playwright (swiftshader, `canvas.dataset.ready`, lista DOM, ES/EN, `?zona=`, OG) en Docker | M | Suite E2E en CI |
| F7-05 | Bundle Analyzer (three no en páginas de zona) + Lighthouse | M | Informe |

## Fase 8 — Despliegue (≈1–2 días)

| ID | Tarea | Resp. | Entregable |
|---|---|---|---|
| F8-01 | Producción en Vercel Hobby (`plan-sitio.md`) | M | URL pública |
| F8-02 | Mover GLB/fotos a R2 si pesan (hash, caché inmutable, CORS) | M | Assets en CDN |
| F8-03 | README: cómo regenerar modelo y contenido | D+M | README |
| F8-04 | Etiqueta `v1.0.0` y release notes | D | Release |

## Fase 9 — Backend opcional (≈1–2 semanas, solo si el curso lo pide)

| ID | Tarea | Resp. | Entregable |
|---|---|---|---|
| F9-01 | Tablas en Neon + migraciones (Alembic o Prisma/Drizzle) | M | Esquema |
| F9-02 | API de lectura `GET /zones`, `GET /zones/{slug}?locale=` + CRUD protegido | M | API |
| F9-03 | Dockerfile + `docker-compose` con Postgres local; despliegue en Render/Railway | M | API desplegada |
| F9-04 | Next lee la API en build + revalidación al guardar | M | Contenido desde API |

## Hitos de versión

| Etiqueta | Al cerrar | Contenido |
|---|---|---|
| `v0.1` | F1 | App base con cubo, i18n, CI, previews |
| `v0.2` | F2 | Layout con losas provisionales navegable |
| `v0.5` | F5 | MVP interactivo con hitos MVP |
| `v1.0.0` | F8 | Publicación |
