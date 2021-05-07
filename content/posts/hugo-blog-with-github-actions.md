---
title: "Hugo Blog With GitHub Actions"
date: 2021-05-04T09:13:07+02:00
draft: true
---

I always wanted to have a personal website where I write articles occasionally. 
I bought baris.io many years ago for this reason but blogging consistently was really difficult for different reasons.

## Why do I want to blog?
Writing articles about technical topics is an incredibly effective way to learn. This is the main reason I want to have a blog.
However, there are other benefits like

- Building up a portfolio over time
- Practicing technical writing skills
- Having an online presence

## Tech Stack
I wanted a fast, clean looking blog with minimal features. I don't need multiple languages, comments, sign-ups, newsletters etc.
For these reasons, static site generators looked like a good fit.

I am quite happy with the current state because it costs me nothing apart from the domain name. 
It is also really easy to push an update. 

repository for this website is on [GitHub](https://github.com/mbaris/hugo-blog)
I use GitHub Actions with [Hugo](hugo) to build the static files from templates
The contents are hosted on GitHub Pages. It hosts the contents in gh-pages branch. I am using my custom domain by adding a CNAME file.

The repository has two branches.
master has hugo templates and markdown files
gh-pages has the html, css and javascript file which can be hosted

I only edit and push to the master branch. GitHub Actions takes care of the rest by listening to updates on master, building the website and publishing it to gh-pages branch.

To post a new website, I just add a markdown file to the master branch and push it to github.




