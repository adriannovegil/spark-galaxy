# Running Galaxy in a Production Environment

The basic installation instructions are suitable for development by a single user, but when setting up Galaxy for a multi-user production environment, there are some additional steps that should be taken for the best performance.

## Why?

By default, Galaxy:

 * Uses SQLite (a serverless database), so you dont't have to run/configure a database server for quick or basic development. However, SQLite does not handle concurrency.
 * Uses a built-in HTTP server, written in Python. Much of the work performed by this server can be moved to Nginx or Apache, wich will increase performance.
 * Runs all tools locally. Moving to a cluster will greatly increase capacity.
 * Runs in a single process, wich is a performance problem in CPython.

Galaxy ships with this default configuration to ensure the simplest, most error-proof configuration possible when doing basic development. As you'll soon see, the goal is to remove as much as work as possible from the Galaxy process, since doing so will gratly speed up the performance of it remaining duties. This is due to the Python Global Interpreter Lock (GIL).

## Groundwork for scalability

### Use a clean environment

Many of the following instructions are best practices for any production application.

 * Create a __NON-ROOT__ user called ```galaxy```. Running as an existing user will cause problems down the line when you want to grant or restrict access to data.
 * Start with a fresh checkout of Galaxy, don't try to convert one previously used for development. Download and install it in the galaxy user home directory.
 * Galaxy should be a managed system service (like Apahce, mail servers, database servers, etc.) run by the galaxy user.Init script, OS X launchd definitions and Solaris SMF manifest are provided in the ```contrib/``` directory of the distribution. You can also use the ```--daemon``` and ```--stop-daemon``` arguments to ```run.sh``` to start and stop by hand, but still run detached. When running as a daemon the server's output log will be written to ```paster.log``` instead of the terminal, unless instructed otherwise with the ```--log-file``` argument.
 * Give Galaxy its own database user and database to prevent Galaxy's schema from conflicting with other tables in your database. Also, restrict Galaxy's database user so it only has access to ist own database.
 * Make sure Galaxy is using a clean Python interpreter. Conflicts in ```$PYTHONPATH``` or the interpreter's ```site-packages/``` directory could cause problems. Galaxy manages its own dependencies for the framework, so you do not need to worry about these. The easiest way to do this is with a ```virtualenv```:

 ```
  $ wget http://bitbucket.org/ianb/virtualenv/raw/tip/virtualenv.py
  --11:18:05--  http://bitbucket.org/ianb/virtualenv/raw/tip/virtualenv.py
             => `virtualenv.py'
  Resolving bitbucket.org... 184.73.244.143, 184.73.226.175
  Connecting to bitbucket.org|184.73.244.143|:80... connected.
  HTTP request sent, awaiting response... 200 OK
  Length: 68,601 (67K) [text/x-python]

  100%[====================================>] 68,601        --.--K/s

  11:18:05 (729.46 KB/s) - `virtualenv.py' saved [68601/68601]

  $ /usr/bin/python2.7 virtualenv.py --no-site-packages galaxy_env
  New python executable in galaxy_env/bin/python2.7
  Also creating executable in galaxy_env/bin/python
  Installing setuptools...............done.
  nate@weyerbacher% . ./galaxy_env/bin/activate
  nate@weyerbacher% cd galaxy-dist
  nate@weyerbacher% sh run.sh
 ```

 * Galaxy can be housed in a cluster/network filesystem (it's been tested with NFS and GPFS), and you'll want to do this if you'll be running it on a cluster.

 ## Basic Configuration

The steps to install Galaxy mostly follow those of the regular instructions ad Admin/GetGalaxy. The difference is that after performing the groundwork above, you shuold initialize the configuration fil (```cp config/galaxy.ini.sample config/galaxy.ini```) and modify it as outlined below before starting the server. If you make any changes to this configuration file while the server is running, you will have to restart the server for the changes to take effect.
