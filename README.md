# Documentation

A GitHub Pages documentation site built with [Jekyll](https://jekyllrb.com/) and the [just-the-docs](https://just-the-docs.com/) theme.

## 🌐 Live Site

Visit the documentation at: **https://arnabnandy1706.github.io/documentation**

## 📁 Structure

```
documentation/
├── _config.yml            # Jekyll site configuration
├── Gemfile                # Ruby gem dependencies
├── index.md               # Home page
├── docs/                  # Documentation pages
│   ├── getting-started.md
│   ├── usage.md
│   ├── configuration.md
│   └── contributing.md
└── .github/
    └── workflows/
        └── deploy.yml     # GitHub Actions deployment workflow
```

## 🚀 Deployment

The site is automatically built and deployed to GitHub Pages whenever changes are pushed to the `main` branch via the GitHub Actions workflow in `.github/workflows/deploy.yml`.

To enable GitHub Pages:
1. Go to **Settings → Pages** in this repository.
2. Under **Source**, select **GitHub Actions**.

## 🛠 Local Development

1. Install [Ruby](https://www.ruby-lang.org/en/downloads/) and [Bundler](https://bundler.io/).
2. Install dependencies:
   ```bash
   bundle install
   ```
3. Serve the site locally:
   ```bash
   bundle exec jekyll serve
   ```
4. Open `http://localhost:4000` in your browser.

## 📝 Contributing

See [Contributing](docs/contributing.md) for guidelines on how to contribute.

## 📄 License

MIT License
