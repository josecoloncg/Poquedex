# Guía de Despliegue — Pokédex Angular

## Índice

- Requisitos previos
- Paso 1: Configuración del repositorio en GitHub
- Paso 2: Creación del recurso en Azure Static Web Apps
- Paso 3: Configuración del pipeline CI/CD
- Paso 4: Seguridad HTTP
- Paso 5: Problemas comunes y soluciones
- Paso 6: Verificación final

---

## Requisitos previos

- Cuenta en [GitHub](https://github.com)
- Cuenta en [Azure for Students](https://azure.microsoft.com/es-es/free/students)
- Node.js 16 o superior instalado
- Angular CLI (`npm install -g @angular/cli`)
- Git instalado

---

## Paso 1: Configuración del repositorio en GitHub

1. Accede a [https://github.com/new](https://github.com/new)
2. Nombra el repositorio: `pokedex-angular`
3. Selecciona visibilidad **Pública**
4. Haz clic en **"Create repository"**

### Subir el código fuente

```bash
git init
git add .
git commit -m "feat: primer commit - pokedex angular"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/pokedex-angular.git
git push -u origin main
```

### Verifica la estructura

Asegúrate de que el repositorio contenga al menos:

```
/
├── src/
├── angular.json
├── package.json
└── tsconfig.json
```

---

## Paso 2: Creación del recurso en Azure

1. Ingresa a [https://portal.azure.com](https://portal.azure.com)
2. Inicia sesión con tu cuenta
3. Haz clic en "Crear un recurso"
4. Busca "Static Web App" y selecciona la opción de Microsoft
5. Haz clic en "Crear"

Completa el formulario:

| Campo             | Valor                        |
|-------------------|------------------------------|
| Suscripción       | Azure for Students           |
| Grupo de recursos | `rg-pokedex` (nuevo)         |
| Nombre            | `app-pokedex`                |
| Plan de hospedaje | Free                         |
| Región            | East US 2                    |
| Origen            | GitHub                       |

Conecta con GitHub, selecciona tu usuario, el repositorio `pokedex-angular` y la rama `main`.

Configura el build:

| Campo           | Valor                |
|-----------------|----------------------|
| App location    | `/`                  |
| Api location    | *(dejar vacío)*      |
| Output location | `dist/pokedex-angular` |

> **Nota:** El campo `output_location` debe coincidir con el `outputPath` de `angular.json`.

Haz clic en "Revisar y crear", revisa el resumen y confirma la creación. Espera mientras Azure aprovisiona el recurso.

---

## Paso 3: Configuración del pipeline CI/CD

Azure generará automáticamente el archivo `.github/workflows/azure-static-web-apps.yml` en tu repositorio. Este archivo define el pipeline de integración y despliegue continuo.

Cada vez que haces `git push` a la rama `main`, GitHub Actions ejecuta estos pasos:

1. Clona el repositorio
2. Instala dependencias con `npm install`
3. Ejecuta el build de producción de Angular (`ng build --configuration production`)
4. Sube el contenido de `dist/pokedex-angular/` a Azure Static Web Apps

Para verificar el estado del pipeline:

1. Ve a tu repositorio en GitHub
2. Haz clic en la pestaña "Actions"
3. Verifica que el último workflow esté en verde ✅

---

## Paso 4: Seguridad HTTP

Azure Static Web Apps no añade encabezados de seguridad por defecto. Para configurarlos, usa el archivo `staticwebapp.config.json` en la raíz del proyecto.

Crea el archivo `staticwebapp.config.json` en la raíz del repositorio:

```json
{
  "globalHeaders": {
    "Content-Security-Policy": "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com; img-src 'self' data: https: blob:; frame-ancestors 'none'; connect-src 'self' https://beta.pokeapi.co https://pokeapi.co https://*.pokeapi.co",
    "Strict-Transport-Security": "max-age=31536000; includeSubDomains; preload",
    "X-Content-Type-Options": "nosniff",
    "X-Frame-Options": "DENY",
    "Referrer-Policy": "no-referrer",
    "Permissions-Policy": "geolocation=(), microphone=(), camera=()"
  },
  "navigationFallback": {
    "rewrite": "/index.html",
    "exclude": ["/assets/*", "/*.css", "/*.js", "/*.png", "/*.gif", "/*.jpg"]
  }
}
```

Confirma y despliega:

```bash
git add staticwebapp.config.json
git commit -m "security: agregar encabezados HTTP de seguridad"
git push
```

GitHub Actions detectará el push y desplegará automáticamente con los nuevos headers.

Para verificar los encabezados:

1. Ve a [https://securityheaders.com](https://securityheaders.com)
2. Ingresa la URL pública de tu aplicación
3. Verifica la calificación obtenida

---

## Paso 5: Problemas comunes y soluciones

### ❌ Error: Imágenes de Pokémon no se muestran en producción

**Síntoma:** Las cards de Pokémon aparecen vacías en producción, pero funcionan localmente.

**Diagnóstico:** Usando las DevTools del navegador, inspecciona el HTML generado y verifica la ruta de las imágenes.

**Causa raíz:** En `src/environments/environment.prod.ts` la ruta de las imágenes puede estar así:

```typescript
imagesPath: '/pokedex-angular/assets/images',
```

Esto es incorrecto para producción en Azure. Debe ser:

```typescript
imagesPath: '/assets/images',
```

Haz commit y push para que GitHub Actions ejecute un nuevo build.

**Resultado:** Las imágenes aparecen correctamente tras el despliegue. ✅

---

### ❌ Advertencias de budget en el build

**Síntoma:** Durante el build aparecen advertencias de tamaño de bundle.

**Solución:** Son advertencias, no errores bloqueantes. El build se completa y la app funciona. Si lo deseas, puedes aumentar el budget en `angular.json`.

---

## Paso 6: Verificación final

### Checklist antes de entregar

- [x] La aplicación carga correctamente desde la URL pública
- [x] Se pueden ver los Pokémon con sus imágenes
- [x] Al hacer clic en un Pokémon se abre el detalle
- [x] Los filtros funcionan
- [x] HTTPS activo
- [x] Sin errores en la consola del navegador
- [x] Calificación A en securityheaders.com
- [x] Pipeline de GitHub Actions en verde

### URL del recurso en Azure

- **Aplicación:** [https://zealous-field-0bb1e0a0f.7.azurestaticapps.net/]
- **Repositorio:** [https://github.com/josecoloncg/Poquedex]
- **Pipeline:** GitHub Actions → pestaña Actions del repositorio
