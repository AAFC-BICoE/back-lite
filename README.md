Back-Lite
=========

Automatic backups script for local & remote databases and filesystems.
Currently supported targets for backup are: mysql, postgresql, and files.

Getting Started
---------------
These steps should allow for a quick start:

  - Edit the beginning of the script: 
    ```bash
    CONFIG="/absolute/path/to/config"
    LOG="/absolute/path/to/log"
    ```
  - Edit the config file as necessary.  Some sane defaults are provided.
  - Create some target files in `serverListDir` (see *Target Files*).
  - Run the script, either directly or with a periodic 
    job scheduler such as `cron`.

Configuration
-------------
The configuration file is a bash script which is sourced at the
beginning of the script. The configuration should set the following
variables:

 Name               | Description
 ------------------ | -----------
 `destRoot`         | Absolute path to the backups storage directory
 `serverListDir`    | Directory where the target files are stored. See *Target Files*
 `logLevel`         | Integer in [1-4], see *Logging* for more information
 `zipper`           | Command used to compress backups and print to standard output
 `zipperExt`        | File extension associated with `zipper`
 `hasher`           | Hashing tool used to create checksums of the backups
 `hasherExt`        | File extension associated with `hasher`
 `defaultMysqlArgs` | Default arguments for `mysqldump`
 `defaultPsqlArgs`  | Default arguments for `pg_dump`
 `defaultTarArgs`   | Default arguments for `tar`

An example config file is provided for a quick start, it only 
requires changing `destRoot` and `serverListDir`

Target Files
------------
In the directory set as `serverListDir`, there need to be
correctly formatted files.  Each file is a *target*.  When the
script is run, it iterates over the target files and generates
a backup for each, using them as configuration.  Each line of a 
target file is a `key = value` style assignment.  The following 
keys are used:

 Key       | Description
 --------- | -----------
 `type`    | The type of backup. One of { mysql, psql, files }
 `host`    | The IP/hostname to connect to for sql database backups
 `port`    | The port on which to connect to the sql database
 `user`    | The username for the sql database connection
 `pass`    | The password of the mysql database
 `include` | mysql/psql: the databases to back up; files: the names of files
 `preCmnd` | Inserted immediately before the dumping command
 `args`    | Arguments provided to the dumping command. Overrides default arguments

Target files support comment lines (lines beginning with `#`) and
escaped line breaks (lines ending with `\`).  The name of a 
target file should have a meaningful name, as it is used for logging, 
as well as for the name of the backup files

Example target file, backs up 3 mysql databases:
```ini
# myDatabaseTarget
type    = mysql
host    = database.server.net
port    = 3306
include = employees \
          products \
          sales
```

Logging
-------
There are 4 logging levels. Each level implies those below.

 Level |  Name  | Description
:-----:| :----: | -----------
  `1`  | `ERR!` | Only log errors which are fatal
  `2`  | `WARN` | Log unexpected/non-ideal state, recovery is attempted
  `3`  | `INFO` | Log information about the script's execution progress, where things are being stored
  `4`  | `DBUG` | Maximum logging. Also enables logging for many tools used by the script

Backups
-------
Within the `destRoot` directory, a directory structure is created 
to keep backups well-organized.  The `destRoot` contains folders 
named for the year of the containing backups.  Within each, folders 
are created for each month.  

The backups within each of these folders are named for the target type,
the name of the target, and the date of the backup in the form 
`year_month_day`.  The backups are `tar` archives, and contain the 
compressed data dumps, as well as corresponding checksum files.

Example directory structure:
```
destRoot/
  2018/
    01/
      mysql_backup_myDatabase_18_01_15.tar
      mysql_backup_myDatabase_18_01_30.tar
    02/
      mysql_backup_myDatabase_18_02_15.tar
```

Contents of `mysql_backup_myDatabase_18_01_15.tar`:
```
./employees.sql.bz2
./employees.sql.bz2.md5
./products.sql.bz2
./products.sql.bz2.md5
./sales.sql.bz2
./sales.sql.bz2.md5
```

Advanced Usage
--------------
Many of the possible keys in *Target Files* are intended to support
extended functionality, such as tunneling the data dumps over a 
connection to a remote machine such as via `ssh`, providing specific 
arguments to the data dump command, or setting up an environment 
immidiately before the dump operation.

One example requirement is to connect to a remote machine using `ssh`,
rather than connecting to the database server directly.  The value of 
the `preCmnd` key is inserted on the same command line as the dump command
(ie. `mysqldump`/`pg_dump`/`tar`).

For example, assuming the following target:

```ini
# myRemoteMysqlTarget
type    = mysql
# host will be the local address because the
# command will be executed on the remote machine
host    = 127.0.0.1
port    = 3306
user    = backups
include = myData
preCmnd = ssh user@remote.server.net
```

When the script handles this target, it will run a command like:

`ssh user@remote.server.net mysqldump ...`

It's important to note that for a target like this to work, the `ssh` server
on the remote machine needs to be set up to execute the commands without any
user interaction or password prompting.

This functionality is particularly helpful with the `files` target type.
These targets are backed up using the `tar` tool, which takes as arguments
the paths to the relevant files.  These paths can be absolute or relative, 
but due to the way `back-lite` changes the working directory, relative 
paths will not work directly.  This can lead to some very ugly archives.

For example, assume the user wishes to back up the logs and userFiles
directories in the following file tree:

```
/
  var/
    lib/
      myApplication/
        logs/
          ...
        userFiles/
          ...
```

The target file could make the backup relative to the myApplication 
directory like this:

```ini
# myRelativeFilesTarget
  ...
include = logs userFiles
args    = -C /var/lib/myApplication
```

*\* note that the -C argument for `tar` is to `cd` to the given directory before archiving*

The target could also look like this:

```ini
# myAlternateRelativeFilesTarget
  ...
include = logs userFiles
preCmnd = cd /var/lib/myApplication &&
```