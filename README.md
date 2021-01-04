---
nav_order: 0
---
# Handbook

This contains information for how MIT's Office of Open Learning (OL) does things. The breadth of this covers coding conventions to project requirements and more.

To make changes, issue a pull request. Discussion can happen within the pull request itself.


## Navigate through docs via a locally-running static site

### Prerequisites

To run this locally, you need Ruby and the bunder gem installed. The recommended steps for that installation are as follows:

1. Install `rvm`
    - Ruby can be installed in other ways, but using `rvm` is **highly** recommend for installing/managing your ruby versions.
    - **Linux users**: It's **NOT** recommended to use the Ubuntu/debian package for this as it never seems to work. Follow the instructions in [this section](https://rvm.io/rvm/install#any-other-system) instead.
    - **OSX users**:
      1. Update Homebrew (`brew update`)
      1. Follow the [rvm installation guide](https://rvm.io/rvm/install#installing-rvm). Running the `gpg` command and the first `curl` command should work (confirmed January 2021).
- `rvm install RUBY_VERSION` - as of this writing, Ruby 2.7.1 was the latest and known to work
- `gem install bundler`


### Build and run jekyll


To run this locally, run the following commands in the handbook repo directory:

- `bundler install` - installs github's flavor of jekyll and its dependencies
- `bundler exec jekyll serve` - runs the site at [http://localhost:4000/](http://localhost:4000/)
