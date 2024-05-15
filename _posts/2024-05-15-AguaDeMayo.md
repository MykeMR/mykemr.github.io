---
title: AguaDeMayo - DockerLabs
published: true
categories: DockerLabs
---

| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  | Fácil       | TheHackersLabs    | 



# Reconocimiento

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c62631be-fc0e-4f8f-9b09-550a64acae36)

Encontramos el puerto 22 y el puerto 80.

Accedemos a la web y encontramos esto:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/fcd30bfb-581b-44ac-a0e9-2ef73a13b1f3)

Vamos a hacer un reconocimiento con gobuster:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c7c3a23d-84e5-4c25-816e-d35879d5302d)

Encontramos el directorio `images`, vamos a acceder:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/5f2a4b46-b96b-49ee-a464-379d9e6bc3c3)

Vemos una imagen, vamos a descargarla:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/95cb9e3e-a9cb-46b8-8fe9-a1281791b281)

Vamos a aplicar esteganografía.

Pero nos pide una contraseña. Si miramos el código fuente de Apache, al final vemos esto:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ccb1c8e4-c3c9-4d98-84e1-04cbf69a0281)

Si se lo preguntamos a ChatGPT, nos dice que es un brainfuck. Vamos a descifrarlo en [dcode.fr](https://www.dcode.fr/brainfuck-language).

En esta web lo podemos descodificar y nos da este resultado:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a264ec54-a346-4cd6-912f-395ce9811e3c)

Yo pensaba que tenía que hacer esteganografía o algo, pero vamos, que si nos vamos a lo fácil, la imagen tiene el nombre de `agua_ssh`. Agua es el usuario y la contraseña es `bebeaguaqueessano`.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/d2f7ac3c-bd47-416e-8dc2-79ebcf396ba8)

Si hacemos un `sudo -l` vemos lo siguiente:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/3049d7ea-4d95-4059-8fb8-d4defa97fa22)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/aae7a79c-2b1b-481c-947f-a17e91a86a2b)

Ejecutamos el binario.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/853bd484-2586-4af6-9b17-c0d427f7d401)

Tenemos un modo que es `!`. Con este símbolo más un comando, ya nos podemos mover libremente por la máquina como root.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/410f5fd6-dd67-42cb-aa2e-8e837b0c7ce7)
