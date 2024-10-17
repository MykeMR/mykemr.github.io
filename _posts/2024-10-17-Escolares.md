---
title: Escolares - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | Luisillo_o        | 


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

![image](https://github.com/user-attachments/assets/7f83b9e1-1efc-47e7-8ce5-c23e2b3e5bce)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `22` (SSH)
- `80` (HTTP)

A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.

```bash
nmap -p 80,22 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/1c698515-45c1-423a-b81b-9a9a0526ba60)

# Exploración Web

Al acceder a la página web, observamos una pagina sobre una *Universidad de Ciberseguridad*.

![image](https://github.com/user-attachments/assets/c1f9320f-5e28-45fe-98bd-25c2a803932c)

Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio.

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

  ![image](https://github.com/user-attachments/assets/f8a22e16-6c9d-4326-a1fc-133004368103)
el 

#WordPress

Exploramos el directorio `wordpress`, pero antes de nada mirando el codigo fuente de la pagina encontramos otra ruta en `profesores.html`.

![image](https://github.com/user-attachments/assets/6a175fca-b27a-4865-b269-c7235cc26717)

Y vemos que luisillo es el adminstrador de `wordpress`, por lo que ya tenemos un usuario interesante, antes de ello vamos a ver que contiene la ruta wordpress para ello volveremos a usar `gobuster`.

Tarda mucho en cargar, mirando vemos que intenta redirigirnos a escolares.dl pero como no lo tenemos en el `/etc/hosts` no llega a cargar del todo bien.

![image](https://github.com/user-attachments/assets/01d0e3c2-59a4-45c6-9a44-830b093601c1)

Por lo que añadiremos.

![image](https://github.com/user-attachments/assets/26d3c04e-4e2c-4aca-af76-f1dd69938f43)

Si probamos otra vez , ahora incluso el `gobuster` funciona mejor.
```bash
gobuster dir -u escolares.dl/wordpress -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img 
```
![image](https://github.com/user-attachments/assets/78af7eab-189b-4cbb-a863-f28a2c2867f6)

Nos encuentra muchas rutas interesantes , pero lo que haremos sera usar wpscan , que esta ella espeficamente para auditar paginas wordpress.

```bash
wpscan --url http://escolares.dl/wordpress -e u,cb,vp,vt
```
Encontramos el usuario que vimos en la ruta de antes de `profesores.html`, por lo que con se ve es un usuario en potencia para probar fuerza bruta.

![image](https://github.com/user-attachments/assets/21a0a639-5334-44e9-9463-f9a831e3f157)

##Fuerza bruta usuario luisillo

Antes vimos informacion de ese usuario por lo que podemos generar un diccionario de contraseñas a traves de ese usuario.
```bash
cupp -i 
```

![image](https://github.com/user-attachments/assets/2be2ec8e-6ba4-4e1c-9f98-70fcac195447)
