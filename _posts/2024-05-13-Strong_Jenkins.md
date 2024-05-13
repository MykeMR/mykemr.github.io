---
title: Strong Jenkins - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Medio      | ElPinguinoDeMario | 

# Reconocimiento

Durante la fase inicial de reconocimiento, identificamos varios puertos abiertos en el sistema. Uno de estos, el puerto 8080, estaba particularmente interesante ya que alojaba una instancia de Jenkins.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/7d4d712c-e2ad-4a70-9c59-7f345c0f0694)

Al acceder a Jenkins, observamos la interfaz de inicio de sesión estándar.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/05694dfc-6236-44c1-bb05-aa62e2facf33)

## Interceptación y Fuerza Bruta

Decidimos interceptar la autenticación usando Burp Suite. Configuramos el proxy para interceptar la solicitud de inicio de sesión con las credenciales de usuario `admin` y contraseña `admin`.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/aafdf149-93b5-4484-8fa9-5788b6f40cd5)

Después de capturar la petición, la enviamos al Intruder de Burp Suite para preparar un ataque de fuerza bruta en el campo de contraseña.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/6b260063-68dd-447c-a6de-522eac82a42a)

Configuramos la pestaña de payloads añadiendo un diccionario de contraseñas comunes para la fuerza bruta. En la pestaña Settings, bajo Grep - Extract, establecimos un filtro para identificar respuestas que indicaran un error de login.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/751baa1d-7eff-4759-88f7-84d85a109527)

Iniciamos el ataque y, tras varias pruebas, descubrimos que la contraseña correcta era `rockyou`, ya que el mensaje no indicaba error de login.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/b72e6a1b-c462-46ac-801b-7f8aa2faf755)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/87b83b45-4ff7-412c-b5cb-68012b801719)

## Escalada de Privilegios

Con acceso a Jenkins, navegamos al apartado 'Manage Jenkins' y luego a 'Script Console', donde ejecutamos código Java para obtener una reverse shell.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/4ae2e2a2-4722-4e19-886f-4962625aadc3)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/064e8672-4227-474f-9fb4-9e609652ade9)

Una vez obtenido acceso a la shell, descubrimos que Python3 tenía configurado el bit SUID, lo cual es una oportunidad para escalar privilegios.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1db8680e-3644-4d8e-ab79-b10e90622295)

Utilizamos este vector para ejecutar Python3 con privilegios elevados y logramos obtener una shell como root, completando la escalada de privilegios.
