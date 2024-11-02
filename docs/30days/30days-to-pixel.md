[↩︎ Volver al índice](/README.md)

# ¿Cómo migrar de «30 Days» a «Pixel positions»?

## 1. Desde la máquina «database»

### Crear una nueva base de datos

Conéctese a MySQL como superadministrador:

```bash
sudo mysql
```

Cree la nueva base de datos:

```sql
CREATE DATABASE pixel;
GRANT ALL PRIVILEGES ON pixel.* TO laravel;
FLUSH PRIVILEGES;
QUIT;
```

**Nota**: En este ejemplo se asume que el usuario `laravel` ya existe, si no es así, puedes crearlo con el comando `create user laravel identified by 'secret'`. Nunca utilices contraseñas débiles como `secret` en entornos productivos.

## 2. Desde la máquina «webserver»

### Crear el nuevo proyecto

Navegue al directorio de proyectos:

```bash
cd /vagrant/sites
```

Realice una copia de seguridad de _«30days.isw811.xyz»_:

```bash
tar --exclude='30days.isw811.xyz/vendor' -czvf 30days.isw811-backup.xyz.tar.gz 30days.isw811.xyz
```

Inicie el nuevo proyecto:

```bash
laravel new pixel.isw811.xyz
```

Copie el repositorio de _«30days»_ a _«pixel»_:

```bash
cp -r 30days.isw811.xyz/.git pixel.isw811.xyz
```

Ingrese al proyecto _«pixel»_:

```bash
cd pixel.isw811.xyz
```

Realice una confirmación temporal (_Work in Progress_, o WIP) para guardar el progreso actual:

```bash
git status
git add .
git commit -m "Episodio 27 - From Design to Blade (WIP)"
```

### Configurar un vhost para «pixel.isw811.xyz»

Cambie al directorio de configuración (desde la máquina virtual):

```bash
cd /vagrant/confs
```

Cree una nueva configuración a partir del archivo existente:

```bash
cp 30days.isw811.xyz.conf pixel.isw811.xyz.conf
sed -i -e 's/30days/pixel/g' pixel.isw811.xyz.conf
```

Despliegue el nuevo archivo de configuración:

```bash
sudo cp /vagrant/confs/pixel.isw811.xyz.conf /etc/apache2/sites-available
sudo a2ensite pixel.isw811.xyz.conf
sudo apache2ctl -t
sudo systemctl reload apache2
```

### Configurar la base de datos en .env

Navegue al directorio del nuevo sitio (permanezca en la máquina virtual del servidor web):

```bash
cd /vagrant/sites/pixel.isw811.xyz
```

Configure la conexión a la base de datos mediante el siguiente bloque de comandos:

```bash
sed -i 's/^DB_CONNECTION=.*/DB_CONNECTION=mysql/' .env && \
sed -i 's/^DB_HOST=.*/DB_HOST=192.168.56.11/' .env && \
sed -i 's/^DB_PORT=.*/DB_PORT=3306/' .env && \
sed -i 's/^DB_DATABASE=.*/DB_DATABASE=pixel/' .env && \
sed -i 's/^DB_USERNAME=.*/DB_USERNAME=laravel/' .env && \
sed -i 's/^DB_PASSWORD=.*/DB_PASSWORD=secret/' .env
```

Ejecute las migraciones:

```bash
php artisan migrate
```

## 3. Desde la máquina anfitriona

### Agregar entradas al archivo _hosts_ en la máquina anfitriona

Para simular la resolución del dominio en nuestro servidor, agregue las siguientes entradas en el archivo _hosts_ de la máquina anfitriona.

#### Si el anfitrión es Windows:
El archivo _hosts_ está ubicado en `C:\Windows\System32\drivers\etc`. Para editarlo, ejecute:

```dos
cd \Windows\System32\drivers\etc
notepad hosts
```

#### Si el anfitrión es Linux/Unix:
El archivo _hosts_ está ubicado en `/etc/hosts`. Para editarlo, ejecute:

```bash
sudo nano /etc/hosts
```

Agregue la entrada correspondiente al dominio que desea simular:

```bash
192.168.56.10 pixel.isw811.xyz
```

Ahora puede visualizar el nuevo sitio en [http://pixel.isw811.xyz](http://pixel.isw811.xyz).

### Continuar con los _commits_

Ahora está listo para continuar con el proyecto final del curso «30 days to learn Laravel». Una vez que complete el «Episodio 27 - From Design to Blade», puede rectificar el _commit_ que realizó al inicio.

```bash
git status
git add .
git commit --amend -m "Episodio 27 - From Design to Blade"
```