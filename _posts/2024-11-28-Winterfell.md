---
title: ShowTime - DockerLabs
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

Encontramos la ruta `/dragon`, el cual vemos que hay un `index` a un archivo llamado `EpisodiosT1`, entramos y vemos lo siguiente:

![image](https://github.com/user-attachments/assets/eb33de46-2ffb-43a3-938d-139862f7374b)

Como no vemos nada mas vamos a ver el puerto `443` si a alguna de estas frases es una contraseña del usuario `jon` que vimos anteriormente.

![image](https://github.com/user-attachments/assets/8719d9c0-03b3-4d5d-88f1-d1d4d72a05ff)

Para ello usaremos `crackmapexec` y meteremos todos los episodios que nos dieron en un .txt

![image](https://github.com/user-attachments/assets/0936c97f-7489-45af-bbf3-5c8bf5ecaf8b)

Ahora haremos fuerza bruta 

```bash
crackmapexec smb 172.17.0.2 -u 'jon' -p episodios.txt 
```
![image](https://github.com/user-attachments/assets/4cc1418f-9bc3-4ebb-b877-1107be74268a)

Vemos el usuario "`jon`" con contraseña "`seacercaelinvierno`", ahora podemos usar `smbmap`.

```bash
smbmap -u 'jon' -p 'seacercaelinvierno' -H 172.17.0.2
```

![image](https://github.com/user-attachments/assets/c94705f1-1e80-4316-98c9-7eec954dd3e4)

