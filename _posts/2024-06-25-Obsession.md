---
title: Obsession - DockerLabs
published: true
categories: DockerLabs
---

| OS    | Dificultad | Creator     |
| ----- | ---------- | ----------- |
| Linux | Muy Fácil  | Juan Carlos |

## Escaneo y Reconocimiento de puertos

Vamos a iniciar nuestro reconocimiento con un rápido escaneo de nmap.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/90d27320-f151-42e1-834e-6ad76bfee791)

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

Vemos que esta abierto el puerto 21 `FTP` , 22 `SSH` y el puerto 80 `HTTP`

## FTP

Vamos a conectarnos via ftp con el siguiente comando

```
ftp 172.17.0.2
```

En usuario ponemos anonymous ya que en el reporte de nmap nos pone que tenemos disponible la conexion anonima y en contraseña no ponemos nada.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/61959215-eb88-4756-a16e-9de4628cd3e6)

Tenemos dos archivos, vamos a descargarlos con el comando GET

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0aa4fac5-b53a-418f-8e2c-442839b43c82)

Estos son los dos archivos que tenemos txt 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/85ffe338-aa32-408e-aa62-24a5ee6f7c5a)

De aqui tenemos poca información como mucho 3 posibles usuarios Gonza, Russoski y Nagore pero como no podemos hacer enumeración de usuarios ya que el SSH esta actualizado vamos a ver la pagina web.

## Enumeración Web
Si accedemos a la pagina web vemos lo siguiente 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ff446025-0906-4278-9b90-4026fd31d3c4)

Usamos `Gobuster` para hacer un reconocimiento web y ver que directorios podemos encontrar en dicho sitio.

`gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php`
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.20.10.5/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

Gobuster no ha encontrado nada

Pero en una de las entradas vemos algo interesante

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a74c4f13-81bc-463e-b153-59ebfea81cbe)

Vamos al directorio backup y nos encontramos con otro txt que dice lo siguente:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/63be6dcd-1798-4243-870f-f0b84859f844)

Y tambien vamos al important y encontramos un important.md.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/f23be760-e42e-408d-9207-aab93ddc4ef4)

Tenemos un usuario vamos hacer fuerza bruta

## Ataque de fuerza bruta con Hydra

Como tenemos un possible usuario, vamos hacer un ataque de fuerza bruta con Hydra al protocolo ssh de la siguiente forma:

`hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2`
- `-l` - Para especificar el usaurio
- `-P` - Para especificar que diccionario de fuerza bruta usaremos
- `ssh://172.17.0.2` - Especificamos que queremos atentar contra el protocolo SSH de la IP victima

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/fa75864b-7244-435b-8253-9b06f2589642)

Como vemos, existe el usuario `russoski` y la contraseña es `iloveme`.

## Intrusión

Ahora accedemos vía SSH 

`ssh russoski@172.17.0.2`

Si hacemos un sudo -l encontramos que podemos ejecutar el vim como ROOT entonces vamos a buscar en GTFOBINS como hacerlo 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0a239044-bfe5-4897-9daf-b55e72d16508)

Para escalar privilegios hacemos lo siguiente:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/da3eb433-c886-4bf0-8aff-7ca88891f3dc)

Ya somos ROOT
