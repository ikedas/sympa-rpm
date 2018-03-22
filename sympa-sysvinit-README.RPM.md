Post-install instruction for RPM package

                                    Jun 11 2017
                                    by IKEDA Soji <ikeda@conversion.co.jp>

If you are reading this text, you may have successfully installed sympa
package.  A bunch of work is needed to start your Sympa service.

1 Run sympa_wizard
==================

  * Run ``sympa_wizard.pl``.  Sympa_wizard asks you some questions to setup
    Sympa and updates site configuration files.  In this process you must
    choose e-mail(s) of listmaster, database settings and so on.

2 Configure mail server
=======================

a. Sendmail
-----------

  * Edit /etc/sympa/aliases.sympa.sendmail file as you prefer.

  * Edit /etc/mail/sendmail.cf:

    * Add following paths to AliasFile:

      * /etc/sympa/aliases.sympa.sendmail

      * /var/lib/sympa/sympa_aliases

    * Or, if you are generating sendmail.cf by cf package, edit sendmail.mc
      to add paths above to argument of ``ALIAS_FILE`` macro.

  * Run ``newaliases``.

  * Run ``service sendmail condrestart``.

N.B.: This instruction is to use newaliases(1) maintainance tool.  If you
wish to use makemap(1), edit /etc/sympa/sympa.conf to add a line
``aliases_program makemap``.

b. Postfix
----------

  * Edit /etc/sympa/sympa.conf to add following line:
    ```
    aliases_program postalias
    ```

  * Edit /etc/sympa/aliases.sympa.postfix file as you prefer.

  * Edit /etc/postfix/main.cf:

    * Add following maps to $alias_maps:

      * hash:/etc/sympa/aliases.sympa.postfix

      * hash:/var/lib/sympa/sympa_aliases

    * Add following map to $alias_database:

      * hash:/etc/sympa/aliases.sympa.postfix

    * Add following line:
      ```
      recipient_delimiter = +
      ```

  * Run ``newaliases``.

  * Run ``su sympa -c /usr/sbin/sympa_newaliases.pl``.

  * Run ``service postfix condrestart``.

N.B.: This instruction is to use postalias(1) maintainance tool.  If you
wish to use postmap(1), edit /etc/sympa/sympa.conf to add a line
``aliases_program postmap``.

c. Other MTAs
-------------

About setting with exim and qmail, see following FAQ answers:

  * http://www.sympa.org/faq/exim (English)

  * http://www.sympa.org/faq/qmail (English)

(Optional) Virtual domains
--------------------------

About virtual domain settings see following pages:

  * http://www.sympa.org/manual/mail-aliases#virtual_domains (English)

  * http://www.sympa.org/ja/manual/mail-aliases#virtual_domains (Japanese)

More sophisticated virtual domain setting on Postfix is described at
following page:

  * http://sympa-ja.org/wiki/Virtual_domain_%28Postfix%29/Real_Virtual_Domain
    (English)

  * http://sympa-ja.org/wiki/Virtual_domain_%28Postfix%29 (Japanese)

(Optional) Automatic lists
--------------------------

See following page:

  * http://www.sympa.org/manual/list-families (English)

If you want to use sympa-milter, you may install ``sympa-milter`` package.
Note that sympa-milter 0.7 or later is required.

3 Setup database
================

  * Appropriate DBI driver (DBD) should be installed: DBD::mysql, DBD::Pg,
    DBD::Oracle, DBD::Sybase or DBD::SQLite.

  * Create a database and a database user dedicated for Sympa.  The user
    must have most of privileges on database.

a. MySQL or MariaDB
-------------------

  * MySQL 4.1.1 or later, and any releases of MariaDB will work.

  * Install perl-DBD-MySQL package.

  * Ensure that sympa.conf includes appropriate values for these parameters:
    db_type, db_name, db_host, db_user and db_passwd.

  * MyISAM or Aria, and InnoDB or XtraDB storage engines may work.  Check
    ``default-storage-engine`` and ``default-table-type`` options of your
    MySQL / MariaDB server.

  * Create database and database user:
    ```
    $ mysql
    mysql> CREATE DATABASE sympa CHARACTER SET utf8;
    mysql> GRANT ALL PRIVILEGES ON sympa.* TO <db_user>@<client host>
        -> IDENTIFIED BY '<db_passwd>';
    mysql> QUIT
    ```

  * Create table structure:
    ```
    # sympa.pl --health_check
    ```

N.B.: MySQL/MariaDB 5.5.3 or later provides ``utf8mb4`` character set
which covers full range of Unicode including such as chinese ideographs
used for persons' names.  As of Sympa-6.2a.33 r8753, both ``utf8`` and
``utf8mb4`` character sets are supported.  To use ``utf8mb4`` character
set, you might want to replace ``utf8`` in SQL statement above with
``utf8mb4``.

b. Oracle
---------

  * Oracle 7 or later may work.  Oracle 9i or later is recommended.

  * Install perl-DBD-Oracle package.

  * Ensure that sympa.conf includes appropriate values for these parameters:
    db_type, db_name, db_host, db_user, db_passwd and db_env.

    * "db_name" must be same as Oracle SID.

    * "db_env" should include definition of NLS_LANG and ORACLE_HOME.

  * Create database user:
    ```
    $ NLS_LANG=American_America.AL32UTF8; export NLS_LANG
    $ ORACLE_HOME=<Oracle home>; export ORACLE_HOME
    $ ORACLE_SID=<Oracle SID>; export ORACLE_SID
    $ sqlplus <system login>/<password>
    SQL> CREATE USER <db_user> IDENTIFIED BY <db_passwd>
      2  DEFAULT TABLESPACE <tablespace>
      3  TEMPORARY TABLESPACE <tablespace>;
    SQL> GRANT CREATE SESSION TO <db_user>;
    SQL> GRANT CREATE TABLE TO <db_user>;
    SQL> GRANT CREATE SYNONYM TO <db_user>;
    SQL> GRANT CREATE VIEW TO <db_user>;
    SQL> GRANT EXECUTE ANY PROCEDURE TO <db_user>;
    SQL> GRANT SELECT ANY TABLE TO <db_user>;
    SQL> GRANT SELECT ANY SEQUENCE TO <db_user>;
    SQL> GRANT RESOURCE TO <db_user>;
    SQL> QUIT
    ```

  * Create table structure:
    ```
    # sympa.pl --health_check
    ```

c. PostgreSQL
-------------

  * PostgreSQL 7.4 or later is required.

  * Install perl-DBD-Pg package.

  * Ensure that sympa.conf includes appropriate values for these parameters:
    db_type, db_name, db_host, db_user, db_passwd.

  * Create database and role:
    ```
    $ psql
    postgres=# CREATE ROLE <db_user> NOSUPERUSER NOCREATEDB NOCREATEROLE
    postgres=# NOINHERIT LOGIN ENCRYPTED PASSWORD '<db_passwd>';
    postgres=# CREATE DATABASE sympa OWNER <db_user> ENCODING 'UNICODE';
    postgres=# \q
    ```

  * Create table structure:
    ```
    # sympa.pl --health_check
    ```

d. SQLite
---------

  * Ensure that SQLite 3.x is installed.  As of Sympa-6.2a.33, SQLite 2.x
    and earlier are no longer supported.

  * Install perl-DBD-SQLite package.

  * Ensure that "db_name" in sympa.conf is absolute path to database file
    you want to create.

  * Create database file and table structure:
    ```
    # touch <db_name>
    # chown sympa:sympa <db_name>
    # sympa.pl --health_check
    ```

e. Other RDBMSes
----------------

See following page:

  * http://www.sympa.org/manual/database (English)

4 Configure HTTP server
=======================

a. Apache HTTP Server
---------------------

  * Install ``sympa-httpd`` package.  It also requires mod_fcgid package
    and so on.

  * Edit /etc/httpd/conf.d/sympa.conf as you prefer.

  * Run ``service httpd condrestart``.

b. lighttpd
-----------

  * Install ``sympa-lighttpd`` package.  It also requires lighttpd-fastcgi
    package and so on.

  * Edit /etc/lighttpd/sympa.conf as you prefer.

  * Edit /etc/lighttpd/lighttpd.conf to add following line at bottom:
    ```
    include /etc/lighttpd/sympa.conf
    ```

  * Run ``service lighttpd condrestart``.

c. nginx
--------

  * Install ``sympa-nginx`` package.  It also requires spawn-fcgi package
    and so on.

  * Edit /etc/nginx/conf.d/sympa.conf as you prefer.

  * Run ``service nginx condrestart``.

(Optional) Configure SOAP HTTP server
-------------------------------------

See following page:

  * http://www.sympa.org/manual/soap (English)

5 Start mailing list service
============================

  * Ensure that RDBMS is running (on SQLite, service need not to be run).

  * Ensure that MTA and HTTP server are running.

  * Run ``service sympa start``.

    * If you are using nginx, you must run ``service wwsympa start`` too.

    * If you are using nginx and wish to use SOAP service, you must run
      ``service sympasoap start`` too.

  * Start Web browser and access to <http://your.host.dom.ain/sympa>.

  * Follow ``First login`` link, input listmaster address chosen on the
    first section, and then click ``Send my password`` button.

  * E-mail describing how to choose your password (are you a listmaster,
    aren't you?) will be sent.  According to description, choose your
    password.

6 Automating Startup
====================

  * Run ``chkconfig sympa on`` to activate sympa service.

    * If you are using nginx, run ``chkconfig wwsympa on`` too.

    * If you are using nginx and wish to use SOAP service, run
      ``chkconfig sympasoap on`` too.

----
[![CC BY-SA 3.0](https://i.creativecommons.org/l/by-sa/3.0/80x15.png)](http://creativecommons.org/licenses/by-sa/3.0/)
