---
layout: home
---

This site is a for remakes of Intersect courses

<div class="card-deck">
  <div class="card">
    {% assign collection =  site.collections | where: "label", "WEBDATA201" | first %}
    <img class="card-img-top" src="assets/logos/sub-brand.png" alt="{{ collection.label }}">
    <div class="card-body">
      <h4 class="card-title"> {{ collection.title }} </h4>
      <p class="card-text"> {{ collection.description }} </p>
      <a href="{{ collection.relative_url | relative_url }}" class="btn btn-primary"> Go to course </a>
    </div>
  </div>
</div>
