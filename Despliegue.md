# Despliegue de la web pokedex en azure

URL: https://proud-desert-03c272f10.5.azurestaticapps.net

Se usó **Azure Static Web Apps** con el plan gratis de students.

## 1. Subir el código a GitHub

1. Descargar el código de `poke-dex-lab`.
2. Probarlo con `npm install` y `npm run build`.
3. Subirlo a un repositorio de GitHub (`PokeDexx`, rama `master`).

## 2. Crear la Static Web App

En portal azure buscamos el servicio → **Static Web Apps** → y le damos en **Crear**:

Configuramos

- Plan: **Free**
- Región: Se uso la region Central US que esta permitida para la suscripción (con `East US 2` dio error)
- Origen: GitHub → repo `PokeDexx`, rama `master`
- Valores preestablecidos: Angular
- Ubicación de la aplicación: `/`
- Ubicación de la API: vacío
- Ubicación de salida: `dist/pokedex-angular`

Azure despliega solo con cada `git push`. Después de crear el recurso hay que hacer `git pull --rebase origin master` antes del siguiente push. Esto se hace porque Azure agrega al repositorio un archivo .yml, que es el pipeline que genera Azure. Entonces, si intentamos hacer push, saldrá un error diciendo que git ve en el repositorio un commit que yo no tengo.

## 3. Cabeceras de seguridad

En esta parte se configuran las cabeceras de seguridad para llegar a la nota A+. Las cabeceras son instrucciones que el servidor le envía al navegador sobre cómo debe proteger la página.

1. Crear `src/staticwebapp.config.json` con las cabeceras (Content-Security-Policy, X-Frame-Options, Permissions-Policy, Strict-Transport-Security, X-Content-Type-Options y Referrer-Policy).
2. Agregarlo a `assets` en `angular.json` para que se copie a `dist/pokedex-angular`.


Esto es necesario porque azure busca el archivo `staticwebapp.config.json` en la carpeta compilada (`dist/pokedex-angular`). Angular, al compilar, solo copia a `dist` lo que está listado en `assets` de `angular.json`. Si el archivo no se agrega ahi, no llega a `dist`, y las cabeceras pueden no aplicarse.

contenido del staticwebapp.config.json

```json
{
  "navigationFallback": {
    "rewrite": "/index.html",
    "exclude": ["/assets/*", "*.{css,js,png,jpg,gif,svg,ico,woff,woff2}"]
  },
  "globalHeaders": {
    "Content-Security-Policy": "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; img-src 'self' data: https://assets.pokemon.com; font-src 'self' data: https://fonts.gstatic.com; connect-src 'self' https://beta.pokeapi.co; frame-ancestors 'none'; base-uri 'self'; form-action 'self'",
    "X-Frame-Options": "DENY",
    "Permissions-Policy": "geolocation=(), microphone=(), camera=(), payment=(), usb=()",
    "Strict-Transport-Security": "max-age=31536000; includeSubDomains; preload",
    "X-Content-Type-Options": "nosniff",
    "Referrer-Policy": "strict-origin-when-cross-origin"
  }
}
```


## 4. Correcciones para que funcione con el Content-Security-Policy

- Se permitió `https://beta.pokeapi.co` en `connect-src` (la app pide los datos ahí).
- Se permitió Google Fonts en `style-src` y `font-src`.
- Se permitió `https://assets.pokemon.com` en `img-src` para las imágenes que se cargan desde ese sitio.
- Se borró el script de GitHub Pages de `src/index.html`: estaba escrito dentro del html y el Content-Security-Policy solo deja ejecutar scripts que vengan de archivos del sitio (`script-src 'self'`). Además no hacía falta en Azure.
- En `angular.json` (`production`) se puso `inlineCritical: false`: Angular generaba un `onload` dentro del HTML para cargar los estilos, y eso también lo bloquea `script-src 'self'`.
- En `environment.prod.ts` se cambió `imagesPath` a `/assets/images` para que carguen las imágenes.

## 5. Publicar

```
npm run build
git add .
git commit -m "Despliegue"
git push
```

Esperar el check en **Actions** de GitHub y abrir la URL.

## 6. Escanear

Entrar a https://securityheaders.com, pegar la URL y presionar `Scan`.
