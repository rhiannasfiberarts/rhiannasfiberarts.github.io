---
layout: default
title: Blog
permalink: /blog/
---

<div class="blog-page container">

  {% comment %}
    ------------------------------------------------------------------
    Pass 1: work out which tags exist across all posts and how many
    posts belong to each one, so the filter dropdown can be built
    (most posts first) before the list itself is rendered below.
    ------------------------------------------------------------------
  {% endcomment %}
  {% assign known_slugs  = "" | split: "," %}
  {% assign known_labels = "" | split: "," %}
  {% assign tag_hits     = "" | split: "," %}

  {% for post in site.posts %}
    {% if post.tags %}
      {% for raw_tag in post.tags %}
        {% assign clean_tag = raw_tag | strip %}
        {% assign tag_slug = clean_tag | slugify %}
        {% if tag_slug != "" %}
          {% unless known_slugs contains tag_slug %}
            {% assign clean_label = clean_tag | capitalize %}
            {% assign known_slugs  = known_slugs | push: tag_slug %}
            {% assign known_labels = known_labels | push: clean_label %}
          {% endunless %}
          {% assign tag_hits = tag_hits | push: tag_slug %}
        {% endif %}
      {% endfor %}
    {% endif %}
  {% endfor %}

  {% comment %}Sort tags by post count, descending (simple selection sort — the tag list is small){% endcomment %}
  {% assign sorted_slugs  = "" | split: "," %}
  {% assign sorted_labels = "" | split: "," %}
  {% assign sorted_counts = "" | split: "," %}
  {% assign used_slugs    = "" | split: "," %}
  {% assign tag_total     = known_slugs.size %}

  {% if tag_total > 0 %}
    {% for step in (1..tag_total) %}
      {% assign best_slug  = "" %}
      {% assign best_label = "" %}
      {% assign best_count = -1 %}
      {% for slug in known_slugs %}
        {% assign idx = forloop.index0 %}
        {% unless used_slugs contains slug %}
          {% assign matches = tag_hits | where_exp: "h", "h == slug" %}
          {% assign this_count = matches.size %}
          {% if this_count > best_count %}
            {% assign best_count = this_count %}
            {% assign best_slug  = slug %}
            {% assign best_label = known_labels[idx] %}
          {% endif %}
        {% endunless %}
      {% endfor %}
      {% if best_slug != "" %}
        {% assign used_slugs    = used_slugs | push: best_slug %}
        {% assign sorted_slugs  = sorted_slugs | push: best_slug %}
        {% assign sorted_labels = sorted_labels | push: best_label %}
        {% assign sorted_counts = sorted_counts | push: best_count %}
      {% endif %}
    {% endfor %}
  {% endif %}

  <header class="page-header page-header--with-filter">
    <h1>Blog</h1>

    <div class="gallery-filter" id="blogFilter">
      <button type="button" class="filter-toggle" id="blogFilterToggle" aria-haspopup="listbox" aria-expanded="false">
        <span class="filter-toggle-label" id="blogFilterToggleLabel">All Tags</span>
        <span class="filter-toggle-caret" aria-hidden="true">&#9662;</span>
      </button>

      <div class="filter-panel" id="blogFilterPanel" hidden>
        <input type="text" class="filter-search" id="blogFilterSearch" placeholder="Search tags…" autocomplete="off" aria-label="Search tags">
        <ul class="filter-options" id="blogFilterOptions" role="listbox">
          <li class="filter-option" data-tag="" role="option" aria-selected="true">
            <span class="filter-option-label">All Tags</span>
            <span class="filter-option-count">{{ site.posts.size }}</span>
          </li>
          {% for slug in sorted_slugs %}
            {% assign idx = forloop.index0 %}
            <li class="filter-option" data-tag="{{ slug }}" role="option" aria-selected="false">
              <span class="filter-option-label">{{ sorted_labels[idx] }}</span>
              <span class="filter-option-count">{{ sorted_counts[idx] }}</span>
            </li>
          {% endfor %}
          <li class="filter-empty" id="blogFilterEmpty">No matching tags</li>
        </ul>
      </div>
    </div>
  </header>

  <div class="post-list" id="postList">
    {% for post in site.posts %}
      {% assign post_tag_slugs = "" | split: "" %}
      {% if post.tags %}
        {% for raw_tag in post.tags %}
          {% assign tag_slug = raw_tag | strip | slugify %}
          {% if tag_slug != "" %}
            {% assign post_tag_slugs = post_tag_slugs | push: tag_slug %}
          {% endif %}
        {% endfor %}
      {% endif %}
      {% assign post_tags_attr = post_tag_slugs | join: " " %}

    <article class="post-card" data-tags="{{ post_tags_attr }}">
      {% if post.cover_image %}
      <a href="{{ post.url | relative_url }}" class="post-card-image">
        <img src="{{ post.cover_image | relative_url }}" alt="{{ post.title }}">
      </a>
      {% endif %}
      <div class="post-card-body">
        <time class="post-date">{{ post.date | date: "%B %-d, %Y" }}</time>
        {% if post.tags and post.tags.size > 0 %}
        <span class="post-tags">
          {% for tag in post.tags %}<span class="tag">{{ tag }}</span>{% endfor %}
        </span>
        {% endif %}
        <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
        <p class="post-excerpt">{{ post.excerpt | strip_html | truncatewords: 40 }}</p>
        <a href="{{ post.url | relative_url }}" class="read-more">Read more &rarr;</a>
      </div>
    </article>
    {% endfor %}

    {% if site.posts.size == 0 %}
    <p class="empty-state">No posts yet — check back soon!</p>
    {% endif %}
  </div>

  <p class="empty-state" id="blogEmptyState" hidden>No posts with this tag yet.</p>

</div>

<script>
(function () {
  var root        = document.getElementById('blogFilter');
  var toggle      = document.getElementById('blogFilterToggle');
  var toggleLabel = document.getElementById('blogFilterToggleLabel');
  var panel       = document.getElementById('blogFilterPanel');
  var search      = document.getElementById('blogFilterSearch');
  var options     = document.getElementById('blogFilterOptions');
  var emptyOption = document.getElementById('blogFilterEmpty');
  var optionEls   = Array.prototype.slice.call(options.querySelectorAll('.filter-option'));
  var items       = Array.prototype.slice.call(document.querySelectorAll('#postList .post-card'));
  var emptyState  = document.getElementById('blogEmptyState');

  if (!root || !toggle) { return; }

  function openPanel() {
    panel.hidden = false;
    toggle.setAttribute('aria-expanded', 'true');
    search.value = '';
    filterOptionList('');
    search.focus();
  }

  function closePanel() {
    panel.hidden = true;
    toggle.setAttribute('aria-expanded', 'false');
  }

  toggle.addEventListener('click', function () {
    if (panel.hidden) { openPanel(); } else { closePanel(); }
  });

  document.addEventListener('click', function (e) {
    if (!root.contains(e.target)) { closePanel(); }
  });

  document.addEventListener('keydown', function (e) {
    if (e.key === 'Escape') { closePanel(); toggle.focus(); }
  });

  function filterOptionList(query) {
    query = query.trim().toLowerCase();
    var visibleCount = 0;
    optionEls.forEach(function (opt) {
      var label = opt.querySelector('.filter-option-label').textContent.toLowerCase();
      var match = query === '' || label.indexOf(query) !== -1;
      opt.style.display = match ? '' : 'none';
      if (match) { visibleCount++; }
    });
    emptyOption.style.display = visibleCount === 0 ? '' : 'none';
  }

  search.addEventListener('input', function () {
    filterOptionList(search.value);
  });

  function applyFilter(tag, label) {
    toggleLabel.textContent = label;
    optionEls.forEach(function (opt) {
      opt.setAttribute('aria-selected', opt.getAttribute('data-tag') === tag ? 'true' : 'false');
    });

    var visibleItems = 0;
    items.forEach(function (item) {
      var show = tag === '';
      if (!show) {
        var itemTags = (item.getAttribute('data-tags') || '').split(/\s+/);
        show = itemTags.indexOf(tag) !== -1;
      }
      item.style.display = show ? '' : 'none';
      if (show) { visibleItems++; }
    });
    emptyState.hidden = visibleItems !== 0;
  }

  optionEls.forEach(function (opt) {
    opt.addEventListener('click', function () {
      var tag = opt.getAttribute('data-tag');
      var label = opt.querySelector('.filter-option-label').textContent;
      applyFilter(tag, label);
      closePanel();
    });
  });
})();
</script>
