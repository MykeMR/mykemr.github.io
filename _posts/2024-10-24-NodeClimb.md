---
title: NodeClimb - DockerLabs
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

![image](https://github.com/user-attachments/assets/271b1bee-8e4e-4c89-8913-01352bc9ce9f)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `22` (SSH)
- `21` (HTTP)
A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.

```bash
nmap -p21,22 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/cb616348-797b-4da8-bbbf-ce9631c61b44)

# FTP

Observamos que el usuario `Anonymous` está habilitado y encontramos un archivo llamado `secretitopicaron.zip`. Procedemos a acceder y descargarlo:
```bash
get secretitopicaron.zip
```

![image](https://github.com/user-attachments/assets/0a9c43bc-615f-4729-a57e-1aca927b9f9c)

Intentamos descomprimir el archivo, pero nos pide una contraseña que no tenemos:
```bash
unzip secretitopicaron.zip
```

![image](https://github.com/user-attachments/assets/36ce0dc9-1094-4d80-a3ef-36c1fdbde3cb)


## Fuerza bruta 

Para extraer la contraseña, utilizamos `zip2john` para obtener el hash del archivo zip:

```bash
zip2john secretitopicaron.zip > hash.hash
```

Luego usamos `john` para realizar fuerza bruta sobre el hash:

```bash
john hash.hash 
```
![image](https://github.com/user-attachments/assets/6c1ec5cc-e9a4-42fa-83ed-883c187caec7)

La contraseña obtenida es `password1`. Ahora podemos descomprimir el archivo zip:

Volvemos a descomprimir el `zip`.

```bash
unzip secretitopicaron.zip
```
Como podemos ver nos ha extraido un archivo llamado `password.txt`. 

![image](https://github.com/user-attachments/assets/bdd31639-1794-43c5-889f-709b12f97361)

Este archivo contiene un usuario `mario` y una contraseña `laKontraseñAmasmalotaHdelbarrioH`. Con estos credenciales accedemos a través de SSH:
```bash
ssh mario@172.17.0.2
```

# Escalada de privilegios.
Al ejecutar `sudo -l`, vemos que podemos ejecutar `node` como `root` en el archivo `/home/mario/script.js`.Utilizando [GTFOBins](https://gtfobins.github.io/gtfobins/awk/#shell) sabemos cómo explotar estos binarios.

Vemos que tenemos permisos sobre el archivo, por lo que podemos modificar su contenido:

![image](https://github.com/user-attachments/assets/86da9927-d885-4b7d-a0c3-597f7c1d9695)

Modificamos el archivo para generar una shell:

![image](https://github.com/user-attachments/assets/26f7653a-5772-4e49-a0c8-a14078605c54)

Finalmente, ejecutamos el script con privilegios de root y ya somos root:

![image](https://github.com/user-attachments/assets/79c2b325-d32d-4631-9e38-c33151f332bd)

