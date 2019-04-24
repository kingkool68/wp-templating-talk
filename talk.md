# From Template Tags to Twig: A Journey Through WordPress Templating

Today I want to walk you through the journey that I took with WordPress templating. I've been doing WordPress development for 12 years. One weekend I sat down and decided I wanted to figure out this whole WordPress theme. I took the default theme [Kubrik](https://github.com/Heilemann/kubrick-for-wordpress) and poked and prodded util it matched the look and feel of the rest of my site. And then once I was done, I decided to start blogging. What a thought. So that's how I got started. I took the default theme, made a copy, and started experimenting. 

Now WordPress doesn't have it's own templating language like other content management systems do. WordPress uses Template Tags and the Template Hierarchy to turn dynamic data into web pages. 

## Template Tags

Template tags look like `the_content()` and `the_title()`. They're just PHP functions that echo out content. If you need to get the value and use it in PHP for something you can use `get_the_content()` or `get_the_title()`. Notice the pattern here. In WordPress functions that begin with `get_the_*` return data for use within PHP. And `the_*` echos the contents to the screen. I feel like this is very approachable as a beginner. It's easy to understand what is happening with its simple language. 

## Template Hierarchy

WordPress' template hierarchy determines what file to load for a given request.

## More Advanced

As you get more advanced your WordPress templates start to get more complicated. I think a natural evolution is to just put extra PHP logic right inline of the HTML where it is eventually rolled out. At the time it probably made sense to do it this way because that is where the custom logic was going to be outputted in the page. In hindsight, it results in some messy, hard-to follow code if you want other people to eventually read it and understand what you did. This realization probably doesn't happen until you are the one stuck reading through the convoluted code trying to make sense of what it does. 

The next step in the WordPress templating journey was realizing I should put the PHP logic parts at the top of the file and leave the HTML rendering at the bottom of the PHP file. 

## Escaping

One thing that WordPress does under the hood with template tags is it handles escaping for you. What is escaping?

> Escaping is the process of securing output by stripping out unwanted data, like malformed HTML or script tags, preventing this data from being seen as code.
>
> <https://developer.wordpress.org/themes/theme-security/data-sanitization-escaping/#escaping-securing-output>

It's important that we secure the data we output. For example, we might have a variable that we expect to be a `$url`. But at some point in our script that variable might change or it might be passed through a WordPress filter and who knows what the value might be if a malicious plugin is installed. 

{{ Need more details about escaping }}

## Twig

Twig is a PHP templating engine. A templating engine is a system that combines dynamic data with a static template used to generate final web pages to serve to browsers. You write your template using a special syntax that Twig understands and Twig handles turning that template into regular PHP that can be understood and rendered by your server. 

Here's an example:

```php
<?php echo $foo; ?>
``` 

```twig
{{ foo }}
```

You can do conditional logic and looping in Twig too just like you would do in PHP

```php
<?php foreach ( $things as $thing ) { ?>
  <div class="thing">
     <h1><?php echo $thing; ?></h1>
  </div>
<?php } ?>
```

```twig
{% for thing in things %}
  <div class="thing">
     <h1>{{ thing }}</h1>
  </div>
{% endfor %}
```

As we can see right off the bat writing Twig is more consice. 

When I first heard about Twig and using a separate templating engine from PHP I thought it was dumb. Why bother with a slightly different syntax for writing templates just because its a little bit less to type? I mean you could always just use PHP short tags:

```php
<?= $foo ?>
```
**Note: Note every server has short tags enabled. Though it should be enabled by default since PHP 5.4 and later**

See https://www.php.net/manual/en/migration54.new-features.php

Then you start seeing Twig templates like this:

```twig
{% set currentPageNum = paginationCurrentPageNumber %}
{% set totalItems     = paginationTotalItems %}
{% set itemsPerPage   = paginationResultsPerPage %}
{% set pageCount      = (totalItems / itemsPerPage)|round(0, 'ceil') %}
{% set pageRange      = 5 %}

{% if pageCount < currentPageNum %}
    {% set currentPageNum = pageCount %}
{% endif %}

{% if pageRange > pageCount %}
    {% set pageRange = pageCount %}
{% endif %}

{% set delta = (pageRange / 2)|round(0, 'ceil') %}

{% if (currentPageNum - delta) > (pageCount - pageRange) %}

    {% set pages = range((pageCount - pageRange + 1), pageCount) %}

{% else %}

    {% if (currentPageNum - delta) < 0 %}

        {% set delta = currentPageNum %}
    {% endif %}

    {% set offset = currentPageNum - delta %}
    {% set pages  = range(offset + 1, offset + pageRange) %}
{% endif %}
```

That sure looks like a lot of logic that I could be doing in PHP. What's the point of a template engine if you're going to make it work just like PHP?

It turns out I didn't hate Twig. I just hate the way that I kept seeing it being used. 

And then I stumbled across an article by Carl Alexander that made Twig click for me. And he was just using regular PHP! https://carlalexander.ca/designing-class-generate-wordpress-html-content/

Carl talks about using a standalone PHP file as a template whose sole job is to generate the HTML using the provided data. He only sent the data that the template needed. This makes the template simpler as you don't need to worry about where all of the data for that template is coming from. Just figure out where the template is being called and then you can see all of the data being exposed to the template. Logic, data transformation, data fetching were being handled separatly from templating and HTML generation. A clear separation of concerns.

## Timber

If you want to start working with Twig in WordPress you will no doubt encounter Timber. Timber is a project from the agency Upstatement that unites Twig with WordPress. Add a plugin and then start writing Twig files. Timber also does a lot of work around massaging WordPress data into Twig. For example:

```php
$context = Timber::get_context();
$context['post'] = new TimberPost(); // It's a new TimberPost object, but an existing post from WordPress.
Timber::render('single.twig', $context);
``` 

```twig
<article>
  <h1 class="headline">{{post.post_title}}</h1>
  <div class="body">
    {{post.content}}
  </div>
</article>
```

Timber gets the job done. My biggest gripe with it is how bloated it is for the way I want to use Twig. Timber includes 132 PHP files for a total of 18,288 lines of code. 

## Conclusion

 - Aim to seperate data from your templates as much as possible
 - Don't tie your templates to their data source