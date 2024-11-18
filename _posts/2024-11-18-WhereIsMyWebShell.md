---
title: WhereIsMyWebShell - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | El Pingüino de Mario        | 


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

![image](https://github.com/user-attachments/assets/2adb662a-8cba-48da-82eb-b98ff3962e89)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `80` (HTTP)
A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.
```bash
nmap -p80 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/2f018b31-e463-4902-93b8-fc617163684a)

# Exploración Web
Al analizar el puerto `80`, encontramos una pagina de una academia de ingles.
¡
