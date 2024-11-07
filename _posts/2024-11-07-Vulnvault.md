---
title: Vulnvault - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | d1se0        | 


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

![image](https://github.com/user-attachments/assets/6e946fa0-5b81-4bed-abe9-2613e64c2209)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `80` (HTTP)
- `22` (SSH)
A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.
```bash
nmap -p22,80 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/2c3ffc6c-8f3f-4cb9-8c05-cbfbfd15aad3)

# Exploración Web
Al analizar el puerto `80`, encontramos una página con varios apartados uno de un formulario y otro de subida de archivos.

Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio.
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

![image](https://github.com/user-attachments/assets/256f19e3-d829-460e-ae65-2c85591c2f9d)

# Apartado de subida de archivos.

## Reverse Shell
Vamos a probar a subir un archivo shell.php, la pagina dice que esta configurada para subir archivos de forma segura, por lo que habra que ofuscar las medidas de seguridad.

![image](https://github.com/user-attachments/assets/bc37ab0d-53ef-44e5-8dd6-64ba5a9f7a98)

Creamos un archivo `shell.php` y cargamos una reverse shell usando la herramienta [Reverseshell de Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) para crear una shell en PHP. La reverse shell se encuentra disponible en:

![image](https://github.com/user-attachments/assets/95b68f1c-c4fb-40a2-8bcd-50ea2dae3809)

Parece que no hemos tenido que oufscar nada, una vez subido el archivo, accedemos a la ruta `/upload.php` y ejecutamos la shell.

No salta un mensaje de que se ha subido el archivo accedmos a la ruta /uploads.php , pero no hay nada.

![image](https://github.com/user-attachments/assets/57d25f26-c130-4c67-b121-413f1309a9dc)

Mirando con burpsuite podemos observar que en response nos indica que tuvo un error al subir el archivo.

![image](https://github.com/user-attachments/assets/fb30aae2-baa8-4179-89a1-eb8146dbd4eb)

# Apartado de formulario.

En el apartado de formulario vemos que si escribimos algo el servidor lo muestra, mirando vemos que hay un `script.js` que es el que se encarga de bloquear intentos de generar comandos.

![image](https://github.com/user-attachments/assets/8da628ac-2ddf-4376-b7a6-8cf9d38b72cd)


![image](https://github.com/user-attachments/assets/d46c00ad-e702-4660-8554-08f99ea8057a)

Probando diferentes maneras de escapar los comandos podemos observar que esto ocurre cuando insertamos `;` y luego un comando, es válido en cualquiera de los campos.

![image](https://github.com/user-attachments/assets/0eaf4fd2-cab4-4778-9ca7-88c3ee11908c)

Por lo que podemos hacer muchas cosas , desde generar una reverseshell a enumerar usuario y hacer fuerza bruta o obtener el rsa de algun usuario.

## Obtener id_rsa

Tenemos un usuario `samara`.

![image](https://github.com/user-attachments/assets/ff67b1ed-4419-4b55-bcc7-492b3fa15aa9)

Tenemos en id_rsa de `samara`.

![image](https://github.com/user-attachments/assets/4e62f320-27f4-44ee-91d7-a3b9d1c5a222)

Lo guardamos en nuestra maquina.

![image](https://github.com/user-attachments/assets/f6f1f477-4a2d-4871-8202-de871824e82c)


# Intrusion Samara
Accedemos por ssh al usuario `samara`.

```bash
ssh -i id_rsa samara@172.17.0.2
```

Ya estamos dentro 

![image](https://github.com/user-attachments/assets/98c3a361-7fe3-4ece-b3e3-ed537da64eb6)

Revisamos permisos SUID y las capabilities y podemos observar que no tenemos nada que nos pueda ser útil.

```bash
find / -perm -4000 2>/dev/null
```

Usaremos `pspy64` para ver si tenemos algún archivo que nos pueda ser útil. Copiamos el archivo en nuestro directorio actual e indicamos nuestro servidor en Python.

![image](https://github.com/user-attachments/assets/25a77ea5-6617-4c4d-8bc5-87476c4359ab)

Descargamos el archivo en la carpeta /tmp.

![image](https://github.com/user-attachments/assets/75581b4f-3b0a-4b4d-a918-52b048a9ce21)

Damos permisos de ejecución y luego ejecutamos.

![image](https://github.com/user-attachments/assets/68af8099-7743-4b64-bff5-f0e1b9d97326)

Revisando los archivos podemos observar que se está ejecutando constantemente el archivo `echo.sh`.

![image](https://github.com/user-attachments/assets/bcb8882c-e18d-44b0-bf55-d57607886a1c)

Vamos al directorio y podemos observar que este script generaba el archivo que vimos en el directorio de samara.

![image](https://github.com/user-attachments/assets/430bb375-e98f-4e12-ae43-b37ca6b959e0)

Tenemos permisos para modificar el archivo por lo que añadiremos lo siguiente.

![image](https://github.com/user-attachments/assets/8954cdc0-ac1b-4f4d-a7bd-f25bab438ae6)

Ahora solo falta ejecutar el siguiente comando y ya somo root.

```bash 
bash -p
```

![image](https://github.com/user-attachments/assets/4f575857-dca7-4e2b-8841-7283268c7015)

Ya somos root.
