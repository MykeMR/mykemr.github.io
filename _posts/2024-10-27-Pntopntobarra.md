---
title: Pntopntobarra - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | maciiii___        | 


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

![image](https://github.com/user-attachments/assets/44261ae0-a713-4bbd-a656-5faac24621f1)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `22` (SSH)
- `80` (HTTP)
A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.
```bash
nmap -p22,80 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/29121e08-d26c-45c2-aef1-be7698e0a060)

# Exploración Web

Accedemos al puerto `80` y encontramos una página web con un mensaje de advertencia y dos botones: "Hacer clic aquí podría empeorar la situación" y "Ejemplos de computadoras infectadas".

![image](https://github.com/user-attachments/assets/303820c1-440e-4ad1-b3b0-6e5d1e27018c)

Al probar con esa ruta, el archivo se carga sin problemas, lo cual indica que posiblemente estamos ante una vulnerabilidad **LFI (Local File Inclusion)**. Esto permite incluir archivos locales en el servidor al manipular el parámetro `images`.

![image](https://github.com/user-attachments/assets/b86d67c6-9b0e-4fa4-810d-d617e6d4ad5c)

Para confirmar, intentamos leer el archivo `/etc/passwd`, que lista los usuarios en el sistema:

![image](https://github.com/user-attachments/assets/ee0c3bfc-c679-47cf-bcc2-9fdc25cc7eca)

```URL
http://172.17.0.2/ejemplos.php?images=./../../../etc/passwd
```

![image](https://github.com/user-attachments/assets/da7342eb-23a3-4476-b2a8-36cf38cfaa5f)

Vamos a ver que mas podemos hacer , ya que el puerto `22` esta abierto vamos a ver si podemos sacar el `.ssh` de algun usuario.

![image](https://github.com/user-attachments/assets/3d27550e-5c05-45ba-8a24-d9f12a312002)

# Intrusion

Descubrimos el usuario `nico` en `/etc/passwd`. Dado que el puerto `22` está abierto, intentamos obtener la clave SSH privada (`id_rsa`) de nico para acceder al sistema como él. El archivo `id_rsa` en `.ssh` contiene una clave privada que permite autenticación SSH sin contraseña.

Intentamos cargar el archivo de clave privada con LFI:

```URL
http://172.17.0.2/ejemplos.php?images=./../../../home/nico/.ssh/id_rsa
```
![image](https://github.com/user-attachments/assets/4b2a0dea-55c1-4d11-816f-e6617861c8eb)

Copiamos el contenido del archivo `id_rsa` en nuestra máquina y le damos permisos adecuados antes de conectarnos:

```bash
chmod 600 id_rsa
ssh -i id_rsa nico@172.17.0.2                      
```
Esto nos da acceso al sistema como el usuario `nico`.

![image](https://github.com/user-attachments/assets/864916ad-318b-4853-a86f-6b4110b300b8)

# Escalada de Privilegios.

Al ejecutar `sudo -l`, notamos que tiene permisos para ejecutar `env` como `root`.Utilizando [GTFOBins](https://gtfobins.github.io/gtfobins/awk/#shell) sabemos cómo explotar estos binarios.

![image](https://github.com/user-attachments/assets/d2ca0d5c-82ce-432f-8d60-329ce7205c51)

Ejecutamos el siguiente comando para obtener una shell con privilegios de root:
```bash
sudo env /bin/sh
```
Al ejecutar este comando, accedemos como root.

![image](https://github.com/user-attachments/assets/aee995db-8183-4286-9b25-71893c4199d3)
