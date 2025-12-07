---
layout: page
title: Projects
permalink: /projects/
description: Research, development, and educational projects spanning AI, embedded systems, and open-source tools.
nav: true
nav_order: 3
display_categories: [research, Master's, tools, educational]
horizontal: true
# Lista manuale dei progetti da mostrare (in ordine di visualizzazione)
projects_to_show:
  - TakuNet
  - AutoDock
  - RL-Swarm
  - MultiVision
  - EdgePowerMeter
  - OSCUP
  - FiremanSam
  - PytorchEssentials
  - MarsSpectrometry
  - BrainForce
---

<!-- pages/projects.md -->
<div class="projects">
{% if page.projects_to_show %}
  <!-- Display only manually selected projects -->
  {% if site.enable_project_categories and page.display_categories %}
    <!-- Display categorized projects -->
    {% for category in page.display_categories %}
    <a id="{{ category }}" href=".#{{ category }}">
      {% if category == "research" %}
        <h2 class="category">🔬 Research Projects</h2>
        <p class="category-description">Academic research in Edge AI and Efficient Deep Learning</p>
      {% elsif category == "Master's" %}
        <h2 class="category">🎓 Master's Degree Projects</h2>
        <p class="category-description">Projects developed during M.Sc. in Artificial Intelligence Engineering at UNIMORE</p>
      {% elsif category == "tools" %}
        <h2 class="category">🛠️ Development Tools & Frameworks</h2>
        <p class="category-description">Open-source tools and frameworks for AI research and embedded systems</p>
      {% elsif category == "educational" %}
        <h2 class="category">📚 Educational & Competition Projects</h2>
        <p class="category-description">Learning projects, tutorials, and machine learning competitions</p>
      {% else %}
        <h2 class="category">{{ category }}</h2>
      {% endif %}
    </a>
    {% assign categorized_projects = site.projects | where: "category", category %}
    {% assign filtered_projects = "" | split: "" %}
    {% for project in categorized_projects %}
      {% assign project_slug = project.path | split: "/" | last | split: "." | first %}
      {% if page.projects_to_show contains project_slug %}
        {% assign filtered_projects = filtered_projects | push: project %}
      {% endif %}
    {% endfor %}
    <!-- Generate cards for each project -->
    {% if page.horizontal %}
    <div class="container">
      <div class="row row-cols-1 row-cols-md-2">
      {% for project in filtered_projects %}
        {% include projects_horizontal.liquid %}
      {% endfor %}
      </div>
    </div>
    {% else %}
    <div class="row row-cols-1 row-cols-md-3">
      {% for project in filtered_projects %}
        {% include projects.liquid %}
      {% endfor %}
    </div>
    {% endif %}
    {% endfor %}
  {% else %}
    <!-- Display projects without categories -->
    {% assign filtered_projects = "" | split: "" %}
    {% for project in site.projects %}
      {% assign project_slug = project.path | split: "/" | last | split: "." | first %}
      {% if page.projects_to_show contains project_slug %}
        {% assign filtered_projects = filtered_projects | push: project %}
      {% endif %}
    {% endfor %}
    <!-- Generate cards for each project -->
    {% if page.horizontal %}
    <div class="container">
      <div class="row row-cols-1 row-cols-md-2">
      {% for project in filtered_projects %}
        {% include projects_horizontal.liquid %}
      {% endfor %}
      </div>
    </div>
    {% else %}
    <div class="row row-cols-1 row-cols-md-3">
      {% for project in filtered_projects %}
        {% include projects.liquid %}
      {% endfor %}
    </div>
    {% endif %}
  {% endif %}
{% else %}
  <!-- Fallback: Display all projects if no manual list is defined -->
  {% if site.enable_project_categories and page.display_categories %}
    <!-- Display categorized projects -->
    {% for category in page.display_categories %}
    <a id="{{ category }}" href=".#{{ category }}">
      <h2 class="category">{{ category }}</h2>
    </a>
    {% assign categorized_projects = site.projects | where: "category", category %}
    {% assign sorted_projects = categorized_projects | sort: "importance" %}
    <!-- Generate cards for each project -->
    {% if page.horizontal %}
    <div class="container">
      <div class="row row-cols-1 row-cols-md-2">
      {% for project in sorted_projects %}
        {% include projects_horizontal.liquid %}
      {% endfor %}
      </div>
    </div>
    {% else %}
    <div class="row row-cols-1 row-cols-md-3">
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
      <div class="row row-cols-1 row-cols-md-2">
      {% for project in sorted_projects %}
        {% include projects_horizontal.liquid %}
      {% endfor %}
      </div>
    </div>
    {% else %}
    <div class="row row-cols-1 row-cols-md-3">
      {% for project in sorted_projects %}
        {% include projects.liquid %}
      {% endfor %}
    </div>
    {% endif %}
  {% endif %}
{% endif %}
</div>
