---
title: ChocolateFire - DockerLabs
published: true
categories: DockerLabs
---

| OS    | Dificultad | Creator           |
| ----- | ---------- | ----------------- |
| Linux | Medio      | ElPinguinoDeMario |

## Escaneo y Reconocimiento de puertos

Vamos a iniciar nuestro reconocimiento con un rápido escaneo de nmap.
`nmap -p- --open -sS -sC -sV --min-rate=5000 -vvv -n -Pn -oN escaneo 172.17.0.2`
-  `-p-`: Escanea todos los puertos (1-65535).
- `--open`: Solo muestra los puertos abiertos.
- `-sS`: Realiza un escaneo SYN, que es rápido y discreto.
- `-sC`: Utiliza los scripts de reconocimiento por defecto de Nmap.
- `-sV`: Detecta la versión del servicio en los puertos abiertos.
- `--min-rate=5000`: Envía al menos 5000 paquetes por segundo, acelerando el escaneo.
- `-n`: No realiza resolución DNS.
- `-Pn`: No realiza ping previo para determinar si el host está activo.
- `-vvv`: Muestra resultados detallados y en tiempo real.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/b34f0c31-9d39-45c2-8bbc-301884d5447b)

Vemos que estan abiertos varios puertos, vamos a investigar cada una hasta encontrar algo interesante.

## Web
Si accedemos al servicio web que corre por el pueto 9090 vemos lo siguiente 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/5ad6d755-64f3-413d-b2c1-94f9f9f1be52)

Si probamos usaurio y contraseña admin:admin accedemos sin problemas.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/21651930-8dd9-4cc8-befa-c1bdd1c37b8f)

Al ver la versión vamos a buscar vías potenciales para acceder a la maquina.

Si vamos a la ventana plugins vemos que podemos subir archivos .jar

Si vamos a la siguiente web https://www.rapid7.com/db/modules/exploit/multi/http/openfire_auth_bypass_rce_cve_2023_32315/ nos muestra como podemos hacer la intrusión con metasploit y como normalemente no hacemos write ups con esta herramienta la voy a usar así que abrimos metasploit.

## Intrusión

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1fbdafd1-58f0-4b5d-a7a1-b3d648b23202)

usamos el siguiente modulo de metasploit

```
use exploit/multi/http/openfire_auth_bypass_rce_cve_2023_32315
```

Ahora hacemos un show options y vamos rellenando los campos que necesitemos para hacer la intrusión

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0b809be2-98f5-4209-b502-99af2cff7956)

En este caso el RHOSTS y el LHOST y usamos el comando run para empezar a lanzar el exploit

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/777d7ccf-88d4-4902-80db-fb1ed9c4fefa)

Ya hemos hecho la intrusión y somos el usuario root.
