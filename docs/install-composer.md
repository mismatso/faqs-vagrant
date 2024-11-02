[↩︎ Volver al índice](/README.md)

# **Instalación de Composer en el _webserver_**

## 1. **Conectarse a la instancia de _webserver_**

Primero, nos dirigimos al directorio donde se alojan las máquinas _Vagrant_ y accedemos al directorio de la instancia del servidor web.

```bash
cd ~/VMs/webserver
```

Iniciamos la máquina virtual.

```bash
vagrant up
```

Luego, nos conectamos a la instancia del servidor web.

```bash
vagrant ssh
```

## 2. **Instalar Composer**

### 2.1 Descargar y ejecutar el instalador de Composer

Descargamos el script de instalación de Composer.

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
```

Ejecutamos el script para generar el binario de Composer.

```bash
php composer-setup.php
```

Eliminamos el archivo temporal _composer-setup.php_.

```bash
php -r "unlink('composer-setup.php');"
```

### 2.2 Mover el binario de Composer y configurar el acceso global

Creamos una carpeta en _/opt_ para almacenar Composer.

```bash
sudo mkdir /opt/composer
```

Movemos el binario de Composer a la carpeta _/opt/composer_.

```bash
sudo mv composer.phar /opt/composer/
```

Creamos un enlace simbólico en _/usr/local/bin_ para que Composer sea accesible desde cualquier ubicación.

```bash
sudo ln -s /opt/composer/composer.phar /usr/local/bin/composer
```

## 3. **Comprobar la instalación de Composer**

Para verificar que Composer se haya instalado correctamente, ejecutamos el siguiente comando desde cualquier ubicación en la máquina virtual:

```bash
composer --version
```

Deberíamos obtener una salida similar a la siguiente:

> _Composer version 2.7.9 2024-09-04 14:43:28_<br>
> _PHP version 8.2.24 (/usr/bin/php8.2)_<br>
> _Run the "diagnose" command to get more detailed diagnostics output._