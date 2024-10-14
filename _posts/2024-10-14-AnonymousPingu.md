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
![image](https://github.com/user-attachments/assets/fbdcf75b-e25d-4d92-8797-3e558b2a92e6)

-  `-p-`: Escanea todos los puertos (1-65535).
- `--open`: Solo muestra los puertos abiertos.
- `-sS`: Realiza un escaneo SYN, que es rápido y discreto.
- `--min-rate=5000`: Envía al menos 5000 paquetes por segundo, acelerando el escaneo.
- `-n`: No realiza resolución DNS.
- `-Pn`: No realiza ping previo para determinar si el host está activo.
- `-vvv`: Muestra resultados detallados y en tiempo real.

## Exploración Web

## Obtención de Acceso

## Escalada de Privilegios

