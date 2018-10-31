# ODL Docker/Django/Webpack Web App Guide

**SECTIONS**
1. [Initial Setup](#initial-setup)
2. [Running and Accessing the App](#running-and-accessing-the-app)
3. [Troubleshooting](#troubleshooting)


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

All commands in this guide should be run from the root directory of your project's repository (unless specified otherwise).

### Build And Configure Docker Containers

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


# Testing

There are a few different commands for running tests/linters:

```bash
# In most of our projects we include a shell script that runs 
# the entire test suite.
./test_suite.sh

### PYTHON TESTS/LINTING
# Run Python tests
docker-compose run web tox
# Run Python tests in a single file
docker-compose run web tox /path/to/test.py
# Run Python test cases in a single file that match some function/class name
docker-compose run web tox /path/to/test.py -- -k test_some_logic
# Run Python linter
docker-compose run web pylint

### JS/CSS TESTS/LINTING
# We also include a helper script to execute JS tests in most of our projects 
# (this file may exist at the project root or in ./scripts/test)
docker-compose run watch ./js_test.sh
# Run JS tests in specific file
docker-compose run watch ./js_test.sh path/to/file.js
# Run JS tests in specific file with a description that matches some text
docker-compose run watch ./js_test.sh path/to/file.js "should test basic arithmetic"
# Run the JS linter
docker-compose run watch npm run lint
# Run SCSS linter
docker-compose run watch npm run scss_lint
# Run the Flow type checker
docker-compose run watch npm run-script flow

# Run prettier-eslint, fixes style issues that may be causing the build to fail
docker-compose run watch npm run fmt
```


# Running Commands

You can run a Django shell with the following command:

    docker-compose run web ./manage.py shell


# Troubleshooting

#### Error message indicating that the `auth_user` table doesn't exist

Try running `docker-compose run web ./manage.py migrate auth`, then run `docker-compose run web ./manage.py migrate`.

#### _[OSX only]_ Error indicating that Docker for Mac is not running (even though it is running)

Open a new terminal tab/window, navigate to the same directory, and re-run the same command (e.g.: `docker-compose up`). For whatever reason, a terminal window can sometimes fail to recognize that Docker for Mac is running, and a fresh terminal window will fix that.
