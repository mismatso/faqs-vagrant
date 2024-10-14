[⬅ Volver al índice](/Readme.md)

# Creación de instancia de servidor web con Vagrant/VirtualBox

## 1. Aprovisionar nueva instancia de Debian Bookworm

Primero, ingresamos al directorio donde se alojarán las máquinas _Vagrant_.

```bash
cd ~/VMs
```

Creamos el directorio para la nueva máquina _Vagrant_, y también las carpetas donde se almacenarán los sitios web y los archivos de configuración de los _Virtual Hosts_ de Apache2.

```bash
mkdir -p webserver/sites
mkdir -p webserver/confs
```

Ingresamos a la carpeta creada.

```bash
cd webserver
```

Creamos un nuevo _Vagrantfile_ para la nueva instancia de servidor web.

```bash
vagrant init debian/bookworm64
```

Obtendremos la siguiente salida:

> _A `Vagrantfile` has been placed in this directory. You are now_
> _ready to `vagrant up` your first virtual environment! Please read_
> _the comments in the Vagrantfile as well as documentation on_
> _`vagrantup.com` for more information on using Vagrant._

Editamos el _Vagrantfile_ para definir la IP del servidor. Descomentamos la línea 35 y modificamos la IP.

```bash
config.vm.network "private_network", ip: "192.168.56.10"
```

Luego, agregamos la siguiente directiva para montar la subcarpeta _«sites»_ en la ruta `/home/vagrant/sites`, estableciendo a _www-data_ como propietario y grupo. Esto le permitirá a Apache2 tener control total sobre los archivos del sitio web.

```bash
config.vm.synced_folder "sites/", "/home/vagrant/sites", owner: "www-data", group: "www-data"
```

También descomentamos las líneas que van de la 59 a la 65 para asignar 2 GB de RAM al servidor web, y comentamos la línea 61. La configuración quedaría así:

```bash
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    # vb.gui = true
  
    # Customize the amount of memory on the VM:
    vb.memory = "2048"
  end
```

Guardamos los cambios en el _Vagrantfile_ y luego iniciamos la máquina virtual.

```bash
vagrant up
```

## 2. Cambiar el nombre de host de la nueva máquina

Nos conectamos a la nueva instancia de servidor web.

```bash
vagrant ssh
```

El nombre de host predeterminado será _bookworm_, pero lo cambiaremos para evitar confusiones.

```bash
sudo hostnamectl set-hostname webserver
```

Actualizamos el archivo _hosts_ de la máquina virtual para reflejar este cambio.

```bash
sudo nano /etc/hosts
```

Luego de reemplazar _bookworm_ por el nuevo nombre de _host_, en este caso _webserver_, presionamos `[Ctrl]+O` seguido de `[Enter]` para guardar, y luego `[Ctrl]+X` para salir.

![Editar archivo hosts](/images/webserver/rename-host.png)

Para aplicar el cambio, salimos de la máquina virtual y volvemos a conectarnos.

```bash
exit
vagrant ssh
```

## 3. Instalar los paquetes del servidor web

Una vez conectados a la máquina virtual (debería mostrar `vagrant@webserver:~$`), actualizamos la lista de paquetes.

```bash
sudo apt-get update
```

Procedemos a instalar Apache, el cliente de MariaDB y las extensiones de PHP necesarias para el servidor.

```bash
sudo apt-get install vim vim-nox curl apache2 mariadb-client php8.2 php8.2-curl php8.2-bcmath php8.2-mysql php8.2-mcrypt php8.2-xml php8.2-zip php8.2-mbstring
```

Para verificar el correcto funcionamiento de Apache, podemos visitar [http://192.168.56.10](http://192.168.56.10) desde el navegador de la máquina anfitriona.

![Sitio por defecto de Apache2](/images/webserver/default-site.png)

## 4. Agregar entradas al archivo _hosts_ en la máquina anfitriona

Para simular la resolución de un dominio en nuestro servidor, agregamos entradas en el archivo _hosts_ de la máquina anfitriona.

### Si el anfitrión es Windows:
El archivo _hosts_ se encuentra en `C:\Windows\System32\drivers\etc`. Para editarlo:

```dos
cd \Windows\System32\drivers\etc
notepad hosts
```

### Si el anfitrión es Linux/Unix:
El archivo _hosts_ se encuentra en `/etc/hosts`. Para editarlo:

```bash
sudo nano /etc/hosts
```

Agregamos la entrada correspondiente al dominio que deseamos simular, en este caso _lospatitos.com_.

```bash
192.168.56.10 lospatitos.com
```

## 5. Crear el sitio de prueba _lospatitos.com_

Nos dirigimos a la carpeta _sites_ y creamos una nueva carpeta para el sitio _lospatitos.com_.

```bash
cd sites
mkdir lospatitos.com
cd lospatitos.com
```

Creamos la estructura de directorios para el sitio.

```bash
mkdir assets images
touch assets/styles.css index.html
```

Descargamos una imagen de prueba para el sitio.

```bash
curl -o ./images/lospatitos.webp https://coding-rs.s3.amazonaws.com/images/lospatitos/lospatitos.webp
```

Editamos el archivo _index.html_.

```bash
code index.html
```

Este archivo debería contener lo siguiente:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Los Patitos</title>
    <link rel="stylesheet" href="assets/styles.css">
</head>
<body>
    <header>
        <h1>Bienvenidos a Los Patitos</h1>
    </header>
    <main>
        <section>
            <img src="images/lospatitos.webp" alt="Patitos" id="patitos-image">
            <p>Este es un sitio de prueba creado para el curso básico de programación web con software libre.</p>
        </section>
    </main>
    <footer>
        <p>&copy; 2024 Los Patitos. Todos los derechos reservados.</p>
    </footer>
</body>
</html>
```

Editamos el archivo _styles.css_.

```bash
code assets/styles.css
```

Este archivo debería contener lo siguiente:

```css
body {
    font-family: Arial, sans-serif;
    text-align: center;
    background-color: #f0f8ff;
}

header {
    background-color: #ffdd59;
    padding: 20px;
}

h1 {
    color: #333;
}

main {
    margin: 20px auto;
    max-width: 800px;
}

img {
    max-width: 100%;
    height: auto;
}

footer {
    background-color: #333;
    color: #fff;
    padding: 10px;
}
```

## 6. Desplegar el sitio _lospatitos.com_ como un _Virtual Host_ de Apache

### 6.1. Crear el archivo de configuración del _Virtual Host_

Desde la carpeta _confs_, creamos un archivo de configuración para el sitio _lospatitos.com_.

```bash
cd ../confs
touch lospatitos.com.conf
```

Editamos el archivo de configuración:

```bash
code lospatitos.com.conf
```

Añadimos lo siguiente:

```apache
<VirtualHost *:80>
ServerAdmin webmaster@lospatitos.com
ServerName lospatitos.com

DirectoryIndex index.php index.html
DocumentRoot /home/vagrant/sites/lospatitos.com

<Directory /home/vagrant/sites/lospatitos.com>
    AllowOverride All
    Require all granted
</Directory>

ErrorLog ${APACHE_LOG_DIR}/lospatitos.com.error.log
CustomLog ${APACHE_LOG_DIR}/lospatitos.com.access.log combined
</VirtualHost>
```

### 6.2. Desplegar el nuevo _Virtual Host_

Nos conectamos al servidor web y habilitamos el soporte para _Virtual Hosts_.

```bash
sudo a2enmod vhost_alias
sudo systemctl restart apache2
```

Copiamos el archivo de configuración al directorio de sitios disponibles.

```bash
sudo cp /vagrant/confs/lospatitos.com.conf /etc/apache2/sites-available/
```

Habilitamos el nuevo sitio.

```bash
sudo a2ensite lospatitos.com.conf
```

Verificamos la configuración de Apache.

```bash
sudo apache2ctl -t
```

Si aparece el error _Could not reliably determine the server's fully qualified domain name_, agregamos la directiva _ServerName_.

```bash
echo "ServerName webserver" | sudo tee -a /etc/apache2/apache2.conf
```

Si no hay errores, reiniciamos Apache.

```bash
sudo systemctl restart apache2
```

Ahora, visitamos [http://lospatitos.com](http://lospatitos.com) desde la máquina anfitriona para ver el sitio desplegado.

![Vista preliminar de LosPatitos.com](/images/webserver/site-lospatitos.png)