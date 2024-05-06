---
title: Vacaciones - DockerLabs
published: true
categories: DockerLabs
---


## Video de YouTube
<iframe width="560" height="315" src="https://www.youtube.com/embed/Dkc39S9YteI" frameborder="0" allowfullscreen></iframe>



| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  | Muy Fácil   | Romabri | 

## Escaneo y Reconocimiento de puertos

Vamos a iniciar nuestro reconocimiento con un rápido escaneo de nmap.
![imagen](https://github.com/romabri/WriteUps/assets/51706860/1d5fc344-4aca-4a00-a985-f1d8eb5932a6)


`nmap -p- --open -sS -sC -sV --min-rate=5000 -vvv -n -Pn -oN escaneo 172.17.0.2`
- `-p-` - Busqueda de puertos abiertos
- `--open` - Enumera los puertos abiertos
- `-sS` - Es un modo de escaneo rapido
- `-sC-` - Que use un conjunto de scripts de reconocimiento
- `-sV` - Que encuentre la version del servicio abirto
- `--min-rate=5000` - Hace que el reconocimiento aun vaya mas rapido mandando no menos de 5000 paquetes
- `-n` - No hace resolución DNS
- `-Pn` - No hace ping
- `-vvv` - Muestra en pantalla a medida que encuentra puertos (Verbose)

Vemos que tiene el puerto 22 y el puerto 80 abierto

## Puerto 80

![imagen](https://github.com/romabri/WriteUps/assets/51706860/d819b9bf-b29d-4c70-901d-2e49022901f8)

La pagina web no muestra nada vamos a probar hacer fuzzing con Gobuster

Usamos `Gobuster` para hacer un reconocimiento web y ver que directorios podemos encontrar en dicho sitio.

`gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php`
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.20.10.5/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

Gobuster tampoco ha encontrado nada pero si inspeccionamos el codigo de la pagina nos encontramos algo

![imagen](https://github.com/romabri/WriteUps/assets/51706860/5ffd2f85-c447-4c29-a37c-4cb4fbc1b518)

Tenemos dos possibles usuarios `Juan` y `Camilo`

## Intrusión y Escalada de privilegios

Vamos hacer un ataque de fuerza burta al puerto SSH con `Medusa` de los dos usuarios para ver si encontramos las credenciales

`medusa -h 172.17.0.2 -u camilo,juan -P /usr/share/wordlists/rockyou.txt -M ssh`

Nos ha encontrado que camilo puede acceder via SSH con la contraseña password1

Vamos a ingresar

![imagen](https://github.com/romabri/WriteUps/assets/51706860/4b204202-b868-4016-95bd-d1ee324eb0ae)

Si hacemos busqueda de permisos SUID o SUDO no tenemos nada vamos a ver si hay un correo como decia el texto que hay en el codigo fuente de la pagina web.

nos dirigimos a `/var/mail/camilo`

Y nos encontramos el siguiente email

![imagen](https://github.com/romabri/WriteUps/assets/51706860/b1c0318a-2f61-4875-8cf0-87874e5bcd0d)

Tenemos una contraseña vamos a cambiar al usuario de juan

![imagen](https://github.com/romabri/WriteUps/assets/51706860/918afa8a-cdb0-4980-b7c6-67b8bc5f83c4)

Si ejecutamos sudo -l nos aparece que podemos usar Ruby como sudo 

Para escalar privilegios con este lo hacemos de la siguiente forma:

`ruby -e 'exec "/bin/sh"`

Ya hemos vulnerado la maquina!!!






