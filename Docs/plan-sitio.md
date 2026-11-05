# Plan del sitio — Despliegue en Vercel

> Responsable: **Mery**. Daniel aporta el GLB optimizado y el render de fallback/OG.

## 1. Por qué Vercel

- Soporte nativo de Next.js 16 (App Router, páginas estáticas, imágenes, OG).
- **Preview deployment por cada PR**: la otra persona revisa el cambio en vivo antes de aprobar.
- Plan **Hobby gratuito**: uso personal no comercial, 100 GB/mes de transferencia. Con un GLB de 3 MB da para ~33.000 cargas completas/mes (sin contar caché del navegador).
- Rollback instantáneo.

Si el proyecto pasa a uso comercial, Hobby deja de servir (ver `plan-legal.md` §8).

## 2. Entornos

| Entorno | Rama | URL | Uso |
|---|---|---|---|
| Producción | `main` | `malecon-3d.vercel.app` (o dominio propio) | Público |
| Preview | cualquier rama con PR | `*-<proyecto>.vercel.app` | Revisión de PR |
| Local | — | `http://localhost:3000` | Desarrollo |

## 3. Configuración inicial (F1-07)

1. Iniciar sesión en Vercel con GitHub (2FA activo).
2. *Add New → Project* → importar el repo.
3. Ajustes:
   - Framework: **Next.js** (autodetectado).
   - Node.js: **20.x o superior** (Next 16 exige ≥ 20.9).
   - Build command: `npm run build` · Install: `npm ci`.
   - Root directory: `/`.
4. Activar *Preview Deployments* para todas las ramas.
5. Activar *Deployment Protection* en previews si no se quieren públicos.
6. Variables de entorno: ninguna en el MVP. Las futuras se crean por entorno (Production / Preview), nunca en el repo.

## 4. Flujo de despliegue

```
rama feature → PR → CI (lint, typecheck, test, build) + Preview Vercel
            → revisión de la otra persona sobre el preview
            → merge a main → despliegue de producción automático
```

- Nunca desplegar a producción desde la CLI local salvo emergencia acordada.
- Si producción falla: **Instant Rollback** en el panel de Vercel y luego corregir por PR.

## 5. Assets pesados

| Situación | Dónde | Caché |
|---|---|---|
| GLB ≤ ~5 MB y pocas fotos | `public/models/`, `public/images/` | Nombres con hash o versión (`malecon.v3.glb`) + `Cache-Control: public, max-age=31536000, immutable` |
| GLB o fotos pesan más / se acerca el límite de transferencia | **Cloudflare R2** (10 GB gratis, sin cobro de egreso) | Igual, con CORS limitado al dominio |

Cabecera de caché en `next.config.ts`:

```ts
async headers() {
  return [
    { source: "/models/:path*", headers: [{ key: "Cache-Control", value: "public, max-age=31536000, immutable" }] },
  ];
}
```

## 6. Requisitos de calidad antes de producción

- [ ] Páginas `/[locale]/zona/[slug]` estáticas (`ensureStatic = 'navigation'` sin errores).
- [ ] three/drei no aparece en el bundle de las páginas de zona (Bundle Analyzer).
- [ ] Lighthouse en páginas de zona: Performance ≥ 90, Accessibility ≥ 95, SEO ≥ 95.
- [ ] GLB dentro del presupuesto (`design.md` §8).
- [ ] Vista previa OG correcta al compartir por WhatsApp.
- [ ] Fallback sin WebGL funciona.
- [ ] Cabeceras de seguridad activas (`plan-seguridad.md` §5).
- [ ] Atribución OSM y aviso "no oficial" visibles (`plan-legal.md`).

## 7. Dominio (opcional)

- Por defecto se usa el subdominio `*.vercel.app`.
- Si se compra dominio: añadirlo en *Settings → Domains*, configurar DNS según indique Vercel, HTTPS automático.

## 8. Observabilidad

- Vercel Web Analytics (sin cookies) y Speed Insights, opcionales. Mencionarlos en la página de privacidad si se activan.
- Revisar logs de build en cada PR fallido antes de reintentar.

## 9. Backend opcional (Fase 9)

- La API (FastAPI/NestJS) **no** se despliega en Vercel: va en Render o Railway en Docker, con base en Neon.
- Next la consulta **solo en build** para que el sitio no dependa de un servidor dormido (Render Free se apaga tras 15 min sin tráfico).
- Revalidación al guardar desde el admin mediante *Deploy Hook* o `revalidateTag` protegido por secreto.
