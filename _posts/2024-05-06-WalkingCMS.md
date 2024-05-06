---
title: WalkingCMS - DockerLabs
published: true
categories: DockerLabs
---

| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  | Muy Fácil   | ElPinguinoDeMario | 

## Reconocimiento

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/fbaf571c-df38-48e6-9158-9fa9730923a2)

Vemos únicamente el puerto 80 abierto.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1dce5b1d-f639-4aa2-a825-f3f6e88e59a3)


Vamos a realizar un reconocimiento web.

Usamos Gobuster para hacer un reconocimiento web y ver que directorios podemos encontrar en dicho sitio.

gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php

    gobuster dir - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
    -u http://172.20.10.5/ Especifica la URL objetivo que deseas escanear en busca de directorios.
    -w Especifica que diccionario queremos usar
    -x Para indicar que tipo de extension queremos que nos encuentre


![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/b4dd3355-01a6-4707-b550-231608084bf4)

Hemos identificado que está instalado WordPress.

Vamos a usar WPScan para analizar la web.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/69ed5e03-5832-481a-967a-f2a922190249)


Encontramos el usuario "Mario". Realizaremos un ataque de fuerza bruta desde la misma herramienta.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a0701430-8f77-4914-a09b-74da966ba8d2)


Se encontró la contraseña "love". Intentaremos acceder al wp-admin.



¡Accedimos!

## Intrusión y Escalada de Privilegios

Ahora, en "Apariencia/Editor de código de tema", vamos a modificar cualquier página que tenga extensión PHP y le pegaremos una reverse shell PHP de las web shells.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1aec294c-5247-4c04-bbc2-1bfa6ed51471)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/5c5d0b34-1f7b-4dd8-a80b-52b818274b22)

Nos ponemos en escucha con NetCat, abrimos la página que hayamos editado y nos dará una reverse shell.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/88c12c9a-fa34-4c97-8b2c-f471e3faf925)

Si realizamos una búsqueda de permisos SUID, vemos lo siguiente:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/b8fa546b-53ed-4ba5-9977-8b040b717ce6)


Como tenemos la posibilidad de explotar el SUID de `env`, realizamos lo siguiente:

  `/usr/bin/env /bin/sh -p`

¡Obtendremos una shell en modo root!
