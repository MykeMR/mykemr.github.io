---
title: LittlePivoting - DockerLabs
published: true
categories: DockerLabs
---

 
| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  | Medio       | ElPinguinoDeMario | 

# Reconocimiento

Vemos el puerto 80 y el puerto 22 abiertos

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/74d84973-ec3a-46ce-a379-04deda329411)

Hacemos un reconocimiento con gobuster

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0e41f8e1-0a72-4479-9f2d-a6cceee90b4d)

Entontramos un directorio /shop vamos a ingresar

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/079f1069-0c63-4f4f-8398-763b1ed48d70)

Vemos que tenemos una pista que añadiendo el parámetro archivo, podemos vulnerar a un LFI

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/76ce5228-8890-420d-a005-416889b2f5cb)

Encontramos 2 posibles usuarios 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/52945197-ab70-4764-948e-479f54c11bb6)

Hacemos un ataque de fuerza bruta y encontramos que manchi tiene la contraseña "lovely"

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/314de088-8cd0-4b1c-b709-4b87b2c0eb32)

Nos descargamos algún script para poder hacer fuerza bruta en local ya que hay dos usuarios para ver si encontramos la contraseña de seller

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/2733e3f6-1eec-4892-8d67-72a448d6ab25)

Encontramos que la contraseña de seller es "qwerty"

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/e07ecc92-e597-44b6-afb4-e4a26e2dd3ba)

Escalamos al usuario seller y hacemos un sudo -l, vemos que tenemos permisos para ejecutar php como cualquier usuario 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ec2c0ff2-2f93-47fe-a9e7-70960885bb5e)

Ahora vamos a enumerar IPs que haya en dicho PC con un script en bash
```
#!/bin/bash

for i in {1..255}; do
        timeout 1 bash -c "ping -c 1 20.20.20.$i" >/dev/null
        if [ $? -eq 0 ]; then
                echo "El host 20.20.20.$i está activo"
        fi
done

```

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/748d3f7f-6bfc-4db7-b084-82d2e2c721b3)

Perfecto, entonces ahora vamos a hacer portforwarding con chisel para traernos los puertos de la máquina 20.20.20.3 a nuestra IP

En la máquina atacante desplegamos el chisel para que haga de servidor

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1461a610-9b51-4afb-8071-2e2ec57ad818)

Y en la máquina víctima lo iniciamos de la siguiente forma

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0858beca-bd76-4d47-8dc3-aad5278647f4)

Esto nos ha creado un túnel entre las dos máquinas

Ahora hacemos un escaneo con nmap para ver los puertos abiertos

```bash
proxychains nmap -sT --top-ports 100 20.20.20.3 -Pn -oN nmap_20.20.20.3
```

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/898f2047-7663-4ee9-9993-23c3cec4ade9)

Ahora vamos a configurar el proxy en foxyproxy para podernos conectar a la web

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1b2356e5-98ce-4a2e-96b3-8be86f021a72)

Una vez hecho esto, ya podemos ver la web alojada en la máquina 2

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/2ab0cd0d-170c-459f-8f52-25a4ef58e424)

Procedemos a hacer un análisis de directorios con gobuster con el parámetro --proxy y le especificamos el socks5 y la IP más el puerto

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/d71c066c-4041-4906-87a4-d7f641f40482)

Encontramos un secret.php

Vamos a ver qué hay

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a4c397e2-c51c-42ec-a7f5-f31b7c626141)

Encontramos un posible usuario mario, vamos a hacer fuerza bruta con hydra

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/4860e520-e5fc-4baf-a1dd-c2c72c068b56)

Encontramos el usuario mario con la contraseña "chocolate"

Vamos a conectarnos a la máquina con esas credenciales

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/5c208042-bb90-4177-a83e-21a10c3cb134)

Ya estamos dentro de la máquina, vamos a escalar privilegios

Podemos ejecutar vim como cualquier usuario, vamos a buscarlo en gtfobins

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/5b2a1732-8ed1-4c3e-a630-0b3ada87deb2)

Ejecutamos el comando sudo -u root /usr/bin/vim -c ':!/bin/sh'

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0d124e11-deef-41b8-9e3b-f79c96c84b57)

Ya somos root de la segunda máquina

Ejecutamos un hostname -I para ver las interfaces de red y vemos que tiene dos

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/fb8b84ea-3812-4e77-87cd-f8ce5d8c3792)

Esta máquina no tiene ping instalado, así que vamos a aprovecharnos del /dev/tcp para hacer un escaneo

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/677a4143-87cb-442b-aa0c-a09a920cf8d0)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0d4bf1a2-155e-4eb9-8ed6-d87b35d8f445)

Nos muestra que la IP 30.30.30.3 está habilitada

Ahora vamos a compartir por el puerto 8080 de la máquina 20.20.20.2 el chisel a la máquina 20.20.20.3

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/36c49bb5-d0da-4806-948e-9131231f185c)

Para que la conexión sea posible, tendremos que hacer que desde esta máquina llegue a nuestra máquina, sin embargo, no tenemos conectividad desde la 30.30.30.0/24 a la 10.10.10.0/24 pero, sí que tiene conectividad con nosotros la red 20.20.20.0/24. Por ello, haremos uso de la herramienta socat ya que nos permitirá que, nosotros desde la máquina 30.30.30.2, nos conectemos con chisel a la máquina 20.20.20.2 y la máquina 20.20.20.2 redirija esa conexión a la 10.10.10.1 que somos nosotros.

Nos descargamos socat de GitHub y lo subimos a la máquina 20.20.20.2

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/fe1edf21-6b32-4bb7-83ac-efa0c3c6b401)

Iniciamos socat haciendo que todas las peticiones que pasen por socat las mande al PC atacante por el mismo puerto que tiene chisel

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/d8b6a86b-a832-4d16-8480-d030af1b819f)

En la máquina hacemos que todo el tráfico lo redirija a la máquina 20.20.20.2 por el mismo puerto que hemos configurado socat

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0b0495e4-4680-479b-88cc-6fe318a0df24)

Ahora vamos a hacer un escaneo de los 100 puertos de la máquina 30.30.30.3

Encontramos el puerto 80 abierto

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1cbfbe38-17f2-468c-9f11-5b6d769a7e8d)

Vamos a configurar el foxyproxy

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/935bbeef-4b2b-4807-9f0c-e79de4bd923c)

Accedemos al puerto 80

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/eb3389bf-9230-49d1-8ba7-e54279081819)

Vamos a hacer un reconocimiento con gobuster

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/2222fe70-de48-4771-b8d5-81e3e926c64b)

Vemos un directorio uploads, entonces vamos a subir un archivo para mandarnos una reverse shell

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ec7ba4a3-55c1-4642-b415-3d5f6ded8ac4)
![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/b782cb79-3b3d-4aab-b5b2-39d03c568567)
![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ba01521d-5257-4200-8d03-c7b6faafa545)

Ya tenemos subida la reverse shell, ahora vamos a configurar que la reverse shell nos llegue a nuestra máquina con socat

En la máquina 20.20.20.2 montamos un socat que redirija todo lo que haya en el 442 a la IP 10.10.10.1 al 441

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0812796f-ee5c-4999-a0bd-d22055f16baa)

Y en la máquina 30.30.30.2 montamos otro socat que redirija todo lo que venga del 443 a la IP 20.20.20.2 al 442

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/888bac3b-6731-4bb9-b6e8-18d4686613f4)

Si ahora en la máquina atacante creamos un nc por el puerto 441 le damos click encima del php-reverse-shell.php y nos debería llegar la reverse shell

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/2469cfc1-0ee9-46f3-bb59-86ebc9315873)

Ahora vamos a escalar privilegios

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/eeea2711-de25-4870-aa88-ec55d04fb77c)

Si hacemos un sudo -l

Vemos que podemos ejecutar env como root, vamos a ver cómo se hace con gtfobins

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/78080cfc-f031-4d0c-af76-3ad9ef73f95b)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/81f322c8-d493-4259-971c-311c6d38d543)

¡Ya hemos llegado al final!
