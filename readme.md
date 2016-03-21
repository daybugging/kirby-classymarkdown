# ClassyMarkdown

A [Kirby](https://getkirby.com) plugin for adding classes to the markup generated by
the built-in Markdown parser.

Current version: `0.1.3`

## What is it good for?

When dealing with complex content, i.e. nested components inside your [Markdown](https://daringfireball.net/projects/markdown/)-formatted texts, you often face the problem of [side effects](http://philipwalton.com/articles/side-effects-in-css/). To avoid this, a lot of developers use BEM-Naming convention these days. That basically means, that every kind of element you want to target with a CSS-selector has it’s own, unique classes. See example below for details.

## Example

Okay, but how can this plugin improve my life? A typical example, of what could get wrong, when dealing with the class-less output of a Markdown parser:

```html
<div class="content">
    <p>A nice introduction with a link to another <a href="http://getkirby.com">website</p>
    <figure>
        <a href="image-full.jpg">
            <img src="thumbnail.jpg" alt="A nice image">
        </a>
    </figure>
</div>
```

```css
.content a {
    border-bottom: 2px solid red;
}
```

You’ll end up with a red-underlined image, which is probably not, what your intention was. It gets even worse, when you have more complex components within your content area. Imagine a tabpane, using `<ul>` and `<li>` tags. *ClassyMarkdown* adds classes to anchor tags and lists (and other elements), generated by Markdown:

```html
<div class="content">
    <p>A nice introduction with a link to another <a href="http://getkirby.com" class="markup-link">website</p>
    <figure>
        <a href="image-full.jpg">
            <img src="thumbnail.jpg" alt="A nice image" class="markup--image">
        </a>
    </figure>
</div>
```

Now, you can distinguish Markdown-hyperlinks from other `<a>` tags, generated by – for example – a Kirbytag:

```css
.markup--link {
    border-bottom: 2px solid red;
}
```

## Download & Requirements

### Requirements

- PHP 5.4+
- Kirby 2.2+
- Your content is written in Markdown (which it probably is, if you’re a Kirby-addict ;-))

### Git Submodule

To install this plugin as a git submodule, execute the following command from the root of your kirby project:

```
$ git submodule add https://github.com/fabianmichael/kirby-classymarkdown.git site/plugins/classymarkdown
```

### Copy & Paste

1. Download the contents of this repository as ZIP-file.
2. Rename the downloaded folder to »classymarkdown« and copy it
into the `site/plugins/` directory in your Kirby project.

## Usage

The plugin has to be enabled via your `config.php` file:

```php
c::set('classymarkdown', true);
```

ClassyMarkdown just hooks into the Markdown parser fo Kirby, so you don’t need to make any changes to you template code. It will only be activated, if you have not set a custom Markdown parser (via `markdown.parser`) in your config file.

By default, the following classes will be added:

```html
<h2 class="graf--h2"><!-- h1-h6 -->
<blockquote class="graf--blockquote">
<pre class="graf--codeblock">
<hr class="graf--hr" />

<ul class="list list--ul">
  <li class="list__item">
<ol class="list list--ol">
  <li class="list__item">

<em class="markup--em">
<strong class="markup--strong">
<img src="…" class="markup--image">
<del class="markup--strikethrough">
<code class="markup--code">

<a href="http://example.com/" class="markup--link">A link</a>
<a href="http://example.com/" class="markup--link markup--link--url">http://example.com/</a>
<a href="mailto:john@example.com" class="markup--link markup--link--email">Mail me!</a>

<table class="table">
  <thead class="table__head">
    <tr class="table__row table__row--head">
    <th class="table__cell table__cell--head">
    <th class="table__cell table__cell--head / align--right"><!-- "/" used as a separator between BEM and utility classes-->
  <tbody class="table__body">
    <tr class="table__row table__row--body">
    <td class="table__cell table__cell--body">
    <td class="table__cell table__cell--body / align--center"><!-- "/" used as a separator between BEM and utility classes-->

<!-- Markdown Extra Features -->

<dl class="definition">
  <dt class="definition__term">
  <dd class="definition__description">

<!-- Footnote Marker -->
<sup class="footnote">
  <a href="#fn:1" class="footnote__link">

<!-- Footnotes List -->
<div class="footnotes">
  <hr class="graf--hr" />
  <ol class="list list--ol">
    <li id="fn:1" class="list__item">
      <a href="#fnref1:1" rev="footnote" class="footnotes__backref">
```

You can use these classes to wrap links to long URLs (nobody likes horizontal scrolling, right?) or whatever you like:

```css
.markup--link--url {
    word-wrap: break-word;
}

.graf--hr {
  border-width: 0 0 2px 0;
  border-style: solid;
  margin-top: 1.5em;
  margin-bottom: 1.5em;
}
```

## Custom Configuration

Set `classymarkdown.classes` in your config file to set custom classes. The default configuration is shown below:

```php
c::set('classymarkdown.classes', [

  'prefix'              => '',
  'utility.align'       => 'align--{%align%}',

  // Markdown Features:

  // Block Elements
  'paragraph'           => '',
  'header'              => '{prefix}graf--{%name%}', // Available placeholders: {%level%} = 1-6
  'blockquote'          => '{prefix}graf--blockquote',
  'code.block'          => '{prefix}graf--codeblock',
  'rule'                => '{prefix}graf--{%name%}',

  'list.base'           => '{prefix}list',
  'list'                => '{list.base} {list.base}--{%name%}',
  'list.item'           => '{list.base}__item', // use {%parent%} for list-type

  // Inline Elements
  'emphasis'            => '', // Example: {prefix}markup--{%name%} => markup--em / markup--strong
  'strikethrough'       => '',
  'image'               => '{prefix}markup--image',
  'code.inline'         => '',

  // Hyperlinks
  'link.base'           => '{prefix}markup--link',
  'link.url'            => '{link.base} {link.base}--url',
  'email'               => '{link.base} {link.base}--email',

  'link'                => function($Link, $parser) {
    // A closure can return a template string with placeholders, but cannot
    // be used as a placeholder.
    if ($Link['element']['attributes']['href'] === $Link['element']['text']) {
      return $parser->settings['link.url'];
    } else {
      return $parser->settings['link.base'];
    }
  },

  // Tables
  'table.base'               => '{prefix}table',
  'table.cell.base'          => '{table.base}__cell',
  'table'                    => '{table.base}',

  'table.body'               => '{table.base}__body',
  'table.body.row'           => '{table.base}__row {table.base}__row--body',
  'table.body.cell'          => '{table.cell.base} {table.cell.base}--body',
  'table.body.cell.aligned'  => '{table.body.cell} / {utility.align}',
  'table.head'               => '{table}__head',
  'table.head.row'           => '{table.base}__row {table.base}__row--head',
  'table.head.cell'          => '{table.cell.base} {table.cell.base}--head',
  'table.head.cell.aligned'  => '{table.head.cell} / {utility.align}', // leave empty, to keep style attribute (text-align: …)

  // Markdown Extra Features:

  'abbreviation'             => '',

  // Definition Lists

  'definition.base'          => '{prefix}definition',
  'definition.list'          => '{definition.base}',
  'definition.term'          => '{definition.base}__term',
  'definition.description'   => '{definition.base}__description',

  // Footnotes                                                // ParsedownExtra Defaults:

  'footnote.marker.wrapper'  => '{prefix}footnote',           // -
  'footnote.marker.link'     => '{prefix}footnote__link',     // footnote-ref

  'footnotes.base'           => '{prefix}footnotes',          // n/a
  'footnotes.container'      => '{footnotes.base}',           // footnotes
  'footnotes.separator'      => '{rule}',                     // -
  'footnotes.list'           => '{list}',                     // -
  'footnotes.list.item'      => '{list.item}',                // -
  'footnotes.backref'        => '{prefix}footnotes__backref', // footnote-backref
]);
```

Every variable, which is not a closure can be used as a template variable in your class name string. You can also add custom variables to the configuration array, which can be used as placeholders in other class names.

## License

ClassyMarkdown is released under the MIT License (MIT).

## Credits

ClassyMarkdown is developed by [Fabian Michael](https://fabianmichael.de).

The plugin itself does not contain any third-party libraries, but relies on
[Parsedown](https://github.com/erusev/parsedown) and [ParsedownExtra](https://github.com/erusev/parsedown-extra) by [Emanuil Rusev](http://erusev.com), whose are part of Kirby.
