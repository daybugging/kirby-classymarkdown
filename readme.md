# Classy Markdown

A [Kirby](https://getkirby.com) plugin for adding classes to the markup generated by
the built-in Markdown parser.

## What is it good for?

When dealing with complex content, i.e. nested components inside your [Markdown](https://daringfireball.net/projects/markdown/)-formatted texts, you often face the problem of [side effects](http://philipwalton.com/articles/side-effects-in-css/). To avoid this, a lot of developers use BEM-Naming convention these days. That basically means, that every element you want to target with a CSS-selector has it’s own, unique class. See example below for details.

## Example

Okay, but how can this plugin improve my life? A typical example, of what could get wrong, when dealing with the class-less output of a Markdown parser:

```css
.content a {
    border-bottom: 2px solid red;
}
```

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

You’ll end up with a red-underlined image, which is probably not, what your intention was. It gets even worse, when you have more complex components within your content area. Imagine a tabpane, using `<ul>` and `<li>` tags. *ClassyMarkdown* adds classes to anchor tags and lists (and other elements), generated by Markdown:

```html
<div class="content">
    <p>A nice introduction with a link to another <a href="http://getkirby.com" class="anchor">website</p>
    <figure>
        <a href="image-full.jpg">
            <img src="thumbnail.jpg" alt="A nice image">
        </a>
    </figure>
</div>
```

Now, you can distinguish Markdown-hyperlinks from other `<a>` tags, generated by – for example – a Kirbytag:

```css
.anchor {
    border-bottom: 2px solid red;
}
```

## Download & Requirements

### Requirements

- PHP 5.4+
- Kirby 2.2+
- Your content is written in Markdown [which it probably is, if you’re a Kirby-addict ;-) ]

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
<h2 class="graf--h2">Heading</h2><!-- h1-h6 -->
<hr class="graf--hr">
<ul class="list--ul">
<ol class="list--ol">
<li class="list__item">
<em class="markup--em">
<strong class="markup--strong">
<del class="markup--strikethrough">
<img src="…" class="markup--image">
<a href="http://example.com/" class="anchor">A link</a>
<a href="http://example.com/" class="anchor anchor--url">http://example.com/</a>
<a href="mailto:john@example.com" class="anchor anchor--email">Mail me!</a>
```

You can use these classes to wrap links to long URLs (nobody likes horizontal scrolling, right?) or whatever you like:

```css
.anchor--url {
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

  // Block Elements
  'paragraph'           => '',
  'blockquote'          => '',
  'header'              => '{prefix}graf--{name}',
  'code.block'          => '',
  'rule'                => '{prefix}graf--{name}',

  'list'                => '{prefix}list--{name}',
  'list.item'           => '{prefix}list__item',

  // Inline Elements
  'emphasis'            => '{prefix}markup--{name}',
  'strikethrough'       => '{prefix}markup--strikethrough',
  'link'                => '{prefix}markup--{name}',
  'image'               => '{prefix}markup--image',
  'code.inline'         => '',

  // Hyperlinks
  'link.base' => '{prefix}anchor',
  'link.url'  => '{link.base} {link.base}--url',
  'email'     => '{link.base} {link.base}--email',

  'link' => function($Link, $parser) {
    if ($Link['element']['attributes']['href'] === $Link['element']['text']) {
      return $parser->settings['link.url'];
    } else {
      return $parser->settings['link.base'];
    }
  },
]);
```

Every variable, which is not a closure can be used as a template variable in your class name string. You can also add custom variables to the configuration array, which can be used as placeholders in other class names.

## License

ClassyMarkdown is released under the MIT License (MIT).

## Credits

ClassyMarkdown is developed by [Fabian Michael](https://fabianmichael.de).

The plugin itself does not contain any third-party libraries, but relies on
[Parsedown](https://github.com/erusev/parsedown) and [ParsedownExtra](https://github.com/erusev/parsedown-extra) by [Emanuil Rusev](http://erusev.com), whose are part of Kirby.