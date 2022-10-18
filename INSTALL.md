# INSTALLATION

It is recommended to install **extract4summon** on an existing utility
server where scheduled jobs for Evergreen already run.  This machine
must be able to connect to an Evergreen PostgreSQL database in order
to run ready-only queries.

In order to install and use **extract4summon**, you must perform the following steps:

 1. Install the modules listed in the [PREREQUISITES](#prerequisites).

 2. Clone the git repository:

    `git clone https://gitlab.cwmars.org/jstephenson/extract4summon.git`

 3. Edit the sample configuration file [summon.yml](summon.yml).

    The sample configuration contains dummy values for the Sys1 system
    from the sample Evergreen data.  It also has "ChangeMe" placeholders
    for the STFP user name and passwords.  You need to change thes values
    as appropriate for your Evergreen installation and for the user name,
    password, and other SFTP parameters that Serials Solutions provided to
    you.

    You can configure summon for multiple institutions by using multiple
    configuration files or by adding multiple configurations in a single
    file.  Be sure to separate the multiple configurations in a single
    file with a line containing three dashes: `---`.

## CRON

Once you have sent an initial extract to Summon for validation, you
will want to setup a daily cron job to run **extract4summon** for each
organization in your configuration file(s).  Assuming that you have
cloned or copied the code to `/home/opensrf` on a utility server, the
following line should run it at 11 PM for all organizations in the
configuration file:

`0 23 * * * cd /home/opensrf/extract4summon && ./extract4summon -c summon.yml`

Note that if you have more than one configuration file, you must add
additional crontab entries, or you can write a shell script to run
them one after the other and substtute the script name for command
part of the above crontab entry.  Keep in mind that the above is
merely a suggestion.  You can change the time that you run the extract
as appropriate for your situation.

The above sample also assumes that the PostgreSQL environment
variables have been set in the crontab file.  You could set the
`HOME` variable to further simplify the setup.  Such instructions are
beyond the scope of this document and can be found in the
[crontab(5)](http://man.he.net/man5/crontab) documentation.

## PREREQUISITES

**extract4summon** uses the following Perl modules that are not part of
a standard Perl distribution.  You will need to add these to your
server.

- DBI
- DBD::Pg
- IO::Pty
- MARC::Record
- MARC::File::XML
- Net::SFTP::Foreign
- YAML::Tiny
