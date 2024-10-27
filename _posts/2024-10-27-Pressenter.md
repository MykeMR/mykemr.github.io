---
title: Pressenter - DockerLabs
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

![image](https://github.com/user-attachments/assets/1e7de8c9-352d-495c-a303-ac2df0163f02)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `80` (HTTP)
A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.

```bash
nmap -p80 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/3541d37b-ed61-4a33-8c81-007e43a2e26f)

# Exploración Web
Al acceder al puerto `80`, encontramos una página con un mensaje de inicio de reto. 
![image](https://github.com/user-attachments/assets/adcfc82b-bf2e-47ef-be51-5f096644bc53)

En el código fuente aparece un dominio oculto: `pressenter.hl`. Lo añadimos en `/etc/hosts` y accedemos nuevamente.

![image](https://github.com/user-attachments/assets/bfe294e8-3aa3-48dd-8c6e-894491656b12)

Observamos que el sitio usa WordPress, por lo que intentamos acceder a `wp-admin` y otros recursos de WordPress.

![image](https://github.com/user-attachments/assets/c01a2a13-8c20-4328-aa02-c205d2f0475c)

Usamos `WPSCAN` para enumerar los usuarios de WordPress:

```bash
wpscan --url http://pressenter.hl/ -e vp,cb,u
```

Identificamos dos usuarios: `pressi` y `hacker`.

## Fuerza bruta 

Ejecutamos fuerza bruta sobre `pressi` con el diccionario `rockyou.txt`:
```bash
wpscan --url http://pressenter.hl/ -U pressi -P /usr/share/wordlists/rockyou.txt 
```

![image](https://github.com/user-attachments/assets/d46efe6d-7489-43f4-925d-98915ec82f38)

Obtenemos la contraseña `dumbass` para `pressi` y accedemos al panel de WordPress.

![image](https://github.com/user-attachments/assets/6330cd25-7744-4289-9cbd-f88a0202a524)

# Reverse Shell

En el panel de WordPress, instalamos el plugin File Manager.

![image](https://github.com/user-attachments/assets/fea1d042-ab86-4ade-a8f1-4b831d2ba14a)

Aprovechamos la sección de subida de archivos para cargar una reverseshell en PHP. ara esto, generamos una reverseshell desde **Pentest Monkey**, una herramienta popular para crear shells PHP que nos permiten obtener acceso remoto al servidor. La ruta de la reverseshell es la siguiente:

[Reverseshell de Pentest Monkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)

![image](https://github.com/user-attachments/assets/51f36ccf-e3a0-4fce-bcfb-5b3f8cba0d4e)

Iniciamos `netcat` en nuestro sistema para escuchar conexiones entrantes:

```bash 
nc -lvnp 4444
```

Accedemos a la reverse shell a través del navegador:

![image](https://github.com/user-attachments/assets/ba14667e-c769-4e9c-9e3c-8ba8c46fbf3d)

Logramos acceso como` www-data`.

![image](https://github.com/user-attachments/assets/0cd9e20f-a723-41ff-8818-445f4d845005)

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

# Intrusion con el usuario enter.

Identificamos al usuario `enter` en el sistema y exploramos el archivo `wp-config.php` en WordPress, donde encontramos las credenciales de la base de datos (`admin` / `rooteable`).

![image](https://github.com/user-attachments/assets/794d5a1f-48ea-4f17-b1b1-2c5f8eaacb71)

```bash
mysql -u admin --password=rooteable
```

Una vez en MySQL, exploramos la base de datos `wordpress` y buscamos en la tabla `wp_usernames`:

```sql
select*from wp_usernames;
```
```sql
+----+----------+-----------------+---------------------+
| id | username | password        | created_at          |
+----+----------+-----------------+---------------------+
|  1 | enter    | kernellinuxhack | 2024-08-22 13:18:04 |
+----+----------+-----------------+---------------------+
```

Esto nos proporciona la contraseña de enter: `kernellinuxhack`.

Con esto, cambiamos al usuario enter:

```bash
su enter
```

# Escalada de Privilegios 

Al ejecutar `sudo -l`, notamos que spencer tiene permisos para ejecutar `cat` como `root` y `whoami` como `root` .Utilizando [GTFOBins](https://gtfobins.github.io/gtfobins/awk/#shell) sabemos cómo explotar estos binarios.

![image](https://github.com/user-attachments/assets/e72c8d19-7017-480f-9c15-c52d5fd1114a)

Exploramos la raíz de `enter` y encontramos `user.txt`, lo que sugiere que un archivo equivalente puede estar en el directorio `/root`. Como podemos usar `cat` como root, lo leemos:

![image](https://github.com/user-attachments/assets/7974fdd2-0ca0-4bb3-bf80-d8edb3b71112)

![image](https://github.com/user-attachments/assets/3c28faab-82ad-4216-b895-63a1ee64529d)

Alternativamente, al probar la misma contraseña de `enter` para root, conseguimos acceso root:

![image](https://github.com/user-attachments/assets/617bef1d-5194-41ab-9785-5e84b6eb268e)

