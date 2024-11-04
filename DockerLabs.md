---
layout: default
title: DockerLabs
permalink: /categorias/dockerlabs/
---

<link rel="icon" href="/assets/favicon.ico" type="image/x-icon">


# 🚀 DockerLabs WriteUps
Bienvenido a mi repositorio de **WriteUps** sobre máquinas vulnerables en la plataforma de **DockerLabs**, creada por [Pingüino de Mario](https://www.youtube.com/@ElPinguinoDeMario) 🐧. Aquí encontrarás guías detalladas para comprometer diversas máquinas en esta plataforma de laboratorios **CTF** (*Capture The Flag*).

## 🔍 Buscar un WriteUp:

<input type="text" id="searchInput" onkeyup="searchFunction()" placeholder="Buscar por título..." />

## 📚 Lista de WriteUps:

<ul id="writeup-list">
  {% for post in site.categories.DockerLabs %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

<!-- Estilos personalizados -->
<style>
  #searchInput {
    width: 100%;
    padding: 12px;
    margin-bottom: 20px;
    background-color: #222;
    color: #eee;
    border: 1px solid #444;
    border-radius: 8px;
  }

  ul#writeup-list {
    list-style-type: none;
    padding: 0;
    margin: 20px 0;
  }

  ul#writeup-list li {
    margin-bottom: 15px;
    background-color: #333;
    padding: 10px;
    border-radius: 5px;
    transition: background-color 0.3s ease;
  }

  ul#writeup-list li:hover {
    background-color: #444;
  }

  ul#writeup-list li a {
    color: #00d1b2;
    font-weight: bold;
    font-size: 1.1em;
    text-decoration: none;
  }

  ul#writeup-list li a:hover {
    color: #ffdd57;
  }

  h1, h2 {
    color: #00d1b2;
    text-shadow: 1px 1px 2px #000;
  }

  body {
    background-color: #1b1b1b;
    color: #eaeaea;
  }
</style>

<!-- Script de búsqueda -->
<script>
  function searchFunction() {
    var input, filter, ul, li, a, i, txtValue;
    input = document.getElementById('searchInput');
    filter = input.value.toUpperCase();
    ul = document.getElementById('writeup-list');
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
