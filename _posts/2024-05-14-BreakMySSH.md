---
title: BreakMySSH - DockerLabs
published: true
categories: DockerLabs
---


| OS     | Dificultad  | Creator           |
| ------ | ----------- | -------------     | 
| Linux  | Muy Fàcil   | ElPinguinoDeMario | 

## Reconocimiento

Vemos que está el puerto 22 abierto

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/ae04dc17-36ee-4f5f-ab81-240c7ce55d62)


Tiene una versión de SSH antigua, la 7.7

Hacemos un ataque de fuerza bruta con hydra

```bash
hydra -L /usr/share/metasploit-framework/data/wordlists/unix_users.txt -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

Nos encuentra la contraseña estrella

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/094984d3-a5c8-4fd7-b1bf-d01457bd3921)

Si nos conectamos por SSH con la contraseña sin proporcionar usuario, nos conectamos directamente como root.

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/30047648-403a-4171-a869-6e59f88b8a15)


