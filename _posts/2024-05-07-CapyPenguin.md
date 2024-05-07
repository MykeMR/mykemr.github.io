---
title: CapyPenguin - DockerLabs
published: true
categories: DockerLabs
---

 
| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  | Fácil       | ElPinguinoDeMario | 


## Escaneo y Reconocimiento de puertos

Vamos a iniciar nuestro reconocimiento con un rápido escaneo de nmap.

![image](https://github.com/romabri/romabri.github.io/assets/51706860/b3fb40f7-4311-492a-b098-ad9f583ddc5e)


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

Vemos que esta abierto el puerto 22 `SSH`, el puerto 80 `HTTP` y el 3306 `mysql`

## Enumeración Web

![image](https://github.com/romabri/romabri.github.io/assets/51706860/40c67357-8381-444d-ada8-ba1580b65c92)


Usamos `Gobuster` para hacer un reconocimiento web y ver que directorios podemos encontrar en dicho sitio.

`gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php`
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.20.10.5/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

![image](https://github.com/romabri/romabri.github.io/assets/51706860/05fddaae-1c42-47a3-a8d3-9c1c9eb13487)


Vemos que nos encuentra un directorio llamado index.html. Si accedemos a él, vemos lo siguiente:

![image](https://github.com/romabri/romabri.github.io/assets/51706860/6edfc822-0829-4802-8ff8-345f70217062)


Es una imagen muy bonita pero con muy poca información...

Vamos a ver el codigo fuente.

![image](https://github.com/romabri/romabri.github.io/assets/51706860/b82a502d-2d87-4789-9eef-bc4a18bc951d)

Encontramos un posible usuario llamado capybarauser y descubrimos que la contraseña se encuentra al final del archivo rockyou.txt. Además, se nos muestra con qué comando podemos invertir el orden del archivo, el comando TAC.

Entonces, copiamos el archivo rockyou donde queramos y ejecutamos el siguiente comando:

`tac rockyou.txt > rockyouINV.txt`


## Ataque de fuerza bruta con Medusa

Esto genera el archivo rockyou invertido, comenzando desde el final.

Ahora utilizaremos Medusa para realizar un ataque de fuerza bruta contra el servicio MySQL:

`medusa -h 172.17.0.2 -u capybarauser -P rockyouinv.txt -M mysql`
Este comando ejecutará Medusa con la IP de destino, el usuario capybarauser, el diccionario de contraseñas rockyou invertido y el servicio MySQL como objetivo del ataque.

`-h: Especifica la dirección IP del objetivo al que se realizará el ataque.`
`-u: Especifica el nombre de usuario que se probará para la autenticación.`
`-P: Especifica el archivo que contiene las contraseñas que se utilizarán en el ataque de fuerza bruta.`
`-M: Especifica el tipo de servicio al que se realizará el ataque. En este caso, se utiliza "mysql" para indicar que se atacará al servicio MySQL.`

![image](https://github.com/romabri/romabri.github.io/assets/51706860/14f9ebd8-ddcb-4ffa-a630-e6421be72239)

Nos encuentra la contraseña `ie168`

Vamos a conectarnos al mysql

`mysql -h 172.17.0.2 -u capybarauser -pie168`

## MYSQL

Vamos a conectarnos al mysql

`mysql -h 172.17.0.2 -u capybarauser -pie168`

![image](https://github.com/romabri/romabri.github.io/assets/51706860/3861d576-1615-4055-bd69-0afcee9fad4f)

![image](https://github.com/romabri/romabri.github.io/assets/51706860/5937b872-76cd-4388-a94d-85678bfd6d42)

Encontramos el usuario mario y la contraseña que puede ser del servidor ssh

## SSH

Accedemos al SSH

`ssh mario@172.17.0.2`

![image](https://github.com/romabri/romabri.github.io/assets/51706860/9837fa0b-fa53-402a-b6e2-58c9b3f43247)

Si hacemos un sudo -l aparece que tenemos permios de sudo sobre el archivo nano

![image](https://github.com/romabri/romabri.github.io/assets/51706860/278d49f8-d43d-48d6-b8b7-835382825020)

Vamos a explotarlo para poder escalar privilegios

![image](https://github.com/romabri/romabri.github.io/assets/51706860/16ef6906-0cec-41d9-a697-6127bbb3dd1f)

![image](https://github.com/romabri/romabri.github.io/assets/51706860/bc701371-82f6-451d-83c0-6af97bce926c)

Ya somos ROOT














