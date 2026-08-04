---
layout: default
title: Gallery
permalink: /gallery/
---

<div class="gallery-page container">

  {% comment %}
    ------------------------------------------------------------------
    Pass 1: work out which tags exist across all posts and how many
    images belong to each one, so the filter dropdown can be built
    (most images first) before the grid itself is rendered below.
    ------------------------------------------------------------------
  {% endcomment %}
  {% assign known_slugs   = "" | split: "," %}
  {% assign known_labels  = "" | split: "," %}
  {% assign tag_hits      = "" | split: "," %}
  {% assign counted_paths = "" | split: "" %}

  {% for post in site.posts %}
    {% assign post_imgs = "" | split: "" %}
    {% if post.cover_image %}
      {% assign post_imgs = post_imgs | push: post.cover_image %}
    {% endif %}
    {% if post.images %}
      {% for img in post.images %}
        {% assign post_imgs = post_imgs | push: img %}
      {% endfor %}
    {% endif %}

    {% comment %}Only count each image toward a tag's tally once{% endcomment %}
    {% assign new_imgs = "" | split: "" %}
    {% for img_src in post_imgs %}
      {% assign img_path = img_src | prepend: "/" %}
      {% unless counted_paths contains img_path %}
        {% assign counted_paths = counted_paths | push: img_path %}
        {% assign new_imgs = new_imgs | push: img_src %}
      {% endunless %}
    {% endfor %}

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
          {% for img in new_imgs %}
            {% assign tag_hits = tag_hits | push: tag_slug %}
          {% endfor %}
        {% endif %}
      {% endfor %}
    {% endif %}
  {% endfor %}

  {% assign total_image_count = counted_paths.size %}

  {% comment %}Sort tags by image count, descending (simple selection sort — the tag list is small){% endcomment %}
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
    <h1>Gallery</h1>

    <div class="gallery-filter" id="galleryFilter">
      <button type="button" class="filter-toggle" id="filterToggle" aria-haspopup="listbox" aria-expanded="false">
        <span class="filter-toggle-label" id="filterToggleLabel">All Tags</span>
        <span class="filter-toggle-caret" aria-hidden="true">&#9662;</span>
      </button>

      <div class="filter-panel" id="filterPanel" hidden>
        <input type="text" class="filter-search" id="filterSearch" placeholder="Search tags…" autocomplete="off" aria-label="Search tags">
        <ul class="filter-options" id="filterOptions" role="listbox">
          <li class="filter-option" data-tag="" role="option" aria-selected="true">
            <span class="filter-option-label">All Tags</span>
            <span class="filter-option-count">{{ total_image_count }}</span>
          </li>
          {% for slug in sorted_slugs %}
            {% assign idx = forloop.index0 %}
            <li class="filter-option" data-tag="{{ slug }}" role="option" aria-selected="false">
              <span class="filter-option-label">{{ sorted_labels[idx] }}</span>
              <span class="filter-option-count">{{ sorted_counts[idx] }}</span>
            </li>
          {% endfor %}
          <li class="filter-empty" id="filterEmpty">No matching tags</li>
        </ul>
      </div>
    </div>
  </header>

  <div class="gallery-grid gallery-grid--large" id="galleryGrid">
    {% assign all_images = site.static_files | where_exp: "file", "file.path contains '/assets/images/'" %}
    {% assign shown_paths = "" | split: "" %}

    {% comment %}Post-linked images, newest post first{% endcomment %}
    {% for post in site.posts %}
      {% assign post_imgs = "" | split: "" %}
      {% if post.cover_image %}
        {% assign post_imgs = post_imgs | push: post.cover_image %}
      {% endif %}
      {% if post.images %}
        {% for img in post.images %}
          {% assign post_imgs = post_imgs | push: img %}
        {% endfor %}
      {% endif %}

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

      {% for img_src in post_imgs %}
        {% assign img_path = img_src | prepend: "/" %}
        {% unless shown_paths contains img_path %}
          {% assign shown_paths = shown_paths | push: img_path %}
          <div class="gallery-item gallery-item--linked" data-tags="{{ post_tags_attr }}">
            <a href="{{ post.url | relative_url }}" title="{{ post.title }}">
              <img src="{{ img_src | relative_url }}" alt="{{ post.title }}" loading="lazy">
              <div class="gallery-overlay">
                <span class="gallery-overlay-text">{{ post.title }}</span>
              </div>
            </a>
          </div>
        {% endunless %}
      {% endfor %}
    {% endfor %}

    {% comment %}Standalone images not associated with any post{% endcomment %}
    {% for image in all_images %}
      {% unless shown_paths contains image.path %}
        <div class="gallery-item" data-tags="">
          <img src="{{ image.path | relative_url }}" alt="Gallery photo" loading="lazy">
        </div>
      {% endunless %}
    {% endfor %}

  </div>

  <p class="empty-state" id="galleryEmptyState" hidden>No photos with this tag yet.</p>

</div>

<script>
(function () {
  var root        = document.getElementById('galleryFilter');
  var toggle      = document.getElementById('filterToggle');
  var toggleLabel = document.getElementById('filterToggleLabel');
  var panel       = document.getElementById('filterPanel');
  var search      = document.getElementById('filterSearch');
  var options     = document.getElementById('filterOptions');
  var emptyOption = document.getElementById('filterEmpty');
  var optionEls   = Array.prototype.slice.call(options.querySelectorAll('.filter-option'));
  var items       = Array.prototype.slice.call(document.querySelectorAll('#galleryGrid .gallery-item'));
  var emptyState  = document.getElementById('galleryEmptyState');

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
