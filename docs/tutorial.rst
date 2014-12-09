.. py:currentmodule:: steelscript.steelhead.core

SteelScript SteelHead Tutorial
================================

This tutorial will walk through the main components of the SteelScript
interfaces for Riverbed SteelHead Appliance.  It is assumed that
you have a basic understanding of the Python programming language.

The tutorial has been organized so you can follow it sequentially.
Throughout the examples, you will be expected to fill in details
specific to your environment.  These will be called out using a dollar
sign ``$<name>`` -- for example ``$host`` indicates you should fill in
the host name or IP address of a SteelHead appliance.

Whenever you see ``>>>``, this indicates an interactive session using
the Python shell.  The command that you are expected to type follows
the ``>>>``.  The result of the command follows.  Any lines with a
``#`` are just comments to describe what is happening.  In many cases
the exact output will depend on your environment, so it may not match
precisely what you see in this tutorial.

Background
----------

Riverbed SteelHead is the industry’s #1 optimization solution for
accelerated delivery of all applications across the hybrid enterprise.
SteelHead also provides better visibility into application and network
performance and the end user experience plus control through an
application-aware approach to hybrid networking and path selection based
on centralized, business intent-based policies for what you want to
achieve – as a business. SteelScript for SteelHead offers a set of interfaces
to control and work with a SteelHead appliance.

SteelHead Objects
-------------------

Interacting with a SteelHead leverages two key classes:

* :py:class:`SteelHead <steelhead.SteelHead>` - provides
  the primary interface to the appliance, handling initialization,
  setup, and communication via command line calls.
  
* :py:class:`CLIAuth <steelhead.CLIAuth>` - used for username/password
  based authentication for command-line access.

With that brief overview, let's get started.

Startup
-------

As with any Python code, the first step is to import the module(s) we
intend to use.  The SteelScript code for working with SteelHead
appliances resides in a module called :py:mod:`steelscript.steelhead.core`.  The main class in this module
is :py:class:`SteelHead <steelhead.SteelHead>`.  This object
represents a connection to a SteelHead appliance.

To start, start python from the shell or command line:

.. code-block:: bash

   $ python
   Python 2.7.3 (default, Apr 19 2012, 00:55:09)
   [GCC 4.2.1 (Based on Apple Inc. build 5658) (LLVM build 2335.15.00)] on darwin
   Type "help", "copyright", "credits" or "license" for more information.
   >>>

Once in the python shell, let's create a SteelHead object:

.. code-block:: python

   >>> from steelscript.steelhead.core import steelhead
   >>> auth = steelhead.CLIAuth(username=$username, password=$password)
   >>> sh = steelhead.SteelHead(host=$host, auth=auth)

At first the module of steelhead is imported. Two classes were used from
the steelhead module, including CLIAuth and SteelHead.
The object auth is created by instantiating the CLIAuth class
with username and password to access an SteelHead appliance. Afterwards,
a SteelHead object is created by instantiating the SteelHead class with
the hostname or IP address of the SteelHead appliance and the existing
authentication object. Note that the arguments ``$username`` and ``$password`` 
need to be replaced with the actual username and password, and the argument
``$host`` need to be replaced with the hostname or IP address of the SteelHead appliance. 

As soon as the ``SteelHead`` object is created, a connection is
established to the appliance, and the authentication credentials are
validated.  If the username and password are not correct, you will
immediately see an exception.

The ``sh`` object is the basis for all communication with the
SteelHead appliance.  We can get some basic version information by
simply executing the "show version" command

.. code-block:: python

   >>> print (sh.cli.exec_command("show version"))
   Product name:      rbt_sh
   Product release:   8.5.2
   Build ID:          #39
   Build date:        2013-12-20 10:10:02
   Build arch:        i386
   Built by:          mockbuild@bannow-worker4

   Uptime:            153d 10h 8m 29s

   Product model:     250
   System memory:     2063 MB used / 974 MB free / 3038 MB total
   Number of CPUs:    1
   CPU load averages: 0.23 / 0.15 / 0.10

Before moving on, exit the python interactive shell:

.. code-block:: python

   >>> [Ctrl-D]
   $

Extending the Example
---------------------

As a last item to help get started with your own scripts, we will post a new
script below, then walk through the key differences with the above-mentioned example.

.. code-block:: python

   #!/usr/bin/env python

   import steelscript.steelhead.core.steelhead as steelhead

   from steelscript.common.app import Application

   class ShowVersionApp(Application):

       def add_options(self, parser):
           super(ShowVersionApp, self).add_options(parser)
           parser.add_option('-H', '--host',
                             help='hostname or IP address')
           parser.add_option('-u', '--username', help="Username to connect with")
           parser.add_option('-p', '--password', help="Password to use")

       def validate_args(self):
           super(ShowVersionApp, self).validate_args()

           if not self.options.host:
               self.parser.error("Host name needs to be specified")

           if not self.options.username:
               self.parser.error("User Name needs to be specified")

           if not self.options.password:
               self.parser.error("Password needs to be specified")

       def main(self):
           auth = steelhead.CLIAuth(username=self.options.username,
                                    password=self.options.password)
           sh = steelhead.SteelHead(host=self.options.host, auth=auth)

           print (sh.cli.exec_command("show version"))

    
   ShowVersionApp().run()

Copy that code into a new file ``script``, and run it from command line. Note that
``hostname``, ``username``, ``password`` are now all items to be
passed to the script.

For example:

.. code-block:: python

   > python $script -H $host -u $username -p $password
   Product name:      rbt_sh
   Product release:   8.5.2
   Build ID:          #39
   Build date:        2013-12-20 10:10:02
   Build arch:        i386
   Built by:          mockbuild@bannow-worker4

   Uptime:            153d 10h 8m 29s

   Product model:     250
   System memory:     2063 MB used / 974 MB free / 3038 MB total
   Number of CPUs:    1
   CPU load averages: 0.23 / 0.15 / 0.10

Let us break down the script. First we need to import some items:

.. code-block:: python

   #!/usr/bin/env python

   import steelscript.steelhead.core.steelhead as steelhead

   from steelscript.common.app import Application

That bit at the top is called a shebang, it tells the system that it should
execute this script using the program after the '#!'. Besides steelhead module,
we are also importing the Application class, which is used to help parse arguments
and simplify the api call to run the application.

.. code-block:: python

   class ShowVersionApp(Application):

       def add_options(self, parser):
           super(ShowVersionApp, self).add_options(parser)
           parser.add_option('-H', '--host',
                             help='hostname or IP address')
           parser.add_option('-u', '--username', help="Username to connect with")
           parser.add_option('-p', '--password', help="Password to use")

       def validate_args(self):
           super(ShowVersionApp, self).validate_args()

           if not self.options.host:
               self.parser.error("Host name needs to be specified")

           if not self.options.username:
               self.parser.error("User Name needs to be specified")

           if not self.options.password:
               self.parser.error("Password needs to be specified")

This section begins the definition of a new class, which inherits from the
class Application.  This is some of the magic of object-oriented programming,
a lot of functionality is defined as part of Application, and we get all
of that for *free*, just by inheriting from it.  In fact, we go beyond that,
and *extend* its functionality by defining the function ``add_options`` and 
``validate_args``.  Here, we add options to pass in a hostname, a username and
a password, and then if the format of the passed-in arguments in the command
is wrong, a help message will be printed out. 

.. code-block:: python

       def main(self):
           auth = steelhead.CLIAuth(username=self.options.username,
                                    password=self.options.password)
           sh = steelhead.SteelHead(host=self.options.host, auth=auth)

           print (sh.cli.exec_command("show version"))

    
   ShowVersionApp().run()

This is the main part of the script, and remains similar to our previous
example. The last line calls the run function as defined in the Application class,
which executes the main function defined in the ShowVersionApp class.