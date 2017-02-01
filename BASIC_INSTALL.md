# Basic Installation

In addition to using the public Galaxy server (a.k.a. [Main](https://wiki.galaxyproject.org/Main)), you can install your own instance of Galaxy (what this page is about), or create a cloud-based instance of Galaxy. Another option is to use one of the ever-increasing number of public Galaxies hosted by other organizations.

### Requirements

 * UNIX/Linux or Mac OS X (although you can [try with Windows](https://wiki.galaxyproject.org/Admin/Config/Windows))
 * Python 2.7 ([details here](https://wiki.galaxyproject.org/Admin/Python))
 * Git (optional - see below)
 * GNU Make, gcc to compile and install tool dependencies
 * Additional tool requirements as detailed in [Tool Dependencies](https://wiki.galaxyproject.org/Admin/Tools/ToolDependencies)

### Get the Code

Galaxy's source code is hosted in a GitHub repository. Below are your basic options on how to obtain the source code so you can use Galaxy. For more information see [source code details](https://wiki.galaxyproject.org/Develop/SourceCode).

#### For Production or Single User

If running a production Galaxy service or creating your own personal Galaxy server, you should use the latest release branch, wich only receives stable code updates.

If you do not have a Galaxy repository yet or you do not want to update the existing instance, run:

```
 $ git clone -b release_16.07 https://github.com/galaxyproject/galaxy.git
```

If you have an existing Galaxy repository you want to update, run:

```
 $ git checkout release_16.07 && git pull --ff-only origin release_16.07
```

#### For Development

If you are getting Galaxy for development, you want to use the default branch after cloning - dev. This is the branch that most pull request should be made against, if you are contributing code back (unless you are fixing a bug in a Galaxy release).

```
 $ git clone https://github.com/galaxyproject/galaxy.git
```

### Start It Up

Galaxy requires a few things to run - a virtualenv, configuration files, and dependent Python modules. However, starting the server for the first time will create/acquire these things as necessary. Symply run the following command:

```
 $ sh run.sh
```

This will start up the server on ```localhost``` and port ```8080```, so Galaxy can be accessed from your web web browser at ```http://localhost:8080```. Galaxy's server will start printing its output to your terminal. To stop the Galaxy server, just hit ```Ctrl-c``` in the terminal from wich Galaxy is running.

To access Galaxy over the network, simply modify the ```config/galaxy.ini``` file and change the ```host``` setting to

```
...
host = 0.0.0.0
...
```

Upon restarting, Galaxy will bind to any available network interfaces instead of just the loopback.

That's it, you have your very own Galaxy running. Congratulations!

### Become an Admin

In order to control your new Galaxy through the UI (installing tools, managing users, creating groups, etc.) you have to become an administrator. First register as a new user and then give the user admin privileges like this: Add the Galaxy login (email) to the Galaxy configuration file (```config/galaxy.ini```).

Check the ```config/galaxy.ini.sample``` file for the default settings.

```
 $ grep "admin_users" config/galaxy.ini
```

The result will be:

```
#admin_users = None
```

The command below will:

 1. remove the leading hash ```#``` character from the line
 2. replace ```None``` with the same email address you used or will use to register the admin account through the Galaxy web interface.
 3. create a ```config/galaxy.ini``` file. __Be careful to type the command exactly as written except for changing admin@email.edu to be your own admin email address__.

```
 $ sed 's/#admin_users = None/admin_users = admin@email.edu/' config/galaxy.ini.sample > config/galaxy.ini
```

Double check that the new ```config/galaxy.ini``` file has the replacement text correct. The result should be the first two changes above (the query is against the third change).

```
 $ grep "admin_users" config/galaxy.ini
```

For the example email address email address above, the result is:

```
admin_users = admin@email.edu
```

Note that you have to restart Galaxy after modifying the configuration for changes to take effect.
