# Repository Guidelines

## Project Structure & Module Organization
- `_config.yml`: site-wide settings, plugin toggles, metadata, and deploy URLs.
- `_posts/<category>/`: dated Markdown posts with YAML front matter for layout, categories, and tags.
- `_layouts/` & `_includes/`: HTML templates and reusable fragments extending the custom Minima theme.
- `_sass/minima/`: SCSS overrides compiled into `assets/css`; keep bespoke styling here.
- `assets/<yyyy-mm-dd-slug>/`: per-article media; use lowercase kebab names and reference via absolute paths.
- `_site/`: generated output; never edit or commit changes directly.

## Build, Test, and Development Commands
- `bundle install`: install or update gems listed in `Gemfile`.
- `bundle exec jekyll serve --livereload`: local server at `http://localhost:4000` with automatic reloads.
- `bundle exec jekyll build`: production build into `_site` for quick smoke checks.
- `JEKYLL_ENV=production bundle exec jekyll build`: mirrors the GitHub Pages pipeline for release validation.

## Coding Style & Naming Conventions
- Posts use `YYYY-MM-DD-slug.md`; front matter must include `layout`, `title`, `date`, `categories`, and `tags`.
- Markdown headings increase one level at a time; add code blocks with language hints (` ```ruby `, ` ```bash `).
- YAML lists indent by two spaces; wrap long arrays across lines for readability.
- SCSS changes live in `_sass/minima/custom-styles.scss`; grouped rules should include a short comment if non-obvious.
- Images and downloads reside under `assets/` with descriptive kebab-case filenames.

## Testing Guidelines
- Run `bundle exec jekyll build` before every PR to ensure the site compiles cleanly.
- `bundle exec jekyll doctor` surfaces configuration regressions and missing includes.
- Preview scheduled work with `bundle exec jekyll serve --drafts --future` to verify dates and pagination.
- Manually confirm new links and images from the local preview to prevent broken references.

## Commit & Pull Request Guidelines
- Follow Conventional Commits observed in history (e.g., `feat(posts): add tech sub categories`).
- Keep commits scoped; pair content updates with their supporting assets in the same change.
- PR descriptions should summarize the change, link related issues, and include screenshots for layout or style updates.
- After merging, monitor the GitHub Pages build and note any manual follow-up in the PR thread.

## Content Workflow
- Draft long-form pieces in `_drafts/`; move to `_posts/<category>/` when ready to publish.
- Update `_config.yml` when introducing new categories or collections so archives stay in sync.
- Extract repeated callouts into `_includes/*-notice.md` to reduce duplication across posts.
