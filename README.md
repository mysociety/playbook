# Playbook prototype

## How to run this site locally

    git clone --recursive git@github.com:mysociety/playbook.git
    cd playbook
    bundle install
    bundle exec jekyll serve --baseurl ''

The site will be accessible at <http://localhost:4000> by default.

## How this repo was initially set up

These instructions might help if you want to create another GOV.UK-inflected Jekyll site like this, from scratch, in the future.

1. Create a directory for the code to go into:
   ```sh
   mkdir playbook
   cd playbook
   ```

1. Copy this content…
   ```ruby
   source "https://rubygems.org"
   gem "jekyll"
   ```

   Then paste it into a `Gemfile`:
   ```sh
   pbpaste > Gemfile
   ```

   It would have been nice to define `gem "github-pages"` in our `Gemfile` instead of `gem "jekyll"` directly, to help ensure our [local development environment matches the production environment](https://help.github.com/en/articles/setting-up-your-github-pages-site-locally-with-jekyll) (on Github Pages). But something about the `github-pages` Gem causes the `govuk-frontend` Sass styles to take 6 times as long to compile (30 seconds with `github-pages`, versus 5 seconds with plain `jekyll`). The slowdown is likely caused by [one of the other Gems](https://rubygems.org/gems/github-pages) that `github-pages` installs. So in the meantime, just installing the `jekyll` Gem directly is an easy compromise.

1. Install the Gems using [Bundler](https://bundler.io/):
   ```sh
   bundle install
   ```

1. Set up the directory as a Git repo:
   ```sh
   git init
   ```

   You might want to do your initial commit now, before we start creating lots of new files. Up to you.

1. At this point it’d be nice to use `bundle exec jekyll new` to create a "blank" jekyll site, but `jekyll new` is pretty opinionated and creates a load of files that we’ll just have to edit or delete anyway. So it’s simpler just to create what we need, manually…

1. Create a `_config.yml` with the following content:
   ```yaml
   title: Default page title goes here
   description: Default page description goes here
   baseurl: /playbook
   host: 0.0.0.0
   markdown: kramdown
   ```

1. Create a `.gitignore` with the following content:
   ```
   _site
   .sass-cache
   .jekyll-metadata
   ```

1. Create a file at `_layouts/default.html` with the following content:
   ```html
   <!DOCTYPE html>
   <html class="govuk-template">
   <head>
       <meta charset="utf-8">
       <meta http-equiv="X-UA-Compatible" content="IE=edge">
       <meta name="viewport" content="width=device-width, initial-scale=1">
       <title>{% if page.title %}{{ page.title }}{% else %}{{ site.title }}{% endif %}</title>
       <meta name="description" content="{% if page.excerpt %}{{ page.excerpt | strip_html | strip_newlines | truncate: 160 }}{% else %}{{ site.description }}{% endif %}">
       <!--[if lt IE 9]>
       <script src="{{ "/vendor/html5shiv/html5shiv.js" | prepend: site.baseurl }}"></script>
       <![endif]-->
       <link rel="stylesheet" href="{{ "/assets/css/style.css" | prepend: site.baseurl }}">
   </head>
   <body class="govuk-template__body {% if page.bodyclass %}{{ page.bodyclass }}{% endif %}">
       <script>document.body.className = ((document.body.className) ? document.body.className + ' js-enabled' : 'js-enabled');</script>
       
       {{ content }}
       
       <script src="{{ "/govuk-frontend/package/all.js" | prepend: site.baseurl }}"></script>
       <script>window.GOVUKFrontend.initAll()</script>
   </body>
   </html>
   ```

1. Create an empty `_includes` directory. Liquid partials will go in here, ie: anything you want to call `{% include … %}` on in future.

1. Create an `index.html` file with the following content:
   ```html
   ---
   layout: default
   title: "Home"
   bodyclass: "home"
   ---
   
   <div class="govuk-width-container">
       <main class="govuk-main-wrapper" id="main-content" role="main">
           <h1 class="govuk-heading-xl">{{ page.title }}</h1>
           <a href="#" role="button" class="govuk-button govuk-button--start">Button</a>
       </main>
   </div>
   ```

1. Create a file at `assets/css/style.scss` with the following content:
   ```scss
   ---
   # This file is read as SCSS, and automatically compiled to CSS by Jekyll
   # See http://jekyllrb.com/docs/assets/
   ---
   @charset "utf-8";
   $baseurl: "{{ site.baseurl }}";
   
   @import "settings"; // from ../../_sass
   @import "all"; // from ../../govuk-frontend/src
   ```

1. Create a file at `_sass/_settings.scss` with the following content:
   ```scss
   $govuk-global-styles: true;
   $govuk-assets-path: "#{$baseurl}/govuk-frontend/src/assets/";
   ```

   As we work on the site, we’ll probably want to add more files to this `_sass` directory, which will then need to be pulled into the final compiled CSS files by `@import` directives in `assets/css/style.scss`.

1. Grab a copy of the latest `html5shiv.js`:
   ```sh
   mkdir -p vendor/html5shiv
   curl -o vendor/html5shiv/html5shiv.js 'https://raw.githubusercontent.com/aFarkas/html5shiv/master/dist/html5shiv.js'
   ```

1. Install [govuk-frontend](https://github.com/alphagov/govuk-frontend/) as a [Git submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules):
   ```sh
   git submodule add https://github.com/alphagov/govuk-frontend.git
   ```

   If you want to track a [particular release of `govuk-frontend`](https://github.com/mysociety/govuk-frontend/releases) then you can go into the submodule and checkout the relevant tag:
   ```sh
   cd govuk-frontend
   git checkout v2.13.0
   ```

1. Add the following `exclude` directive to your Jekyll `_config.yml`:
   ```yaml
   exclude:
     - Gemfile
     - Gemfile.lock
     - node_modules/
     - govuk-frontend/app/
     - govuk-frontend/bin/
     - govuk-frontend/config/
     - govuk-frontend/dist/
     - govuk-frontend/docs/
     - govuk-frontend/lib/
     - govuk-frontend/package/
     - govuk-frontend/tasks/
     - govuk-frontend/*.md
     - govuk-frontend/*.njk
     - govuk-frontend/*.json
   ```

1. Add the following `sass` directive to your Jekyll `_config.yml`:
   ```yaml
   sass:
     load_paths:
       - _sass
       - govuk-frontend/src
   ```

1. Compile the site with Jekyll:
   ```sh
   bundle exec jekyll serve --baseurl ''
   ```

1. Check `localhost:4000` in your browser. If everything’s worked you should see "Hello" in a large, GOV.UK-style typeface, and a green button with a white arrow.
