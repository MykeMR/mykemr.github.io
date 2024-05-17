---
title: Collections - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  | Medio       | ElPinguinoDeMario | 


 Write-up de Reconocimiento y Explotación

## Reconocimiento

### Descubrimiento de Puertos

Encontramos los siguientes puertos abiertos en el objetivo:
- **Puerto 22**: Utilizado por el servicio SSH.
- **Puerto 80**: Utilizado por el servicio HTTP.
- **Puerto 27017**: Utilizado por el servicio MongoDB.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/9c8e0704-d267-47a7-ab1f-788c4598a775)

### Acceso a la Web

Accedemos a la página web hospedada en el puerto 80.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/5ca6c2e0-a61b-4543-84f4-33b1498d3131)

No hay nada visible en la página principal, así que realizamos un escaneo con `gobuster`.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a67a6442-6a64-4280-98ba-356be20563fb)

### Descubrimiento de WordPress

Encontramos un sitio WordPress.

Vamos a inspeccionarlo.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/6bf00810-5dd9-4f77-a995-bd74b4342c2c)

La página se visualiza incorrectamente. Vamos a inspeccionar el código fuente para buscar dominios adicionales.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/b086648b-264c-4103-9bf9-cc8cfea7ede3)

Efectivamente, encontramos un dominio adicional. Vamos a agregarlo al archivo `/etc/hosts` como `collections.dl`.

### Escaneo con WPScan

Realizamos un escaneo con `wpscan` para buscar usuarios y vulnerabilidades.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/bc65f256-45a2-4df1-8480-218c2d947710)

Identificamos un usuario y un directorio `upload`. Procedemos a realizar un ataque de fuerza bruta para obtener la contraseña.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/59150701-1f60-4367-8720-f3e8a231f79d)

Obtenemos una contraseña. Los comandos utilizados fueron:

```bash
wpscan --url [URL] --enumerate u,vp
wpscan --url [URL] --passwords /usr/share/wordlists/rockyou.txt --usernames [usuario]
```
### Autenticación en WordPress

Nos autenticamos en el portal de administración de WordPress en /wp-admin.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/17458155-38de-4aa4-83fb-1f70675f0a92)

### Explotación del Plugin Site Editor

Accedemos al panel de administración y encontramos el plugin Site Editor en la versión 1.1, conocido por ser vulnerable a un LFI (Local File Inclusion). Descargamos el script de explotación desde GitHub.
https://github.com/jessisec/CVE-2018-7422

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/327c21f3-9caf-4583-8d17-b0acf8bca904)


Ejecutamos el script y comprobamos la explotación con un cat /etc/passwd.

Identificamos dos usuarios: chocolate y dbadmin. Realizamos un ataque de fuerza bruta al puerto SSH con hydra.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/35fc2156-af87-4cd2-9e6d-4605c0540067)


Encontramos las credenciales para el usuario chocolate: estrella. Nos conectamos al SSH.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0131e1b8-f7ae-4df9-97c5-dc108ca51c43)

### Conexión SSH y Explotación de MongoDB

Nos conectamos al SSH y encontramos un archivo mongodb-27017.sock en el directorio /tmp. Ejecutamos ps -e -f y observamos que el proceso mongod está ejecutándose con la opción --bind_ip_all, lo que significa que está escuchando en todas las interfaces de red. Esto es un riesgo de seguridad.

Intentamos conectarnos a MongoDB.

### Exploración de MongoDB

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/d57e0a1b-c1a5-4157-85d3-7cbd28c5cefc)


Enumeramos las bases de datos en MongoDB y encontramos información interesante.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/4d06f1cf-133b-4e2d-9bda-b81a3d4612eb)


Identificamos las credenciales del usuario dbadmin: chocolaterequetebueno123. Intentamos usarlas para conectarnos al SSH.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/3ee3b11f-20a3-441b-ba35-aef0ee0782b0)

### Escalada de Privilegios

Nos autenticamos como dbadmin. No encontramos nada inusual para escalar privilegios, así que probamos la misma contraseña chocolaterequetebueno123 para el usuario root.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/fa49b94f-ae7d-4c55-9557-680474293e19)
