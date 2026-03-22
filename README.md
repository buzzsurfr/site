# Teddy Spark — teddyspark.co

[![Netlify Status](https://api.netlify.com/api/v1/badges/5139b5cc-6f53-459b-b698-db0d1f653628/deploy-status)](https://app.netlify.com/projects/resilient-rugelach-110f76/deploys)

Hugo site with Decap CMS, deployed on Netlify.

## Stack
- **Hugo** — static site generator
- **Decap CMS** — browser-based content editor at `/admin/`
- **Netlify** — hosting, forms, and identity

## Local development

```bash
# Install Hugo (if needed)
brew install hugo          # macOS
winget install Hugo.Hugo.Extended  # Windows

# Run local server
hugo server -D

# Visit http://localhost:1313
```

## Deploying to Netlify

1. Push this folder to a GitHub repository
2. Go to netlify.com → New site from Git → connect your repo
3. Build command: `hugo --minify`
4. Publish directory: `public`
5. Deploy — it reads `netlify.toml` automatically

## Setting up Decap CMS

After deploying:

1. Netlify → Site Settings → Identity → Enable Identity
2. Set registration to **Invite only**
3. Site Settings → Services → Git Gateway → Enable
4. Identity tab → Invite yourself by email
5. Visit `https://teddyspark.co/admin/` to log in and start writing

## Adding content via CLI

```bash
hugo new blog/my-post-title.md
hugo new portfolio/my-project-name.md
```

## Replacing the About photo

In `layouts/about.html`, replace the SVG placeholder with:
```html
<img src="/images/your-photo.jpg" alt="Teddy Spark">
```
Add your photo to `static/images/`.

## Updating your YouTube link

In `hugo.toml`:
```toml
[params]
  youtube = "https://youtube.com/@YOUR_CHANNEL"
```
