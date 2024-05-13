---
title: HackPenguin - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Medio      | ElPinguinoDeMario | 

# Reconocimiento

Durante la etapa inicial de reconocimiento, identificamos que los puertos 80 (HTTP) y 22 (SSH) estaban abiertos en el sistema objetivo. Esta observación nos proporciona un punto de partida para explorar más a fondo la infraestructura del servidor.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/d7993927-8914-47c6-b08f-9e9eaeb4b43a)


## Exploración Web

Nuestro primer paso fue explorar la página web accesible a través del puerto 80.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/fb2f9192-9114-4073-b743-acf8ee678fcc)


Decidimos utilizar la herramienta Gobuster para hacer fuzzing en busca de directorios o archivos ocultos. Durante este proceso, descubrimos un archivo interesante llamado `penguin.html`.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/42808530-58d5-40a9-b202-4bb99407c87c)


Continuamos con el fuzzing, pero no encontramos más información relevante. Sin embargo, al intentar acceder a la imagen mencionada en `penguin.html`, nos dimos cuenta de que estaba protegida por una contraseña. Ante la falta de pistas, optamos por crear un script en Bash para realizar un ataque de fuerza bruta y descifrar la contraseña.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a0e7c7b1-2c2f-47f3-9366-6fd15e5ec862)


El ataque de fuerza bruta fue exitoso, y descubrimos que la contraseña era `chocolate`. Con esta contraseña, accedimos al contenido oculto dentro de la imagen.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/cd387c40-a352-481a-a9cb-98f267a193f0)


## Extracción de Datos

Dentro de la imagen, encontramos un archivo `.kdbx`, un contenedor de base de datos de contraseñas utilizado por KeePass. Intentamos abrirlo, pero nuevamente nos encontramos con la necesidad de una contraseña.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/5abbbdaa-9c64-40e7-87c5-e2fbd60bc4be)


Para descifrar esta nueva contraseña, configuramos un ataque de fuerza bruta. El proceso involucró extraer el hash de la contraseña del archivo `.kdbx` y utilizar herramientas como John the Ripper para descifrarlo.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/03e83aa2-e64c-437c-a0ba-c1ecaf9867db)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/dbfd147d-4beb-4b0e-87c1-0ab36f3a399d)


Finalmente, logramos obtener la contraseña `password1`.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/bbb4624f-3505-4bc6-bd8c-ea19267692d1)

Con esta contraseña, pudimos acceder a la base de datos de KeePass.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/3f397233-685c-4c2e-83a5-9b1c5b5d3ef9)


## Acceso al SSH

Utilizando las credenciales obtenidas de KeePass, procedimos a acceder al sistema a través de SSH.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1d927f4f-2539-4069-ba26-c4823b490a0d)

Dentro del sistema, encontramos dos archivos relevantes: un archivo `.txt` y un script `.sh`. El script generaba un archivo `.txt`, pero modificamos este script para que también nos otorgara permisos SUID a la bash, lo cual nos permitiría elevar nuestros privilegios.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/13decfeb-58c4-40b5-b8ca-63a083d73c4d)


Tras ejecutar el script modificado, logramos obtener permisos de root en el sistema.

Ya somos root.
