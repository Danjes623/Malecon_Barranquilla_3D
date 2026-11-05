# Prohibiciones

> Lo que **no** se hace en este proyecto. Saltarse una prohibición requiere un PR que modifique este archivo con la justificación y la aprobación de ambos.

## Git y repositorio

1. Prohibido hacer *push* directo a `main`. Todo entra por PR.
2. Prohibido fusionar un PR propio sin la aprobación de la otra persona.
3. Prohibido fusionar con CI en rojo.
4. Prohibido `git push --force` sobre `main` o sobre ramas compartidas.
5. Prohibido subir secretos, tokens, contraseñas o archivos `.env`.
6. Prohibido subir `.blend`, `.glb` fuente o fotos pesadas **sin Git LFS**.
7. Prohibido subir `node_modules/`, `.next/`, builds, `*.blend1` o archivos temporales.
8. Prohibido reescribir el historial de `main`.

## Datos y licencias

9. Prohibido usar **Google Photorealistic 3D Tiles, Google Maps o Street View** para calcar, derivar o referenciar geometría.
10. Prohibido usar assets sin licencia clara, con licencia "Editorial", "NoDerivatives" o "NonCommercial" incompatibles.
11. Prohibido añadir un asset de terceros sin registrarlo en `CREDITS.md` en el mismo PR.
12. Prohibido copiar fotos o textos de prensa o de sitios oficiales.
13. Prohibido usar logos, escudos o marcas oficiales (Alcaldía, Puerta de Oro, Junior, etc.).
14. Prohibido presentar el sitio como oficial.
15. Prohibido publicar un dato (fecha, altura, horario) sin fuente en `sources`.
16. Prohibido publicar la geometría o etiquetas OSM crudas dentro del GLB.

## Contenido de la maqueta

17. Prohibido incluir lugares que no están en el Gran Malecón: Ventana al Mundo, estatua de Joe Arroyo, Parque Cultural del Caribe, Gran Malecón del Suroriente, Kristal Mall.
18. Prohibido numerar zonas por "unidad funcional"; se etiquetan por nombre de sector.
19. Prohibido modelar personas reales con detalle realista; las estatuas son siluetas simplificadas.
20. Prohibido fotografiar o publicar personas identificables sin permiso.

## Técnico

21. Prohibido usar **R3F 10** (alfa) o el renderer **WebGPU**.
22. Prohibido usar **Theatre.js**, **Spline** (exportar código es de pago) o `@react-three/a11y`.
23. Prohibido importar three.js en componentes de servidor o en páginas de zona.
24. Prohibido renderizar el canvas con SSR (siempre `dynamic(..., { ssr: false })`).
25. Prohibido poner presets de cámara o textos de zonas en el código; van en `content/`.
26. Prohibido usar `gltfjsx --transform` sobre un GLB ya optimizado.
27. Prohibido optimizar sin `--join false --flatten false` (se pierde la separación de zonas).
28. Prohibido renombrar un `slug` o un `meshName` publicado sin migrar contenido y enlaces.
29. Prohibido usar sombras en tiempo real; la iluminación se hornea.
30. Prohibido `dangerouslySetInnerHTML` con contenido de zonas.
31. Prohibido poner claves privadas en variables `NEXT_PUBLIC_*`.
32. Prohibido superar el presupuesto de rendimiento (`design.md` §8) sin acuerdo explícito.
33. Prohibido que las páginas públicas dependan de un backend en tiempo de ejecución.

## Hosting

34. Prohibido usar Vercel Hobby para un uso comercial o encargo pagado.
35. Prohibido desplegar a producción desde la máquina local salvo emergencia acordada por ambos.
36. Prohibido activar analíticas con cookies o datos personales.
