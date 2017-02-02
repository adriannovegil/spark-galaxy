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

### Disable the Developer settings

Two options are set in the sample ```config/galaxy.ini``` wich should not be enabled on a production server. You should set both to ```False```:

 * ```debug = False``` - Disable middleware that loads the entire response in memory for displaying debugging information in the page. If left enabled, the proxy server may timeout waiting for a response or your Galaxy process may run out of memory if it's serving large files.
 * ```use_interactive = False``` - Disables displaying and live debugging of tracebacks via the web. Leaving it enabled will expose your configuration (database password, ```id_secret```, etc.).

Aditional, disable ```filter-with = gzip``` - Leaving the ```gzip``` filter enabled will cause UI failures because of the way templates are streamed once ```debug``` is set to ```False```. You will still be able (and are encouraged) to enable gzip in the proxy server.

During deployment, you may run into problems with failed jobs. By default, Galaxy removes files related to job execution. You can instruct Galaxy to keep files to failed jobs with: ```cleanup_job = onsuccess```.

### Switching to a Database server

The most importan recommendation is to swith to an actual database server. By default, Galaxy uses SQLite, wich is a serverless simple file database engine. Since it serverless, all of the database processing occurs in the Galaxy process itself. This has two downsides: it occupies the aforementioned GIL (meaning that the process is not free to do other tasks), and it is not nearly as efficient as a dedicated database server.

There are other drawbacks too, for example, when load increases with multiple users, the risk of transactional locks also increases.

For this reason, Galaxy also supports PostgreSQL and MySQL. PostgreSQL is much preferred since we've found it works better with the abstraction layer, SQLAlchemy.

To configure Galaxy, set ```database_connection``` in Galaxy's config file, ```config/galaxy.ini```. The syntax for a database URL is explained in the [SLQAlchemy documentation](http://docs.sqlalchemy.org/en/latest/core/engines.html).

Here follow two example database URLs with username and password:

```
postgresql://username:password@localhost/mydatabase
mysql://username:password@localhost/mydatabase
```

It's worth nothing that some platforms (for example, Debian/Ubuntu) store database sockets in a directory other than the database engine's default. If you're connecting to a database server on the same host as the Galaxy server and the socket is in a non-standard location, you'll need to user these custom arguments (these are the defaults for Debian/Ubuntu, change as necessary for your installation):

```
postgresql:///mydatabase?host=/var/run/postgresql
mysql:///mydatabase?unix_socket=/var/run/mysqld/mysqld.sock
```

### Using a Proxy server

Galaxy contains a standalone web server and can serve all of its content directly to clients. However, some tasks (such as serving static content) can be offloaded to a dedicated server that handles these tasks more efficiently. A proxy server also allows you to authenticate users externally using any method supported by the proxy (for example, Kerberos or LDAP), instruct browsers to cache content, and compress outbound data. Also, Galaxy's built-in web server does not support byte-range requests (required for many external display applications), but this functionality can be offloaded to a proxy server. In addition to freeing the GIL, compression and caching will reduce page load times.

Downloading and uploading data can also be moved to the proxy server.

Virtually any server that proxies HTTP should work, although we provide configuration examples for:

 * [Apache](https://wiki.galaxyproject.org/Admin/Config/ApacheProxy), and
 * [Nginx](https://wiki.galaxyproject.org/Admin/Config/nginxProxy), a high performance reverse proxy, used by the public Galaxy sites.

### Cleaning up datasets

When datasets are deleted from a history or library, it is simply marked as deleted and not actually removed, since it can later be undeleted. To free disk space, a st of scripts can be run (e.g. ```cron```) to remove that data files as specified by local policy.

### Rotate Log files

To use ```logrotate``` Galaxy log files, add a new file named "galaxy" to ```/etc/logrotate.d/``` directory with something like:

```
PATH_TO_GALAXY_LOG_FILES {
  weekly
  rotate 8
  copytruncate
  compress
  missingok
  notifempty
}
```

### Local Data

To get started with setting up local data, please see [Data integration](https://wiki.galaxyproject.org/Admin/DataIntegration)

 * All local reference genomes must be included in the ```builds.txt``` file.
 * Some tools (for example, Extract Genomic DNA) require that you cache (petentially huge) local ```.2bit``` data.
 * Other tools (form example, Bowtie2) require that you cache both ```.fasta data``` and ```tool-specific``` indexes.
 * The ```galaxy_dist/tool-data/``` directory contains a set of sample location (```<data_label>.loc```) file that describe the metadata and path to local data and indexes.
 * Installed tool packages from the Tool Shed may also include location files.
 * Comments in location files explain the expected format.
 * Wikis linked from [Data integration](https://wiki.galaxyproject.org/Admin/DataIntegration) explain how to obtain, create, or ```rsync``` many common data and indexes. See an individual Tool Shed repository's documentation for more details.

### Enable Upload Via FTP

File sizes have grown very large thanks to rapidly advancing sequencer technology, and it is not always practical to upload these files through the browser. Thankfully, a simple solution is to allow Galaxy user to upload them via FTP ad import those files in to their histories. Configuration for FTP is explained on the [Admin/COnfig/Upload via FTP](https://wiki.galaxyproject.org/Admin/Config/Upload%20via%20FTP) page.
