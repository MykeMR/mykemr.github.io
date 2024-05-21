---
title: Picadilly - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | kaikoperez        | 

## Reconocimiento Inicial

Durante el reconocimiento inicial, encontramos los puertos 80 y 443 abiertos.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/9d0ec3cb-58d9-4dfb-a8cf-e41554ff0183)

Accedimos a la IP a través del puerto 80 y encontramos lo siguiente:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/613be135-3f74-4a29-9162-18e0ce679457)

Al abrir la página, observamos el siguiente contenido:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/2a39900a-5979-4070-8897-d926a65594c5)

Encontramos el usuario `mateo` y una contraseña cifrada con el cifrado de César. Vamos a buscar en internet un desencriptador para este método, lo que nos proporciona varias posibles contraseñas.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/b8a55cc1-e233-4c91-8ed5-2b0425fb5571)

## Exploración del Puerto 443

Ahora avanzaremos con el puerto 443. En el escaneo de Nmap, vemos que hay un dominio asociado, `picadilly.lab`. Vamos a añadir este dominio al archivo `/etc/hosts`.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/b22fa845-f41f-4743-9e9d-d1e9c02ce9ed)

Al acceder al sitio web, observamos una página con una sección para subir archivos. Antes de realizar cualquier subida, haremos un reconocimiento con Gobuster. En este caso, debemos añadir el parámetro `-k` para que ignore los errores de certificado SSL.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/98814b2a-d6de-4904-8dbc-afcdb1c837c8)

Encontramos el repositorio `uploads`, que es donde se almacenarán los archivos que subamos.

## Subida de Reverse Shell

Intentaremos subir un archivo `reverse-shell.php` para verificar si permite la subida de este tipo de archivos.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/4ededa93-075d-4043-bebe-9b7ab3cc36a6)

Vemos que nos permite subir el archivo PHP. Vamos a ejecutar la reverse shell haciendo clic sobre el archivo subido y nos ponemos en escucha con Netcat por el puerto que hayamos especificado en el archivo, en mi caso, el puerto 443.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ea264bf1-6805-4692-bced-5471741a1021)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/4b1a8a6a-d0a1-4caf-be1b-ca872f1eaf5e)

## Escalada de Privilegios

Una vez dentro de la máquina, vemos que en el directorio `home` existe el usuario `mateo`. Podemos probar las contraseñas que desciframos previamente. La primera opción es `easycrxazy`, pero nos da error. Eliminamos la `x`, dejando la contraseña como `easycrazy`, y obtenemos acceso.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/9a3ef02b-d8ff-4582-9bc8-ff4b1deeb5eb)

Ahora que somos el usuario `mateo`, ejecutamos `sudo -l` y vemos que podemos usar el binario `php` como cualquier usuario.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c43e255a-c8f0-4532-9124-15eb5e06cdd6)

Consultamos GTFObins para ver cómo podemos escalar privilegios usando el binario `php`.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/90285f16-d116-4f20-817e-20945b87baf2)

En lugar de usar el comando `CMD=...` para declarar la variable `cmd`, escribimos directamente `/bin/sh` dentro de `system` para que se genere una terminal.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0ab08e43-1e90-440c-883d-e98b4e8805df)

¡Ahora somos root!
