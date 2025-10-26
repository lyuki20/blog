---
title: automating obsidian to github pages
date: 2025-10-26T19:01:11+09:00
draft: false
summary: How I wired Obsidian, Hugo, and GitHub Pages together with a PowerShell deployment script.
tags:
  - automation
  - hugo
  - obsidian
  - github-pages
---

Over the last session I turned my local Obsidian notes into a fully automated publishing pipeline. Here's the journey end to end, plus a diagram of the flow.
![](/obsidian/Pasted%20image%2020251026201107.png)

![](/obsidian/Pasted%20image%2020251026203152.png)

## Goals

- Write notes in Obsidian (E:/iCloud/iCloudDrive/Obsidian/notes/posts)
- Sync them into my Hugo site stored inside WSL (/home/ubuntu/blog/blog)
- Build static pages, commit, and push to GitHub
- Publish automatically using GitHub Pages

## Project structure

`	ext
Obsidian Vault (Windows)        Hugo site (WSL)
└── posts/                      └── content/
    ├── First post.md   ─┐          ├── posts/
    ├── ...               ├─ sync → │   ├── First post.md
    └── automation...md ─┘          │   └── automation...
                                  ├── layouts/index.html
                                  ├── hugo.toml
                                  └── public/
`

## The deployment script

I created deploy-blog.ps1 so that one command handles everything:

1. Validates paths and required commands (
sync, hugo, git).
2. Uses 
sync --delete to mirror Obsidian posts into content/posts.
3. Runs hugo --minify inside WSL to build the site.
4. Auto-detects changes, staging and committing them (timestamp default message).
5. Pushes main to git@github.com:lyuki20/blog.git via SSH.
6. Publishes the public/ directory by pushing a gh-pages branch via git subtree split.

PowerShell invocation example:

`powershell
powershell.exe -ExecutionPolicy Bypass -File .\deploy-blog.ps1 -CommitMessage "Publish new notes"
`

## GitHub setup

- Created the log repository under my lyuki20 account and configured SSH access from WSL.
- Pushed the Hugo source to main.
- Enabled GitHub Pages on the gh-pages branch.
- Confirmed the live site at https://lyuki20.github.io/blog/.

## Hugo configuration

- Updated hugo.toml with my site metadata and correct aseURL.
- Added front matter to content/_index.md and created content/posts/_index.md so the theme shows “Home” and “Blog” in the navigation.
- Overrode layouts/index.html to render a “Latest posts” section on the landing page that lists the five newest entries with dates.

## Publishing flow (text diagram)

`	ext
+------------------+      rsync       +------------------------+
| Obsidian posts   | ───────────────▶ | content/posts (Hugo)   |
| E:\...\posts     |                 | /home/ubuntu/blog/blog |
+------------------+                 +------------------------+
          │                                   │
          │ hugo build                        │ git commit/push
          ▼                                   ▼
+------------------+      minified site       +------------------------+
| hugo --minify    | ───────────────────────▶ | public/                |
+------------------+                          +------------------------+
                                                     │ subtree split
                                                     ▼
                                           +---------------------------+
                                           | gh-pages branch on GitHub |
                                           +---------------------------+
                                                     │ GitHub Pages
                                                     ▼
                                           https://lyuki20.github.io/blog/
`

## Result

Running the script now syncs whatever I write in Obsidian, rebuilds the Hugo site, pushes the commits, and republishes GitHub Pages within a minute. The homepage automatically lists the most recent posts, and new notes are live at the posts/ URL after each deployment.

Next steps will be to expand front matter templates in Obsidian and maybe add Netlify-like preview builds, but the core automation is complete and working.