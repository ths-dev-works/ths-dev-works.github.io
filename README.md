# THS Dev Works

A multilingual Hugo static site for THS Dev Works - Go Developer & Technical Advisor portfolio website.

## Features

- **Multilingual Support**: English and Portuguese
- **Modern Theme**: Built with PaperMod theme
- **Responsive Design**: Mobile-friendly layout
- **SEO Optimized**: Meta tags, Open Graph, and structured data
- **Fast Performance**: Minified assets and optimized builds
- **GitHub Pages Deployment**: Automated CI/CD pipeline

## Prerequisites

- [Hugo Extended](https://gohugo.io/installation/) (latest version recommended)
- [Git](https://git-scm.com/)

## Installation & Setup

### 1. Clone the Repository

```bash
git clone https://github.com/ths-dev-works/ths-dev-works.github.io.git
cd ths-dev-works.github.io
```

### 2. Initialize Git Submodules

The project uses the PaperMod theme as a Git submodule:

```bash
git submodule update --init --recursive
```

### 3. Install Hugo

#### macOS (using Homebrew)
```bash
brew install hugo
```

#### Windows (using Chocolatey)
```bash
choco install hugo-extended
```

#### Linux (using Snap)
```bash
sudo snap install hugo-extended
```

#### From Source
```bash
go install -tags extended github.com/gohugoio/hugo@latest
```

## Running the Development Server

Navigate to the project directory and start the development server:

```bash
cd home
hugo server -D
```

The `-D` flag includes draft content during development.

The site will be available at `http://localhost:1313`

### Additional Development Options

```bash
# Run with specific port
hugo server -D --port 1337

# Run with live reload (default)
hugo server -D --navigateToChanged

# Build and serve without drafts
hugo server

# Run with verbose output
hugo server -D --verbose
```

## Building for Production

To build the site for production:

```bash
cd home
hugo --minify
```

The static files will be generated in the `public/` directory.

## Project Structure

```
ths-dev-works/
├── .github/workflows/     # GitHub Actions CI/CD
├── home/                  # Hugo site root
│   ├── archetypes/        # Content templates
│   ├── assets/           # SCSS, JS, and other assets
│   ├── content/          # Site content (Markdown files)
│   ├── data/             # Data files
│   ├── i18n/             # Internationalization files
│   ├── layouts/          # Custom templates
│   ├── public/           # Generated static site (build output)
│   ├── static/           # Static assets (images, CSS, JS)
│   ├── themes/           # Hugo themes (PaperMod submodule)
│   └── hugo.yaml         # Hugo configuration
├── .gitmodules           # Git submodules configuration
└── README.md            # This file
```

## Multilingual Support

The site supports two languages:
- **English** (default): `/`
- **Português**: `/pt/`

Content is organized in the `content/` directory with language-specific folders:
```
content/
├── _index.md           # English homepage
├── about/              # English pages
├── services/           # English pages
├── pt/
│   ├── _index.md       # Portuguese homepage
│   ├── about/          # Portuguese pages
│   └── services/       # Portuguese pages
```

## Creating New Content

### Create a New Page

```bash
# English content
hugo new content/about/new-page.md

# Portuguese content
hugo new content pt/about/nova-pagina.md
```

### Create a New Post

```bash
# English post
hugo new content posts/new-post.md

# Portuguese post
hugo new content pt/posts/novo-post.md
```

## Customization

### Theme Configuration

The site uses the PaperMod theme. Configuration is in `home/hugo.yaml`. Key sections:

- **Languages**: Configure multilingual settings
- **Params**: Site-wide parameters and theme settings
- **Menu**: Navigation menu items
- **Social**: Social media links

### Adding Images

Place images in the `static/images/` directory:

```bash
# Add to static/images/
cp your-image.png home/static/images/
```

Reference in content:
```markdown
![Alt text](/images/your-image.png)
```

## Deployment

### GitHub Pages (Automatic)

The site is automatically deployed to GitHub Pages via GitHub Actions when pushing to the `main` branch.

### Manual Deployment

1. Build the site:
   ```bash
   cd home
   hugo --minify
   ```

2. Deploy the `public/` directory to your hosting provider.

## SEO & Analytics

- **Meta Tags**: Configured in `hugo.yaml`
- **Open Graph**: Social media sharing optimization
- **Google Analytics**: Add tracking ID in `params.analytics.google`
- **Sitemap**: Automatically generated at `/sitemap.xml`
- **Robots.txt**: Enabled for search engine crawling

## Development Workflow

1. **Make changes** to content or configuration
2. **Test locally**: `hugo server -D`
3. **Build**: `hugo --minify`
4. **Deploy**: Push to `main` branch (automatic) or manual deployment

## Useful Hugo Commands

```bash
# Check Hugo version
hugo version

# Create new site (for reference)
hugo new site my-site

# Clean build cache
hugo --gc

# Show build statistics
hugo --stats

# Validate configuration
hugo config

# Show help
hugo help
```

## Troubleshooting

### Common Issues

1. **Submodule not initialized**: Run `git submodule update --init --recursive`
2. **Theme not found**: Ensure PaperMod submodule is properly initialized
3. **Build errors**: Check Hugo version compatibility (use latest extended version)
4. **Port already in use**: Use `--port` flag to specify different port

### Getting Help

- [Hugo Documentation](https://gohugo.io/documentation/)
- [PaperMod Theme Docs](https://github.com/adityatelange/hugo-PaperMod/wiki)
- [Hugo Forums](https://discourse.gohugo.io/)

## License

This project is open source and available under the [MIT License](LICENSE).
