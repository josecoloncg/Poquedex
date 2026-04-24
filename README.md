# Pokédex Angular — Despliegue en Azure Static Web Apps

![Azure](https://img.shields.io/badge/Azure-Static%20Web%20App-blue?logo=microsoftazure)
![Angular](https://img.shields.io/badge/Angular-14-red?logo=angular)
![Security](https://img.shields.io/badge/Security%20Headers-A-brightgreen)

Aplicación web tipo Pokédex desarrollada en Angular, desplegada en Microsoft Azure Static Web Apps con encabezados HTTP de seguridad configurados para obtener calificación A+ en [securityheaders.com](https://securityheaders.com).

**URL pública:** [https://zealous-field-0bb1e0a0f.7.azurestaticapps.net/]

---

## Tabla de contenidos

- [Descripción del proyecto](#descripción-del-proyecto)
- [Tecnologías utilizadas](#tecnologías-utilizadas)
- [Despliegue en Azure](#despliegue-en-azure)
- [Estructura del repositorio](#estructura-del-repositorio)
- [Seguridad implementada](#seguridad-implementada)
- [Resultado del escaneo de seguridad](#resultado-del-escaneo-de-seguridad)
- [Reflexión técnica](#reflexión-técnica)

---

## Descripción del proyecto

Pokédex Angular es una aplicación web estática desarrollada en Angular que permite explorar Pokémon de distintas generaciones. La información se consume desde la API pública [PokéAPI](https://pokeapi.co). Este repositorio documenta su desarrollo y despliegue en la nube.

---

## Tecnologías utilizadas

- **Angular** — Framework frontend
- **Microsoft Azure Static Web Apps** — Plataforma de hosting
- **GitHub Actions** — CI/CD automático
- **PokéAPI** — API REST y GraphQL de datos Pokémon
- **securityheaders.com** — Validación de cabeceras de seguridad

---

## Despliegue en Azure

1. Crea una cuenta en [Azure for Students](https://azure.microsoft.com/es-es/free/students) o [Azure](https://azure.microsoft.com/)
2. Accede al [Portal de Azure](https://portal.azure.com)
3. Haz clic en "Crear un recurso" y busca "Static Web App"
4. Completa el formulario:
   - **Suscripción:** Azure for Students
   - **Grupo de recursos:** Crear uno nuevo (ej: `rg-pokedex`)
   - **Nombre:** `app-pokedex`
   - **Plan de hospedaje:** Free
   - **Región:** East US 2
   - **Origen de implementación:** GitHub
5. Autoriza Azure para acceder a tu repositorio de GitHub
6. Selecciona el repositorio y la rama `main`
7. Configura los parámetros de build:
   - **App location:** `/`
   - **Api location:** *(vacío)*
   - **Output location:** `dist/pokedex-angular`
8. Haz clic en "Revisar y crear" → "Crear"

Azure generará automáticamente un archivo `.github/workflows/azure-static-web-apps.yml` en el repositorio para CI/CD.

---

## Estructura del repositorio

```
pokedex-angular/
├── .github/
│   └── workflows/
│       └── azure-static-web-apps.yml   ← Pipeline CI/CD
├── src/
│   ├── app/                            ← Componentes Angular
│   ├── assets/                         ← Imágenes y fuentes
│   └── environments/
│       ├── environment.ts              ← Config desarrollo
│       └── environment.prod.ts         ← Config producción
├── staticwebapp.config.json            ← Configuración Azure + Headers de seguridad
├── angular.json
└── README.md
```

---

## Seguridad implementada

La seguridad se configura mediante el archivo `staticwebapp.config.json` ubicado en la raíz del proyecto. Este archivo le indica a Azure qué encabezados HTTP debe enviar junto a cada respuesta.

### Archivo `staticwebapp.config.json`

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

#### Descripción de cada encabezado

| Encabezado | Propósito |
|---|---|
| `Content-Security-Policy` | Previene XSS controlando de dónde se cargan recursos |
| `Strict-Transport-Security` | Fuerza HTTPS por 1 año |
| `X-Content-Type-Options` | Evita que el navegador adivine tipos de archivo |
| `X-Frame-Options` | Bloquea que la app se muestre en iframes (anti-clickjacking) |
| `Referrer-Policy` | No revela la URL de origen al navegar a otros sitios |
| `Permissions-Policy` | Restringe acceso a hardware del dispositivo |

---

## Resultado del escaneo de seguridad

Calificación obtenida en [securityheaders.com](https://securityheaders.com): *A+*

Las advertencias sobre `unsafe-inline` y `unsafe-eval` en el `script-src` son inherentes al funcionamiento de Angular, que requiere estos flags para compilar y ejecutar templates en el navegador.

---

## Reflexión técnica

### 1. ¿Qué vulnerabilidades previenen los encabezados implementados?

Cada encabezado actúa como una defensa contra ataques como XSS, clickjacking, man-in-the-middle, MIME sniffing y protege la privacidad del usuario.

### 2. ¿Qué aprendiste sobre la relación entre despliegue y seguridad web?

Desplegar una aplicación funcional y una segura son objetivos distintos que deben planearse juntos. La seguridad debe ser parte integral del proceso de despliegue.

### 3. ¿Qué desafíos encontraste en el proceso?

El principal desafío fue ajustar rutas y variables de entorno para producción, especialmente para recursos estáticos como imágenes. Es fundamental revisar los archivos de entorno antes de cada despliegue y no asumir rutas locales.
