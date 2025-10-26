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
I turned my local obsidian notes into a fully automated publishing pipeline. here's how i did it.
## goals

- write notes in obsidian
- Sync them into my Hugo site directory
- build static pages, commit, and push to GitHub
- publish automatically using github pages

## architecture overview
![](/obsidian/Pasted%20image%2020251026210649.png)
## The deployment script

created a script so that one command handles everything:

1. validates paths and required commands (
sync, hugo, git).
2. uses 
sync --delete to mirror obsidian posts into content/posts.
3. Runs hugo --minify inside WSL to build the site.
4. Pushes main to github repository via SSH.
5. publishes the public/ directory by pushing a gh-pages branch via git subtree split.
```
[CmdletBinding()]

Param(

    [string]$CommitMessage

)

  

$obsidianWindowsPath = "E:\iCloud\iCloudDrive\Obsidian\notes\posts"

$obsidianWslPath = "/mnt/e/iCloud/iCloudDrive/Obsidian/notes/posts"

$hugoProjectPath = "/home/ubuntu/blog/blog"

$hugoContentPostsPath = "$hugoProjectPath/content/posts"

$remoteName = "origin"

$mainBranch = "main"

$pagesBranch = "gh-pages"

  

function Invoke-WSL {

    param(

        [Parameter(Mandatory = $true)]

        [string]$Command

    )

  

    Write-Host "WSL> $Command"

    & wsl.exe bash -lc $Command

    if ($LASTEXITCODE -ne 0) {

azsA        throw "WSL command failed with exit code ${LASTEXITCODE}: $Command"

    }

}

  

function ConvertTo-BashSingleQuoted {

    param(

        [Parameter(Mandatory = $true)]

        [string]$Value

    )

  

    return "'" + ($Value -replace "'", "'\\''") + "'"

}

  

function Ensure-WSLCommand {

    param(

        [Parameter(Mandatory = $true)]

        [string]$Name

    )

  

    Invoke-WSL "command -v $Name >/dev/null 2>&1 || { echo 'Missing required command: $Name' >&2; exit 1; }"

}

  

function Ensure-WSLPath {

    param(

        [Parameter(Mandatory = $true)]

        [string]$Path

    )

  

    Invoke-WSL "if [ ! -d '$Path' ]; then echo 'Missing required path: $Path' >&2; exit 1; fi"

}

  

try {

    if (-not (Test-Path -LiteralPath $obsidianWindowsPath)) {

        throw "Obsidian source path not found: $obsidianWindowsPath"

    }

  

    Ensure-WSLPath -Path $hugoProjectPath

    Ensure-WSLPath -Path $hugoContentPostsPath

    Ensure-WSLCommand -Name "rsync"

    Ensure-WSLCommand -Name "hugo"

    Ensure-WSLCommand -Name "git"

  

    Write-Host "Syncing Obsidian posts into Hugo content..."

    $syncCommand = "rsync -av --delete --exclude '.git' --exclude '_index.md' '$obsidianWslPath/' '$hugoContentPostsPath/'"

    Invoke-WSL $syncCommand

  

    Write-Host "Building site with Hugo..."

    Invoke-WSL "cd '$hugoProjectPath' && hugo --minify"

  

    Write-Host "Checking for changes..."

    $gitStatus = & wsl.exe bash -lc "cd '$hugoProjectPath' && git status --porcelain"

    if ([string]::IsNullOrWhiteSpace($gitStatus)) {

        Write-Host "No changes detected. Nothing to commit."

        exit 0

    }

  

    Write-Host "Staging changes..."

    Invoke-WSL "cd '$hugoProjectPath' && git add -A"

  

    if ([string]::IsNullOrWhiteSpace($CommitMessage)) {

        $CommitMessage = "Publish $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')"

    }

    $commitShellArg = ConvertTo-BashSingleQuoted -Value $CommitMessage

  

    Write-Host "Committing..."

    Invoke-WSL ("cd '{0}' && git commit -m {1}" -f $hugoProjectPath, $commitShellArg)

  

    Write-Host "Pushing to $mainBranch..."

    Invoke-WSL "cd '$hugoProjectPath' && git push $remoteName $mainBranch"

  

    Write-Host "Publishing static site to $pagesBranch..."

    Invoke-WSL "cd '$hugoProjectPath' && git branch -D gh-pages-tmp >/dev/null 2>&1 || true"

    Invoke-WSL "cd '$hugoProjectPath' && git subtree split --prefix public -b gh-pages-tmp"

    Invoke-WSL "cd '$hugoProjectPath' && git push $remoteName gh-pages-tmp:$pagesBranch"

    Invoke-WSL "cd '$hugoProjectPath' && git branch -D gh-pages-tmp"

  

    Write-Host "Deployment complete."

}

catch {

    Write-Error $_

    exit 1

}
```
PowerShell invocation example:

`powershell
```
powershell.exe -ExecutionPolicy Bypass -File .\deploy-blog.ps1 -CommitMessage "publish new notes"
```
## GitHub setup

- Created the repository under my lyuki20 account and configured SSH access from WSL.
- Pushed the hugo source to main.
- Enabled github pages on the `gh-pages`branch.
## Result

Running the script now syncs whatever I write in Obsidian, rebuilds the Hugo site, pushes the commits, and republishes GitHub Pages instantly. 

