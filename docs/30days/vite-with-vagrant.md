[↩︎ Volver al índice](/README.md)

# Configurar Vite para funcionar en un entorno virtualizado

## ¿Qué es _Vite_?

Vite es una herramienta moderna de _bundling_ y desarrollo para aplicaciones web que tiene como objetivo mejorar el rendimiento y la experiencia del desarrollador. A diferencia de otras herramientas como Webpack, Vite está diseñado para ser rápido y eficiente desde el inicio, especialmente en proyectos basados en frameworks modernos como Vue y React, aunque también es compatible con otros entornos como Laravel.

### Opción 1: Ejecutar _Vite_ dentro de la máquina virtual

Si no queremos o no podemos instalar Node.js directamente en la máquina anfitriona, podemos ejecutar _Vite_ en la máquina virtual. Para esto, debemos hacer algunos ajustes, comenzando por el archivo _package.json_. En la sección de configuración de Vite, añade `--host` al script `dev` para que sea accesible a través de la red anfitriona del entorno virtual.

```json
"scripts": {
    "build": "vite build",
    "dev": "vite --host"
}
```

Luego de haber ejecutado `npm install` por primera vez en tu proyecto, edita el archivo `vite.config.js` y agrega la sección `server`.

La configuración `usePolling: true` es necesaria para asegurar que los cambios se detecten en tiempo real en un entorno virtualizado (como el que utilizamos con _Vagrant_). Aunque introduce un ligero retraso, es necesario para manejar la falta de compatibilidad con la observación de archivos en sistemas virtualizados.

Así quedaría el archivo `vite.config.js`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
    server: {
        host: '0.0.0.0',
        port: 5173,
        strictPort: true,
        watch: {
            usePolling: true, // Necesario para entornos virtuales (Vagrant, WSL2, etc.)
            interval: 100, // Intervalo de sondeo en milisegundos
            ignored: ['!**/dist/'],
        },
        hmr: {
            host: '192.168.56.10', // IP de la máquina virtual
            port: 5173,
            protocol: 'ws',
        },
    },
});
```

Por último, edita el archivo `.env` para asegurarte de configurar el parámetro `APP_URL`:

```env
APP_URL=http://30days.isw811.xyz
```

En este punto, estarías listo para ejecutar _Vite_ dentro de la máquina virtual.

```bash
npm run dev --verbose
```

Deberías obtener una salida como la siguiente:

![Vite con --host](/images/30days/vitedev-anfitrion.png)

Desde un navegador en la máquina anfitriona, puedes ir a [http://30days.isw811.xyz:5173/](http://30days.isw811.xyz:5173/) para comprobar que Vite está en ejecución y es accesible desde la máquina anfitriona.

![Vite desde la máquina anfitriona](/images/30days/vite-on-domain.png)

Además, puedes visitar [http://30days.isw811.xyz/](http://30days.isw811.xyz/) (sin el puerto) para visualizar el sitio, tal y como lo has venido haciendo. Internamente, en la máquina virtual, el sitio sigue estando hospedado por Apache2, mediante el vhost _(Virtual Host)_ que se definió previamente.

![Sitio con Vite desde la máquina anfitriona](/images/30days/site-with-vite.png)

### Opción 2: Ejecutar _Vite_ en la máquina anfitriona

Otra opción para usar _Vite_ en nuestra arquitectura de entornos de desarrollo virtualizados es ejecutarlo directamente en la máquina anfitriona. Para esto necesitarás instalar Node.js. Te recomiendo utilizar un gestor de versiones de Node.js; existen alternativas para Windows, Linux y macOS.

#### NVM en Windows

Si utilizas Windows como sistema operativo anfitrión, puedes utilizar [NVM-Windows](https://github.com/coreybutler/nvm-windows) de _Corey Butler_.

Una vez instalado NVM, puedes proceder a instalar la última versión estable de Node.js con:

```bash
nvm install lts
```

Si quieres instalar exactamente la versión recomendada para Laravel 11, el comando sería el siguiente:

```bash
nvm install 22.11.0
```

#### NVM en GNU/Linux

Si utilizas GNU/Linux, Unix, macOS, Windows WSL, o cualquier otro sistema donde puedas correr una shell compatible con POSIX (sh, dash, ksh, zsh, bash), puedes utilizar [nvm](https://github.com/nvm-sh/nvm) de _Creationix_.

Una vez instalado NVM, puedes proceder a instalar la última versión estable de Node.js con:

```bash
nvm install --lts
nvm alias default lts/\*
```

Si quieres instalar exactamente la versión recomendada para Laravel 11, los comandos serían los siguientes:

```bash
nvm install 22.11.0
nvm alias default 22.11.0
```

#### Ejecutar _Vite_

Para utilizar _Vite_ directamente en la máquina anfitriona, no es necesario hacer ninguna configuración particular en el archivo `package.json`. La sección de configuración de Vite quedaría así:

```json
"scripts": {
    "build": "vite build",
    "dev": "vite"
}
```

Luego de haber ejecutado `npm install` por primera vez en tu proyecto (en este caso, lo podrías hacer desde la máquina virtual o desde la máquina anfitriona, ya que tienes Node.js en ambas), revisa la configuración del archivo `vite.config.js`. Debería verse así:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            refresh: true,
        }),
    ],
});
```

Por último, edita el archivo `.env` para asegurarte de configurar el parámetro `APP_URL`:

```env
APP_URL=http://30days.isw811.xyz
```

En este punto, estarías listo para ejecutar _Vite_ desde la máquina anfitriona. Primero debes moverte hasta la carpeta del proyecto. En este caso, asumo que estás usando la convención recomendada en este repositorio `~/VMs/webserver/sites/30days.isw811.xyz`.

```bash
cd ~/VMs/webserver/sites/30days.isw811.xyz
```

Ejecuta _Vite_ con npm así (el parámetro _verbose_ es opcional):

```bash
npm run dev --verbose
```

Deberías obtener una salida como la siguiente:

![Vite con --host](/images/30days/vitedev-anfitrion.png)

Desde un navegador en la máquina anfitriona, puedes ir a [http://localhost:5173/](http://localhost:5173/) para comprobar que Vite está en ejecución y es accesible desde la máquina anfitriona.

![Vite desde la máquina anfitriona](/images/30days/vite-on-localhost.png)

Además, puedes visitar [http://30days.isw811.xyz/](http://30days.isw811.xyz/) para visualizar el sitio tal y como lo has venido haciendo. Internamente, en la máquina virtual, el sitio sigue estando hospedado por Apache2, mediante el vhost _(Virtual Host)_ que se definió previamente.

![Sitio con Vite desde la máquina anfitriona](/images/30days/site-with-vite.png)