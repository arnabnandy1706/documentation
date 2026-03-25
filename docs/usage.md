---
layout: default
title: Usage
nav_order: 3
---

# Usage
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Writing Documentation

Add Markdown files to the `docs/` folder. Each file should include a front matter block at the top:

```yaml
---
layout: default
title: My Page Title
nav_order: 4
---
```

| Field       | Description                                      |
|:------------|:-------------------------------------------------|
| `layout`    | Use `default` for standard documentation pages. |
| `title`     | The title shown in the navigation sidebar.       |
| `nav_order` | Controls the page order in the navigation menu.  |

## Adding Child Pages

To create a nested section, add a `parent` field in the child page's front matter:

```yaml
---
layout: default
title: Child Page
parent: Usage
nav_order: 1
---
```

## Using Callouts

The `just-the-docs` theme supports callout blocks for highlighting important content:

{: .note }
This is a **note** callout.

{: .warning }
This is a **warning** callout.

{: .important }
This is an **important** callout.

## Code Blocks

Use fenced code blocks with a language identifier for syntax highlighting:

````markdown
```python
def hello():
    print("Hello, world!")
```
````

Renders as:

```python
def hello():
    print("Hello, world!")
```
