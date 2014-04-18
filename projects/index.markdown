---
layout: default
title: Projects
project_item: project_item.html
---

Here you'll find all of my projects I think are worth sharing. With some of the recent projects I've been recording my development and posting them on [youtube](https://www.youtube.com/channel/UCfgnY89MkCnR1Q0p-YpTyXg) just for fun.

<div id="project-container">
  <div class="project-list-div">
    <h1>Current Projects</h1>
    <ul class="project-list">
      {% for project in site.data.projects.current-projects %}
      {% include {{page.project_item}} proj=project %}
      {% endfor %}
    </ul>
  </div>

  <div class="project-list-div">
    <h1>Past Projects</h1>
    <ul class="project-list">
      {% for project in site.data.projects.past-projects %}
      {% include {{page.project_item}} proj=project %}
      {% endfor %}
    </ul>
  </div>
</div>
