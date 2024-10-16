---
title: DockerLabs - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | ElPinguinoDeMario | 


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

![image](https://github.com/user-attachments/assets/505dcb13-4f0c-4d3c-9a85-243319937411)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `80` (HTTP)

A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.

```bash
nmap -p 80-sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/a3b78674-c6ff-471a-8c2d-73efd12ec5c6)

# Exploración Web

Accedemos a la maquina y no vemos nada interesante a primera vista.

![image](https://github.com/user-attachments/assets/8e867cf8-4c54-46ae-82fe-37ce63c38252)

Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio.
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

![image](https://github.com/user-attachments/assets/1c7abed3-4cb1-4b91-9502-c2cb52735451)

Encontramos varios directorios interesantes, entrando en machines.php podemos ver que podemos subir maquinas, pero y si en vez de eso subimos una reverse shell a traves de php.

![image](https://github.com/user-attachments/assets/a0124837-926e-4e38-9af7-d4b601fcfe15)

## Reverse Shell

Generamos la reverseshell utilizando **Pentest Monkey**, una herramienta popular para crear shells PHP que nos permiten obtener acceso remoto al servidor. La ruta de la reverseshell es la siguiente:

[Reverseshell de Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)


