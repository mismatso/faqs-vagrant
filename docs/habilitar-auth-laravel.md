[⬅ Volver al índice](/Readme.md)

# **Habilitar autenticación con Laravel/UI**

## 1. Instalar Node Version Manager (NVM)

Para utilizar diferentes versiones de NodeJS en nuestro servidor web, lo más conveniente es instalar primero un gestor de versiones de Node, como _NVM-Sh_.

```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

Recargamos el script de arranque de _bash_ para que los cambios surtan efecto.

```bash
source ~/.bashrc
```

Ahora instalamos NodeJS. Para trabajar en un proyecto de Laravel versión 8.6.12, debemos utilizar una versión de NodeJS cercana a la fecha de lanzamiento de Laravel 8.6.12. En este caso, corresponde a la versión LTS «Erbium».

```bash
nvm install --lts=erbium
```

Si en cualquier momento deseamos cambiar la versión _default_, simplemente ejecutamos el siguiente comando:

```bash
nvm alias default lts/erbium
```

## 2. Instalar Laravel/UI

Ahora instalamos _laravel/ui_. Para trabajar con Laravel 8.6.12, instalamos una versión de _laravel/ui_ cercana a la fecha de lanzamiento, que es la 3.4.6.

```bash
composer require laravel/ui:3.4.6
```

Integramos _Bootstrap_ con Laravel, generando las vistas y archivos necesarios para trabajar con este framework CSS.

```bash
php artisan ui bootstrap
```

Instalamos las dependencias de NodeJS necesarias para Laravel UI.

```bash
npm install && npm run dev
```

## 3. Posibles errores

### 3.1. Memoria RAM insuficiente

Si la instalación de las dependencias de _NodeJS_ falla, es probable que el equipo no tenga suficiente memoria RAM. Para aumentar la memoria, editamos el _Vagrantfile_ del _webserver_. Descomentamos las líneas 60 a 66, y comentamos la línea 62. La configuración quedaría así:

```bash
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    # vb.gui = true
  
    # Customize the amount of memory on the VM:
    vb.memory = "2048"
  end
```

Para aplicar los cambios, apagamos y volvemos a iniciar la máquina virtual _webserver_.

```bash
vagrant halt
vagrant up
```

### 3.2. Usuario sin permisos para crear _symlinks_ en SO anfitrión

Otro error posible durante la ejecución de `npm install` es la imposibilidad de crear enlaces simbólicos (_symlinks_).

![Error de symlink](/images/symlink/00-symlink-error.png)

Si utiliza **Windows Home Edition**, puede seguir las instrucciones indicadas en este [guion](https://github.com/mismatso/faqs-vagrant/blob/master/symlink-error.md) para habilitar los _symlinks_.

Luego de corregir los errores anteriores, intentamos nuevamente la instalación de las dependencias de NodeJS.

Nos desplazamos al directorio del proyecto Laravel.

```bash
cd /vagrant/sites/lfts.isw811.xyz/
```

Eliminamos los restos de la instalación fallida.

```bash
rm -rf node_modules
```

Reintentamos la instalación de las dependencias de NodeJS.

```bash
npm install && npm run dev
```

Para habilitar las vistas de autenticación, ejecutamos el siguiente comando:

```bash
php artisan ui bootstrap --auth
```

Y nuevamente instalamos las dependencias de NodeJS.

```bash
npm install && npm run dev
```

## 4. Conectar Laravel con la base de datos

Para este paso, debemos editar el archivo `.env` ubicado en la raíz de nuestro proyecto _Laravel_.

```bash
DB_CONNECTION=mysql
DB_HOST=192.168.56.11
DB_PORT=3306
DB_DATABASE=lfts
DB_USERNAME=laravel
DB_PASSWORD=secret
```

Nos conectamos nuevamente al _webserver_, nos desplazamos hasta el directorio del sitio Laravel y ejecutamos el siguiente comando para ejecutar las migraciones de la base de datos.

```bash
php artisan migrate
```

En este punto, puede abrir su navegador y visitar el sitio. En la esquina superior derecha verá las nuevas opciones de _Login_ y _Register_. Intente registrar un usuario para verificar que la aplicación está correctamente conectada a la base de datos.

![Sitio LFTS con enlace de Login y Register](/images/webserver/site-lfts-with-auth.png)

Si hacemos clic en _«Register»_ podremos registrar un nuevo usuario.

![Registrando un nuevo usuario](/images/database-server/register-john.png)

Posteriormente podremos hacer «Login».

![Registrando un nuevo usuario](/images/database-server/login-john.png)

Para finalmente llegar al _dashboard_ de usuario autenticado.

![Registrando un nuevo usuario](/images/database-server/authenticated-john.png)