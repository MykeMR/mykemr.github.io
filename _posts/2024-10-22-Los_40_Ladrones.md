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

![image](https://github.com/user-attachments/assets/d41b0946-80ee-44bd-93a7-301c8cff5311)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `80` (HTTP)
- `22` (SSH)
A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.

```bash
nmap -p 80,22 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/237d04c4-8a6f-47d5-af9c-a6818d7248e6)

# Exploración Web

Al acceder a la página web, encontramos el panel de Apache por defecto:
