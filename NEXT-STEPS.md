# Naive Docs — Next Steps

This document explains how to test the docs locally, deploy to production with Mintlify, and set up custom domains.

## Local Development

### Prerequisites: Node 22 LTS

Mintlify requires Node 22 LTS (not Node 25+). If your system Node is 25+, use a version manager:

```bash
# Install fnm (fast node manager) if you don't have one
brew install fnm

# Add to your shell (add this to ~/.zshrc permanently)
eval "$(fnm env)"

# Install and use Node 22 LTS
fnm install 22
fnm use 22

# Verify
node --version  # should show v22.x.x
```

Alternatively, create a `.node-version` file in `naive-docs/` so fnm auto-switches:

```bash
echo "22" > .node-version
```

### 1. Install the Mintlify CLI

```bash
npm install -g mint
```

### 2. Run the dev server

```bash
cd naive-docs
fnm use 22  # if your default Node is 25+
mint dev
```

This starts a local preview at `http://localhost:3000`. The server hot-reloads when you edit MDX files or `docs.json`.

### 3. Verify everything renders

Check each tab:
- **Documentation** — Getting started pages and guides
- **API Reference** — All 58 endpoint pages
- **MCP Server** — Overview, core tools, meta-tools, context, catalog
- **CLI** — All command reference pages

Look for:
- Broken links (Mintlify shows warnings in the terminal)
- MDX syntax errors (shown as red banners on the page)
- Missing pages listed in `docs.json` that don't have corresponding `.mdx` files
- Navigation order making sense

### 4. Build check

```bash
mint build
```

This runs the full static build and catches any errors that the dev server might miss.

## Deploy to Mintlify

### Option A: GitHub Integration (Recommended)

1. **Create a GitHub repo** for the docs:
   ```bash
   cd naive-docs
   git init
   git add -A
   git commit -m "initial docs"
   git remote add origin https://github.com/usenaive/docs.git
   git push -u origin main
   ```

2. **Sign up at [mintlify.com](https://mintlify.com)** and connect your GitHub account.

3. **Create a new project** in the Mintlify dashboard:
   - Select the `usenaive/docs` repo
   - Branch: `main`
   - Root directory: `/` (since docs.json is at the root)

4. **Deploy** — Mintlify automatically builds and deploys on every push to `main`.

### Option B: Manual Deploy via CLI

```bash
cd naive-docs
mint deploy
```

This pushes the docs directly to Mintlify's hosting. You'll need to authenticate with your Mintlify account first.

## Custom Domain Setup

### 1. Choose your docs domain

Options:
- `docs.usenaive.ai` (subdomain — recommended)
- `usenaive.ai/docs` (subpath — requires proxy config)

### 2. Configure in Mintlify Dashboard

1. Go to your project settings in the Mintlify dashboard
2. Navigate to **Custom Domain**
3. Enter your domain: `docs.usenaive.ai`
4. Mintlify provides DNS records to add

### 3. Add DNS Records

At your domain registrar (Cloudflare, Route53, etc.), add:

```
Type: CNAME
Name: docs
Value: cname.mintlify.com
```

Wait for DNS propagation (usually 5-30 minutes).

### 4. Verify

Mintlify automatically provisions an SSL certificate. Once DNS propagates, your docs are live at `https://docs.usenaive.ai`.

## Adding Logo and Favicon

1. Create a `/logo` directory in `naive-docs/`:
   ```
   naive-docs/
     logo/
       light.svg    # Logo for light mode
       dark.svg     # Logo for dark mode
     favicon.svg    # Browser tab icon
   ```

2. Update `docs.json`:
   ```json
   {
     "logo": {
       "light": "/logo/light.svg",
       "dark": "/logo/dark.svg"
     },
     "favicon": "/favicon.svg"
   }
   ```

## Adding an OpenAPI Spec (Optional)

If you want Mintlify to auto-generate API reference pages from an OpenAPI spec instead of the hand-written MDX pages:

1. Create an `openapi.json` or `openapi.yaml` at the repo root
2. Add to `docs.json`:
   ```json
   {
     "api": {
       "openapi": "/openapi.json",
       "playground": {
         "display": "interactive"
       }
     }
   }
   ```
3. Remove the hand-written API reference MDX files (or keep both)

For now, the hand-written MDX pages give more control over content, examples, and security warnings.

## Updating Docs

### Adding a new page

1. Create the `.mdx` file in the appropriate directory
2. Add the path to `docs.json` under the correct navigation group
3. Push to `main` — Mintlify auto-deploys

### Updating an existing page

Just edit the `.mdx` file and push. Changes deploy automatically.

### Adding a new tool

When a new tool is added to the SDK:
1. Create `api-reference/<category>/<tool-name>.mdx`
2. Add the page path to `docs.json` navigation
3. Update `mcp/tool-catalog.mdx` with the new tool
4. If it's a core tool, update `mcp/core-tools.mdx`
5. If it needs a workflow guide, update or create a guide in `guides/`

## File Structure

```
naive-docs/
  docs.json                           # Mintlify configuration
  NEXT-STEPS.md                       # This file
  getting-started/
    introduction.mdx                  # What is Naive
    quickstart.mdx                    # 3-step setup
    authentication.mdx                # Bearer tokens, rate limits
    installation.mdx                  # npm install, runtime config
  guides/
    email.mdx                         # Email workflow guide
    social-media.mdx                  # Social media setup + posting
    video.mdx                         # HeyGen + Submagic workflows
    images.mdx                        # Image generation guide
    payments.mdx                      # Virtual card purchasing
    integrations.mdx                  # Composio + CMS integrations
    browser.mdx                       # Browser automation
    credentials.mdx                   # Credential management
    research.mdx                      # Web search + deep research
  api-reference/
    overview.mdx                      # API overview
    communication/                    # 6 endpoint pages
    distribution/                     # 9 endpoint pages
    creation/                         # 10 endpoint pages
    research/                         # 3 endpoint pages
    integrations/                     # 4 endpoint pages
    domains/                          # 7 endpoint pages
    payments/                         # 3 endpoint pages
    connections/                      # 5 endpoint pages
    browser/                          # 3 endpoint pages
    credentials/                      # 3 endpoint pages
    billing/                          # 1 endpoint page
    system/                           # 3 endpoint pages (status, tools search/detail)
  mcp/
    overview.mdx                      # MCP server architecture
    core-tools.mdx                    # 8 core registered tools
    meta-tools.mdx                    # 4 discovery/execute tools
    context-tool.mdx                  # naive_context deep dive
    tool-catalog.mdx                  # All 54 tools listed
  cli/
    overview.mdx                      # CLI purpose and commands
    init.mdx                          # Interactive setup
    status.mdx                        # Connection health
    tools.mdx                         # Tool browsing
    keys.mdx                          # Key management
    mcp-start.mdx                     # MCP server launcher
```

## Checklist Before Go-Live

- [ ] Run `mint dev` and verify all pages render without errors
- [ ] Run `mint build` to check for build errors
- [ ] Add logo SVGs (light + dark) and favicon
- [ ] Update colors in `docs.json` to match Naive brand
- [ ] Create GitHub repo and push
- [ ] Connect to Mintlify dashboard
- [ ] Configure custom domain (docs.usenaive.ai)
- [ ] Verify SSL certificate is provisioned
- [ ] Test on mobile
- [ ] Share with team for review
