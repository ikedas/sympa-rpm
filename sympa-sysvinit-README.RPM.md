Post-install instruction for RPM package

                                    Aug 28 2018
                                    by IKEDA Soji <ikeda@conversion.co.jp>

If you are reading this text, you may have successfully installed sympa
package.  A bunch of work is needed to start your Sympa service.

1 Initial configuration
=======================

  * Edit ``/etc/sympa/sympa.conf`` as you prefer.

For more details see "Generate initial configuration" in "Sympa Administration
Manual":
https://sympa-community.github.io/manual/install/generate-initial-configuration.html

2 Setup database
================

See "Setup database" in "Sympa Administration Manual":
https://sympa-community.github.io/manual/install/setup-database.html

You have to install appropriate package providing database driver:
perl-DBD-MySQL, perl-DBD-Oracle, perl-DBD-Pg or perl-DBD-SQLite.

3 Configure mail server
=======================

See "Configure mail server" in "Sympa Administration Manual":
https://sympa-community.github.io/manual/install/configure-mail-server.html

(Optional) Automatic lists
--------------------------

See "Automatic list creation" in "Sympa Administration Manual":
https://sympa-community.github.io/manual/customize/automatic-lists.html

If you want to use sympa-milter, you may install ``sympa-milter`` package.
Note that sympa-milter 0.7 or later is required.

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

See "SOAP/HTTP API" in "Sympa::Administration Manual":
https://sympa-community.github.io/manual/customize/soap-api.html

You have to install ``perl-SOAP-Lite`` package.

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
