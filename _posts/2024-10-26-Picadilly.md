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
En el puerto `80`, observamos que hay un archivo `backup.txt`.

![image](https://github.com/user-attachments/assets/c223e6da-3c42-4ce9-8f6c-135dac67d123)

Al revisar su contenido vemos lo siguiente.(Explica que es).

```txt
/// The users mateo password is ////


----------- hdvbfuadcb ------------

"To solve this riddle, think of an ancient Roman emperor and his simple method of shifting letters."

////////////////////////////////////
```

Por lo que que se ve pone que esa encriptado en el cifrado cesar. (Explica que era el cifrado Cesar brevemente).Si intentamos averiguar lo que pone podemos ver que la que mas se asemeja es `easycrazy`.

![image](https://github.com/user-attachments/assets/b267e0da-c76c-4e08-b998-d81d1639215d)

Tenemos eso pero no sabemos como usarlo asi que haremos fuzzing, por si exite algun directorio mas.

Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio.
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

![image](https://github.com/user-attachments/assets/4124f3fa-0b88-4c21-896b-45ff077a1736)

Viendo el escaneo del principio vemos que el puerto 443 responde al nombre de host: `picadilly.lab` , con lo que vamos a añadirlo al `/etc/hosts` y analizar que contiene esa pagina.

![image](https://github.com/user-attachments/assets/fc002df1-e9f5-4c21-be95-0544d4f4068c)

Al explorar la pagina del puerto `443` vemos a primera vista que hay un apartado de subida de archivos y una ruta dentro del codigo fuente de la pagina llamado `uploads.php`, asi que empezaremos por ahi.

## Intrusión en la Máquina
Lo primero que haremos sera subir una reverseshell por `php`, ya que hemos visto  la pagina de subida su extension es `uplodas.php` suponemos que la pagina responde  `php.

Generamos la reverseshell utilizando **Pentest Monkey**, una herramienta popular para crear shells PHP que nos permiten obtener acceso remoto al servidor. La ruta de la reverseshell es la siguiente:

[Reverseshell de Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

Nos ponemos a escuchar por netcat en mi caso por el 4444

```bash 
nc -lvnp 4444
```

### Pasos para Cargar la Reverseshell

1. **Preparar la Reverseshell**: Creamos un archivo de reverseshell en nuestro sistema local utilizando el código disponible en la ruta mencionada anteriormente.
2. **Subir el Archivo**: pasamos el archivo para subir al apartado de subida de la pagina.
3. **Ejecutar la Reverseshell**: Una vez que el archivo esté en el servidor, lo ejecutaremos para establecer la conexión.

Luego, accedemos a la ruta de la página web donde hemos cargado la reverseshell:

![image](https://github.com/user-attachments/assets/4f49a3e9-f08d-4bdb-8af7-ea29c73bed4a)

Y ya tendriamos acceso a la maquina.

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

Por lo que generaremos una shell a traves de php y entraremos.

```bash
sudo /usr/bin/php -r "pcntl_exec('/bin/sh', ['-p']);"
```

Ya somos root 

![image](https://github.com/user-attachments/assets/f15c6502-638b-45a7-a34e-555f458897ef)
