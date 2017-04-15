---
layout: default
title: Projects
project_item: project_item.html
---

<div id="project-container">
  <div class="project-list-div">
    <h1>Completed Projects</h1>
    <ul class="project-list">
      {% for project in site.data.projects.completed-projects %}
      {% include {{page.project_item}} proj=project %}
      {% endfor %}
    </ul>
  </div>
<!--
  <div class="project-list-div">
    <h1>Abandoned Projects</h1>
    <ul class="project-list">
      {% for project in site.data.projects.abandoned-projects %}
      {% include {{page.project_item}} proj=project %}
      {% endfor %}
    </ul>
  </div>
	-->
</div>
