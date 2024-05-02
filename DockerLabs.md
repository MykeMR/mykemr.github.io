---
layout: default
title: DockerLabs
permalink: /categorias/dockerlabs/
---

# WriteUps DockerLab

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c786c0f9-71ee-46b7-b482-6e16406a9ea5)

# Descripción:

Bienvenido a mi repositorio web de WriteUps de Máquinas Vulnerables. Aquí encontrarás una colección detallada de documentos que describen el proceso de explotación y compromiso de diversas máquinas vulnerables en entornos de laboratorio y plataformas de CTF (Capture The Flag)

## Contenido:

WriteUps detallados: Cada WriteUp proporciona una narrativa paso a paso de cómo se realizó la explotación de una máquina vulnerable, desde la enumeración inicial hasta la obtención de acceso privilegiado. Técnicas y Herramientas: Descubre las técnicas de hacking utilizadas, así como las herramientas y scripts empleados durante el proceso de pentesting. Consejos y Estrategias: Aprende de los enfoques creativos y las estrategias efectivas utilizadas para superar los desafíos planteados por cada máquina vulnerable. Niveles de Dificultad Variados: Desde máquinas diseñadas para principiantes hasta desafíos avanzados, encontrarás una amplia gama de niveles de dificultad para poner a prueba tus habilidades de hacking.

## Objetivo

Este repositorio tiene como objetivo principal proporcionar recursos educativos y de aprendizaje para la comunidad de ciberseguridad, ayudando a los estudiantes, entusiastas y profesionales a mejorar sus habilidades en pentesting y seguridad informática.

¡Explora, aprende y mejora tus habilidades de ciberseguridad con nuestros WriteUps de Máquinas Vulnerables!

<style>
  ul {
    list-style-type: none; /* Elimina los puntos de la lista */
    padding: 0; /* Elimina el relleno predeterminado de la lista */
  }

  ul li {
    margin-bottom: 10px; /* Ajusta el espaciado entre los elementos de la lista */
  }

  ul li a {
    font-size: 1.2em; /* Cambia el tamaño de fuente según tus preferencias */
  }
</style>

<ul>
  {% for post in site.categories.dockerlabs %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

