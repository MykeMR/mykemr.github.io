---
title: DockerLabs - DockerLabs
published: true
categories: DockerLabs
---
 
| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  | Fácil       | ElPinguinoDeMario | 

# Reconocimiento

## Escaneo Inicial
Realizamos un escaneo con **nmap** y observamos que el puerto 80 está abierto.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/61783089-5920-4589-acdb-6f9236ffb2e5)

## Acceso a la Web
Al acceder a la web a través del puerto 80, nos encontramos con la siguiente interfaz:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ff93c9eb-4ac7-4173-97f0-a61c63a9d2b5)

## Reconocimiento con Gobuster
Para profundizar en el reconocimiento, ejecutamos **gobuster** para enumerar directorios y archivos ocultos en la web.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ad4ebebf-6551-4c45-817a-6c3dd05fcbef)

### Resultados de Gobuster
Encontramos dos directorios interesantes: `upload` y `uploads`. Procedemos a investigarlos más a fondo.

## Exploración de Repositorios
### Directorio upload
Dentro del directorio `upload`, encontramos el archivo `upload.php`, pero no contiene nada útil. Sin embargo, en `machine.php`, detectamos la capacidad de subir archivos. Vamos a intentar subir una **reverse shell** en PHP para obtener acceso.

## Intento de Subida de Reverse Shell
Al intentar subir un archivo PHP, recibimos el siguiente error:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a3535e24-9ec9-4a22-892d-aad2c0387654)

## Bypass de Restricciones de Extensión PHP
Para sortear esta restricción, utilizamos diferentes extensiones de archivo PHP. Las extensiones disponibles son:

.php, .php2, .php3, .php4, .php5, .php6, .php7, .phps, .phps, .pht, .phtm, .phtml, .pgif, .shtml, .htaccess, .phar, .inc, .hphp, .ctp, .module

perl


Vamos a probar cada una de ellas para ver cuál nos permite subir el archivo.

![Pruebas de extensiones PHP](Pasted%20image%2020240521091412.png)

### Subida Exitosa
Finalmente, la extensión correcta es `.phar`. Ahora, vamos al directorio `uploads` y hacemos clic en el archivo que hemos subido: `php-reverse.shell.phar`.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/61c6502c-a4f4-4224-b7a5-2811dfd81d39)

## Acceso a la Máquina
Hemos obtenido acceso a la máquina. Dentro de la carpeta `OPT`, encontramos una nota con el siguiente contenido:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/4faf3ddd-b469-4e75-a3c1-6d388dd14e59)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/49728234-6e87-428d-83e5-fd0fdc72ef13)


## Comprobación de Privilegios
Ejecutamos el comando `sudo -l` y obtenemos la siguiente salida:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a2ed1876-a0d7-4f0c-9706-98f840122ccc)

## Escalación de Privilegios
Buscamos el binario `cut` en **GTFOBins** y encontramos una vulnerabilidad explotable. Procedemos a ejecutarlo para ver qué sucede.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/755286ee-30a4-46e4-97b8-fbd6697f7dea)

### Lectura de Archivos
Hemos podido leer el archivo con éxito.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/37cbbeb1-8614-45cb-8f31-97de1ecef548)

## Acceso a Root
Utilizamos la contraseña encontrada para obtener acceso root.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/87a90878-b960-41eb-8568-f866158780c2)

