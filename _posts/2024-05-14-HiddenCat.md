---
title: HiddenCat - DockerLabs
published: true
categories: DockerLabs
---

| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  | Fácil       | ElPinguinoDeMario | 

# Reconocimiento

Encontramos los puertos 22, 8080 y 8009 abiertos.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/6eba2df7-fc84-4a9d-a454-0c7ec338fc1d)

## Puerto 8080

Tenemos un Tomcat versión 9.0.30.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/bce3ed7e-b28d-4acd-9bad-42bc9ab2ac3e)

Al hacer fuzzing con gobuster, descubrimos la ruta `/manager`.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ecf4daa4-8863-4e3d-90ba-4c58fe488e30)

Sin embargo, no podemos hacer mucho con esta información.

## Puerto 8009

Investigando sobre el puerto 8009, encontramos que es vulnerable a un LFI debido a que Tomcat no está actualizado. Más información se puede encontrar en el siguiente enlace: [HackTricks - 8009 Pentesting Apache JServ Protocol (AJP)](https://book.hacktricks.xyz/v/es/network-services-pentesting/8009-pentesting-apache-jserv-protocol-ajp).

HackTricks proporciona un script que, al ejecutarlo, revela lo siguiente:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/940ab50b-e293-432b-80d3-6ba2cf09ef7b)

Descubrimos un posible usuario llamado "jerry".

## Ataque de Fuerza Bruta por SSH

Procedemos a realizar un ataque de fuerza bruta por SSH con hydra.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c2895963-151f-4718-af93-d07d5c66a9c5)


Vemos que efectivamente tenemos credenciales válidas en SSH y nos conectamos.

## Permisos SUID

Buscando permisos SUID, encontramos tanto `perl` como `python`.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/7aaade85-4830-420e-af1c-e522922e4899)

## Escalada de Privilegios

Consultamos GTFObins para encontrar una forma de escalar privilegios con `python`. Ejecutamos el siguiente comando:

```bash
/usr/bin/python3.7 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c170cc0b-8b4d-4a87-86b3-88c8a1e93b2c)

YA SOMOS ROOT!
