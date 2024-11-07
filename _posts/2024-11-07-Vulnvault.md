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
Al inspeccionar el puerto `80`, encontramos una página con dos secciones: un formulario y una opción de carga de archivos. Ejecutamos **Gobuster** para identificar posibles directorios ocultos en el sitio web:

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
Intentamos cargar una reverse shell PHP (shell.php) mediante la función de subida, que indica estar configurada para bloquear archivos inseguros.

![image](https://github.com/user-attachments/assets/bc37ab0d-53ef-44e5-8dd6-64ba5a9f7a98)

Usamos una reverse shell de [Reverseshell de Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) para intentar obtener acceso. Aunque la subida se completa, no encontramos el archivo en `/uploads.php`.

![image](https://github.com/user-attachments/assets/95b68f1c-c4fb-40a2-8bcd-50ea2dae3809)

Al analizar la respuesta en `Burp Suite`, vemos un mensaje de error en la subida del archivo:

![image](https://github.com/user-attachments/assets/57d25f26-c130-4c67-b121-413f1309a9dc)

Mirando con burpsuite podemos observar que en response nos indica que tuvo un error al subir el archivo.

![image](https://github.com/user-attachments/assets/fb30aae2-baa8-4179-89a1-eb8146dbd4eb)

# Apartado de formulario.

Observamos que cualquier texto ingresado en el formulario es reflejado por el servidor. Un archivo `script.js` intenta bloquear ciertos comandos en las entradas del formulario.

![image](https://github.com/user-attachments/assets/8da628ac-2ddf-4376-b7a6-8cf9d38b72cd)


![image](https://github.com/user-attachments/assets/d46c00ad-e702-4660-8554-08f99ea8057a)

Al probar diferentes métodos para escapar la restricción de comandos, descubrimos que insertar un comando tras el carácter `;` permite ejecutarlo en el servidor.

![image](https://github.com/user-attachments/assets/0eaf4fd2-cab4-4778-9ca7-88c3ee11908c)

Por lo que podemos hacer muchas cosas , desde generar una reverseshell a enumerar usuario y hacer fuerza bruta o obtener el rsa de algun usuario.

## Obtener id_rsa

Utilizamos este método para acceder al archivo id_rsa de un usuario llamado `samara`.

![image](https://github.com/user-attachments/assets/ff67b1ed-4419-4b55-bcc7-492b3fa15aa9)

Guardamos la clave privada `id_rsa` en nuestro sistema para usarla más adelante.

![image](https://github.com/user-attachments/assets/4e62f320-27f4-44ee-91d7-a3b9d1c5a222)

Lo guardamos en nuestra maquina.

![image](https://github.com/user-attachments/assets/f6f1f477-4a2d-4871-8202-de871824e82c)


# Intrusion Samara
Utilizando `ssh`, accedemos al sistema como `samara`:
```bash
ssh -i id_rsa samara@172.17.0.2
```

Ya estamos dentro 

![image](https://github.com/user-attachments/assets/98c3a361-7fe3-4ece-b3e3-ed537da64eb6)

Examinamos permisos **SUID** , sin encontrar opciones útiles:
```bash
find / -perm -4000 2>/dev/null
```

Descargamos `pspy64` en el sistema para monitorear procesos en ejecución y detectar posibles vulnerabilidades.

![image](https://github.com/user-attachments/assets/25a77ea5-6617-4c4d-8bc5-87476c4359ab)

Descargamos el archivo en la carpeta `/tmp`.

![image](https://github.com/user-attachments/assets/75581b4f-3b0a-4b4d-a918-52b048a9ce21)

Después de ejecutar `pspy64`, descubrimos que el script `echo.sh` se ejecuta recurrentemente.

![image](https://github.com/user-attachments/assets/bcb8882c-e18d-44b0-bf55-d57607886a1c)

Vamos al directorio y podemos observar que este script generaba el archivo que vimos en el directorio de samara.

![image](https://github.com/user-attachments/assets/430bb375-e98f-4e12-ae43-b37ca6b959e0)

Nos dirigimos al directorio de `echo.sh`, donde verificamos que podemos modificar el script. Insertamos un comando para obtener una shell de root.

![image](https://github.com/user-attachments/assets/8954cdc0-ac1b-4f4d-a7bd-f25bab438ae6)

Para escalar privilegios, ejecutamos:
```bash 
bash -p
```

![image](https://github.com/user-attachments/assets/4f575857-dca7-4e2b-8841-7283268c7015)

Ya somos root.







