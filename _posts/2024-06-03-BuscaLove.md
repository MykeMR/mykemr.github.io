---
title: BuscaLove - DockerLabs
published: true
categories: DockerLabs
---

| OS    | Dificultad | Creator  |
| ----- | ---------- | -------- |
| Linux | Fácil      | Prendelo |

## Escaneo y Reconocimiento de puertos

Vamos a iniciar nuestro reconocimiento con un rápido escaneo de nmap.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/73fab817-5001-4d12-b048-1ee5f39a5324)


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

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/44a5184a-a0c0-46a3-8c29-05b5961b701a)


Usamos `Gobuster` para hacer un reconocimiento web y ver que directorios podemos encontrar en dicho sitio.

`gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php`
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.20.10.5/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

Gobuster ha encontrado un /wordpress

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/3aa34da4-8ad1-49ae-bb73-1aac7053e97c)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/4fe7d7f3-d59e-4bc6-bb11-ef33a0287475)


Si miramos el código fuente de la pagina vemos un comentario

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/d963237b-831b-464d-a9d9-df833d0d096f)

Hacemos otro reconocimiento con gobuster y encuentra únicamente un index.php

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/d8ec9909-498e-4188-96be-c6ca342ba922)

Lo que se me ocurre es hacer fuzzing a palabras que puedan ser explotadas a un LFI.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a49f2a10-e6d7-4b84-abc1-b5bc5ae3bc95)

Encontramos la palabra love vamos a probar.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/b42d9af1-dc96-48a1-a6f8-2abc49721668)

Funciona, ahora tenemos dos posibles usuarios pedro y rosa, vamos hacer un ataque de fuerza bruta a estos usuarios al protocolo SSH con hydra


## Ataque de fuerza bruta SSH


![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/f751ed74-3328-43f9-9cdc-008e2686e8ae)

`hydra -l pedro -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2`. <br> 
`-h` ⮞ dirección IP de la máquina victima <br>
`-u` ⮞ nombre del posible usuario <br> 
`-P` ⮞ ruta del rockyou. 

Como ya tenemos una contraseña vamos acceder a la maquina vía SSH.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/faa2272c-1eea-4c37-b3ae-692121f61c73)

## Escalada de privilegios

Si hacemos un sudo -l vemos que tenemos los binarios ls y cat que los podemos ejecutar como cualquier usuario 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/2f4c3ba8-b0d5-4637-a68f-1fd7130abfe1)

Si miramos en el interior de la carpeta root encontramos lo siguiente

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/5c6996e8-7534-414c-9a60-786a97ea548f)

Es en exadecimal lo que vamos hacer es decodificarlo

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/bfa1c572-decc-4c24-9eff-649be562396e)

Tenemos una contraseña vamos a probar de acceder con esta contraseña con el usuario root o pedro.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1e386310-9d9d-42eb-b3b6-60821d5d8a7c)


Ahora con el usuario pedro vamos a buscar algo para poder escalar a root.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/626974c3-761c-4744-b2d5-0b5a95abb677)

Tenemos el binario env para escalar privilegios a root haremos lo siguiente

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/bc5b47ad-a506-4e70-a8ff-fb052345a346)

Ya somos ROOT
