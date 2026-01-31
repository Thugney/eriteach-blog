# Eriteach Blog

Microsoft Cloud Tech blog - Intune, Autopilot, Entra ID, Defender, Purview.

## Your Workflow

1. **Create content:** Chat with Claude in VSCode, ask it to write a post to `/content/drafts/`
2. **Review:** Open the draft, check accuracy, edit if needed
3. **Publish:** Move file to `/content/posts/` and change `draft: true` to `draft: false`
4. **Push:** Commit and push to GitHub
5. **Auto-deploy:** GitHub Actions builds and deploys to GitHub Pages
6. **Social:** n8n picks up new post and shares to LinkedIn/X

## Local Setup

```bash
# Clone repo
git clone https://github.com/YOUR_USERNAME/eriteach-blog.git
cd eriteach-blog

# Install theme (run once)
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/papermod

# Run locally
hugo server -D

# View at http://localhost:1313
```

## Folder Structure

```
content/
├── drafts/      # AI-generated drafts for review
├── posts/       # Approved, published posts
└── archives.md  # Archive page
```

## Post Frontmatter

```yaml
---
title: "Your Post Title"
date: 2025-01-31
draft: false          # true = draft, false = published
tags: ["intune", "autopilot"]
categories: ["How-To"]
summary: "Brief description for previews"
---
```

## Tags to Use

- intune
- autopilot
- entra-id
- defender
- purview
- troubleshooting
- how-to
- powershell

## GitHub Pages Setup

1. Go to repo Settings → Pages
2. Source: GitHub Actions
3. Custom domain: blog.eriteach.com
4. Add CNAME record in your DNS: `blog` → `YOUR_USERNAME.github.io`

## DNS Setup (at your registrar)

Add CNAME record:
- Type: CNAME
- Name: blog
- Value: YOUR_USERNAME.github.io
- TTL: 3600
test
test2
test3
test4
