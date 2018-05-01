+++
Description = "Jekyll drafts are good for local previews, but don't work for public sharing. Here is a drafts implementation using the Jekyll collections feature. Define a drafts collection and use front matter defaults to tailor the layout of a draft, and to exclude drafts from sitemaps and indexing. Maintain the same link before and after publishing by using the same permalink format for collections and for posts. Read how I did it."
Tags = ["sample"]
Categories = ["tech"]
Title = "Jekyll Drafts with Shareable Links"
comments = "true"
rightaside = "true"
leftaside = "true"
author = "aravind"
date = "2018-04-13"
+++

{{< figure class="centered" link="/images/IMG_20170110_135335111-01.jpg" src="/images/IMG_20170110_135335111-01-640.jpg" alt="Desk showing headphones, a laptop, a book and a pen" caption="Drafts are a work in progress." >}}

Drafts are posts that are not yet ready to be published. Jekyll has built-in support for [drafts](https://jekyllrb.com/docs/drafts/). This is how it works.

Create a `_drafts` folder at your site's root, create a new post in that folder, and run `jekyll serve` with the `--drafts` switch. Your draft turns up at the top of your recent posts (or wherever you have set up new posts to appear on your site).

## Problems and possible solutions
While this method is fine for locally previewing your drafts, what if you wanted to share a link to a friend or a co-worker, to ask for a review of a draft? Well, there are two problems with Jekyll drafts:

1. If you host your Jekyll site on [Github Pages](https://pages.github.com/), then you cannot use the `--drafts` switch. If you host elsewhere, you may be able to use a custom build command which includes `--drafts`. Or you could pre-build your site with `--drafts` locally, and push it.
2. Jekyll does not provide a page variable to determine if a page is a draft or not. You could add (say) `drafts: true` in the [YAML front matter](http://jekyllrb.com/docs/frontmatter/) of the draft and use `page.drafts` to tailor your layout.

A quick Internet search suggested two solutions for public drafts: one, which builds on [Jekyll's tag or category feature](http://hamishwillee.github.io/2014/06/11/public-drafts-in-jekyll/) and two, which builds on [Jekyll's collection feature](https://bsn.io/2017/01/public-drafts-with-a-github-pages-blog).

## Requirements
First I outlined my requirements:

1. Publish drafts without using the `--drafts` switch.
2. Exclude drafts from `sitemap.xml`, the site search index (I use [Algolia](https://www.algolia.com/)), and indexing by robots.
4. Exclude social sharing buttons, comments, related posts and links to next and previous posts from drafts.
5. A link to a draft post must not change after it is published.

I adapted the [latter solution](https://bsn.io/2017/01/public-drafts-with-a-github-pages-blog) to meet these requirements. Here is how it works.

## Enable drafts as a collection
First of all, to enable the `drafts` collection, I added the following to my `_config.yml`:

{{< highlight yaml "linenos=table" >}}
collections:
  drafts:
    output: true
{{< / highlight >}}

Now, if I create posts in the `_drafts` folder at my site's root, they will be published without requiring the `--drafts` switch. That is because the `collections` specification named `drafts` overrides the default treatment of the `_drafts` folder.

## Set up defaults for drafts
Next, I set the following front matter defaults to all posts under the `drafts` collection:

{{< highlight yaml "linenos=table" >}}
defaults:
  - scope:
      path: ""
      type: drafts
    values:
      comments: false
      share: false
      related: false
      sitemap: false  #To hide from sitemap.xml
      noindex: true   #To hide from robots and crawlers
{{< / highlight >}}

These defaults turn off commenting, social sharing buttons, related posts and inclusion in `sitemap.xml`. Next, I used the `page.noindex` variable to add a snippet in the header of each draft post as follows:

{{< highlight html "linenos=table" >}}
{% if page.noindex == true %}
  <meta name="robots" content="noindex">
{% endif %}
{{< / highlight >}}

This prevents robots from indexing draft posts. To exclude `_drafts` from the site search index, I added the following (Algolia-specific setting) to the `_config.yml`.

{{< highlight yaml "linenos=table" >}}
algolia:
  files_to_exclude:
    - _drafts/*.markdown
{{< / highlight >}}

## Maintaining draft links after publication
My posts use the following permalink format:

{{< highlight yaml "linenos=table" >}}
 permalink: /:categories/:title/
{{< / highlight >}}

I added the same permalink specification to the `drafts` collection as follows:

{{< highlight yaml "linenos=table" >}}
collections:
  drafts:
    output: true
    permalink: /:categories/:title/
{{< / highlight >}}

Now, if I do not change the categories and title of my post, then the permalink would not change on publication. When I am ready to publish, I simple move the file from `_drafts` to `_posts`. That's it!

The reason this works lies in the way Jekyll treats files in `_posts` and files in collections (in our case, `_drafts`). Files in `_posts` are accessible in the variables `site.posts` and `site.categories`, but files in collections are not. But categories defined in a file, remain accessible as `page.categories` whether the file is in `_posts` or `_drafts`.

That's why the link doesn't change on publication, but the file content appears in recent posts or a category index page, only when it is in `_posts` (and not when it is still in `_drafts`).

That concludes how I set up Jekyll drafts with shareable links.

{{% notice color="washed-green" %}}
**Feedback**: Did you enjoy reading or think it can be improved? Don't forget to leave your thoughts in the comments section below! If you liked this article, please share it with your friends, and read a few more!
{{% / notice %}}
