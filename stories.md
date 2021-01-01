---
layout: page
title: Stories
order: 4
description: "A collection of short stories experienced by Tariq Hawis in his daily life"
permalink: /stories/index.html
image: /img/stories-hero.jpg
hero_image: /img/stories-hero.jpg
---

<div class="container">
    <section class="section">
        <div class="columns is-multiline">
            <div class="column is-12">
                <div class="content">
                    <p>A collection of short stories I have experienced in my life and I would like to share it with you</p>
                </div>
            </div>
            {% for item in site.stories %}
            <div class="column is-4">
                <div class="card">
                    <div class="card-image">
                        <img src="{{ item.hero_image }}" alt="{{ item.title }}" />
                    </div>
                    <div class="card-content">
                            <p class="title is-5">{{ item.title }}</p>
                        <div class="content">
                            <p>{{ item.description }}</p>
                        </div>
                    </div>
                    <footer class="card-footer">
                        <p class="card-footer-item">
                        <span>
                            <a href="{{ item.url }}" class="button is-info">Read Story</a>
                        </span>
                        </p>
                    </footer>
                </div>

            </div>
            {% endfor %}
        </div>
    </section>
</div>


