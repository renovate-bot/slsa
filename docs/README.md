# slsa.dev sources

This directory contains sources for https://slsa.dev, rendered with Jekyll and
served by [Netlify](https://netlify.com).

## Development

### Building and testing locally

1.  Clone this repo and change directory to `/docs`:

    ```bash
    git clone https://github.com/slsa-framework/slsa
    cd slsa/docs
    ```

2.  Install system dependencies:

    -   Ruby, bundler, and the dev headers:

        ```bash
        apt install ruby ruby-dev bundler
        ```

        Alternatively, you can use `rbenv` to use the exact version of Ruby,
        but Debian's versions are likely close enough.

    -   Node and NPM:

        ```bash
        curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
        nvm install 18
        nvm use 18
        ```

        See https://github.com/nvm-sh/nvm for more instructions.

    -   [Netlify CLI](https://docs.netlify.com/cli/get-started/):

        ```bash
        npm install -g netlify-cli
        netlify login
        ```

3.  Install local dependencies:

    ```bash
    bundle config set --local path 'vendor/bundle'
    bundle install
    ```

    You will need to re-run `bundle install` whenever the Gemfile.lock changes.

4.  (optional) To enable `jekyll-github-metadata` to read metadata about the
    slsa repository from the GitHub API, create a GitHub
    [personal access token](https://github.com/settings/tokens/new) and add it
    to your `~/.netrc`, like so:

    ```none
    machine api.github.com
        login github-username
        password 123abc-your-token
    ```

5.  Run the development server locally with
    [Netlify CLI](https://github.com/netlify/cli/blob/main/docs/netlify-dev.md):

    ```bash
    netlify dev
    ```

    If you would like livereload (autorefresh page after every change) and/or
    incremental builds (faster builds but possibly missing some changes), use:

    ```bash
    netlify dev -c 'bundle exec jekyll serve --livereload --incremental'
    ```

6.  Browse to http://localhost:8888 to view the site locally.

### Deploy previews

Netlify automatically builds and deploys previews of every pull request. Shortly
after creating a PR, Netlify will add a comment with a link to a preview. The
URL is of the form `https://deploy-preview-#--slsa.netlify.app` where `#` is the
pull request number. This preview is updated on every push.

### Comparing built versions

The script `../tools/diff_site` allows you to easily compare two different build
results, for example to check that an upgrade to a new version of Jekyll did not
break anything. It works with both locally built versions (`_site`) and archives
downloaded from Netlify (`deploy-*.zip`).

Example 1: comparing two locally built versions of the site

```bash
# Prepare version A 
$ bundle exec jekyll build
$ mv _site _site.A
# Prepare version B
$ bundle exec jekyll build
$ mv _site _site.B
# Run the script
$ ../tools/diff_site _site.A _site.B
```

Example 2: comparing a Netlify pull request preview to the latest production
version

Download the `deploy-*.zip` snapshots from
https://app.netlify.com/sites/slsa
([screenshot](../readme_images/netlify_download_screenshot.png)), one for the
latest production deploy and one for the pull request. **You must be
logged in to Netlify to see the Download link.** Then run:

```bash
../tools/diff_site deploy-latest.zip deploy-preview.zip
```

## Production

### Netlify configuration

Site configuration: https://app.netlify.com/sites/slsa \
Team configuration: https://app.netlify.com/teams/slsa

Prefer to configure the site using `netlify.toml` rather than the web UI, when
possible.

To be added to ACL to allow you to configure the site, contact Mark Lodato or
Joshua Lock via email or Slack. In the event that no SLSA team member has
access, contact OpenSSF.

### Production builds

Netlify automatically builds and deploys the `main` branch to https://slsa.dev.

### DNS

OpenSSF (Linux Foundation) owns the DNS registration for slsa.dev and runs the
DNS server. To request changes, email operations@openssf.org.

It is configured to point to Netlify:

```none
slsa.dev      ALIAS  apex-loadbalancer.netlify.com
www.slsa.dev  CNAME  slsa.netlify.app
```

## Playbooks

### Something is wrong with the site. How do I debug and/or roll back?

Go to https://app.netlify.com/sites/slsa/deploys?filter=main to see recent
deployments. You need to be logged into Netlify to see the list of deployments,
and in the "slsa" team to perform mutations (e.g. Publish).

View a previous version of the site by clicking on a deployment's date:

![screenshot of permalinks](../readme_images/netlify_permalinks_screenshot.png)

If you find that a previous version did not have the problem, you may roll back
to that version by clicking on the row (not the date) and then **Publish
deploy**.

![screenshot of rollback button](../readme_images/netlify_rollback_screenshot.png)

This will stay active until the next push to `main`.

## Insights on some of the internals of the SLSA website

This section provides a collection of tips on some of the internals of how the
slsa.dev website. This is not complete but it should list all the files to know
about when publishing a new version of the SLSA specification.

### Configuration files

Several configuration files govern the website layout and behavior.

#### docs/_data/versions.yml

This file defines the status of the different versions of the specification
(Draft, Approved, or Retired) along with which version is the current one
(which is used to bring up the header indicating the version being looked at is
not the current one with a pointer to the current version).

The `hidden` property was used by the version selector which was abandoned and
is no longer in use.

#### docs/_data/nav/*

This folder contains a set of files defining the navigation side bars to be
used for the different versions of the specification.

When the version selector was abandoned we switched to only using one global
navigation menu for all pages and versions of the spec meant to be accessible
through the navigation menu (i.e., not hidden). This menu is defined in
`main.yaml`.

The version specific files, such as `v0.1.yml`, are used when someone directly
accesses an older version which is "hidden" (i.e., not part of the main
navigation bar and only accessible via a direct link from an old blog post for
instance).

Care must be taken to ensure that every hidden version has an associated
navigation file or no navigation side menu will be rendered, making it
impossible to navigate through the pages of that version of the
specification. Practically speaking this means that, when you remove a specific
version from `main.yaml` you should always create a corresponding `v***.yaml`
if none exists already.

The content of the navigation files is used (in `docs/_includes/nav.html`) to
define the navigation side bar but also to define (in
`docs/_includes/pagination.html`) how pages are chained via the previous and
next links displayed at the bottom of a page).

The `config.yml` file defines the mapping of the version of the specification
taken from the page URL to the key in the site.data.nav array where the various
navigation menu definitions are stored.

### docs/_redirects

Typical website configuration type of file, defining a bunch of redirects used
to maintain backwards compatibility and provide for some resilience by defining
paths that can be updated without requiring any content change.

Addition of new redirects should be kept to a minimum to limit maintenance
burden.

## Publishing a new version of the SLSA specification

This lists some of the steps one must take to publish a new version of the specification:

-   Edit `docs/_data/versions.yml` to add the new version, with its status, and possibly update the status of the previous one.
-   Edit `docs/_data/nav/config.yml` to add the new version
-   Edit `docs/_data/main.yml` to add the new version and possibly remove any older versions to be hidden. If you remove an old version, make sure that a version specific file exists because it will be needed if the hidden version is accessed directly.
-   Edit `docs/_redirects` as necessary including the definition of `/spec/latest/*`
