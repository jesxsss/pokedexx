# PokeDex – Cuenta en Azure for Students

Este repositorio contiene la aplicación **PokeDex** (Angular 14) desplegada en **Azure Static Web Apps** usando una suscripción **Azure for Students**.
Este README explica, paso a paso, cómo se creó la cuenta en la nube pública y su configuración inicial. El proceso de despliegue está en [Despliegue.md](Despliegue.md).

- **URL pública:**  https://black-dune-06431e410.6.azurestaticapps.net
- **Repositorio:** https://github.com/jesxsss/pokedexx.git

---

## 1. Crear la cuenta de Azure for Students

1. Entra a <https://azure.microsoft.com/free/students/> y pulsa **Start free** / **Activar ahora**.
2. Inicia sesión con tu **correo institucional** (el de tu universidad). Si no tienes cuenta Microsoft asociada, crea una usando ese correo.
3. Verifica que eres estudiante: Azure valida el dominio del correo institucional (puede pedir un código que llega a ese correo).
4. Completa el formulario de perfil (país, nombre, teléfono) y acepta los términos.
5. No se pide tarjeta de crédito en Azure for Students. Se asignan créditos (aprox. 100 USD) y servicios gratuitos mientras seas estudiante.
6. Al terminar entrarás al portal: <https://portal.azure.com>. Verifica que existe la suscripción **Azure for Students**:
   *Portal → Suscripciones*.

## 2. Configuración inicial en el portal

1. **Suscripciones:** confirma que la suscripción `Azure for Students` está en estado *Activa* y revisa el saldo de créditos en *Cost Management + Billing*.
2. **Grupo de recursos:** *Portal → Grupos de recursos → Crear*.
   - Suscripción: Azure for Students
   - Nombre: `rg-pokedex`
   - Región: `East US 2` (o la más cercana permitida por la suscripción)
3. **(Recomendado) Alerta de presupuesto:** *Cost Management → Budgets → Add* con un límite bajo (ej. 10 USD) para evitar sorpresas.

## 3. Elegir el servicio: Azure Static Web Apps

La PokeDex es una SPA (HTML + JS + CSS) que consume APIs externas (PokéAPI), por lo que no necesita servidor propio. **Azure Static Web Apps** (plan **Free**) es ideal porque:

- Sirve contenido estático con HTTPS y certificado gratuito.
- Se integra con GitHub y despliega con cada `push`.
- Permite configurar **encabezados de seguridad** mediante `staticwebapp.config.json`.

## 4. Crear la cuenta en GitHub y el repositorio

1. Crea o usa tu cuenta en <https://github.com>.
2. Crea un repositorio **público** llamado `pokedex` (sin README ni .gitignore, para evitar conflictos al hacer el primer push).
3. Instala Git y configura tu identidad:

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu-correo@ejemplo.com"
```

## 5. Siguiente paso

Continúa con [Despliegue.md](Despliegue.md) para subir el código y publicar la aplicación.

## 6. Seguridad (resumen)

- Encabezados HTTP configurados en `src/staticwebapp.config.json` (CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy, Permissions-Policy, COOP, CORP).
- Resultado en <https://securityheaders.com/>: 
- Auditorías opcionales: SSL Labs / OWASP ZAP 
## Créditos

Código fuente original: [keilermora/pokedex-angular](https://github.com/keilermora/pokedex-angular).
