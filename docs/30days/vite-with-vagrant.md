[↩︎ Volver al índice](/README.md)

# Configurar Vite para funcionar en entorno virtualizado

## ¿Qué es _Vite_?

Vite es una herramienta moderna de bundling y desarrollo para aplicaciones web que tiene como objetivo mejorar el rendimiento y la experiencia del desarrollador. A diferencia de otras herramientas como Webpack, Vite está diseñado para ser rápido y eficiente desde el inicio, especialmente en proyectos basados en frameworks modernos como Vue y React, aunque también es compatible con otros entornos como Laravel.

## ¿Cómo usar _Vite_ en una instancia virtual?

Para utilizar _Vite_ en una arquitectura de desarrollo basada en virtualización, debemos hacer un ajuste en el archivo _package.json_. En la sección de configuración de Vite, añade `--host` al script `dev` para que sea accesible a través de la red anfitriona del entorno virtual.

```json
"scripts": {
    "build": "vite build",
    "dev": "vite --host"
}
```

Luego de haber ejecutado `npm install` por primera vez en tu proyecto, edita el archivo `vite.config.js` y agrega la sección `server`. Así quedaría el archivo:

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
        hmr: {
            host: '30days.isw811.xyz',
            port: 5173,
        },
    },
});
```

Por último, edita el archivo `.env` para asegurarte de configurar el parámetro `APP_URL`.

```env
APP_URL=http://30days.isw811.xyz
```

En este punto, estarías listo para ejecutar Vite.

```bash
npm run dev
```

Deberías obtener una salida como la siguiente.

![Vite con --host](/images/30days/vite-with-host.png)

Desde un navegador en la máquina anfitriona, puedes ir a [http://30days.isw811.xyz:5173/](http://30days.isw811.xyz:5173/) para comprobar que Vite está en ejecución y es accesible desde la máquina anfitriona.

![Vite desde la máquina anfitriona](/images/30days/vite-on-5173.png)

Además, puedes visitar [http://30days.isw811.xyz/](http://30days.isw811.xyz/) (sin el puerto) para visualizar el sitio, tal y como lo has venido haciendo. Internamente, en la máquina virtual, el sitio sigue estando hospedado por Apache2, mediante el vhost _(Virtual Host)_ que se definió previamente.

![Sitio con Vite desde la máquina anfitriona](/images/30days/site-with-vite.png)
