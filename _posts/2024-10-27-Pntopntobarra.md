---
title: Pntopntobarra - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | maciiii___        | 


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

![image](https://github.com/user-attachments/assets/857b077a-8e60-4fab-bc4b-2011e3d219dd)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `22` (SSH)
- `80` (HTTP)
A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.

```bash
nmap -p80,443 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/454db12c-80e6-4883-b45e-bd5ebedde5cd)

# Exploración Web
Al navegar en el puerto `80`, encontramos un archivo `backup.txt` que contiene la siguiente información encriptada:
