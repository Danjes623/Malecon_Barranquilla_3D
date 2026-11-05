# Maqueta 3D del Gran Malecón de Barranquilla

Maqueta web interactiva, en estilo low-poly y en miniatura, del **Gran Malecón del Río** de Barranquilla (Colombia). Recorre los 5,5 km del paseo, desde el Centro de Eventos **Puerta de Oro** hasta el **Jardín del Río** en La Loma, en una tira 3D navegable desde el navegador, en computador y en móvil.

Al hacer clic en un monumento o zona, la cámara vuela hasta él y se abre un panel con información en **español e inglés**: descripción, datos clave, horarios, fotos y fuentes.

> **Proyecto independiente y no oficial.** No está afiliado a la Alcaldía de Barranquilla, a Puerta de Oro ni a los operadores del Gran Malecón.

**Estado:** Fase 0 (organización y decisiones). Ver [`Docs/fases-tareas.md`](Docs/fases-tareas.md).

---

## Tabla de contenido

1. [Qué es y qué lo hace distinto](#1-qué-es-y-qué-lo-hace-distinto)
2. [Funcionalidades](#2-funcionalidades)
3. [Hitos representados](#3-hitos-representados)
4. [Equipo](#4-equipo)
5. [Stack tecnológico](#5-stack-tecnológico)
6. [Arquitectura](#6-arquitectura)
7. [Pipeline del modelo 3D](#7-pipeline-del-modelo-3d)
8. [Estructura del repositorio](#8-estructura-del-repositorio)
9. [Puesta en marcha](#9-puesta-en-marcha)
10. [Scripts](#10-scripts)
11. [Contenido y datos](#11-contenido-y-datos)
12. [Fases del proyecto](#12-fases-del-proyecto)
13. [Cómo contribuir](#13-cómo-contribuir)
14. [Despliegue](#14-despliegue)
15. [Documentación](#15-documentación)
16. [Licencias y créditos](#16-licencias-y-créditos)

---

## 1. Qué es y qué lo hace distinto

El proyecto se inspira en maquetas web tipo "ciudad en tu mano", pero con una diferencia clave: el Gran Malecón es **lineal**, no circular. Para que 5,5 km quepan en una pantalla sin perder legibilidad se aplica una **compresión no uniforme sobre la curva real del río Magdalena**:

- La orilla se traza como una curva que conserva el **trazado y la orientación reales**.
- Cada zona es una **"losa"** independiente colocada sobre esa curva, en su orden real de noroeste a sureste.
- Las zonas densas (Puerta de Oro, Caimán del Río y el núcleo Pabellón de Cristal–Luna del Río) mantienen una escala casi uniforme.
- Los tramos de césped y sendero se reducen al 10–20 %, y el río se estrecha 3–5×.
- Los monumentos se **exageran 1,5–3×** para que se lean bien y sean fáciles de tocar en móvil.

El resultado es una maqueta con intención: decide qué tramos importan, en vez de ser un mapa a escala.

## 2. Funcionalidades

- **Maqueta 3D navegable** con cámara limitada: rotar, acercar y desplazarse por la tira sin perderse.
- **Clic o toque en cualquier hito** para volar hasta él y abrir su panel de información.
- **Panel de información**: lateral en escritorio y desde abajo en móvil. Muestra fotos, horario, enlaces y fuentes, y se cierra con Esc.
- **"Recorrer la ruta"**: tour automático por las zonas en orden, con Pausa y Siguiente.
- **Enlaces directos** a cada zona (`?zona=luna-del-rio`), compartibles.
- **Páginas estáticas por zona** (`/es/zona/<slug>`) con vista previa al compartir por WhatsApp o redes.
- **Español / inglés** y **modo oscuro**, que también cambia la iluminación de la escena.
- **Accesibilidad**: lista de monumentos navegable con teclado y lector de pantalla, respeto a `prefers-reduced-motion` y alternativa sin WebGL.
- **Efecto miniatura** (tilt-shift) desactivable, agua estilizada y una barca animada.

## 3. Hitos representados

**MVP:**

| Orden | Hito | Sector |
|---|---|---|
| 1 | Centro de Eventos Puerta de Oro | Puerta de Oro |
| 3 | Caimán del Río | Gastronómico |
| 7 | Monumento El Ancla / Plaza de los Marinos | Recreodeportivo |
| 9 | Plaza de las Luces | Recreodeportivo |
| 10 | Pabellón de Cristal | Recreodeportivo |
| 11 | Luna del Río (rueda de 65 m) | Junto al Pabellón |
| 17 | Estatua de Shakira | Sector Nativo |
| 21 | Ventana de Campeones ("Aleta del Tiburón") | La Loma |
| 22 | Jardín del Río | La Loma |

**Ampliación:** Carrusel del Río, Estatua de Sofía Vergara, Rin Rin Renacuajo, Sector Nativo, Skatepark, puentes a La Loma, murales de la calle 78, Manglares, zonas recreativas y más.

**Fuera de la maqueta** (no están en el Malecón): Ventana al Mundo, estatua de Joe Arroyo, Parque Cultural del Caribe, Gran Malecón del Suroriente y Kristal Mall.

El inventario completo, con fechas y fuentes, está en [`Docs/Maqueta 3D Malecón Barranquilla.md`](Docs/Maqueta%203D%20Malecón%20Barranquilla.md).

## 4. Equipo

| Integrante | Rol | Responsabilidades |
|---|---|---|
| **Daniel Mejía** | Modelado 3D | Referencia OSM en Blender, layout de losas, modelado de hitos y props, paleta y material, horneado de iluminación y pipeline de exportación GLB |
| **Meriyein Manosalva** | Desarrollo web y contenido | Next.js y React Three Fiber, interacción y cámara, interfaz, i18n, textos y fotos, accesibilidad, SEO, pruebas, despliegue y backend opcional |

Todo cambio lo revisa la otra persona antes de entrar a `main`.

## 5. Stack tecnológico

| Capa | Tecnología |
|---|---|
| Modelado | Blender 5.x (Blender 4.5 LTS como respaldo para importar OSM) |
| Referencia geográfica | OpenStreetMap vía BlenderGIS / Blosm |
| Optimización 3D | glTF-Transform (meshopt + texturas WebP) y gltfjsx |
| Framework web | Next.js 16.4 (App Router) + TypeScript |
| 3D en la web | three.js r187, React Three Fiber 9.8, drei 10.7 |
| Estado | Zustand |
| Interfaz | Tailwind CSS + shadcn/ui |
| Idiomas | next-intl v4 |
| Validación de datos | Zod |
| Pruebas | Vitest y Playwright |
| Hosting | Vercel |
| Backend (opcional) | FastAPI o NestJS en Docker + Neon Postgres |

Todo el stack principal es gratuito y de código abierto. El detalle y las razones de cada elección están en [`Docs/design.md`](Docs/design.md).

## 6. Arquitectura

```
[Blender .blend] → export .glb → gltf-transform (meshopt + webp) → gltfjsx --types → Malecon.tsx
                                                                        │
Next.js 16.4 (Vercel) ───────────────────────────────────────────────────┘
 ├─ app/[locale]/page.tsx        → <SceneClient/> ('use client', dynamic ssr:false)
 │     ├─ <Canvas frameloop="demand"> R3F + drei (CameraControls, Html, Instances)
 │     ├─ store Zustand { selected, hovered }  ⇄  URL ?zona=slug
 │     └─ DOM: <nav aria-label="Monumentos">, panel de info, ES/EN, tema
 ├─ app/[locale]/zona/[slug]     → página estática SEO
 └─ contenido: content/zonas/*.json (MVP)  ──o──  lectura en build desde API (Fase 9)
```

Principios:

- La escena 3D se carga **solo en el navegador**. Las páginas de zona son estáticas y no descargan three.js.
- Textos y posiciones de cámara viven en **archivos de datos**, no en el código. Así añadir un hito nuevo es añadir una losa y un JSON.
- El sitio público **no depende de ningún servidor en tiempo de ejecución**.

## 7. Pipeline del modelo 3D

1. **Referencia**: importar orilla, río y vías de OpenStreetMap a Blender. Solo se usa como guía y no se exporta.
2. **Layout**: trazar la curva de la orilla y colocar las losas según la hoja de escalas.
3. **Modelado**: hitos low-poly hechos a mano, más props CC0 (Kenney, Quaternius) recoloreados. Toda la escena usa **un solo material con una textura-paleta**.
4. **Nombres y metadatos**: cada objeto clickeable se llama `mon_*` o `zone_*` y lleva las propiedades `zone_id`, `order` y `clickable`. Estas llegan a la web como `userData`.
5. **Horneado**: la oclusión ambiental se hornea en Blender, sin sombras en tiempo real.
6. **Exportación y optimización**: `npm run model` exporta el GLB, lo comprime con meshopt + WebP sin unir zonas y genera el componente tipado `Malecon.tsx`.

Presupuesto de rendimiento: menos de 150 draw calls, 100–300 k triángulos, GLB de 2–5 MB y DPR máximo 1,5.

## 8. Estructura del repositorio

```
/
├─ Docs/                     # Documentación del proyecto (fuente de verdad)
├─ blender/                  # Archivos .blend, paleta y referencias (Git LFS)
├─ scripts/model.mjs         # Pipeline export → optimize → gltfjsx
├─ public/models/            # malecon.glb optimizado
├─ content/
│  ├─ zonas/*.json           # Una zona por archivo (ES/EN, cámara, fotos, fuentes)
│  └─ geo.json               # Coordenadas de referencia
├─ src/
│  ├─ app/[locale]/          # Rutas de Next.js
│  ├─ components/scene/      # Escena 3D (client-only)
│  ├─ components/ui/         # Interfaz
│  ├─ lib/                   # Esquemas, store, carga de contenido
│  └─ messages/{es,en}.json  # Textos de la interfaz
├─ tests/{unit,e2e}/
├─ CREDITS.md                # Atribuciones de assets y datos
└─ README.md
```

> La estructura de código se crea en la Fase 1. Por ahora el repositorio solo contiene `Docs/`.

## 9. Puesta en marcha

> Disponible a partir de la Fase 1.

**Requisitos**

- Node.js **20.9 o superior**
- Git y **Git LFS** (`git lfs install`)
- Blender 5.x (solo para trabajar el modelo)

**Instalación**

```bash
git clone <url-del-repositorio>
cd Malecon_Barranquilla_3D
git lfs pull
npm ci
npm run dev
```

Abrir `http://localhost:3000/es`.

## 10. Scripts

| Comando | Qué hace |
|---|---|
| `npm run dev` | Servidor de desarrollo |
| `npm run build` | Build de producción |
| `npm run lint` | ESLint |
| `npm run typecheck` | Verificación de tipos |
| `npm run test` | Pruebas unitarias (Vitest) |
| `npm run test:e2e` | Pruebas end-to-end (Playwright) |
| `npm run model` | Regenera `public/models/malecon.glb` y `Malecon.tsx` desde Blender |

## 11. Contenido y datos

Cada zona es un archivo `content/zonas/<slug>.json` validado con Zod. Contiene:

- nombre, resumen y descripción en `es` y `en`
- categoría, sector y orden del recorrido
- el nombre del objeto en el GLB y el preset de cámara
- coordenadas (marcadas como no verificadas hasta contrastarlas)
- horario, fotos con texto alternativo y crédito
- **fuentes** de cada dato publicado

Para añadir o corregir una zona basta editar su JSON y abrir un PR. Las pruebas comprueban que existan ambos idiomas y que el objeto exista en el modelo.

## 12. Fases del proyecto

| Fase | Contenido | Versión |
|---|---|---|
| 0 | Organización, inventario, coordenadas, paleta, licencias | — |
| 1 | Esqueleto técnico: Next.js, escena mínima, i18n, CI, Vercel | `v0.1` |
| 2 | Referencia OSM y layout de losas | `v0.2` |
| 3 | Modelado low-poly de hitos y props, textos y fotos | — |
| 4 | Pipeline de exportación automatizado | — |
| 5 | Interacción 3D: selección, cámara, tour | `v0.5` |
| 6 | Interfaz, accesibilidad y SEO | — |
| 7 | Rendimiento y pruebas | — |
| 8 | Despliegue | `v1.0.0` |
| 9 | Backend opcional (API + base de datos) | — |

Las tareas, responsables y entregables de cada fase están en [`Docs/fases-tareas.md`](Docs/fases-tareas.md).

## 13. Cómo contribuir

1. Tomar un issue del tablero. Cada uno lleva el ID de su tarea (`[F3-04] …`).
2. Crear una rama desde `main`, por ejemplo `model/F3-04-luna-del-rio` o `feat/F5-02-seleccion-click`.
3. Hacer commits con Conventional Commits en español, por ejemplo `feat(escena): sincronizar zona con ?zona=`.
4. Abrir un PR con la plantilla, el enlace al preview de Vercel y capturas.
5. Esperar la revisión de la otra persona y el CI en verde. La fusión se hace con *Squash and merge*.

Antes de empezar, leer [`Docs/reglas.md`](Docs/reglas.md) y [`Docs/prohibiciones.md`](Docs/prohibiciones.md).

## 14. Despliegue

- **Producción** en Vercel, desplegada automáticamente al fusionar en `main`.
- **Preview** automático por cada PR.
- El GLB y las fotos se sirven con caché inmutable y pasan a Cloudflare R2 si crecen.

Detalles en [`Docs/plan-sitio.md`](Docs/plan-sitio.md).

## 15. Documentación

| Documento | Contenido |
|---|---|
| [`Docs/design.md`](Docs/design.md) | Diseño técnico y visual, modelo de datos, convenciones |
| [`Docs/fases-tareas.md`](Docs/fases-tareas.md) | Fases, tareas y responsables |
| [`Docs/reglas.md`](Docs/reglas.md) | Flujo de trabajo, ramas, commits, PRs |
| [`Docs/prohibiciones.md`](Docs/prohibiciones.md) | Lo que no se hace en el proyecto |
| [`Docs/plan-legal.md`](Docs/plan-legal.md) | Licencias, atribuciones, privacidad |
| [`Docs/plan-seguridad.md`](Docs/plan-seguridad.md) | Accesos, secretos, dependencias, cabeceras |
| [`Docs/plan-sitio.md`](Docs/plan-sitio.md) | Despliegue en Vercel |
| [`Docs/Maqueta 3D Malecón Barranquilla.md`](Docs/Maqueta%203D%20Malecón%20Barranquilla.md) | Investigación original y plan base |

## 16. Licencias y créditos

- Licencias propuestas: **MIT** para el código, **CC BY-NC 4.0** para el modelo 3D y **CC BY 4.0** para los textos. La decisión final se toma en la tarea F0-09 y queda en `LICENSE`.
- Datos geográficos de referencia: **© OpenStreetMap contributors**, bajo licencia ODbL ([openstreetmap.org/copyright](https://www.openstreetmap.org/copyright)).
- Assets de terceros (Kenney, Quaternius y otros): listados en `CREDITS.md`.
- Las esculturas se representan como siluetas simplificadas, citando a sus autores.

Más detalles en [`Docs/plan-legal.md`](Docs/plan-legal.md).
