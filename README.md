# KubeGene Docs

This repository contains the source code for the [kubegene.io](https://kubegene.io/).

We use [Hugo](https://gohugo.io/) to format and generate our docs website, and
[Netlify](https://www.netlify.com/) to manage the deployment of the site. Hugo
is an open-source static site generator that provides us with templates, content
organisation in a standard directory structure, and a website generation engine.
You write the pages in Markdown, and Hugo wraps them up into a website.

## Quickstart

Here's a quick guide to updating the docs. It assumes you're familiar with the
GitHub workflow and you're happy to use the automated preview of your doc
updates:

1. Fork the [kubegene/docs repo](https://github.com/kubegene/kubegene-website) on GitHub.
1. Make your changes and send a pull request (PR).
1. If you're not yet ready for a review, add a comment to the PR saying it's a
  work in progress. You can also add `/hold` in a comment to mark the PR as not
  ready for merge. (**Don't** add the Hugo declarative "draft = true" to the
  page front matter, because that will prevent the auto-deployment of the
  content preview described in the next point.)
1. Wait for the automated PR workflow to do some checks. When it's ready,
  you should see a comment like this: **deploy/netlify — Deploy preview ready!**
1. Click **Details** to the right of "Deploy preview ready" to see a preview
  of your updates.
1. Continue updating your doc until you're happy with it.

## Previewing your changes on a local website server

If you'd like to preview your doc updates as you work, you can install Hugo
and run a local server. This section shows you how.

### Install Hugo

See the [Hugo installation guide][hugo-install]. Here are some examples:

#### Mac OS X:

```
brew install hugo
```

#### Debian:

1. Download the latest Debian package from the [Hugo website][hugo-install].
  For example, `hugo_0.46_Linux-64bit.deb`.
1. Install the package using `dpkg`:

    ```
    sudo dpkg -i hugo_0.46_Linux-64bit.deb
    ```

1. Verify your installation:

    ```
    hugo version
    ```

### Clone the docs repo and run a local website server

Follow the usual GitHub workflow to fork the repo on GitHub and clone it to your
local machine, then use your local repo as input to your Hugo web server:

1. **Fork** the [kubegene/docs repo](https://github.com/kubegene/kubegene-website) in the GitHub UI.
1. Clone your fork locally. This example uses SSH cloning:

    ```
    mkdir kubegene.io
    cd kubegene.io/
    git clone git@github.com:<your-github-username>/kubegene-website.git
    cd kubegene-website/
    ```

1. Start your website server. Make sure you run this command from the
   `/kubegene-website/` directory, so that Hugo can find the config files it needs:

    ```
    hugo server -D
    ```

1. Your website is at [http://localhost:1313/](http://localhost:1313/).

1. Continue with the usual GitHub workflow to edit files, commit them, push the
  changes up to your fork, and create a pull request. (See below.)

1. While making the changes, you can preview them on your local version of the
  website at [http://localhost:1313/](http://localhost:1313/). Note that if you
  have more than one local git branch, when you switch between git branches the
  local website reflects the files in the current branch.

Useful Hugo docs:
- [Hugo installation guide](https://gohugo.io/getting-started/installing/)
- [Hugo basic usage](https://gohugo.io/getting-started/usage/)
- [Hugo site directory structure](https://gohugo.io/getting-started/directory-structure/)
- [hugo server reference](https://gohugo.io/commands/hugo_server/)
- [hugo new reference](https://gohugo.io/commands/hugo_new/)
