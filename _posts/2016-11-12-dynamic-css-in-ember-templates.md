---
category: posts
mainTopic: "ember"
layout: single
title: "Dynamic CSS in Ember templates"
excerpt: "When you need a dirty workaround, Ember's templating engine can reactively spit out practically any markup a browser can understand."
header:
  image: "http://jurgen.si/images/dynamic-css-post/series-selector.png"
---

Recently I was challenged with a task to dynamically colorize checkboxes when they are selected. The colors are generated on-the-fly when the select action is triggered:

<iframe src="//giphy.com/embed/qeUvEToKvHGBa" width="400" height="250" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>

<small>Finished functionality. It’s a selector for chart series.</small>

Since we generate colors dynamically and thus can’t make predefined CSS classes that determine the checkbox’s color, I thought it should be enough to simply inline the color style, like so:

```handlebars
{% raw %}{{#each chartSeries as |series|}}
  <div class="checkbox">
    <input type="checkbox" style={{html-safe (concat 'color:' series.color)}} checked=… >
    <label for={{series.seriesId}}>
      {{series.name}}
    </label>
  </div>
{{/each}}{% endraw %}
```

<small>Random color is set to the the `series.color` property on checkbox change event.</small>

Due to the way these custom checkboxes are implemented, I learned this is not going to work. The place to define checkbox's background color is actually in the label’s `::before` pseudo selector:

<figure class="half">
    <a href="/images/dynamic-css-post/1.jpg"><img src="/images/dynamic-css-post/1.jpg"></a>
    <figcaption>You can't implement pseudo-element declarations using inline CSS.</figcaption>
</figure>

## Lesson 1: CSS is not designed to accept pseudo-element styles in the style attribute.

What can we do?

The only solution that came to my mind is to generate CSS classes dynamically. I naively wrote this piece of code in the template, and for some reason, I didn’t expect it to work:

```handlebars
{% raw %}<style>
  {{#each chartSeries as |series|}}
    input[type="checkbox"]#{{series.seriesId}}.styled:checked + label:before {
      background: {{series.color}};
      border-color: {{series.color}};
    }
  {{/each}}
</style>{% endraw %}
```

But it did. I was amazed by how Ember’s templating engine digests this without problems, and dynamically adds the necessary CSS declarations to the `style` tag when the `chartSeries` array changes:

<figure class="half">
    <a href="/images/dynamic-css-post/2.jpg"><img src="/images/dynamic-css-post/2.jpg"></a>
    <figcaption>The templating engine inserts some whitespace, but we can live with that.</figcaption>
</figure>

## Lesson 2: When you need a dirty workaround, Ember's templating engine can reactively spit out practically any markup a browser can understand.

Just be careful not to ruin your site’s performance.

In our case, every time the `chartSeries` array changes, the browser needs to recalculate style, which could lead to page clunkiness if we had a lot of elements and a lot of updates in a short amount of time.
