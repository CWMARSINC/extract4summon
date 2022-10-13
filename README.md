# NAME

extract4summon - extract, and send, bibliographic records for Summon

# SYNOPSIS

extract4summon \[-U|--user=string\] \[-h|--host=string\]
\[-d|--database=string\] \[-P|--password=string\] \[-p|--port=integer\]
\[-c|--configuration=file\] \[-l|--logfile=file\]
\[-o|--organization=string\] \[-n|--dry-run\] \[-f|--full\]
\[-s|--since=date\]

# DESCRIPTION

**extract4summon** extracts bibliographic records from an Evergreen ILS
database and sends them to the Summon service by Serials Solutions.

## MAIN OPTIONS

- -c | --configuration

    This required option takes an argument of the path to a configuration
    file.  The configuration file is expected to be in YAML format and is
    read by the [YAML::Tiny](https://metacpan.org/pod/YAML%3A%3ATiny) module.  See [CONFIGURATION](#configuration) for the layout
    and values in this file.

- -l | --logfile

    Specifies the path to a log file.  The default is `summon.log`
    relative to the current working directory.

- -o | --organization

    This required option's argument specifies the organization whose
    records are to be dumped.  The value must match one of the
    organization values specified in the configuration file.  See
    [CONFIGURATION](#configuration).

    Omitting this option will process each organization listed in the
    given configuration file in turn.  This is useful if you keep all of
    your configuraiton in one file.

- -f | --full

    Extract a full set of records owned by the organization.  This gets
    all of the records that have call numbers attached by the Evergreen
    organizational units listed in the `orgs` configuration option (see
    [CONFIGURATION](#configuration)).  This options is useful for the first run, or if
    you intend to send a complete set of records to refresh the Summon
    database.

    In the absence of the `--full` option, daily update files are
    produced.  These files include a file of new or changed records from
    the past 24 hours, and a file of records that should be deleted
    because the holding for one of the affected `orgs` have been deleted
    in the past 24 hours.

    If this option is not used, for instance in a crontab entry, a full
    update is extracted and sent on the first day of January, April, July,
    and October.  This behavior is not controlled by any options.

- -s | --since

    The `--since` option allows you to specify a date timestamp in ISO
    format that overrides the default 24 hour period for upate files.  The
    argument to this option can be either a date, i.e. `2022-08-02`, or a
    date with a time, i.e. `2022-08-02 16:18:00-04`, that PostgreSQL will
    understand.  When this option is used, **extract** will gather records
    that have been modified or deleted since the specified date and/or
    datetime.  This option is useful if the program has not been run on a
    daily basis for a while.

    The `--since` and `--full` options are mutually exclusive.  It is an
    error to specify them both at the same time.

- -n | --dry-run

    Extract the records, but do not send them.  This options prevents the
    upload of the file or files to the Summon server.  This is useful when
    testing or if you want to verify a file before sending it.

## DATABASE OPTIONS

The following command line options are used to specify the PostgreSQL
database connection:

- -U | --user

    Specifies a string argument specifying the database username to use
    when connecting to the database.  The `PGUSER` environment variable
    will be used if this option is not set.  See [ENVIRONMENT](#environment).  The
    default is `evergreen` if the option is not used.

- -h | --host

    The string argument specifies the host name or IP address of the
    database server.  The `PGHOST` environment variable will be used if
    this option is not set.  See [ENVIRONMENT](#environment).  The default is `db2` if
    the option is not used.

- -d | --database

    Specifies a string argument specifying the name of the database to
    connect to.  The `PGDATABASE` environment variable will be used if
    this option is not set.  See [ENVIRONMENT](#environment).  The default is
    `evergreen` if the option is not used.

- -p | --port

    Specifies the numeric port to use when connecting to PostgreSQL on the
    database host.  The `PGPORT` environment variable will be used if
    this option is not set.  See [ENVIRONMENT](#environment).  The default is `5432`
    if the option is not used.

- -P | --password

    Specifies a string argument specifying the password of the database
    user.  The `PGPASSWORD` environment variable will be used if this
    option is not set.  See [ENVIRONMENT](#environment).  The default is `evergreen`
    if the option is not used.

    **NB:** It is recommended that you **DO NOT USE** this option.  The
    value can be read by and process that can read your execution
    environment and get the command line of running processes.

## CONFIGURATION

**extract4summon** is configured using a [subset of
YAML](https://metacpan.org/pod/YAML::Tiny#YAML-TINY-SPECIFICATION)
supported by the [YAML::Tiny](https://metacpan.org/pod/YAML%3A%3ATiny) module.  The configuration variables
are described below.

- organization

    Takes a string to identify a unique section of configuration.  The
    string must be unique and corresponds to the `--organization` command
    line option.  The argument of that option is used to find the
    appropriate configuration in a multisection file or must match the
    value of this field in a single configuration file.

- orgs

    Takes a list of actor.org\_unit.ids from the database identifying the
    org. units whose records will be exported.  Even if only 1 is used, it
    must be configured as a list.

- sourceid

    A string identifying the organization to Summon.  The value will be
    provided by Serials Solutions and is used in the names of the upload
    files.

- loc-code

    The organization's MARC code.

    If you are in the United States, this is assigned by the Library of
    Congress.  The Library of Congress has [an online
    tool](https://www.loc.gov/marc/organizations/org-search.php) to look it
    up.  If you are in a country other than the United States of America,
    you will need to get this code from your nation's responsible agency.

- output-dir

    Directory where the output file will be written.  It must be readable
    and writable by the user running the summon process.  If not set, the
    current directory is used.

    By default this setting is not present in the configuration and must
    be added in order to be used.

 - FTP

    A set of key/value pairs used to configure the file upload.  Its keys
    and values are described below.

    - host

        The SFTP host where files are sent, typically
        `ftp.summon.serialssolutions.com`.  You will be given this value by
        Serials Solutions.

    - user

        The user name used to login to the SFTP server.  You will be given
        this value by Serials Solutions.

    - password

        The password for the SFTP server account.  You will be given this
        value by Serials Solutions.

    Despite the name of this option being `FTP`, this program uses SFTP
    to upload files.

## ENVIRONMENT

**extract4summon** uses the standard PostgreSQL connection environment
variables.

- PGUSER

    This variable can be used in lieu of the `--user` option.

- PGHOST

    This variable can be used in lieu of the `--host` option.

- PGDATABASE

    This variable can be used in lieu of the `--database` option.

- PGPORT

    This variable can be used in lieu of the `--port` option.

- PGPASSWORD

    This variable can be used in lieu of the `--password` option.  It is
    recommended that you **NOT** use this variable.  It poses a security
    risk.  You should set passwords in a .pgpass file instead.

# TODO

Logging could be done through [Sys::Syslog](https://metacpan.org/pod/Sys%3A%3ASyslog).

More settings or command line options could be moved to the
configuration file.

# COPYRIGHT

**extract4summon** is Copyright (C) 2021, 2022 by CW MARS, INC.

# AUTHOR

Jason Stephenson [jstephenson@cwmars.org](mailto:jstephenson@cwmars.org)

# LICENSE

**extract4summon** is free software; you can redistribute it and/or
modify it under the terms of the GNU General Public License as
published by the Free Software Foundation; either version 2 of the
License, or (at your option) any later version.

**extract4summon** is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
General Public License for more details.

# SEE ALSO

[YAML::Tiny](https://metacpan.org/pod/YAML%3A%3ATiny) - for configuration file format

[PostgreSQL Environment Variables](https://www.postgresql.org/docs/current/libpq-envars.html)
