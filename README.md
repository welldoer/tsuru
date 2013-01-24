#Tsuru

[![Build Status](https://secure.travis-ci.org/globocom/tsuru.png?branch=master)](http://travis-ci.org/globocom/tsuru)

##What is Tsuru?

Tsuru is an open Platform-as-a-Service (PaaS).

##Installation

For development, check:
http://tsuru.readthedocs.org/en/latest/contribute/setting-up-your-tsuru-development-environment.html.

For execution environment (production, staging, etc.):
http://tsuru.readthedocs.org/en/latest/build.html

##Server configuration

Please refer to http://tsuru.rtfd.org/config.

##Usage

After installing the server, build the cmd/main.go file with the name you wish,
and add it to your $PATH. Here we'll call it `tsuru`. Then you must set the
target with your server url (tsuru target), like:

	% tsuru target tsuru.myhost.com
	% tsuru target localhost:8080

After that, all you need is create a user and authenticate to start creating
apps and pushing code to them:

	% tsuru user-create youremail@gmail.com
	% tsuru login youremail@gmail.com

Every command has a help, to access it, try:

	% tsuru help command

Then, create a team:

	% tsuru team-create myteam

Your user will be automatically added to this team.

To create an app:

	% tsuru app-create myapp gunicorn

The available frameworks are listed at https://github.com/globocom/charms.

The command output will return your the git remote url for your app, you should
add it to your git repository:

	% git remote add tsuru git@tsuru.myhost.com:myapp.git

When your app is ready, you can push to it. To check whether it is ready or
not, you can use:

	% tsuru app-list

This will return something like:

	+-------------+-------------------------+--------------------------------------------+
	| Application | Units State Summary     | IP                                         |
	+-------------+-------------------------+--------------------------------------------+
	| myapp       | 1 of 1 units in-service | myapp-88392040.us-east-1.elb.amazonaws.com |
	+-------------+-------------------------+--------------------------------------------+

You can try to push now, but you'll get a permission error, because you haven't
pushed your key yet.

	% tsuru key-add

This will search for a `id_rsa.pub` file in ~/.ssh/, if you don't have a
generated key yet, you should generate one before running this command. If your
public key is in some other format, you can pass it as argument to `key-add`
command:

	% tsuru key-add $HOME/.ssh/id_dsa.pub

Now you can push you application to your cloud:

	% git push tsuru master

After that, you can check your app's url in the browser and see your app there.
You'll probably need run migrations or other deploy related commands.

To run a single command, you should use the command line:

	% tsuru run --app myapp env/bin/python manage.py syncdb
	% tsuru run --app myapp env/bin/python manage.py migrate

By default, the commands are run from inside the app root directory, which is
/home/application/current. If you have more complicated deploy related
commands, you should use the app.conf pre-restart and pos-restart scripts,
those are run before and after the restart of your app, which is triggered
everytime you push code.

Below is a app.conf sample::

```yaml
pre-restart: deploy/pre.sh
pos-restart: deploy/pos.sh
```

The ``app.conf`` file is located in your app's root directory, and the scripts
path in the yaml are relative to it.
