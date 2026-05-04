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

## Shortcodes

### `callout`

Renders a styled callout box inside a blog post.

```markdown
{{</* callout type="note" */>}}
Your message here. Markdown is supported.
{{</* /callout */>}}
```

| `type` | Label color | Use for |
|--------|-------------|---------|
| `note` (default) | Green | Tips, asides, extra context |
| `warning` | Amber | Caveats, gotchas |

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

## Social media automation

When a new post is published (detected via RSS), an n8n workflow drafts platform-specific social posts with Claude and routes each one through a per-platform email + form review before publishing.

The workflow is at [`.n8n/workflows/blog-social-media.json`](.n8n/workflows/blog-social-media.json) and is imported manually into n8n — it is not part of the Hugo build.

### Flow

1. RSS feed polled every 30 min — triggers on new items
2. Claude drafts separate posts for Facebook (hobbyist/maker audience), LinkedIn (professional/insight-led), and Bluesky (punchy, open web crowd)
3. **Sequential per-platform review:** email with the draft → form with Approve / Approve with edits / Reject
   - *Approve* — posts as drafted
   - *Approve with edits* — posts the text you enter in the form
   - *Reject* — skips that platform, continues to the next
4. Platforms run in order: Facebook → LinkedIn → Bluesky

### n8n setup

**Import:** Workflows → Import from File → select the JSON. n8n will prompt you to remap credentials.

**Credentials to create** (Settings → Credentials):

| Name in workflow | Type | Value |
|---|---|---|
| `Anthropic API Key` | Header Auth | Name: `x-api-key` / Value: API key |
| `Gmail OAuth2` | Gmail OAuth2 | Google OAuth flow |
| `LinkedIn Bearer Token` | Header Auth | Name: `Authorization` / Value: `Bearer <token>` |
| `Facebook Page Token` | Header Auth | Name: `Authorization` / Value: `Bearer <page_token>` |

**Variables to set** (Settings → Variables):

| Variable | Value |
|---|---|
| `BLUESKY_HANDLE` | `teddyspark.co` |
| `LINKEDIN_PERSON_ID` | Your LinkedIn member ID (from `/v2/me` API or profile URL) |
| `FACEBOOK_PAGE_ID` | Numeric ID of your Facebook Page |

`BLUESKY_APP_PASSWORD` is **not** an n8n variable — it is injected as an environment variable from 1Password via the pibernetes cluster (see below). The workflow reads it as `$env.BLUESKY_APP_PASSWORD`.

### 1Password / pibernetes infrastructure

n8n runs on pibernetes at `automation.salvo.services` (Tailscale). The 1Password Kubernetes Operator is already installed in the cluster. Secrets management for n8n lives in the pibernetes repo at [`cluster/apps/automation/`](https://github.com/buzzsurfr/pibernetes/tree/main/cluster/apps/automation).

Create two items in the **pibernetes** 1Password vault before pushing the config:

| 1Password item | Field | Value |
|---|---|---|
| `n8n-encryption` | `key` | Random 32+ char string — set once, never rotate |
| `n8n-social-media` | `bluesky-app-password` | Bluesky App Password (Settings → App Passwords) |

`OnePasswordItem` CRDs in `cluster/apps/automation/config/secrets.yaml` sync these to Kubernetes Secrets in the `automation` namespace. The n8n Helm values (`cluster/apps/automation/application.yaml`) inject them as `N8N_ENCRYPTION_KEY` and `BLUESKY_APP_PASSWORD` env vars.

**Anthropic, LinkedIn, and Facebook** credentials are stored directly in n8n's credential store (Settings → Credentials), not as env vars, since n8n's OAuth and Header Auth credential types don't support env var references at the field level.
