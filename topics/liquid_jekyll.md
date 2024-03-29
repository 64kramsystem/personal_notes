# Liquid/Jekyll

- [Liquid/Jekyll](#liquidjekyll)
  - [Liquid](#liquid)
    - [Syntax](#syntax)
    - [Strings](#strings)
    - [Timestamps](#timestamps)
    - [Collections (Array)](#collections-array)
    - [General functions](#general-functions)
  - [Jekyll](#jekyll)
    - [(Some) Metadata](#some-metadata)
    - [Using URLs](#using-urls)
    - [Define site-wide collections](#define-site-wide-collections)
    - [Redirect old pages](#redirect-old-pages)

## Liquid

### Syntax

Object (show content): `{{ <content> }}`
Tags (execute logic); one statement per tag:  `{% <logic> %}`

Filters (process an object):

    object | filter

For cycle:

    for post in site.posts
    endfor

[Control flow](https://help.shopify.com/en/themes/liquid/tags/control-flow-tags):

    if post.tags contains page.slug
    elif tagsList != ""
    endif

[Operators](https://help.shopify.com/en/themes/liquid/basics/operators):

    and
    or
    <string> contains <substring>
    <strings_array> contains <string>

Comment out content:

    {% comment %} <ignored> {% endcomment %}

[Variables](https://shopify.github.io/liquid/tags/variable):

    assign myStatement = "frontend development"
    {% capture myStatement %}{{ myStatement }} sucks {% endcapture %}     # compose strings

In order to escape double opening curly braces, treat them as strings:

    key: ${{ '{{' }} runner.os }}-cargo-${{ '{{' }} hashFiles('**/Cargo.lock') }}

### Strings

String operations:

    capitalize
    split: <string>
    append: <string>
    prepend: <string>

### Timestamps

Format a timestamp (example):

    date: "%B %e, %Y"

### Collections (Array)

Array handling is quite awkward. Don't split via empty string!

    assign myArray = 'a,b,c' | split: ','        # create
    assign newEntry = 'd' | split: ','           # new entry for append
    assign myArray = myArray | concat: newEntry  # append a new entry

Access:

    myArray[0]
    myArray.last

Useful operations:

    join: "concatenator"
    map: "myField"
    concat: otherArray
    sort: "myField"
    uniq

### General functions

Reference to a page:

    {% post_url 2017-11-08-Shell-scripting-adventures-part-1 %}

## Jekyll

### (Some) Metadata

- `page.date`
- `page.title`
- `page.url`        : relative!
- `site.baseurl`
- `site.description`
- `site.name`
- `site.url`
- `site.baseurl`    : baseurl, when set
- `site.posts`      : important! collection of posts


### Using URLs

WATCH OUT!! If there is a baseurl, links must be represented as `{{ site.baseurl }}{{ instance.url }}`.

See https://mademistakes.com/mastering-jekyll/site-url-baseurl.

### Define site-wide collections

Reference: https://jekyllrb.com/docs/collections/

Sample, to be defined in `_config.yml`:

```
collections:
  tags:
    output: true
    permalink: /tag/:name/
  books: {}
```

Each collection entry is a document in a directory with name `_<collection` (e.g. `_tags`).

### Redirect old pages

Use `jekyll-redirect-from` gem, and add this to the front matter:

```yml
redirect_from:
- Quickly-building-a-custom-linux-ubuntu-kernel-with-modified-configuration-kernel-timer-frequency
- Quickly-building-a-custom-linux-ubuntu-kernel-with-modified-configuration-kernel-timer-frequency/
```

WATCH OUT! The trailing slash makes no difference locally, but it does on GH Pages!
