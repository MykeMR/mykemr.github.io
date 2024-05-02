---
layout: default
title: DockerLabs
permalink: /categorias/dockerlabs/
---

# WriteUps DockerLab

![imagen](https://github.com/romabri/romabri.github.io/assets/51706860/c786c0f9-71ee-46b7-b482-6e16406a9ea5)

# Descripción:

Bienvenido a mi repositorio web de WriteUps de Máquinas Vulnerables. Aquí encontrarás una colección detallada de documentos que describen el proceso de explotación y compromiso de diversas máquinas vulnerables en entornos de laboratorio y plataformas de CTF (Capture The Flag)

## WriteUps:

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

  #searchInput {
    width: 100%;
    padding: 10px;
    background-color: black; /* Fondo negro */
    color: lime; /* Letras de color lima */
    border: none;
    border-radius: 5px;
    margin-bottom: 10px;
    box-sizing: border-box; /* Incluir el padding en el ancho total */
  }
</style>

<input type="text" id="searchInput" onkeyup="searchFunction()" placeholder="Buscar por título...">
<br><br> <!-- Espacio adicional entre el buscador y la lista de enlaces -->

<ul>
  {% for post in site.categories.DockerLabs %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

<script>
  function searchFunction() {
    var input, filter, ul, li, a, i, txtValue;
    input = document.getElementById('searchInput');
    filter = input.value.toUpperCase();
    ul = document.querySelector('ul');
    li = ul.getElementsByTagName('li');

    for (i = 0; i < li.length; i++) {
      a = li[i].getElementsByTagName('a')[0];
      txtValue = a.textContent || a.innerText;
      if (txtValue.toUpperCase().indexOf(filter) > -1) {
        li[i].style.display = '';
      } else {
        li[i].style.display = 'none';
      }
    }
  }
</script>
