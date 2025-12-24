---
layout: page
title: tools
permalink: /projects/
description: A work-in-process collection of self-developed interactive tools for scientific research or education.
nav: true
nav_order: 2
display_categories: [polarization]
horizontal: false
---

<h3>Interactive web apps: </h3>
- Polarization state calculator <br> (Links: <a href="https://ilopushenko.github.io/projects/stokes" target="_blank" rel="noopener noreferrer">web app & reference info on polarization ellipse</a>; <a href="https://github.com/ilopushenko/StokesPolarization" target="_blank" rel="noopener noreferrer">GitHub</a>; <a href="https://se.mathworks.com/matlabcentral/fileexchange/162151-stokes-polarization" target="_blank" rel="noopener noreferrer">MATLAB Exchange</a>)

<h3>Libraries:</h3>
- MATLAB interface to W. Wiscombe's Mie scattering Fortran program MIEV0 <br> (Links: <a href="https://github.com/ilopushenko/miev0_matlab_interface/releases" target="_blank" rel="noopener noreferrer">GitHub</a>; <a href="https://doi.org/10.5281/zenodo.17069741" target="_blank" rel="noopener noreferrer">Zenodo DOI</a>; <a href="https://se.mathworks.com/matlabcentral/fileexchange/181979-matlab-interface-to-w-wiscombe-s-mie-scattering-program" target="_blank" rel="noopener noreferrer">MATLAB Exchange</a>)

<!-- pages/projects.md -->
<div class="projects">
{% if site.enable_project_categories and page.display_categories %}
  <!-- Display categorized projects -->
  {% for category in page.display_categories %}
  <h2 class="category">{{ category }}</h2>
  {% assign categorized_projects = site.projects | where: "category", category %}
  {% assign sorted_projects = categorized_projects | sort: "importance" %}
  <!-- Generate cards for each project -->
  {% if page.horizontal %}
  <div class="container">
    <div class="row row-cols-2">
    {% for project in sorted_projects %}
      {% include projects_horizontal.liquid %}
    {% endfor %}
    </div>
  </div>
  {% else %}
  <div class="grid">
    {% for project in sorted_projects %}
      {% include projects.liquid %}
    {% endfor %}
  </div>
  {% endif %}
  {% endfor %}

{% else %}

<!-- Display projects without categories -->

{% assign sorted_projects = site.projects | sort: "importance" %}

  <!-- Generate cards for each project -->

{% if page.horizontal %}

  <div class="container">
    <div class="row row-cols-2">
    {% for project in sorted_projects %}
      {% include projects_horizontal.liquid %}
    {% endfor %}
    </div>
  </div>
  {% else %}
  <div class="grid">
    {% for project in sorted_projects %}
      {% include projects.liquid %}
    {% endfor %}
  </div>
  {% endif %}
{% endif %}
</div>
