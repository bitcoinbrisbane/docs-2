# Install iRedAdmin on Debian, Ubuntu

!!! attention

	 Check out the lightweight on-premises email archiving software developed by iRedMail team: [Spider Email Archiver](https://spiderd.io/).

[TOC]

!!! warning

    This tutorial is out of date and deprecated. Since iRedMail-0.9.8, Apache
    was removed and Nginx is the only one web server.

> This tutorial is used to install iRedAdmin from scratch, running under Apache
> web server.
>
> If you already have iRedAdmin open source edition or old iRedAdmin-Pro release
> installed, you can simply migrate it to the latest iRedAdmin by follow our short
> tutorial: [Migrate or upgrade iRedAdmin](./migrate.or.upgrade.iredadmin.html).

## Requirements

iRedMail will install all required packages for you, you don't need to
install them manually, below info just for your reference.

* The latest iRedMail release. [Download the latest iRedMail](https://www.iredmail.org/download.html).
  NOTE: You must have iRedMail installed on server first.

* `Apache` 2.2+. Web server.

    * `mod_wsgi` 2.1+. Apache module used to host Python application which supports the Python WSGI interface.

* `Python` 2.4+, core programming language. Warning: Python 3.x is not supported yet.

    * `web.py`, 0.32+. A python-powered web framework.
    * `MySQLdb`: A thread-compatible interface to the popular MySQL database server that provides the Python database API. Required if you store mail accounts in OpenLDAP, MySQL or MariaDB.
    * `Python-LDAP` 2.3.7+: An object-oriented API to access LDAP directory servers from Python programs. Required if you store mail accounts in OpenLDAP.
    * `Python-psycopg2`: interface to the PostgreSQL database server from Python programs. Required if you store mail accounts in PostgreSQL.

## Download iRedAdmin and configure Apache web server

* Download iRedAdmin open source edition [here](https://dl.iredmail.org/yum/misc/).
  If you're trying to install iRedAdmin-Pro, please [contact us](https://www.iredmail.org/contact.html)
  to get download link of iRedAdmin-Pro.

* Copy iRedAdmin to `/opt/www`, set correct file permissions, and create symbol link.

```
# tar xjf iRedAdmin-x.y.z.tar.bz2 -C /opt/www/
# cd /opt/www/
# chown -R iredadmin:iredadmin iRedAdmin-x.y.z
# chmod -R 0555 iRedAdmin-x.y.z
# ln -s iRedAdmin-x.y.z iredadmin
```

* Add apache configure file: `/etc/apache2/conf.d/iredadmin.conf`.

    Note: If you're running Ubuntu 14.04 or later releases, it's
    `/etc/apache2/conf-available/iredadmin.conf`. After you added this file,
    enable it with command `a2enconf iredadmin`.

```
WSGISocketPrefix /var/run/wsgi
WSGIDaemonProcess iredadmin user=iredadmin threads=15
WSGIProcessGroup iredadmin

AddType text/html .py

<Directory /opt/www/iredadmin/>
    Order deny,allow
    Allow from all
</Directory>
```

* Edit `/etc/apache2/sites-enabled/default-ssl`, make iredadmin accessible via HTTPS.
  Add below lines before `</VirtualHost>`:

```
WSGIScriptAlias /iredadmin /opt/www/iredadmin/iredadmin.py/
Alias /iredadmin/static /opt/www/iredadmin/static/
```

* Enable mod_wsgi module and restart Apache service:

```
# a2enmod wsgi
# service apache2 restart
```

## Create required MySQL database and grant privileges

* Create MySQL database: `iredadmin`.

```
# mysql -uroot -p
mysql> CREATE DATABASE iredadmin DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
mysql> USE iredadmin;
mysql> SOURCE /opt/www/iredadmin/docs/samples/iredadmin.sql;
```

* Grant privileges to iredadmin user and set password for it. WARNING: Here we
  use 'secret_passwd' as password of iredadmin user, please replace it with
  your own password.

```
# mysql -uroot -p
mysql> GRANT SELECT,INSERT,UPDATE,DELETE ON iredadmin.* TO iredadmin@localhost IDENTIFIED BY 'secret_passwd';
mysql> FLUSH PRIVILEGES;
```

## Configure iRedAdmin

* Copy sample config file, and make it not world-writeable.

    * settings.py.ldap.sample: sample config file for OpenLDAP backend
    * settings.py.mysql.sample: sample config file for MySQL/MariaDB backend
    * settings.py.pgsql.sample: sample config file for PostgreSQL backend

```
# cd /opt/www/iredadmin/
# cp settings.py.[backend].sample settings.py
# chown iredadmin:iredadmin settings.py
# chmod 0400 settings.py
```

* Update settings.py with correct values. Please read `settings.py` for more
  information, it's self-documented.

* Restart apache web server.

```
# service apache2 restart
```

## Access iRedAdmin

Open your web browser to access iRedAdmin: `httpS://your_server_ip_address/iredadmin/`

Make sure you use `HTTPS://` instead of `HTTP://`.

## Troubleshooting & Debug

If iRedAdmin doesn't work as expected, you can simplily set `DEBUG = True` in
`settings.py`, restart apache web server, use your favourite web browser to
access it again, create a new [forum](https://forum.iredmail.org/) topic and
paste error message in your forum topic.
