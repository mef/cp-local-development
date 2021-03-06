# Community Platform (Zen) Local Development

Instructions for setting up local development for the CoderDojo Community on Mac and Linux. There are four main parts to setting up your local development environment:

* Install the tools (node.js, PostgreSQL, etc)
* Setting up the Community Platform code
* Loading test data into your fresh setup
* Making code changes

## Install Tools

To develop the Community Platform you need the following tools installed:

* [Node.js](http://nodejs.org) version 0.10.38 - ideally installed with [nvm](https://github.com/creationix/nvm) as described in this [article](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-with-nvm-node-version-manager-on-a-vps). Note that **only** node 0.10.x is supported currently.

* [PostgreSQL](http://www.postgresql.org/) version 9.4 - see [here](https://wiki.postgresql.org/wiki/Detailed_installation_guides) for installation instructions for your platform. You also may want to install the [pgAdmin](http://www.pgadmin.org/). When PostgreSQL is installed, you need to create a new user (you can do this from pgAdmin 'Login Roles' in the tree):
  * username: platform
  * password: QdYx3D5y

Give the new `platform` user all the admin privileges, i.e. make them a super user.

You also need to have [Git](https://git-scm.com/) installed in order to get the Community Platform code from [GitHub](https://github.com/coderdojo). You will also need to have [Grunt](http://gruntjs.com/) installed globally: `npm install -g grunt`.

## Code Setup

Next step is to get the Community Platform code cloned and up and running. To do that you clone this repo and use the `localdev` command to both setup and run the code:

```
$ git clone https://github.com/CoderDojo/cp-local-development.git && cd cp-local-development
$ npm install
```

Then run `./localdev.js` to make sure the `localdev` tool can run ok. You should see output similar to the following:

```
Usage "./localdev.js <command>" where command is one of:
  "init <system>": does a fresh setup of your local dev environment
  "run <system>": runs all the services in your system
  "testdata <system>": loads test data for each service
```

The `localdev` tool has one system configured, called `zen` (this can be extended to use multiple systems, but for now, 'zen' is the only one).

### localdev init

Next, run `./localdev.js init zen`. Behind the scenes this does the following:

* creates a `workspace-zen` directory
* install each service defined in the system, see [system.js](system.js)
* each service is cloned from github
* the `master` branch is checked out 
* an npm install is done in each repo
* connects to your local PostgreSQL and creates any databases necessary

Note that the `init` command is idempotent, you can run it multiple times with no adverse side effects.

### localdev run

Next, run `./localdev.js run zen`. Behind the scenes (see [run.js](run.js)) this does the following:

* sets up 'watchers' for each service in the system
* runs each service

The output from each service is displayed in the `run` shell. Note that this is not a daemon process, the `run` command will not exit until you hit ctrl-c. The 'watchers' detect any changes to code files in any of the service code repos, if a file is changed the service is restarted automatically. This allows for really quick development.

Note that you can also run services individually if you wish, e.g. `./localdev.js run zen cp-zen-platform`.

Once `run` looks to be running all the services ok (you'll see a lot of stack traces in the output if they are not running ok!) you should be able to hit `http://localhost:8000` in your browser. If this is your first time running, you should see the world map but with no dojo markers, these will appear when we install some test data.

Note that the Forums and Badges will not be operable in local development mode, to run these, you need to install both [NodeBB](www.nodebb.org) and [BadgeKit](http://badgekit.openbadges.org/) locally.

## Test Data

Next, in another shell (the system has to be running before you load the test data), run `./localdev.js testdata zen`. Behind the scenes, each service has a mechanism for loading test data into its database, and this command simply tells each service to load its test data.

Note that this command will fail if run more than once.

When all the test data is loaded, you should see Dojos appearing when you refresh your home page. You should also be able to login as an Admin user with the following credentials:

* user: manager@example.com
* password: test

## Making code changes and working locally

When you initialise a system, it creates a `workspace-<systemName>` folder for each system, e.g. `workspace-zen`. If you open this directory in your code editor you will see all the code repositories that make up this system, e.g. cp-zen-platform, cp-dojos-service, cp-users-service, cp-countries-service, etc. When you first set it up, the `init` command will checkout the default branch for that system, e.g. `my-zen-branch` for each service. From then on, it's up to you to manage the contents of this directory, e.g. creating branches, changing branches, etc. The `run` command will simply run whatever is in those directories, it doesn't care about what branch they're on, etc.

So a typical development workflow would be:

```
$ cd ./workspace-zen/cp-zen-platform
$ git checkout -b my-new-branch
$ # make changes to code in your editor of choice. Any time code changes in a service the `run` command will automatically reload the service
$ git push -u origin my-new-branch
$ # to pull request, code review, merge, etc on github
```

To update your forked repository:

```
$ cd ./forked-repository
$ # add another remote named 'upstream'
$ git remote add upstream git://github.com/CoderDojo/original-repository
$ git fetch upstream

$ # then: (like "git pull" which is fetch + merge)
$ git merge upstream/master master

$ # or, better, replay your local work on top of the fetched branch
$ # like a "git pull --rebase"
$ git rebase upstream/master
```

## The `localdev` tool

This tool is designed to make it as easy as possible to on-board new developers to the Community Platform. It is also designed to be cross platform, so developers can contribute to the community on their OS of choice, e.g. Windows, Mac & Linux.

The [system.js](system.js) file is a mixture of code and data, and it's where the 'systems' are defined (just 'zen' now). At its core, each System contains a set of Services, and each service has its own code repository on github. When each command starts, it prints out the full information of the system it's using, e.g.

```
zen: {
  systemBranch: 'master',
  services: [{
    name: 'cp-salesforce-service',
    repo: 'git@github.com:CoderDojo/cp-salesforce-service',
    branch: 'master',
    start: './start.sh empty service.js',
    env: {
      POSTGRES_USERNAME: 'platform',
...
```

### Environment values

`localdev` has four places where environment variables can be set:

* env vars that are global to all systems, these are read first
* env vars that are system specific, these are read next
* env vars that are service specific, these are read next
* local setup specific variables, these are read last

Settings at any level will override existing settings, so for example anything you set in your local setup will override any previous setting.

See [system.js](system.js) for global, system and service environment variables.

To set local environment variables, do the following:

* create a file called `local-dev.js`. Note this file is in the `.gitignore` so it won't be added to git
* override any variables you want as follows:

```
module.exports = {
  POSTGRES_HOST: '192.168.59.103',
  SALESFORCE_ENABLED: 'false'
}
```

### Debug

`localdev` uses the [Debug](http://npm.im/debug) module, to get extra debug information, run commands prefixed with 'DEBUG=localdev:* ..', e.g.

```
DEBUG=localdev:* ./localdev.js run zen
```

### Windows

Support for developing the platform on Windows is currently work in progress, and we would welcome contributions here.

In addition to the tools mentioned above, (Git, Node, PostgreSql, etc), on Windows, you need to install additional tools:

* [Microsoft Visual Studio 2015](https://www.visualstudio.com/en-us/downloads/download-visual-studio-vs.aspx)
* [Python 2.7](https://www.python.org/downloads)

Both of these are needed in order to compile native node.js modules (which are built in c++).
