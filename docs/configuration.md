---
layout: default
title: Configuration
nav_order: 4
---

# Configuration
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Site Configuration

The site is configured via `_config.yml` in the root of the repository.

### Basic Settings

| Option        | Description                          | Default         |
|:--------------|:-------------------------------------|:----------------|
| `title`       | The title of the site.               | `Documentation` |
| `description` | A short description of the site.     | *(empty)*       |
| `baseurl`     | The subpath of the site (e.g., `/documentation`). | *(empty)* |
| `url`         | The base hostname and protocol.      | *(empty)*       |

### Theme

This site uses the [just-the-docs](https://just-the-docs.com/) Jekyll theme. You can switch the color scheme in `_config.yml`:

```yaml
color_scheme: dark
```

Available schemes: `light` (default) and `dark`.

### Search

Search is enabled by default. To disable it:

```yaml
search_enabled: false
```

### Navigation

Control page ordering using `nav_order` in front matter. Lower numbers appear first in the sidebar.

```yaml
---
title: My Page
nav_order: 2
---
```

### Auxiliary Links

Add links to the top-right navigation bar:

```yaml
aux_links:
  "GitHub":
    - "https://github.com/arnabnandy1706/documentation"
```

## GitHub Pages Deployment

The site is deployed automatically via GitHub Actions whenever changes are pushed to the `main` branch. See [`.github/workflows/deploy.yml`](https://github.com/arnabnandy1706/documentation/blob/main/.github/workflows/deploy.yml) for the workflow configuration.
