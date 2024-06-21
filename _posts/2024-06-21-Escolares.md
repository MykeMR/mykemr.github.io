---
title: Escolares - DockerLabs
published: true
categories: DockerLabs
---

| OS    | Dificultad | Creator    |
| ----- | ---------- | ---------- |
| Linux | Fácil      | Luisillo_o |

## Escaneo y Reconocimiento de puertos

Vamos a iniciar nuestro reconocimiento con un rápido escaneo de nmap.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/7b4dd24a-7fec-4081-8c76-b3bad573dbba)

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

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/8285fde8-3d14-44ca-b714-eb0879d1a685)

Si vamos a investigar el codigo fuente encontramos lo siguiente

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/607c279e-15ae-420c-8eb1-5607707056e5)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a01e424b-2bde-4f45-82df-d1c62eb76df2)

Encontramos diferentes usuario vamos anotarlos en un documento por si mas adelante nos sirven.


Usamos `Gobuster` para hacer un reconocimiento web y ver que directorios podemos encontrar en dicho sitio.

`gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php`
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.20.10.5/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/e6917c8d-6f9a-4acc-9d12-430349e0fcbc)

Encontramos dos repositorios interesantes un wordpress y un phpmyadmin

Vamos a ver el wp-admin

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ac58d7be-9ac7-422d-89f8-20bf75ef67bc)

Lo tenemos que agregar en el archivo /etc/hosts

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/7c61eb3c-2e65-4b64-9101-ba41b066a9cc)

Vemos que nos dice que el usuario juan no esta registrado asi que vamos a probar cual si es el correcto de los usuarios de los profesores.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/bb02c92a-3528-4f0a-9856-f75f79065f49)

Vemos que luisillo nos cambia el error por eso sabremos que este es el usuario vulnerable

Vamos hacer un ataque de fuerza bruta con este usuario, pero no encuentra la contraseña así que como tenemos bastante información del usuario podríamos crear un diccionario de contraseñas para este usuario.

Para ello vamos a usar el script del usuario Mebus de Github, pero también podríamos usar crunch entre otras.

https://github.com/Mebus/cupp.git


Una vez creado el diccionario vamos a usar wpscan para hacer un ataque de fuerza bruta con el usuario luisillo y el diccionario creado.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/bdf3fba0-6273-4fea-929b-265c5b62550b)

Nos encuentra la contraseña, vamos acceder al wordpress.


![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/81b79a43-4dd4-45ae-8a87-54aca7c13c85)

Vemos que tiene WP File Manager esto es un gestor de archivos donde podemos ver, eliminar y subir archivos al wordpress

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/6a00abb8-97d0-4f94-b1e2-b5e0602362e7)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/b376e91c-a019-4351-9de1-5c4ef23978fe)

Como podemos subir archivo vamos a subir un archivo php que nos mande una reverse shell a nuestro equipo, usaremos la de pentest monkey

La subimos y le damos click encima para que nos mande la reverse shell.

Es importante que en el terminal nos pongamos en escucha con NetCat.


![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/dbdc332c-d232-4e7b-8787-bc46177d198f)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/2946c493-1eb1-449b-89e6-24f4714e3594)

Si nos dirigimos al directorio Home encontramos un secret.txt

En su interior encontramos la contraseña del usuario luisillo.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/cad7ca6c-c560-460f-b14e-206c775c6c77)

Ahora que somos luisillo si hacemos un sudo -l vemos que podemos ejecutar el binario awk como cualquier usuario vamos a investigar este binario a  ver de que forma podemos escalar privilegios.

## Escalada de privilegios

En GTFobins encontramos la repuesta 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/5c557908-e7f8-45b1-8311-52ae9092b602)

Ya somos root
