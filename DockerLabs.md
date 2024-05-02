---
layout: default
title: DockerLabs
permalink: /categorias/dockerlabs/
---

# Publicaciones de la categoría DockerLabs
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
