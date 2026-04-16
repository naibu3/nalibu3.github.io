---
layout: page
title: TL;DR
permalink: /about/
---

<div class="container mt-4">
  <div class="about-layout">
    <div class="about-profile">
      <img src="/images/pages/about/pablo.jpg" alt="Foto de Pablo" class="about-profile-image" id="profileImage">
    </div>
    <div class="about-copy">
      <p>
        ¡Hola! Soy Pablo (aunque en ctfs me encontraréis como <strong>naibu3</strong> o <strong>pablipoyo</strong>), soy graduado en Ingeniería Informática por la 
        <a href="http://www.uco.es/">Universidad de Córdoba</a> y actualmente trabajo como Pentester en DEKRA Málaga. Como podéis ver, una de mis grandes pasiones es la ciberseguridad, regularmente participo en 
        competiciones de tipo <strong>CTF</strong> con el equipo 
        <a href="https://ctftime.org/team/225933/">Caliphal Hounds</a> y formo parte del 
        <a href="https://www.uco.es/aulaRedesSeguridad/">Aula de Ciberseguridad y Redes</a> de la Universidad de Córdoba.
      </p>
      <p>
        En este blog subo tanto apuntes, como resoluciones de retos resueltos durante las competiciones en las que participo. 
        Además de mis propias experiencias en diversos eventos. ¡Si quieres estar al tanto de lo que hago no dudes en seguirme en mis redes sociales!
      </p>
    </div>
  </div>

  <hr>

  <div class="about-carousel" aria-label="Certificaciones">
    <button class="about-carousel-control" type="button" aria-label="Anterior" data-carousel-prev>&lt;</button>
    <div class="about-carousel-track" data-carousel-track>
      <img src="/images/posts/eJPT_cert.png" alt="eJPT" class="about-carousel-slide is-active">
      <img src="/images/pages/about/CRTP.png" alt="CRTP" class="about-carousel-slide">
      <img src="/images/pages/about/NCL_cert.png" alt="NCL" class="about-carousel-slide">
      <img src="/images/pages/about/hack4u_cert.png" alt="Hack4U Introducción al Hacking" class="about-carousel-slide">
      <img src="/images/pages/about/hackademics.jpg" alt="Hackademics" class="about-carousel-slide">
      <img src="/images/pages/about/HF3.png" alt="Hackademics 3" class="about-carousel-slide">
      <img src="/images/pages/about/NHNCTF.png" alt="NHNCTF" class="about-carousel-slide">
      <img src="/images/pages/about/UGR.png" alt="UGR" class="about-carousel-slide">
    </div>
    <button class="about-carousel-control" type="button" aria-label="Siguiente" data-carousel-next>&gt;</button>
  </div>
</div>

<script>
document.addEventListener("DOMContentLoaded", function () {
  const track = document.querySelector("[data-carousel-track]");
  if (!track) return;

  const slides = Array.from(track.querySelectorAll(".about-carousel-slide"));
  const prev = document.querySelector("[data-carousel-prev]");
  const next = document.querySelector("[data-carousel-next]");
  let currentIndex = 0;

  function renderSlide(index) {
    slides.forEach((slide, slideIndex) => {
      slide.classList.toggle("is-active", slideIndex === index);
    });
  }

  prev.addEventListener("click", function () {
    currentIndex = (currentIndex - 1 + slides.length) % slides.length;
    renderSlide(currentIndex);
  });

  next.addEventListener("click", function () {
    currentIndex = (currentIndex + 1) % slides.length;
    renderSlide(currentIndex);
  });
});
</script>
