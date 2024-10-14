---
title: AnonymousPingu - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Facil      | ElPinguinoDeMario | 


# Reconocimiento 
Comenzamos el proceso de reconocimiento identificando los puertos abiertos en el sistema objetivo. Descubrimos que los puertos que se encuentran activos son el 21 (FTP) y el 80 (HTTP).
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

![image](https://github.com/user-attachments/assets/fbdcf75b-e25d-4d92-8797-3e558b2a92e6)

A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.

```bash
nmap -p 21,80 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/35d4dab8-0433-48d9-9943-52e4e2a36bfd)

## Exploración Web
Al acceder a la página web, observamos lo siguiente:

![image](https://github.com/user-attachments/assets/ede1f8d5-c313-4e69-94ee-aee4e813aa5e)

Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio. El siguiente comando fue ejecutado:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

![image](https://github.com/user-attachments/assets/7684d11b-a469-40f3-8254-b0d964ec0112)

Vemos que hay un directorio upload pero no hay nada dentro.

![image](https://github.com/user-attachments/assets/d2061512-4cf0-4b1a-90a6-0abaad99a259)

Procedemos a explorar el servicio FTP, ya que hemos identificado anteriormente que el usuario `anonymous` está habilitado y que tenemos permisos de escritura en el directorio `upload`. 

A continuación, intentamos conectarnos al servidor FTP utilizando el siguiente comando:
```bash
ftp 172.17.0.2
```

## Obtención de Acceso

## Escalada de Privilegios

