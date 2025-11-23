# Fabio Bombiela - Personal Website

Personal portfolio website built with [Academic Pages](https://github.com/academicpages/academicpages.github.io) template and hosted on GitHub Pages.

## About

This website showcases my professional profile, projects, talks, and teaching experience as an Automation Tester Engineer and Electronic Engineering graduate.

**Live site:** [https://fbombi13.github.io/FabioBombiela/](https://fbombi13.github.io/FabioBombiela/)

## Running Locally

### Prerequisites

- Ruby (with ruby-dev)
- Bundler
- Node.js

### Setup

```bash
# Install dependencies
bundle install

# Run local server
bundle exec jekyll serve -l -H localhost
```

Visit `http://localhost:4000` to view the site.

### Using Docker

```bash
chmod -R 777 .
docker compose up
```

### Using VS Code DevContainer

Open the repository in VS Code and select **Reopen in Container** when prompted.

## Repository Structure

- `_config.yml` - Site configuration
- `_pages/` - Main pages (About, CV, etc.)
- `_talks/` - Talk entries
- `_teaching/` - Teaching entries
- `_projects/` - Project portfolio
- `images/` - Images and assets
- `files/` - Downloadable files (PDFs, etc.)

## Credits

This site is based on the [Academic Pages](https://github.com/academicpages/academicpages.github.io) template, which was forked from [Minimal Mistakes Jekyll Theme](https://mmistakes.github.io/minimal-mistakes/).

## License

MIT License - See [LICENSE](LICENSE) for details.
