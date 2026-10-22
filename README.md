# o3osatoshi blog

Personal blog about technology, books, and everyday learning.
Site: <https://blog.o3osatoshi.engr.work>

## Development

Use Ruby 3.3 and Bundler 2.5.7, then run:

```sh
bundle install
JEKYLL_ENV=production bundle exec jekyll build
bundle exec jekyll doctor
bundle exec jekyll serve
```

The locked GitHub Pages dependencies currently use Jekyll 3.9.5.
The theme is pinned to a reviewed commit in
[o3osatoshi-echo/minima](https://github.com/o3osatoshi-echo/minima).

## Publishing

Published article sources are under `_posts/`; their category-based URLs are
preserved. Put article images in `assets/` and preview before pushing to `main`.
Drafts, editable artwork, and previous Git history are retained privately and
must not be added to this public repository. Create drafts outside the checkout.

GitHub Pages builds the root of `main`. DNS and the custom domain are configured
separately after ownership verification; the domain is not changed by a local build.
The original Google Analytics configuration is retained. Comments are disabled
until a new Giscus installation is configured; old discussion data is not migrated.

## Rights

The Minima theme retains its upstream MIT license. No new license is granted for
article text, third-party covers, quotations, or attached materials by this migration.
