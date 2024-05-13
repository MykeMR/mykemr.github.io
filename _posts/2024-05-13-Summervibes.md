![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/6bc03885-c370-499b-ac13-7f6f9fe2f2ec)![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/99ac1335-d4c8-4c99-8df9-b1a1e02e4bcf)---
title: Summervibes - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  | Difícil     | ElPinguinoDeMario | 


# Reconocimiento

Durante el reconocimiento inicial, identificamos que los puertos 22 (SSH) y 80 (HTTP) estaban abiertos en el sistema.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/6c41c568-bfbc-4468-bfa6-7d0a83d0f45e)

## Exploración Web

Navegamos hacia la página web a través del puerto 80 y nos encontramos con la página por defecto de Apache2.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1078b250-eaf2-4b0b-ac58-5917bef3a747)

En el código fuente de la página, descubrimos una pista que nos llevó a investigar más sobre el repositorio `cmsms`.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/55e8679e-0752-4fb4-8916-70cee5122ad2)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/19e218ea-894f-4f8c-a568-0c397b257719)


Utilizando Gobuster, realizamos un escaneo para descubrir directorios ocultos.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ce787e44-a9ca-4410-aa6b-f63f1dbac2a9)

Entre los directorios encontrados, destacó uno llamado `admin`, que contenía un panel de login.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0e423936-9971-4d31-ac75-284473a7fb87)

## Ataque de Fuerza Bruta

Decidimos realizar un ataque de fuerza bruta contra el panel de login utilizando Hydra:

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt "http-post-form://172.17.0.2/cmsms/admin/login.php:username=^USER^&password=^PASS^&loginsubmit=Submit:User name or password incorrect"
```
![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0ada9ddc-2624-42a6-8629-e8f0d4c4f90d)

El resultado fue exitoso, obteniendo como credenciales admin con la contraseña chocolate. Accedimos al panel de administración del CMS.
Explotación de Vulnerabilidades

Buscando en internet, encontramos vulnerabilidades conocidas para la versión 2.2.19 de CMS Made Simple. Decidimos utilizar un exploit disponible en:

https://github.com/capture0x/CMSMadeSimple

En el panel, bajo "Extensions" y "User defined tags", añadimos un nuevo tag con el siguiente código PHP para comprobar la vulnerabilidad:


``` php
<?php echo system('id'); ?>
```

Al ejecutarlo, confirmamos que el sistema era vulnerable.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/5d7a3fd5-3b5b-4416-9be9-bc97bdb377a9)

Procedimos a enviarnos una reverse shell.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/317ea3c6-f1a4-41c2-b14c-cd50acd91dc1)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/d0be3271-b539-4d60-a610-e15b44ad99df)

## Escalada de Privilegios

Una vez dentro del sistema, ejecutamos LinPEAS, pero no encontramos vectores de escalada evidentes. Decidimos intentar un ataque de fuerza bruta en local contra el usuario root.

Descargamos tanto la lista rockyou como un script de fuerza bruta, disponible en:

https://raw.githubusercontent.com/Maalfer/Sudo_BruteForce/main/Linux-Su-Force.sh

Ejecutamos el script con el usuario root y la lista rockyou.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/207488ce-b5cb-442b-99dd-3fe2fb6817a5)

Con éxito, escalamos privilegios y obtuvimos acceso como usuario root.

Ya somos root.
