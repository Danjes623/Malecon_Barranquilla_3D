# Plan legal

> Proyecto académico/personal, no comercial y **no oficial**. Este documento no es asesoría jurídica. Ante dudas o uso comercial, consultar a un abogado.
> Responsable de mantenerlo: **Mery**. Responsable de `CREDITS.md` de assets 3D: **Daniel**.

## 1. Naturaleza del proyecto

- Proyecto no oficial, sin relación con la Alcaldía de Barranquilla, Puerta de Oro, el Junior ni otras entidades.
- El sitio muestra un aviso visible: *"Proyecto independiente y no oficial. No está afiliado a la Alcaldía de Barranquilla ni a los operadores del Gran Malecón."*
- No se usan logos, escudos oficiales ni identidad gráfica de esas entidades como marca del sitio.

## 2. Licencias del proyecto

| Elemento | Licencia propuesta | Decisión en |
|---|---|---|
| Código fuente | MIT | F0-09 |
| Modelo 3D propio (`.blend`, `.glb`) | CC BY-NC 4.0 (evita reventa del modelo) | F0-09 |
| Textos de zonas | CC BY 4.0 | F0-09 |

La decisión final queda registrada en `LICENSE` y en el README.

## 3. Datos geográficos

| Fuente | Uso permitido | Obligación |
|---|---|---|
| **OpenStreetMap** (ODbL) | Referencia para orilla, río y vías | Atribución "© OpenStreetMap contributors" con enlace a `https://www.openstreetmap.org/copyright`, visible en el visor o en el panel "(i) Créditos" |
| Mapillary / KartaView | Referencia visual | Respetar la licencia de cada imagen; no redistribuirlas |
| **Google Photorealistic 3D Tiles / Google Maps / Street View** | **Ninguno** | Sus políticas prohíben objetos derivados "by hand or machine" y el uso offline |

- No se publican la geometría ni las etiquetas OSM crudas dentro del GLB. Las capas `ref_*` no se exportan.

## 4. Assets 3D de terceros

| Fuente | Licencia | Regla |
|---|---|---|
| Kenney | CC0 | Se puede usar y modificar; se acredita por cortesía |
| Quaternius | CC0 | Igual |
| Poly Pizza | Mixta (CC0 / CC-BY) | Registrar autor, URL y licencia **por modelo** |
| Sketchfab u otros | Variable | Solo CC0 o CC-BY; nunca "Editorial", "NoDerivatives" ni sin licencia clara |

Todo asset de terceros se registra en `CREDITS.md` en el mismo PR que lo añade:

```
| Asset | Autor | Fuente (URL) | Licencia | Modificado |
```

## 5. Monumentos, esculturas y personas

- Las esculturas (Shakira, Sofía Vergara, Rin Rin Renacuajo, El Ancla) son obras de autores identificados. Se modelan como **siluetas simplificadas low-poly** con fines informativos.
- En el panel se cita al autor de cada obra (ej. "Escultura de Yino Márquez").
- No se usan nombres ni imágenes de personas famosas para promocionar el sitio, ni de forma que sugiera respaldo.
- Pendiente de verificar: alcance de la excepción de obras en espacio público en la legislación colombiana de derecho de autor (Ley 23 de 1982) para reproducciones 3D. Mientras no se confirme, mantener el uso no comercial y la representación simplificada.
- La "Ventana de Campeones" alude al Junior: no reproducir su escudo ni marca.

## 6. Fotografías y textos

- Preferir **fotos propias**. Sin personas identificables en primer plano; si aparecen, desenfocar o pedir permiso.
- Fotos de terceros solo con licencia CC compatible o permiso escrito. Cada imagen lleva `credit` en el JSON.
- Prohibido copiar fotos de prensa (El Heraldo, El Tiempo, Infobae, etc.).
- Los textos se redactan con palabras propias. Las fuentes se enlazan en el campo `sources` de cada zona.
- Datos (fechas, alturas, horarios) siempre con fuente. Si no hay fuente, se marca como "no verificado" o no se publica.

## 7. Privacidad (Ley 1581 de 2012 — habeas data)

- El MVP **no recoge datos personales**: sin formularios, sin cuentas, sin cookies de seguimiento.
- Si se activan analíticas, usar una opción sin cookies ni datos personales (ej. Vercel Web Analytics) y mencionarlo en una página de privacidad.
- Si la Fase 9 añade login de administración, solo para Daniel y Mery; sin datos de visitantes.
- Cualquier recolección futura de datos requiere política de tratamiento y aviso de privacidad **antes** de desplegarla.

## 8. Hosting y servicios

- **Vercel Hobby**: solo uso personal y no comercial. Si el proyecto se vuelve un encargo pagado (ej. turismo), migrar a plan Pro u otro hosting antes de cobrar.
- Neon, Cloudflare R2, Render: revisar sus términos al activarlos (Fase 8–9).

## 9. Checklist legal antes de cada release

- [ ] Atribución OSM visible.
- [ ] `CREDITS.md` completo y coincide con los assets del GLB.
- [ ] Cada imagen tiene `credit` y licencia válida.
- [ ] Aviso de "proyecto no oficial" visible.
- [ ] Sin logos oficiales ni marcas de terceros.
- [ ] Sin datos derivados de Google 3D Tiles.
- [ ] `LICENSE` presente.
