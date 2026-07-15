# snuderek.github.com

Personal site built with [Hugo](https://gohugo.io/) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme.

## Local development

Install Hugo Extended, then run the local preview server:

```bash
hugo server -D
```

Open <http://localhost:1313/>. Hugo watches the repo and rebuilds when content or config files change.

To run a production-style build locally:

```bash
hugo --gc --minify
```

Generated output is written to `public/` and should not be committed.

## Updating the Main Page

The home page is controlled mainly by `hugo.yaml`.

Edit this section to change the main intro text:

```yaml
params:
  homeInfoParams:
    Title: "Notes and projects"
    Content: "A personal site for technical writing, experiments, and links worth keeping."
```

Edit `params.socialIcons` in the same file to add or change links shown under the intro:

```yaml
socialIcons:
  - name: github
    url: "https://github.com/SNUDerek"
  - name: linkedin
    url: "https://www.linkedin.com/in/derek-hommel-4a646869/"
```

The home page content file is `content/_index.md`. Keep it for page metadata such as title and description.

## Creating a New Post

Create posts under `content/posts/`. Use lowercase, hyphenated filenames:

```bash
hugo new posts/my-new-post.md
```

If you are creating the file manually, use this front matter:

```markdown
---
title: "My New Post"
date: 2026-07-15T00:00:00+09:00
draft: true
tags:
  - example
categories:
  - notes
---

Write the post here.
```

Set `draft: false` when the post is ready to publish. Draft posts show locally with `hugo server -D`, but they are excluded from the GitHub Pages production build.

## Categories and Series Links

Use `categories` for broad groups or post series, and `tags` for more specific topics.

Example front matter:

```yaml
categories:
  - diarization
tags:
  - speech
  - machine learning
```

Hugo creates category pages automatically. A category named `diarization` is available at:

```text
/categories/diarization/
```

You can link to that category from the bottom of a post:

```markdown
See other posts in this series: [diarization](/categories/diarization/).
```

Multi-word category names are usually slugified. For example, `speaker diarization` becomes `/categories/speaker-diarization/`.

## Adding Images and Other Media

For post-specific media, use a Hugo page bundle:

```text
content/posts/my-new-post/
  index.md
  diagram.png
  screenshot.jpg
```

Reference bundled media from `index.md` with a relative path:

```markdown
![Architecture diagram](diagram.png)
```

For media shared across multiple pages, put files under `static/`:

```text
static/images/profile.jpg
static/files/resume.pdf
```

Reference shared static files from the site root:

```markdown
![Profile photo](/images/profile.jpg)
[Resume](/files/resume.pdf)
```

Prefer repo-local media for posts so the site remains portable and does not depend on external image hosts.

## Publishing

Push changes to `master`. The GitHub Pages workflow in `.github/workflows/hugo.yaml` builds and deploys the site automatically.

PaperMod is included as a git submodule under `themes/PaperMod`, so clone this repo with submodules when setting it up on a new machine:

```bash
git clone --recurse-submodules git@github.com:SNUDerek/snuderek.github.com.git
```

If the repo was already cloned without submodules:

```bash
git submodule update --init --recursive
```
