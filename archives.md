---
layout: base
title: Archives
permalink: /archives/
---

{%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}


<h2 class="archive-heading">
  {% include icons/folder.svg %}
  Categories
</h2>

{%- assign parents = site.posts | group_by_exp: "post", "post.categories[0]" | sort: "name" -%}

{%- for parent in parents -%}
  {%- assign parent_name  = parent.name | default: "uncategorized" -%}
  {%- assign parent_posts = parent.items -%}

  {%- assign children_groups = parent_posts
      | where_exp: "p", "p.categories.size > 1"
      | group_by_exp: "p", "p.categories[1]"
      | sort: "name"
  -%}

  {%- assign only_parent_posts = parent_posts | where_exp: "p", "p.categories.size == 1" -%}

  <details id="cat-{{ parent_name | slugify }}" class="archive-section">
    <summary>
      <span>{{ parent_name }}</span>

      <div class="contributions">
        <div class="square">
          {%- assign blk = parent_posts | size -%}
          {%- assign rest  = blk | modulo: 10 -%}
          {%- assign packs = blk | divided_by: 10 -%}
          {%- for i in (1..rest) -%}
            <div class="square__unit" title="1 post"></div>
          {%- endfor -%}
          {%- for i in (1..packs) -%}
            <div class="square__unit square__unit--pack" title="10 posts"></div>
          {%- endfor -%}
        </div>

        <span class="square-size">{{ parent_posts | size }}</span>
      </div>
    </summary>

    <ul class="post-list">
      {%- assign parent_sorted = parent_posts | sort: "date" | reverse -%}
      {%- for post in parent_sorted -%}
        <li>
          <span class="post-meta">
            {%- include post-dates.html item=post -%}

            {%- if post.tags -%}
              <span class="post-meta__item">
                {% include icons/tag.svg %}
                {{ post.tags | join: ", " }}
              </span>
            {%- endif -%}

            {%- if post.categories -%}
              <span class="post-meta__item">
                {% include icons/folder.svg %}
                {{ post.categories | join: ", " }}
              </span>
            {%- endif -%}
          </span>
          <h3>
            <a href="{{ post.url | relative_url }}">
              {{ post.title | escape }}
            </a>
          </h3>
        </li>
      {%- endfor -%}
    </ul>
  </details>
  {%- for child in children_groups -%}
    {%- assign child_name = child.name -%}
    {%- assign child_posts = child.items -%}

    <details id="cat-{{ parent_name | slugify }}-{{ child_name | slugify }}" class="archive-section child-group">
      <summary>
        <span>{{ child_name }}</span>

        <div class="contributions">
          <div class="square">
            {%- assign blk = child_posts | size -%}
            {%- assign rest  = blk | modulo: 10 -%}
            {%- assign packs = blk | divided_by: 10 -%}
            {%- for i in (1..rest) -%}
              <div class="square__unit" title="1 post"></div>
            {%- endfor -%}
            {%- for i in (1..packs) -%}
              <div class="square__unit square__unit--pack" title="10 posts"></div>
            {%- endfor -%}
          </div>

          <span class="square-size">{{ child_posts | size }}</span>
        </div>
      </summary>

      <ul class="post-list">
        {%- for post in child_posts -%}
          <li>
            <span class="post-meta">
              {%- include post-dates.html item=post -%}

              {%- if post.tags -%}
                <span class="post-meta__item">
                  {% include icons/tag.svg %}
                  {{ post.tags | join: ", " }}
                </span>
              {%- endif -%}

              {%- if post.categories -%}
                <span class="post-meta__item">
                  {% include icons/folder.svg %}
                  {{ post.categories | join: ", " }}
                </span>
              {%- endif -%}
            </span>
            <h3>
              <a href="{{ post.url | relative_url }}">
                {{ post.title | escape }}
              </a>
            </h3>
          </li>
        {%- endfor -%}
      </ul>
    </details>
  {%- endfor -%}
{%- endfor -%}


<h2 class="archive-heading archive-heading--spaced">
  {% include icons/tag.svg %}
  Tags
</h2>

{%- assign tags = site.tags | sort -%}

{%- for tag in tags -%}
  {%- assign tag_name = tag[0] -%}
  {%- assign posts_in_tag = tag[1] | sort: "date" | reverse -%}

  <details id="tag-{{ tag_name | slugify }}" class="archive-section">
    <summary>
      <span>{{ tag_name }}</span>
    
      <div class="contributions">
        <div class="square">
          {%- assign blk = posts_in_tag | size -%}
          {%- assign rest  = blk | modulo: 10 -%}
          {%- assign packs = blk | divided_by: 10 -%}
          {%- for i in (1..rest) -%}
            <div class="square__unit" title="1 post"></div>
          {%- endfor -%}
          {%- for i in (1..packs) -%}
            <div class="square__unit square__unit--pack" title="10 posts"></div>
          {%- endfor -%}
        </div>

        <span class="square-size">{{ posts_in_tag | size }}</span>
      </div>
    </summary>

    <ul class="post-list">
      {%- for post in posts_in_tag -%}
        <li>
          <span class="post-meta">
            {%- include post-dates.html item=post -%}

            {%- if post.tags -%}
              <span class="post-meta__item">
                {% include icons/tag.svg %}
                {{ post.tags | join: ", " }}
              </span>
            {%- endif -%}

            {%- if post.categories -%}
              <span class="post-meta__item">
                {% include icons/folder.svg %}
                {{ post.categories | join: ", " }}
              </span>
            {%- endif -%}
          </span>
          <h3>
            <a href="{{ post.url | relative_url }}">
              {{ post.title | escape }}
            </a>
          </h3>
        </li>
      {%- endfor -%}
    </ul>
  </details>
{%- endfor -%}


<h2 class="archive-heading archive-heading--spaced">
  {% include icons/calendar.svg %}
  Years
</h2>

{%- assign posts_by_year = site.posts | group_by_exp: "post", "post.date | date: '%Y'" -%}
{%- assign sorted_years = posts_by_year | sort: "name" | reverse -%}

{%- for group in sorted_years -%}
  <details id="year-{{ group.name }}" class="archive-section">
    <summary>
      <span>{{ group.name }}</span>
    
      <div class="contributions">
        <div class="square">
          {%- assign blk = group.items | size -%}
          {%- assign rest  = blk | modulo: 10 -%}
          {%- assign packs = blk | divided_by: 10 -%}
          {%- for i in (1..rest) -%}
            <div class="square__unit" title="1 post"></div>
          {%- endfor -%}
          {%- for i in (1..packs) -%}
            <div class="square__unit square__unit--pack" title="10 posts"></div>
          {%- endfor -%}
        </div>

        <span class="square-size">{{ group.items | size }}</span>
      </div>
    </summary>

    <ul class="post-list">
      {%- for post in group.items -%}
        <li>
          <span class="post-meta">
            {%- include post-dates.html item=post -%}

            {%- if post.tags -%}
              <span class="post-meta__item">
                {% include icons/tag.svg %}
                {{ post.tags | join: ", " }}
              </span>
            {%- endif -%}

            {%- if post.categories -%}
              <span class="post-meta__item">
                {% include icons/folder.svg %}
                {{ post.categories | join: ", " }}
              </span>
            {%- endif -%}
          </span>
          <h3>
            <a href="{{ post.url | relative_url }}">
              {{ post.title | escape }}
            </a>
          </h3>
        </li>
      {%- endfor -%}
    </ul>
  </details>
{%- endfor -%}


<script>
  (function () {
    function openFromHash() {
      var hash = location.hash && location.hash.slice(1);
      if (!hash) return;
      var el = document.getElementById(hash);
      if (el && el.tagName.toLowerCase() === 'details') el.open = true;
    }
    openFromHash();
    window.addEventListener('hashchange', openFromHash);
  })();
</script>
