WADDLE Corporation Blog
========

Document
--------

* Get Started
	* [Environment](#environment)
	* [Get Started](#get-started)
	* [Write Posts](#write-posts)
* Components
	* [SideBar](#sidebar)
	* [Mini About Me](#mini-about-me)
	* [Featured Tags](#featured-tags)
	* [Friends](#friends)
	* [Keynote Layout](#keynote-layout)
* Comment & Analysis
	* [Comment](#comment)
	* [Analytics](#analytics)
* Advanced
	* [Build from source](#build-from-source)
	* [Header Image](#header-image)
	* [SEO Title](#seo-title)
	* [Page Build Warning](#page-build-warning)
* FAQ

### Get Started

If you have `npm` and `jekyll` installed, simply run `npm run start` from CLI and preview the themes at `localhost:4000` in your browser. It's watched and live-reloaded.


### Start to customize

You can easily customize the blog by modifying `_config.yml`:

```yml
# Site settings
title: Hux Blog             # title of your website
SEOTitle: Hux Blog          # check out docs for more detail
description: "Cool Blog"    # ...

# SNS settings      
github_username: huxpro     # modify this account to yours
weibo_username: huxpro      # the footer woule be auto-updated.

# Build settings
paginate: 10                # nums of posts in one page
```

For more options, please check out [Jekyll - Official Site](http://jekyllrb.com/). 
Most of them are very descriptive so feel brave to dive into code directly as well. 


### Writing Posts

Posts are simply Markdown files in the `_posts/`. 

Metadata of posts are written in **front-matter**. A example post could start with:

```yml
---
layout:     post
title:      "Hello 2015"
subtitle:   "Hello World, Hello Blog"
date:       2015-01-29 12:00:00
author:     "Hux"
header-img: "img/post-bg-2015.jpg"
tags:
    - Life
---
```

### SideBar

![](http://huangxuan.me/img/blog-sidebar.jpg)

**SideBar** provides possible modules to show off more personal information.

```yml
# Sidebar settings
sidebar: true   # default true
sidebar-about-description: "your description here"
sidebar-avatar: /img/avatar-hux.jpg     # use absolute URL.
```

Modules *[Featured Tags](#featured-tags)*, *[Mini About Me](#mini-about-me)* and *[Friends](#friends)* are turned on by default and you can add your own. The sidebar is naturally responsive, i.e. be pushed to bottom in a smaller screen (`<= 992px`, according to [Bootstarp Grid System](http://getbootstrap.com/css/#grid))  


### Mini About Me

**Mini-About-Me** displays your avatar, description and all SNS buttons if  `sidebar-avatar` and `sidebar-about-description` variables are set. 

It would be hidden in a smaller screen when the entire sidebar are pushed to bottom. Since there is already SNS portion there in the footer.

### Featured Tags

**Featured-Tags** is similar to any cool tag features in website like [Medium](http://medium.com).
Started from V1.4, this module can be used even when sidebar is off and displayed always in the bottom. 

```yml
# Featured Tags
featured-tags: true  
featured-condition-size: 1     # A tag will be featured if the size of it is more than this condition value
```

The only thing need to be paid attention to is `featured-condition-size`, which indicate a criteria that tags need to have to be able to "featured". Internally, a condition `{% if tag[1].size > {{site.featured-condition-size}} %}` are made.

### Friends

Friends is a common feature of any blog. It helps with SEO if you have a bi-directional hyperlinks with your friends sites.
This module can live when sidebar is off as well.

Friends information is configured as a JSON string in `_config.yml`

```yml
# Friends
friends: [
    {
        title: "Foo Blog",
        href: "http://foo.github.io/"
    },
    {
        title: "Bar Blog",
        href: "http://bar.github.io"
    }
]
```


### Keynote Layout

![](http://huangxuan.me/img/blog-keynote.jpg)

There is a increased trend to use Open Web technology for keynotes and presentations via Reveal.js, Impress.js, Slides, Prezi etc. I consider a modern blog should have first-class support to embed these HTML based presentation so **Keynote layout** are made.

To use, in the **front-matter**:

```yml
---
layout:     keynote
iframe:     "http://huangxuan.me/js-module-7day/"
---
```

The `iframe` element will be automatically resized to adapt different form factors and device orientation. 
Because most of the keynote framework prevent the browser default scroll behavior. A bottom-padding is set to help user and imply user that more content could be presented below.


### Comment

Currently, [Disqus](http://disqus.com) <del> and [Duoshuo](http://duoshuo.com)</del> are supported as third party discussion system.

First of all, you need to sign up and get your own account. **Repeat, DO NOT use mine!** (I have set Trusted Domains) It is deathly simple to sign up and you will get the full power of management system. Please give it a try!

Second, from V1.5, you can easily complete your comment configuration by just adding your **short name** into `_config.yml`:

```yml
duoshuo_username: _your_duoshuo_short_name_
# OR
disqus_username: _your_disqus_short_name_
```

**To the old version user**, it's better that you pull the new version, otherwise you have to replace code in `post.html`, `keynote.html` and `about.html` on your own.

<del>Furthermore, Duoshuo support Sharing. if you only wanna use Duoshuo comment without sharing, you can set `duoshuo_share: false`. </del>



### Analytics

From V1.5, Google Analytics and Baidu Tongji are supported with a simple config away:

```yml
# Baidu Analytics
ba_track_id: 4cc1f2d8f3067386cc5cdb626a202900

# Google Analytics
ga_track_id: 'UA-49627206-1'            # Format: UA-xxxxxx-xx
ga_domain: huangxuan.me
```

Just checkout the code offered by Google/Baidu, and copy paste here, all the rest is already done for you.

(Google might ask for meta tag `google-site-verification`)


### Build from source

More customization could be made by changing the source code. [Grunt](gruntjs.com) were used for building this blog. (Thanks to Clean Blog.)

There are numbers of tasks includes minifing JavaScript, compiling `.less` to `.css`, adding banners to keep the Apache 2.0 license intact, watching for changes, etc. Running `grunt ` to build files and `grunt watch` for watch-build.

Critical code are located in `_include/` and `_layouts/`. Most of them are simply Jekyll [Liquid](https://github.com/Shopify/liquid/wiki) template.


### Header Image

Change header images of any pages or any posts is pretty easy as mentioned above. But, thanks to [issue #6 (in Chinese)](https://github.com/Huxpro/huxpro.github.io/issues/6) asked, **how to make it looks great?**

**Well...it is actually a design issue**, not a coding stuff. It is better that you have basic design knowledge, but not is ok, let me told you how to make it well-designed:

Seeing the title text above image is **white**, the image should be **dark** to emphasize the contract. so we can easily add a **black overlay with fews of opacity**, which is depends on the brightness of the original images you used. you can process it in Photoshop, Sketch etc.

In technical views, it can be done with CSS. However, the opacity of the black overlay is really hard to assigned, **every image has different brightness so the  degree it should be adjusted is different so it is impossible to hard code it.**


### SEO Title

Before V1.4, site setting `title` is not only used for displayed in Home Page and Navbar, but also used to generate the `<title>` in HTML.
It's possible that you want the two things different. For me, my site-title is **“Hux Blog”** but I want the title shows in search engine is **“黄玄的博客 | Hux Blog”** which is multi-language.

So, the SEO Title is introduced to solve this problem, you can set `SEOTitle` different from `title`, and it would be only used to generate HTML `<title>` and setting DuoShuo Sharing.

### Page Build Warning

There are many possible reasons to cause a "Page Build Warning" email or similar error.

One of these is that github changes its build environment.

> You are attempting to use the 'pygments' highlighter, which is currently unsupported on GitHub Pages. Your site will use 'rouge' for highlighting instead. To suppress this warning, change the 'highlighter' value to 'rouge' in your '_config.yml'.

So, just edit `_config.yml`, find `highlighter: pygments`, change it to `highlighter: rouge` and the warning will be gone.

For other circumstances, check out existing issues or create a new one!



FAQ
---

### How can I customize the theme of code block?

This theme uses the default code syntax highlighter of jekyll, "rouge, which is compatible with Pygments theme so just pick any pygments theme css (e.g. from [here](http://jwarby.github.io/jekyll-pygments-themes/languages/javascript.html) and replace the content of `highlight.less`.

### cannot load such file -- jekyll-paginate

Executing this command to install this plugin:

```yml
$ gem install jekyll-paginate 
```

This blog started in Jekyll 2 time when `jekyll-paginate` is standard. With Jekyll 3, it's a plugin we included in `_config.yml`.



License
-------

Apache License 2.0.
Copyright (c) 2015-2020 Huxpro

Hux Blog is derived from [Clean Blog Jekyll Theme (MIT License)](https://github.com/BlackrockDigital/startbootstrap-clean-blog-jekyll/)
Copyright (c) 2013-2016 Blackrock Digital LLC.
