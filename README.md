Build trend
===========

[![Build Status](https://travis-ci.org/ruleant/buildtime-trend.svg)](https://travis-ci.org/ruleant/buildtime-trend)
[![Coverage Status](https://coveralls.io/repos/ruleant/buildtime-trend/badge.png?branch=master)](https://coveralls.io/r/ruleant/buildtime-trend?branch=master)
[![Code Health](https://landscape.io/github/ruleant/buildtime-trend/master/landscape.png)](https://landscape.io/github/ruleant/buildtime-trend/master)
[![status](https://sourcegraph.com/api/repos/github.com/ruleant/buildtime-trend/badges/status.png)](https://sourcegraph.com/github.com/ruleant/buildtime-trend)

Create a trendline of the different stages of a build process

Dependencies
------------

- lxml (python wrapper for libxml2 and libxslt)
- matplotlib v1.2.0 or higher (for drawing the trend graph, stackplot is introduced in v1.2.0)

Install using pip :

`pip install lxml 'matplotlib>=1.2.0'`

or as a Debian package :

`apt-get install python-lxml python-matplotlib`

Usage
-----

First the timestamp recording needs to be initialised :

`source /path/to/init.sh`

This script will detect the location of the build-trend script folder,
adds it to the PATH and cleans logfiles of previous runs.
Executing the init script with `source` is required to be able to export environment variables to the current shell session.

Because the script dir is added to PATH, no path needs to be added
when logging a timestamp :

`timestamp.sh eventname`

This will log the current timestamp to a file and display it on STDOUT.
Repeat this step as much as needed.

When finished, run 

`analyse.sh`

to analyse the logfile with timestamps and print out the results.
It will calculate the duration between the timestamps and add them to
a file with the analysed data of previous builds.
When run on Travis CI, it will automatically add build/job related info.

To generate a graph from previous builds, run

`generate_trend.py`

It will take the file with analysed data generated by the analyse script and turn it into a trend graph and saves this graph.

Use the `sync-buildtime-trend-with-gh-pages.sh` script when you run it as part of a Travis CI build. See below for details.

Integrate with Travis CI
------------------------

You can integrate Buildtime Trend with your build process on Travis CI : 
install and initialise the buildtime trend scripts, add timestamp labels, generate the trend
and synchronise it with your gh-pages branch.

All you need is a github repo, a travis account for your repo and a gh-pages branch to publish the results.

You also need to create an encrypted GH_TOKEN to get write access to your repo (publish results on gh-pages branch) :
- [create](https://github.com/settings/applications) the access token for your github repo, `repo` scope is sufficient
- encrypt the environment variable using the [travis tool](http://docs.travis-ci.com/user/encryption-keys/) :
`travis encrypt GH_TOKEN=github_access_token`
- add the encrypted token to your .travis.yml file (see below)
 
The generated trend graph and build-data will be put in folder `buildtime-trend` on your `gh-pages` branch.
The trend is available on http://{username}.github.io/{repo_name}/buildtime-trend/trend.png

Example `.travis.yml` file :

    language: python
    env:
      global:
        secure: # your secure GH_TOKEN goes here (required to share trend on gh-pages)
    before_install:
      # install and initialise build-trend scripts
      - git clone https://github.com/ruleant/buildtime-trend.git $HOME/buildtime-trend
      # initialise buildtime-trend scripts
      - source $HOME/buildtime-trend/init.sh
    install:
      # generate timestamp with label 'install'
      - timestamp.sh install
      # install buildtime-trend dependencies
      - CFLAGS="-O0" pip install -r ${BUILD_TREND_HOME}/requirements.txt
      # install dependencies
    script:
      # generate timestamp with label 'tests'
      - timestamp.sh tests
      # run your tests
    after_success:
      # generate timestamp with label 'after_success'
      - timestamp.sh after_success
      # after_success scripts
    after_script:
      # synchronise buildtime-trend result with gh-pages
      - sync-buildtime-trend-with-gh-pages.sh

Bugs and feature requests
-------------------------

Please report bugs and add feature requests in the Github [issue tracker](https://github.com/ruleant/buildtime-trend/issues).


License
-------

Copyright (C) 2014 Dieter Adriaenssens <ruleant@users.sourceforge.net>

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.
