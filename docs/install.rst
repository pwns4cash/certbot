=====================
Get Certbot
=====================

.. contents:: Table of Contents
   :local:


About Certbot
=============

Certbot is packaged for many common operating systems and web servers. Check whether
``certbot`` (or ``letsencrypt``) is packaged for your web server's OS by visiting
certbot.eff.org_, where you will also find the correct installation instructions for
your system.

.. Note:: Unless you have very specific requirements, we kindly suggest that you use the Certbot packages provided by your package manager (see certbot.eff.org_). If such packages are not available, we recommend using ``certbot-auto``, which automates the process of installing Certbot on your system.

.. _certbot.eff.org: https://certbot.eff.org


System Requirements
===================

The Let's Encrypt Client presently only runs on Unix-ish OSes that include
Python 2.6 or 2.7; Python 3.x support will hopefully be added in the future. The
client requires root access in order to write to ``/etc/letsencrypt``,
``/var/log/letsencrypt``, ``/var/lib/letsencrypt``; to bind to ports 80 and 443
(if you use the ``standalone`` plugin) and to read and modify webserver
configurations (if you use the ``apache`` or ``nginx`` plugins).  If none of
these apply to you, it is theoretically possible to run without root privileges,
but for most users who want to avoid running an ACME client as root, either
`letsencrypt-nosudo <https://github.com/diafygi/letsencrypt-nosudo>`_ or
`simp_le <https://github.com/kuba/simp_le>`_ are more appropriate choices.

The Apache plugin currently requires OS with augeas version 1.0; currently `it
supports
<https://github.com/certbot/certbot/blob/master/certbot-apache/certbot_apache/constants.py>`_
modern OSes based on Debian, Fedora, SUSE, Gentoo and Darwin.

Alternate installation methods
================================

If you are offline or your operating system doesn't provide a package, you can use
an alternate method for installing ``certbot``.

.. _certbot-auto:

Certbot-Auto
------------

The ``certbot-auto`` wrapper script installs Certbot, obtaining some dependencies
from your web server OS and putting others in a python virtual environment. You can
download and run it as follows::

  user@webserver:~$ wget https://dl.eff.org/certbot-auto
  user@webserver:~$ chmod a+x ./certbot-auto
  user@webserver:~$ ./certbot-auto --help

.. hint:: The certbot-auto download is protected by HTTPS, which is pretty good, but if you'd like to
          double check the integrity of the ``certbot-auto`` script, you can use these steps for verification before running it::

            user@server:~$ wget -N https://dl.eff.org/certbot-auto.asc
            user@server:~$ gpg2 --recv-key A2CFB51FA275A7286234E7B24D17C995CD9775F2
            user@server:~$ gpg2 --trusted-key 4D17C995CD9775F2 --verify certbot-auto.asc certbot-auto

The ``certbot-auto`` command updates to the latest client release automatically.
Since ``certbot-auto`` is a wrapper to ``certbot``, it accepts exactly
the same command line flags and arguments. For more information, see
`Certbot command-line options <https://certbot.eff.org/docs/using.html#command-line-options>`_.

For full command line help, you can type::

  ./certbot-auto --help all

Running with Docker
-------------------

Docker_ is an amazingly simple and quick way to obtain a
certificate. However, this mode of operation is unable to install
certificates or configure your webserver, because our installer
plugins cannot reach your webserver from inside the Docker container.

Most users should use the operating system packages (see instructions at
certbot.eff.org_) or, as a fallback, ``certbot-auto``. You should only
use Docker if you are sure you know what you are doing and have a
good reason to do so.

You should definitely read the :ref:`where-certs` section, in order to
know how to manage the certs
manually. `Our ciphersuites page <ciphers.html>`__
provides some information about recommended ciphersuites. If none of
these make much sense to you, you should definitely use the
certbot-auto_ method, which enables you to use installer plugins
that cover both of those hard topics.

If you're still not convinced and have decided to use this method,
from the server that the domain you're requesting a cert for resolves
to, `install Docker`_, then issue the following command:

.. code-block:: shell

   sudo docker run -it --rm -p 443:443 -p 80:80 --name certbot \
               -v "/etc/letsencrypt:/etc/letsencrypt" \
               -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
               quay.io/letsencrypt/letsencrypt:latest certonly

Running Certbot with the ``certonly`` command will obtain a certificate and place it in the directory
``/etc/letsencrypt/live`` on your system. Because Certonly cannot install the certificate from
within Docker, you must install the certificate manually according to the procedure
recommended by the provider of your webserver.

For more information about the layout
of the ``/etc/letsencrypt`` directory, see :ref:`where-certs`.

.. _Docker: https://docker.com
.. _`install Docker`: https://docs.docker.com/engine/installation/

Operating System Packages
-------------------------

**FreeBSD**

  * Port: ``cd /usr/ports/security/py-certbot && make install clean``
  * Package: ``pkg install py27-certbot``

**OpenBSD**

  * Port: ``cd /usr/ports/security/letsencrypt/client && make install clean``
  * Package: ``pkg_add letsencrypt``

**Arch Linux**

.. code-block:: shell

   sudo pacman -S certbot

**Debian**

If you run Debian Stretch or Debian Sid, you can install certbot packages.

.. code-block:: shell

   sudo apt-get update
   sudo apt-get install certbot python-certbot-apache

If you don't want to use the Apache plugin, you can omit the
``python-certbot-apache`` package.

Packages exist for Debian Jessie via backports. First you'll have to follow the
instructions at http://backports.debian.org/Instructions/ to enable the Jessie backports
repo, if you have not already done so. Then run:

.. code-block:: shell

   sudo apt-get install certbot python-certbot-apache -t jessie-backports

**Fedora**

.. code-block:: shell

    sudo dnf install certbot python2-certbot-apache

**Gentoo**

The official Certbot client is available in Gentoo Portage. If you
want to use the Apache plugin, it has to be installed separately:

.. code-block:: shell

   emerge -av app-crypt/certbot
   emerge -av app-crypt/certbot-apache

When using the Apache plugin, you will run into a "cannot find a cert or key
directive" error if you're sporting the default Gentoo ``httpd.conf``.
You can fix this by commenting out two lines in ``/etc/apache2/httpd.conf``
as follows:

Change

.. code-block:: shell

   <IfDefine SSL>
   LoadModule ssl_module modules/mod_ssl.so
   </IfDefine>

to

.. code-block:: shell

   #<IfDefine SSL>
   LoadModule ssl_module modules/mod_ssl.so
   #</IfDefine>

For the time being, this is the only way for the Apache plugin to recognise
the appropriate directives when installing the certificate.
Note: this change is not required for the other plugins.

**Other Operating Systems**

OS packaging is an ongoing effort. If you'd like to package
Certbot for your distribution of choice please have a
look at the :doc:`packaging`.

Installing from source
----------------------

Installation from source is only supported for developers and the
whole process is described in the :doc:`contributing`.

.. warning:: Please do **not** use ``python setup.py install`` or
   ``python pip install .``. Please do **not** attempt the
   installation commands as superuser/root and/or without virtual
   environment, e.g. ``sudo python setup.py install``, ``sudo pip
   install``, ``sudo ./venv/bin/...``. These modes of operation might
   corrupt your operating system and are **not supported** by the
   Certbot team!

