---
layout: default
title: Getting Started
nav_order: 2
---

# Getting Started
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Prerequisites

Before you begin, ensure you have the following installed:

- **Git** – [Download Git](https://git-scm.com/downloads)
- **A text editor** – We recommend [VS Code](https://code.visualstudio.com/)

## Installation

### Step 1 – Clone the repository

```bash
git clone https://github.com/arnabnandy1706/documentation.git
cd documentation
```

### Step 2 – Explore the project

```
documentation/
├── _config.yml       # Jekyll configuration
├── index.md          # Home page
├── docs/             # Documentation pages
│   ├── getting-started.md
│   ├── usage.md
│   ├── configuration.md
│   └── contributing.md
└── .github/
    └── workflows/
        └── deploy.yml  # GitHub Actions deployment workflow
```

### Step 3 – Run locally (optional)

To preview the site locally, install [Jekyll](https://jekyllrb.com/docs/installation/) and run:

```bash
bundle exec jekyll serve
```

Then open your browser at `http://localhost:4000`.

## Next Steps

- Read the [Usage](usage) guide to learn about the core features.
- See the [Configuration](configuration) page to customize your setup.
