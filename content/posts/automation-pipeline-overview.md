---
title: automating obsidian to github pages
date: 2025-10-26T19:01:11+09:00
draft: false
summary: How I wired Obsidian, Hugo, and GitHub Pages together with a PowerShell deployment script.
tags:
  - automation
  - git
  - powershell
  - hugo
---
i turned my local obsidian notes into a fully automated publishing pipeline. here's how i did it.
## goals

- write notes in obsidian
- sync them into my hugo site directory
- build static pages, commit, and push to gitHub
- publish automatically using github pages

## architecture overview
![](/obsidian/pasted-image-20251026210649.png)
## the deployment script

created a script so that one command handles everything:

1. validates paths and required commands (
sync, hugo, git).
2. uses 
sync --delete to mirror obsidian posts into content/posts.
3. runs hugo --minify to build the site.
4. pushes main to github repository via ssh.
5. publishes the public/ directory by pushing a `gh-pages` branch via git subtree split.
## github setup

- created the repository under my account and configured ssh access. 
- pushed the hugo source to main.
- enabled github pages on the `gh-pages`branch.
## result

running the script now syncs whatever I write in obsidian, rebuilds the hugo site, pushes the commits, and republishes github pages instantly. 

