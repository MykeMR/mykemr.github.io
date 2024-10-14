---
title: AnonymousPingu - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Facil      | ElPinguinoDeMario | 


# Reconocimiento 
Comenzamos el proceso de reconocimiento identificando los puertos abiertos en el sistema objetivo. Descubrimos que los puertos que se encuentran activos son el 21 (FTP) y el 80 (HTTP).
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

![image](https://github.com/user-attachments/assets/fbdcf75b-e25d-4d92-8797-3e558b2a92e6)

A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.

```bash
nmap -p 21,80 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/35d4dab8-0433-48d9-9943-52e4e2a36bfd)

## Exploración Web
Al acceder a la página web, observamos lo siguiente:

![image](https://github.com/user-attachments/assets/ede1f8d5-c313-4e69-94ee-aee4e813aa5e)

Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio. El siguiente comando fue ejecutado:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

![image](https://github.com/user-attachments/assets/7684d11b-a469-40f3-8254-b0d964ec0112)

Vemos que hay un directorio upload pero no hay nada dentro.

![image](https://github.com/user-attachments/assets/d2061512-4cf0-4b1a-90a6-0abaad99a259)

## FTP

Procedemos a explorar el servicio FTP, ya que hemos identificado anteriormente que el usuario `anonymous` está habilitado y que tenemos permisos de escritura en el directorio `upload`. 

**Nota:** Los usuarios `anonymous` no suelen tener contraseñas o son opcionales en muchos servidores FTP.

A continuación, intentamos conectarnos al servidor FTP utilizando el siguiente comando:
```bash
ftp 172.17.0.2
```

Al intentar acceder al servidor FTP con el usuario `anonymous` y sin contraseña, observamos que podemos entrar perfectamente. 

![image](https://github.com/user-attachments/assets/ea8254c0-879e-4ece-b542-3f19975c77a5)

Generamos la reverseshell utilizando **Pentest Monkey**, una herramienta popular para crear shells PHP que nos permiten obtener acceso remoto al servidor. La ruta de la reverseshell es la siguiente:

[Reverseshell de Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

### Pasos para Cargar la Reverseshell

1. **Preparar la Reverseshell**: Creamos un archivo de reverseshell en nuestro sistema local utilizando el código disponible en la ruta mencionada anteriormente.
2. **Subir el Archivo**: Utilizamos el comando de FTP para subir el archivo al directorio `upload`.
3. **Ejecutar la Reverseshell**: Una vez que el archivo esté en el servidor, lo ejecutaremos para establecer la conexión.

![image](https://github.com/user-attachments/assets/c6b9c6c5-e334-466c-ac7c-931e21464118)

Nos ponemos a escuchar por netcat en mi caso por el 444

```bash 
nc -lvnp 4444
```

Luego, accedemos a la ruta de la página web donde hemos cargado la reverseshell:

![image](https://github.com/user-attachments/assets/efce64bc-e77c-42cb-8b26-c651f630a409)

Y ya tendriamos acceso a la maquina.

![image](https://github.com/user-attachments/assets/bbc7b73c-09e9-4095-a526-e827fdead208)

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

## Intrusión en la Máquina

Ya estamos dentro de la máquina. Si ejecutamos el comando `sudo -l`, vemos que podemos ejecutar `man` como `Pingu`. Para ello, debemos hacer lo siguiente:

![Interfaz de la herramienta](https://github.com/user-attachments/assets/cef8486b-9c3c-4905-90b8-1dc9d1c54986)

Este tipo de herramienta tiene un apartado que, al final, dice *MORE*. Si en lugar de presionar para bajar, escribimos `!/bin/bash`, se generará una shell con ese usuario.
```shell
sudo -u pingu /usr/bin/man find
```

![image](https://github.com/user-attachments/assets/7300b460-b586-4311-b862-5bd5df506bd4)

Si ahora ponemos `!/bin/bash`:

![image](https://github.com/user-attachments/assets/eb9e8678-2261-4e21-96bc-9a49f19abafc)

Vemos que hemos subido de usuario y ya estamos con el usuario `pingu`.

![Cambio de usuario](https://github.com/user-attachments/assets/0c3da884-f0fb-45df-9692-59312d0752ab)

## Usuario pingu

Vamos a volver a ejecutar el comando `sudo -l`.

![Comando sudo](https://github.com/user-attachments/assets/0cedf58f-0919-4d19-a6eb-c37df62b1d4b)

## Usuario gladys
Observamos que podemos ejecutar `nmap` y `dpkg` como `gladys`. Para ello, debemos proceder de la siguiente manera:

En mi caso, utilizaré `dpkg` y aplicaré el mismo método que antes con `man`.

```bash
sudo -u gladys /usr/bin/dpkg -l
```
Si ahora ponemos `!/bin/bash`:

![image](https://github.com/user-attachments/assets/7f86e264-8d5c-4fb9-8302-0177b78e7dbf)

Vemos que hemos subido de usuario y ya estamos con el usuario `gladys`.

![image](https://github.com/user-attachments/assets/0e482934-f8cd-4cd1-bede-abf6bd888fa6)


## Usuario root
Vamos a volver a ejecutar el comando `sudo -l`.

![image](https://github.com/user-attachments/assets/fb5bbc2b-bd10-4091-bb4c-a9b168f0d487)

Observamos que podemos ejecutar `chown` como `root`. Para ello, debemos proceder de la siguiente manera:

Vamos a cambiar los permisos de `/etc/passwd` para poder tener permisos con nuestro usuario.
