---
title: "Creating a Blog on a custom domain with GitHub Actions and Hugo"
date: 2021-05-09T09:09:00+02:00
draft: false
tags: ["meta"]
description: "Writing articles about technical topics is an incredibly effective way to learn. This is the main reason I want to have a blog"
---

I always wanted to have a personal website where I write articles occasionally. 
Bought baris.io many years ago for this reason but finding time to blog consistently was really difficult for me.
Between work, chores, friends/family, Twitch/YouTube/video games and rest, there was never enough time.

While it also has some drawbacks, one of the few benefits of COVID-19 is that we are working from our homes, 
so I do not need to commute to work anymore. This means saving 1-2 hours every day that I can use to focus on a blog.    

## Reasons for blogging
Writing articles about technical topics is an incredibly effective way to learn. This is the main reason I want to have a blog.
However, there are other benefits as well.

- Motivation to more thoroughly investigate things 
- Building up a portfolio over time
- Practicing technical writing skills
- Having an online presence

Also, I really feel like having progress on a publicly visible website for a long time would be really satisfying.

## Tech Stack
I wanted a simple, fast, clean looking personal blog because I am a big fan of minimalist products. 
Features like multiple language support, comments, sign-ups, newsletters or any other dynamic data was not necessary in my opinion, at least initially.
For these reasons, static site generators like hugo, jekyll and gatsby looked like good ideas.

In the end I went with [Hugo](https://gohugo.io/) and [hermit](https://github.com/Track3/hermit) theme.

All the code related to this blog is [publicly available on GitHub]((https://github.com/mbaris/hugo-blog))

The website is hosted on [GitHub Pages](https://pages.github.com/)

Using baris.io as a custom domain by [adding a CNAME file](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site) to the repository

GitHub Actions are used with [Hugo Setup Action ](https://github.com/peaceiris/actions-hugo) to build static files from templates and trigger a deployment after every commit.

I am quite happy with the current tech stack because it costs me nothing apart from the domain name to host this website.
It is also really easy to push updates and make changes.

## Pushing an Update
The repository has two branches.
*master* has hugo templates and markdown files
*gh-pages* has the html, css and javascript file which can be hosted

I create new posts with the command 
``` bash
hugo new posts/post-name.md
```
Then edit markdown files on Intellij IDEA Community Edition. It might seem like an overkill but idea has a built-in 
markdown editor, git diff tool, terminal and a plugin for hugo server, so it is actually really effective in my opinion.

After preparing markdown files, I push commits to the master branch on GitHub and GitHub Actions takes care of the rest.
I eventually want to prepare more in depth posts about Observability or Infrastructure but 
my main focus right now writing posts consistently, even if they are a little shallow. 
