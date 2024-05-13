---
title: Inclusion - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  |  Medio      | ElPinguinoDeMario | 

# Reconocimiento

Durante la etapa de reconocimiento inicial, determinamos que los puertos 22 (SSH) y 80 (HTTP) estaban abiertos en el sistema objetivo. Este hallazgo establece dos vías principales de exploración: una directa a través de SSH y otra a través de la interfaz web.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/a9b4e5ea-df19-488d-b82e-736fe43b8487)


![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/0c0accea-d80d-4111-bb1e-05a4483939dd)


## Reconocimiento Web

Procedimos a realizar un reconocimiento más detallado de la interfaz web del sistema. Utilizamos herramientas de exploración web para identificar directorios y archivos accesibles.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c7d88d9c-9179-4686-a1ec-1b772de380d2)


Descubrimos un directorio interesante llamado `shop`.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/75e7e7d5-055a-49fc-a418-c72580ab8d8a)

Al explorar este directorio, observamos un mensaje de error en la parte inferior izquierda de la página. Este error indica la posibilidad de que el sitio sea susceptible a ataques específicos contra su implementación de PHP.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/6ec57f14-107d-4fbb-ba00-4b5254b67fc6)


Aprovechando la información revelada por el error, pudimos navegar por los directorios del sistema, incluido el acceso al archivo `/etc/passwd`. Sin embargo, en los directorios de usuario no encontramos archivos `id_rsa`, lo que nos llevó a optar por un ataque de fuerza bruta para descubrir contraseñas.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/709b21c0-c2ec-4847-a82f-3563351d7b7e)

Identificamos un usuario válido y procedimos a intentar el acceso SSH.

## Escalada de Privilegios

En la cuenta de usuario `manchi`, no encontramos ficheros con capacidades especiales ni archivos SUID que nos permitieran elevar privilegios. Por lo tanto, decidimos preparar un ataque de fuerza bruta para la cuenta de usuario `seller`.

Descargamos el siguiente script de fuerza bruta:

```bash
wget https://raw.githubusercontent.com/Maalfer/Sudo_BruteForce/main/Linux-Su-Force.sh
```

Necesitábamos subir este script junto con la lista de contraseñas rockyou al servidor a través de SCP para ejecutarlo.

Una vez ejecutado el script, logramos obtener la contraseña del usuario seller.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/4f7e072b-02bd-424d-b4f1-cbd0ff237cac)


Con esta nueva contraseña, ejecutamos el comando su para cambiar al usuario seller.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/74e78abd-b63d-4113-a6e7-30d7e51a68b5)


Descubrimos que el usuario seller tenía permisos para ejecutar PHP como sudo sin contraseña. Utilizando esta ventaja, ejecutamos un comando PHP para iniciar una shell con privilegios de root.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/6a1fe543-843d-4290-b2fe-86c882467fa5)

Con este método, logramos obtener acceso como usuario root al sistema.

Somos root.
