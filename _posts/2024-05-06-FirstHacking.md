---
title: FirstHacking - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  | Muy Fácil   | ElPinguinoDeMario | 

## Escaneo y Reconocimiento de puertos

Vamos a iniciar nuestro reconocimiento con un rápido escaneo de nmap.
![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a1b2aabd-a85e-4403-b43e-d9e19a74cca4)


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

Vemos que tiene el puerto 21 FTP

## Puerto 21 
Observamos que el puerto FTP está abierto con una versión específica. Ahora procederemos a buscar exploits que puedan aprovechar vulnerabilidades en esa versión

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/91154c4b-a68f-46a7-8baf-dc581aea5e78)

Al usar Searchsploit, encontramos un exploit. Ahora, vamos a investigar qué hace este exploit llamado 'Backdoor Command Execution'.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/49e59760-1abb-4f3b-ba8b-149337f6b2e1)


## Intrusión y Escalada de privilegios

Solicita solo el host, entonces procederemos con eso.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/3dae2849-dae0-4d1a-a25b-c0f894d1def3)

Ya hemos vulnerado la maquina!!!

