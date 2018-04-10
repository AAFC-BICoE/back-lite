Backups System (name TBD)
=========================

Automatic backups script for local & remote databases and filesystems.
Currently supported targets for backup are: mysql, postgresql, and files.

Getting Started
---------------
These steps should allow for a quick start:

  - Set up configuration (see Configuration)
  - Edit the beginning of the script: 
    ```bash
    CONFIG=/absolute/path/to/config
    LOG=/absolute/path/to/log
    ```
  - Create some target files in the `serverListDir`
    that was described in the configuration file
  - Run the script, either directly or with a periodic 
    job scheduler such as `cron`

Configuration
-------------
The configuration file is a bash script which is sourced at the
beginning of the script. The configuration should set the following
variables:

 Name             | Description
 ---------------- | -----------
 destRoot         | A directory structure is created under this path to store the backups
 serverListDir    | Directory where the target files are stored. See *Target Files*
 logLevel         | Integer in [1-4], see *Logging* for more information
 zipper           | Command used to compress backups and print to standard output
 zipperExt        | File extension associated with `$zipper`
 hasher           | Hashing tool used to create checksums of the backups
 hasherExt        | File extension associated with `$hasher`
 defaultMysqlArgs | Default arguments for `mysqldump`
 defaultPsqlArgs  | Default arguments for `pg_dump`

Target Files
------------
In the directory set as `serverListDir`, there need to be
correctly formatted files.  Each file is one *target*.  When the
script is run, it iterates over the target files and generates
a backup for each, using them as configuration.  Each line of a 
target file is a `key = value` style assignment.  The following 
keys are used:

 Key     | Description
 ------- | -----------
 type    | The type of backup. One of { mysql, psql, files }
 host    | The IP/hostname to connect to for sql database backups
 port    | The port on which to connect to the sql database
 user    | The username for the sql database connection
 pass    | The password of the mysql database
 include | mysql/psql: the databases to back up; files: the names of files
 preCmnd | Inserted immediately before the dumping command
 args    | Arguments provided to the dumping command. Overrides default arguments

Target files support comment lines (lines beginning with \#) and 
escaped line breaks (lines ending with \\)

The name of a target file should have a meaningful name, as it
is used for logging, as well as for the name of the backup files

Example target file, backs up 3 local mysql databases:
```
type    = mysql
host    = 127.0.0.1
port    = 3306
include = employees \
          products \
          sales
```

Logging
-------
There are 4 logging levels. Each level implies those below.

 Level | Name | Description
:-----:|:----:| -----------
   1   | ERR! | Only log errors which are fatal
   2   | WARN | Unexpected/Non-ideal state, recovery is attempted
   3   | INFO | log information about the script's execution progress
   4   | DBUG | verbose logging. Enables logging for many of the tools used by the script

Backups
-------
Within the `destRoot` directory, a directory structure is created 
to keep backups well-organized.  The `destRoot` contains folders 
named for the year of the containing backups.  Within each, folders 
are created for each month.  

The backups within each of these folders are named for the target type,
the name of the target, and the date of the backup in the form 
(year\_month\_day).  The backups are `tar` archives, and contain the 
compressed backups, as well as corresponding checksum files.
