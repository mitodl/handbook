# Docker/Django/Webpack Web App Guide

**SECTIONS**
1. [Initial Setup](#initial-setup)
1. [Running and Accessing the App](#running-and-accessing-the-app)
1. [Testing and Formatting](#testing-and-formatting)
1. [Administering your Local App](#administering-your-local-app)
1. [Troubleshooting](#troubleshooting)


# Initial Setup

### Major Dependencies
- Docker
  - _[OSX only]_ We use **Docker for Mac**&#42;, a desktop development environment that includes Docker.  
    Recommended OSX install method: [Download from Docker website](https://store.docker.com/editions/community/docker-ce-desktop-mac)
- docker-compose
  - Recommended install: pip (`pip install docker-compose`)

_&#42; For OSX development, we previously used docker-machine, which is used to run Docker containers
inside of a VM. Since that time, Docker for Mac has improved to the point that Docker can be run from
the host machine._

### Build And Configure Docker Containers

All commands in this guide should be run from the root directory of your project's repository (unless specified otherwise).

#### 1) Create your ``.env`` file

    cp .env.example .env

The `.env.example` file contains settings variables for which you will need to provide values in order to run the
app. Sometimes default values are given. To get a better idea about what values are needed, refer to the specific
project's README file.

#### 2) Build the containers

    docker-compose build

*NOTE: You will also need to run this command whenever requirements files change (``requirements.txt``, ``test_requirements.txt``, etc.)*

#### 3) Create database tables from the Django models

    docker-compose run web ./manage.py migrate

*NOTE: You will also need to run this command whenever there are new migrations (i.e.: database models have been changed/added/removed).*

#### _(Optional)_ Create a superuser
Some of our apps include user creation as part of their specific setup steps. If the given app does not
include steps to create a user, you can create one easily via Django's `createsuperuser` command.
It will prompt you for the username and some other details for this user.

    docker-compose run web ./manage.py createsuperuser

### Add `/etc/hosts` alias for the site

There are two scenarios where this will be needed:

1. Multiple locally-running apps need to share a cookie (e.g.: MicroMasters and Open Discussions).
1. You are an OSX user. Due to networking differences between Docker for Mac and standard Docker, locally running apps can only communicate
with each other in OSX if `/etc/hosts` aliases are created for each app.

Our established pattern is to use `odl.local` as the domain. The `/etc/hosts` entry for a locally-running site will look like this:

```
127.0.0.1       <site_abbreviation>.odl.local

# Example: for Micromasters...
127.0.0.1       mm.odl.local
# Example: for open-discussions...
127.0.0.1       od.odl.local
```

# Running and Accessing the App

#### 1) Run the containers

Start all the services that are required to run the app:

    docker-compose up

#### 2) Navigate to the running app in your browser

Your app should now be accessible via browser:

1. The URL of the locally-running site needs to include the port number that the `nginx` service is running on.
    - One-line command to figure out that port number: `docker-compose ps nginx | perl -nle '/[0-9\.]*:(\d+)/ && print "$1";'`
    - This port number is specified for the `nginx` service in `docker-compose.yml`
1. _[Linux only]:_ Navigate to `localhost:PORT` (e.g.: `localhost:8079`)
1. _[OSX only]:_ Navigate to `ETC_HOSTS_ALIAS:PORT` (e.g.: `mm.odl.local:8079`). Like Linux users, you _can_ navigate to `localhost:PORT` (e.g.: `localhost:8079`) to use the
  locally-running site, but it's recommended/essential that you add an `/etc/hosts` alias and use that URL instead.
  More info on that in the section above.


# Testing and Formatting

There are a few different commands for running tests/linters and formatting code.

*NOTE: The `--rm` option for the `docker-compose run` command tells Docker to destroy the container after it finishes running. This is useful for running specific commands in one-off containers. This prevents the accumulation of unused Docker containers on your machine.*

### Python tests/linting/formatting

We use `pytest` (with various plugins) to run our Python test suite in most of our projects. In some of legacy projects, we use `tox` to invoke `pytest`. You'll run `pytest` if there is no `tox` requirement in `test_requirements.txt` for your project.

##### With `pytest`...
```bash
# Run Python tests with linting
docker-compose run --rm web pytest
# Run Python tests in a single file
docker-compose run --rm web pytest /path/to/test.py
# Run Python test cases in a single file that match some function/class name
docker-compose run --rm web pytest /path/to/test.py -k test_some_logic
# Run Python tests without linter, without coverage report, and without log capture
docker-compose run --rm web pytest --no-cov --no-pylint --show-capture=no
# Some of our projects allow you to pass in a single flag to run tests only (no linter, cov report, or log capture)
docker-compose run --rm web pytest --simple

### Linting
# If the pytest-pylint is NOT installed, run pylint directly
docker-compose run --rm web pylint
# If the pytest-pylint IS installed, run the pylint via pytest
docker-compose run --rm web pytest --pylint -m pylint
```

##### With `tox`...
```bash
# Run Python tests
docker-compose run --rm web tox
# Run Python tests in a single file
docker-compose run --rm web tox /path/to/test.py
# Run Python test cases in a single file that match some function/class name
docker-compose run --rm web tox /path/to/test.py -- -k test_some_logic
# Run Python linter
docker-compose run --rm web pylint
```

##### Formatting
Most of our projects use the `black` formatter to enforce certain formatting rules. This should be run before any commit that makes changes to Python code (ignore this if your project does not list `black` in the `test_requirements.txt`).
```bash
# Format all Python files in the repo
docker-compose run --rm web black .
# Format a specific file
docker-compose run --rm web black /path/to/file.py
```

##### Speeding up test development
There are many scenarios where you'll want to run tests many times in a row (authoring new tests, fixing old tests and checking if they pass). In that case it will save time to run a bash shell in a new container and run these commands as needed in that container.

```bash
# On host machine...
docker-compose run --rm web bash
# On the bash prompt inside the new container
# Without tox...
pytest /path/to/test.py
# With tox...
tox /path/to/test.py
```

### JS/CSS tests/linting/formatting

```bash
# We include a helper script to execute JS tests in most of our projects
# (this file may exist at the project root or in ./scripts/test)
docker-compose run --rm watch ./scripts/test/js_test.sh
# Run JS tests in specific file
docker-compose run --rm watch ./scripts/test/js_test.sh path/to/file.js
# Run JS tests in specific file with a description that matches some text
docker-compose run --rm watch ./scripts/test/js_test.sh path/to/file.js "should test basic arithmetic"
# Run the JS linter
docker-compose run --rm watch npm run lint
# Run SCSS linter
docker-compose run --rm watch npm run scss_lint
# Run the Flow type checker
docker-compose run --rm watch npm run-script flow
# Run prettier-eslint, which fixes style issues that may be causing the build to fail
docker-compose run --rm watch npm run fmt
```

**NOTE:** In some of our projects we include a shell script that runs the entire test suite: `./test_suite.sh`

##### Speeding up test development
You can speed up JS test development in the same way described in the Python testing section above by starting the
`watch` container instead of the `web` container.


# Administering your Local App

#### Running a Django shell

    docker-compose run --rm web ./manage.py shell

#### Running the app in a notebook

Some of our repos include a config for running a [Jupyter notebook](https://jupyter.org/) in a
Docker container. This enables you to do in a Jupyter notebook anything you might otherwise do
in a Django shell, with the added benefit of saving code that you frequently run, and running entire
blocks of code at once. **If the repo includes a `.ipynb.example` file, that means the app is configured
to run in a Notebook.** To get started:

- Copy the example file
    ```bash
    # Choose any name for the resulting .ipynb file
    cp localdev/app.ipynb.example localdev/app.ipynb
    ```
- Build the `notebook` container _(for first time use, or when requirements change)_
    ```bash
    docker-compose -f docker-compose-notebook.yml build
    ```
- Run all the standard containers (`docker-compose up`)
- In another terminal window, run the `notebook` container
    ```bash
    docker-compose -f docker-compose-notebook.yml run --rm --service-ports notebook
    ```
- Visit the running notebook server in your browser. The `notebook` container log output will
  indicate the URL and `token` param with some output that looks like this:
    ```
    notebook_1  |     To access the notebook, open this file in a browser:
    notebook_1  |         file:///home/mitodl/.local/share/jupyter/runtime/nbserver-8-open.html
    notebook_1  |     Or copy and paste one of these URLs:
    notebook_1  |         http://(2c19429d04d0 or 127.0.0.1):8080/?token=2566e5cbcd723e47bdb1b058398d6bb9fbf7a31397e752ea
    ```
  Here is a one-line command that will produce a browser-ready URL from that output. Run this in a separate terminal:
    ```bash
    # Replace "myapp.odl.local" with the etc/hosts alias for your app (e.g.: "xpro.odl.local")
    APP_HOST="myapp.odl.local"; docker logs $(docker ps --format '{{.Names}}' | grep "_notebook_run_") | grep -E "http://(.*):8080[^ ]+\w" | tail -1 | sed -e 's/^[[:space:]]*//' | sed -e "s/(.*)/$APP_HOST/"
    ```
  OSX users can pipe that output to `xargs open` to open a browser window directly with the URL from that command.
- Navigate to the `.ipynb` file that you created and click it to run the notebook
- Execute the first block to confirm it's working properly (click inside the block
  and press Shift+Enter)

From there, you should be able to run code snippets with a live Django app just like you
would in a Django shell.


# Troubleshooting

#### Error message indicating that the `auth_user` table doesn't exist

Try running `docker-compose run web ./manage.py migrate auth`, then run `docker-compose run web ./manage.py migrate`.

#### _[OSX only]_ Error indicating that Docker for Mac is not running (even though it is running)

Open a new terminal tab/window, navigate to the same directory, and re-run the same command (e.g.: `docker-compose up`). For whatever reason, a terminal window can sometimes fail to recognize that Docker for Mac is running, and a fresh terminal window will fix that.
