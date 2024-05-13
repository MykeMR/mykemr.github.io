---
title: Fileception - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Medio      | ElPinguinoDeMario | 

# Reconocimiento

Durante el reconocimiento inicial del sistema objetivo, identificamos que los puertos 21, 22 y 80 estaban abiertos. Esta información es crucial para planificar nuestros siguientes pasos en la evaluación de la seguridad.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ebbdc720-296f-4591-8b10-f839916dc2c8)


# FTP

Al probar el acceso FTP, logramos conectarnos utilizando el usuario *anonymous*. Dentro del servidor FTP, encontramos varios archivos, incluyendo una imagen que podría contener información oculta. Procedimos a descargar esta imagen utilizando el comando `get` para explorarla más a fondo.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/628b39e0-b6ba-4564-bc98-1e9afd3360d7)

# WEB

Al acceder al puerto 80 con un navegador, nos encontramos con la página por defecto de Apache2 en Debian, lo cual indica que no se ha personalizado la configuración inicial del servidor web.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/75d5da67-0a72-4921-b300-649104f52ae9)


Al inspeccionar el código fuente de la página, descubrimos un comentario que podría ser una pista importante para avanzar en nuestra evaluación.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/d4f897b9-b732-45a5-86ab-046eb5e49a04)


A partir de este punto, tenemos dos pistas principales: la primera sugiere que deberíamos aplicar técnicas de esteganografía a la imagen descargada del FTP; la segunda pista es una contraseña cifrada que probablemente necesitaremos descifrar para avanzar.

Después de probar varios métodos de desencriptación, descubrimos que el cifrado utilizado era base85. Logramos descifrar la contraseña en la siguiente página web:

[https://www.rfctools.com/base85-decoder/](https://www.rfctools.com/base85-decoder/)

La contraseña obtenida es: `base_85_decoded_password`.

# Esteganografía en la imagen

Utilizando la herramienta Steghide, aplicamos la contraseña desencriptada a la imagen. Como resultado, obtuvimos un archivo de texto que parecía no contener información útil a primera vista. Sin embargo, al analizarlo detenidamente, identificamos un posible nombre de usuario: `[peter]`.

Investigando más sobre el contenido del archivo, descubrimos que estaba escrito en un lenguaje de programación esotérico. Encontramos un decodificador para este tipo de código en la siguiente URL:


![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/f5636243-f244-4ca6-baf7-87f0712044a0)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a85119ba-25c8-45d0-a54e-31dddd1a0422)

[https://www.cachesleuth.com/bfook.html](https://www.cachesleuth.com/bfook.html)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/51f38b9e-2766-4b36-838a-b8fc54186923)


Al ejecutar el código esotérico en el decodificador, conseguimos una posible contraseña para el usuario Peter: `9h889h23hhss2`. Decidimos probar esta contraseña en el siguiente paso.

# SSH

Con la contraseña en mano, intentamos acceder al sistema a través de SSH como el usuario Peter y tuvimos éxito.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/704f9d5a-d4ff-48f5-a6d2-7cce65315b28)


Una vez dentro, ejecutamos el comando `ls` para explorar los archivos disponibles en el directorio actual.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/01263ee3-0633-4354-9d56-79631adb03c8)


Nos dirigimos al directorio `/tmp` para continuar nuestra exploración.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c023f849-916b-41da-bd80-eb5ec9a863f2)


Nos preguntamos si cambiar la extensión del archivo `.odt` a `.txt` nos permitiría leer su contenido. Descargamos el archivo a nuestro equipo atacante y cambiamos la extensión usando el comando `mv`.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/fb84b30e-50d7-4712-97f7-6ed428ee9b39)


Aunque intentamos convertir el archivo en varios formatos, no encontramos información relevante hasta que lo convertimos a `.zip`. Dentro del archivo comprimido, encontramos varios documentos, incluido uno llamado `leerme.xml`. Al abrir este archivo con `cat`, descubrimos el nombre de usuario `octopus` y su contraseña cifrada.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/04d56d89-9b51-4ea0-88ad-43473396b7ca)


![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/73f07694-9880-4a0e-9149-6461160bc6ea)

La contraseña estaba cifrada en base64, así que procedimos a desencriptarla.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/9f21aed0-4e07-4afa-b99a-0636738654f1)

Finalmente, comprobamos los privilegios del usuario Octopus con `sudo -l` y descubrimos que podía ejecutar cualquier comando como superusuario. Ejecutamos `sudo su` para elevar nuestros privilegios y convertirnos en administradores del sistema.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/6204be05-6eab-41cf-8e10-3ca3ea703010)


