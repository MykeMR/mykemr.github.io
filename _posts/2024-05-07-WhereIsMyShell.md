---
title: WhereIsMyShell - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | ElPinguinoDeMario | 

## Escaneo y Reconocimiento de puertos

Vamos a iniciar nuestro reconocimiento con un rápido escaneo de nmap.

![image](https://github.com/romabri/romabri.github.io/assets/51706860/1274f7a2-8289-494d-80b6-edd77c3fd1cc)


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
Si accedemos a la pagina web vemos lo siguiente:

![image](https://github.com/romabri/romabri.github.io/assets/51706860/8a1fe415-49c2-421d-9d79-0897d4a69525)


Usamos `Gobuster` para hacer un reconocimiento web y ver que directorios podemos encontrar en dicho sitio.

`gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php`
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.20.10.5/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

Encontramos un warning y un shell.php


![image](https://github.com/romabri/romabri.github.io/assets/51706860/0ad829cf-f609-4ba4-8e68-1e2f4cf800ba)


En el warning.html nos dice lo siguiente

![image](https://github.com/romabri/romabri.github.io/assets/51706860/bedaaded-d1dd-4253-9720-e8a7b71518c6)

Y en el shell.php no muestra nada, entonces con Wffuzz vamos a intentar ver que tipo de parametro es el que falta ya que normalmente para la ejecucion de comandos desde la URL se ve tal que asi "http://172.17.0.2/shell.php?[parametro]=id"
Lo que nos falta saber es cual es ese parametro asi que vamos a buscarlo de la siguiente forma:

![image](https://github.com/romabri/romabri.github.io/assets/51706860/d5b9aa55-c9ef-44ab-ba5c-9e68653bcae9)

Vemos que el parametro es `parameter` asi que vamos hacer la prueba

![image](https://github.com/romabri/romabri.github.io/assets/51706860/99d2e58c-b414-4abf-b7af-3cd6d23a0688)

Vemos que efectivamente funciona.

Ahora vamos a enviarnos una reverse shell para conseguir acceso al equipo 

![image](https://github.com/romabri/romabri.github.io/assets/51706860/df32cd17-a5ef-45a3-a469-2e4c8aa476b2)

Hemos de remplazar los '&' por '%26' ya que al ser una URL el simbolo '&' si lo URL encodeamos es el %26 si no no funcionara la reverse shell

![image](https://github.com/romabri/romabri.github.io/assets/51706860/afcbfe1f-e1f3-4160-8ca7-943e0ddcd1b8)

Vamos a ver como podemos escalar privilegios 

En la pagina web nos decía que en el directorio /tmp hay algo y si no s dirigimos a ese directorio vemos una archivo oculto con la contraseña de root 

![image](https://github.com/romabri/romabri.github.io/assets/51706860/258a7b69-8403-41d6-b649-d062ceaf9083)

Si la introducimos ya somos ROOT

![image](https://github.com/romabri/romabri.github.io/assets/51706860/ad775929-94c9-4c6e-a362-ecb6a34517f8)









