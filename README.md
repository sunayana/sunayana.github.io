# Sunayana Ghosh — Personal Website

Personal website built with Hugo and the Blowfish theme, showcasing work in Medtech, Climate Tech, and Computational Geometry.

## Prerequisites

Hugo Extended v0.155.3 or later.

## Running Locally

```bash
hugo server
```

Visit `http://localhost:1313` to view the site.

## Site Structure

```
.
├── assets/css/              # Custom CSS and theme overrides
├── config/_default/         # Hugo and Blowfish configuration
├── content/                 # All markdown content
│   ├── about.md
│   ├── contact.md
│   ├── expertise.md
│   ├── publications.md
│   └── work/                # Work/project pages
├── layouts/partials/        # Theme layout overrides
├── static/                  # Static files (resume PDF, images)
└── themes/blowfish/         # Blowfish theme (git submodule)
```

## Deployment

The site deploys automatically to GitHub Pages via GitHub Actions on every push to `main`. To enable for a new repository:

1. Go to **Settings → Pages**
2. Set **Source** to **GitHub Actions**

## Theme

This site uses the [Blowfish](https://blowfish.page) theme.

## License

Content © 2026 Sunayana Ghosh. All rights reserved.
Theme: [Blowfish](https://github.com/nunocoracao/blowfish) (MIT License)
