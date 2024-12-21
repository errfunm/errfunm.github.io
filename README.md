# Obsidian Blogger

Transform your Obsidian notes into a beautiful, modern blog with ease. Built with Astro.js for blazing-fast performance and seamless Markdown support.

![Obsidian Blogger](public/icon.png)

## ✨ Features

- 🚀 **Blazing Fast**: Built with Astro.js for optimal performance and SEO
- 📝 **Markdown Support**: Write in pure Markdown, just like in Obsidian
- 🎨 **Beautiful Design**: Modern, responsive layout with dark mode support
- 🏷️ **Tag System**: Organize posts with tags and browse by categories
- 📱 **Mobile-First**: Looks great on any device
- 🔍 **SEO Optimized**: Built-in SEO with OpenGraph and canonical URLs
- 📰 **RSS Feed**: Automatic RSS feed generation
- 🗺️ **Sitemap**: Automatic sitemap generation
- 🖼️ **Featured Images**: Support for post featured images
- 📅 **Date-based Sorting**: Chronological post organization
- 🌙 **Dark Mode**: Built-in dark mode support

## 🚀 Quick Start

1. Clone the repository:
```bash
git clone https://github.com/yourusername/obsidian-blogger.git
cd obsidian-blogger
```

2. Install dependencies:
```bash
npm install
```

3. Configure your site:
   - Copy `.env.example` to `.env`
   - Update the environment variables for your deployment

4. Start the development server:
```bash
npm run dev
```

5. Visit `http://localhost:4321` to see your blog!

## 📝 Creating Blog Posts

1. Add your Markdown files to `src/content/blog/`
2. Include required frontmatter:
```yaml
---
title: 'Your Post Title'
description: 'Post description for SEO'
publish: true
created_date: 2024-01-19
slug: custom-url
tags:
  - tag1
  - tag2
---
```

### Supported Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `title` | Yes | Post title |
| `description` | No | SEO description |
| `publish` | Yes | Set to `true` to publish |
| `created_date` | No | Publication date |
| `slug` | No | Custom URL slug |
| `tags` | No | Array of tags |
| `featured_image` | No | Hero image URL |
| `subtitle` | No | Optional subtitle |

## 🎨 Customization

### Site Configuration

Update `src/consts.ts` to modify:
- Site title
- Site description
- Other global constants

### Styling

- Global styles: `src/styles/global.css`
- Component styles: Inline in respective `.astro` files
- Theme variables: CSS custom properties in global styles

## 🚀 Deployment

See [DEPLOYMENT.md](DEPLOYMENT.md) for detailed deployment instructions for:
- GitHub Pages
- Netlify
- Vercel
- Custom domains

## 📦 Project Structure

```
obsidian-blogger/
├── src/
│   ├── components/    # Reusable components
│   ├── content/       # Blog posts and content
│   ├── layouts/       # Page layouts
│   ├── pages/         # Route components
│   ├── styles/        # Global styles
│   └── utils/         # Utility functions
├── public/           # Static assets
├── astro.config.mjs  # Astro configuration
└── package.json      # Project dependencies
```

## 🛠️ Development Commands

| Command | Action |
|---------|--------|
| `npm run dev` | Start dev server |
| `npm run build` | Build for production |
| `npm run preview` | Preview production build |

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 💖 Support

If you find this project helpful, please consider:
- Starring the repository
- Sharing it with others
- Contributing to its development

## 🔗 Links

- [Documentation](https://github.com/yourusername/obsidian-blogger/wiki)
- [Issues](https://github.com/yourusername/obsidian-blogger/issues)
- [Discussions](https://github.com/yourusername/obsidian-blogger/discussions)
