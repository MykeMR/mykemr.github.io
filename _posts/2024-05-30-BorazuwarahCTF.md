---
title: BorazuwarahCTF - DockerLabs
published: true
categories: DockerLabs
---


| OS    | Dificultad | Creator     |
| ----- | ---------- | ----------- |
| Linux | Muy Facil  | Borazuwarah |


## Escaneo y Reconocimiento de puertos

Vamos a iniciar nuestro reconocimiento con un rápido escaneo de nmap.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/60057ccd-3ae1-4488-b435-80e57be12546)


`nmap -p- --open -sS -sC -sV --min-rate=5000 -vvv -n -Pn -oN escaneo 172.17.0.2`
-  `-p-`: Escanea todos los puertos (1-65535).
- `--open`: Solo muestra los puertos abiertos.
- `-sS`: Realiza un escaneo SYN, que es rápido y discreto.
- `-sC`: Utiliza los scripts de reconocimiento por defecto de Nmap.
- `-sV`: Detecta la versión del servicio en los puertos abiertos.
- `--min-rate=5000`: Envía al menos 5000 paquetes por segundo, acelerando el escaneo.
- `-n`: No realiza resolución DNS.
- `-Pn`: No realiza ping previo para determinar si el host está activo.
- `-vvv`: Muestra resultados detallados y en tiempo real.

Vemos que esta abierto el puerto 22 `SSH` y el puerto 80 `HTTP`

## Enumeración Web
Si accedemos a la pagina web vemos lo siguiente 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ec6bf7ad-1d08-4f06-a2dc-5521566e7d0c)


Usamos `Gobuster` para hacer un reconocimiento web y ver que directorios podemos encontrar en dicho sitio.

`gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php`
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.20.10.5/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

Gobuster no ha encontrado nada

Vamos a descargar la imagen del huevo kinder

## Esteganografía

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/7a034b01-2ab4-4c9d-8391-631f7de46e66)

Y vemos este mensaje

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/fe660f0d-d28f-4a05-aa4e-b8629e53680d)

Si usamos otra herramienta de esteganografía como exiftool vemos lo siguiente.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/6675ef5f-2e4c-4687-bbb2-ad479e995c1e)

Ya tenemos un usuario ahora vamos hacer un ataque con fuerza bruta al SSH.

## Ataque de fuerza bruta con Hydra

Como tenemos un possible usuario, vamos hacer un ataque de fuerza bruta con Hydra al protocolo ssh de la siguiente forma:

`hydra -l user -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2`
- `-l` - Para especificar el usaurio
- `-P` - Para especificar que diccionario de fuerza bruta usaremos
- `ssh://172.17.0.2` - Especificamos que queremos atentar contra el protocolo SSH de la IP victima

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/4c963eaf-07ae-4dba-9c3f-2cd72c9351a3)


Como vemos, existe el usuario `borazuwarah` y la contraseña es `123456`.

## Intrusión

Ahora accedemos vía SSH 

`ssh borazuwarah@172.17.0.2`

En la maquina no encontramos nada interesante así que vamos a escalar privilegios
## Escalada de privilegios


Si hacemos un `sudo -l` nos aparece lo siguiente

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a0b30b01-1bd7-4eb6-97f1-01463af89d12)

Ejecutamos el siguiente comando `sudo -u root /bin/bash`

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c9ac9885-30c0-4eca-9c26-d9e3eaa559c7)

Ya somos ROOT!
