[⬅ Volver al índice](/Readme.md)

# **Instalación del Composer en el webserver**

## 1. **Conectarnos a la instancia de _webserver_**

Primero, ingresamos al directorio donde se alojan las máquinas _Vagrant_ y nos dirigimos al folder de la instancia de _webserver_.

```bash
cd ~/VMs/webserver
```

Iniciamos la máquina virtual.

```bash
vagrant up
```

Nos conectamos a la instancia de servidor web.

```bash
vagrant ssh
```

## 2. **Instalar _Composer_**

1. Obtener el binario de composer.

    Descargamos el script de _composer_ que nos permitirá generar el binario.

    ```bash
    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    ```

    Generamos el binario.

    ```bash
    php composer-setup.php
    ```

    Eliminamos el archivo temporal _composer-setup.php_.

    ```bash
    php -r "unlink('composer-setup.php');"
    ```

2. Instalar _composer_ para que se accesible desde cualquier ubicación.

    Creamos una carpeta en _/opt_.

    ```bash
    sudo mkdir /opt/composer
    ```

    Movemos el binario de _composer_ a  _/opt/composer_.

    ```bash
    sudo mv composer.phar /opt/composer/
    ```

    Crear el enlace simbólico _composer_ en _/usr/local/bin/composer_.

    ```bash
    sudo ln -s /opt/composer/composer.phar /usr/local/bin/composer
    ```

## 3. *Comprobar que tenemos Composer debidamente instalado**

Desde cualquier ubicación de la máquina virtual _webserver_ lanzamos el siguiente comando.

```bash
composer --version
```

Deberíamos obtener una salida similar a la siguiente.

> _Composer version 2.7.9 2024-09-04 14:43:28_<br>
> _PHP version 8.2.24 (/usr/bin/php8.2)_<br>
> _Run the "diagnose" command to get more detailed diagnostics output._