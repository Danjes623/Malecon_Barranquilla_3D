# Diseño — Maqueta 3D del Gran Malecón de Barranquilla

> Documento de referencia técnica y visual. Fuente: `Docs/Maqueta 3D Malecón Barranquilla.md`.
> Cualquier cambio de stack, estructura o modelo de datos se propone por PR que modifique este archivo.

## 1. Objetivo

Maqueta 3D web, low-poly y clickeable del Gran Malecón del Río (5,5 km, de Puerta de Oro a Jardín del Río / La Loma). Cada hito se puede seleccionar para ver un panel de información en español e inglés. La forma real lineal del río se conserva, pero los tramos vacíos se comprimen.

## 2. Equipo

| Persona | Rol principal | Áreas |
|---|---|---|
| **Daniel Mejía** | Modelado 3D | Blender, referencia OSM, layout de losas, hitos, props, materiales, horneado, pipeline de exportación GLB |
| **Meriyein Manosalva (Mery)** | Desarrollo web y contenido | Next.js, R3F, interacción, UI, i18n, datos, accesibilidad, SEO, pruebas, despliegue, backend opcional |

Ambos revisan el trabajo del otro (ver `reglas.md`).

## 3. Stack

| Capa | Tecnología | Versión fijada |
|---|---|---|
| Modelado | Blender 5.x (+ Blender 4.5 LTS para importar OSM si falla BlenderGIS) | — |
| Referencia geográfica | OpenStreetMap vía BlenderGIS o Blosm | — |
| Optimización | `@gltf-transform/cli` (meshopt + webp) | 4.x |
| Componente 3D | `gltfjsx --types --keepnames --keepgroups --instance` | — |
| Framework | Next.js App Router + TypeScript | 16.4 |
| React | React | 19.3 (verificar en Fase 1) |
| 3D | `three` / `@react-three/fiber` / `@react-three/drei` | 0.187 / 9.8 / 10.7 |
| Estado | Zustand | — |
| UI | Tailwind + shadcn/ui | — |
| i18n | next-intl | v4 |
| Validación | Zod | — |
| Pruebas | Vitest (unidad), Playwright (E2E) | — |
| Despliegue | Vercel Hobby (ver `plan-sitio.md`) | — |
| Backend opcional | FastAPI o NestJS en Docker + Neon Postgres | Fase 9 |

**No usar:** R3F 10 (alfa), WebGPU, Theatre.js, Spline (ver `prohibiciones.md`).

## 4. Arquitectura

```
[Blender .blend] → export .glb → gltf-transform (meshopt + webp) → gltfjsx --types → Malecon.tsx
                                                                        │
Next.js 16.4 (Vercel) ───────────────────────────────────────────────────┘
 ├─ app/[locale]/page.tsx        → <SceneClient/> ('use client', dynamic ssr:false)
 │     ├─ <Canvas frameloop="demand"> R3F + drei (CameraControls, Html, Instances)
 │     ├─ store Zustand { selected, hovered }  ⇄  URL ?zona=slug
 │     └─ DOM: <nav aria-label="Monumentos">, Sheet/Drawer de info, ES/EN, tema
 ├─ app/[locale]/zona/[slug]     → página estática SEO (generateStaticParams/Metadata)
 └─ contenido: content/zonas/*.json (MVP)  ──o──  build-time fetch a API (Fase 9)
GLB y fotos: /public (si es pequeño) o Cloudflare R2 con caché inmutable
```

Principios:

1. El canvas 3D solo se carga en cliente (`next/dynamic` con `ssr:false`).
2. Las páginas de zona son estáticas y no cargan three.js.
3. Las páginas públicas nunca dependen de un backend en tiempo de ejecución.
4. Zonas y presets de cámara viven en **datos**, no en código.

## 5. Estructura del repositorio

```
/
├─ Docs/                     # Estos documentos
├─ blender/                  # .blend fuente (Git LFS), paleta, referencias propias
│  ├─ malecon.blend
│  ├─ palette.png
│  └─ refs/                  # Fotos propias (sin personas identificables)
├─ scripts/model.mjs         # Pipeline export → optimize → gltfjsx (npm run model)
├─ public/models/            # malecon.glb optimizado
├─ content/
│  ├─ zonas/*.json           # Una zona por archivo
│  └─ geo.json               # Coordenadas ancla (verified:false hasta confirmar)
├─ src/
│  ├─ app/[locale]/          # Rutas
│  ├─ components/scene/      # Canvas, Malecon.tsx generado, controles
│  ├─ components/ui/         # shadcn/ui, panel, nav
│  ├─ lib/                   # schema Zod, carga de contenido, store
│  └─ messages/{es,en}.json  # Textos de interfaz
├─ tests/{unit,e2e}/
├─ CREDITS.md                # Atribuciones de assets y datos
└─ README.md
```

## 6. Modelo de datos

```ts
// src/lib/schema.ts
const Localized = z.object({ es: z.string().min(1), en: z.string().min(1) });

export const Zone = z.object({
  slug: z.string().regex(/^[a-z0-9-]+$/),        // estable, nunca se renombra
  name: Localized,
  summary: Localized,
  description: Localized,
  category: z.enum(["monumento", "gastronomia", "recreacion", "naturaleza", "infraestructura"]),
  order: z.number().int(),                         // orden del recorrido NO→SE
  sector: z.string(),                              // por nombre, nunca por número de UF
  scene: z.object({
    meshName: z.string(),                          // debe existir en el GLB
    hotspot: z.tuple([z.number(), z.number(), z.number()]),
    camera: z.object({
      position: z.tuple([z.number(), z.number(), z.number()]),
      target: z.tuple([z.number(), z.number(), z.number()]),
    }),
  }),
  geo: z.object({ lat: z.number(), lng: z.number(), verified: z.boolean() }).optional(),
  hours: Localized.optional(),
  images: z.array(z.object({ src: z.string(), alt: Localized, credit: z.string() })),
  sources: z.array(z.string().url()),              // de dónde sale cada dato del panel
  published: z.boolean(),
});
```

Backend opcional (Fase 9): tablas `zones`, `zone_translations (zone_id, locale)`, `zone_images`, o JSONB `{"es":…, "en":…}`.

## 7. Diseño visual

### 7.1 Estilo

- Low-poly, efecto miniatura: cámara de lente larga u ortográfica, ángulo alto, `TiltShift2` desactivable.
- **Un solo material** con textura-paleta pequeña (método PixPal), interpolación "Closest".
- Paleta de 16–32 colores definida en Fase 0. Decisión pendiente: río ocre realista o azul estilizado.
- Iluminación y AO horneados en Blender. Sin sombras en tiempo real.
- Modo oscuro: cambia fondo, iluminación de escena y emisivos (Plaza de las Luces).

### 7.2 Layout lineal comprimido

- La orilla del Magdalena se traza como curva en Blender y conserva trazado y orientación reales.
- Cada zona es una "losa" independiente (base con borde de tierra cortada + props) colocada sobre la curva.
- Zonas densas (Puerta de Oro, Caimán del Río, núcleo Pabellón/Luna/Carrusel/Sofía Vergara): escala casi uniforme.
- Tramos de césped y sendero: 10–20 % de su longitud real.
- Río: comprimido 3–5× en ancho.
- Hitos: exagerados 1,5–3× para leerse y tocarse en móvil.
- Escala total orientativa: 5,5 km → 300–600 unidades Blender (hoja de escalas en Fase 2).

### 7.3 Inventario de hitos

**MVP (obligatorios):** 1 Puerta de Oro, 3 Caimán del Río, 7 El Ancla, 9 Plaza de las Luces, 10 Pabellón de Cristal, 11 Luna del Río, 17 Estatua de Shakira, 21 Ventana de Campeones, 22 Jardín del Río.

**Ampliación:** 12 Carrusel del Río, 13 Estatua de Sofía Vergara, 15 Rin Rin Renacuajo, 16 Sector Nativo, 18 Skatepark, 20 Puentes, y el resto de la tabla del documento fuente.

**Fuera de la maqueta:** Ventana al Mundo, estatua de Joe Arroyo, Parque Cultural del Caribe, Gran Malecón del Suroriente, Kristal Mall, ampliación calle 84. Las macrofiguras de Carnaval solo como capa estacional opcional.

### 7.4 Convenciones de nombres en Blender

| Prefijo | Uso | Ejemplo |
|---|---|---|
| `zone_` | Losa / zona completa | `zone_caiman_del_rio` |
| `mon_` | Monumento clickeable | `mon_luna_del_rio` |
| `*_hit` | Hitbox invisible (mesh real transparente) | `mon_luna_del_rio_hit` |
| `prop_` | Prop instanciado no clickeable | `prop_palmera` |
| `ref_` | Referencia OSM (no se exporta) | `ref_orilla` |

- Solo ASCII, minúsculas, `snake_case`.
- Custom Properties obligatorias en clickeables: `zone_id` (= `slug` con `_`), `order`, `clickable`.
- El `meshName` de cada JSON debe coincidir con el nombre en el GLB. Un test lo verifica.

## 8. Presupuesto de rendimiento (heurístico)

| Métrica | Límite |
|---|---|
| Draw calls | < 100–150 |
| Triángulos | 100–300 k |
| GLB comprimido | 2–5 MB |
| DPR máximo | 1,5 |
| `frameloop` | `"demand"` |

## 9. Accesibilidad

- `<nav aria-label="Monumentos">` como lista DOM que llama al mismo `focusLandmark(id)` que el clic 3D.
- Panel cerrable con Esc, foco gestionado.
- `prefers-reduced-motion`: cortes instantáneos, sin tour automático.
- Fallback sin WebGL: imagen pre-renderizada + lista.

## 10. Incertidumbres abiertas

Ver la sección "Lo que sigue sin verificar" del documento fuente. Cada una se cierra con una tarea en `fases-tareas.md` o se acepta explícitamente en un PR.
