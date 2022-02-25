===================
Plone 4 on mod_wsgi
===================

This buildout is an example of how to get the latest Plone 4 with Zope 2.13 working under mod_wsgi.

Benefits of using WSGI
======================

Several key benefits of running Zope2 under WSGI are:

1. You can use different WSGI servers (including Apache via mod_wsgi) to serve up your Zope application by changing configuration.

2. You can employ various pieces of WSGI middleware to show error tracebacks in differ- ent ways in your web browser (e.g. egg:Paste#cgitb or egg:Paste#evalerror) while you develop.

3. You can use WSGI middleware to perform access and error logging (replacing the analogue of those features built in to Zope 2).

4. You can run non-Zope applications (e.g.
`Pyramid <http://docs.pylonshq.com/pyramid/dev/index.html>`_,
`Pylons <http://docs.pylonshq.com/>`_, 
`Roundup <https://roundup.sourceforge.net/>`_,
`Trac <https://trac.edgewall.org>`_, static file serving, etc) in the same process space as your Zope application(s) by changing your Paste configuration.

5. You can skin dissimilar applications using applications like 
`Deliverance <http://deliveranceproject.org>`_, or 
`dv.xdvserver <https://pypi.python.org/pypi/dv.xdvserver>`_ which are middleware tools that perform output transformation.

6. You can externalize some functionality (like authentication) using middleware.

7. You can create your own middleware pieces to perform deployment-specific tasks when it's more appropriate to do this than it is to understand some specific Zope internals (e.g. transform all HTML output, adding a footer).

8. Your software will be compatible with yet-to-be-created WSGI servers, which may offer specific benefits for various types of deployments.

9. You can use WSGI middleware with non-Zope WSGI applications, which can provide a bit of development continuity for Zope developers when they need to develop non-Zope applications.

Zope2 paste configuration
*************************

TODO

Configuring authentication
--------------------------

TODO

Configuring error logging
-------------------------

The Zope 2 through-the-web ``error_log`` object will not show any exception history. Instead, you must use the repoze.errorlog ``/__error_log__`` URL to see an analogue of this information (or just view console output).

The repoze.errorlog is enabled in the zope2.ini file::

    [pipeline:main]
    pipeline =
        ...
        egg:repoze.errorlog#errorlog
        zope2


Configuring Apache and mod_wsgi
*******************************

Installing mod_wsgi
-------------------

Apache2 needs to be installed as well as the ``libapache2-mod-wsgi`` module. This is assuming you are using Ubuntu. Other Linux distros may name this module something different::

    $ apt-get install libapache2-mod-wsgi
    
Please note: if you are running Ubuntu Maverick 10.10, the default version of Python that will get installed with libapache2-mod-wsgi is Python 2.7 and Python 3, which obviously won't work with Zope 2.13. 

So you'll have to compile mod_wsgi from source, in order to get make sure the same version of Python 2.6 is used with mod_wsgi as is used with Zope.

Compiling mod_wsgi from source
------------------------------

You only need to do this if the package libapache2-mod-wsgi doesn't use Python 2.6::

    $ apt-get install apache2-threaded-dev
    $ wget http://modwsgi.googlecode.com/files/mod_wsgi-3.3.tar.gz
    $ tar xzf mod_wsgi-3.3.tar.gz
    $ cd mod_wsgi-3.3
    $ ./configure --with-python=/usr/bin/python2.6
    $ make
    $ sudo make install

Notice above that when we configure mod_wsgi, we are using the ``--with-python`` parameter to indicate we want to use Python 2.6.

Then you'll need to make a file ``/etc/apache2/mods-available/wsgi.load`` and put the following in it::

    LoadModule wsgi_module /usr/lib/apache2/modules/mod_wsgi.so

Once you've created this file, then you need to enable the module with this command, and restart Apache::

    $ a2enmod wsgi
    $ /etc/init.d/apache2 restart

Enabling the Apache config
--------------------------

Our buildout will create an ``etc/apache2.conf`` file for us. All we need to do is to symlink it into the ``/etc/apache2/sites-available`` directory, and enable the site, and reload the Apache config::

    $ cd /etc/apache2/sites-available
    $ ln -s /home/zope/plone4wsgi.buildout/etc/apache2.conf plone4
    $ a2ensite plone4
    $ /etc/init.d/apache2 reload
    
Configuring Zeo server and zope.conf
************************************

By default, mod_wsgi will use a single process to run the application. Since this configuration is intended for production use, it may be desirable to have a higher number of processes available to serve the application. The ZODB that Zope uses comes with a server named ZEO (Zope Enterprise Objects) that allows us to add as many processes to our configuration as our system permits, providing unlimited horizontal scalability. Typically, the recommended number of processes is one for each core in the system's processors. Let's set up a ZEO server and configure the Zope process to connect to it.

Setting the directory permissions
---------------------------------

The ``var`` directory needs to be owned by the same user that is running Apache. On Ubuntu this means the ``www-data`` user. So we need to change the ownership of this dir to this user::

    $ chown -R www-data var
    
Start up the Zeo server 
-----------------------

You must start up the Zeo server::

    $ ./bin/zeo start
    . 
    daemon process started, pid=8383
    

Augmenting the number of processes
----------------------------------

Recall that we mentioned earlier that mod_wsgi runs the application in a single process by default. To really take advantage of ZEO, we want to have more processes available. We need to make a small addition to our mod_wsgi Apache virtual host configuration for that. Change the WSGIDaemonProcess line near the top to look like this::

    WSGIDaemonProcess grok user=cguardia group=cguardia processes=2 threads=4 maximum-requests=10000

In this example, we'll have two processes running, with four threads each. Using ZEO and mod_wsgi, we now have an scalable site.
    
Resources
*********

`Developing with repoze.zope2 <http://static.repoze.org/misc/developingwithrepoze-zope2.pdf>`_ by Chris McDonough

`Installing and setting up Grok under mod_wsgi <http://grok.zope.org/documentation/tutorial/installing-and-setting-up-grok-under-mod-wsgi/>`_ by Carlos de la Guardia
