# Construye el Malecón en miniatura con React Three Fiber

**Recomendación directa:** construye la maqueta con **Blender (modelado low-poly) → glTF optimizado con gltf-transform → React Three Fiber 9 + drei 10 sobre Three.js r187, dentro de Next.js 16.4 con TypeScript**. Los clics se resuelven con eventos por mesh, la cámara con `CameraControls` y la ventana de información con un panel DOM (shadcn/ui) conectado por un store de Zustand. Todo es MIT y gratuito, y R3F te ahorra casi todo el código imperativo de Three.js, algo que importa si tu fuerte es el backend. Para conservar la forma lineal del Gran Malecón sin la metáfora circular de Zaragoza, la técnica es colocar las zonas sobre una curva que sigue la orilla real del Magdalena y comprimir de forma no uniforme los tramos vacíos. Así 5,5 km reales caben en una tira legible donde cada hito sigue en su orden. El backend (FastAPI o NestJS sobre Neon, en Docker) es opcional: para el MVP basta un JSON tipado en el repositorio, y la API se añade después si el curso pide mostrar backend. Hay tres incertidumbres que conviene tener presentes desde el inicio. No se sabe con qué tecnología se hizo "Zaragoza cabe en tu mano". Falta confirmar la compatibilidad exacta de R3F con React 19.3. Y no hay coordenadas verificadas de ningún punto del Malecón, así que hay que tomarlas a mano.

## Nadie ha publicado cómo se hizo Zaragoza; R3F es la apuesta segura

No hay fuente pública que diga con qué se construyó "Zaragoza cabe en tu mano". No aparece URL en vivo, ni texto del post de LinkedIn (LinkedIn bloquea la lectura automatizada), ni repositorio, ni entrevista. El portafolio de Javier Balvín Lau se presenta como "Diseñador freelance desde Zaragoza", y su sección de modelado 3D solo lista dos trabajos antiguos que no son web: una parada de tranvía modelada en **Autodesk Inventor** (hacia 2015) y unas infografías de edificio ([javierbalvin.com](https://www.javierbalvin.com/diseno-grafico-zaragoza/ux-portfolio-cat/modelado-3d/); [Parada Tranvía](https://www.javierbalvin.com/diseno-grafico-zaragoza/ux-portfolio/3d-parada-tranvia-de-zaragoza/)). Lo que se ve del proyecto (globo low-poly, monumentos clickeables con etiqueta, recorrido de cámara, selector de idioma, modo oscuro) encaja con Three.js/R3F más etiquetas HTML, y también con Spline. Es una deducción por patrón, no un dato. **No afirmes un stack para Zaragoza.** Si consigues la URL, ábrela con DevTools y busca estas señales: `window.__THREE__` indica Three.js, peticiones a `prod.spline.design` o archivos `.splinecode` indican Spline, un script `babylon` indica Babylon.js, y archivos `.glb` con `EXT_meshopt_compression` o Draco indican un pipeline Blender→glTF.

Para tu stack (TypeScript, Next.js, Vite) la mejor opción es **React Three Fiber 9.8.2 + drei 10.7.9 sobre three 0.187.0**, versiones verificadas en npm el 2026-11-05 ([npm @react-three/fiber](https://www.npmjs.com/package/@react-three/fiber); [npm three](https://registry.npmjs.org/three)). drei trae lo que una maqueta clickeable necesita: `<Html>` para etiquetas que se ocultan detrás de la geometría con `occlude`, `<CameraControls>` con `fitToBox`, `setLookAt` y `lerpLookAt` para volar hasta un monumento, `<Bounds>` para encuadrar, `<Instances>` y `<Merged>` para palmeras y bancas, y `PerformanceMonitor` para bajar resolución en móviles débiles ([drei docs](https://drei.docs.pmnd.rs/getting-started/introduction); [camera-controls](https://github.com/yomotsu/camera-controls)).

| Opción | Cuándo elegirla | Por qué no es la principal aquí |
|---|---|---|
| **R3F + drei (recomendada)** | Next/Vite + TS, control total, licencia MIT | Hay que aprender algo de 3D |
| Three.js puro + camera-controls | Si prefieres una app Vite sin React | Más código imperativo |
| Babylon.js 9 (Apache-2.0) | Motor "con todo incluido" | Integración con React con menos comunidad |
| PlayCanvas 2.23 | Editor visual | Runtime de ≈655 KB gzip (medido) |
| Spline | Prototipo rápido de diseño | Exportar código exige el plan **Pro, $25–30/usuario/mes**; runtime de ≈0,5–0,9 MB gzip ([spline.design/pricing](https://spline.design/pricing)) |
| Threlte 8 | Si el equipo usa Svelte | Fuera de tu stack React/Next |

Hay un punto de compatibilidad que **debes verificar antes de fijar versiones**. Next.js 16.4 (2026-10-06) trae **React 19.3** ([Next.js 16.4](https://nextjs.org/blog/next-16-4)). Las notas de R3F 9.5.0 decían compatibilidad "entre React 19.0 y 19.2" ([R3F v9.5.0](https://github.com/pmndrs/react-three-fiber/releases/tag/v9.5.0)), pero la versión actual 9.8.2 declara en npm `react >=19 <19.4`, lo que incluye 19.3 ([npm](https://www.npmjs.com/package/@react-three/fiber)). Instala y corre una escena mínima antes de construir encima. **No uses R3F 10** (sigue en alfa, 10.0.0-alpha.6) ni WebGPU: una maqueta low-poly no está limitada por la GPU, y el `WebGLRenderer` por defecto de R3F 9 es lo más estable ([Utsubo](https://www.utsubo.com/blog/threejs-2026-what-changed)). Para el "Recorrer la ruta", encadena `await controls.setLookAt(..., true)` por parada y abre el panel en el evento `rest`, que se dispara cuando la cámara se detiene. GSAP 3.15.1 es ahora **gratuito incluso para uso comercial** si quieres tweens más finos ([Webflow](https://webflow.com/blog/gsap-becomes-free)). Theatre.js no publica versión desde mayo de 2024, así que evítalo ([npm](https://registry.npmjs.org/@theatre/core)).

## Next.js 16.4 con canvas client-only y contenido estático primero

El frontend debe ser **Next.js 16.4 App Router + TypeScript**. El canvas 3D se carga solo en cliente: un componente pequeño con `'use client'` que importa la escena con `next/dynamic` y `ssr:false`. Sin eso aparece "window is not defined", porque three.js accede a `window` ([tutorial Noqta](https://www.noqta.tn/en/tutorials/react-three-fiber-nextjs-3d-interactive-web-2026)). Cada zona tiene además su propia página estática `/[locale]/zona/[slug]` con `generateStaticParams` y `generateMetadata`. Así, cuando alguien comparte por WhatsApp el enlace del Carrusel del Río, aparece una vista previa con título e imagen, cosa que una SPA de Vite no da. Next 16.4 añade `export const ensureStatic = 'navigation'`, que hace fallar el build si se cuela contenido dinámico, y un Bundle Analyzer para comprobar que three.js no se cargue en las páginas que no lo usan ([Next.js 16.4](https://nextjs.org/blog/next-16-4)). Next 16 exige Node 20.9+, reemplaza `middleware.ts` por `proxy.ts` y obliga a usar `await` con `params` ([Next.js 16](https://nextjs.org/blog/next-16)). Para ES/EN usa **next-intl v4**, con `proxy.ts` y rutas bajo `app/[locale]/` ([next-intl](https://next-intl.dev/docs/getting-started/app-router/with-i18n-routing)). **Vite** encaja en dos sitios: como *sandbox* para probar la escena R3F aislada en las primeras fases, y como alternativa completa (Vite + React Router) si el SEO no cuenta en la evaluación.

La capa de datos se escala por fases. El **MVP** usa `content/zonas/*.json` validado con Zod, con campos `es`/`en`: sin base de datos, sin arranques en frío y todo pre-renderizado. Si después necesitas administración, tienes dos caminos. El primero es **Payload CMS 3 dentro de la misma app Next sobre Neon**, con localización por campo (`localized: true`) y admin en `/admin` ([Payload localization](https://payloadcms.com/docs/v3/configuration/localization.md); [Neon + Payload](https://neon.com/guides/payload)). El segundo, que aprovecha mejor tu perfil backend, es una **API FastAPI (o NestJS) en Docker sobre Neon**. Con el segundo, el frontend debe leer la API **en tiempo de build**, para que las páginas públicas nunca dependan de un servidor dormido: Render Free apaga el servicio tras **15 minutos sin tráfico**, tarda ~1 minuto en despertar, y sus Postgres gratuitos **caducan a los 30 días** ([Render Free](https://render.com/docs/free)). Usa Neon para la base. El plan Free da 1 GB por proyecto, 100 CU-hora/mes y escala a cero tras 5 minutos, más que suficiente para 20–30 zonas de texto ([Neon Plans](https://neon.com/docs/introduction/plans)). Las fotos no van en Postgres sino en **Cloudflare R2** (10 GB gratis, sin cobro de egreso) ([R2 pricing](https://developers.cloudflare.com:2053/r2/pricing)).

El despliegue va en **Vercel Hobby**: gratis, uso personal no comercial, **100 GB/mes de transferencia** ([Vercel Hobby](https://vercel.com/docs/plans/hobby)). Con un GLB de 3 MB eso da para unas 33.000 cargas completas al mes, sin contar la caché del navegador. Si el proyecto pasa a uso comercial (por ejemplo, un encargo de turismo), Hobby deja de servir.

Arquitectura propuesta:

```
[Blender .blend] → export .glb → gltf-transform (meshopt + webp) → gltfjsx --types → Malecon.tsx
                                                                        │
Next.js 16.4 (Vercel) ───────────────────────────────────────────────────┘
 ├─ app/[locale]/page.tsx        → <SceneClient/> ('use client', dynamic ssr:false)
 │     ├─ <Canvas frameloop="demand"> R3F + drei (CameraControls, Html, Instances)
 │     ├─ store Zustand { selected, hovered }  ⇄  URL ?zona=slug
 │     └─ DOM: <nav aria-label="Monumentos">, Sheet/Drawer de info (shadcn/ui), ES/EN, tema
 ├─ app/[locale]/zona/[slug]     → página estática SEO (generateStaticParams/Metadata)
 └─ contenido: content/zonas/*.json (MVP)  ──o──  build-time fetch a API
                                                   FastAPI|NestJS (Docker) → Neon Postgres
GLB y fotos: /public (si es pequeño) o Cloudflare R2 con caché inmutable
```

El modelo de datos de una zona tiene: `slug` estable, `name/summary/description` localizados, `category`, `order` (orden del recorrido) y `scene` con `meshName`, `hotspot` y `camera {position, target}`. Además lleva `geo {lat,lng}` opcional, `hours`, `images[]` con `alt` localizado y `credit`, y `published`. En Postgres son las tablas `zones`, `zone_translations (zone_id, locale)` y `zone_images`, o JSONB `{"es":…, "en":…}` si prefieres simplicidad. Guarda los presets de cámara **en los datos, no en el código**, para poder ajustarlos sin redeploy.

## El flujo de modelado: OSM como referencia, Blender a mano, una sola paleta

La base geométrica legal es **OpenStreetMap**, importado a Blender con **BlenderGIS** (gratis, GPL, ya parcheado para Blender 5) o **Blosm** (base gratis; Pro €17,80) ([BlenderGIS releases](https://github.com/domlysz/blendergis/releases); [Blosm](https://github.com/vvoovv/blosm)). Usa esa importación solo como **capa de referencia** (línea de orilla, Vía 40/Av. del Río, río) y remodela encima en low-poly. **No uses Google Photorealistic 3D Tiles ni siquiera para calcar**: sus políticas prohíben objetos 3D "extracted, traced, or otherwise derived by hand or machine" de los tiles, y también el uso offline ([Google Map Tiles Policies](https://developers.google.com/maps/documentation/tile/policies?authuser=1)). En una esquina del visor o en un panel "(i)/Créditos" pon "© OpenStreetMap contributors" con enlace a openstreetmap.org/copyright ([OSMF Attribution](https://osmfoundation.org/wiki/Attribution)). No publiques la geometría ni las etiquetas OSM crudas dentro del GLB.

La forma lineal comprimida se logra con **técnica de practicante, sin un caso publicado que la respalde**. Primero trazas la orilla como curva en Blender. Después construyes cada zona como una "losa" independiente (base con borde de tierra cortada, más sus props) y las colocas a lo largo de la curva. La curva conserva el trazado y la orientación reales; lo que se comprime es la distancia entre losas. Las zonas densas (Puerta de Oro, Caimán del Río, el núcleo del Pabellón de Cristal) quedan a escala casi uniforme, y los tramos de césped y sendero se reducen al 10–20 %. El río se comprime 3–5× en ancho y los monumentos se exageran 1,5–3× para que se lean y sean fáciles de tocar en móvil. El contraste de densidad es real: El Heraldo cuenta **15 "estaciones" en solo 1,7 km**, entre el Pabellón de Cristal y los food trucks ([El Heraldo, 2022](https://www.elheraldo.co/sociedad/15-estaciones-imperdibles-en-su-visita-al-gran-malecon-del-rio-878515)). Remata el efecto miniatura con cámara de lente larga u ortográfica, ángulo alto y `TiltShift2` de @react-three/postprocessing ([TiltShift2](https://react-postprocessing.docs.pmnd.rs/effects/tilt-shift-2)), que debe poder apagarse en móviles.

El estilo low-poly se apoya en **un solo material con una textura-paleta pequeña** (método PixPal de Imphenzia). Las UV de cada cara se escalan sobre un cuadro de color, de modo que toda la escena comparte una textura de pocos KB y una paleta cambiable de un golpe ([Wintermute Digital](https://wintermutedigital.netlify.app/post/lowpoly-colour-palettes)). Para vegetación y mobiliario genérico usa **Kenney (CC0)** y **Quaternius (CC0)**, recoloreados con tu paleta ([Kenney](https://kenney.nl/support); [Quaternius](https://quaternius.itch.io/150-lowpoly-nature-models)). Poly Pizza mezcla CC0 y CC-BY, así que registra autor y licencia por modelo. Los hitos únicos (Luna del Río, Ventana de Campeones, estatuas, El Ancla, Pabellón) los modelas tú, porque no existen como asset gratuito. Una decisión de diseño a tomar: el Magdalena es **marrón/ocre**, no azul. Elige con intención entre realismo y legibilidad.

Para que el clic funcione, cada zona o monumento es un objeto (o un grupo bajo un *empty*) con nombre ASCII estable (`mon_luna_del_rio`, `zone_caiman_del_rio`) y Custom Properties (`zone_id`, `order`, `clickable`). Esas propiedades salen como `extras` en glTF y llegan a three.js como `userData`. Activa "Custom Properties" en el exportador (verifica las etiquetas exactas en la UI de Blender 5.2, que no se pudieron confirmar). La optimización es el paso donde más fácil se pierde la separación por zonas. `gltf-transform optimize` usa **meshopt por defecto desde v4.0** ([CHANGELOG](https://www.github.com/donmccurdy/glTF-Transform/blob/main/CHANGELOG.md)) y puede unir y aplanar meshes, así que pasa `--join false --flatten false`, o une las piezas de cada zona en Blender (Ctrl+J) para tener una draw call por zona que siga siendo clickeable. Luego `npx gltfjsx out.glb --types --keepnames --keepgroups --instance` genera un componente R3F tipado ([gltfjsx](https://github.com/pmndrs/gltfjsx)). No apliques `gltfjsx --transform` sobre un archivo ya optimizado. Cada hito lleva además un *hitbox* invisible simple para que el toque en móvil sea tolerante. Ojo: las geometrías invisibles o de tamaño cero no registran eventos, así que usa un mesh real transparente ([Noqta](https://www.noqta.tn/en/tutorials/react-three-fiber-nextjs-3d-interactive-web-2026)).

En rendimiento, la documentación de R3F recomienda **"unos pocos cientos" de draw calls como máximo óptimo** ([R3F scaling-performance](https://github.com/pmndrs/react-three-fiber/blob/master/docs/advanced/scaling-performance.mdx)). Usa `frameloop="demand"` con `invalidate()` en el evento `change` de los controles, iluminación y AO horneados en Blender en vez de sombras en tiempo real, e instancias para palmeras, faroles y bancas. Como presupuesto heurístico (no medido en dispositivos reales): menos de 100–150 draw calls, 100–300 k triángulos, un GLB de 2–5 MB comprimido y DPR máximo 1,5.

## Diecisiete hitos en orden, de Puerta de Oro a La Loma

El Gran Malecón mide **5,5 km** desde el sector Puerta de Oro hasta Jardín del Río (La Loma). Abrió su primer tramo el **17 de julio de 2017**, y en septiembre de 2026 seguía en 5,5 km con cinco sectores ([El Heraldo, 2024](https://www.elheraldo.co/barranquilla/2024/07/16/este-miercoles-se-celebran-7-anos-del-gran-malecon-en-barranquilla-1110893/); [El Tiempo, sep 2026](https://www.eltiempo.com/colombia/barranquilla/barranquilla-megaproyecto-de-mas-de-1-billon-llegara-al-gran-malecon-tendra-centro-comercial-hotel-oficinas-y-una-laguna-3590256)). Las fuentes numeran las "unidades funcionales" de forma contradictoria, así que **etiqueta las zonas por nombre de sector, no por número de UF**. Una segmentación útil para las losas, de noroeste a sureste, es: Puerta de Oro (~0,3 km), Gastronómico/Caimán (~0,5 km), Recreodeportivo calle 78–72 (~1,5 km), Sector Nativo calle 72–caño (~1 km) y puentes + La Loma (~2 km, este último tramo deducido por resta, sin fuente). El grupo formado por el Pabellón de Cristal, la Luna del Río, el Carrusel y la estatua de Sofía Vergara es el nuevo centro del Malecón en 2025–2026 y merece la escala más generosa.

| # | Zona / hito | Sector | Dato clave para el panel |
|---|---|---|---|
| 1 | **Centro de Eventos Puerta de Oro** | Puerta de Oro | Ancla noroeste, Vía 40 con 79B ([El Tiempo 2023](https://www.eltiempo.com/colombia/barranquilla/barranquilla-avanza-la-tercera-fase-del-gran-malecon-738211)) |
| 2 | Muelle Puerta de Oro / RioBús Karakalí, food trucks | Puerta de Oro | Av. del Río con calle 78; paseos en barco, bote La Mita ([Colombia.travel](https://colombia.travel/es/encanto/gran-malecon-de-barranquilla-arte-gastronomia-naturaleza-y-diversion)) |
| 3 | **Caimán del Río** (+ Flor energética, Zona informativa, Teatrino) | Gastronómico | Mercado gastronómico abierto el 23 oct 2019, 25–27 cocinas ([El Heraldo 2022](https://www.elheraldo.co/sociedad/15-estaciones-imperdibles-en-su-visita-al-gran-malecon-del-rio-878515)) |
| 4 | Murales de la calle 78 | Gastronómico | Más de 10 murales del festival Killart ([Colombia.travel](https://colombia.travel/es/encanto/gran-malecon-de-barranquilla-arte-gastronomia-naturaleza-y-diversion)) |
| 5 | Manglares del Río | Recreodeportivo | Plazoleta gastronómica de 8 negocios con temática de manglar |
| 6 | Escudo 3D de Barranquilla | Recreodeportivo | Posición exacta dentro del tramo 78–72 sin confirmar ([Wikipedia ES](https://es.wikipedia.org/wiki/Gran_Malec%C3%B3n)) |
| 7 | **Monumento El Ancla / Plaza de los Marinos** | Recreodeportivo | Nov 2019, anclas del ARC Buenaventura |
| 8 | Zona BBQ, parque de mascotas, picnic, canchas, gimnasios | Recreodeportivo | Instanciables; BBQ y canchas se reservan gratis ([El Heraldo 2022](https://www.elheraldo.co/sociedad/15-estaciones-imperdibles-en-su-visita-al-gran-malecon-del-rio-878515)) |
| 9 | **Plaza de las Luces** | Recreodeportivo | Show de luz y música desde ~7:30 p. m. (buen candidato a emisivo nocturno) |
| 10 | **Pabellón de Cristal** | Recreodeportivo | Pabellón de eventos; plazoleta usada por BMX y skaters |
| 11 | **Luna del Río** | junto al Pabellón | Rueda de **65 m**, 44 cabinas, Fabbri Group, inaugurada el 13 dic 2025, 10 a. m.–10 p. m. ([Infobae](https://www.infobae.com/colombia/2025/12/14/la-luna-del-rio-inicia-operaciones-en-el-malecon-de-barranquilla-asi-fue-su-inauguracion/)) |
| 12 | **Carrusel del Río** | frente a la Luna | Abierto el 20 jun 2026, dos pisos, 15,2 m, 68 pasajeros ([El Heraldo, jun 2026](https://www.elheraldo.co/atlantico/2026/06/20/carrusel-del-rio-encanta-a-propios-y-visitantes-en-su-inauguracion/)) |
| 13 | **Estatua de Sofía Vergara** | cerca del Pabellón | Bronce, **7,6 m**, Yino Márquez, 10 jul 2025 ([El Heraldo, jul 2025](https://www.elheraldo.co/sociedad/2025/07/11/sofia-vergara-toca-las-estrellas-desde-el-gran-malecon-con-su-estatua/)) |
| 14 | La Madriguera del Río | antes de calle 72 | Zona gastronómica inspirada en la fauna del río (orden respecto al Pabellón sin confirmar) |
| 15 | Calle 72: canchas + **9 esculturas de Rin Rin Renacuajo** | Calle 72 | Esculturas develadas el 2 nov 2024 ([El Heraldo, nov 2024](https://www.elheraldo.co/atlantico/2024/11/02/rinrin-renacuajo-se-toma-el-gran-malecon-con-nueve-esculturas/)) |
| 16 | **Sector Nativo** (Jardín Nativo) | calle 72–caño Las Compañías | Abierto el 19 may 2024; juegos infantiles en materiales naturales ([El Heraldo, may 2024](https://www.elheraldo.co/barranquilla/sector-nativo-la-nueva-zona-recreativa-del-gran-malecon-que-inauguro-el-alcalde-char)) |
| 17 | **Estatua de Shakira** | Sector Nativo | Bronce, **6,5 m**, Yino Márquez, 26 dic 2023 ([Blu Radio](https://www.bluradio.com/regiones/caribe/escultura-en-homenaje-a-shakira-fue-develada-en-el-malecon-del-rio-de-barranquilla-rg10)) |
| 18 | **Skatepark** | junto a Shakira, antes del puente | ~2.400 m², 6 zonas, inaugurado ~4 nov 2025 ([Caracol](https://caracolgirardot.com/2025/11/con-una-inversion-de-mas-de-4-000-millones-el-nuevo-skatepark-del-malecon-ya-es-realidad/)) |
| 19 | A Bocas del Río | Sector Nativo | 10 puntos de comida |
| 20 | Puente abatible (y puente fijo) a La Loma | Enlace | Abatible instalado en ago 2019; fijo en sep 2021 ([El Heraldo 2019](https://www.elheraldo.co/barranquilla/2019/08/27/puente-abatible-hacia-la-loma-ya-esta-instalado-660801/)) |
| 21 | **Ventana de Campeones = "Aleta del Tiburón"** | La Loma | Homenaje al Junior, **33 m × 25 m**, ~600 paneles de vidrio, 13 oct 2021; es **un solo monumento** ([El Universal](https://www.eluniversal.com.co/regional/junior-ya-tiene-su-aleta-del-tiburon-en-un-sitio-especial-de-barranquilla-ID5555813)) |
| 22 | **Jardín del Río** | La Loma (fin sureste) | Jardín gastronómico frente al río, muelle turístico, 4 restaurantes |

Los marcados en negrita son los hitos "héroe" que conviene modelar a mano y exagerar. El resto pueden ser props instanciados o losas genéricas. Si necesitas recortar alcance, un MVP con 1, 3, 7, 9, 10, 11, 17, 21 y 22 ya cuenta la historia completa.

**No están en el Malecón y no deben ir en la maqueta**: la **Ventana al Mundo** (47 m, en la glorieta de Circunvalar con Vía 40, al norte de la ciudad) ([Wikipedia EN](https://en.wikipedia.org/wiki/Ventana_al_Mundo,_Barranquilla)); la **estatua de Joe Arroyo**, que está en el Parque de los Músicos ([El Tiempo, may 2025](https://www.eltiempo.com/colombia/barranquilla/barranquilla-se-alista-para-homenajear-a-sofia-vergara-con-una-estatua-en-el-gran-malecon-esto-se-sabe-3456043)); el **Parque Cultural del Caribe**, en el centro (dato no verificado en esta investigación); y el **Gran Malecón del Suroriente**, un proyecto distinto sobre el arroyo Rebolo, de la calle 30 al caño de la Auyama ([El Heraldo, jul 2024](https://www.elheraldo.co/barranquilla/instalan-las-primeras-vigas-en-el-gran-malecon-del-suroriente-1112283)). Tampoco hay fuente que ubique allí el monumento a Esthercita Forero. "Malecón Leandro Díaz", "Leonor Lagos", "El Pescador" o un "Mirador" con nombre propio **no aparecieron en ninguna fuente**. Las macrofiguras de Carnaval son temporales, así que, si las incluyes, que sea como capa estacional opcional. La ampliación de la calle 84 y el proyecto **Kristal Mall** (anunciado en sep 2026, sin fecha de inicio ni ubicación exacta) quedan fuera del paseo ([Diario La Libertad](https://diariolalibertad.com/2026/04/28/avanza-la-prolongacion-de-la-calle-84-hacia-el-gran-malecon-con-tres-frentes-de-obra-en-barranquilla/)).

**Coordenadas: no hay ninguna verificada para puntos del Malecón.** Nominatim/OSM quedó bloqueado durante la investigación. Tómalas a mano en openstreetmap.org (clic derecho → "Mostrar dirección") al menos para Puerta de Oro, Caimán del Río, Pabellón/Luna del Río, calle 72, estatua de Shakira, puente abatible, Ventana de Campeones y Jardín del Río, y márcalas como estimadas hasta contrastarlas.

## Plan por fases, tarea por tarea

La estimación heurística para una persona sola es de **~60–125 horas** (3–6 semanas a tiempo parcial). No hay fuente publicada para este dato. Los mayores riesgos son inflar el detalle de los hitos, que BlenderGIS o Blosm fallen en Blender 5.x (ten Blender 4.5 LTS a mano para importar) y perder la separación de zonas en el optimizador.

**Fase 0 — Decisiones y referencia (≈1 semana)**
1. Pide la URL o el post de LinkedIn de "Zaragoza cabe en tu mano" e inspecciónalo con DevTools para confirmar o descartar su stack.
2. Congela el inventario: elige los 9–12 hitos héroe de la tabla y descarta los que no están en el Malecón.
3. Toma a mano las coordenadas de los 8 puntos ancla en openstreetmap.org y guárdalas en `content/geo.json` con la marca `verified:false`.
4. Reúne referencias visuales propias (fotos en sitio) y de Mapillary/KartaView. Nada de Google 3D Tiles.
5. Define la paleta (16–32 colores) y decide si el río será ocre realista o azul estilizado.

**Fase 1 — Esqueleto técnico (≈3–5 días)**
6. Crea el repositorio con `create-next-app` (Next 16.4, TS, Tailwind) y añade `three@0.187`, `@react-three/fiber@9.8`, `@react-three/drei@10.7` y `zustand`.
7. Monta una escena mínima (un cubo clickeable) en un client wrapper con `dynamic(..., {ssr:false})` y confirma que no haya errores de React 19.3 ni de "window is not defined". Si falla, fija React según la matriz de soporte de R3F.
8. Configura next-intl (`proxy.ts`, `app/[locale]/`) con `es` y `en`.
9. Crea el esquema Zod `Zone` y `content/zonas/*.json` con 3 zonas de prueba, más un test en Vitest que exija textos ES y EN en cada zona.

**Fase 2 — Datos geográficos y layout (≈1 semana)**
10. Instala BlenderGIS en Blender 5.x (o Blosm en 4.5 LTS) e importa orilla, río y Vía 40/Av. del Río por Overpass o desde un `.osm` local.
11. Traza la curva de la orilla y simplifícala.
12. En una hoja de cálculo, asigna a cada sector su longitud de maqueta (por ejemplo, 5,5 km → 300–600 unidades Blender) según su importancia y no según su longitud real.
13. Coloca losas vacías (cajas con nombre) a lo largo de la curva y valida la composición desde la cámara final.

**Fase 3 — Modelado low-poly (≈2–3 semanas)**
14. Crea el material único con la textura-paleta (interpolación "Closest").
15. Modela la base y el río de cada losa (borde de tierra cortada, agua plana).
16. Importa y recolorea props CC0 de Kenney/Quaternius (palmeras, faroles, bancas, canchas) como *linked duplicates*.
17. Modela los hitos héroe en este orden: Luna del Río → Ventana de Campeones → Pabellón de Cristal → Puerta de Oro → Caimán del Río → El Ancla → estatuas (siluetas simplificadas) → Carrusel.
18. Exagera los hitos 1,5–3× y crea un *hitbox* `*_hit` por hito.
19. Nombra cada objeto (`mon_*`, `zone_*`) y añade Custom Properties (`zone_id`, `order`, `clickable`).
20. Hornea AO en Cycles sobre vertex colors o en un lightmap.
21. Lleva `CREDITS.md` con cada asset CC-BY.

**Fase 4 — Pipeline de exportación (≈2 días)**
22. Exporta un `.glb` con +Y Up, Apply Modifiers y Custom Properties activados, sin Draco.
23. Optimiza con `gltf-transform` (meshopt + webp, `--join false --flatten false`) y comprueba con `gltf-transform inspect` que los nombres de las zonas siguen ahí.
24. Genera `Malecon.tsx` con `gltfjsx --types --keepnames --keepgroups --instance`.
25. Automatiza los pasos 22–24 en un script `npm run model`.

**Fase 5 — Interacción 3D (≈1–2 semanas)**
26. Monta `<Canvas frameloop="demand">` con `CameraControls` restringido: ángulos polares y azimutales limitados y traslación dentro de la caja de la tira. Llama a `invalidate()` en `change`.
27. Implementa `onClick` → `e.stopPropagation()` → `select(userData.zone_id)` en el store, y el hover con cursor y resaltado.
28. Conecta el store con la URL `?zona=slug` (lectura al cargar, `router.replace` al seleccionar).
29. Haz volar la cámara con `setLookAt(...preset, true)` y abre el panel en el evento `rest`.
30. Añade etiquetas `<Html>` solo para los hitos héroe (usa `occlude` con moderación).
31. Implementa "Recorrer la ruta": un bucle `await` sobre las zonas ordenadas por `order`, con botones de Pausa y Siguiente.
32. Añade agua estilizada (plano con desplazamiento de UV), TiltShift2 desactivable y una barca animada sobre una curva.

**Fase 6 — Interfaz, accesibilidad y SEO (≈1 semana)**
33. Crea el panel de información: Sheet lateral en escritorio y Drawer inferior en móvil (shadcn/ui), con carrusel de fotos, horario y enlaces. Se cierra con Esc.
34. Crea `<nav aria-label="Monumentos">` como lista DOM que llama al mismo `focusLandmark(id)` del clic 3D ("Ver monumentos").
35. Respeta `prefers-reduced-motion` (cortes instantáneos y sin tour automático) y ofrece un fallback sin WebGL (imagen pre-renderizada + lista).
36. Añade el selector ES/EN, el modo oscuro (fondo e iluminación de la escena incluidos) y el panel "(i) Créditos" con la atribución de OSM.
37. Genera las páginas estáticas `/[locale]/zona/[slug]` con `generateStaticParams`, `generateMetadata` (OG image) y `ensureStatic = 'navigation'`.

**Fase 7 — Rendimiento y pruebas (≈1 semana)**
38. Mide con `r3f-perf` y añade `PerformanceMonitor` + `AdaptiveDpr` (DPR máximo 1,5).
39. Prueba en un Android de gama media real y recorta hasta cumplir el presupuesto (draw calls, triángulos, tamaño del GLB).
40. Escribe los E2E con Playwright: Chromium `--use-gl=swiftshader`, bandera `canvas.dataset.ready`, selección vía la lista DOM, panel ES/EN, deep link `?zona=` y metadatos OG ([guía Playwright WebGL](https://barthpaleologue.github.io/Blog/posts/webgl-webgpu-playwright-setup)). Ejecútalos en Docker.
41. Revisa con el Bundle Analyzer de Next 16.4 que three/drei no aparezcan en las páginas de zona, y pasa Lighthouse por ellas.

**Fase 8 — Despliegue (≈1–2 días)**
42. Despliega en Vercel Hobby. Mueve el GLB y las fotos a R2 si pesan, con nombres con hash, `Cache-Control` inmutable y CORS.
43. Etiqueta la versión y documenta en el README cómo regenerar el modelo y el contenido.

**Fase 9 — Backend opcional (≈1–2 semanas, solo si el curso lo pide)**
44. Diseña en Neon las tablas `zones`, `zone_translations` y `zone_images` (o JSONB) y escribe las migraciones (Alembic con FastAPI; Prisma o Drizzle con NestJS).
45. Implementa una API de solo lectura (`GET /zones`, `GET /zones/{slug}?locale=`) y un CRUD protegido para administrar.
46. Dockeriza la API (Dockerfile + `docker-compose` con Postgres local para desarrollo) y despliégala en Render o Railway.
47. Cambia la fuente de contenido de Next a *fetch en build* contra la API y añade revalidación al guardar desde el admin. Las páginas públicas no deben depender del backend en tiempo de ejecución.

## Lo que sigue sin verificar

Antes de presentar el proyecto conviene revisar estos puntos, porque ninguna fuente los resolvió:

- **El stack real de "Zaragoza cabe en tu mano"**: desconocido. Todo lo que se diga es inferencia.
- **R3F 9.8.2 con React 19.3**: el rango de peer deps lo admite, pero no hay una nota de versión explícita. Pruébalo en la Fase 1.
- **Coordenadas**: ninguna de los puntos del Malecón está verificada; hay que tomarlas a mano.
- **Orden fino de algunos hitos**: La Madriguera frente al Pabellón, posición del Escudo 3D, posición exacta de los puentes fijo y abatible, y longitud del tramo de La Loma.
- **Horarios oficiales de 2026**: no confirmados. Las páginas de barranquilla.gov.co no cargaron. Tripadvisor indica 5 a. m.–11 p. m. ([Tripadvisor](https://www.tripadvisor.com/Attraction_Review-g297473-d12785540-Reviews-Gran_Malecon-Barranquilla_Atlantico_Department.html)).
- **Compatibilidad de Blosm con Blender 5.x**: no confirmada.
- **Etiquetas exactas del exportador glTF de Blender 5.2**: no confirmadas.
- **`@react-three/a11y`**: sin versión desde 2022, no probado con R3F 9; por eso la accesibilidad se resuelve con la lista DOM.
- **Presupuestos de rendimiento y horas estimadas**: son heurísticos, no medidos.

## Conclusión

Tu proyecto se aparta de Zaragoza justo donde se vuelve más interesante. La forma lineal obliga a decidir qué tramos importan, y esa decisión, la compresión no uniforme sobre la curva real del río, es la que convierte un mapa en una maqueta con intención. Por eso la hoja de escalas de la Fase 2 merece más cuidado que cualquier shader. Además, el inventario envejece rápido: tres de los hitos más llamativos (Luna del Río, Carrusel, Sofía Vergara) y el skatepark aparecieron en los últimos 16 meses, y el Kristal Mall viene en camino. Eso favorece guardar zonas y presets de cámara como datos y modelar cada hito como pieza independiente, para que añadir el próximo sea una losa más y no rehacer la maqueta.

Para tu perfil, la mejor inversión es el orden del plan: primero un MVP estático y sólido en Next + R3F, y solo después la API FastAPI/NestJS en Docker sobre Neon. Con ese orden el backend no frena la parte visual, que es lo que se evalúa a primera vista, y la API llega como una mejora demostrable: un admin real para que los presets y los textos se editen sin tocar código.
