## contributing

1. fork this repo.
2. submit a pr.

## setup instructions

1. clone repository to your computer.
2. using terminal, navigate to the `perfect-jekyll` directory with `cd`.
3. in the directory type `bundle exec jekyll serve --baseurl ''`.
4. visit [http://127.0.0.1:4000](http://127.0.0.1:4000) in your browser.
5. to work with the css: `cd assets/stylesheets` and run `sass --watch style.sass:style.css`.

## adding posts

just add a new `YYYY-MM-DD-your-post-name.md` file in the `_posts` directory. note that the name of the file will become the url and you must follow the date convention. at the top of the file, add this frontmatter:

```
---
layout: post
title:  "your post title"
author: "@your_name"
date:   2020-12-01
---
```

## adding pages

just add a new `your-page-name.md` file in the `_pages` directory. at the top of the file, add this frontmatter:

```
---
layout: page
title: "your page name"
permalink: /your-page-name/
---
```

## Navigation

configure main navigation with `_data/nav.yml`.