# Despliegue de PokeDex en Azure Static Web Apps

Guía paso a paso para publicar la aplicación (Angular 14) en Azure Static Web Apps con despliegue continuo desde GitHub.

## Requisitos

- Cuenta Azure for Students (ver [README.md](README.md)).
- Cuenta de GitHub y repositorio `pokedex` creado vacío.
- Git instalado.
- (Opcional, para probar en local) **Node.js 18 LTS** y npm. *Angular 14 no es compatible con Node 20+.*

## 1. Cambios hechos al código fuente original

| Archivo | Cambio | Motivo |
|---|---|---|
| `src/environments/environment.prod.ts` | `imagesPath: '/assets/images'` | El original usaba `/pokedex-angular/` (ruta de GitHub Pages). En Azure la app está en la raíz. |
| `src/index.html` | Se eliminó el script inline de "SPA para GitHub Pages" | Un script inline obliga a usar `unsafe-inline` en la CSP y baja la nota de seguridad. |
| `src/404.html` y `angular.json` | Se eliminó `404.html` de los assets | Solo servía para GitHub Pages. |
| `src/staticwebapp.config.json` | **Nuevo** | Fallback de rutas de la SPA + encabezados de seguridad. Se copia a la raíz del build. |
| `angular.json` | Se agregó `staticwebapp.config.json` a `assets` | Para que quede en `dist/pokedex-angular/`. |
| `package.json` | `engines.node` y `prepare` tolerante | Evita que Azure use una versión de Node incompatible y que `husky` falle en CI. |
| `.github/workflows/pipelines.yml` | Eliminado | Era el pipeline de cobertura (Codecov) del autor original y fallaría sin token. |

## 2. Subir el código a GitHub

Desde la carpeta del proyecto:

```bash
git init
git add .
HUSKY=0 git commit --no-verify -m "feat: pokedex lista para Azure Static Web Apps"
git branch -M main
git remote add origin https://github.com/<TU-USUARIO>/pokedex.git
git push -u origin main
```

> Si Git pide credenciales, usa tu usuario y un **Personal Access Token** (GitHub → Settings → Developer settings → Tokens) en lugar de la contraseña, o inicia sesión con GitHub CLI (`gh auth login`).

## 3. Crear el recurso Static Web App

1. Portal de Azure → **Crear un recurso** → buscar **Static Web App** → **Crear**.
2. Pestaña **Datos básicos**:
   - Suscripción: `Azure for Students`
   - Grupo de recursos: `rg-pokedex`
   - Nombre: `pokedex`
   - Tipo de plan: **Free**
   - Región: la más cercana disponible
3. **Detalles de implementación**:
   - Origen: **GitHub** → *Iniciar sesión con GitHub* y autorizar.
   - Organización: tu usuario · Repositorio: `pokedex` · Rama: `main`
4. **Detalles de compilación**:
   - Valores preestablecidos: **Angular**
   - Ubicación de la aplicación: `/`
   - Ubicación de la API: *(vacío)*
   - Ubicación de salida: `dist/pokedex-angular`
5. **Revisar y crear** → **Crear**.

Azure agrega automáticamente al repositorio un workflow en `.github/workflows/azure-static-web-apps-*.yml` y el secreto `AZURE_STATIC_WEB_APPS_API_TOKEN_*`.

## 4. Verificar el workflow

1. En GitHub → pestaña **Actions**: debe aparecer una ejecución "Azure Static Web Apps CI/CD" que termine en verde (tarda unos minutos).
2. Si falla por la versión de Node, abre el workflow generado y en el paso de `Azure/static-web-apps-deploy@v1` agrega en `env:` la variable:

```yaml
env:
  NODE_VERSION: 18
```

3. Descarga el workflow que creó Azure a tu máquina para no tener conflictos en futuros push:

```bash
git pull origin main
```

## 5. Obtener la URL pública

Portal → recurso `pokedex` → **Información general** → **URL** (`https://xxxx.azurestaticapps.net`). Abre la URL y comprueba que:

- Carga el listado de Pokémon.
- Al entrar a un Pokémon y recargar la página (F5) no da 404.
- Las imágenes se ven.
- La consola del navegador (F12) no muestra errores de CSP.

## 6. Encabezados de seguridad

Están definidos en `src/staticwebapp.config.json` → `globalHeaders`:

| Encabezado | Valor / propósito |
|---|---|
| `Strict-Transport-Security` | Fuerza HTTPS por 2 años |
| `Content-Security-Policy` | Solo permite recursos propios + Google Fonts, `assets.pokemon.com` (imágenes) y PokéAPI (datos) |
| `X-Frame-Options` | `DENY` – evita clickjacking |
| `X-Content-Type-Options` | `nosniff` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | Desactiva cámara, micrófono, geolocalización, etc. |
| `Cross-Origin-Opener-Policy` / `Cross-Origin-Resource-Policy` | Aislamiento de origen |

### Escaneo

1. Entra a <https://securityheaders.com/>.
2. Pega la URL de tu app, marca *Follow redirects* y pulsa **Scan**.
3. Guarda captura del resultado y pégala en el README.

Para puntos extra: escanea el dominio en <https://www.ssllabs.com/ssltest/> y/o ejecuta OWASP ZAP (Automated Scan) contra la URL.

## 7. Actualizar la aplicación

Cada `git push` a `main` dispara un nuevo despliegue automático:

```bash
git add .
git commit -m "cambios"
git push
```

## 8. Probar en local (opcional)

```bash
npm install
npm start          # http://localhost:4200
npm run build      # genera dist/pokedex-angular
```

## Problemas frecuentes

- **Pantalla en blanco y errores CSP en consola:** falta autorizar un dominio en `Content-Security-Policy` (agregarlo en `connect-src`, `img-src`, etc.).
- **Falla `npm install` en Actions:** usar Node 18 (ver paso 4).
- **404 al recargar una ruta:** revisa que `staticwebapp.config.json` esté dentro de `dist/pokedex-angular`.
