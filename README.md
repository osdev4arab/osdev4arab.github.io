Bal6eem
=======

Bal6eem ia based on [Herring Cove](https://github.com/arnp/herring-cove), a clean and responsive theme for Jekyll.

### Overview 

* Good support for Arabic and other RTL languages
* Clean and blog-friendly
* Categories and tag support
* Easy to configure

### Screenshots

![screenshot](/images/screenshot1.png)
![screenshot](/images/screenshot2.png)

### Setup
#### in General
1. Install Jekyll
2. Fork or [download](https://github.com/a3f/bal6eem/archive/master.zip) this theme repo
3. Edit the `_config.yml` file
#### Github Pages
##### Organization or user pages
1. Create a {username}.github.io or {orgname}.github.io repository
2. Fork this project. Delete the master branch and rename the gh-pages to master
##### Project page
1. clone the gh-pages branch of this repository to your own project

### Use
#### Add a blog post
A new file needs to be created in the `\_posts/` directory with following schema: `yyyy-mm-dd-Some-Title.markdown`.

The post format is Github-flavored markdown, which is a strict superset of HTML, so you are free to mix them as you please. Every File has a YAML Front Matter at its start, with optional parameters that specify how the site is to be displayed. A full example	(## are comments):

```
---
title:  "Some title for your site"
date:   2013-11-08 19:55:16
categories: osdev bootloader
tags: c x86 real-mode
direction: ltr ## this is optional, if left out it defaults to rtl (right to left)
comments: true ## enable disqus comments?
post_author: a3f ## can be left out
post_gravatar: deadbeefdeadbdeefdeadbeefdeadbeef ## Get Gravatar avatar
---
```

#### Adding new tags
Tags are meant as keywords for your posts, if you want to add new tags, navigate to `_data/tags.yml`. A tag entry looks like this:
```
ia32:
	name: IA32
```
This can then be used to tag posts via the `tags:` YAML-Parameter. The tag overview page has to be created manually. For that duplicate any tag file in `/tag/*.md` and adjust the parameters. 

#### Adding new categories
If you want to add new categories, navigate to `_data/categories.yml`. A category entry looks like this:
```
linux:
	name: Linux
```
This can then be used to categorize posts via the `categories:` YAML-Parameter. The category overview page has to be created manually. For that duplicate any category file in `/category/*.md` and adjust the parameters. An overview page of all categories exists at `/categories.html` and is populated automatically according to posts in `/_posts` and categories in `_data/categories.yml`.


#### Right-to-Left

The default direction is RTL. If a Left-to-Right direction is wished for in a single blog post this can be done by adding this line in the YAML frontmatter (the parameters at the top of the blog post):
    
     direction: ltr


### License
* [MIT](http://opensource.org/licenses/MIT)

