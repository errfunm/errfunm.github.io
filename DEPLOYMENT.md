# Deployment Guide

This guide explains how to deploy your Obsidian Blogger site to different platforms while ensuring correct URL handling.

## URL Configuration

The site uses environment variables to handle URLs correctly across different deployment environments:

- `SITE_URL`: The full URL of your site
- `BASE_URL`: The base path where your site is served from

## Deployment Scenarios

### 1. Local Development

```env
SITE_URL=http://localhost:4321
BASE_URL=/
```

Run the development server:
```bash
npm run dev
```

### 2. GitHub Pages

If your repository is at `github.com/username/repo-name`:

```env
SITE_URL=https://username.github.io
BASE_URL=/repo-name/
```

#### Automatic Deployment

1. Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18
      - name: Install dependencies
        run: npm ci
      - name: Build website
        run: npm run build
        env:
          SITE_URL: https://<username>.github.io
          BASE_URL: /<repo-name>/
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

2. Enable GitHub Pages in your repository settings:
   - Source: GitHub Actions

### 3. Custom Domain

If your site is at `yourdomain.com`:

```env
SITE_URL=https://yourdomain.com
BASE_URL=/
```

#### Netlify Deployment

1. Add environment variables in Netlify dashboard:
   - Key: `SITE_URL`, Value: `https://yourdomain.com`
   - Key: `BASE_URL`, Value: `/`

2. Configure build settings:
   - Build command: `npm run build`
   - Publish directory: `dist`

#### Vercel Deployment

1. Add environment variables in Vercel dashboard:
   - Key: `SITE_URL`, Value: `https://yourdomain.com`
   - Key: `BASE_URL`, Value: `/`

2. The build settings should be auto-detected

## URL Handling

The site automatically handles URLs using the utility functions in `src/utils/url.ts`:

```typescript
// Get correct URL for any path
import { getUrl } from '../utils/url';

// Usage in components
<a href={getUrl('/blog')}>Blog</a>
```

## Important Notes

1. Always ensure `SITE_URL` does not end with a slash
2. Always ensure `BASE_URL` starts and ends with a slash
3. For GitHub Pages, `BASE_URL` must match your repository name
4. For custom domains, use `/` as `BASE_URL`
5. Update your sitemap configuration in `astro.config.mjs` if needed

## Troubleshooting

1. **404 errors on subpages**:
   - Check if `BASE_URL` is correctly set
   - Ensure all internal links use the `getUrl()` function

2. **Assets not loading**:
   - Ensure all asset URLs use the `getUrl()` function
   - Check if public assets are in the correct directory

3. **GitHub Pages not updating**:
   - Check GitHub Actions workflow status
   - Ensure GitHub Pages is enabled in repository settings

## Need Help?

If you encounter any issues:
1. Check the [Astro deployment docs](https://docs.astro.build/en/guides/deploy/)
2. Open an issue in the GitHub repository
3. Check existing issues for similar problems 