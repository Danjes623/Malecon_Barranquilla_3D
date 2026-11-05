# Plan de seguridad

> Responsable de configuración del repo: **Daniel** (admin). Responsable de CI, dependencias y cabeceras: **Mery**.

## 1. Cuentas y acceso

- [ ] **2FA obligatorio** en GitHub, Vercel y (si se usan) Cloudflare, Neon y Render, para ambos.
- [ ] Repo con dos colaboradores: Daniel (admin) y Mery (maintain o write).
- [ ] Vercel: proyecto en cuenta Hobby conectada al repo. Si la cuenta es de Daniel, Mery accede a los *previews* por enlace; no se comparten contraseñas.
- [ ] Nunca compartir contraseñas ni tokens por chat. Usar un gestor de contraseñas si hace falta compartir algo.

## 2. Protección del repositorio

- [ ] Rama `main` protegida:
  - PR obligatorio, mínimo **1 aprobación** de la otra persona.
  - CI en verde obligatorio (`lint`, `typecheck`, `test`, `build`).
  - Sin *force push* ni borrado.
- [ ] **Secret scanning** y **push protection** activados.
- [ ] **Dependabot** activado (alertas + actualizaciones de seguridad).
- [ ] `CODEOWNERS`:
  ```
  /blender/        @<usuario-daniel>
  /public/models/  @<usuario-daniel>
  /src/            @<usuario-mery>
  /content/        @<usuario-mery>
  /Docs/           @<usuario-daniel> @<usuario-mery>
  ```

## 3. Secretos

- Ningún secreto en el repositorio. `.env*` en `.gitignore`; solo se sube `.env.example` con nombres vacíos.
- El MVP no necesita secretos (todo es estático).
- Variables con prefijo `NEXT_PUBLIC_` son **públicas**: nunca poner ahí claves privadas.
- Secretos de producción solo en **Vercel → Environment Variables** y en **GitHub Actions secrets**.
- Si un secreto se sube por error: **revocarlo de inmediato** en el servicio, rotarlo y después limpiar el historial. Borrar el commit no basta.

## 4. Dependencias

- Versiones fijadas con `package-lock.json` (o `pnpm-lock.yaml`) siempre subido.
- `npm audit --audit-level=high` en CI.
- Antes de añadir un paquete: comprobar mantenimiento reciente, descargas y licencia. Evitar paquetes abandonados (ej. Theatre.js, `@react-three/a11y`).
- Revisar los PR de Dependabot como cualquier otro PR.
- No ejecutar scripts `postinstall` de paquetes desconocidos sin revisarlos.

## 5. Sitio web (cabeceras)

Configurar en `next.config.ts` (o `vercel.json`):

| Cabecera | Valor base |
|---|---|
| `Content-Security-Policy` | `default-src 'self'; img-src 'self' data: blob: <dominio-R2>; connect-src 'self' <dominio-R2>; script-src 'self' 'unsafe-eval'*; style-src 'self' 'unsafe-inline'; frame-ancestors 'none'` |
| `X-Content-Type-Options` | `nosniff` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=()` |
| `Strict-Transport-Security` | Vercel lo gestiona en dominios propios |

\* Verificar si three/drei necesita `'unsafe-eval'`; quitarlo si no hace falta.

- Enlaces externos con `rel="noopener noreferrer"`.
- Contenido de zonas renderizado como texto, nunca con `dangerouslySetInnerHTML`.
- Todo JSON de contenido validado con Zod en build.

## 6. Assets y almacenamiento (R2, Fase 8)

- Bucket de solo lectura pública para GLB y fotos; escritura solo con token de CI.
- CORS limitado al dominio de producción y a `*.vercel.app` del proyecto.
- Nombres con hash + `Cache-Control: public, max-age=31536000, immutable`.

## 7. Backend opcional (Fase 9)

- API pública **solo lectura**. CRUD protegido por autenticación (solo Daniel y Mery).
- Conexión a Neon con usuario de mínimos privilegios para lectura; usuario aparte para migraciones.
- Validación de entrada en todos los endpoints; consultas parametrizadas (ORM).
- CORS limitado al dominio del sitio. Rate limiting en endpoints de escritura.
- Imágenes Docker con base oficial y versión fijada; sin secretos en el `Dockerfile`.

## 8. Incidentes

1. Detectar (alerta de GitHub, Vercel o reporte).
2. Contener: revocar credenciales, revertir el despliegue en Vercel (*Instant Rollback*).
3. Corregir por PR con revisión.
4. Anotar en un issue con etiqueta `security`: qué pasó, impacto y qué cambió.
