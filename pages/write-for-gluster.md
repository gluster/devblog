---
layout: page
title: Write for Gluster
permalink: /write-for-gluster
---

Are you interested in write blog post in Gluster devblog? Why wait,
clone the [devblog](https://github.com/gluster/devblog) repo and start
writing. This post will help you to get started.

```
git clone git@github.com:gluster/devblog.git
cd devblog
```

First time users need to add Author information by running
`./newauthor <github-username>`.

```
$ ./newauthor aravindavk
Created Author page(author/aravindavk.md)
Updated author information in _data/authors.yml
```

Update the details in `$SRC/_data/authors.yml` file as required.

To write new blog post, run `./newblog <title>` to create the markdown
file under `$SRC/_posts` directory. For example,

```
$ ./newblog "My Awesome Blog"
Created _posts/2019-03-28-my-awesome-blog.md

Happy Blogging!
```

## Installation and Preview

- Install ruby-devel package(`sudo dnf install ruby-devel`)
- Install jekyll using `gem install bundler jekyll`
- Install the dependencies from project directory `bundle install --path vendor/bundle`

From this project directory, run `bundle exec jekyll serve
--baseurl=""` and open http://localhost:4000 for preview.

## Publish

Once the preview is satisfactory, commit the changes and create pull
request. Once merged, the blog url will be
https://gluster.github.com/devblog/my-awesome-blog

## Editing help

Follow markdown format to write post. Additionally, if images to be
used in blog, add the image to `$SRC/images` directory and use
relative URL to include that in the post. For example, `![Alt
Text](images/blog-image.jpg)`

If caption needs to be added to the image then use the following
syntax.

```
{% raw %}{% include image.html url="images/blog-image.jpg" description="This is a image" %}{% endraw %}
```
